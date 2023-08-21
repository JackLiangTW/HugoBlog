+++
showonlyimage = false
draft = false
date = "2023-08-20T29:25:22+05:30"
title = "TypeScript:型別註記模式"
weight = 1
showmoretile = false
tags = "TypeScript"
+++

#### type:
###### 型別(結構)  
`主要用來定義靜態格式, 會使用交叉型別（Intersection）和聯合型別（Union）來擴充`
```ts
type Person = {
    name: string;
    age: number;
};

type Student = {
    name: string;
    age: number;
    grade: string;
};

// StudentPerson要符合(Person, Student)兩型別所有屬性
type StudentPerson = Person & Student;

// 至少要完全符合(Person, Student)其中一種型別, 其他部分符合也可接受
type StudentPerson2 = Person | Student;
```  
#### interface:  
###### 介面(類別結構)  
`主要用來定義物件結構`  
```ts
type TestType = {
    info: string;
}

interface Person {
    name: string;
    age: number;
}

//- 使用extends擴充介面
interface Student extends Person {
    grade: string;
}

//- interface也可以擴充type
interface Student2 extends TestType {
    grade: string;
}

const student: Student = {
    name: "Bob",
    age: 22,
    grade: "B"
};

const student2: Student2 = {
    grade: "B",
    info: "Come from Taiwan"
};
```  
#### class:  
###### 類, 類介面  
`class可以當作 類, 或是介面使用`  
```ts

//- class當作 介面 (class前面可加上declare)
class Student{
    name:string;
    age:number;
}

const man:Student = {
    name:"Jack",
    age:12
}

//- 類
class ClassroomSys{
    private _students:string[] = [];
    constructor(public teacher:string){}

    insert(name:string):void{
        this._students.push(name);
    }

    showClassroom():void{
        console.log("老師:", this.teacher, "學生:", this._students);
    }
}
const myClassRoom:ClassroomSys = new ClassroomSys("Teacher Shu"); //- 建構class
myClassRoom.insert("Jack");
myClassRoom.insert("Paul");
myClassRoom.showClassroom(); //- 老師: Teacher Shu 學生: [ 'Jack', 'Paul' ]

```

* * *  