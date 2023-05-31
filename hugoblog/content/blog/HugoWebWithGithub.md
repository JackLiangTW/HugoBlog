+++
date = "2023-05-17T19:41:01+05:30"
title = "Hugo Web完整架設流程"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Hugo,Web"
+++

[Hugo流程參考來源](https://youtu.be/LIFvgrRxdt4)  
[Hugo win10安裝教學](https://youtu.be/N-QRjEJsBRU)

#### 1. **Github新增2個專案:**  
> **(A)** Source code repository / Hugo source code存放 (HugoBlog)  
> **(B)** Production repository / 展示GithubPage (jackliangtw.github.io)  
* * *  


#### 2. **安裝Hugo:**
> **win10安裝流程:**  
> **下載安裝Hugo:[Hugo release](https://github.com/gohugoio/hugo/releases)**  
> **C槽建立 /hugo/bin 資料夾**  
> **下載最新版(amd64.zip) [eg: hugo_extended_0.111.3_windows-amd64.zip]**  
> **解壓縮到 C/hugo/bin/ 當中**  
> **環境變數Path設定新增: C:\Hugo\bin**  
> **cmd檢查hugo指令是否成功: hugo version**  
* * *  

#### 3. **Git clone第一步驟(A),(B)到local端**  
###### cd 到(B)專案目錄下:  
```bash
(建立新分支main)git checkout -b main  
(建立readme檔案)touch README.md  
git add .  
git commit -m "add README.md"  
git push origin main  
```  
* * *  


###### cd到(A)路徑中, 建立hugo專案 && cd到hugo themes中(準備安裝主題)  
```bash
hugo new sit myHugoBlog  
cd myHugoBlog/themes  
```
* * *  

###### 選擇想要的[Hugo theme主題](https://themes.gohugo.io/)安裝
```bash
git clone https://github.com/kishaningithub/hugo-creative-portfolio-theme.git
```  
* * *  

#### 4. **把源專案(A)區中/public設為github(B)專案(submodule)**  
###### cd目錄在(A)當中:(指令前/public資料夾必須 **不存在** ,否則會衝突)  
```bash
git submodule add -b main https://github.com/JackLiangTW/jackliangtw.github.io.git public
```  

###### 查看/public當中的源是否已更改到(B)
```bash
git remote -v
```  
* * *  

#### 5. **Hugo build輸出專案**  
###### cd(A) 目錄當中:build輸出, hugo-creative-portfolio-theme = 套用的主題:  
```bash
hugo -t hugo-creative-portfolio-theme
```  
* * *  


#### 6. **Push到github(B)專案中**  
###### cd到(A) /public當中, 把hugo -t所產生的檔案push到github:主題:  
```bash
git add .
git commit -m "Init"
git push origin main
```  
* * *  

#### 7. **Push源專案(A)到github**  
###### cd到(A), 如果themes主題會有問題可以移除該themes/中的.git:  
```bash
cd themes
rm hugo-creative-portfolio-theme/.git
選擇全部皆是
```  
##### 此時git 會跳出fatal的錯誤 => 所以繼續以下步驟
> **1. 將/themes/hugo-creative-portfolio-theme 整個資料夾移到(A)專案外面**  
> **2. 先做一次commit**  
> **3. 把外面移動的/hugo-creative-portfolio-theme移回(A)的themes/中**  
> **4. 可以正常commit了**
* * *  

#### 其他**開發架設指令**  
###### 架設專案
```bash
hugo server
```