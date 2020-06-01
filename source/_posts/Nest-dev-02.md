---
title: Nest应用（二）
tags: nest
categories: nestjs
keywords:
  - nodejs
  - nest
top_img: 'http://kamilmysliwiec.com/public/nest-logo.png'
cover: >-
  https://d33wubrfki0l68.cloudfront.net/e937e774cbbe23635999615ad5d7732decad182a/26072/logo-small.ede75a6b.svg
abbrlink: 21285
date: 2020-04-03 15:00:00
---
# Nest 6.x 快速指南

## 1、 路由/视图层

如果我们希望在Nest里面管理路由的跳转，即项目保持高耦合的前后端不分离架构，我们则需要对Nest配置模板引擎，针对目前的开发模式和要求，这种方式有很多限制，所以我推荐实行前后端分离的模式。Nest只是处理I/O，专注于提供数据接口。

### 1、安装模板引擎 ejs(或者pugjs)

```javascript
cnpm i ejs --save
```

### 2、配置模板引擎

```javascript
app.setBaseViewsDir(join(__dirname, '..', 'views')) // 放视图的文件
app.setViewEngine('ejs');	
```

### 3、配置完整代码

```javascript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NestExpressApplication } from '@nestjs/platform-express';
import {join} from 'path';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // app.useStaticAssets('public'); 
  app.useStaticAssets(join(__dirname, '..', 'public'),{
    prefix: '/static/',   //设置虚拟路径---保证路径文件下文件安全性
 }); 

  app.setBaseViewsDir(join(__dirname, '..', 'views')) // 放视图的文件（模板浏览器渲染页面）
  app.setViewEngine('ejs');

  await app.listen(3000);
}
bootstrap();
```

### 3、渲染页面

Nestjs中 Render装饰器可以渲染模板

```javascript
import { Get, Controller, Render } from '@nestjs/common';

@Controller()
export class AppController {
  @Get()
  @Render('index') //跳转路由--指定到视图层（View）下具体存在的文件
  root() {
    return { message: 'Hello world!' };
  }
}
```

### 4、ejs

```ejs
&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;
&lt;head&gt;
    &lt;meta charset="UTF-8"&gt;
    &lt;meta name="viewport" content="width=device-width, initial-scale=1.0"&gt;
    &lt;meta http-equiv="X-UA-Compatible" content="ie=edge"&gt;
    &lt;title&gt;Document&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    这是ejs演示代码
    &lt;br&gt;    
    &lt;%=message%&gt;
&lt;/body&gt;
&lt;/html&gt;
	
```

## 2、 注解

注解：即为一个个单独的包装函数，含有特定的实现功能，和java里面Spring一样表示为“@functionName()”,并且多个注解之间没有影响，只是集成所有方法的功能。

## 3、服务

### **1、创建服务**

```shell
nest g service news
```

创建好服务后就可以在服务中定义对应的方法

```javascript
	import { Injectable } from '@nestjs/common';

	@Injectable()
	export class NewsService {	
		findAll(){	
			return [
				{"title":"新闻1"},
				{"title":"新闻2"},
			];
		}
	}
	
```

### **2、使用服务**

1、需要在根模块引入并配置

```javascript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserController } from './user/user.controller';
import { NewsService } from './news/news.service';
import { NewsController } from './news/news.controller';
import { ArticleController } from './article/article.controller';

@Module({
  imports: [],
  controllers: [AppController, UserController, NewsController, ArticleController],
  providers: [AppService, NewsService],
})
export class AppModule {}
```

2、在用到的地方引入并配置

```javascript
	import { Controller, Get ,Render} from '@nestjs/common';

	import { NewsService } from './news.service';
	
	@Controller('news')
	export class NewsController {
	
		constructor(private newsServices:NewsService){}
	
		@Get()
		@Render('default/news')
		index(){
			return {
				newsList:this.newsServices.findAll()
			}
		}
	}
	
```

## 4 、控制器

控制器注解@Controller(),控制层负责处理传入的HTTP请求。在Nest中，控制器是一个带有`@Controller()`装饰器的类。

```javascript
import { Controller, Get, Post, HttpStatus } from '@nestjs/common';

@Controller('users')
export class UsersController {
    @Get('')  // /user
    getAllUsers() {}

    @Get(':id') // user/01
    getUser() {}

    @Post('') // /user
    addUser() {}
}
```

> 获取传值注解

| Nest                     | Express            |
| :----------------------- | ------------------ |
| @Request() / @Req()      | req                |
| @Response() @Res()       | res                |
| @Next()                  | next               |
| @Session()               | res.session        |
| @Param(param?:string)    | req.params[param]  |
| @Body(param?: string)    | req.body[param]    |
| @Query(param?: string)   | req.query[param]   |
| @Headers(param?: string) | req.headers[param] |



## 5、Session

session是另一种记录客户状态的机制，不同的是Cookie保存在客户端浏览器中，而session保存在服务器上。当浏览器访问服务器并发送第一次请求时，服务器端会创建一个session对象，生成一个类似于key,value的键值对， 然后将key(cookie)返回到浏览器(客户)端，浏览器下次再访问时，携带key(cookie)，找到对应的session(value)。 客户的信息都保存在session中

### 1、express-session 使用

**1.安装 express-session**

```shell
cnpm install express-session  --save
```

**2.引入express-session**

```shell
import * as session from 'express-session';
```

**3.配置session中间件**

```javascript
 app.use(session({ secret: 'keyboard cat', cookie: { maxAge: 60000 }}))
```

**4.nestjs中使用session**

```javascript
设置值 req.session.username = "张三";
获取值 req.session.username			
```

### 2、express-session参数

