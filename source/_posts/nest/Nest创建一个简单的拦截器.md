---
title: Nest创建一个简单的拦截器
date: 2023-06-08 23:27:13
tags: [Nest]
index_img: /img/nest.svg
---

### 实现一个 `NestInterceptor` 类
具体代码如下👇：
```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from '@nestjs/common';
import { Observable, map } from 'rxjs';

interface Data<T> {
  data: T;
}

// 声明为可注射的
@Injectable()
export class ResponseInterceptor<T> implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Data<T>> {
    return next.handle().pipe(
      map((data) => {
        return {
          data,
          status: 0,
          message: 'success',
        };
      }),
    );
  }
}

```

### 1、实现 `NestInterceptor` 类中的方法 `intercept`
`intercept` 函数两个参数
- `context`: ExecutionContext 执行上下文  可以获取 getClass getHandler
- `CallHandler`: 只有一个handle方法，返回一个Observable 监听响应数据流
```ts
export interface NestInterceptor<T = any, R = any> {
    /**
     * Method to implement a custom interceptor.
     *
     * @param context an `ExecutionContext` object providing methods to access the  执行上下文
     * route handler and class about to be invoked.
     * @param next a reference to the `CallHandler`, which provides access to an  返回一个Observable
     * `Observable` representing the response stream from the route handler.
     */
    intercept(context: ExecutionContext, next: CallHandler<T>): Observable<R> | Promise<Observable<R>>;
}
```
### 2、使用 `rxjs` 中的`Observable` 进行响应式编程
[了解rxjs](https://rxjs.dev/)
什么是响应式编程呢，可以看看这个: [响应式编程（Reactive Programming）介绍](https://zhuanlan.zhihu.com/p/27678951)
大概就是一个异步的、流式的编程方式

### 3、添加全局拦截器
给拦截器类添加注解 `@Injectable()`

在服务入口文件中(main.ts)中，调用Nest服务实例方法`useGlobalInterceptors`添加全局拦截器

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ResponseInterceptor } from './interceptor/response.interceptor';
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 添加全局拦截
  app.useGlobalInterceptors(new ResponseInterceptor());
  await app.listen(3000);
}
bootstrap();
```

### 额外知识点： 返回值利用了协变原理
```ts
 interface Data<T>{
    data: T
 }

 type Func<T> = (arg: T) => Data<T>
 const func: Func<string> = (arg) => {
    return {
        data: arg,
        age: 12,
    }
 }
 // 能够正常运行
 // 返回值能够满足接口 Data 是因为 TS的协变原理 即：子类能够赋值给父类
 ```