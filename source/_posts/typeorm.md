---
title: typeorm
abbrlink: 63925
cover : https://github.com/typeorm/typeorm/raw/master/resources/logo_big.png
top_img: https://github.com/typeorm/typeorm/raw/master/resources/logo_big.png
date: 2020-07-13 16:55:11
categories: typeorm
tags: [typeorm,orm]
---
# typeOrm

## 1.安装(和官方一样使用mysql,其它数据库请安装其它的依赖包)

```javascript
npm install typeorm reflect-metadata @types/node mysql --save
```

## 2.TypeScript编译器版本3.3或更高版本，并且您已启用以下设置tsconfig.json

```javascript
“ emitDecoratorMetadata ”： true，
“ experimentalDecorators ”： true，
```

## 3.连接到mysql数据库（单个数据库连接）

1. 方法一

```javascript
import {createConnection} from "typeorm";
try{
    const connection = await createConnection({
        type: "mysql", // 数据库类型
        host: "localhost", // host，本地localhost/127.0.0.1
        port: 3306, // 数据库端口号,mysql默认3306
        username: "test", // 登录数据库的账号
        password: "test", // 登录数据库的密码
        database: "test", // mysql中的test数据库
        entities: [__dirname + "/entity/*{.js,.ts}"], //表示扫描到的实例类的位置，src下面的/entity文件夹下面的任意.ts或.js文件
           synchronize: true // 是否同步
    });
}catch(error){//连接出错
    console.log(error);
}
```

2. 方法二
   
   在项目根目录（和package.json同级的地方）创建ormconfig.json。抽离出上面的连接对象放到该文件中

```javascript
{
   "type": "mysql",
   "host": "localhost",
   "port": 3306,
   "username": "test",
   "password": "test",
   "database": "test",
   "entities": ["src/entity/*{.js,.ts}"],
   "synchronize": true
}
```

然后连接就不需要传递对象了，typeORM会在根目录下查找ormconfig.json文件，并连接

```javascript
import {createConnection} from "typeorm";
try{
    const connection = await createConnection();
}catch(error){//连接出错
    console.log(error);
}
```

如果是多个数据库连接，就使用数组，存放一个个上面ormconfig.json那样的对象

```javascript
[{
    name: "db1Connection",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "db1",
    entities: ["src/entity/*{.js,.ts}"],
    synchronize: true
}, {
    name: "db2Connection",
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "db2",
    entities: ["src/entity/*{.js,.ts}"],
    synchronize: true
}]
//连接,通过name连接
await createConnection('db1Connection');
await createConnection('db2Connection');
```

## 4.创建实例类

```javascript
import { Entity, Column, PrimaryColumn, Double} from "typeorm";
@Entity()
export class Photo {
    @PrimaryColumn()
    id: number;
    @Column({
        length: 100,
        default: '姓',
        charset: 'utf-8',
        readonly: false,
        type: 'string',
        unique: true,
        insert: true,
        primary: false,
        select: true,
        name: 'name',
        nullable: false,
    })
    name: string;
    @Column('text')
    description: string;
    @Column(type => String)
    filename: string;
    @Column(type => Double)
    views: number;
    @Column()
    isPublished: boolean;
}
```

Entity装饰器，表示这个类是一个实例类，和数据库的表字段一致，便于修改删除等操作
Column装饰器，表示这是一列，就是一个数据库字段
PrimaryColumn装饰器，就是主列，相当于mysql的主键
PrimaryGeneratedColumn:装饰器，也就是自动生成的主键，相当于mysql的自增长主键
Entity装饰的类相当于mongodb数据库的一个文档，Column装饰的属性等于文档里的一个字段
Column装饰器传参:
1.传递一个字符串表示类型,比如text类型，mysql
2.传递一个回调函数,type => Double,表示这是一个double类型的字段
3.传递一个对象，对象参数有许多，介绍几个

## 5.连接到mysql数据库（单个数据库连接）

