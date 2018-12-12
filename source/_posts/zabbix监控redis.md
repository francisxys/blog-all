---
title: zabbix监控redis
copyright: true
date: 2018-12-06 14:42:14
abbrlink:
tags:
- zabbix
- redis
categories:
- monitor
- zabbix
---

zabbix监控redis实例有个不方便的地方是只能监控固定的6379端口，如果是非6379端口的话，需要修改模板，如果主机有多个redis实例的话，需要具有不同的redis模板，然后在管理监控，很是麻烦，为了解决这个问题，我使用lld（low level discovery）方式监控redis，只需要你在正则表达式里把需要监控的端口标上，就可以监控redis多实例。
## 一、agent端配置
agent端脚本，获取正在运行的redis实例端口
``` bash
# pwd
/etc/zabbix/scripts
# cat redis_low_discovery.sh 
#!/bin/bash
#Script_name redis_low_discovery.sh
redis() {
#            port=($(sudo netstat -tpln | awk -F "[ :]+" '/redis/ && /0.0.0.0/ {print $5}'))
            port=($(netstat -tpln | awk -F "[ :]+" '/redis/ && /0.0.0.0/ {print $5}'))
            printf '{\n'
            printf '\t"data":[\n'
               for key in ${!port[@]}
                   do
                       if [[ "${#port[@]}" -gt 1 && "${key}" -ne "$((${#port[@]}-1))" ]];then
              socket=`ps aux|grep ${port[${key}]}|grep -v grep|awk -F '=' '{print $10}'|cut -d ' ' -f 1`
                          printf '\t {\n'
                          printf "\t\t\t\"{#REDISPORT}\":\"${port[${key}]}\"},\n"
                     else [[ "${key}" -eq "((${#port[@]}-1))" ]]
              socket=`ps aux|grep ${port[${key}]}|grep -v grep|awk -F '=' '{print $10}'|cut -d ' ' -f 1`
                          printf '\t {\n'
                          printf "\t\t\t\"{#REDISPORT}\":\"${port[${key}]}\"}\n"
                       fi
               done
                          printf '\t ]\n'
                          printf '}\n'
}
$1
```
验证脚本是否正常
监控redis实例的json展示
``` bash
# ./redis_low_discovery.sh redis
{
	"data":[
          {
			"{#REDISPORT}":"6379"}
	 ]
	 {
			"{#REDISPORT}":"6380"}
	 ]
}
```
添加UserParameter
``` bash
# pwd
/etc/zabbix/zabbix_agentd.d
# cat userparameter_redis.conf 
UserParameter=zabbix_low_discovery[*],/bin/bash /etc/zabbix/scripts/redis_low_discovery.sh $1
UserParameter=redis_stats[*],(/bin/echo info; sleep 1) | telnet 127.0.0.1 $1   2>&1 |grep $2|cut -d : -f2
```
注：zabbix_agentd.conf配置文件中吧UnsafeUserParameters=1设置为1并打开注释即可，这里我的redis示例没有设置密码，如果有密码就加上-a password，telnet对空密码可以。有密码的话telnet换成$(which redis-cli)。
``` bash
UserParameter=zabbix_low_discovery[*],/bin/bash /etc/zabbix/scripts/redis/redis_low_discovery.sh $1
UserParameter=redis_stats[*],(/bin/echo info; sleep 1) | /data/redis/bin/redis-cli -h 172.16.109.138 -p $1 -a 5U7pp/pQLbdGLA   2>&1 |grep $2|cut -d : -f2
```
然后重启agent即可
把redis_low_discovery.sh文件存放到/etc/zabbix/scripts/目录下，然后给与755权限，并修改用户与组为zabbix，同时允许zabbix用户无密码运行
``` bash
echo "zabbix ALL=(root) NOPASSWD:/bin/netstat">>/etc/sudoers
```
关闭requiretty
``` bash
sed -i 's/^Defaults.*.requiretty/#Defaults    requiretty/' /etc/sudoers
```
## 二、server端配置
使用zabbix_get获取redis键值
``` bash
# zabbix_get -s 172.19.231.227 -p 10050 -k zabbix_low_discovery[redis]
{
"data":[
 {
"{#REDISPORT}":"6379"},
 {
"{#REDISPORT}":"6380"}
 ]
}
```
redis每秒更新时间
``` bash
# zabbix_get -s 172.19.231.227 -p 10050 -k redis_stats[6381,uptime_in_seconds]
8
```
## 三、web界面配置
导入模板以及主机连接模板，还需要设置正则等
模板在[此处下载](https://github.com/francisxys/zabbix/blob/master/Templates/Template%20Redis%20Auto%20Discovery.xml)，然后导入模板，并且关联对应的主机。
设置正则表达式
name：Redis regex
Result TRUE  = ^(6380|6381)$
正则表达式根据端口配置，可以增加
![Alt text](http://pjakaipln.bkt.clouddn.com/20181206001.png "添加正则表达式")
![Alt text](http://pjakaipln.bkt.clouddn.com/20181206002.png "测试正则表达式")
![Alt text](http://pjakaipln.bkt.clouddn.com/20181206003.png "测试正则表达式")
最后把主机连接到模板上即可，默认间隔时间1小时，方便测试我改成60s，数据收集后然后改过了即可
检查思路如下：
* 1：agent端可以使用脚步获取json化的信息
* 2：server端可以zabbix_get获取json化信息以及item的值
注：基于以上2步骤，按理说可以获取到相应的item值了。
* 3：打开agent端debug模式获取更多的日志信息，日志无问题，显示过程中没有显示json化的item
* 4：检查redis多实例模板中自动发现规则的键值与agent端中UnsafeUserParameters中定义键值不一样，修改与模板中对应的键值一样即可，重启agent即可
