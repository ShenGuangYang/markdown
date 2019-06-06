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



## 配置yum源

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



## 安装必要的工具

```shell
# 命令行自动补全
yum -y install bash-completion
# 安装第三方软件包库
yum -y install epel-release
# 域名解析
yum -y install bind-utils
# 下载文件命令
yum -y install wget
# 安装ifconfig命令
yum -y install net-tools
# 安装vim
yum -y install vim*
# 安装完整版man-page
yum install man-pages
# 安装gcc gcc-c++ make
yum -y install gcc gcc-c++ make



```



## xshell 远程连接不上问题

```shell
vi /etc/ssh/sshd_config
```

![img](..\img\xshell)



## 防火墙设置

