---
title: mysql安装
abbrlink: 41952
date: 2018-11-29 14:37:43
tags: 
- mysql
- centos
categories: 
- DB
- mysql
copyright: true
---

## 一、软件部署
### 1、MySQL安装
``` bash
## 安装软件依赖
shell> yum install libaio -y

## 创建用户和组
shell> groupadd mysql
shell> useradd -r -g mysql -s /bin/false mysql

## 解压软件包并创建软连接
shell> tar xzvf mysql-5.7.22-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
shell> cd /usr/local/
shell> ln -s mysql-5.7.22-linux-glibc2.12-x86_64 mysql

## 修改软件目录权限为mysql用户
shell> cd /usr/local/mysql
shell> chown -R mysql:mysql .

## 创建数据目录权限并修改权限为mysql用户
shell> mkdir -p /data/mysql/{data,tmp}
shell> chown -R mysql:mysql /data/mysql/

## 拷贝启动脚本至系统启动目录
shell> cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld

## 执行数据库初始化操作
shell> cd /usr/local/mysql
shell> bin/mysqld --initialize --user=mysql

## 启动MySQL
shell> systemctl enable mysqld
shell> systemctl start mysqld
shell> systemctl status mysqld
shell> systemctl disable mysqld
shell> cat /data/mysql/data/mysql-error.log |grep pass
shell> /usr/local/mysql/bin/mysql -S /data/mysql/data/mysql_3306.sock -p
## 输入查看到的随机密码

## 登录数据库后需先修改密码
mysql> set password='emhlbnhpbmcxMDA0';
mysql> exit;
```
### 2、配置环境变量
``` bash
shell> vim ~/.bash_profile
	    MYSQL_HOME=/usr/local/mysql
	    PATH=$PATH:$HOME/bin:$MYSQL_HOME/bin
shell> source ~/.bash_profile
shell> mysql -V
```
从库的搭建方式为在从库所在服务器重复步骤一和步骤二全部操作

## 二、复制配置

### 1、复制用户创建
``` bash
## 主库创建复制用户
mysql> grant replication slave on *.* to 'repl'@'%' identified by 'cmVwbGRiYQ==';
```
### 2、配置复制同步
``` bash
## 从库配置复制同步
mysql> change master to 
master_host='172.16.109.144',
master_user='repl',
master_password='cmVwbGRiYQ==',
master_port=3306,
master_auto_position=1;

## 从库启动复制
mysql> start slave;
mysql> show slave status\G;
mysql> show processlist;
```
## 三、备份配置

### 1、部署xtrabackup

主从都需要部署，文件上传操作：略
``` bash
## 解压percona-xtrabackup
cd /opt/for_gongbao_mysql/
tar xzvf percona-xtrabackup-2.4.9-Linux-x86_64.tar.gz

## 拷贝命令至系统可执行目录
cp percona-xtrabackup-2.4.9-Linux-x86_64/bin/* /usr/local/bin/
rm percona-xtrabackup-2.4.9-Linux-x86_64 -rf

## 拷贝qpress解压缩命令至系统可执行目录
cd /opt/for_gongbao_mysql/
cp qpress /usr/local/bin/
```
### 2、配置自动化备份脚本

脚本模板已上传，暂未配置

## 四、慢查询配置

### 1、日志轮换脚本配置

脚本模板已上传，暂未配置

### 2、日志格式化脚本配置

脚本模板已上传，暂未配置