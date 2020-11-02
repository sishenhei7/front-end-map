## 概述

今天我在我的 AWS EC2 服务器上安装了 nodejs。没想到竟然这么麻烦，比在 windows 和 mac 上麻烦多了。所以我把心得记录下来，供以后开发时参考，相信对其他人也有用。

参考资料：

[LINUX 安装NODEJS环境以及路径配置](https://www.cnblogs.com/ldld/p/7400086.html)
[]()

### 选择包管理器

**对于不同的服务器，需要选择不同的包管理器**。比如 macOS 就建议用 homebrew。

我的服务器是 CentOS 系统，所以使用 yum。

在安装了 yum 之后，最好也安装一下 wget:

```bash
yum install wget
```

### 获取 nodejs 包

我们尝试用 ```yum install nodejs```，但是提示包不存在。所以我们使用 wget 在线下载 nodejs 包。

我们打开[nodejs 官方下载页](https://nodejs.org/en/download/)，找到 Linux Binaries (x64) 这行，然后复制下载地址 [https://nodejs.org/dist/v10.16.3/node-v10.16.3-linux-x64.tar.xz](https://nodejs.org/dist/v10.16.3/node-v10.16.3-linux-x64.tar.xz)。

注意：千万不要点 source code 那一行，就是 .tar.gz 结尾的那个。

然后我们使用 wget 下载：

```bash
wget https://nodejs.org/dist/v10.16.3/node-v10.16.3-linux-x64.tar.xz
```

### 安装并改名

我们下载下来的包一般会放在当前的文件夹，我当前的文件夹是 yangzhou 用户的主文件夹，路径是 /home/yangzhou，也可以用 pwd 查看当前目录的路径。

我们希望把 nodejs 放到** /usr/local **路径下面，所以我们执行下面操作：

```bash
cp node-v10.16.3-linux-x64.tar.xz /usr/local   ## 复制
xz -d node-v10.16.3-linux-x64.tar.xz           ## 将tar.xz解压成tar文件
tar -xvf node-v10.16.3-linux-x64.tar           ## 将tar文件解压成文件夹
mv node-v10.16.3-linux-x64 node                ## 改名为 node

rm -rf node-v10.16.3-linux-x64.tar             ## 删除不必要的包
cd /home/yangzhou                              ## 切换回以前的文件夹
rm -rf node-v10.16.3-linux-x64.tar.xz          ## 删除不必要的包
cd /usr/local                                  ## 切换回来
```

### 检查 node 是否可以启动

我们执行下面的命令：

```bash
/usr/local/node/bin/node -v
```

如果能够正常输出版本号，则证明 node 可以正常启动。

### 配置软连接

我们需要 node 可以在**全局启动**，所以配置如下软连接：

```bash
ln -s /usr/local/node/bin/node /usr/bin/node  ## 将node源文件映射到usr/bin下的node文件
ln -s /usr/local/node/bin/npm /usr/bin/npm
```

然后我们检查是否可以全局启动：

```bash
node -v
npm -v
```

如果都能输出版本号则证明可以全局启动。

### 配置 node 的全局安装文件夹和缓存

我们给 node 设置**全局安装文件夹和缓存**：

```bash
cd /usr/local/node
mkdir node_global
mkdir node_cache
npm config set prefix "node_global"
npm config set cache "node_cache"
```

以后全局安装的包都在 node_global 文件夹里面了。

### 安装 cnpm

有时我们需要使用** cnpm**，所以我们全局安装 cnpm：

```bash
npm install cnpm -g --registry=https://registry.npm.taobao.org
```

我们之前说过，全局安装的包都在 node_global 文件夹里，所以 cnpm 也在里面。我们查到它的路径，然后设置软连接，让 cnpm 全局也可以使用：

```bash
ln -s /usr/local/node/node_global/bin/cnpm /usr/bin/cnpm
```

这样就完成啦~~

