---
title: Nest应用（三）
date: 2020-04-03 17:00:00
tags: nest
categories : nestjs
keywords: ['nodejs','nest']
top_img: http://kamilmysliwiec.com/public/nest-logo.png
cover: https://d33wubrfki0l68.cloudfront.net/e937e774cbbe23635999615ad5d7732decad182a/26072/logo-small.ede75a6b.svg
---
# Nest 指南（三）

## 1、模块

模块是具有 @Module() 装饰器的类。 @Module() 装饰器提供了元数据，Nest 用它来组织应用程序结构。

[NestModule](https://docs.nestjs.com/modules)

![Nestjs中的模块](https://www.itying.com/nestjs/statics/images/module.png)



每个 Nest 应用程序至少有一个模块，即根模块。根模块是 Nest 开始安排应用程序树的地方。事实上，根模块可能是应用程序中唯一的模块，特别是当应用程序很小时，但是对于大型程序来说这是没有意义的。在大多数情况下，您将拥有多个模块，每个模块都有一组紧密相关的功能。

**@module() 装饰器接受一个描述模块属性的对象：**

```javascript
providers		由 Nest 注入器实例化的提供者，并且可以至少在整个模块中共享
controllers	    必须创建的一组控制器
imports	        导入模块的列表，这些模块导出了此模块中所需提供者
exports	        由本模块提供并应在其他模块中可用的提供者的子集
```

### 1、创建模块

```shell
nest g module admin
```

![Nestjs中的模块](https://www.itying.com/nestjs/statics/images/module2.png)

### 2、共享模块

实际上，每个模块都是一个共享模块。一旦创建就能被任意模块重复使用。假设我们将在几个模块之间共享 CatsService 实例。 我们需要把 CatsService 放到 exports 数组中，如下所 示：

```javascript
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';
@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

## 2、TypeORM操作Mysql数据库

Nestjs中操作mysql数据库可以使用Nodejs封装的DB库，也可以使用TypeORM。下面我们主要给大家讲讲在Nestjs中使用TypeORM操作mysql数据库

### 1、关于TypeORM

TypeORM是一个ORM框架，是一款比较成熟的对象关系映射器，它是由typescript写的。 支持 MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, WebSQL等数据库。

### 2、安装TypeORM 操作Mysql模块

Nest 操作Mysql[官方文档](https://docs.nestjs.com/techniques/database)

```shell
npm install --save @nestjs/typeorm typeorm mysql
```

### 3、配置数据库

在app.module.ts中配置数据库连接

```typescript
      import { Module } from '@nestjs/common';
      import { TypeOrmModule } from '@nestjs/typeorm';
      @Module({
        imports: [
          TypeOrmModule.forRoot({
            type: 'mysql',
            host: 'localhost',
            port: 3306,
            username: 'root',
            password: 'root',
            database: 'test',
            entities: [__dirname + '/**/*.entity{.ts,.js}'],
            synchronize: true,
          }),
        ],
      })
      export class AppModule {}
    
```

### 3、配置实体entity

```typescript
      import { PrimaryGeneratedColumn, Column, Entity } from "typeorm";
      @Entity()
      export class Nav {
        @PrimaryGeneratedColumn()
        id: number;
      
        @Column({length: 45})
        name: string;
      
        @Column({length:255})
        url: string;
      
        @Column('int')
        status: number;
      }
    
```

### 4、在控制器对应的Module中配置Model

```typescript
      import { Module } from '@nestjs/common';
      import { UserController } from './controller/user/user.controller';
      import { NewsController } from './controller/news/news.controller';
      import { TypeOrmModule } from '@nestjs/typeorm';
      import {Nav} from '../../entity/nav.entity';
      import {Navinfo} from '../../entity/navinfo.entity';
      import { AppService } from '../../app.service';
      
      @Module({
        imports:[TypeOrmModule.forFeature([Nav,Navinfo])],
        controllers: [UserController, NewsController],
        providers:[AppService]
      })
      export class AdminModule {}

    
```

### 5、在服务里面使用@InjectRepository获取数据库Model实现操作数据库

```typescript
      import { Injectable } from '@nestjs/common';
      import { InjectRepository } from '@nestjs/typeorm';
      import { Repository } from 'typeorm';
      import {Nav} from './entity/nav.entity';
      import {Navinfo} from './entity/navinfo.entity';
      @Injectable()
      export class AppService {
        constructor(
          //依赖注入
          @InjectRepository(Nav) private readonly navRepository: Repository,
          @InjectRepository(Navinfo) private readonly navinfoRepository: Repository,
        ) {}
        async findAll(){   
          return await this.navRepository.find();    
        }
      }
      
```

## 3、Redis

### 1、基本语法

- Redis 字符串数据类型的相关命令用于管理 redis 字符串值。

```json
  基本语法：
  查看所有的key:         keys *
  普通设置：             set key value
  设置并加过期时间：     set key value EX 30           表示30秒后过期
  获取数据：             get key
  删除指定数据：         del key
  删除全部数据:          flushall 
  查看类型：             type key
  设置过期时间:          expire key  20              表示指定的key5秒后过期
  
```

- Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

```json
基本语法：
列表右侧增加值：             rpush key value
列表左侧增加值：             lpush key value
右侧删除值：                 rpop key
左侧删除值：                 lpop key
获取数据：                   lrange key
删除指定数据：               del key
删除全部数据:                flushall 
查看类型：                   type key
```

- Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。它和列表的最主要区别就是没法增加重复值

```
基本语法：
给集合增数据：               sadd key value
删除集合中的一个值：         srem key value
获取数据：                   smembers key
删除指定数据：               del key
删除全部数据:                flushall 
```

- Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

```
  基本语法：
  设置值hmset ：               hmset zhangsan name "张三" age 20  sex “男”
  设置值hset ：                 hset zhangsan name "张三"
  获取数据：                     hgetall key
  删除指定数据：                 del key
  删除全部数据:                  flushall 
```

- Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

> **发布**

```typescript
  client.publish('testPublish', 'message from publish.js');
```

> **订阅**

```typescript
client.subscribe('testPublish');
client.on('message', function(channel, msg){
	console.log('client.on message, channel:', channel, ' message:', msg);
});
```

### 2、Nest使用

[Nestjs Redis 官方文档](https://github.com/kyknow/nestjs-redis )

#### **1、安装redis**

```shell
npm install nestjs-redis --save
```

#### **2、用到redis的模块中注册RedisModule**

options是一个对象，里面配置了连接redis服务器的信息

```typescript
  import { RedisModule} from 'nestjs-redis'
  关于options
  let options={
          port: 6379,
          host: '127.0.0.1',
          password: '',
          db: 0
  }
   @Module({
      imports: [
          RedisModule.register(options)
      ],
  })
```

#### **3、创建一个cache.service.ts 服务 封装操作redis的方法**

```typescript
import { Injectable } from '@nestjs/common';
import { RedisService } from 'nestjs-redis';
@Injectable()
export class CacheService {    
    public client;
    constructor(private redisService: RedisService) {
        this.getClient();
    }
    async getClient() {
        this.client = await this.redisService.getClient()
    }

    //设置值的方法
    async set(key:string, value:any, seconds?:number) {
        value = JSON.stringify(value);
        if(!this.client){
            await this.getClient();
        }
        if (!seconds) {
            await this.client.set(key, value);
        } else {
            await this.client.set(key, value, 'EX', seconds);
        }
    }

    //获取值的方法
    async get(key:string) {
        if(!this.client){
            await this.getClient();
        }
        var data = await this.client.get(key);           
        if (!data) return;
        return JSON.parse(data);       
    }
}
```

#### **4、调用cache.service.ts 里面封装的方法操作redis数据库**

```typescript
await this.cache.set('username','李四');

await this.cache.get('username')
```

## 4、GraphQL

GraphQl是一种新的API 的查询语言，它提供了一种更高效、强大和灵活API 查询,下面给大家讲讲在Nestjs中使用Graphql

Nestjs中使用[Graphql官方文档](https://docs.nestjs.com/graphql/quick-start)

### 1、安装

```shell
 $ npm i --save @nestjs/graphql graphql-tools graphql        
```

### 2、定义数据库Schema

src目录下面新建app.graphql，Schema（数据映射模型）代码如下

```typescript
type Query {
  hello: String
  findCat(id: ID): Cat
  cats: [Cat]
}

type Cat {
  id: Int
  name: String
  age: Int
}

type Mutation {
  addCat(cat: InputCat): Cat
}

input InputCat {
  name: String
  age: Int
}
 
```

### 3、定义 resolvers 操作数据库的方法

通过命令创建resolvers

```shell
nest g resolver app
```

这样会在src目录下面生成app.resolvers.ts ，然后配置如下代码

```typescript
import { ParseIntPipe } from '@nestjs/common';
import { Query, Resolver, Args, Mutation } from '@nestjs/graphql';
import { AppService } from './app.service';

@Resolver()
export class AppResolver {
  constructor(private readonly appService: AppService) {}

  // query { hello }
  @Query()
  hello(): string {
    return this.appService.hello();
  }

  // query { findCat(id: 1) { name age } }
  // 网络传输过来的id会是字符串类型，而不是number
  @Query('findCat')
  findOneCat(@Args('id', ParseIntPipe) id: number) {
    return this.appService.findCat(id);
  }

  // query { cats { id name age } }
  @Query()
  cats() {
    return this.appService.findAll();
  }

  // mutation { addCat(cat: {name: "ajanuw", age: 12}) { id name age } }
  @Mutation()
  addCat(@Args('cat') args) {
    console.log(args);
    return this.appService.addCat(args)
  }
}
        
```

### 4、定义 服务 app.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { Cat } from './graphql.schema';

@Injectable()
export class AppService {
  private readonly cats: Cat[] = [
    { id: 1, name: 'a', age: 1 },
    { id: 2, name: 'b', age: 2 },
  ];
  hello(): string {
    return 'Hello World!';
  }

  findCat(id: number): Cat {
    return this.cats.find(c => c.id === id);
  }

  findAll(): Cat[] {
    return this.cats;
  }

  addCat(cat: Cat): Cat {
    const newCat = { id: this.cats.length + 1, ...cat };
    console.log(newCat);
    this.cats.push(newCat);
    return newCat;
  }
}
        
```

### 5、定义接口 graphql.schema.ts （非必须）

```typescript
export class Cat {
  id: number;
  name: string;
  age: number;
}
        
```

### 6、配置app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { AppService } from './app.service';

import { GraphQLModule } from '@nestjs/graphql';
import { AppResolver } from './app.resolvers';

@Module({
  imports: [
    GraphQLModule.forRoot({
      typePaths: ['./**/*.graphql'],   //加载目录下面所有以graphql结尾的schema文件，当做固定写法
    }), 
  ],
  providers: [AppService, AppResolver],
})
export class AppModule {}
        
```

### 7、操作graphql数据库

```typescript
// 发送
query { hello }

// 返回
{
  "data": {
    "hello": "hello nest.js"
  }
}
```

## 5、Websocket(socket.io实时通信)

在 Nest 中，网关只是一个用 @WebSocketGateway() 装饰器注解的类。从技术上讲，网关与平台无关，这使得它们在创建适配器之后就可以与任何 WebSockets 库兼容。有两个开箱即用的WS平台:socket.io和ws。你可以选择最适合你需要的。另外，您可以按照本指南构建自己的适配器。

[官方文档](https://docs.nestjs.com/websockets/gateways)

### 1、安装 socketIo对应模块

```shell
$ npm i --save @nestjs/websockets @nestjs/platform-socket.io
$ npm i --save-dev @types/socket.io      
```

### 2、生成gateway并配置socket

```shell
nest g gateway events  
```

通过上面命令会在src目录下面生成events.gateway.js

这个里面可以自定义方法接受客户端广播

```typescript
import { 
        SubscribeMessage, 
        WebSocketGateway, 
        WsResponse, 
        WebSocketServer 
        } from '@nestjs/websockets';
import { Observable, of } from 'rxjs';  //异步流数据处理框架
import { map } from 'rxjs/operators'
const l = console.log

@WebSocketGateway()
export class EventsGateway {
  @WebSocketServer() server;

  @SubscribeMessage('events')
  onEvent(client: any, payload: any): Observable> | any {
    // this.server.emit('resmsg', data);  // io.emit('resmsg', payload)
    let { name } = payload;
    if (name === 'ajanuw') {
      return of({
        event: 'events',
        data: {
          msg: 'hello ajanuw!'
        }
      })
    }
    if (name === 'alone') {
      return of('hi', '实打实')
        .pipe(
          map($_ =>
            ({
              event: 'events', data: {
                msg: $_
              }
            }))
        );
    }
    return of(payload);
  }

}
      
```

### 3、app.module.ts

```typescript
app.module.ts
import { EventsGateway } from './events/events.gateway'
@Module({
  providers: [EventsGateway],
})
      
```

### 4、socket客户端

```javascript
    &lt;script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.js"&gt;&lt;/script&gt;
  &lt;script&gt;
    const l = console.log
    let socket = io('http://localhost:5000');
    socket.on('connect', function () {
      console.log('链接成功');
      // 发射
      socket.emit('events', {
        name: 'ajanuw'
      });
      // 发射
      socket.emit('events', {
        name: 'alone'
      });
      // 发射
      // socket.emit('identity', 0, (response) =&gt; console.log('Identity:', response));
    });
    // 监听
    socket.on('events', (data) =&gt; {
      l(data.msg)
    });
  &lt;/script&gt;
  
```

### 5、分组广播和监听进入离开事件

```typescript
  import { SubscribeMessage, WebSocketGateway,WebSocketServer} from '@nestjs/websockets';
  import { of } from 'rxjs';
  import * as url from "url"
  
  @WebSocketGateway()
  export class EventsGateway {
    @WebSocketServer() server;
    
    private clientsArr:any[]=[];
   
    handleConnection(client: any,){
      
      console.log('有人链接了'+client.id);   
   
    }
  
    handleDisconnect(client:any){
     
    }
    
    @SubscribeMessage('addCart')
    addCart(client: any, payload: any) {
         console.log(payload) 
      
        var roomid=url.parse(client.request.url,true).query.roomid;   /*获取房间号 获取桌号*/
        client.join(roomid);
        // this.server.to(roomid).emit('addCart','Server AddCart Ok');    //广播所有人包含自己
  
        client.broadcast.to(roomid).emit('addCart','Server AddCart Ok');   //不包括自己
  
  
    }
  
  }  
```

**客户端代码:**

```typescript
&lt;!DOCTYPE html&gt;
&lt;html lang="zh-CN"&gt;
&lt;head&gt;
    &lt;meta charset="UTF-8"&gt;
    &lt;title&gt;socket.io&lt;/title&gt;
    &lt;script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.js"&gt;&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;socket.io的多房间1111&lt;/h1&gt;
&lt;input type="button" value="加入购物车" onclick="addCart()"&gt;&lt;br&gt;


&lt;/body&gt;
&lt;/html&gt;

&lt;script type="text/javascript"&gt;

    //和服务器建立长连接
    var socket = io.connect('http://localhost:3000?roomid=1');

    //接收服务器返回的信息
    socket.on('addCart',function(data){

        console.log(data);
    });

    function addCart(){
        socket.emit('addCart','addCart');
    }

&lt;/script&gt;
```

## 6、微服务

除了传统的(有时称为单片)应用程序架构之外，Nest 还支持微服务架构风格的开发。本文档中其他地方讨论的大多数概念，如依赖项注入、装饰器、异常过滤器、管道、保护和拦截器，都同样适用于微服务。Nest 会尽可能地抽象化实现细节，以便相同的组件可以跨基于 HTTP 的平台，WebSocket 和微服务运行。

[文档](https://docs.nestjs.com/microservices/basics)

Nest 支持几种内置的传输层实现，称为传输器，负责在不同的微服务实例之间传输消息。大多数传输器本机都支持请求 - 响应和基于事件的消息样式。Nest 在规范接口的后面抽象了每个传输器的实现细节，用于请求 - 响应和基于事件的消息传递。这样可以轻松地从一个传输层切换到另一层，例如，利用特定传输层的特定可靠性或性能功能，而不会影响您的应用程序代码。

通俗的讲:Nestjs中的管道可以将输入数据转换为所需的输出。此外，它也可以处理验证，当数据不正确时可能会抛出异常。

### **1、安装**

```shell
$ npm i --save @nestjs/microservices		
```

> **main.ts**

```typescript
import { NestFactory } from '@nestjs/core';
import { Transport } from '@nestjs/microservices';

import { AppModule } from './app.module';

async function bootstrap() {
	const app = await NestFactory.create(AppModule);
	app.connectMicroservice({
	transport: Transport.TCP,
	});

	await app.startAllMicroservicesAsync();
	await app.listen(5000);
}
bootstrap();
```

> **app.controller.ts**

```typescript
import { Controller, Get, Query, UsePipes } from '@nestjs/common';
import {
  MessagePattern,
  Client,
  Transport,
  ClientProxy,
} from '@nestjs/microservices';
import { AppService } from './app.service';

import { Observable, from } from 'rxjs';

import { ValidationPipe } from './validation.pipe';

@Controller()
export class AppController {
  @Client({ transport: Transport.TCP })
  client: ClientProxy;

  constructor(private readonly appService: AppService) {}

  @UsePipes(new ValidationPipe())
  @Get()
  getHello(@Query('data') data): Observable {
    // const pattern = { cmd: 'sumObservable' };
    // const pattern = { cmd: 'sumAsync' };
    const pattern = { cmd: 'sum' };

    // 使用 send 调用微服务
    const r = this.client.send(pattern, data);
    return r;
  }

  @MessagePattern({ cmd: 'sum' })
  sum(data: number[]): number {
    return data.reduce((acc, el) => acc + el);
  }

  // 返回promise异步响应
  @MessagePattern({ cmd: 'sumAsync' })
  sumAsync(data: number[]): Promise {
    const result = data.reduce((acc, el) => acc + el) + 1;
    return Promise.resolve(result);
  }

  // 程序将响应3次
  @MessagePattern({ cmd: 'sumObservable' })
  sumObservable(data: number[]): Observable {
    return from([1, 2, 3]);
  }
}
```

> **validation.pipe.ts**

```typescript
import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  PayloadTooLargeException,
  BadRequestException,
} from '@nestjs/common';

@Injectable()
export class ValidationPipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    if (metadata.type === 'query') {
      try {
        return JSON.parse(value);
      } catch (error) {
        throw new BadRequestException();
      }
    } else {
      throw new PayloadTooLargeException();
    }
  }
}
```

> **输入：**

```typescript
http://localhost:5000/?data=[1,2, 3] // 6
```