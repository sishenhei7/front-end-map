## 概述

今天我用homebrew安装Mysql8.0，安装完成之后，**用Workbench和Sequel Pro连接数据库都失败了**，并且都报caching_sha2_password相关的错误，经过查资料，原因是Mysql8.0的默认认证方式改用sha2了，但是**Workbench**和**Sequel Pro**里面都没有sha2的插件，所以报错了。我把解决方法记录下来，供以后开发时参考，相信对其他人也有用。

### 解决方法

网上流行的解决方案是把sha2认证**改回以前的认证方式**，方法如下：

```
// 启动Mysql服务
mysql.server start

// 登录Mysql(需要输入密码)
mysql -u root -p

// 选择数据库(这一步不可省略)
use mysql

// 查看plugin设置
select host, user, plugin from user;

// 可以看到root的plugin是caching_sha2_password，我们希望改成mysql_native_password
ALTER USER root@localhost IDENTIFIED WITH mysql_native_password BY 'xxxxx';

// 大功告成，关闭Mysql
exit
mysql.server stop
```

### 其它

其实我们还可以**用ssh进行免密登录**，这样就绕过了caching_sha2_password认证了（我的猜想）

以后用ssh登录试一试~~
