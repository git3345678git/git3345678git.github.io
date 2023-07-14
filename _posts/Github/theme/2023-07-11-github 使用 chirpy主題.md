---
layout: post
title: "github 使用 chirpy主題"
description: ""
date: 2023-07-11
categories:
  - Github
  - theme
author: ""
tags: ['Github', 'jekyll']
---





教學文:
https://zhangge6.github.io/posts/jekell%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%B5%81%E7%A8%8B%E8%AE%B0%E5%BD%95/

https://zhgchg.li/posts/a0c08d579ab1/#%E8%83%8C%E6%99%AF


這篇是記錄不懂的知識:


Jekyll是一個簡單的靜態網站生成器，用於生成個人，專案或組織的網站。 它由GitHub聯合創始人湯姆·普雷斯頓·沃納用Ruby編寫


Jekyll不使用資料庫 ，使用者通過編寫Markdown、Textile或Liquid檔案， 生成一個完整的靜態網站，並且可以由Apache HTTP Server ， Nginx或其他Web伺服器提供服務。  Jekyll是GitHub Pages的引擎。

Jekyll非常靈活，可以與Bootstrap Semantic UI等前端框架結合使用。

Jekyll網站可以連接到基於雲的CMS軟體，例如CloudCannon，Forestry， Netlify或Siteleaf，使編輯者無需知道如何編程即可修改網站內容。



安装Ruby, Bundler.

Bundler 能够跟踪并安装所需的特定版本的 gem，以此来为 Ruby 项目提供一致的运行环境。

Bundler 是 Ruby 依赖管理的一根救命稻草，它可以保证你所要依赖的 gem 如你所愿地出现 在开发、测试和生产环境中。 利用 Bundler 启动项目简单到只用一条命令


我自己使用的環境是win10 

安裝選擇這個包:
Ruby+Devkit 3.2.2-1 (x64) 






jekyll-theme-chirpy
Public

應該是比較傳統的方式


chirpy-starter
快速方式

主題使用:
https://github.com/cotes2020/chirpy-starter


1. 点击use the starter template 后新建一个<user_name>.github.io的仓库;

2. clone下来;

3. 修改_config.yml

4. 在项目根目录下执行：bundle；


5. 在push 之前:
先看:https://github.com/cotes2020/jekyll-theme-chirpy/issues/502#issuecomment-1114183037
```
If anyone is encountering a Permissions issue for the Actions:

Go to Settings > Actions > General, then scroll all the way down to "Workflow Permissions" and make sure that "Read and write permissions" is selected.

```



6. push到远程：

```
git add .
git commit -m 'first commit'
git push origin main   # main是当前分支名称
```
push后会在github的action看到进程，action进程成功完成后会在github分支中看到gh-pages分支；

进入 https://<user_name>.github.io，此时主页已初始配置完成了。






ZMediumToMarkdown 

hackmd 

obsidian
https://ithelp.ithome.com.tw/articles/10280373























