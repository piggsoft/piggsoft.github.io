---
title: SpringBoot添加url版本控制注解ApiVersion
date: 2024-01-31 14:17:33
keyword:
- spring boot
- 版本
- version
- url中的version
- 固定的path variable
tags:
  - Java
  - Spring Boot
  - ApiVersion
categories:
  - [Java, Spring Boot]
---

作为服务端研发，在设计一个接口的时候，为了保证后续业务的新增和修改，会有各种的扩展性设计，其中关于URL的通用扩展设计是在path中加入版本。
例如: `https://domain.com/api/{version}/path` 。这种设计方便后续对接口升级时，只需要简单的升级版本号就可以了。

在有多个版本号之后，我们如何在Spring Boot中优雅的根据版本号进行不同的逻辑处理呢，本篇文章会介绍我正在使用的一种比较优雅的方法在Spring Boot中来处理版本号。

<!--more-->

## 准备工作
