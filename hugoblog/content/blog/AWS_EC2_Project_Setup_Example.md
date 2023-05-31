+++
showonlyimage = false
draft = false
date = "2023-05-05T31:25:22+05:30"
title = "AWS EC2專案複製架設範例(Dokcer)"
weight = 1
showmoretile = false
tags = "AWS,EC2,Nodejs,Docker"
+++

#### 需要準備:
###### 1. 建立好EC2, 並取得.pem金鑰
###### 2. 開好EC2 Inbound Rule(非必須)
###### 3. 建立好Node.js測試專案 
* * *   

#### 0. Local專案目錄範例:
```bash
ls
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----       2022/5/11  下午 04:32        2114799 backend.zip  (要複製到EC2的專案)
-a----       2022/3/22  下午 03:04           1700 TestDemo.pem  (金鑰)
```  
* * *   

#### 1. SSH到EC2(終端機在第0步驟):  
```bash
ssh -i "TestDemo.pem" ec2-user@ec2-XX-XXX-XXX-XXX.ap-northeast-1.compute.amazonaws.com
```  
* * *   

#### 2. EC2建立Dokcer環境:  
```bash
# 更新環境
$ sudo yum update -y

# 下載Amazon Dokcer 安裝檔:
$ sudo amazon-linux-extras install docker

# 安裝Docker
$ sudo yum install docker

# 取得用戶名
$ echo $USER
ec2-user

# 將用户添加到 docker用戶組, ec2-user = $USER：
$ sudo usermod -aG docker ec2-user

# 以管理員權限啟動Docker
sudo service docker start

# (非必須)(有安全風險)始docker指令不用sudo & 允許所有用戶Docker操作。
sudo chmod 666 /var/run/docker.sock
```  
* * *   

#### 3. EC2建立專案資料夾:project:  
```bash
$ mkdir project
```  
* * *  


#### 4. Local端複製zip到EC2 /project中 (終端機再0. Local專案目錄):  
```bash
scp -i "./TestDemo.pem" backend.zip ec2-user@ec2-XX-XXX-XXX-XXX.ap-northeast-1.compute.amazonaws.com:~/project
```  
* * *  

#### 5. EC2啟動專案流程(終端機SSH到EC2中):  
```bash
# 解壓縮專案
$ cd project
$ unzip backend.zip

# 移除zip
$ rm -r backend.zip

$ cd backend

# Docker將該專案建置成Docker Image
docker build -t="testDemo:v1" . --no-cache

# Docker Image建置Container
docker run -p 80:80 -d testDemo:v1

# Docker查看Container ID
$ docker ps

# 設定Container 發生當機自動重啟
$ docker update --restart unless-stopped <containerId>

```  
* * *  
