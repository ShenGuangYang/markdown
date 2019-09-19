# 简介

MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

在高负载的情况下，添加更多的节点，可以保证服务器性能。

MongoDB 旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组

## 主要特点

- MongoDB的提供了一个面向文档存储，操作起来比较简单和容易。
- 你可以在MongoDB记录中设置任何属性的索引 (如：FirstName=”Sameer”,Address=”8 Gandhi Road”)来实现更快的排序。
- 你可以通过本地或者网络创建数据镜像，这使得MongoDB有更强的扩展性。
- 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片。
- Mongo支持丰富的查询表达式。查询指令使用JSON形式的标记，可轻易查询文档中内嵌的对象及数组。
- MongoDb 使用update()命令可以实现替换完成的文档（数据）或者一些指定的数据字段 。
- Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作。
- Map和Reduce。Map函数调用emit(key,value)遍历集合中所有的记录，将key与value传给Reduce函数进行处理。
- Map函数和Reduce函数是使用Javascript编写的，并可以通过db.runCommand或mapreduce命令来执行MapReduce操作。
- GridFS是MongoDB中的一个内置功能，可以用于存放大量小文件。
- MongoDB允许在服务端执行脚本，可以用Javascript编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可。
- MongoDB支持各种编程语言:RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言。
- MongoDB安装简单。



## MongoDB 有用资源

1、 MongoDB 官网地址 : https://www.mongodb.com/
2、 MongoDB 官方英文文档 : https://docs.mongodb.com/manual/
3、 MongoDB 各平台下载地址： [MongoDB 3.4.9 Community Server](https://www.mongodb.com/download-center#community)

# 术语



|  SQL 术语   | MongoDB 术语 |                解释                 |
| :---------: | :----------: | :---------------------------------: |
|  database   |   database   |               数据库                |
|    table    |  collection  |            数据库表/集合            |
|     row     |   document   |           数据记录行/文档           |
|   column    |    field     |             数据字段/域             |
|    index    |    index     |                索引                 |
| table joins |              |        表连接,MongoDB不支持         |
| primary key | primary key  | 主键,MongoDB自动将_id字段设置为主键 |



# 安装

## windows 平台安装MongoDB

下载msi包安装即可

## Linux 平台安装 MongoDB

1、 下载

```shell
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.2.0.tgz
```

2、 解压

```shell
tar -zxvf mongodb-linux-x86_64-rhel70-4.2.0.tgz
```

3、 将解压包拷贝到指定目录

```shell
mv mongodb-linux-x86_64-rhel70-4.2.0 /usr/local/mongodb
```

4、 添加 PATH MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 **PATH** 路径中，并重新加载环境变量

```shell
vim /etc/profile
export MONGODB_HOME=/usr/local/mongodb
export PATH=$MONGODB_HOME/bin:$PATH
source /etc/profile
```

5、创建数据存储目录和日志文件目录

```shell
mkdir -p /usr/local/mongodb/data/db
mkdir -p /usr/local/mongodb/logs
```

6、编辑MongoDB启动配置文件 

```shell
/usr/local/mongodb/bin ##在此文件夹下创建一个mongodb.conf文件，手动创建并编辑。
cd /usr/local/mongodb/bin
vim mongodb.conf 
## 添加上下面配置
dbpath = /usr/local/mongodb/data/db 
logpath = /usr/local/mongodb/logs/mongodb.log 
fork = true ##后台运行
auth=false
bind_ip=0.0.0.0
```

7、启动MongoDB

```shell
mongod -f mongodb.conf
```

8、连接MongoDB并访问

```shell
mongo
```



## docker安装MongoDB

==所有命令均在一个文件夹下执行==

1、 安装docker，创建DockerFile文件

```shell
vim DockerFile
#添加一下内容
FROM docker.io/mongo
MAINTAINER "shenguangyang"<4345@qq.cn>
ENV TZ Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

2、 创建启动命令脚本

```bash
vim rebuild.sh
#添加一下内容
docker stop mongodb #停止mongodb
docker rm mongodb   #删除容器mongodb
docker rmi mongodb	#删除镜像mongodb
docker build -t mongodb .   #重新构造
docker run --privileged=true -dt 
	-m 128M 			#内存
	-p 27017:27017 \ 	#端口映射
    -v /home/jdimage/docker/mongodb/logs:/logs \	#地址挂载
    -v /home/jdimage/docker/mongodb/db:/db     \    #地址挂载
    --name mongodb mongodb        \				#名称
    --dbpath /db              \
    --logpath /logs/mongo.log \
    #--auth   #使用用户名密码

```

3、 本地创建dbpath、logpath目录

4、运行容器

```shell
./rebuild.sh
docker exec -it mongodb bash
```

5、MongoDB使用

 1. 用户的创建和数据库的建立

    ```bash
    mongo
    ```

 2. 创建用户

     ```bash
        # 进入 admin 的数据库
        use admin
        # 创建管理员用户
        db.createUser(
           {
             user: "admin",
             pwd: "123456",
             roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
           }
         )
         # 创建有可读写权限的用户. 对于一个特定的数据库, 比如'demo'
         db.createUser({
             user: 'test',
             pwd: '123456',
             roles: [{role: "read", db: "demo"}]
         })
     ```

 3. 建立数据库

     ```bash
     #使用数据库
     use demo;
     #先写入一条数据
     db.info.save({name: 'test', age: '22'})
     #查看写入的数据
     db.info.find();
     #结果如下
     { "_id" : ObjectId("5c973b81de96d4661a1c1831"), "name" : "test", "age" : "22" }
     ```

 4. 远程连接的开启

     ```bash
     #更新源、安装 vim
     apt-get update && apt-get install vim
     # 修改 mongo 配置文件
     vim /etc/mongod.conf.orig
     
     bindIp: 127.0.0.1 #注释掉`# bindIp: 127.0.0.1` 或者改成`bindIp: 0.0.0.0` 即可开启远程连接
     ```

     



# 命令

##  连接数据库



## 切换数据库



## 删除数据库



## 备份数据

