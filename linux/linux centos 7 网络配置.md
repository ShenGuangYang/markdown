# linux 网络配置

## centos 配置NAT模式网络-分配静态ip

1. 虚拟机设置成NAT 模式

2. 记住NAT设置中的子网IP、子网掩码、网关IP三项，接下来配置文件主要是这三项。
3. 编辑Linux中的网络配置文件

```vim
vi /etc/sysconfig/network-scripts/ifcfg-ens33   #注 网络配置文件名可能会有不同，在输入到ifcfg时，可以连续按两下tab键，获取提示，比如我的机器 为 ifcfg-ens33
```

内容替换如下：

```
TYPE=Ethernet
BOOTPROTO=static
NAME=ens32
UUID=f366b723-41b4-4474-9768-495deffd8bbd
DEVICE=ens32
ONBOOT=yes
IPADDR=192.168.1.200
PREFIX=24
GATEWAY=192.168.1.2
DNS1=8.8.8.8
DNS2=8.8.8.4
```

4. 重启网络
   
```shell
service network restart
```
