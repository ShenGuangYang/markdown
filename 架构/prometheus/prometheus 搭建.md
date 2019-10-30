# 基于docker 搭建promtheus





## docker 下载镜像包

```dockerfile
docker pull prom/prometheus
docker pull docker.io/prom/node-exporter
docker pull docker.io/grafana/grafana
```



## 运行node-exporter

创建dockerfile

```dockerfile
FROM docker.io/prom/node-exporter
MAINTAINER "shenguangyang"<shenguangyang@jdimage.cn>
#同步时间
#ENV TZ Asia/Shanghai
#RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```

docker启动shell文件 

```shell
docker stop node-exporter
docker rm node-exporter
docker rmi node-exporter

docker build -t node-exporter .
docker run --privileged=true -dt -p 9100:9100 \
  -v /home/jdimage/docker/prometheus/node-exporter/proc:/host/proc:ro \
  -v /home/jdimage/docker/prometheus/node-exporter/sys:/host/sys:ro \
  -v /home/jdimage/docker/prometheus/node-exporter/:/rootfs:ro \
  --net=host \
   --name node-exporter node-exporter \
```





```dockerfile
FROM docker.io/grafana/grafana
MAINTAINER "shenguangyang"<shenguangyang@jdimage.cn>
#同步时间
#ENV TZ Asia/Shanghai
#RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```








## 无数据展示问题

node-exporter、prometheus、grafana都有正常运行的情况下，没有数据展示。

就要查看宿主机与docker容器时间是不是一致

```shell
# 时间同步
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
date # 查看时间
#不行执行下面命令
yum install ntpdate -y; 
ntpdate time.windows.com 
```







