+++
showonlyimage = false
draft = false
date = "2023-06-01T29:25:22+05:30"
title = "演算法:列出全部組合可能"
weight = 1
showmoretile = false
tags = "演算法,JS"
+++

#### 題目說明:  
###### 給定不重複正整數 m 種類型 m1 m2 m3 ...，  
###### 每種類型對應正整數數量為 n1 n2 n3 ...，  
###### 需要取出正整數 p 個任意類型，請列出所有!!取法!! 
###### eg: Apple:3、Banana:2、Citron:4，要取出三個，則取法有:
###### [ Apple-3, Apple-2/Banana-1, Apple-2/Citron-1, ... ]
* * * *  
#### 解題說明:  
###### 1. 宣告變數mem用來存取前面算到的排列組合情況  
###### eg: mem = { 1:[ /.目前拿1個的情況./], 2:[ /.目前拿2個的情況./], ... , p:[] }
###### 2. for loop跑種類  
###### 3. Inside(2.) for loop跑 max(該種類最大值, p-1) ~ 1  
###### 為防止計算到重覆結果, 先重最大值開始算, 因為mem[i+1].push會使從小開始算造成重複解  
###### 4. Inside(3.) for loop跑 mem 當中 (p-i) ~ 1,  
###### i和 (p-i : 達到目標) - (1 : 前面只拿一種組合) 組合合併變成新可能  
###### 5. Inside(4.) for loop跑 mem[ii]組合新可能並存到到mem[ii+i]  
###### 6. Outside(3.) for loop跑只拿  max(該種類最大值, p-1) ~ 1 的該類可能:放最後是避免重複計算  
* * * *  

####  Full Code:
```js
function run(d,p){
    let mem = {};
    for(let i=1; i<=p; i++){ //- (1.)Init mem from 1 ~ p
        mem[i] = [];
    }
    
    for(let _key in d){ //- (2.)
        const nb = d[_key];

        //- (3.)拿 min(該種最大數, 目標p-1上限) ~ 1 種可能 , 用 -- 讓大的先判斷疊加,用++小的加上去會造成重複解
        for(let i=Math.min(nb, p-1); i>0; i--){ 

            for (let ii = (p-i); ii > 0; ii--) { //- (4.)跑之前mem中的 (p-i) ~ 1

                for(let o of mem[ii]){ //- (5.) mem[ii] 都提取出來加上 _key*i種
                    mem[ii+i].push(o+`/${_key}*${i}`);        
                }
            }
        }
        for(let i=1; i<=Math.min(nb, p); i++){ //- (6.)加上只拿 1~nb 數量可能, 最後加:避免重複算
            mem[i].push(`${_key}*${i}`);
        }
    }
    return mem[p];
}

const d0 = {
    "apple":3,
    "banana":2,
    "citron":4
};
const d1 = {
    "apple":2,
    "banana":2,
    "citron":2,
    "do":2,
};
const p0 = 3;

console.log( run(d0, p0) );
/*
[
  'apple*3',
  'apple*1/banana*2',
  'apple*2/banana*1',
  'apple*1/citron*2',
  'banana*1/citron*2',        
  'apple*2/citron*1',
  'apple*1/banana*1/citron*1',
  'banana*2/citron*1',        
  'citron*3'
]
*/

console.log( run(d1, p0) );
/*
[
  'apple*1/banana*2',
  'apple*2/banana*1',
  'apple*1/citron*2',
  'banana*1/citron*2',        
  'apple*2/citron*1',
  'apple*1/banana*1/citron*1',
  'banana*2/citron*1',        
  'apple*1/do*2',
  'banana*1/do*2',
  'citron*1/do*2',
  'apple*2/do*1',
  'apple*1/banana*1/do*1',
  'banana*2/do*1',
  'apple*1/citron*1/do*1',
  'banana*1/citron*1/do*1',
  'citron*2/do*1'
]
*/

```  
* * *  