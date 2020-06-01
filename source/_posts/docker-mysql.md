---
title: Docker搭建MySQL服务
tags: Docker
categories: Docker
keywords:
  - Docker
  - MySQL
cover: 'https://labs.mysql.com/common/logos/mysql-logo.svg?v2'
top_img: 'https://labs.mysql.com/common/logos/mysql-logo.svg?v2'
abbrlink: 18102
date: 2020-04-09 23:23:23
---

# [使用Docker搭建MySQL服务](https://www.cnblogs.com/sablier/p/11605606.html)

### 一、安装docker[#](https://www.cnblogs.com/sablier/p/11605606.html#2145598473)

windows 和 mac 版可以直接到官网下载 docker desktop

linux 的安装方法可以参考 https://www.cnblogs.com/myzony/p/9071210.html

可以在shell中输入以下命令检查是否成功安装： `sudo docker version`

### 二、建立镜像[#](https://www.cnblogs.com/sablier/p/11605606.html#244533582)

1. 拉取官方镜像（我们这里选择5.7，如果不写后面的版本号则会自动拉取最新版）

   ```shell
   Copydocker pull mysql:5.7   # 拉取 mysql 5.7
   docker pull mysql       # 拉取最新版mysql镜像
   ```

   [MySQL文档地址](https://hub.docker.com/_/mysql/)

2. 检查是否拉取成功

   ```
   Copy$ sudo docker images
   ```

3. 一般来说数据库容器不需要建立目录映射

   ```shell
   Copysudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
   ```

   - –name：容器名，此处命名为`mysql`
   - -e：配置信息，此处配置mysql的root用户的登陆密码
   - -p：端口映射，此处映射 主机3306端口 到 容器的3306端口

4. 如果要建立目录映射

   ```shell
   Copyduso docker run -p 3306:3306 --name mysql \
   -v /usr/local/docker/mysql/conf:/etc/mysql \
   -v /usr/local/docker/mysql/logs:/var/log/mysql \
   -v /usr/local/docker/mysql/data:/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=123456 \
   -d mysql:5.7
   ```

   - -v：主机和容器的目录映射关系，":"前为主机目录，之后为容器目录

5. 检查容器是否正确运行

   ```shell
   Copydocker container ls
   ```

   - 可以看到容器ID，容器的源镜像，启动命令，创建时间，状态，端口映射信息，容器名字

### 三、连接mysql[#](https://www.cnblogs.com/sablier/p/11605606.html#303544512)

1. 进入docker本地连接mysql客户端

   ```shell
   Copysudo docker exec -it mysql bash
   mysql -uroot -p123456
   ```

2. 使用 Navicat 远程连接mysql

   我只找到了 mac 版本的 Navicat Premiun（如下），没有找到windows和linux的。大家可以自行寻找。

    [百度云连接](https://pan.baidu.com/s/1bcJVyIvFneiEoMZPU-oIbA#list/path=/) 密码: qps3 （该软件包来自著名分享网站 Pirate ，并感谢网友 chaosgod 和 jor_ivy）

   [![img](https://img-blog.csdn.net/20180801090208199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvcl9pdnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)](https://img-blog.csdn.net/20180801090208199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvcl9pdnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   [![img](https://img-blog.csdn.net/20180801090230920?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvcl9pdnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)](https://img-blog.csdn.net/20180801090230920?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2pvcl9pdnk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   安装完之后：复制中文包”zh-Hans.lproj”放到 /Contents/Resources 即可。（应用程序右键显示包内容）

3. 使用远程连接软件时要注意一个问题

   我们在创建容器的时候已经将容器的3306端口和主机的3306端口映射到一起，所以我们应该访问：

   ```
   Copyhost: 127.0.0.1
   port: 3306
   user: root
   password: 123456
   ```

4. 如果你的容器运行正常，但是无法访问到MySQL，一般有以下几个可能的原因：

   - 防火墙阻拦

     ```shell
     Copy# 开放端口：
     $ systemctl status firewalld
     $ firewall-cmd  --zone=public --add-port=3306/tcp -permanent
     $ firewall-cmd  --reload
     # 关闭防火墙：
     $ sudo systemctl stop firewalld
     ```

   - 需要进入docker本地客户端设置远程访问账号

     ```shell
     Copy$ sudo docker exec -it mysql bash
     $ mysql -uroot -p123456
     mysql> grant all privileges on *.* to root@'%' identified by "password";
     ```

     原理：

     ```shell
     Copy# mysql使用mysql数据库中的user表来管理权限，修改user表就可以修改权限（只有root账号可以修改）
     
     mysql> use mysql;
     Database changed
     
     mysql> select host,user,password from user;
     +--------------+------+-------------------------------------------+
     | host                    | user      | password                                                                 |
     +--------------+------+-------------------------------------------+
     | localhost              | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
     | 192.168.1.1            | root     | *A731AEBFB621E354CD41BAF207D884A609E81F5E      |
     +--------------+------+-------------------------------------------+
     2 rows in set (0.00 sec)
     
     mysql> grant all privileges  on *.* to root@'%' identified by "password";
     Query OK, 0 rows affected (0.00 sec)
     
     mysql> flush privileges;
     Query OK, 0 rows affected (0.00 sec)
     
     mysql> select host,user,password from user;
     +--------------+------+-------------------------------------------+
     | host                    | user      | password                                                                 |
     +--------------+------+-------------------------------------------+
     | localhost              | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
     | 192.168.1.1            | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
     | %                       | root      | *A731AEBFB621E354CD41BAF207D884A609E81F5E     |
     +--------------+------+-------------------------------------------+
     3 rows in set (0.00 sec)
     ```