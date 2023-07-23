
---
title: TS定义类型声明
date: 2023-05-14 17:17:10
tags: [TypeScript]
index_img: /img/TypeScript.png
categories: [学习]
---
## 初始化一个ts项目
全局安装`typescript`, `tsc` 是typescript的脚手架工具， 安装了`typescript` 就会携带 `tsc`，

初始化ts配置文件 `tsconfig.json`
```bash
tsc --init
```

## 自定义类型来源

### namespace 命名空间

命名空间会在全局上挂载一个对象
如：
```ts
// index.ts
const result = MyLib.getResult()

// index.d.ts
declare namespace MyLib {
  function getResult(): string
}
```

### module 定义类型模块
在类似`@types/node` 这类很多的`@types/xx`npm类型包中的类型声明，使用`module`模式来全局定义的。

`@types` 包是由 [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) 项目统一管理的
```ts
// index.ts
function fnTest(fn: moduleFn) {
  fn.getString('1')
}

// index.d.ts
declare module 'ts/test' {
  type moduleFn = {
    getString(str: string): string 
  }
  type moduleVariable = 'variable'
}
```
### 全局变量和函数
通过 `declare` 声明变量和函数
```ts
// index.ts
console.log(getFn('str'))
console.log(foo)

// index.d.ts
declare function getFn(str: string): string
declare var foo: string
```
### 描述函数重载
申明语句中只能定义类型，不要在声明语句中定义具体的实现
```ts
// index.ts

let widget = getWidget(45)
let widgets = getWidget('str')

// index.d.ts
declare function getWidget(arg: string): string
declare function getWidget(arg: number): string[]

```

### 声明class
```ts
// index.ts
class SubGreet extends Greet {
  constructor(str: string) {
    super(str)
  }
}

// index.d.ts
declare class Greet {
  constructor(s: string);

  greeting: string;
  showGreeting(): void;

}
```

## 类型的作用域
默认 编写声明文件 `xx.d.ts`， dts中的类型声明默认是全局的;
如果模块内部有`import` `export`  则需要通过`import type`单独进行类型引用
```ts
// index.ts
import type { localVariable } from './types/local-module'
const localVar: localVariable = 'str'

// local-module.d.ts
export type localVariable = string
```

如果一个全局的dts中依赖了外部的类型,这个时候会去`import` 外部的类型声明，但是这时候就会将全局的dts变成本地化，即不能全局使用声明的变量

```ts
// index.ts
// 报错
const globalVar: globalVariable = 'str' // Cannot find name 'globalVariable'

// global.d.ts
import * as fs from 'fs'
declare type globalVariable = string
```
这个时候 可以通过 编辑器指令`reference`引入类型声明
```ts
// index.ts
const globalVar: globalVariable = 'str'

// global.d.ts
/// <reference lib="node" />
declare type globalVariable = string
```

## 总结
- 声明类型的三种方式
  - `namespace`: 声明一个全局对象，属性即为声明的类型
  - `module`： 跟 `namespace` 没有区别
  - `es module` es 标准的模块语法， 额外拓展了 `import type`

- `.d.ts` dts文件的类型声明默认是全局的，如果其中有 `import` `export` 将会变成局部的
- 全局的类型声明来源: `.d.ts`、 `@types` 以及 ts的内置声明`lib` 包括 dom 和 es