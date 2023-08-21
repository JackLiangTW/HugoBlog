+++
showonlyimage = false
draft = false
date = "2023-08-20T29:25:22+05:30"
title = "TypeScript:Extends vs implements"
weight = 1
showmoretile = false
tags = "TypeScript"
+++

#### Typescript中的extends, implements使用時機與範例:  
#### implements:  
###### 實現介面(定義類別結構)  
`只有class可以使用, 後面可以接type, interface, class`
```ts
type cccT = {
    name:string
}
interface cccI{
    age:number
}
class cccC{
    info:string
}

class cccAll implements cccT, cccI, cccC {
    name = "Jack";
    age = 11;
    info = "Hi"
}
const use = new cccAll();
console.log(use); //- { name: 'Jack', age: 11, info: 'Hi' }
```  
#### extends:  
###### (類)繼承, (實現介面)擴展  
介面擴展:`只有interface, class能使用, type 擴展只能用 &,|`  
class(類)繼承:`eg:classA extends classB{}`  
```ts
type cccT = {
    name:string
}
interface cccI{
    age:number
}
class cccC{
    info:string
}

interface newInter1 extends cccT, cccI, cccC{ //- interface可以擴展 with 三個介面
    more:string
}
const vvvv1: newInter1 = {
    name: "Jane",
    age: 33,
    info: "Good",
    more: "nothing"
};
console.log(vvvv1); //- { name: 'Jane', age: 33, info: 'Good', more: 'nothing' }

declare class newInter2 extends cccC{ //- class介面只能和class介面擴展
    more:string
}
const vvv2:newInter2 = {
    info:"Good",
    more:"Fine"
}
console.log(vvv2); //- { info: 'Good', more: 'Fine' }


//-----------------------  擴展End ---------------------

//-----------------------  class類 繼承
// 基礎類別（父類別）
class Animal {
    name: string;
    constructor(name: string) {
        this.name = name;
    }
    makeSound() {
        console.log("Generic animal sound");
    }
}

// 衍生類別（子類別），繼承自 Animal
class Dog extends Animal {
    constructor(name: string) {
        super(name); // 呼叫父類別的建構子
    }

    makeSound() {
        console.log("Woof woof!");
    }
}

const dog = new Dog("Buddy");
dog.makeSound(); // 輸出: Woof woof!
console.log(dog.name); // 輸出: Buddy

```    