## 版本对应

>  CentOS7.6 对应jenkins正常运行版本为 2.350， 对应java jdk环境为Java11

## 安装

### 查看本地是否自带 java 环境

```shell
yum list installed | grep java
```

### 卸载自带的 Java，使用 root 用户进行卸载，并且删除原先版本的Jenkins

```shell
yum -y remove java-1.8.0-openjdk* 
yum -y remove tzdata-java*

#删除原先版本的jenkins
systemctl stop jenkins.service
rpm -e jenkins
rpm -qa | grep jenkins      # 查看是否还有jenkins依赖，有就删除
rm -rf /etc/sysconfig/jenkins.rpmsave
rm -rf /var/cache/jenkins/
rm -rf /var/lib/jenkins/
rm -rf /var/log/jenkins
rm -rf /usr/lib/jenkins
```

### 安装Java

```shell
yum install java-11-openjdk.x86_64
```

### 安装Jenkins

```shell
wget https://pkg.jenkins.io/redhat/jenkins-2.350-1.1.noarch.rpm

rpm -ivh jenkins-2.350-1.1.noarch.rpm
```

### 修改当前目录权限

```shell
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

### 重启jenkins服务

```shell
systemctl restart jenkins
```

### 赋值权限

```shell
chown -R jenkins /root
```

## jenkins常见指令

### 启动jenkins

```shell
systemctl start jenkins
```

### 查看jenkins运行状态

```shell
systemctl status jenkins
```

### 设置服务器开机自启动jenkins

```shell
systemctl enable jenkins
```

### 重启jenkins服务

```shell
systemctl restart jenkins
```

