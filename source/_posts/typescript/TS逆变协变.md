---
title: TS逆变协变
date: 2023-05-09 20:45:01
tags: [TypeScript]
index_img: /img/TypeScript.png
---
## TS逆变协变

- 什么是父类型子类型
- 了解什么是逆变协变
- 为什么要有逆变协变

### 父类型 子类型
> 子类型就是比父类型更加具体
```ts

type parent = string | boolean | number
type child = string | boolean
type test1 = child extends parent ? true : false // true
type test2 = parent extends child ? true : false // false 


interface Person {
    name: string
    age: number
}

interface Sun {
    name: string
    age: number
    class: string
}
type test3 = Sun extends Person ? true : false // true
type test4 = Person extends Sun ? true : false // false
```
可以看出 child 是 parent 的子类型， 比parent 更加具体；

同样 Sun 是 Person 的子类型， 比Person多了个属性， 比Person 更加具体；

***

### 协变
> 什么是协变： 子类型可以赋值给父类型就叫做协变
```ts
let p1:Person = {
    name: 'jack',
    age: 56
}

let s1:Sun = {
    name: 'tom',
    age: 12,
    class: 'c1'
}


p1 = s1
s1 = p1 // roperty 'class' is missing in type 'Person' but required in type 'Sun'.
console.log(p1.class) // Property 'class' does not exist on type 'Person'
```

在这个例子中 Sun 类型比 Person 类型多了一个class 属性，Sun 中其他属性跟 Person 一样， 所以可认为 Sun 是 Person 的子类型;

并且 p1 是 Person 类型， s1 是 Sun 类型， s1 可以赋值给 p1, 但 p1 不能赋值给 s1;

**PS: 为什么父类型的不能赋值给子类型的?**

子类型中可能有拓展的属性，所以不能直接赋值；
同样，子类型赋值给父亲类型的变量，也只会保留父类型的属性

***
### 逆变
> 什么是逆变： 父类型可以赋值给子类型

第一个案例

```ts
let func1 = (str: string | number) => undefined
let func2 = (str: string | number | boolean) => undefined 

func1 = func2
func2 = func1 // error 
```
func1 中的参数 是 func2 中的参数的子类型，func2 能够赋值给func1， 子类型能够赋值给父类型


```ts
let splitStringByType_string = (str: string) => {
    return str.split('.')
}

let splitStringByType = (str: string | number | boolean) => {
    switch (typeof str) {
        case 'string':
            return str.split('.');
        case 'number': 
            return [str.toString()];
        case 'boolean': 
            return [str.toString()];
        default:
            return [];
    }
}

splitStringByType_string = splitStringByType
splitStringByType = splitStringByType_string // error  splitStringByType 可能有number  boolean的情况
```
splitStringByType 中的参数也是 splitStringByType_string 的父类型，能够这样赋值的逻辑就是
**参数为父类型的函数会处理更多条件，所以一定兼容着子类的条件场景**


***
### 总结
- 判断父子类型，通过结构判断， 更加具体的那个是子类型
- 逆变协变的赋值逻辑都是后者需要兼容前者才能赋值