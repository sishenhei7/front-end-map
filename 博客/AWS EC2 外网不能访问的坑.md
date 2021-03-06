## 概述

今天我在 AWS EC2 上配置并启动了 nginx，但是**通过外网不能访问**，查了一下资料终于解决了，记录下来供以后开发时参考，相信对其它人也有用。

### 外网访问不了的原因

外网访问不了的原因不外乎有 2 个：

1. 防火墙
2. 安全组

### 防火墙

linux 上的防火墙就是 firewall 了，可以用下面的任意一种方式查看是否开启了防火墙：

```
firewall-cmd --state
systemctl status firewalld
status iptables.service
```

这里 firewall 的相关知识我没有深入学习，就略过了~

### 安全组

像阿里云、亚马逊的 linux 实例里面都设置有安全组。安全组起着虚拟防火墙的作用，可控制一个或多个实例的流量。安全组的设置在阿里云或亚马逊的后台里面设置。

比如说我的 AWS EC2 在外网不能访问，就是因为安全组没有对外开启 80 端口。所以只需要在 AWS EC2 后台的安全组里面设置如下规则即可：

```
类型：http
协议：TCP
端口范围：80
来源：0.0.0.0/0, ::/0

类型：https
协议：TCP
端口范围：433
来源：0.0.0.0/0, ::/0
```

**注意：由于没有配置 ICMP 规则，所以外网可以访问，但是 ping 不通~~**

### 其它

我们知道，http 的默认端口是 80；https 的默认端口是 433

还有，ssh 的默认端口是 22；telnet 的默认端口是 23；ftp 的默认端口是 20 和 21，其中 20 负责连接，21 负责传输数据。

