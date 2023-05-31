+++
showonlyimage = false
draft = false
date = "2023-05-31T29:25:22+14:30"
title = "Python Boto3 ApiGateway刪除/列出"
weight = 1
showmoretile = false
tags = "AWS,Python,Boto3,ApiGateway"
+++

[Boto3 ApiGateway Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/apigateway.html#apigateway)  

#### 0. Config相關引入:  
```python
import boto3
from botocore.exceptions import ClientError #- For try catch 監聽Boto3各種錯誤資訊
client = boto3.client('apigateway', region_name="ap-northeast-1")
```  
* * *   


#### 列出Region全部ApiGateway API: 
```python
def ListAllOfApis(costumeClient): 
    ll = [] #- retrun list bucket
    try:
        LastPosition = None
        while True: #- Traverse to keep query items until there is no next page token
            arg = { 'limit':500 } #- Base on document, once max items can query is 500
            if LastPosition: #- next page token
                arg['position'] = LastPosition
            rr = costumeClient.get_rest_apis(**arg)
            LastPosition = rr['position'] if 'position' in rr else None
            ll+=rr['items']
            if not LastPosition: #- no next page token > Break
                break
        return ll
    except ClientError as e:
        print("Error Create apiGateway deploy: {}".format(e))
    return ll
```  
* * *  

#### 刪除(特定/全部)ApiGateway API:  
> **CustomeClonApis = List string / List of ApiGateway API name**  
> **eg: [ "Api_dev", "Api_for_register" ]**  
> **if param:CustomeClonApis == None, CustomeClonApis will be all of ApiGateway APIs**  
> **CustomeClonApis 可以帶想要刪除的ApiGateway APIs**  
> **如果CustomeClonApis沒帶, 將會自動scan region 全部ApiGateway APIs作刪除**  
**中間需要time.sleep, 因為AWS處理刪除ApiGateway需要緩衝, 不能一次刪除多個**  
```python
def DeleteApiGateways(CustomeClonApis = None):
    TotalAPIs = ListAllOfApis(client2)
    if not CustomeClonApis:
        CustomeClonApis = TotalAPIs
    else: #- Filter By CustomeClonApis
        CustomeClonApis = list(filter(lambda o: o['name'] in CustomeClonApis, TotalAPIs))

    print(f"刪除多個API需要一些時間-AWS作業時間不一定....")
    print(f"欲刪除API數量:{len(CustomeClonApis)}")
    RunningRetry = 0 #- 遞迴山出同一筆次數
    LastNbs = len(CustomeClonApis) #- 待刪除總比數
    while CustomeClonApis: #- 持續Traverse刪除動作, 值到目標APIs都刪除完成
        _obj = CustomeClonApis[0]
        if RunningRetry > 10:
            print(f"!!!嘗試刪除API失敗太多次  >  已中斷   >  剩餘未刪除:")
            print([ oo['name'] for oo in CustomeClonApis])
            break
        if RunningRetry > 0:
            print(f"=====持續刪除API:{_obj['name']}中...({RunningRetry})")
        try:
            client2.delete_rest_api(
                restApiId=_obj['id']
            )
            LastNbs += -1
            RunningRetry = 0
            CustomeClonApis.pop(0) #- 刪除成功:移出列表
            print(f">>刪除API:{_obj['name']}成功 -- 剩餘{LastNbs}筆")
        except ClientError as e:
            RunningRetry += 1

        if CustomeClonApis:
            time.sleep(5) #- 每次Loop刪除時候間隔5秒

    print(">>> Delete ApiGateway APIs Done")
```  
* * *   