+++
showonlyimage = false
draft = false
date = "2023-06-01T29:25:22+05:30"
title = "演算法:四則運算"
weight = 1
showmoretile = false
tags = "演算法,JS"
+++

#### 題目說明:  
###### 對一四則運算字串, 在不使用套件情況下將其運算完畢  
###### eg: Input:"(3+4)*2", Output:14
* * * *  
#### 解題說明:  
###### 1. 先把String字串依據運算符號+左右括號切成陣列  
###### 2. while loop將所有'('的index位置加入到Idxs_Conclude中, 遇到')'實則開始用算到上一個"(", 運算完後將值寫到起始位(lastIdx)
###### 3. 把沒有括號的運酸都丟給Calculate_without_brackets做運算  
###### 4. Calculate_without_brackets:先做乘除(*/) 再做 加減(+-) 運算, 乘除完後直接將值寫到起始位(i-1)  
###### 5. 步驟4後再做一次Calculate_without_brackets, 後還須將 +-*/ 剩餘完成


####  Full Code:
```js
function Calculate_Invoke(signals){ //- signals = string, eg:"3+4*2"

    // 使用正則表達式將表達式拆分成元素 eg: "3+4*2" => ["3", "+", "4", "*", "2"]
    let elements = signals.split(/(\+|\-|\*|\/|\(|\))/).filter(e => e.trim() !== "");

    let i = 0;
    let Idxs_Conclude = []; //- Stacks For 左括號 "(" 各index位置
    while(i<elements.length){ 
        let ele = elements[i];
        if(ele == "("){ Idxs_Conclude.unshift(i); } //- 紀錄 "(" index位置
        i+=1;
        if(ele == ")"){ //- 遇到")"開始運算到上一個"("
            const lastIdx = Idxs_Conclude.shift(); //- 找上一個 左括號 "(" index位置
            const t1 = elements.splice(lastIdx+1, i-lastIdx-1); //- 切掉 "(" , ... , ")" 的運算值
            var newV = Calculate_without_brackets( t1.slice(0, t1.length-1) ); //- 傳入括號內的 運算
            elements[lastIdx] = newV; //- 將原 "("值改成運算完()內的值
            i=lastIdx+1; //- i位置回調成 "("下一位
        }
    }
    return Calculate_without_brackets(elements);
}

function Calculate_without_brackets(datas){ //- datas = 沒有括號的四則運算
    let i = 0;
    while(i<datas.length){ //- 先只做 */ 運算
        const ele = datas[i];
        if(ele == "/" || ele == "*"){

            //- 把datas[i-1] : 值設成運算完成值
            if(ele == "*")datas[i-1] = parseInt(datas[i-1]) * parseInt(datas[i+1]);
            else{ datas[i-1] = parseInt(datas[i-1]) / parseInt(datas[i+1]); }
            i = i-1; //- 運算位置調回i-1:重設點
            datas.splice(i+1, 2); //- 切掉後面運算完成元素 (+-*/) + (NaN)
        }
        else{i+=1;}
    };
    let ans = 0; //- 目前值
    let lastOper = "+"; //- 鎖藥使用運算符號(預設為+)
    for(let v of datas){ //- 做只剩 +- 運算
        if(v=="+" || v=="-"){
            lastOper = v;
        } else{ //- 用最後運算符號 與現在v 做運算
            if(lastOper == "+"){
                ans = ans + parseInt(v);
            } else{
                ans = ans - parseInt(v);
            }
        }
    }
    return ans;
}

var q = "((9-8)*(6+2))-(3 + 4) * 2 - 5";
console.log( Calculate_Invoke(q) ); // -11

var q2 = "(6 + 2) * 3 / 4";
console.log( Calculate_Invoke(q2) ); //- 6


var q3 = "6 + 9 * 3 - 7 + 2 * 4 / 2"; //- 30
console.log( Calculate_Invoke(q3) );

```  
* * *  