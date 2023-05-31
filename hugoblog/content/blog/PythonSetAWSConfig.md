+++
showonlyimage = false
draft = false
date = "2023-05-05T28:25:22+05:30"
title = "Python設定AWS金鑰"
weight = 1
showmoretile = false
tags = "AWS,Python"
+++

#### cmd指令開啟config參數位置
```bash
nano ~/.aws/config
```  
* * *  

#### 設定參數如以下
```bash
region = ap-northeast-1
output = json
aws_access_key_id = MY_AWS_ACCESS_ID
aws_secret_access_key = MY_AWS_SECRET_KEY
```