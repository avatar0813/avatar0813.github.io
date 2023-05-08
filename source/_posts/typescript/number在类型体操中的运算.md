---
title: number 在类型体操中的运算
date: 2023-04-30 19:53:00
tags: [TypeScript]
index_img: /img/TypeScript.png
---

> 在TS类型体操运算中有时候涉及到number类型的操作比较麻烦
即不能直接进行字符串操作以及加减乘除等数字运算
---


### 1、转成字符串
---

假如现在有个题目 [trunc](https://github.com/type-challenges/type-challenges/tree/main/questions/05140-medium-trunc) 要求实现跟Math.trunc() 函数一样的功能，即取出数字整数部分，TS类型体操中应该怎么处理呢？

- 了解将number类型转成string类型， 通过`${}` 字符串模板转换

```ts
// 将number类型转成string
type NumberToString<T extends number> = `${T}`

type test = NumberToString<123> // "123"
```

- 通过TS推算`infer` 就可以进行拆分提取了
```ts
// 类型函数Rrunc
type Trunc<T extends string | number> = 
`${T}` extends `${infer L}.${infer Rest}` 
? L extends '' 
  ? '0'
  : L
: `${T}`

// 测试用例
import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<Trunc<0.1>, '0'>>,
  Expect<Equal<Trunc<0.2>, '0'>>,
  Expect<Equal<Trunc<1.234>, '1'>>,
  Expect<Equal<Trunc<12.345>, '12'>>,
  Expect<Equal<Trunc<-5.1>, '-5'>>,
  Expect<Equal<Trunc<'.3'>, '0'>>,
  Expect<Equal<Trunc<'1.234'>, '1'>>,
  Expect<Equal<Trunc<'-10.234'>, '-10'>>,
  Expect<Equal<Trunc<10>, '10'>>,
]
```

### 2、加法运算

```ts
type BuildArray<Length extends number, Ele  = unknown, Arr extends unknown[] = []> =
    Arr['length'] extends Length
        ? Arr
        : BuildArray<Length, Ele, [...Arr, Ele]>

type Add<Num1 extends number, Num2 extends number> = 
    [...BuildArray<Num1>, ...BuildArray<Num2>]['length']

type Add2<Num1 extends 1000, Num2 extends 1> = 
    [...BuildArray<Num1>, ...BuildArray<Num2>]['length']

type AddResult1 = Add<999, 1> // 1000
type AddResult2 = Add<1000, 1> // Type instantiation is excessively deep and possibly infinite: 类型深度可能是无限的 
```

> ts在递归超过1000次的时候会报 深度可能是无限的错误