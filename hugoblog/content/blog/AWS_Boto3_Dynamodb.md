+++
showonlyimage = false
draft = false
date = "2023-05-31T29:25:22+15:30"
title = "Python Boto3 Dynamodb刪除/列出"
weight = 1
showmoretile = false
tags = "AWS,Python,Boto3,Dynamodb"
+++

[Boto3 Dynamodb Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html#dynamodb)  

#### 0. Config相關引入:  
```python
import boto3
from botocore.exceptions import ClientError #- For try catch 監聽Boto3各種錯誤資訊
db_client = boto3.client('dynamodb', region_name="ap-northeast-1")
```  
* * *    


#### 列出Region全部Dynamodb Tables: 
```python
def ListDbNames(): 
    ll = []
    try:
        LastEvaluatedTableName = None
        while True:
            arg = { 'Limit':100 }#- Base on document, once max items can query is 100
            if LastEvaluatedTableName: #- next page token
                arg['ExclusiveStartTableName'] = LastEvaluatedTableName
            rr = db_client.list_tables(**arg)
            LastEvaluatedTableName = rr['LastEvaluatedTableName'] if 'LastEvaluatedTableName' in rr else None
            ll+=rr['TableNames']
            if not LastEvaluatedTableName: #- no next page token > Break
                break
        return ll
    except ClientError as e:
        print("Error Create apiGateway deploy: {}".format(e))
    return ll
```  
* * *  

#### 刪除(特定/全部)Dynamodb Tables:  
> **ListOfTables = List string / List of Dynamodb Tables name**  
> **eg: [ "db_users_test", "db_users" ]**  
> **if param:ListOfTables == None, ListOfTables will be all of Dynamodb Tables**  
> **ListOfTables 可以帶想要刪除的Dynamodb Tables**  
> **如果ListOfTables沒帶, 將會自動scan region 全部Dynamodb Tables作刪除**  

```python
def DeleteDbs(ListOfTables=None):
    if not ListOfTables:
        ListOfTables = ListDbNames()
    for _tableName in ListOfTables:
        try:
            db_client2.delete_table(
                TableName=_tableName
            )
            print(f"Delete Table:{_tableName} === OK")
        except ClientError as e:
            print(f"Delete Table:{_tableName} Error", e)
        time.sleep(1)
    print(">>> Delete Tables Done")
```  
* * *   