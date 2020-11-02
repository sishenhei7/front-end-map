## 概述

很久之前就想研究一下 ssh 的**多秘钥管理**，今天正好有时间就研究了一下，挺简单的，记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[Git - 生成 SSH公钥 , Linux 下多密钥管理](https://juejin.im/post/5c20e89f6fb9a049dc024399)

[.ssh/config 文件的解释算法及配置原则](https://www.cnblogs.com/xjshi/p/9146296.html)

### 查看 ssh-keygen 命令信息

我们首先**查看一下 ssh-keygen 这个命令的信息**，先使用 man :

```
man ssh-keygen
```

提示没有手册信息：

```
No manual entry for ssh-kengen
```

那我们再使用 info :

```
info ssh-keygen
```

能显示 ssh-keygen 的各个参数信息，具体可以自己看。

### 生成 ssh 多秘钥

一般我们生成 ssh 的命令是：

```
ssh-keygen -t rsa
```

但是这会覆盖原先的秘钥。那怎么生成一个新的不覆盖旧的秘钥呢？我们先来介绍一下 ssh-keygen 的**相关指令**：

```
ssh-keygen 参数：

-t 指定密钥类型，默认是 rsa ，可以省略。
-f 指定密钥文件存储文件名。
-C 设置注释文字，比如邮箱。
```

所以我们只需要使用 -f 参数即可：

```
// 生成 github 的 ssh
ssh-keygen -t rsa -f ~/.ssh/id_rsa.github
```

需要注意的是，-C 表示注释，不一定要填邮箱。

### ssh 秘钥复制

生成 ssh 秘钥之后我们需要复制它然后填到 github 里面去，复制命令是：

```
clip < ~/.ssh/id_rsa.github.pub
```

有些 shell 里面没有 **clip 这个命令**，这个时候如果是 mac 的话，可以使用 **pbcopy 命令**：

```
// 方法一：使用 <
pbcopy < ~/.ssh/id_rsa.github.pub

// 方法二：使用管道
cat ~/.ssh/id_rsa.github.pub | pbcopy
```

### ssh 测试是否能够连接成功

然后我们来测试一下能不能免密连接：

```
ssh -T git@github.com
```

提示不能连接，因为需要我们配置一下 config 文件。

### ssh 多秘钥管理

一般我们生成了多秘钥之后，默认是不生效的，需要我们配置一下 **config 文件**。配置模板如下：

```
Host github
	HostName github.com
	IdentityFile ~/.ssh/id_rsa.github
	User git
```

每一行的解释如下：

```
// 设置别名为 github
// 域名为 github.com
// 使用 ssh 文件为 ~/.ssh/id_rsa.github
// 用户为 git
```

这个设置别名的功能非常方便，在设置了别名之后，我们只需要**用别名就可以测试是否能够连接成功**：

```
ssh -T github
```

这个时候就能收到成功提示：

```
Hi xxxxxxx! You've successfully authenticated, but GitHub does not provide shell access.
```

### 一个坑

在用如上方法设置之后，能够正常登陆，但是关机后再开机，又不能正常登陆了，这个时候需要把 rsa 添加到 **ssh 高速缓存**里面去，命令如下：

```
ssh-add -K ~/.ssh/id_rsa
```

那我们肯定不能每次开机都要输入上面的命令吧，我们需要开机自动运行上面的命令：

```
// 创建 .bash_profile
touch ~/.bash_profile
// 在 .bash_profile 中写入命令
echo 'ssh-add -K ~/.ssh/id_rsa' >> ~/.bash_profile
```

然后在开机的时候就会自动运行 ~/.bash_profile 里面的命令了。

但是如果我们使用了别的 bash 比如 zsh，上述的方法可能会不管用，这个时候我们需要打开 zsh 的配置文件，**用配置文件来启动 ~/.bash_profile 这个文件**：

```
// 打开 zsh 的配置文件
open ~/.zshrc
// 写入下面的代码
source ~/.bash_profile
```

或者直接用下面的命令写入：

```
echo 'source ~/.bash_profile' >> ~/.zshrc
```

注意：echo 命令后面如果使用单引号则表示写入**代码片段，会自动换行**；如果使用双引号则表示写入**单一字段，不会自动换行**！
