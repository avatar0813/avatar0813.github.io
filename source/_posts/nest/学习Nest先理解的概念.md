---
title: 学习Nest先理解的概念
date: 2023-06-04 11:44:11
tags: [Nest]
index_img: /img/nest.svg
---

## 什么是IOC (inverse of control)

**IOC 也就是控制反转, 什么是控制反转？**

从主动创建依赖，到被动等待依赖注入就是控制反转。

**什么叫主动创建依赖？**

手动创建对象,选择依赖的过程；

**什么是被动等待依赖注入？**

申明控制器(controller)，控制器接受传入的依赖(service)，就是一个等待被动注入依赖的过程。

整个过程分别声明了controller以及service，并通过module将两者整合在一起。

## controller 和 service上的注解

在controller中的class上声明了`@Controller`，表示可以被注入，且只能被注入，顾名思义，控制器，只能接受注入，不能将控制器注入到其他服务中。

在service中的class上声明了`@Injectable`表示这个class 可以注入, 也可以被注入。

在module中的class上声明了`@Module`，其他`providers`也就是表示供应者

## 什么是AOP (Aspect Oriented Programming)
一个请求过来，会进过controller， Service，Repository等逻辑，当想要添加日志或权限校验错误处理等逻辑时，每个业务单独添加这种方式是具有侵入业务代码很难维护且不够健壮的， 而面向切面变成的意思就是在这些业务逻辑时，在另一个切面当度来实现，这种变成方式叫做面向切面编程

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f99087120e847eab901738bf8504d21~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)