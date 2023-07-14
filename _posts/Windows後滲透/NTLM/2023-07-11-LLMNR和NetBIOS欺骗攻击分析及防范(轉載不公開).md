---
layout: post
title: "LLMNR和NetBIOS欺骗攻击分析及防范(轉載不公開)"
description: ""
date: 2023-07-11
categories:
  - Windows後滲透
  - NTLM
author: ""
tags: ['Windows後滲透', 'NTLM']
---



# LLMNR和NetBIOS欺骗攻击分析及防范(轉載不公開)
###### tags: `Windows 域滲透`





### LLMNR协议

(1) 主机在自己的内部名称缓存中查询名称。如果在缓存中没有找到名称，那么主机就会向自己配置的主DNS服务器发送查询请求。如果主机没有收到回应或收到了错误信息，主机还会尝试搜索配置的备用DNS服务器。如果主机没有配置DNS服务器，或者如果在连接DNS服务器的时候没有遇到错误但失败了，那么名称解析会失败，并转为使用LLMNR。


(2) 主机通过UDP写发送多播查询，查询主机名对应的IP地址，这个查询会被限制在本地子网(也就是所谓的链路局部)内。



(3) 链路局部范围内每台支持LLMNR，并且被配置为响应传入查询的主机在收到这个查询请求后，会将被查询的名称和自己的主机名进行比较。如果没有找到匹配的主机名，那么计算机就会丢弃这个查询。如果找到了匹配的主机名，这台计算机会传输一条包含了自己IP地址的单播信息给请求该查询的主机。


### NetBIOS的定义
NetBIOS(Network Basic Input Output System)
NetBIOS Name Service (NBNS)

网络基本输入输出系统，它提供了OSI模型中的会话层服务，让在不同计算机上运行的不同程序，可以在局域网中，互相连线，以及分享数据。严格来说，Netbios是一种应用程序接口(API)，系统可以利用WINS服务、广播及Lmhost文件等多种模式将NetBIOS名解析为相应IP地址，几乎所有的局域网都是在NetBIOS协议的基础上工作的。NetBIOS也是计算机的标识名称，主要用于局域网内计算机的互访。NetBIOS的工作流程就是正常的机器名解析查询应答过程。在Windows操作系统中，默认情况下在安装 TCP/IP 协议后会自动安装NetBIOS。



#### 当访问者输入一个不存在的域名，而hosts文件和DNS服务器均无法给出解析时，Windows就会发送NBNS(UDP)请求，询问本地广播域中的所有主机。


### 查询顺序为

#### 簡略版:
```
1.本地hosts文件解析
2.使用DNS解析
3.使用LLMNR解析
4.使用NBNS解析
```

#### 詳細版:

在使用传输控制协议 (TCP) 和互联网协议 (IP) 堆栈的网络（包括当今大多数网络）上，需要将资源名称转换为 IP 地址以连接到这些资源。因此，网络需要将“resource.domain.com”解析为“xxxx”的 IP 地址，以便准确知道将流量发送到何处。Microsoft Windows 客户端在尝试将名称解析为地址时遵循一系列方法，在成功将名称与 IP 地址匹配时停止搜索。

```
1.检查以确认请求是否针对本地机器名称。
2.检查最近成功解析的名称的本地缓存。
3.搜索本地主机文件，该文件是存储在本地计算机上的 IP 地址和名称列表。根据设备的不同，此文件可能已加载到本地缓存中。
4.查询 DNS 服务器（如果已配置）。
5.如果启用了 LLMNR，则跨本地子网广播 LLMNR 查询以询问其对等方进行解析。
6.如果启用了 NetBIOS，如果名称不在本地 NetBIOS 缓存中，则通过向本地子网广播 NetBIOS-NS 查询来尝试 NetBIOS 名称解析。如果如此配置，此步骤可能会使用 Windows Internet 名称服务 (WINS) 服务器以及 LAN 管理器主机 (LMHOSTS) 文件。
```




也就是说，如果在缓存中没有找到名称，DNS名称服务器又请求失败时，Windows系统就会通过链路本地多播名称解析（LLMNR）和Net-BIOS名称服务（NBT-NS）在本地进行名称解析。这时，客户端就会将未经认证的UDP广播到网络中，询问它是否为本地系统的名称。这就产生了一个安全问题。由于该过程未被认证，并且广播到整个网络，从而允许网络上的任何机器响应并声称是目标机器。通过工具监听LLMNR和NetBIOS广播，攻击者可以伪装成受害者要访问的目标机器，并从而让受害者交出相应的登陆凭证。


 
如上所述，为了得到通信对象的IP地址，137端口就要交换很多信息包。在这些信息包中，包括有很多信息。利用广播管理计算机名时，会向所有电脑发送这些信息。如果使用NBT，就会在用户没有查觉的情况下，由电脑本身就会向外部散布自己的详细信息。

### 防范
1.最实用的方法在每台计算机的 NIC 上禁用 NetBIOS 并通过 DHCP 禁用 LLMNR ，但是这对于一些需要使用到NetBIOS和LLMNR服务的组织不太友好，所以更好的方法是每个系统的主机防火墙上通过阻止 NetBIOS 协议和 TCP 端口 139 以及 LLMNR UDP 端口来限制出站 NetBIOS 和 LLMNR 流量端口为5355(LLMNR协议工作再5355端口)。这样就可以有效的防止这几个端口流量出站。

2.但是使用禁用端口流量的方法并不保险，可能攻击者会使用端口转发的方法从另外端口将NetBIOS和LLMNR的流量转发出站。所以另外一种防范办法就是将默认端口改成一些不会引起攻击者注意的高端口，即将端口重定向。这种重定向的方法修改注册表里面的PortNumber修改即可。

```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TerminalServer\Wds\rdpwd\Tds\tcp]
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TerminalServer\WinStations\RDP-Tcp]
```




以上文章內容截取自 先知社區 
標題:LLMNR和NetBIOS欺骗攻击分析及防范
創作者:茶寂messi996
鏈接:https://xz.aliyun.com/t/9714