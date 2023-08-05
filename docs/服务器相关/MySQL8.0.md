## centos7.6 安装mysql8.0

### 在/usr/local/目录下创建mysql文件夹

```shell
cd  /usr/local

mkdir mysql

cd mysql
```

### 下载

```shell
wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.16-2.el7.x86_64.rpm-bundle.tar
```

### 解压

```shell
 tar -xvf mysql-8.0.16-2.el7.x86_64.rpm-bundle.tar
```

### 安装

```shell
rpm -ivh mysql-community-common-8.0.16-2.el7.x86_64.rpm --nodeps --force

rpm -ivh mysql-community-libs-8.0.16-2.el7.x86_64.rpm --nodeps --force

rpm -ivh mysql-community-client-8.0.16-2.el7.x86_64.rpm --nodeps --force

rpm -ivh mysql-community-server-8.0.16-2.el7.x86_64.rpm --nodeps --force
```

### 查看

```shell
rpm -qa | grep mysql
```

### 初始化相关配置

```shell
mysqld --initialize;
```



!> 如果报错mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

```shell
# 安装对应依赖
yum install -y libaio.so.1

# 如果还是无法解决 进行下面操作
yum install -y libaio

chown mysql:mysql /var/lib/mysql -R;

systemctl start mysqld.service;

systemctl enable mysqld;
```

### 查看初始密码

```shell
cat /var/log/mysqld.log | grep password
```

### 参考文章

https://zhuanlan.zhihu.com/p/368470304

https://blog.csdn.net/weixin_42266606/article/details/80879571

https://blog.csdn.net/qq_26030541/article/details/109446955
