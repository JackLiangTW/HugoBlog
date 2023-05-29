+++
showonlyimage = false
draft = false
date = "2016-11-05T18:25:22+05:30"
title = "Hugo githubPage架設筆記"
weight = 1
+++

Github新增2個專案:
(A)Source code repository / Hugo source code存放 (HugoBlog)
(B)Production repository / 展示GithubPage (jackliangtw.github.io)
教學來源:https://youtu.be/LIFvgrRxdt4


安裝Hugo:
win10安裝流程:
C槽建立 /hugo/bin 資料夾
https://github.com/gohugoio/hugo/releases  (Google 搜尋 hugo release)
下載最新版(amd64.zip) [eg: hugo_extended_0.111.3_windows-amd64.zip]
解壓縮到 C/hugo/bin/ 當中
環境變數Path設定 C:\Hugo\bin
cmd檢查hugo指令是否成功: hugo version
參考:

Git clone(A),(B)到local


====(只有最初建立時候需要)== START ==
cd 到(B)下:
(建立新分支main)git checkout -b main
(建立readme檔案)touch README.md
git add .
git commit -m "add README.md"
git push origin main

====(只有最初建立時候需要)== START ==
cd到(A)路徑中, 建立hugo專案
hugo new sit myHugoBlog

(cd到hugo themes中)
cd myHugoBlog/themes

選擇想要的hugo theme主題, 複製使用
https://themes.gohugo.io/

eg:
git clone https://github.com/kishaningithub/hugo-creative-portfolio-theme.git
====(只有最初建立時候需要)== END ==


cmd目錄在local(A)當中:(把/public源設定到(B)當中, 指令前/public資料夾必須不存在)
git submodule add -b main https://github.com/JackLiangTW/jackliangtw.github.io.git public

(查看/public當中的源是否已更改到(B))
git remote -v


在(A) /public當中:
git add .
git commit -m "Init"
git push origin main