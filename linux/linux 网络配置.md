# linux 网络配置

## 设置ip、网关、掩码

```sh
$ cd /etc/network/
$ vi interfaces

##### vi 输入 ######
#iface ens32 inet dhcp
iface ens32 inet static
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.2

```

## 设置dns

```sh
$ cd /etc/resolvconf/resolv.conf.d/
$ vi base

##### vi 新增 ######
nameserver 192.168.1.2 (网关地址)
# 可选
nameserver 114.114.114.114
nameserver 8.8.8.8

```

## 重启服务 或者 重启电脑

```sh
# 重启服务
$ cd /etc/init.d/
$ ./networding restart
# 重启电脑指令
$ sudo init 6
```