```javascript
import {createConnection} from "typeorm";
try{
 const connection = await createConnection({
 type: "mysql", // 数据库类型
 host: "localhost", // host，本地localhost/127.0.0.1
 port: 3306, // 数据库端口号,mysql默认3306
 username: "test", // 登录数据库的账号
 password: "test", // 登录数据库的密码
 database: "test", // mysql中的test数据库
 entities: [__dirname + "/entity/*{.js,.ts}"], //表示扫描到的实例类的位置，src下面的/entity文件夹下面的任意.ts或.js文件
 synchronize: true // 是否同步
 });
}catch(error){//连接出错
 console.log(error);
}
```

- ormconfig.json
在项目根目录（和package.json同级的地方）创建ormconfig.json。抽离出上面的连接对象放到该文件中

```json
{
 "type": "mysql",
 "host": "localhost",
 "port": 3306,
 "username": "test",
 "password": "test",
 "database": "test",
 "entities": ["src/entity/*{.js,.ts}"],
 "synchronize": true
}
```

然后连接就不需要传递对象了，typeORM会在根目录下查找ormconfig.json文件，并连接

```javascript
import {createConnection} from "typeorm";
try{
 const connection = await createConnection();
}catch(error){//连接出错
 console.log(error);
}
```

- 如果是多个数据库连接
就使用数组，存放一个个上面ormconfig.json那样的对象

```javascript
[{
 name: "db1Connection",
 type: "mysql",
 host: "localhost",
 port: 3306,
 username: "root",
 password: "admin",
 database: "db1",
 entities: ["src/entity/*{.js,.ts}"],
 synchronize: true
}, {
 name: "db2Connection",
 type: "mysql",
 host: "localhost",
 port: 3306,
 username: "root",
 password: "admin",
 database: "db2",
 entities: ["src/entity/*{.js,.ts}"],
 synchronize: true
}]
//连接,通过name连接
await createConnection('db1Connection');
await createConnection('db2Connection');
```


## 6.创建实例类

```javascript
import { Entity, Column, PrimaryColumn, Double} from "typeorm";
@Entity()
export class Photo {
 @PrimaryColumn()
 id: number;
 @Column({
 length: 100,
 default: '姓',
 charset: 'utf-8',
 readonly: false,
 type: 'string',
 unique: true,
 insert: true,
 primary: false,
 select: true,
 name: 'name',
 nullable: false,
 })
 name: string;
 @Column('text')
 description: string;
 @Column(type => String)
 filename: string;
 @Column(type => Double)
 views: number;
 @Column()
 isPublished: boolean;
}
```

Entity装饰器，表示这个类是一个实例类，和数据库的表字段一致，便于修改删除等操作
Column装饰器，表示这是一列，就是一个数据库字段
PrimaryColumn装饰器，就是主列，相当于mysql的主键
PrimaryGeneratedColumn:装饰器，也就是自动生成的主键，相当于mysql的自增长主键
Entity装饰的类相当于mongodb数据库的一个文档，Column装饰的属性等于文档里的一个字段
Column装饰器传参:
1.传递一个字符串表示类型,比如text类型，mysql
2.传递一个回调函数,type => Double,表示这是一个double类型的字段
3.传递一个对象，对象参数有许多，介绍几个

```javascript
{
        length: 100, // 长度设置，mysql的int类型没有长度，
        default: '姓', // 默认值
        charset: 'utf-8', // 编码格式
        readonly: false, // 是否只读
        type: 'string', // 类型，string，text，time，int，double等等等
        unique: true, // 是否是唯一的
        insert: true, // 是否可写
        primary: false, // 是否是主键
        select: true, // 是否可查询
        name: 'name', // 对应数据库字段名
        nullable: false, // 是否可以为空,false表示不为空
    }
```

## 7.简单连接使用增加一列

```javascript
import {createConnection} from "typeorm";
try{
    const connection = await createConnection();
    //增加
    let photo = new Photo(); // 创建一个对象
    photo.name = "Me and Bears"; // 给对象注入参数
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;
```

