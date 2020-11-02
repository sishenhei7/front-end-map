## 概述

今天我碰到 fabric 和 ssh 的一个坑，记录下来，供以后开发时参考，相信对其他人也有用。

### ssh

今天用 ssh 登录远程服务器用不了 npm，查了下，发现原因是：

ssh登录时**不会加载 .bashrc 而是加载 .bash_profile**，所以以ssh的默认登录不会是 bash ，需要在 .bash_profile 中添加以下代码：

```
if [ -f ~/.bashrc ]; then
   . ~/.bashrc
fi
```

### fabric

我赶了一件蠢事就是把 .bash_profile 文件删了，然后在 .profile 里面引用 .bashrc 文件。然后 fabric 就不能使用 npm 了，查了下，发现原因是：fabric 读取环境变量的时候**只会读 .bash_profile 里面的环境变量**。

所以最好的方法还是在 .bash_profile 里面引入 .bashrc

