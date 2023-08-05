## 安装MongoDB4.0

### 使用vim命令创建repo文件

```shell
vim /etc/yum.repos.d/mongodb-org-4.0.repo
```

### 将如下内容粘贴里面

```shell
[mongodb-org-4.0]

name=MongoDB Repository

baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/

gpgcheck=1

enabled=1

gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

### 运行命令 等待安装

```shell
sudo yum install -y mongodb-org
```

### 启动MongoDB

```shell
sudo service mongod start
```

### 暂停MongoDB

```shell
sudo service mongod stop
```

!> 如果mondgo连接失败，是因为mongodb未开启远程连接权限

>  将 /etc/mongodb.conf 的 bind_ip = 127.0.0.1 改为 bind_ip = 0.0.0.0 

!> 但是线上正式库必须设置密码，不然极易遭到攻击
