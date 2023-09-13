---
title: TS装饰器 Decorators
date: 2023-09-12 16:39:34
tags: [TypeScript]
index_img: /img/TypeScript.png
categories: [学习]
---

## 什么是装饰器

在TypeScript中，装饰器是一种特殊类型的声明，它可以被附加到**类声明、方法、访问器、属性或参数**上。装饰器使用 `@` 表示符，后面跟着装饰器的名字。

装饰器在编译时会被移除，它们主要用于在运行时对类和其成员进行一些额外的操作，例如进行一些自动的类型检查、注入依赖项等。

装饰器还是一个实验中的功能，要使用还需要在`tsconfig.json`中设置`experimentalDecorators: true`, 同时，如果想要使用元数据`metadata`, 需要设置`emitDecoratorMetadata: true`,还需要引入polyfill => `reflect-metadata`。

## 了解元数据 `metadata`

`metadata` 是`Reflect`的一个API，`Reflect.metadata`是ES7的一个提案，它提供了一种元编程的方式，可以**将元数据附加到类和类的方法上**。**元数据**是描述数据的数据，它可以让我们在代码中添加额外的信息，而无需在代码本身中修改任何东西。可以参考此文[reflect-metadata](https://rbuckton.github.io/reflect-metadata/)。

具体来说，`Reflect.metadata`可以用来实现以下两个功能：

- 描述类和类的方法，这些描述信息可以在其他地方使用；
- 在运行时访问这些描述信息，以便动态地决定如何处理类。

## 有哪些装饰器

### 装饰器工厂

首先了解**装饰器工厂**，装饰器工厂顾名思义也就是返回一个装饰器。

比如创建一个装饰器工厂，用来给class的原型对象添加color属性：

```ts
// 颜色工厂
export function ColorFactories(color: string) {
  return function (target: Record<string, any>) { // 返回一个类装饰器
    target.prototype.color = color
  }
}
// 定义一个与class 同名的 interface
interface RedFlower {
  name: string
  color: string
}

@ColorFactories('red')
class RedFlower {
  constructor(name: string) {
    this.name = name
  }
}

let rise = new RedFlower('rise')

console.log('rise:', rise, rise.color) // { name: 'rise' } red
```

在这段代码中，`ColorFactories`是装饰器工厂，用来创建不同的类装饰器。在这里有一个小技巧，可以定一个与`class`同名的`interface`，用来拓展ts对由`class`创建的实例的属性，比如不写`interface RedFlower`，直接访问`rise`原型上的属性`color`，会报错:`Property 'color' does not exist on type 'RedFlower'`。

### 类装饰器

类装饰器也就是修饰类的装饰器

只有一个参数: 构造函数作为唯一参数

```ts
export function classDecorator(target: Record<string, any>) {
  target.prototype.class = 'c1'
}


interface ConeStudent {
  name: string
  class: string
}

@classDecorator
class ConeStudent {
  constructor(name: string) {
    this.name = name
  }
}

const tom = new ConeStudent('tom')
console.log('tom:', tom, tom.class) // tom:  {name: 'tom'}, c1
```

可以直接在构造函数的原型对象上添加属性；

也可以直接修改构造函数:

```ts
// 重写构造函数
function reportableClassDecorator<T extends { new(...args: any[]): {} }>(target: T) {
  return class extends target {
    reportingURL = "http://www...";
    private name: string

    constructor(...args: any[]) {
      super(args)
      this.name = args[0]
    }
  }
}

@reportableClassDecorator
class Test {
  constructor(name: string) { }
}

const test = new Test('txt')
console.log('test:', test) // { reportingURL: 'http://www...', name: 'txt' }
```

在这个例子中，通过`reportableClassDecorator`装饰器重写类的构造函数，为其添加了一个属性`reportingURL`。

### 方法装饰器

方法装饰器在方法声明之前声明。装饰器应用于方法的属性描述符，可用于观察、修改或替换方法定义。方法装饰器不能在声明文件、重载或任何其他环境上下文中（例如在 declare 类中）使用。

方法装饰器有三个参数

- `target`: 静态成员的类的构造函数，或实例成员的类的原型
- `propertyKey`: 成员的姓名
- `descriptor`: 成员的属性描述符

```ts
function writable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.writable = value
  }
}

class MethodDecorator {
  private _name: string = ''
  private _nike: string = ''
  constructor() { }

  getNike() {
    return this._nike
  }

  @writable(false)
  getName() {
    return this._name
  }
}
const methodDecoratorItem = new MethodDecorator()
methodDecoratorItem.getNike = () => '' // OK
methodDecoratorItem.getName = () => '' // TypeError: Cannot assign to read only property 'getName' of object '#<MethodDecorator>'

```

在这个例子中，重写了属性`getName`方法的属性描述符`writable`，使得该属性不能够被重新赋值。
因此，当修改实例上的`getName`方法时报错。

### 属性装饰器

属性装饰器在属性声明之前声明。
属性装饰器有个参数：

- `target`: 静态成员的类的构造函数，或实例成员的类的原型
- `propertyKey`: 成员的姓名

```ts
import "reflect-metadata"

function propertyFormat(target: any, propertyKey: string) {
  // 获取到target，propertyKey
  console.log('propertyFormat', target, propertyKey)
}


// 设置唯一标识
const formatMetadataKey = Symbol.for("format")
function format(formatString: string) {
  // 设置元数据， 格式为： 唯一标识：格式化内容
  return Reflect.metadata(formatMetadataKey, formatString)
}
function getFormat(target: any, propertyKey: string) {
  // 根据之前设置的唯一标识，取出存储的格式化内容
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey)
}

class PropertyDecorator {
  @propertyFormat
  age = '12'

  @format('hello, %s')
  name = 'propertyDecorator'
  constructor() { }

  getName() {
    // 根据装饰器中设置的格式化内容进行格式化
    return getFormat(this, 'name').replace('%s', this.name)
  }
}

let propertyDecoratorItem = new PropertyDecorator()
console.log('propertyDecoratorItem.name:', propertyDecoratorItem.getName()) // hello, propertyDecorator
```

在上述例子中，`format`装饰器用来给属性添加格式化模版，之后在有使用该属性的地方直接调用格式化方法`getFormat`进行格式化。方式为:

- 1、使用`Reflect.metadata`创建唯一标识，并返回一个新的装饰器。
- 2、在格式化方法中使用`Reflect.getMetadata`根据唯一标识获取到格式化模板。

### 参数装饰器

参数装饰器在参数声明之前声明。参数装饰器应用于类构造函数或方法声明的函数。
参数装饰器有三个参数：

- `target`: 静态成员的类的构造函数，或实例成员的类的原型
- `propertyKey`: 成员的姓名
- `propertyIndex`: 函数参数列表中参数的顺序索引

```ts
import "reflect-metadata"
// 定义全局唯一标识符
const requiredMetadataKey = Symbol("required")

// 方法装饰器 validate
function validate(target: any, propertyName: string, descriptor: PropertyDescriptor) {
  // 引用原方法
  let method = descriptor.value!

  // 重写方法
  descriptor.value = function () {
    let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName)
    if (requiredParameters) {
      for (let parameterIndex of requiredParameters) {
        if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
          throw new Error("Missing required argument.")
        }
      }
    }
    return method.apply(this, arguments)
  }
}

// required 参数装饰器： 将标识属性添加到全局元数据中
function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || []
  existingRequiredParameters.push(parameterIndex)
  Reflect.defineMetadata(requiredMetadataKey, existingRequiredParameters, target, propertyKey)
}


class BugReport {
  type = "report";
  title: string

  constructor(t: string) {
    this.title = t
  }

  @validate
  print(@required verbose?: boolean) {
    if (verbose) {
      return `type: ${this.type}\ntitle: ${this.title}`
    } else {
      return this.title
    }
  }
}

const bug = new BugReport('bugReport')
bug.print() // throw new Error("Missing required argument.")
```

在上面例子中，添加属性装饰器`required`, 使用`Reflect.defineMetadata`将唯一标识、参数下标索引、对象以及属性相关联; 在方法装饰器中`validate` 中重写方法，主要是判断方法是参数是否符合参数要求：符和则调用原方法，不符合则抛出异常。

## 装饰器的原理

还是以第一个类装饰器为例:

```ts
// 类装饰器
// 构造函数作为唯一参数
export function classDecorator(target: Record<string, any>) {
  target.prototype.class = 'c1'
}

// 声明一个与class同名的interface 抽象定义class的类型
interface ConeStudent {
  name: string
  class: string
}
@classDecorator
class ConeStudent {
  constructor(name: string) {
    this.name = name
  }
}

const tom = new ConeStudent('tom')
console.log('tom:', tom, tom.class) // tom:  {name: 'tom'}, c1
```

执行编译: `tsc --target ES6 --experimentalDecorators`

整理一下编译之后的内容：

```js
"use strict"
var __decorate = (this && this.__decorate) ||
    function (decorators, target, key, desc) {
        // c:参数长度
        var c = arguments.length
        // r: 返回结果, 此时如果 c < 3, c === target（构造函数或者成员对象）,不然 c === desc 属性描述符
        r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc
        d

        if (typeof Reflect === "object" && typeof Reflect.decorate === "function") {
            // Reflect.decorate 也不是es6的标准
            r = Reflect.decorate(decorators, target, key, desc)
        } else {
            // 遍历装饰器
            for (var i = decorators.length - 1;i >= 0;i--) {
                if (d = decorators[i]) {
                    /**
                     * 判断参数长度:
                     * < 3， 只有 target， 此时 r === target
                     * 执行 d(r) 也就是将构造函数传给装饰器 (类装饰器)
                     * > 3 则有三个参数 target，key，此时 r === desc 属性修饰符
                     * 执行 d(target, key, r)  (方法装饰器，参数装饰器)
                     * === 3 则 只有2个参数 target, key
                     * 执行 d(target, key) (属性装饰器)
                     */
                    r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r
                }
            }
        }
        return c > 3 && r && Object.defineProperty(target, key, r), r
        /**
         * 处理结果;
         * 如果 c > 3， 
         *  则 r 为 方法装饰器 | 参数装饰器 的返回结果，需要给原构造函数或者成员对象target的属性key添加属性描述符。
         *  如果 r 为 null ｜ undefined, 说明装饰器没有 return 内容。(表明装饰器可以返回属性描述符去修改原属性)
         * 否则 c <= 3,
         * 则 r 为 类装饰器 ｜ 参数装饰器 的返回结果。
         * 函数__decorate直接return
         */
    }

Object.defineProperty(exports, "__esModule", { value: true })
exports.classDecorator = void 0

// 类装饰器
// 构造函数作为唯一参数
function classDecorator(target) {
    target.prototype.class = 'c1'
}
exports.classDecorator = classDecorator
let ConeStudent = class ConeStudent {
    constructor(name) {
        this.name = name
    }
}
// 给class ConeStudent 添加装饰器
ConeStudent = __decorate([
    classDecorator
], ConeStudent)
const tom = new ConeStudent('tom')
console.log('tom:', tom, tom.class) // tom:  {name: 'tom'}, c1
```

原理也很直接，就是将装饰器中的方法执行一遍，只是判断一下装饰器的类型做了不同的处理。
方法装饰器和属性装饰器可以通过return 属性描述符来修改属性；
类装饰器和参数装饰器也就是执行了一遍。
