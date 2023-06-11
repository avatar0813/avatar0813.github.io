---
title: Nest自定义一个异常拦截器
date: 2023-06-11 12:14:18
tags: [Nest]
index_img: /img/nest.svg
---
## 自定义一个的异常拦截器需要实现 `ExceptionFilter` 类

先看全部代码 👇：
```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
} from '@nestjs/common';
import { Request, Response } from 'express';

// 捕获 httpException
@Catch(HttpException)
export class HttpFilter implements ExceptionFilter {
  /**
   *
   * @param exception Http请求错误
   * @param host ： ArgumentsHost 返回执行结果数组
   */
  catch(exception: HttpException, host: ArgumentsHost) {
    // 选择http上下文
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();
    // 获取错误状态
    const status = exception.getStatus();

    // express 拦截响应请求重新发送
    response.status(status).json({
      success: false,
      time: new Date(),
      data: exception.message,
      status,
      path: request.path,
      message: '请求失败辣',
    });
  }
}

```

### `@Catch()` 装饰器指明需要捕获的异常
这里指明捕获 `HttpException` 即只捕获http异常。

### 实现`ExceptionFilter`类中的`catch`方法
`catch`方法有两个参数：
- `exception`： 绑定的异常类型
- `host`:  根据执行上下文获取处理程序参数`ArgumentsHost` 


### `host.switchToHttp()` 选择执行上下文
在执行上下文中，可以获取到请求与响应。不过要方便后续的使用还是建议添加类型断言
从express中取出类型
```ts
import { Request, Response } from 'express';
...
// 选择http上下文
    const ctx = host.switchToHttp();
    const request = ctx.getRequest<Request>();
    const response = ctx.getResponse<Response>();

```

### 当前请求状态
当前的请求状态就是从第一个参数`exception`异常中去获取。

### 最后需要将拦截的请求重新发送给客户端
封装自定义的拦截响晴信息
```ts
 // express 拦截响应请求重新发送
    response.status(status).json({
      success: false,
      time: new Date(),
      data: exception.message,
      status,
      path: request.path,
      message: '请求失败辣',
    });
```
## 添加拦截

### 添加全局拦截
使用 `useGlobalFilters` 添加全局拦截
```ts
// main.ts

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalFilters(new HttpFilter());
  await app.listen(3000);
}

```

### 指定拦截
使用`@UseFilters`指定函数拦截或类拦截
```ts
// app.controller.ts
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get('/error')
    @UseFilters(HttpFilter2)
    getError(): string {
      return this.appService.getError();
    }
}
```