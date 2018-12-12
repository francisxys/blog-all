---
title: 阿里云ECS漏洞修复
copyright: true
date: 2018-12-05 15:24:19
abbrlink:
tags:
- 阿里云
- 软件漏洞修复
- glibc
- curl
- nss-pem
categories:
- OS
---

## 一、关于glibc的报警
### 1、问题确认
关于glibc的报警，具体如下所示：
![Alt text](http://pjakaipln.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181205144338.png "漏洞报警")
![Alt text](http://pjakaipln.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181205144351.png "漏洞报警")
此处报警我们可以根据漏洞编号进行查询，可以看到是因为glibc的版本太低，具有以下上图所示两个漏洞。
阿里云进行的报警的依据是根据探测ECS相关软件的版本，如下图所示：
![Alt text](http://pjakaipln.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181205154508.png "漏洞报警")
### 2、解决方案
上图可以看到我们需要升级的软件一共有5个，分别是glibc、glibc-common、glibc-headers、glibc-devel、nscd。
rpm包可从[此处](http://cbs.centos.org/kojifiles/packages/)下载，选择稳定版本即可。
### 3、升级脚本
将下载、安装过程脚本话
```bash
#!/bin/sh
#Francis

#切换到下载目录
cd /data/backup

# update glibc to 2.22 for CentOS 7
wget http://cbs.centos.org/kojifiles/packages/glibc/2.22.90/21.el7/x86_64/glibc-2.22.90-21.el7.x86_64.rpm
wget http://cbs.centos.org/kojifiles/packages/glibc/2.22.90/21.el7/x86_64/glibc-common-2.22.90-21.el7.x86_64.rpm
wget http://cbs.centos.org/kojifiles/packages/glibc/2.22.90/21.el7/x86_64/glibc-headers-2.22.90-21.el7.x86_64.rpm
wget http://cbs.centos.org/kojifiles/packages/glibc/2.22.90/21.el7/x86_64/glibc-devel-2.22.90-21.el7.x86_64.rpm
wget http://cbs.centos.org/kojifiles/packages/glibc/2.22.90/21.el7/x86_64/nscd-2.22.90-21.el7.x86_64.rpm

rpm -Uvh glibc-2.22.90-21.el7.x86_64.rpm \
         glibc-common-2.22.90-21.el7.x86_64.rpm \
         glibc-devel-2.22.90-21.el7.x86_64.rpm \
         glibc-headers-2.22.90-21.el7.x86_64.rpm \
		 nscd-2.22.90-21.el7.x86_64.rpm \
         --force --nodeps
```
## 二、关于curl的报警
### 1、问题确认
关于glibc的报警，具体如下所示：
![Alt text](http://pjakaipln.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181205144447.png "漏洞报警")
此处报警我们可以根据漏洞编号进行查询，可以看到是因为curl的版本太低，具有以下上图所示四个漏洞。
阿里云进行的报警的依据是根据探测ECS相关软件的版本，如下图所示：
![Alt text](http://pjakaipln.bkt.clouddn.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20181205154512.png "漏洞报警")
### 2、解决方案
上图可以看到curl版本7.12到7.58都有漏洞，所以我们选择安装7.60以后的版本
#### 1、编译安装
```bash
wget https://curl.haxx.se/download/curl-7.62.0.tar.gz
tar -zxvf curl-7.62.0.tar.gz 
cd curl-7.62.0
./configure --prefix=/usr
make
make install
```
查看curl版本
```bash
# curl --version
curl 7.62.0 (x86_64-pc-linux-gnu) libcurl/7.47.1 NSS/3.21.3 Basic ECC zlib/1.2.7 libidn/1.28 libssh2/1.4.3
Release-Date: 2018-10-31
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz UnixSockets 
# curl-config --version
libcurl 7.62.0
```
#### 2、yum安装
查看已安装版本
``` bash
# curl --version
curl 7.29.0 (x86_64-redhat-linux-gnu) libcurl/7.29.0 NSS/3.34 zlib/1.2.7 libidn/1.28 libssh2/1.4.3
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smtp smtps telnet tftp 
Features: AsynchDNS GSS-Negotiate IDN IPv6 Largefile NTLM NTLM_WB SSL libz unix-sockets 
```
安装repo
``` bash
rpm -Uvh  http://www.city-fan.org/ftp/contrib/yum-repo/rhel6/x86_64/city-fan.org-release-2-1.rhel6.noarch.rpm
```
查看该 repo 包含的 curl 版本
``` bash
# yum --showduplicates list curl --disablerepo="*" --enablerepo="city*"
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * city-fan.org: cityfan.mirror.digitalpacific.com.au
 * city-fan.org-debuginfo: cityfan.mirror.digitalpacific.com.au
 * city-fan.org-source: cityfan.mirror.digitalpacific.com.au
Installed Packages
curl.x86_64                                                                          7.29.0-51.el7                                                                                 @base       
Available Packages
curl.x86_64                                                                          7.62.0-1.7.cf.rhel7                                                                           city-fan.org
```
修改该repo的enable为1
``` bash
vi /etc/yum.repos.d/city-fan.org.repo

[city-fan.org]

name=city-fan.org repository for Red Hat Enterprise Linux (and clones) $releasever ($basearch)

#baseurl=http://mirror.city-fan.org/ftp/contrib/yum-repo/rhel$releasever/$basearch

mirrorlist=http://mirror.city-fan.org/ftp/contrib/yum-repo/mirrorlist-rhel$releasever

enabled=1

gpgcheck=1

gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-city-fan.org
```
升级最新的curl
``` bash
yum upgrade curl -y
```
查看版本
``` bash
# curl -V
curl 7.62.0 (x86_64-redhat-linux-gnu) libcurl/7.62.0 NSS/3.36 zlib/1.2.7 libpsl/0.7.0 (+libicu/50.1.2) libssh2/1.8.0 nghttp2/1.31.1
Release-Date: 2018-10-31
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz HTTP2 UnixSockets HTTPS-proxy PSL Metalink 
```
升级完成