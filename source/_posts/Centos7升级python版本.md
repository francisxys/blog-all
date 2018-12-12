---
title: Centos7升级python版本
copyright: true
date: 2018-12-10 13:55:44
abbrlink:
tags:
- centos7
- python
categories:
- code
- python
---

centos7.4中默认python默认安装版本为2.7.*。
```bash
# python
Python 2.7.5 (default, Oct 30 2018, 23:45:53) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> quit()
```
但是随着python3的成熟，生产需要需要使用python3版本进行支持，以下升级安装3.*版本python。
### 1、python3安装
python官网地址：https://www.python.org/
查看python的各个支持版本的最新版，本次开发需求为python3.6版本，官网最新3.6版本为3.6.6版本。
首先安装编译模块支持
yum -y install zlib*
其他在安装过程中提示需要的模块请自行安装。
下载、安装
```bash
wget https://www.python.org/ftp/python/3.6.6/Python-3.6.6.tgz
tar -zxvf Python-3.6.6.tgz
mkdir /data/python
cd Python-3.6.6
./configure --prefix=/data/python
make
make install
```
安装检查
```bash
# /data/python/bin/python3
Python 3.6.6 (default, Aug 10 2018, 16:26:38) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> quit()
# /data/python/bin/pip3 -V
pip 10.0.1 from /data/python/lib/python3.6/site-packages/pip (python 3.6)
```
可以看到安装的python版本为3.6.6，pip版本为10.0.1。
### 2、环境配置
为了符合代码习惯制作软连接
```bash
ln -s /data/python/bin/python3.6   /usr/bin/python3
ln -s /data/python/bin/pip3  /usr/bin/pip3
```
再次验证
```bash
# python3
Python 3.6.6 (default, Aug 10 2018, 16:26:38) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print("hello world")
hello world
>>> quit()
# pip3 -V
pip 10.0.1 from /data/python/lib/python3.6/site-packages/pip (python 3.6)
# whereis python3
python3: /usr/bin/python3
```
查看/usr/bin/下python版本
```bash
# ll /usr/bin/py*
-rwxr-xr-x. 1 root root   78 Aug  4  2017 /usr/bin/pydoc
lrwxrwxrwx  1 root root   24 Aug 10 16:55 /usr/bin/python -> /data/python/bin/python3
lrwxrwxrwx. 1 root root    9 Oct 15  2017 /usr/bin/python2 -> python2.7
-rwxr-xr-x. 1 root root 7136 Aug  4  2017 /usr/bin/python2.7
lrwxrwxrwx  1 root root   26 Aug 10 16:33 /usr/bin/python3 -> /data/python/bin/python3.6
```
可以看到同时存在两个版本，python2.7和python3.6。
不建议将python3直接软连接到/usr/bin/python，因为可能影响yum使用。
可以在代码中头文件中说明调用/usr/bin/python3。
### 3、单版本配置
同上述步骤，在配置软连接
```bash
ln -s /data/python/bin/python3.6   /usr/bin/python
```
查看/usr/bin/
```bash
ll /usr/bin/py*
-rwxr-xr-x. 1 root root   78 Aug  4  2017 /usr/bin/pydoc
lrwxrwxrwx  1 root root   30 Aug  6 18:24 /usr/bin/python -> /data/python/bin/python3
lrwxrwxrwx. 1 root root    9 Oct 15  2017 /usr/bin/python2 -> python2.7
-rwxr-xr-x. 1 root root 7136 Aug  4  2017 /usr/bin/python2.7
lrwxrwxrwx  1 root root   26 Aug  6 14:01 /usr/bin/python3 -> /data/python/bin/python3.6
lrwxrwxrwx. 1 root root    7 Oct 15  2017 /usr/bin/python.bak -> python2
```
此时，系统默认python版本即为python3.6.6。
需要注意的是，由于系统默认python版本更换，需要更改之前使用python2的应用配置，比如yum等。