```javascript
await connection.manager.save(photo); // 调用manager管理属性，然后.save(对象)，增加一列
// 或者使用
// let photoRepository = connection.getRepository(Photo);
// await photoRepository.save(photo);
console.log("Photo has been saved. Photo id is", photo.id);}
catch(error){//连接出错
    console.log(error);
}

getRepository：传入一个类，表示连接到这个表,然后save添加到这个表中
- 删除，修改，查询
// 1.查询全局，传入类
 const savedPhotos = await connection.manager.find(Photo);
 console.log("All photos from the db: ", savedPhotos);
// or 查询一个，传入条件
let photoRepository = connection.getRepository(Photo);
let meAndBearsPhoto = await photoRepository.findOne({ name: "Me and Bears" });
console.log("Me and Bears photo from the db: ", meAndBearsPhoto);
// 查询全部并返回总条数
let [allPhotos, photosCount] = await photoRepository.findAndCount();
//返回的数组，第一个是全部数据，第二个是总条数
// 2.查询后修改插回去
meAndBearsPhoto.name = "Me, my friends and polar bears";
await photoRepository.save(photoToUpdate);
// 3.查询后删除
await photoRepository.remove(meAndBearsPhoto); 
```

## 8.getConnection、getManager、getRepository

- getConnection：获取连接的数据库

```javascript
import {getConnection} from "typeorm";
import {User} from "./entity/User";

const entityManager = getConnection().manager;
const photo = await entityManager.findOne(Photo, 1);
photo.name = "Umed";
await entityManager.save(photo);
```

- getManager：获取连接管理，

```javascript
import {getManager} from "typeorm";
import {User} from "./entity/User";

const entityManager = getManager();
const photo = await entityManager.findOne(Photo, 1);
photo.name = "Umed";
await entityManager.save(photo);
```

- getRepository：获取连接成功的存储库，接收一个通过Entity装饰的类参数

```javascript
import {getRepository} from "typeorm";
import {User} from "./entity/User";

const userRepository = getRepository(Photo); // getConnection().getRepository() or getManager().getRepository()
const photo = await userRepository.findOne(1);
photo.name = "Umed";
await userRepository.save(photo);
```

## 9.OneToOne,OneToMany,ManyToOne,ManyToMany

(多表关联的一对一，一对多，多对一，多对多的关系) 

```javascript
import {Entity,
Column,
PrimaryGeneratedColumn,
OneToOne,
JoinColumn} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class PhotoMetadata {
/* ... other columns */
@OneToOne(type => Photo, photo => photo.metadata)
@JoinColumn()
photo: Photo;}
```

```javascript
import {Entity, Column, PrimaryGeneratedColumn, OneToOne} from "typeorm"; import {PhotoMetadata} from "./PhotoMetadata";
@Entity()
export class Photo {
/* ... other columns */
@OneToOne(type => PhotoMetadata, photoMetadata => photoMetadata.photo, {
 cascade: true,
})
metadata: PhotoMetadata;
}
```

JoinColumn表示连接另外一个表,OneToOne表示一对一，第一个回调函数是关联的表，第二个回调函数是对面表的字段对应这个实体类
cascade:是否相关联的，为true的话，表示只需要改变整个Photo类，对应关联的PhotoMetadata类的表也会对应改变

```javascript
let photoRepository = connection.getRepository(Photo);
let photos = await photoRepository.find({ relations: ["metadata"] });
//查询全部的photo，并连接到PhotoMetadata数据库，把metadata字段换成那个类对应的表字段
```

其它的多对多，多对一也差不多这样使用

## 10.查询生成器
 
```javascript
const photos = getRepository(Photo) 
.createQueryBuilder("photo") // 创建查询生成器，生成photo对象
 .innerJoinAndSelect("photo.metadata", "metadata") // photo.metadata 关联 metadata属性，并查询
 .getMany(); // 最后查询出全部
```
