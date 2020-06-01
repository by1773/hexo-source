---
title: Nest应用（一）
tags: nest
categories: nestjs
keywords:
  - nodejs
  - nest
top_img: 'http://kamilmysliwiec.com/public/nest-logo.png'
cover: >-
  https://d33wubrfki0l68.cloudfront.net/e937e774cbbe23635999615ad5d7732decad182a/26072/logo-small.ede75a6b.svg
abbrlink: 50015
date: 2020-04-03 12:00:00
---

# Nest 6.x 快速指南

## 1、简介

Nest 是一个用于构建高效，可扩展的 [Node.js](http://nodejs.cn/) 服务器端应用程序的框架。它使用渐进式 JavaScript，内置并完全支持 [TypeScript](https://www.tslang.cn/)，结合了 OOP（面向对象编程），FP（函数式编程）和 FRP（函数式响应编程）的元素。

## 2、特征

模块、控制器、组件

- 框架

Nest是一个渐进的Node.js框架，可以在TypeScript和JavaScript (ES6、ES7、ES8)之上构建高效、可伸缩的企业级服务器端应用程序。

- 哲学

Nest 提供了一个开箱即用的应用程序架构，允许开发人员和团队创建高度可测试，可扩展，松散耦合且易于维护的应用程序。

- 平台无关

技术上讲，Nest 可以在创建适配器后使用任何 Node HTTP 框架。 有两个支持开箱即用的 HTTP 平台：express 和 fastify。 您可以选择最适合您需求的产品。

- 理念

在设计上的很多灵感来自于 Angular，Angular 的很多模式又来自于 Java 中的 Spring 框架，依赖注入、面向切面编程等，所以我们也可以认为： Nest 是 Node.js 版的 Spring 框架。

## 3、NestJS和Egg.js **的一些简单对比** 

> 1. Egg.js 是和 Nest.js 都是为企业级框架和应用而生。 
> 2. Egg.js 和 Nest.js 都是非常优秀的 Nodejs 框架。Egg.js 基于 Koa，Nest.js 默认基于 
> 3. Express，nest 也可以基于其他框架 
> 4. Egg.j**s** 文档相比 **Nestjs** 优秀很多 
> 5. Express、Koa 是 Node.js 社区广泛使用的框架，简单且扩展性强，非常适合做个 
> 6. 人项目。但框架本身缺少约定，标准的 MVC 模型会有各种千奇百怪的写法。Egg 和 Nestjs 
> 7. 都是按照约定进行开发的。但是 Egg 相比 Nestjs 约定更标准。 
> 8. 面向对象方面 Nestjs 优于 Egg.js，Nestjs 基于 TypeScript，如果你会 angular 或者 java 
> 9. 学习 Nestjs 非常容易。 

- **Egg.js** **的特性：** 

  提供基于 Egg 定制上层框架的能力高度可扩展的插件机制 

  内置多进程管理 

  基于 Koa 开发，性能优异 

  框架稳定，测试覆盖率高 

  渐进式开发 

  

  **Nestjs** **的特性：** 

  依赖注入容器 

  模块化封装 

  可测试性 

  内置支持 TypeScript 

  可基于 Express 或者 fastify

## 3、创建项目

首先，您可以使用 Nest CLI 构建项目，也可以克隆启动项目（两者都会产生相同的结果）。

要使用 Nest CLI 构建项目，请运行以下命令。这将创建一个新的项目目录，并生成 Nest 核心文件和支持模块，为您的项目创建传统的基础结构。建议初学者使用Nest CLI 创建新项目。我们将在[第一步](https://www.bookstack.cn/read/nest-6/$6-firststeps?id=第一步)继续使用这种方法。

- ####   使用 CLI 安装

```shell
$ npm i -g @nestjs/cli$ 
$ nest new project-name
```

- #### **使用 Git 安装**

```shell
$ git clone https://github.com/nestjs/typescript-starter.git project
$ cd project
$ npm install
$ npm run start
```

## 4、常用命令

基于nest-cli脚手架工具我们可以快速的创建项目所需要的组件，该命令与angulr-cli极其相似，大致命令如下：

| 命令                                   | 说明               |
| -------------------------------------- | ------------------ |
| nest  g  service   "路径/服务名称"     | 创建一个服务       |
| nest  g  module   "路径/模块名称"      | 创建一个模块       |
| nest  g  pip"路径/管道名称"            | 创建一个管道       |
| nest  g  provider"路径/服务提供者名称" | 创建一个服务提供者 |
| nest  g  controller"路径/控制器名称"   | 创建一个控制器     |
| ···                                    | ···                |

## 5、工程解释

> 入口文件main.ts

``` typescript
import { NestFactory } from '@nestjs/core';
import * as rateLimit from 'express-rate-limit';
import * as compression from 'compression';
import * as helmet from 'helmet';
import { TransformInterceptor } from './interceptors/transform.interceptor';
import { HttpExceptionFilter } from './filters/http-exception.filter';
import { AppModule } from './app.module';
import { NestExpressApplication } from '@nestjs/platform-express'
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.enableCors();
  app.setGlobalPrefix('/api'); //设置全局请求前置
  app.use(
    rateLimit({
      windowMs: 60 * 1000, // 1 minutes
      max: 1000, // limit each IP to 1000 requests per windowMs
    })
  );
  app.use(compression()); // 启用 gzip 压缩
  app.use(helmet());
  app.useGlobalInterceptors(new TransformInterceptor()); // 正常情况下，响应值统一
  app.useGlobalFilters(new HttpExceptionFilter()); // 异常情况下，响应值统一
  app.useStaticAssets('public') //配置静态资源访问地址
  // 配置swagger-ui接口文档
  const options = new DocumentBuilder()   //实例化swagger 文档
    .setTitle('网站后台管理系统与前端页面接口文档') //设置文档标题
    .setDescription('网站后台管理系统与前端页面接口文档') //设置文档描述
    .setVersion('1.0.0') //设置文档版本
    // .addTag('Next与Nest建站') //设置文档标签
    .build(); //执行构建
  const document = SwaggerModule.createDocument(app, options);
  SwaggerModule.setup('api-docs', app, document);   //设置本地访问地址
  await app.listen(4000);  //设置服务监听端口
}

bootstrap();
```

要创建一个 Nest 应用实例，我们使用了 `NestFactory` 核心类。`NestFactory` 暴露了一些静态方法用于创建应用实例。 `create()` 方法返回一个实现 `INestApplication` 接口的对象, 并提供一组可用的方法, 在后面的章节中将对此进行详细描述。 在上面的main.ts示例中，我们只是启动 HTTP 服务器，它允许应用程序等待入站 HTTP 请求。