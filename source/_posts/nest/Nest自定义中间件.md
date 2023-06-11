---
title: Nest自定义中间件MiddleWare
date: 2023-06-11 15:23:36
tags: [Nest]
index_img: /img/nest.svg
---

## Nest自定义中间件的使用
### 实现一个自定义中间件类
```ts
import { Injectable, NestMiddleware } from '@nestjs/common';

@Injectable()
export class Logger implements NestMiddleware {
  use(req: any, res: any, next: () => void) {
    console.log('loggerMiddleWare-调用');
    next();
  }
}
```
自定义中间件实现了`NestMiddleware`类，只有一个方法`use`
```ts
/**
 * @see [Middleware](https://docs.nestjs.com/middleware)
 *
 * @publicApi
 */
export interface NestMiddleware<TRequest = any, TResponse = any> {
    use(req: TRequest, res: TResponse, next: (error?: Error | any) => void): any;
}
```

在中间件中能够获取到请求信息，能够通过`next`方法执行下一个中间件,
也能够提前响应请求
```ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
@Injectable()
export class Logger implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    res.send({ message: '中间件拦截结束响应' });
  }
}
```

### 自定义中间件的使用
在`module`中使用，`module`实现`NestModule`类
```ts
import {
  Module,
  NestModule,
  MiddlewareConsumer,
  RequestMethod,
} from '@nestjs/common';
import { Logger } from './middleWare/logger.middleware';

export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(Logger).forRoutes({ path: '*', method: RequestMethod.GET });
  }
}

```
- 实现的`NestModule`中的`configure`方法
- `configure` 方法接受一个参数 `consumer` (消费者)
- `configure` 通过`apply`方法接受一个或多个中间件，返回一个`MiddlewareConfigProxy` 中间件代理
- 代理对象上有`exclude`： 排除的路由, `forRoutes`： 要匹配的路由 两个方法


**`forRoutes`的匹配方式**
```ts
// 以下三种方式均可传多个参数匹配多个
// 方式一: route匹配
consumer.apply(Logger).forRoutes('*');

// 方式二：routeInfo匹配
consumer.apply(Logger).forRoutes({ path: '*', method: RequestMethod.GET });

// 方式三：控制器匹配
consumer.apply(Logger).forRoutes(appController);
```


### 函数式中间件
```ts
// loggerFn.middleware.ts
import { Request, Response, NextFunction } from 'express';

export function LoggerFn(req: Request, res: Response, next: NextFunction) {
  console.log('函数式中间件拦截');
  next();
}
```
结构跟自定义类中间件一样, `nest`全局中间件只能使用函数式的中间件.
