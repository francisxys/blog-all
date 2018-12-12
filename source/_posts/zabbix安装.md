---
title: zabbix安装
copyright: true
date: 2018-12-11 16:45:55
abbrlink:
tags:
- zabbix
categories:
- monitor
- zabbix
---
## 一、环境配置
关闭selinux
```bash
vim /etc/sysconfig/selinux
SELINUX=disabled
```
## 二、Nginx安装
Nginx在生产环境推荐使用编译方式安装
### 1、安装编译环境、gcc
```bash
yum -y install gcc gcc-c++ automake autoconf libtool make
yum install gcc gcc-c++
```
一般我们都需要先装pcre, zlib，前者为了重写rewrite，后者为了gzip压缩。从ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/ 下载最新的 PCRE 源码包，使用下面命令下载编译和安装 PCRE 包：
### 2、安装pcre
```bash
cd /data/backup
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.42.tar.gz
tar -zxvf pcre-8.42.tar.gz
cd pcre-8.42
./configure
make
make install
```
### 3、安装zlib
从http://zlib.net下载最新的 zlib 源码包，使用下面命令下载编译和安装 zlib包：
```bash
cd /data/backup
wget http://zlib.net/zlib-1.2.11.tar.gz 
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
```
### 4、安装openssl
```bash
cd /data/backup
wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
tar -zxvf openssl-1.1.0h.tar.gz
```
### 5、安装Nginx
```bash
cd /data/backup
wget http://nginx.org/download/nginx-1.14.0.tar.gz
tar -zxvf nginx-1.14.0.tar.gz 
cd nginx-1.14.0
./configure --prefix=/data/nginx/ --with-http_v2_module --with-http_ssl_module --with-http_flv_module --with-http_mp4_module --with-http_sub_module --with-http_stub_status_module --with-http_gzip_static_module --without-http-cache --with-http_realip_module --with-pcre=/data/backup/pcre-8.42 --with-zlib=/data/backup/zlib-1.2.11 --with-openssl=/data/backup/openssl-1.1.0h
make
make install
```
创建Nginx软连接到环境变量
```bash
ln -s /data/nginx/sbin/* /usr/local/sbin/
```
## 三、php安装
Zabbix界面需要支持的PHP组件可以从官网查看，如下图：
![Alt text](http://pjakaipln.bkt.clouddn.com/20181211001.png)
### 1、安装插件
```bash
yum install -y  libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel   libicu-devel
```
### 2、编译安装PHP
```bash
wget http://cn2.php.net/distributions/php-7.2.8.tar.gz
tar -zxvf php-7.2.8.tar.gz 
cd php-7.2.8/
./configure --prefix=/data/php --with-curl --with-freetype-dir --with-gd --with-gettext --with-iconv-dir --with-kerberos --with-libdir=lib64 --with-libxml-dir --with-mysqli --with-openssl --with-pcre-regex --with-pdo-mysql --with-pdo-sqlite --with-pear --with-png-dir --with-jpeg-dir --with-xmlrpc --with-xsl --with-zlib --with-bz2 --with-mhash --enable-fpm --enable-bcmath --enable-libxml --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-opcache --enable-pcntl --enable-shmop --enable-soap --enable-sockets --enable-sysvsem --enable-sysvshm --enable-xml --enable-zip
make
make install
```
复制配置文件
```bash
cp php.ini-development /data/php/lib/php.ini
cp /data/php/etc/php-fpm.conf.default /data/php/etc/php-fpm.conf
```
启动
```bash
/data/php/sbin/php-fpm
```
## 三、Mysql安装
### 1、yum安装
```bash
yum -y install libaio
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
yum localinstall mysql-community-release-el7-5.noarch.rpm
```
验证下是否添加成功
```bash
yum repolist enabled | grep "mysql.*-community.*"
yum install mysql-devel
yum install mysql-community-server
systemctl start  mysqld
```
更改数据存放目录
```bash
mkdir /home/data
mysqladmin -u root -p shutdown
mv /var/lib/mysql /home/data
```
修改 /etc/my.cnf 文件
```bash
[mysqld]
datadir=/data/mysqldata/mysql
socket=/data/mysqldata/mysql/mysql.sock
[mysql]
socket=/data/mysqldata/mysql/mysql.sock
```
授权
```bash
chown -R mysql:mysql /data/mysqldata/mysql
```
重启mysql服务
配置开机自起
```bash
# systemctl is-enabled mysql.service;echo $?
enabled
0
```
如果是 enabled 则说明是开机自动，如果不是，执行
```bash
chkconfig --levels 235 mysqld on
```
修改 /etc/my.cnf 文件，添加字符集的设置
```bash
[mysqld]  
character_set_server = utf8
[mysql]
default-character-set = utf8
```
创建数据库
```bash
create database zabbix default charset utf8;
```
### 2、源码包安装
参考链接：[mysql安装](http://www.cnops.top/2018/11/29/mysql%E5%AE%89%E8%A3%85/)
## 四、Zabbix Server端安装
创建用户
```bash
groupadd zabbix
useradd -g zabbix zabbix
```
安装zabbix server
从官网查找最新稳定版本，当前为3.0.*，下载后解压
```bash
mkdir /data/zabbix
./configure --prefix=/data/zabbix/ --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2 --enable-java
make install
```
启动server进程
```bash
/data/zabbix/sbin/zabbix_server -c /data/zabbix/etc/zabbix_server.conf
```
启动agent进程
```bash
/data/zabbix/sbin/zabbix_agentd -c /data/zabbix/etc/zabbix_agentd.conf
```
可能出现的报错：
```bash
checking for mysql_config... no
configure: error: MySQL library not found
#解决方法
yum -y install mysql-devel
```
```bash
configure: error: Invalid Net-SNMP directory - unable to find net-snmp-config
#解决方法
yum -y install net-snmp net-snmp-devel
```
## 五、Zabbix Web安装
```bash
cd /data/backup/zabbix-3.0.8/frontends/php
cp -a . /data/watch01.sa.mtiancity.com/zabbix/
```
配置nginx可以访问，略过
修改php.ini
```bash
post_max_size = 16M
max_execution_time = 300
date.timezone =Asia/Shanghai
always_populate_raw_post_data = -1
max_input_time = 300
```
导入数据库
```bash
cd /data/backup/zabbix-3.0.8/database/mysql/
mysql -u root zabbix<schema.sql
mysql -u root zabbix<images.sql
mysql -u root zabbix<data.sql
```
打开nginx配置的url访问，如下图：
![Alt text](http://pjakaipln.bkt.clouddn.com/20181211002.png)
也可能出现报错
![Alt text](http://pjakaipln.bkt.clouddn.com/20181211003.png)
此时按照提示，将配置文件放入相应目录即可
初始化完成后，登陆zabbix web，默认用户名：Admin，密码：zabbix
## 六、Zabbix Agnet端安装
安装方法二选一即可，推荐rpm安装
### 1、编译安装
创建用户组和用户
```bash
groupadd zabbix
useradd -g zabbix zabbix
```
yum安装组件
```bash
yum -y install mysql-devel libxml2-devel unixODBC-devel OpenIPMI-devel curl-devel net-snmp-devel

```
安装zabbix agent
```bash
tar -zxvf zabbix-3.0.8.tar.gz
cd zabbix-3.0.8/
./configure --prefix=/data/zabbix/ --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2
make && make install
```
复制配置文件
```bash
cp /data/backup/zabbix_agentd.conf /data/zabbix/etc/
```
启动agent
```bash
/data/zabbix/sbin/zabbix_agentd -c /data/zabbix/etc/zabbix_agentd.conf
```
### 2、rpm包安装
```bash
yum -y install unixODBC
#centos6
rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/6/x86_64/zabbix-agent-3.0.1-1.el6.x86_64.rpm
#centos7
rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-agent-3.0.9-1.el7.x86_64.rpm
```
配置文件位置/etc/zabbix/zabbix_agentd.conf，可以根据实际场景修改
启动客户端
```bash
zabbix_agentd
```