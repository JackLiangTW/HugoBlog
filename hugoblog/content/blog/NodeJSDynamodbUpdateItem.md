+++
showonlyimage = false
draft = false
date = "2023-05-05T28:25:22+05:30"
title = "Node.js/AWS Dynamodb UpdateItem"
weight = 1
showmoretile = false
tags = "AWS,Nodejs,Dynamodb"
+++

[來源](https://forums.aws.amazon.com/thread.jspa?threadID=291877)

######  Update Item 不能 Batch update(批次),一次只能Update一筆(一個id)
```node.js
const AWS = require("aws-sdk");
AWS.config.update({ 
    region: "ap-northeast-1",
    accessKeyId: "AWS--accessKeyId",
    secretAccessKey: "AWS--secretAccessKey"
});
const docClient = new AWS.DynamoDB.DocumentClient();

const param = {
    TableName: "UserBackPack",
    Key: { uid: abc123456 } ,
    UpdateExpression: "set #key1 = :val1",        
    ExpressionAttributeNames: {
        "#key1": "message"
    },
    ExpressionAttributeValues: {
        ":val1": "hello2",
    }  
};
await docClient.update(param).promise();
```  
* * *  

######  參數說明:
> **1. TableName : Dynamodb資料表Table**  
> **2. Key : 該Table partition key(Id) 或 Secondary Id**  
> **3. UpdateExpression : 更新參數邏輯, 用下方(4),(5)參數做更新條件**  
> **4. ExpressionAttributeNames : Table Item參數代寫**  
> **5. ExpressionAttributeValues : 更新值參數代寫**