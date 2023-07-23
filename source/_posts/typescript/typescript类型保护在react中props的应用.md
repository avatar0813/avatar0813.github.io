---
title: typescript类型保护在react中props的应用
date: 2023-07-22 09:46:03
tags: [TypeScript]
index_img: /img/TypeScript.png
categories: [学习]
---

# typescript类型保护在react中props的应用

> 场景: 假如在react的父组件向子组件传递参数有两种类型，而两种参数类型有公共的也有不同的，如何不用可选参数去规范传递参数呢？

如： 现在`parent.tsx`组件中想要`child.tsx`子组件，传递男性、女性两种对象，他们有公共属性`name`,`gender`，不同的是他们有各自的私有属性：男性有`salary`，女性有`weight`。

## 可选参数的方式

### 实现方式

在`child`组件中声明`props`类型, 限制`gender`为：`male ｜ female`

```ts
// child.tsx
type Props = {
  name: string
  gender: 'male' | 'female'
  salary?: number // male 私有
  weight?: number // female 私有
}
```

现在想要在`child`中展示根据传入的数据分别展示男性和女性的数据

```ts
// parent.tsx
import * as React from 'react'
import Child from './child'

export default function Parent() {
  return (
    <div>
      <Child name="tom" gender="male" salary={1200} weight={50}></Child>
      <Child name="mary" gender="female" salary={1200} weight={50}></Child>
    </div>
  )
}

```

```ts
// child.tsx
export default function Child(props: Props) {
  function InfoItem() {
    if (props.gender === 'male') {
      return (
        <p>
          gender: {props.gender}, 工资: {props.salary}
        </p>
      )
    } else {
      return (
        <p>
          gender: {props.gender}, 体重: {props.weight}
        </p>
      )
    }
  }

  return (
    <div>
      <p>child.name: {props.name}</p>
      <div>
        <p>InfoItem:</p>
        <InfoItem />
      </div>
    </div>
  )
}
```

### 结果与问题

能够达到正常的效果，但是又好像有点不太友好，因为我想要当`gender`为`male`时只能传`salary`，不能传`weight`,同样，为`female`时则只能传`weight`。

目前可选参数并不能做到限制`parent`组件中传参和`child`渲染判断 `gender` 与 `salary` `weight`之间的关系。

[![pCbvZGT.png](https://s1.ax1x.com/2023/07/22/pCbvZGT.png)](https://imgse.com/i/pCbvZGT)

## 使用union类型做类型保护

### 可以使用联合类型的方式来做类型保护

```ts
type Props = {
  name: string
} & (maleProps | femaleProps)

type maleProps = {
  gender: 'male'
  salary: number
}

type femaleProps = {
  gender: 'female'
  weight: number
}
```

在联合类型`maleProps`、`femaleProps`中限制`gender`属性值和各自的私有属性.
这样当`props.gender`为`male`时，则命中`maleProps`，实际`props`就等同于:

```ts
type Props = {
  name: string
  gender: 'male'
  salary: number
}
```

当`props.gender`为`famale`时，则命中`famaleProps`，实际`props`就等同于:

```ts
type Props = {
  name: string
  gender: 'female'
  weight: number
}
```

### 实现的效果

此时`child`中判断就能够根据前置条件判断属性了 `parent`也会根据之前传入的参数做限制了。

[![pCbvWLj.png](https://s1.ax1x.com/2023/07/22/pCbvWLj.png)](https://imgse.com/i/pCbvWLj)

[![pCbxilD.png](https://s1.ax1x.com/2023/07/22/pCbxilD.png)](https://imgse.com/i/pCbxilD)

## 在response响应数据中的应用

在请求数据的时候也可以根据响应的状态来限制响应体中只有数据或错误信息.

```ts
/* eslint-disable @typescript-eslint/no-unused-vars */
type ResponseData<T> = SuccessRes<T> | ErrorRes

type SuccessRes<T> = { status: 'success', data: T, timestamp: Date }
type ErrorRes = { status: 'error', message: string, timestamp: Date }

const res1:ResponseData<number> = {
  status: 'success',
  data:100,
  timestamp: new Date()
}
const res2:ResponseData<number> = {
  status: 'error',
  message: 'api error',
  timestamp: new Date()
}
```

如果属性没有对应上`status`也会报错

[![pCbxW9K.png](https://s1.ax1x.com/2023/07/22/pCbxW9K.png)](https://imgse.com/i/pCbxW9K)

ps:

[https://stackblitz.com/edit/stackblitz-starters-hyceax?file=src%2FApp.tsx](https://stackblitz.com/edit/stackblitz-starters-hyceax?file=src%2FApp.tsx)

[https://www.youtube.com/watch?v=9i38FPugxB8](https://www.youtube.com/watch?v=9i38FPugxB8)