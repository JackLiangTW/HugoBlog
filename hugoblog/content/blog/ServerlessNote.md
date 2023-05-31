+++
showonlyimage = false
draft = false
date = "2023-05-05T29:25:22+05:30"
title = "AWS Serverless筆記"
weight = 1
showmoretile = false
tags = "AWS,Nodejs,Lambda"
+++


[參考網址1-Lambda Express setup](https://www.youtube.com/watch?v=lm7fn72eA8c)  
[參考網址2-Lambda file upload](https://www.youtube.com/watch?v=GBn5zi_Hhqk)  

#### 1. 安裝serverless  
```bash
npm i -g serverless
```  

###### 設定AWS權限:  
[參考教學](https://www.serverless.com/framework/docs/providers/aws/guide/credentials/)
**serverless IAM登入:**
```bash
serverless config credentials --provider aws --key <MyIAM-Key> --secret <MyIAM-SecretKey>
```  

**aws cli IAM登入:**
```bash
aws configure
依造步驟輸入...
```  
* * *  


#### 2. Serverless 建置 node.js模板:  
[參考教學](https://www.serverless.com/framework/docs/providers/aws/cli-reference/create)  
```bash
serverless create --template aws-nodejs --path myNewProjectName
```  
* * *  


#### 3. node.js專案設定(install裝需要package):  
```bash
cd myNewProjectName
npm init
npm install 
npm install some-need-lib
```  
* * *  

#### 4. 部屬到Lambda(cmd在.yml同一層):  
```bash
(直接部屬)
sls deploy

(部屬到特定region中)
sls deploy --region ap-northeast-1
```  
* * *  


####  yml參數說明:
```bash
service: myservice
# app and org for use with dashboard.serverless.com
app: myservice
org: k8865432110

# You can pin your service to only deploy with a specific Serverless version
# Check out our docs for more details
frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs12.x # lambda使用環境
  region: us-east-1 # aws部屬地區
  stage: dev
  environment: # 環境變數
      tableName: ${self:custom.tableName}
      bucketName: ${self:custom.bucketName}
  iamRoleStatements: # lambda裝置權限
      - Effect: Allow
        Action:
            - dynamodb:*
            - s3:*
            - ses:*
        Resource: '*'

package:
    individually: true

custom: # 自訂參數
    tableName: player-points-table
    bucketName: myserverlessprojectuploadbucket-12312356 #s3 bucket Name要先到aws建立好

functions: # lambda 函數 一次部屬產生多個(Lambdas)
  hello:
    handler: lambdas/endpoints/handler.hello # code來源(相對目錄)
    events: # api-gateway 設置
        - http:
              path: hello
              method: GET
              cors: true
  imageUpload:
        handler: lambdas/endpoints/uploadIcon.handler
        events:
            - http:
                  path: image-upload
                  method: POST
                  cors: true


```  
* * *  