```json
secret	一个String类型的字符串，作为服务器端生成session的签名。 
name	返回客户端的key的名称，默认为connect.sid,也可以自己设置。 
resave	强制保存session即使它并没有变化,。默认为true。建议设置成false。 don't save session if unmodified 
saveUninitialized	强制将未初始化的session存储。当新建了一个session且未设定属性或值时，它就处于
未初始化状态。在设定一个cookie前，这对于登陆验证，减轻服务端存储压力，权限控制是有帮助的。（默认：true）。建议手动添加。
cookie	设置返回到前端key的属性，默认值为{ path: ‘/’, httpOnly: true, secure: false, maxAge: null }。
rolling	在每次请求时强行设置cookie，这将重置cookie过期时间（默认：false）
```

```javascript
app.use(session({
  secret: '12345',
  name: 'name',
  cookie: {maxAge: 60000},
  resave: false,
  saveUninitialized: true
}));
```

### 3、express-session方法

```javascript
req.session.destroy(function(err) {   /*销毁session*/})
req.session.username='张三';     //设置session
req.session.username            //获取session
req.session.cookie.maxAge=0;    //重新设置cookie的过期时间
```

## 6、文件上传

Nestjs内置了文件上传的方法，Nestjs中通过file-upload可以实现单文件上传，多文件上传

Nestjs [file-upload官方文档](https://docs.nestjs.com/techniques/file-upload)

### **1、前端代码**

上传图片的时候From表单中需要配置**enctype="multipart/form-data"**

```ejs
    &lt;form action="user/add" method="post" enctype="multipart/form-data"&gt;    
        &lt;input type="text" name="title1" id="" placeholder="新闻标题"/&gt;    

        &lt;input type="text" name="keywords" id="" placeholder="关键词"/&gt;       

        &lt;input type="text" name="author" id="" placeholder="作者" /&gt;

        &lt;input type="file" name="pic" id="" /&gt;

        &lt;input type="text" name="status" id="" placeholder="状态" /&gt;       

        &lt;input type="submit" value="提交"&gt;        
    &lt;/form&gt;
```

### **2、后端代码**

```javascript
import { 
        Controller, 
        Get, 
        Render, 
        Post,
        UseInterceptors,
        UploadedFile
		} from '@nestjs/common';
import { FileInterceptor,FilesInterceptor } from '@nestjs/platform-express';
@Post('doAdd')
@UseInterceptors(FileInterceptor('pic'))
addUser(@UploadedFile() file,@Body() body){
        console.log(body); 
        console.log(file);     
        const writeImage = createWriteStream(join(__dirname, '..','../public/upload', `${file.originalname}`))
        writeImage.write(file.buffer)
        return '上传成功';
}
```

**3、nestjs多文件上传**

```javascript
import { 
        Controller, 
        Get, 
        Render, 
        Post,
        UseInterceptors,
        UploadedFiles
        } from '@nestjs/common';
import { FileInterceptor,FilesInterceptor } from '@nestjs/platform-express';
@Post('doAddAll')
@UseInterceptors(FilesInterceptor('pic'))
addAllUser(@UploadedFiles() files,@Body() body){
for (const file of files) {
    const writeImage = 
createWriteStream(join(__dirname, '../../', 'public/upload', `${body.name}-${Date.now()}-${file.originalname}`));
    writeImage.write(file.buffer);
}
  return '上传成功';
}
```

## 7、中间件

### **1、创建中间件**

```powershell
nest g middleware init
```

```javascript

import { Injectable, NestMiddleware } from '@nestjs/common';
@Injectable()
export class InitMiddleware implements NestMiddleware {
  use(req: any, res: any, next: () => void) {
    console.log('init');
    next();
  }
}
```

### **2、配置单中间件**

在app.module.ts中继承NestModule然后配置中间件

```javascript
export class AppModule implements NestModule {
	configure(consumer: MiddlewareConsumer) {
		consumer
		.apply(InitMiddleware)
		.forRoutes({ path: '*', method: RequestMethod.ALL })
		.apply(NewsMiddleware)
		.forRoutes({ path: 'news', method: RequestMethod.ALL })
		.apply(UserMiddleware)
        .forRoutes(
            { path: 'user', method: RequestMethod.GET },                                
            { path: '', method: RequestMethod.GET });
	}
}
```

### 2、配置多中间件

```javascript
consumer.apply(cors(), helmet(), logger).forRoutes(CatsController);
```

### 3、函数式中间件

```javascript
export function logger(req, res, next) {
	console.log(`Request...`);
	next();
};
```

### 4、全局中间件

```javascript
const app = await NestFactory.create(ApplicationModule);
app.use(logger);
await app.listen(3000);
```

## 8、管道

文档：[Nestjs-Pipe]( https://docs.nestjs.com/pipes)

通俗的讲:Nestjs中的管道可以将输入数据转换为所需的输出。此外，它也可以处理验证，当数据不正确时可能会抛出异常。也可以是过滤filters函数。

```javascript
const data = {...}
const filterData = data.filter((item)=>return item typeof 'string')
```



### **1、创建管道**

```shell
nest g pipe user  //ng g pipe  pip/user
```

管道创建完成后生成如下代码：

```javascript
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';
@Injectable()
export class UserPipe implements PipeTransform {
        transform(value: any, metadata: ArgumentMetadata) {
        //这个里面可以修改传入的值以及验证转入值的合法性
        return value;
	}
}
```

### **2、使用管道**

```javascript
import { Controller,Get,UsePipes,Query} from '@nestjs/common';
import {UserPipe} from '../../user.pipe';
@Controller('user')
export class UserController {    
	@Get()
	index(){
		return '用户页面';
	}
	@Get('pipe')
	@UsePipes(new UserPipe())
	pipe(@Query() info){
		console.log(info);
		return `this is Pipe`;
	}
}
```

