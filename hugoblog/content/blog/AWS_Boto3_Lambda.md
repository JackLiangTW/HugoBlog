+++
showonlyimage = false
draft = false
date = "2023-05-31T29:25:22+11:30"
title = "Python Boto3 Lambda複製"
weight = 1
showmoretile = false
tags = "AWS,Python,Boto3,Lambda"
+++

#### 0. Config相關引入:  
```python
import boto3
import urllib3 #- [複製Lambda到Local需要]
import os      #- [複製Lambda到Local需要]
import io      #- [複製Lambda到Local需要]
import zipfile #- [複製Lambda到Local需要]
import shutil  #- [複製Lambda到Local需要]For Remove folder
from botocore.exceptions import ClientError #- For try catch 監聽Boto3各種錯誤資訊
lambda_client = boto3.client("lambda", region_name="ap-northeast-1")
```  
* * *   


#### 列出Region全部Lambda functions:  
[Boto3 Lambda Docs](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/lambda/client/list_functions.html)
```python
def ListAllFunctions(useSource = True):
    try:
        ll = [] #- retrun list bucket
        nextMarker = None
        while True: #- Traverse to keep query items until there is no next page token
            arg = { 'MaxItems':50 } #- Base on document, once max items can query is 50
            if nextMarker: #- next page token
                arg['Marker'] = nextMarker
            response = lambda_client.list_functions(**arg)
            ll+=[ itm['FunctionName'] for itm in response['Functions']]
            nextMarker = response['NextMarker'] if 'NextMarker' in response else None
            if not nextMarker: #- Break and leave traverse
                break
        
        return ll
    except:
        return []
```  
* * *   


#### 複製Lambda functions到本機:
> **CustomeCloneLambdas = List string / List of lambda function name**  
> **eg: [ "lambda_upload_img", "lambda_chat_msg" ]**  
> **if param:CustomeCloneLambdas == None, CustomeCloneLambdas will be all of region lambda functions**  
> **CustomeCloneLambdas 可以帶想要複製的lambda function names**  
> **如果CustomeCloneLambdas沒帶, 將會自動scan region 全部lambda functions作複製**  
```python
def CloneLambdaLocal(CustomeCloneLambdas = None):
    if not CustomeCloneLambdas:
        CustomeCloneLambdas = ListAllFunctions()
    print(f"Lambda複製總數:{len(CustomeCloneLambdas)}")

    for funtionName in CustomeCloneLambdas:
        try:
            print(f"Lambda:{funtionName} 複製到Local...")
            function_data = lambda_client.get_function(
                FunctionName=funtionName,
                Qualifier="$LATEST"
            )
            
            function_data.pop("ResponseMetadata")

            # Download function code from temporary URL
            code_url = function_data.pop("Code")["Location"]
            http = urllib3.PoolManager()
            response = http.request("GET", code_url)
            if not 200 <= response.status < 300:
                raise Exception(f"Failed to download function code: {response}")
            
            folder_path = f'./files/CloneLambdaPlace' #- 複製Lambda目標存放目錄
            if os.path.exists(folder_path): #- Remove exist old folder
                shutil.rmtree(folder_path)
            os.makedirs(folder_path) #- Create new folder

            zip_data = response.data
            # 解壓縮 ZIP 文件並提取其中的內容
            with zipfile.ZipFile(io.BytesIO(zip_data)) as zip_file:
                zip_file.extractall(folder_path)
            
        except ClientError as e:
            print(f"ERROR:{funtionName} 複製到Local <<<<<<< ")
```  
* * *  


#### 刪除(特定/全部)Lambda functions:  
> **CustomeCloneLambdas = List string / List of lambda function name**  
> **eg: [ "lambda_upload_img", "lambda_chat_msg" ]**  
> **if param:CustomeCloneLambdas == None, CustomeCloneLambdas will be all of region lambda functions**  
> **CustomeCloneLambdas 可以帶想要刪除的lambda function names**  
> **如果CustomeCloneLambdas沒帶, 將會自動scan region 全部lambda functions作刪除**  
```python
def DeleteLambdas(CustomeCloneLambdas = None):
    if not CustomeCloneLambdas:
        CustomeCloneLambdas = ListAllFunctions(False)
    for _functionName in CustomeCloneLambdas:
        try:
            lambda_client2.delete_function(
                FunctionName=_functionName
            )
            print(f"Delete Lambda:{_functionName} === OK")
        except ClientError as e:
            print(f"Delete Lambda:{_functionName} Error", e)
        time.sleep(1)
    print(">>> Delete Lambdas Done")
```  
* * *   