# 基于docker 搭建 promtheus、node-exporter、alertmanager



## 搭建 node-exporter 组件

`node-export` 组件是监控服务器硬件信息，cpu、内存、磁盘、温度等。

1. 创建 `node-exporter` 目录

2. 在 `node-exporter` 目录下，创建 `Dockerfile` 

   ```dockerfile
   FROM docker.io/prom/node-exporter
   MAINTAINER "shenguangyang"<shenguangyang@jdimage.cn>
   #同步时间
   #ENV TZ Asia/Shanghai
   #RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
   ```

3. 在node-exporter目录下，创建 `start.sh` 文件 

   ```shell
   docker stop node-exporter
   docker rm node-exporter
   docker rmi node-exporter
   docker build -t node-exporter .
   
   docker run --privileged=true -dt -p 9100:9100 \
     -v /proc:/host/proc:ro \
     -v /sys:/host/sys:ro \
     -v /:/rootfs:ro \
     --net=host \
     --name node-exporter node-exporter \	
   ```

   

4. root账号运行， `sh start.sh` 

5. 浏览器打开 [http://192.168.2.100:9100/metrics](http://192.168.2.100:9100/metrics) ，显示如下说明搭建成功。

   ![node-exporter](../../img/prometheus/node-exporter.png)



## 搭建 prometheus server

1. 创建 `prometheus ` 文件夹

2. 在 `prometheus ` 文件夹下，创建 `Dockerfile` 

   ```dockerfile
   FROM docker.io/prom/prometheus
   MAINTAINER "shenguangyang"<shenguangyang@jdimage.cn>
   #同步时间
   #ENV TZ Asia/Shanghai
   #RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
   ```

3. 创建 `prometheus.yml` 

   ```yml
   # 全局设置
   global:
     scrape_interval:     15s # 15s 执行一次
     evaluation_interval: 15s # 15s 执行一次
   
   # 监控配置
   scrape_configs:
     - job_name: 'prometheus-host' 
       static_configs:
       - targets: ['192.168.2.100:9090']
   
     - job_name: 'node-exporter' # 目标主机
       static_configs:
       - targets: ['192.168.2.100:9100']
   
   ```

   

4. 创建 `prometheus` 启动文件  `start.sh` 

   ```shell
   docker stop prometheus
   docker rm prometheus
   docker rmi prometheus
   docker build -t prometheus .
   docker run --privileged=true  -d  -p 9090:9090 \
    -v "$(pwd)"/prometheus.yml:/etc/prometheus/prometheus.yml \
    -v "$(pwd)"/rule.yml:/etc/prometheus/rule.yml \
    --name prometheus prometheus --config.file=/etc/prometheus/prometheus.yml \
    --web.enable-lifecycle \
   ```

   

5. root账号运行， `sh start.sh` 

6. 浏览器打开，[http://192.168.2.100:9090/targets](http://192.168.2.100:9090/targets) 显示如下说明搭建成功

   ![prometheus](../../img/prometheus/prometheus.png)



## 搭建 grafana 图形化展示组件



1. 创建 `grafana` 文件夹 

2. 在 `grafana` 文件夹下创建 `Dockerfile` 

   ```dockerfile
   FROM docker.io/grafana/grafana
   MAINTAINER "shenguangyang"<shenguangyang@jdimage.cn>
   #同步时间
   #ENV TZ Asia/Shanghai
   #RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
   ```

3. 在 `grafana` 文件夹，创建 grafana-storage 文件夹，设置 777 权限

   ```shell
   mkdir grafana-storage
   chmod 777 -R grafana-storage
   ```

4. 在 `grafana` 文件夹，创建 `start.sh` 启动文件 

   ```shell
   docker stop grafana
   docker rm grafana
   docker rmi grafana
   docker build -t grafana .
   
   docker run --privileged=true  -dt -p 3000:3000 \
    -v "$(pwd)"/grafana-storage:/var/lib/grafana \
    --name grafana grafana
   ```

5. 运行 `start.sh` 启动文件 

6. 浏览器打开 [http://192.168.2.100:3000/](http://192.168.2.100:3000/) 初始默认密码admin、admin

7. 在 [https://grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards) 查找需要展示图形模板，记录id

8. 在如图操作

   1. 点击如图所示按钮

      ![grafana](../../img/prometheus/granfa.png)

   2. 把现在好的模板id复制过来

      ![grafana 2](../../img/prometheus/granfa-2.png)

   3.  选择数据库

      ![grafana 3](../../img/prometheus/granfa-3.png)

   4. 看到如下如展示说明成功

      ![grfana 4](../../img/prometheus/granfa-4.png)





## 搭建 alertmanager 组件（实现邮件报警）

主要流程：搭建alertmanager组件、配置prometheus、配置报警规则

1. 创建 `alertmanager ` 文件夹

2. 在 `alertmanager ` 文件夹下创建 `Dockerfile` 

   ```dockerfile
   FROM docker.io/prom/prometheus
   MAINTAINER "shenguangyang"<shenguangyang@jdimage.cn>
   #同步时间
   #ENV TZ Asia/Shanghai
   #RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
   ```

3. 在 `alertmanager ` 文件夹下创建 `alertmanager.yml` 

   ```yml
   global:
     smtp_smarthost: 'smtp.163.com:25'
     smtp_from: '....@163.com'	# 发送人邮箱
     smtp_auth_username: '15268183286@163.com' # 发送人邮箱
     smtp_auth_password: '0UdGZlNvjDVMxKjX' # 这里是邮箱的授权密码，不是登录密码
     smtp_require_tls: false
   templates:
     - 'etc/alertmanager/template/email.tmpl'
   route:
     group_by: ['alertname', 'cluster', 'service']
     group_wait: 30s
     group_interval: 5m
     repeat_interval: 10m
     receiver: default-receiver
   receivers:
   - name: 'default-receiver'
     email_configs:
     - to: '434539550@qq.com' # 接收人
       headers: { Subject: "[WARN] 报警邮件 test" }
   ```

4. 在 `alertmanager ` 文件夹下创建 `template` 文件夹

5. 在 `template` 文件夹下创建 `email.tmpl` 

   ```go
   {{ define "wechat.default.message" }}
   {{ range $i, $alert :=.Alerts }}
   ========监控报警==========
   告警状态：{{   .Status }}
   告警级别：{{ $alert.Labels.severity }}
   告警类型：{{ $alert.Labels.alertname }}
   告警应用：{{ $alert.Annotations.summary }}
   告警主机：{{ $alert.Labels.instance }}
   告警详情：{{ $alert.Annotations.description }}
   触发阀值：{{ $alert.Annotations.value }}
   告警时间：{{ $alert.StartsAt.Format "2006-01-02 15:04:05" }}
   ========end=============
   {{ end }}
   {{ end }}
   ```

6. 在 `alertmanager ` 文件夹下创建 `start.sh` 

   ```shell
   docker stop alertmanager
   docker rm alertmanager
   docker rmi alertmanager
   docker build -t alertmanager .
   docker run --privileged=true -dt -p 9093:9093 \
    -v "$(pwd)"/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
    -v "$(pwd)"/template:/etc/alertmanager/template \
    --name alertmanager alertmanager  \
    --config.file=/etc/alertmanager/alertmanager.yml \
   ```

7. 运行 `sh start.sh`

8. 以下需要配置 **prometheus server** ，所有操作均在`prometheus` 文件下操作

9. 在 `prometheus.yml` 增加以下配置

   ```yml
   # Alertmanager configuration
   alerting:
     alertmanagers:
     - static_configs:
       - targets: ['192.168.2.100:9093']
   
   # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
   rule_files:
     # - "first_rules.yml"
     # - "second_rules.yml"
     - "/etc/prometheus/rule.yml"
   ```

10. 在 `prometheus` 文件夹下创建 `rule.yml`

    ```yml
    groups:
    - name: rules.yml
      rules:
      - alert: InstanceStatus # alert 名字
        expr: up{job="jd-manager-actuator"} == 0 # 判断条件
        for: 5s # 条件保持 10s 才会发出 alter
        labels: # 设置 alert 的标签
          severity: "critical"
        annotations:  # alert 的其他标签，但不用于标识 alert
          description: "服务器  已当机超过 20s"
          summary: "服务器  运行状态"
    ```

11. 重新启动  **prometheus server** 

12. 浏览器打开 [http://192.168.2.100:9090/alerts](http://192.168.2.100:9090/alerts) 出现如下图所示说明配置成功

    ![alertmanager](../../img/prometheus/alertmanager.png)

13. 服务挂掉，可以正常收到邮件。



## grafana无数据展示问题

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







