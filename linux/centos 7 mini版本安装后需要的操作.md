# centos 7 mini版本安装后需要的操作



## 配置普通用户的sudo权限

1. 切换到root用户下，命令是：

```shell
su 
```

2. 添加sudo文件的写权限，命令是:

```shell
chmod u+w /etc/sudoers
```

3. 编辑sudoers文件

```shell
vi /etc/sudoers
# root ALL=(ALL) ALL,在他下面添加xxx ALL=(ALL) ALL (这里的xxx是你的用户名)
```

4. 撤销sudoers文件写权限,命令:

```shell
chmod u-w /etc/sudoers
```

5. 这样普通用户就可以使用sudo了

## 配置网络连接

1. 使用nat模式，配置虚拟机的子掩码、网关等信息

![png](../img/network配置.png)

2. 打开centos-7 mini，获取mac地址并记录下来，输入指令：

```shell
ip addr
```

![1559808908199](../img/network_mac.png)

3. 配置网络信息

```shell
cd /etc/sysconfig/network-scripts
vi ifcfg-ens32 #各个电脑会有差别
```

添加修改如下信息

![1559809219954](../img/network2.png)



4. 重启网络

```shell
service network restart
```



## 安装必要的工具

```shell
# 下载文件命令
yum -y install wget
# 命令行自动补全
yum -y install bash-completion
# 安装第三方软件包库
yum -y install epel-release
# 安装gcc gcc-c++ make
yum -y install gcc gcc-c++ make
# 安装vim
yum -y install vim*

# 安装ifconfig命令
yum -y install net-tools
# 安装完整版man-page
yum -y install man-pages
# 域名解析
yum -y install bind-utils


```



## 安装docker

```shell
# 查看内核，建议3.10以上
uanme -a
# 更新最新yum
yum update
# 安装需要的软件包
yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装docker
yum -y install docker
# 启动docker
systemctl start docker
# 设置为开机启动
systemctl enable docker
# 验证是否安装成功(有client和service两部分表示docker安装启动都成功了)
docker version

```





## 配置yum源(可选)

1. 首先备份/etc/yum.repos.d/CentOS-Base.repo

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

2. 下载对应版本repo文件, 放入/etc/yum.repos.d/(操作前请做好相应备份)

```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

3. 运行以下命令生成缓存

```shell
yum clean all
yum makecache
```



## xshell 远程连接不上问题(可选)

```shell
vi /etc/ssh/sshd_config
```

![img](..\img\xshell.png)



## 防火墙设置（可选）

```shell
# 启动
systemctl start firewalld
# 关闭
systemctl stop firewalld
# 查看状态
systemctl status firewalld 
# 开机启动
systemctl enable firewalld
# 开机禁用
systemctl disable firewalld

=============

# 那怎么开启一个端口呢
# 添加（--permanent永久生效，没有此参数重启后失效）
firewall-cmd --zone=public --add-port=80/tcp --permanent  
# 重新载入
firewall-cmd --reload
# 查看
firewall-cmd --zone= public --query-port=80/tcp
# 删除
firewall-cmd --zone= public --remove-port=80/tcp --permanent
```

