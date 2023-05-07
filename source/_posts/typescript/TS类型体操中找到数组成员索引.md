---
title: TS类型体操中找到数组成员索引
date: 2023-04-30 19:56:19
tags: [TypeScript, 数组]
index_img: /img/TypeScript.png
---
### 如何在类型体操中找到一个数组的索引

参考 `type-challenges` 中的中等题 [`indexOf`](https://github.com/type-challenges/type-challenges/tree/main/questions/05153-medium-indexof)

 
 #### 测试用例
 ```ts
 import type { Equal, Expect } from '@type-challenges/utils'

type cases = [
  Expect<Equal<IndexOf<[1, 2, 3], 2>, 1>>,
  Expect<Equal<IndexOf<[2, 6, 3, 8, 4, 1, 7, 3, 9], 3>, 2>>,
  Expect<Equal<IndexOf<[0, 0, 0], 2>, -1>>,
  Expect<Equal<IndexOf<[string, 1, number, 'a'], number>, 2>>,
  Expect<Equal<IndexOf<[string, 1, number, 'a', any], any>, 4>>,
  Expect<Equal<IndexOf<[string, 'a'], 'a'>, 1>>,
  Expect<Equal<IndexOf<[any, 1], 1>, 1>>,
]
 ```

要求在元组中根据成员找到下标

#### 分析问题&解决问题

- 问题的关键就在于怎么返回元组的索引
- 其次是判断全等

**如何返回元组的索引呢？**
在TS类型体操中，不能直接获取到索引，所以我们的目标是遍历元组，并且存储元组，通过存储的临时元组, 通过 `元组['length']`返回的元组长度，认为是他的索引。

**遍历判断是否相等**
 *    相等则返回临时变量的长度
 *    不相等则将剩下的成员递归，且临时变量长度+1

```ts
type IndexOf<T extends unknown[], U, Res extends any[] = []> =
T extends [infer L, ...infer Rest]
? U extends L
? Res['length']
: IndexOf<Rest, U, [L, ...Res]>
: -1
```

这里就会出现另一个问题 `1 extends number`、`number extends number` 的问题了， 这里两个都是`true`，那么不能判断全等，也就不能正确找到下标。

先借助 `type-challenges` 函数中的 `Equal` 方法判断全等
```ts
import type { Equal } from '@type-challenges/utils'

export type IndexOf<T extends unknown[], U, Res extends any[] = []> =
T extends [infer L, ...infer Rest]
? Equal<U, L> extends true
  ? Res['length']
  : IndexOf<Rest, U, [L, ...Res]>
: -1
```
