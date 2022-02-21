---
title: 开源堡垒机jumpserver安装部署
date: 2022-01-14 15:18:48
tags: 堡垒机
---
# 临时高可用架构方案如下


![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E5%A0%A1%E5%9E%92%E6%9C%BA.png)

一，安装mysql数据库

1 ，安装（步骤省略）
```java
yum -y localinstall http://mirrors.ustc.edu.cn/mysql-repo/mysql57-community-release-el7.rpm

yum install -y mysql-community-server

systemctl enable mysqld

systemctl start mysqld

2 ，数据库授权

mysql –uroot

create database jumpserver default charset 'utf8' ;

set global validate_password_policy = LOW ;

create user 'jumpserver' @'%' identified by ' z&cemb2iqYPiC$N3' ;

grant all on jumpserver . * to 'jumpserver' @'%' ;

flush privileges ;

exit
```


二，安装jumperver

1 ，下载安装包
```java
cd /data

wget https://github.com/jumpserver/installer/releases/download/v2.16.3/jumpserver-installer-v2.16.3.tar.gz

tar -xf jumpserver-installer-v2.16.3.tar.gz

cd jumpserver-installer-v2.16.3
```
2 ，调整配置模板

# 根据需要修改配置文件模板, 如果不清楚用途可以跳过修改
```java
cat config-example.txt
```
# 以下设置如果为空系统会自动生成随机字符串填入

## 迁移请修改 SECRET_KEY 和 BOOTSTRAP_TOKEN 为原来的设置

## 完整参数文档 https://docs.jumpserver.org/zh/master/admin-guide/env/

##  MySQL 配置, USE_EXTERNAL_MYSQL=1 表示使用外置数据库, 请输入正确的 MySQL 信息
```java
USE_EXTERNAL_MYSQL=0

DB_HOST= 10.0.77.38

DB_PORT= 3306

DB_USER= jumpserver

DB_PASSWORD= z&cemb2iqYPiC$N3

DB_NAME= jumpserver

  （ # 主要关注数据库相关配置文件）
```
3 ，安装 jumpserver 服务
```java 
[root@jumpserver-slave /data/jumpserver-installer-v2.16.3]$ ./jmsctl.sh install



       ██╗██╗   ██╗███╗   ███╗██████╗ ███████╗███████╗██████╗ ██╗   ██╗███████╗██████╗

       ██║██║   ██║████╗ ████║██╔══██╗██╔════╝██╔════╝██╔══██╗██║   ██║██╔════╝██╔══██╗

       ██║██║   ██║██╔████╔██║██████╔╝███████╗█████╗  ██████╔╝██║   ██║█████╗  ██████╔╝

  ██   ██║██║   ██║██║╚██╔╝██║██╔═══╝ ╚════██║██╔══╝  ██╔══██╗╚██╗ ██╔╝██╔══╝  ██╔══██╗

  ╚█████╔╝╚██████╔╝██║ ╚═╝ ██║██║     ███████║███████╗██║  ██║ ╚████╔╝ ███████╗██║  ██║

   ╚════╝  ╚═════╝ ╚═╝     ╚═╝╚═╝     ╚══════╝╚══════╝╚═╝  ╚═╝  ╚═══╝  ╚══════╝╚═╝  ╚═╝

                                                                     Version:  v2.16.3

需要手动操作的内容如下：

    是否需要自定义 docker 存储目录, 默认将使用目录 /var/lib/docker? (y/n)  (默认为 n): n

    是否需要支持 IPv6? (y/n)  (默认为 n): n

   是否需要自定义持久化存储, 默认将使用目录 /opt/jumpserver? (y/n)  (默认为 n): y

   Persistent storage directory (default /opt/jumpserver): /data/jumpserver

   是否使用外部 MySQL? (y/n)  (默认为 n): y

   请输入 MySQL 的主机地址 (无默认值): 10.0.77.38

   请输入 MySQL 的端口 (默认为3306): 3306

   请输入 MySQL 的数据库(事先做好授权) (默认为jumpserver): jumpserver

   请输入 MySQL 的用户名 (无默认值): jumpserver

   请输入 MySQL 的密码 (无默认值): z&cemb2iqYPiC$N3

    是否使用外部 Redis? (y/n)  (默认为 n): n

    是否需要配置 JumpServer 对外访问端口? (y/n)  (默认为 n): n

```



三，修改docker数据目录
```
#默认docker安装的数据目录为/var/lib/docker，需要迁移到数据盘/data/目录下

mv -f /var/lib/docker /data/docker

ln -sf /data/docker /var/lib/docker

systemctl restart docker
```


四，启动服务
```
# 安装完成后配置文件 /opt/jumpserver/config/config.txt

cd /data/jumpserver-installer-v2.16.3

# 启动

./jmsctl.sh start

# 其他常用操作

# 停止

./jmsctl.sh down

# 卸载

./jmsctl.sh uninstall

# 帮助

./jmsctl.sh –h

```

五，登录验证服务
```
登录：localhost

默认账号：admin   默认密码：admin
```

![流程](https://zkk-1300025204.cos.ap-nanjing.myqcloud.com/%E5%A0%A1%E5%9E%92%E6%9C%BA1.png)

六，部署高可用jumpserver服务



另外一台服务器上部署jumpserver服务，连接第一台服务器得mysql数据库

部署步骤如下：
```
1 ，下载安装包

cd /data

wget https://github.com/jumpserver/installer/releases/download/v2.16.3/jumpserver-installer-v2.16.3.tar.gz

tar -xf jumpserver-installer-v2.16.3.tar.gz

cd jumpserver-installer-v2.16.3
```
2 ，安装服务
```
[root@jumpserver-slave /data/jumpserver-installer-v2.16.3]$ ./jmsctl.sh install

是否需要自定义 docker 存储目录, 默认将使用目录 /var/lib/docker? (y/n)  (默认为 n): n

是否需要支持 IPv6? (y/n)  (默认为 n): n

是否需要自定义持久化存储, 默认将使用目录 /opt/jumpserver? (y/n)  (默认为 n): y

   Persistent storage directory (default /opt/jumpserver): /data/jumpserver

是否使用外部 MySQL? (y/n)  (默认为 n): y

   请输入 MySQL 的主机地址 (无默认值): 10.0.77.38

   请输入 MySQL 的端口 (默认为3306): 3306

   请输入 MySQL 的数据库(事先做好授权) (默认为jumpserver): jumpserver

   请输入 MySQL 的用户名 (无默认值): jumpserver

   请输入 MySQL 的密码 (无默认值): z&cemb2iqYPiC$N3
```
3 ，修改 docker 默认数据目录
```
mv -f /var/lib/docker /data/docker

ln -sf /data/docker /var/lib/docker

systemctl restart docker
```
4 ，启动服务

# 安装完成后配置文件 /opt/jumpserver/config/config.txt
```
cd /opt/jumpserver-installer-v2.16.3
```
# 启动
```
./jmsctl.sh start
```