# Docker学习



## Docker安装

```shell
yum install -y docker
```

> [老实点修改镜像地址,不然等的你奔溃](https://github.com/ShenGuangYang/my_note/blob/master/linux/centos%E4%BF%AE%E6%94%B9%E9%95%9C%E5%83%8F.md)

## Docker启动

```shell
# centos7 启动
systemctl start docker
# 开机启动
systemctl enable docker
```

## Docker常用指令

### 帮助命令

```shell
docker version
docker info 
docker --help
```

### 镜像命令

```shell
docker images # 列出本地主机上的镜像
docker search XXX # 查询XXX镜像
docker pull NAME[:TAG] # 拉取镜像
docker rmi XXX #删除镜像
```

### 容器命令

```shell
docker run [OPTIONS] IMAGE [COMMAND] [ARG...] # 新建并启动项目
docker ps # 列出当前所有正在运行的容易
docker start XXX # 启动容器
docker restart XXX # 重启容器
docker stop XXX # 停止容器
docker kill XXX # 强制停止容器
docker rm XXX # 删除容器
docker logs XXX # 查看容器日志
docker top XXX # 查看容器内运行的进程
docker inspect XXX # 查看容器内部细节
```



## Dockerfile 









## Docker搭建私服

和Maven 一样，Docker 不仅提供了一个中央仓库，同时也允许我们搭建私有仓库。如果读者对Maven有所了解，将会很容易理解私有仓库的优势：

- 节省带宽，镜像无需从中央仓库下载，只需从私有仓库中下载即可

- 对于私有仓库中已有的镜像，提升了下载速度
- 便于内部镜像的统一管理

下面我们来讲解一下如何搭建、使用私有仓库

准备两台安装有Docker 的CentOs7的机器（仅供参考）：

| 主机  | ip            | 角色           |
| ----- | ------------- | -------------- |
| node0 | 192.168.1.100 | Docker开发机   |
| node1 | 192.168.1.101 | Docker私有仓库 |

