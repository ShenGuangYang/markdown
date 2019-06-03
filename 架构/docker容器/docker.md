# docker 

## 安装
```shell
yum install -y docker
```

## 启动
```shell
# centos7 启动
systemctl start docker
# 开机启动
systemctl enable docker
```

## docker commit 创建镜像
```shell
# 运行一个docker容器
docker run -ti centos bash
# 安装wget
yum install -y wget
# 退出docker容器
exit
# 查看所有docker容器
docker ps -a
# 从容器创建一个新的镜像 
docker commit 92d7f373cd13 wget:0.1
# 列出本地镜像
docker images
```

## Dockerfile创建镜像
```shell
vi /data/docker/Dockerfile
```
添加一下内容
```vi
FROM centos
LABEL maintainer "shen"
RUN yum install -y wget
```
然后运行docker build
```shell
docker build -t wget:0.3
```
