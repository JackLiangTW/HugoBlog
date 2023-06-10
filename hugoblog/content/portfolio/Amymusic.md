+++
date = "2023-05-25T14:33:01+05:30"
title = "AmyMusic"
draft = false
image = "img/portfolio/amymusic_1.jpg"
showonlyimage = false
weight = 1
+++

#### 技術架構  
###### Node.js / Express / Mongodb / Puppeteer爬蟲 / GCP App Engine / Socket.io
###### 使用 Node.js Express框架架設網站, 部屬到GCP App Engine
###### Puppeteer爬蟲蒐集Youtube清單到Mongodb中, 紀錄Youtube video Id, 影片title等
###### 使用Socket.io製作聊天室功能,可以聊天以及分享歌單
###### Mongodb Tables:
1. Users: 訪客登入, 可以記錄該人所創立的歌單
2. ShareLists: 播放清單, 
3. MessageLists: 聊天紀錄/歌單分享, 
4. DefaultCategory: 預設清單分類(搜尋欄下方一排按鈕:華語男歌手,....)
5. UserFavoriteList: 使用者最愛清單
* * * 
###### 前端架構說明:
###### JS / JQuery / Cookie / Socket.io / Youtube Embed API
* * *  

#### 右邊部分為住播放區, 左邊為聊天室,可以傳輸訊息和分享自己的歌單
![結果示意][1]
* * * 

#### 已分享歌單清單, 都為玩家分享的歌單
![結果示意][2]
* * * 

#### 支援搜尋功能, Node.js後端直接打request去Youtube網頁搜尋並解析Video Id/ Title
![結果示意][3]
![結果示意][4]  
* * * 

#### 右邊加入自訂歌單撥放器
![結果示意][5]
* * * 

#### 手機板演示 
![結果示意][6]  
![結果示意][7]
* * * 


[1]: /img/portfolio/amymusic_1.jpg
[2]: /img/portfolio/amymusic_2.jpg
[3]: /img/portfolio/amymusic_3.jpg
[4]: /img/portfolio/amymusic_4.jpg
[5]: /img/portfolio/amymusic_5.jpg
[6]: /img/portfolio/amymusic_6.jpg
[7]: /img/portfolio/amymusic_7.jpg