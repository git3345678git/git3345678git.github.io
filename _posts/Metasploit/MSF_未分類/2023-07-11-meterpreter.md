---
layout: post
title: "meterpreter"
description: ""
date: 2023-07-11
categories:
  - Metasploit
  - MSF_未分類
author: ""
tags: ['Metasploit']
---



# meterpreter
https://www.twblogs.net/a/5d4c559cbd9eee5327fc3ac3


```
//進入desktop
getdesktop


//列出當前目錄
ls   or (dir)

//列出本機當前目錄
lls


// 創建目錄1111
mkdir 1111


//截圖
screenshot


//監控
screenshare


//查看目標當前路徑
pwd   
getwd


//查看本機當前路徑
getlwd



//創建 aaa 目錄
mkdir aaa



//複製 A檔案為B檔案
cp 123.txt  456.txt



//edit 檔案
edit 456.txt


//查看掛載
show_mount


//尋找檔案
search -f *.doc


//下載 檔案
download 123.txt



//上傳 檔案
upload hello.exe



//查看hash
checksum md5 hello.exe

```




```
Core Commands
=============

    Command                   Description
                       -
    cat           Read the contents of a file to the screen
    cd            Change directory
    checksum      Retrieve the checksum of a file
    cp            Copy source to destination
    del           Delete the specified file
    dir           List files (alias for ls)
    download      Download a file or directory
    edit          Edit a file
    getlwd        Print local working directory
    getwd         Print working directory
    lcat          Read the contents of a local file to the screen
    lcd           Change local working directory
    lls           List local files
    lpwd          Print local working directory
    ls            List files
    mkdir         Make directory
    mv            Move source to destination
    pwd           Print working directory
    rm            Delete the specified file
    rmdir         Remove directory
    search        Search for files
    show_mount    List all mount points/logical drives
    upload        Upload a file or directory


Stdapi: Networking Commands
===========================

    Command       Description
           -
    clearev       Clear the event log
    drop_token    Relinquishes any active impersonation token.
    execute       Execute a command
    getenv        Get one or more environment variable values
    getpid        Get the current process identifier
    getprivs      Attempt to enable all privileges available to the current process
    getsid        Ge