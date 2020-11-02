## 概述

今天我给我的 AWS EC2 搭建环境，数据库用的是 Mysql，网上安装 Mysql 的教程大多是安装的 Mysql5，我查了一下最后成功安装了 Mysql8，把心得记录下来，供以后开发时参考，相信对其他人也有用。

参考文档：

[centos7 利用yum安装mysql8.0](https://www.cnblogs.com/yichenscc/articles/10663844.html)

### rpm

rpm(Redhat Package Manager) 是CentOS，特别是 Redhat 的包管理器。

rpm 的常用命令如下：

1.查询所有的安装包：

```
rpm -qa
```

所以我们可以用这个指令查看有没有安装 Mysql：

```
rpm -qa | grep mysql
```

2.其它常用指令：

```
-vh：显示安装进度
-U：升级软件包
-qpl：列出RPM软件包内的文件信息
-qpi：列出RPM软件包的描述信息
-qf：查找指定文件属于哪个RPM软件包
-Va：校验所有的RPM软件包，查找丢失的文件
-qa: 查找相应文件，如 rpm -qa mysql
-ql: 查找安装位置
-e --nodeps: 强制删除软件包
```

注意：这里查找的时候需要带包的全名，比如使用```rpm -qa mysql```是查找不到的，需要先使用```rpm -ql | grep mysql```列出所有的 mysql 包和全名，然后查找包的全名，比如```rpm -ql mysql80-community-release-el7-3.noarch```

### yum

yum(Yellow dog Updater, Modified)是基于 rpm 的包管理器，比 rpm 好用得多。下面是常用的 yum 命令：

```
yum search              搜索软件包
yum install             安装软件包
yum remove              写在软件包
yum list                列出所有可安装和已经安装的的软件包
yum list | grep mysql   列出所有可安装和已经安装的的 mysql 软件包
yum makecache           缓存软件包信息，便于下次搜索
yum repolist all        作用貌似和 yum list 一样，具体不知

yum-config-manager --disable   禁用软件包
yum-config-manager --enable    启用软件包
```

值得一提的是，yum 是利用 **yum 源**来安装软件包的，只要有相关软件的 yum 源，就能顺利安装。本地的 yun 源配置文件都在```/etc/yum.repos.d```这个文件夹里面。

### 安装过程

只要清楚了上面介绍的东西，就能很方便的安装了，过程如下：

1.在 MySQL 官网中查看 YUM 源 rpm 安装包地址：[https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)

2.使用 wget 远程下载rpm安装包。

```
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

3.安装 rpm 源配置文件。

```
rpm -ivh mysql80-community-release-el7-3.noarch.rpm

// 或者
yum localinstall mysql80-community-release-el7-3.noarch.rpm
```

4.更新 yum 源

```
yum clean all
yum makecache
```

5.查看可安装的软件包

```
yum list | grep mysql
```

6.安装 mysql

```
yum install mysql-community-server
```

### mysql

安装完之后有 2 点需要注意一下：

1.mysql在安装后会创建一个root@locahost账户，并且把初始的密码放到了/var/log/mysqld.log文件中。查看初始密码：

```
cat /var/log/mysqld.log | grep password
```

2.官方限制的密码位数大小写等，登录 mysql 后，关闭它们的指令是：

```
set global validate_password.policy=0;
set global validate_password.length=4;
```


