## 安装Node

### 打开要安装的文件夹

```shell
cd /opt
```

### 下载二进制文件

```shell
wget https://nodejs.org/dist/v17.0.0/node-v17.0.0-linux-x64.tar.xz
```

### 解压二进制文件

```shell
tar -xvf node-v17.0.0-linux-x64.tar.xz
```

### 修改文件夹名称（非必须，可不修改）

```shell
mv node-v17.0.0-linux-x64 nodejs
```

### 建立软链接

```shell
ln -s /opt/nodejs/bin/node /usr/local/bin/
ln -s /opt/nodejs/bin/npm /usr/local/bin/
```

### 安装校验

```shell
node -v
npm -v
```

!> 如果开启了 node端口之后，项目无法正常访问，我们需要放行防火墙的对应端口，即可访问

```shell
iptables -I INPUT -p tcp --dport 3000 -j ACCEPT
```

## Git

```shell
# 安装
yum install git
```