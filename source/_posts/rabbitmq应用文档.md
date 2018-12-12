---
title: rabbitmq应用文档
copyright: true
date: 2018-12-10 16:02:07
abbrlink:
tags:
- erlang
- rabbitmq
- 集群
categories:
- OS
- service
---

## 一、Rabbitmq简介
### 1、Rabbitmq介绍
RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端，如：Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、STOMP等，支持AJAX。用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。
AMQP，即Advanced message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。
AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。
### 2、Rabbitmq系统概念
* RabbitMQ Server		也叫broker server，是一种传输服务，负责维护一条从Producer到consumer的路线，保证数据能够按照指定的方式进行传输。
* Producer	数据的发送方。
* Consumer	数据的接收方。
* Exchanges 	接收消息，转发消息到绑定的队列。主要使用3种类型：direct， topic， fanout。
* Queue 	RabbitMQ内部存储消息的对象。相同属性的queue可以重复定义，但只有第一次定义的有效。
* Bindings 	绑定Exchanges和Queue之间的路由。
* Connection	就是一个TCP的连接。Producer和consumer都是通过TCP连接到RabbitMQ Server的。
* Channel		虚拟连接。它建立在上述的TCP连接中。数据流动都是在Channel中进行的。也就是说，一般情况是程序起始建立TCP连接，第二步就是建立这个Channel。

### 3、AMQP协议简介
AMQP在一致性客户端和消息中间件(也称为"brokers")之间创建了全功能的互操作．
为了完全实现消息中间件的互操作性，需要充分定义网络协议和消息代理服务的功能语义。因此，AMQP通过如下来定义了网络协议(AMQP是协议！)和服务端服务：
1、一套确定的消息交换功能，也就是“高级消息交换协议模型”。AMQP模型包括一套用于路由和存储消息的功能模块，以及一套在这些模块之间交换消息的规则。
2、	一个网络线级协议（数据传输格式），AMQP促使客户端可使用AMQ模型来与服务器交互。
可以只实现AMQP协议规范中的的部分语义，但是我们相信这些明确的语义有助于理解这个协议。
我们需要明确定义服务器语义，因为所有服务器实现都应该与这些语义保持一致性，否则就无法进行互操作.  因此AMQ 模型定义了一系列模块化组件和标准规则来进行协作. 有三种类型的组件可以连接服务器处理链来创建预期的功能:
1、"交换器(exchange)" ：接收来自发布者应用程序的消息，并基于任意条件(通常是消息属性和内容）将这些消息路由到消息队列(message queues).
2、"消息队列(message queue)"：存储消息直到它们可以被消费客户端应用程序(或多线程应用程序)安全处理。
3、"绑定(binding)":定义了消息队列与交换器之间的关系，并提供了消息路由条件．
使用这些模型我们可以很容易地模拟经典的存储转发队列和面向消息中间件的主题订阅概念. 我们还可以表示更为复杂的概念，例如：基于内容的路由，工作负载分配和按需消息队列。
大致上讲， AMQP 服务器类似与邮件服务器, 每个交换器都扮演了消息传送代理,每个消息队列都作为邮箱，而绑定则定义了每个传送代理中的路由表.发布者发送消息给独立的传送代理,然后传送代理再路由消息到邮箱中.消费者从邮箱中收取消息. 相比较而言，在AMQP之前的许多中间件系统中，发布者直接发送消息到独立收件箱(在存储转发队列的情况下),或者发布到邮件列表中 (在主题订阅的情况下)。
区别就在于用户可以控制消息队列和交换器之间的绑定规则，这可以做很多有趣的事情，比如定义一条规则：“将所有包含这样消息头的消息都复制一份再发送到消息队列中”。
AMQ模型是基于下面的需求来驱动设计的：
1、支持与主要消息产品相媲美的语义。.
2、 提供与主要消息产品相媲美的性能水平.
3、允许通过应用程序使用服务器特定语义来编程.
4、灵活性，可扩展性，简单性
## 二、安装Rabbitmq
### 1、安装好系统运行
```bash
yum update -y
reboot  #一般情况不用重启
```
### 2、安装依赖文件
```bash
yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget
#部分依赖可能在安装其他服务时已安装
```
### 3、安装erlang语言环境
官网地址：http://www.erlang.org/downloads，由于环境支持问题，建议下载最新版本
```bash
wget http://erlang.org/download/otp_src_20.3.tar.gz
tar -zxvf otp_src_20.3.tar.gz 
cd otp_src_20.3
./configure --prefix=/data/erlang
make
make install
```
配置erlang环境变量
```bash
vi /etc/profile  #在底部添加以下内容 

#set erlang environment
ERL_HOME=/data/erlang
PATH=$ERL_HOME/bin:$PATH
export ERL_HOME PATH

source /etc/profile  #生效
```
在控制台输入命令erl
如果进入erlang的shell则证明安装成功，退出即可。
### 4、安装rabbitmq
官网地址：http://www.rabbitmq.com/install-generic-unix.html，下载最新稳定版本
```bash
wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.4/rabbitmq-server-generic-unix-3.7.4.tar.xz
xz -d rabbitmq-server-generic-unix-3.7.4.tar.xz 
tar -xvf rabbitmq-server-generic-unix-3.7.4.tar 
mv rabbitmq_server-3.7.4 /data/rabbitmq
```
配置rabbitmq环境变量
```bash
vi /etc/profile  #在底部添加以下内容 

#set rabbitmq environment
export PATH=$PATH:/data/rabbitmq/sbin

source /etc/profile  #生效
```
启动服务
```bash
rabbitmq-server -detached   #启动rabbitmq，-detached代表后台守护进程方式启动
```
出现以下界面
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210019.png "Francis'Blog") 
此时rabbit-servery已经启动
官网解释：http://www.rabbitmq.com/rabbitmq-server.8.html
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210020.png "Francis'Blog")
但是由于没有写入PID file文件，可能导致服务停止
查看状态
```bash
rabbitmqctl status
```
如果显示如下截图说明安装成功
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210021.png "Francis'Blog") 
其他相关命令
* 启动服务：rabbitmq-server -detached 或者/data/rabbitmq/sbin/rabbitmq-server -detached 
* 查看状态：rabbitmqctl status 或者/usr/local/rabbitmq/sbin/rabbitmqctl status  
* 关闭服务：rabbitmqctl stop 或者/usr/local/rabbitmq/sbin/rabbitmqctl stop  
* 列出角色：rabbitmqctl list_users 或者/usr/local/rabbitmq/sbin/rabbitmqctl list_users

### 5、配置网页插件
首先创建目录，否则可能报错
```bash
mkdir /etc/rabbitmq
```
启用web管理插件
```bash
rabbitmq-plugins enable rabbitmq_management
```
### 6、配置防火墙
配置linux端口15672网页管理5672AMQP端口
```bash
firewall-cmd --permanent --add-port=15672/tcp
firewall-cmd --permanent --add-port=5672/tcp
systemctl restart firewalld.service
```
在浏览器输入http://ip:15672，可以看到RabbitMQ的WEB管理页面，如下
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210022.png "Francis'Blog")
### 7、配置访问账号密码和权限
默认网页是不允许访问的，需要增加一个用户修改一下权限
```bash
rabbitmqctl add_user action action #添加用户，后面两个参数分别是用户名和密码
rabbitmqctl set_permissions -p / action ".*" ".*" ".*"  #添加权限
rabbitmqctl set_user_tags action administrator  #修改用户角色
```
查看角色并确认
```bash
# rabbitmqctl list_users
Listing users ...
action	[administrator]
```
然后就可以远程访问了，然后可直接配置用户权限等信息。登录：http://ip:15672 登录之后在admin里面把guest删除。
### 8、Rabbitmq配置
RabbitMQ 提供了三种方式来定制服务器:
* 环境变量：
定义端口，文件位置和名称(接受shell输入,或者在环境配置文件（rabbitmq-env.conf）中设置)
* 配置文件：
为服务器组件设置权限,限制和集群，也可以定义插件设置.
* 运行时参数和策略：
可在运行时进行修改集群设置

#### 1）配置文件
rabbitmq.config 文件
rabbitmq.config配置文件允许配置RabbitMQ 核心程序， Erlang 服务和RabbitMQ 插件. 它是标准的Erlang 配置文件, 文档位于Erlang Config Man Page.
最小化的样例配置文件如下:
```bash
[     {rabbit, [{tcp_listeners, [5673]}]}   ].
```
这个例子中将会修改RabbitMQ监听AMQP 0-9-1 客户端连接端口，5672修改为5673.
配置文件不同于环境配置文件rabbitmq-env.conf
这些文件的位置分布特定的. 默认情况下，这些文件是没有创建的,但每个平台上期望的位置如下：
```bash
Generic UNIX - $RABBITMQ_HOME/etc/rabbitmq/
Debian - /etc/rabbitmq/
RPM - /etc/rabbitmq/
Mac OS X (Homebrew) - ${install_prefix}/etc/rabbitmq/, the Homebrew prefix is usually/usr/local
Windows - %APPDATA%\RabbitMQ\
```
如果rabbitmq-env.conf不存在, 可在默认位置中手动创建.。它不能用于Windows系统.
如果rabbitmq.config不存在，可以手动创建它. 如果你修改了位置，可设置RABBITMQ_CONFIG_FILE 环境变量来指定. Erlang 运行时会自动在此变量值后添加.config扩展名，重启服务器后生效。
Windows 服务用户在删除配置文件后，需要重新安装服务。
#### 2）rabbitmq.config中的变量配置
大部分的RabbitMQ用户都会不会修改这些值，有些是相当模糊的。为了完整性，他们都在这里列出。

| Key                                   | Documentation                                                |
| ------------------------------------- | ------------------------------------------------------------ |
| tcp_listeners                         | 用于监听 AMQP连接的端口列表(无SSL). 可以包含整数 (即"监听所有接口")或者元组如 {"127.0.0.1", 5672} 用于监听一个或多个接口.   Default: [5672] |
| num_tcp_acceptors                     | 接受TCP侦听器连接的Erlang进程数。   Default: 10              |
| handshake_timeout                     | AMQP 0-8/0-9/0-9-1 handshake (在 socket 连接和SSL 握手之后）的最大时间, 毫秒为单位.   Default: 10000 |
| ssl_listeners                         | 如上所述，用于SSL连接。   Default: []                        |
| num_ssl_acceptors                     | 接受SSL侦听器连接的Erlang进程数。   Default: 1               |
| ssl_options                           | SSL配置.参考**SSL   documentation**.   Default: []           |
| ssl_handshake_timeout                 | SSL handshake超时时间,毫秒为单位.   Default: 5000            |
| vm_memory_high_watermark              | 流程控制触发的内存阀值．相看**memory-based   flow control** 文档.   Default: 0.4 |
| vm_memory_high_watermark_paging_ratio | 高水位限制的分数，当达到阀值时，队列中消息消息会转移到磁盘上以释放内存. 参考**memory-based   flow control** 文档.   Default: 0.5 |
| disk_free_limit                       | RabbitMQ存储数据分区的可用磁盘空间限制．当可用空间值低于阀值时，流程控制将被触发. 此值可根据RAM的总大小来相对设置 (如.{mem_relative, 1.0}). 此值也可以设为整数(单位为bytes)或者使用数字单位(如．"50MB"). 默认情况下，可用磁盘空间必须超过50MB. 参考 **Disk   Alarms** 文档.   Default: 50000000 |
| log_levels                            | 控制日志的粒度.其值是日志事件类别(category)和日志级别(level)成对的列表．   level 可以是 'none' (不记录日志事件),   'error' (只记录错误),   'warning' (只记录错误和警告), 'info'   (记录错误，警告和信息), or   'debug' (记录错误，警告，信息以及调试信息).   目前定义了４种日志类别. 它们是：   channel -针对所有与AMQP   channels相关的事件   connection - 针对所有与网络连接相关的事件   federation - 针对所有与**federation**相关的事件   mirroring -针对所有与 **mirrored   queues**相关的事件   Default: [{connection, info}] |
| frame_max                             | 与客户端协商的允许最大frame大小. 设置为０表示无限制，但在某些QPid客户端会引发bug. 设置较大的值可以提高吞吐量;设置一个较小的值可能会提高延迟.           Default: 131072 |
| channel_max                           | 与客户端协商的允许最大chanel大小. 设置为０表示无限制．该数值越大，则broker使用的内存就越高．   Default: 0 |
| channel_operation_timeout             | Channel 操作超时时间(毫秒为单位） (内部使用，因为消息协议的区别和限制，不暴露给客户端).   Default: 5000 |
| heartbeat                             | 表示心跳延迟(单位为秒) ，服务器将在connection.tune frame中发送.如果设置为 0, 心跳将被禁用. 客户端可以不用遵循服务器的建议, 查看 **AMQP reference** 来了解详情. 禁用心跳可以在有大量连接的场景中提高性能，但可能会造成关闭了非活动连接的网络设备上的连接落下．   Default: 60 (3.5.5之前的版本是580) |
| default_vhost                         | 当RabbitMQ从头开始创建数据库时创建的虚拟主机. amq.rabbitmq.log交换器会存在于这个虚拟主机中.   Default: <<"/">> |
| default_user                          | RabbitMQ从头开始创建数据库时，创建的用户名.   Default: <<"guest">> |
| default_pass                          | 默认用户的密码.   Default: <<"guest">>                       |
| default_user_tags                     | 默认用户的Tags.   Default: [administrator]                   |
| default_permissions                   | 创建用户时分配给它的默认**Permissions** .   Default: [<<".*">>,   <<".*">>, <<".*">>] |
| loopback_users                        | 只能通过环回接口(即localhost)连接broker的用户列表   如果你希望默认的guest用户能远程连接,你必须将其修改为[].   Default: [<<"guest">>] |
| cluster_nodes                         | 当节点第一次启动的时候，设置此选项会导致集群动作自动发生. 元组的第一个元素是其它节点想与其建立集群的节点. 第二个元素是节点的类型，要么是disc,要么是ram   Default: {[], disc} |
| server_properties                     | 连接时向客户端声明的键值对列表   Default: []                 |
| collect_statistics                    | 统计收集模式。主要与管理插件相关。选项：   none (不发出统计事件)   coarse (发出每个队列 /每个通道 /每个连接的统计事件)   fine (也发出每个消息统计事件)   你自已可不用修改此选项.   Default: none |
| collect_statistics_interval           | 统计收集时间间隔(毫秒为单位)． 主要针对于 **management plugin**.   Default: 5000 |
| auth_mechanisms                       | 提供给客户端的**SASL   authentication mechanisms**.   Default: ['PLAIN', 'AMQPLAIN'] |
| auth_backends                         | 用于 **authentication   / authorisation backends** 的列表. 此列表可包含模块的名称(在模块相同的情况下，将同时用于认证来授权)或像{ModN, ModZ}这样的元组，在这里ModN将用于认证，ModZ将用于授权.   在２元组的情况中, ModZ可由列表代替,列表中的所有元素必须通过每个授权的确认，如{ModN, [ModZ1, ModZ2]}. 这就允许授权插件进行组合提供额外的安全约束.   除rabbit_auth_backend_internal外，其它数据库可以通常 **plugins**来使用.   Default: [rabbit_auth_backend_internal] |
| reverse_dns_lookups                   | 设置为true,可让客户端在连接时让RabbitMQ 执行一个反向DNS查找, 然后通过 rabbitmqctl 和 管理插件来展现信息.           Default: false |
| delegate_count                        | 内部集群通信中，委派进程的数目. 在一个有非常多核的机器(集群的一部分)上,你可以增加此值.   Default: 16 |
| trace_vhosts                          | **tracer**内部使用. 你不应该修改.   Default: []              |
| tcp_listen_options                    | 默认socket选项. 你可能不想修改这个选项.   Default:   [{backlog,       128},          {nodelay,       true},          {exit_on_close, false}] |
| hipe_compile                          | 将此选项设置为true,将会使用HiPE预编译部分RabbitMQ,Erlang的即时编译器.    这可以增加服务器吞吐量，但会增加服务器的启动时间．    你可以看到花费几分钟延迟启动的成本，就可以带来20-50% 更好性能.这些数字与高度依赖于工作负载和硬件．   HiPE 支持可能没有编译进你的Erlang安装中.如果没有的话，启用这个选项,并启动RabbitMQ时，会看到警告消息． 例如, Debian   / Ubuntu 用户需要安装erlang-base-hipe 包.   HiPE并非在所有平台上都可用, 尤其是Windows.   在 Erlang/OTP 1７.５版本之前，HiPE有明显的问题 . 对于HiPE,使用最新的OTP版本是高度推荐的．   Default: false |
| cluster_partition_handling            | 如何处理网络分区.可用模式有:   ignore   pause_minority   {pause_if_all_down, [nodes], ignore autoheal}where [nodes] is a list of node names    (ex: ['rabbit@node1', 'rabbit@node2'])   autoheal   参考**documentation on partitions** 来了解更多信息   Default: ignore |
| cluster_keepalive_interval            | 节点向其它节点发送存活消息和频率(毫秒). 注意，这与 **net_ticktime**是不同的; 丢失存活消息不会引起节点掉线   Default: 10000 |
| queue_index_embed_msgs_below          | 消息大小在此之下的会直接内嵌在队列索引中. 在修改此值时，建议你先阅读　 **persister   tuning** 文档.   Default: 4096 |
| msg_store_index_module                | 队列索引的实现模块. 在修改此值时，建议你先阅读　 **persister   tuning** 文档.   Default: rabbit_msg_store_ets_index |
| backing_queue_module                  | 队列内容的实现模块. 你可能不想修改此值．   Default: rabbit_variable_queue |
| msg_store_file_size_limit             | Tunable value for the persister. 你几乎肯定不应该改变此值。   Default: 16777216 |
| mnesia_table_loading_timeout          | 在集群中等待使用Mnesia表可用的超时时间。   Default: 30000    |
| queue_index_max_ journal_entries      | Tunable value for the persister. 你几乎肯定不应该改变此值。   Default: 65536 |
| queue_master_locator                  | Queue master 位置策略. 可用策略有:   <<"min-masters">>   <<"client-local">>   <<"random">>   查看**documentation on queue master location** 来了解更多信息．   Default: <<"client-local">> |
此外，许多插件也可以在配置文件中配置, 其名称是rabbitmq_plugin的形式. 我们的维护的插件被记录在以下位置：

```bash
rabbitmq_management
rabbitmq_management_agent
rabbitmq_mochiweb
rabbitmq_stomp
rabbitmq_shovel
rabbitmq_auth_backend_ldap
```
#### 3）文件位置

| 名称                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| RABBITMQ_BASE                 | 此基础目录包含了RabbitMQ   server的数据库，日志文件的子目录. 另外，也可以独立设置RABBITMQ_MNESIA_BASE 和 RABBITMQ_LOG_BASE 目录. |
| RABBITMQ_CONFIG_FILE          | 用于配置文件的路径，无.config扩展名. 如果 **configuration file** 存在，服务器将使用它来配置RabbitMQ组件. 参考 **Configuration   guide** 来了解更多信息. |
| RABBITMQ_MNESIA_BASE          | 包含RabbitMQ 服务器Mnesia数据库文件子目录的基本目录,除非明确设置了RABBITMQ_MNESIA_DIR目录，否则每个节点都应该配置一个. (除了Mnesia文件，这个位置还包含消息存储和索引文件以及模式和集群的细节．） |
| RABBITMQ_MNESIA_DIR           | RabbitMQ节点Mnesia数据库文件安放的目录. (除了Mnesia文件，这个位置还包含消息存储和索引文件以及模式和集群的细节.) |
| RABBITMQ_LOG_BASE             | 用于包含RabbitMQ 服务器日志文件的基本目录, 除非明确设置了RABBITMQ_LOGS 或 RABBITMQ_SASL_LOGS. |
| RABBITMQ_LOGS                 | RabbitMQ 服务器的Erlang日志文件路径.在Window上不能覆盖此变量． |
| RABBITMQ_SASL_LOGS            | RabbitMQ服务器的Erlang SASL (System Application   Support Libraries)日志文件路径. 在Window上不能覆盖此变量． |
| RABBITMQ_PLUGINS_DIR          | 用于查找**插件**的**目录** .                                 |
| RABBITMQ_PLUGINS_EXPAND_DIR   | 用于在启动服务器时扩展启用插件的工作目录。                   |
| RABBITMQ_ENABLED_PLUGINS_FILE | 此文件记录了显式启用的插件。                                 |
| RABBITMQ_PID_FILE             | 此文件中包含了rabbitmqctl所等待进程ID的信息．                |
Unix默认位置
在下面的表格中，${install_prefix}表示某个路径. Homebrew 安装时使用*installation-prefix* (Homebrew Cellar) . 默认是/usr/local.
Deb / RPM 包安装使用空${install_prefix}.

| **Name**                      | **Location**                                            |
| ----------------------------- | ------------------------------------------------------- |
| RABBITMQ_BASE                 | (Not used)                                              |
| RABBITMQ_CONFIG_FILE          | ${install_prefix}/etc/rabbitmq/rabbitmq                 |
| RABBITMQ_MNESIA_BASE          | ${install_prefix}/var/lib/rabbitmq/mnesia               |
| RABBITMQ_MNESIA_DIR           | $RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME                |
| RABBITMQ_LOG_BASE             | ${install_prefix}/var/log/rabbitmq                      |
| RABBITMQ_LOGS                 | $RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME.log               |
| RABBITMQ_SASL_LOGS            | $RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME-sasl.log          |
| RABBITMQ_PLUGINS_DIR          | $RABBITMQ_HOME/plugins                                  |
| RABBITMQ_PLUGINS_EXPAND_DIR   | $RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME-plugins-expand |
| RABBITMQ_ENABLED_PLUGINS_FILE | ${install_prefix}/etc/rabbitmq/enabled_plugins          |
| RABBITMQ_PID_FILE             | $RABBITMQ_MNESIA_DIR.pid                                |
Windows默认位置

| Name                          | Location                                                  |
| ----------------------------- | --------------------------------------------------------- |
| RABBITMQ_BASE                 | %APPDATA%\RabbitMQ                                        |
| RABBITMQ_CONFIG_FILE          | %RABBITMQ_BASE%\rabbitmq                                  |
| RABBITMQ_MNESIA_BASE          | %RABBITMQ_BASE%\db                                        |
| RABBITMQ_MNESIA_DIR           | %RABBITMQ_MNESIA_BASE%\%RABBITMQ_NODENAME%                |
| RABBITMQ_LOG_BASE             | %RABBITMQ_BASE%\log                                       |
| RABBITMQ_LOGS                 | %RABBITMQ_LOG_BASE%\%RABBITMQ_NODENAME%.log               |
| RABBITMQ_SASL_LOGS            | %RABBITMQ_LOG_BASE%\%RABBITMQ_NODENAME%-sasl.log          |
| RABBITMQ_PLUGINS_DIR          | *Installation-directory*/plugins                          |
| RABBITMQ_PLUGINS_EXPAND_DIR   | %RABBITMQ_MNESIA_BASE%\%RABBITMQ_NODENAME%-plugins-expand |
| RABBITMQ_ENABLED_PLUGINS_FILE | %RABBITMQ_BASE%\enabled_plugins                           |
| RABBITMQ_PID_FILE             | (Not currently supported)                                 |
通用Unix默认位置（即本次测试使用Rabbitmq版本）
当解压Generic Unix tar文件并运行时，由于默认获得到位置，不需要进行. 在下面的表格中，$RABBITMQ_HOME指的是rabbitmq_server-3.7.4解压后的目录。

| Name                          | **Location**                                            |
| ----------------------------- | ------------------------------------------------------- |
| RABBITMQ_BASE                 | (Not used)                                              |
| RABBITMQ_CONFIG_FILE          | $RABBITMQ_HOME/etc/rabbitmq/rabbitmq                    |
| RABBITMQ_MNESIA_BASE          | $RABBITMQ_HOME/var/lib/rabbitmq/mnesia                  |
| RABBITMQ_MNESIA_DIR           | $RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME                |
| RABBITMQ_LOG_BASE             | $RABBITMQ_HOME/var/log/rabbitmq                         |
| RABBITMQ_LOGS                 | $RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME.log               |
| RABBITMQ_SASL_LOGS            | $RABBITMQ_LOG_BASE/$RABBITMQ_NODENAME-sasl.log          |
| RABBITMQ_PLUGINS_DIR          | $RABBITMQ_HOME/plugins                                  |
| RABBITMQ_PLUGINS_EXPAND_DIR   | $RABBITMQ_MNESIA_BASE/$RABBITMQ_NODENAME-plugins-expand |
| RABBITMQ_ENABLED_PLUGINS_FILE | $RABBITMQ_HOME/etc/rabbitmq/enabled_plugins             |
| RABBITMQ_PID_FILE             | $RABBITMQ_MNESIA_DIR.pid                                |
#### 4）持久化配置
RabbitMQ持久层的目的是为了得到好的结果，在大多数情况下没有配置。然而，一些配置有时是有用的。
首先，先讲一下背景: 持久化和短暂消息都可以写入磁盘。持久化消息一旦到达队列，就会写入磁盘,而短暂消息只在内存压力较大被赶出内存时才会写入磁盘。持久化消息在内存紧张释放内存时，依然也会存在内存中． 持久层指的是存储这两种类型消息到磁盘的机制。队列是指无镜像队列或master队列或slave队列。 队列镜像会发生以上的持久化。
持久层有两个组件: 队列索引和消息存储.队列索引负责维护消息在队列的位置，以及是否被投递，是否应答的信息. 因此，每个队列都有一个队列索引。
消息存储是消息的key-value存储, 由服务器中的所有队列共享.消息(消息体, 消息属性或消息头)可直接存储于队列索引，也可以写到消息存储中.在技术上有两个消息存储（一个暂时的和一个持久的消息），但他们通常一起被被认为是“消息存储”。
内存成本
在内存压力下，持久层试图尽可能多地写入磁盘，并尽可能的从内存中删除。然而有一些事情必须留在内存中：
每个队列都会为每个未应答消息维护一些元数据．如果它的目的地是消息存储，则消息本身可以从内存中删除。
消息存储需要索引. 默认消息存储索引对于存储中的每个消息会使用少量内存。
队列索引中的消息
将消息写入队列索引有优点也有缺点。
优点:
消息可在一个操作中(而不是两个）写入磁盘; 对于微小的消息，这可以是一个实质性的增益。
写入队列索引的消息不需要消息存储索引中的条目，因此当页出(paged out)时，不需要花费内存成本。
缺点:
队列索引在内存中保有固定数量的记录块;如果写入队列索引中的消息不是小消息，那么内存占用也是巨大的。
如果一个消息通过一个交换路由到多个队列，则消息将需要写入多个队列索引。如果这样的消息被写入消息存储区，则只有一个副本需要被写入。
目的地是队列索引的未应答消息总会保存在内存中。
将小消息存储在队列索引中目的是优化，所有其它消息将会写入消息存储.这可以配置项queue_index_embed_msgs_below来配置.默认情况下，序列后大小小于4096字节 (包括属性和头)会存储在队列索引中。
当从磁盘中读取消息时，每个队列索引至少需要在内存中保留一个段文件(segment file). 段文件中包含了16,384个消息. 因此要谨慎如果增加queue_index_embed_msgs_below；小的增加会导致大量的内存使用。
无意中有限的持久性能(Accidentally limited persister performance）
持久化有可能表现不佳，因为持久化受限于文件句柄的数目或与它工作的异步线程.在这两种情况下，当您有大量需要同时访问磁盘的队列时，会发生这样的情况。.
太少的文件句柄
RabbitMQ 服务器通常受限于它能打开的文件句柄数量(在Unix上，无论如何). 每个运行的网络连接都需要一个文件句柄, 其余的可用于队列使用。如果磁盘访问队列比考虑到网络连接后文件句柄更多，那么磁盘访问队列将与文件句柄一起共享; 每个都会在它返回交给另一个队列之前，都会使用文件句柄一段时间。
当有太多磁盘访问队列时，这可以防止服务器崩溃,但代价是昂贵的. 管理插件可以显示集群中每个节点的统计I/O统计信息，如读，写，查找的速率．同时它也会显示重新开始(reopens)的速率- 文件句柄通过这种方式来回收利用. 一个有太少文件句柄繁忙的服务器每秒可能会做几百次reopens - 在这种情况下，如果增加文件句柄，就有可能提高性能。
太少的异步线程
Erlang 虚拟机创建异步线程池来处理长时间运行的文件I/O操作. 这些线程池是所有队列所共享的.每个活跃的文件I/O操作都会使用一个异步线程. 太少的异步线程可以因此伤害性能。
注意，异步线程的情况并不完全类似与文件句柄的情况. 如果一个队列按顺序来执行一定数量的I/O操作，假设它持有一个文件句柄来所理所有操作，其性能是最好的，否则，我们会占用ＣＰＵ来做更多的刷新，查找 操作. 然而,队列不能从持有一个异步线程执行一系列的操作中获益(事实上也做不到)。
因此理论上应该要有足够的文件句柄来处理所有队列上的I/O流操作, 并且要有足够的线程来处理并发的 (simultaneous )的I/O操作。
由异步线程缺乏造成的性能问题，不是太明显. (一般情况下都不太可能，可首先检查其它地方!) 。
太少异步线程的典型症状是，当服务器忙于持久化时，在很短的时间内，每秒 I/O操作的数目将会下降到０(管理插件可报告) ,报告的每个 I/O操作的时间将会增加。
Erlang虚拟主机的异步线程数目可通过+A 参数进行配置，这里有描述, 通常情况下，也可以通过环境变量RABBITMQ_SERVER_ERL_ARGS来配置. 默认值是 +A 30. 在修改之前，多进行几次尝试总是好主意。
## 三、Rabbitmq集群
RabbitMQ是用erlang开发的，集群非常方便，因为erlang天生就是一门分布式语言,但其本身并不支持负载均衡。
Rabbit模式大概分为以下三种：单一模式、普通模式、镜像模式
单一模式：最简单的情况，非集群模式。
普通模式：默认的集群模式。
对于Queue来说，消息实体只存在于其中一个节点，A、B两个节点仅有相同的元数据，即队列结构。
当消息进入A节点的Queue中后，consumer从B节点拉取时，RabbitMQ会临时在A、B间进行消息传输，把A中的消息实体取出并经过B发送给consumer。
所以consumer应尽量连接每一个节点，从中取消息。即对于同一个逻辑队列，要在多个节点建立物理Queue。否则无论consumer连A或B，出口总在A，会产生瓶颈。
该模式存在一个问题就是当A节点故障后，B节点无法取到A节点中还未消费的消息实体。如果做了消息持久化，那么得等A节点恢复，然后才可被消费；如果没有持久化的话，就会很容易发生故障。
镜像模式：把需要的队列做成镜像队列，存在于多个节点，属于RabbitMQ的HA方案。
该模式解决了上述问题，其实质和普通模式不同之处在于，消息实体会主动在镜像节点间同步，而不是在consumer取数据时临时拉取。
该模式带来的副作用也很明显，除了降低系统性能外，如果镜像队列数量过多，加之大量的消息进入，集群内部的网络带宽将会被这种同步通讯大大消耗掉，所以在对可靠性要求较高的场合中适用。
### 1、集群中的基本概念
RabbitMQ的集群节点包括内存节点、磁盘节点。顾名思义内存节点就是将所有数据放在内存，磁盘节点将数据放在磁盘。不过，如前文所述，如果在投递消息时，打开了消息的持久化，那么即使是内存节点，数据还是安全的放在磁盘。
一个rabbitmq集群中可以共享 user，vhost，queue，exchange等，所有的数据和状态都是必须在所有节点上复制的，一个例外是，那些当前只属于创建它的节点的消息队列，尽管它们可见且可被所有节点读取。rabbitmq节点可以动态的加入到集群中，一个节点它可以加入到集群中，也可以从集群环集群会进行一个基本的负载均衡。
集群中有两种节点：
1 内存节点：只保存状态到内存（一个例外的情况是：持久的queue的持久内容将被保存到disk）
2 磁盘节点：保存状态到内存和磁盘。
内存节点虽然不写入磁盘，但是它执行比磁盘节点要好。集群中，只需要一个磁盘节点来保存状态 就足够了
如果集群中只有内存节点，那么不能停止它们，否则所有的状态，消息等都会丢失。
### 2、集群模式配置
#### 1）配置hosts
在2台节点服务器中，分别修改/etc/hosts文件
```bash
10.186.21.84	10-186-21-84
10.186.21.85	10-186-21-85
```
还有hostname文件也要正确，分别是10-186-21-84、10-186-21-85，如果修改hostname建议安装rabbitmq前修改。
请注意RabbitMQ集群节点必须在同一个网段里，如果是跨广域网效果就差。
#### 2）设置每个节点Cookie
Rabbitmq的集群是依赖于erlang的集群来工作的，所以必须先构建起erlang的集群环境。Erlang的集群中各节点是通过一个magic cookie来实现的，这个cookie存放在  /var/lib/rabbitmq/.erlang.cookie 中，文件是400的权限。所以必须保证各节点cookie保持一致，否则节点之间就无法通信。
将10-186-21-84中的cookie 复制到10-186-21-85中，先修改下10-186-21-84中的.erlang.cookie权限
```bash
chmod 777  /root/.erlang.cookie
```
将queue的/root/.erlang.cookie这个文件，拷贝到10-186-21-85的同一位置（反过来亦可），该文件是集群节点进行通信的验证密钥，所有节点必须一致。拷完后重启下RabbitMQ。
复制好后别忘记还原.erlang.cookie的权限，否则可能会遇到错误
chmod 400 /root/.erlang.cookie
设置好cookie后先将三个节点的rabbitmq重启
```bash
rabbitmqctl stop
rabbitmq-server start
```
#### 3）重启所有节点
停止所有节点RabbitMq服务，然后使用detached参数独立运行，尤其增加节点停止节点后再次启动遇到无法启动都可以参照这个顺序（此步骤很重要）
```bash
rabbitmqctl stop
rabbitmq-server -detached
```
分别查看节点集群信息
第一台
```bash
[root@10-186-21-84 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@10-186-30-84 ...
[{nodes,[{disc,['rabbit@10-186-30-84']}]},
 {running_nodes,['rabbit@10-186-30-84']},
 {cluster_name,<<"rabbit@10-186-30-84">>},
 {partitions,[]},
 {alarms,[{'rabbit@10-186-30-84',[]}]}]
 ```
第二台
```bash
[root@10-186-21-85 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@10-186-21-85 ...
[{nodes,[{disc,['rabbit@10-186-21-85']}]},
 {running_nodes,['rabbit@10-186-21-85']},
 {cluster_name,<<"rabbit@10-186-21-85">>},
 {partitions,[]},
 {alarms,[{'rabbit@10-186-21-85',[]}]}]
 ```
#### 4）配置集群
10-186-21-84作为内存节点，10-186-21-84作为磁盘节点创建集群。
在10-186-21-84上操作加入集群rabbit@10-186-21-85
```bash
[root@10-186-21-84 ~]# rabbitmqctl stop_app
Stopping rabbit application on node rabbit@10-186-30-84 ...
[root@10-186-21-84 ~]# rabbitmqctl join_cluster --ram rabbit@10-186-21-85
Clustering node rabbit@10-186-30-84 with rabbit@10-186-21-85
[root@10-186-21-84 ~]# rabbitmqctl start_app
Starting node rabbit@10-186-30-84 ...
 completed with 3 plugins.
```
此时重新查看节点上集群信息
在10-186-21-84上查看
```bash
[root@10-186-21-84 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@10-186-21-84 ...
[{nodes,[{disc,['rabbit@10-186-21-85']},{ram,['rabbit@10-186-21-84']}]},
 {running_nodes,['rabbit@10-186-21-85','rabbit@10-186-21-84']},
 {cluster_name,<<"rabbit@10-186-21-85">>},
 {partitions,[]},
 {alarms,[{'rabbit@10-186-21-85',[]},{'rabbit@10-186-21-84',[]}]}]
```
在10-186-21-85上查看
```bash
[root@10-186-21-85 ~]# rabbitmqctl cluster_status
Cluster status of node rabbit@10-186-21-85 ...
[{nodes,[{disc,['rabbit@10-186-21-85']},{ram,['rabbit@10-186-21-84']}]},
 {running_nodes,['rabbit@10-186-21-84','rabbit@10-186-21-85']},
 {cluster_name,<<"rabbit@10-186-21-85">>},
 {partitions,[]},
 {alarms,[{'rabbit@10-186-21-84',[]},{'rabbit@10-186-21-85',[]}]}]
```
disc代表磁盘模式
ram代表内存模式
cluster_name代表集群名称
查看消息队列是否一致
```bash
rabbitmqctl list_queues -p hrsystem
```
#### 5）节点相关操作
change_cluster_node_type {disc | ram}
修改集群节点的类型. 要成功执行此操作，必须首先停止节点，要将节点转换为RAM节点，则此节点不能是集群中的唯一disc节点。ram节点转换为disc节点亦然。
例如:
```bash
rabbitmqctl change_cluster_node_type disc
```
此命令会将一个RAM节点转换为disc节点。

forget_cluster_node [--offline]
[--offline]
允许节点从脱机节点中删除. 这只在所有节点都脱机且最后一个掉线节点不能再上线的情况下有用，从而防止整个集群从启动。它不能使用在其它情况下，因为这会导致不一致．
远程删除一个集群节点.要删除的节点必须是脱机的, 而在删除节点期间节点必须是在线的，除非使用了--offline 标志.
当使用--offline 标志时，rabbitmqctl不会尝试正常连接节点;相反，它会临时改变节点以作修改.如果节点不能正常启动的话，这是非常有用的.在这种情况下，节点将变成集群元数据的规范源（例如，队列的存在），即使它不是以前的。因此，如果有可能，你应该在最新的节点上使用这个命令来关闭。
例如:
```bash
rabbitmqctl -n hare@mcnulty forget_cluster_node rabbit@stringer
```
此命令会从节点hare@mcnulty中删除rabbit@stringer节点.
```bash
rename_cluster_node {oldnode1} {newnode1} [oldnode2] [newnode2 ...]
```
支持在本地数据库中重命名集群节点.
此子命令会促使rabbitmqctl临时改变节点以作出修改. 因此本地集群必须是停止的，其它节点可以是在线或离线的．
这个子命令接偶数个参数，成对表示节点的旧名称和新名称.你必须指定节点的旧名称和新名称，因为其它停止的节点也可能在同一时间重命名.
同时停止所有节点来重命名也是可以的(在这种情况下，每个节点都必须给出旧名称和新名称)或一次停止一个节点来重命名(在这种情况下，每个节点只需要被告知其名句是如何变化的).
例如:
```bash
rabbitmqctl rename_cluster_node rabbit@misshelpful rabbit@cordelia
```
此命令来将节点名称rabbit@misshelpful 重命名为rabbit@cordelia.
```bash
update_cluster_nodes {clusternode}
clusternode
```
用于咨询具有最新消息的节点.
指示已集群的节点醒来时联系clusternode.这不同于join_cluster ，因为它不会加入任何集群 - 它会检查节点已经以clusternode的形式存在于集群中了．
需要这个命令的动机是当节点离线时，集群可以变化.考虑这样的情况，节点Ａ和节点Ｂ都在集群里边，这里节点Ａ掉线了，Ｃ又和Ｂ集群了，然后Ｂ又离开了集群．当Ａ醒来的时候，它会尝试联系Ｂ，但这会失败，因为Ｂ已经不在集群中了.update_cluster_nodes -n A C 可解决这种场景．
```bash
force_boot
```
确保节点将在下一次启动，即使它不是最后一个关闭的。通常情况下，当你关闭整个RabbitMQ 集群时，你重启的第一个节点应该是最后一个下线的节点，因为它可以看到其它节点所看不到的事情. 但有时这是不可能的:例如，如果整个集群是失去了电力而所有节点都在想它不是最后一个关闭的．
在这种节点掉线情况下，你可以调用rabbitmqctl force_boot ．这就告诉节点下一次无条件的启动节点.在此节点关闭后，集群的任何变化，它都会丢失．
如果最后一个掉线的节点永久丢失了，那么你需要优先使用rabbitmqctl forget_cluster_node --offline, 因为它可以确保在丢失的节点上掌握的镜像队列得到提升。
例如:
```bash
rabbitmqctl force_boot
```
这可以强制节点下次启动时不用等待其它节点．
```bash
sync_queue [-p vhost] {queue}
queue
```
同步队列的名称
指示未同步slaves上的镜像队列自行同步.同步发生时，队列会阻塞(所有出入队列的发布者和消费者都会阻塞).此命令成功执行后，队列必须是镜像的。注意，未同步队列中的消息被耗尽后，最终也会变成同步. 此命令主要用于未耗尽的队列。
```bash
cancel_sync_queue [-p vhost] {queue}
queue
```
取消同步的队列名称.
指示同步镜像队列停止同步.
```bash
purge_queue [-p vhost] {queue}
queue
```
要清除队列的名称.
清除队列(删除其中的所有消息).
```bash
set_cluster_name {name}
```
设置集群名称. 集群名称在client连接时，会通报给client,也可用于federation和shovel插件记录消息的来源地. 群集名称默认是来自在群集中的第一个节点的主机名，但可以改变。
例如:
```bash
rabbitmqctl set_cluster_name london
```
设置集群名称为"london".
### 3、镜像模式配置
上面配置RabbitMQ默认集群模式，但并不保证队列的高可用性，尽管交换机、绑定这些可以复制到集群里的任何一个节点，但是队列内容不会复制，虽然该模式解决一部分节点压力，但队列节点宕机直接导致该队列无法使用，只能等待重启，所以要想在队列节点宕机或故障也能正常使用，就要复制队列内容到集群里的每个节点，需要创建镜像队列。
下面配置镜像模式来解决复制的问题，从而提高可用性。
#### 1）增加负载均衡器
选择HAProxy作为RabbitMQ前端的LB
安装haproxy
```bash
yum install haproxy
```
配置负载均衡，在/etc/haproxy/haproxy.cfg中加入以下内容
```bash
listen rabbitmq_cluster 0.0.0.0:5672
    mode tcp
    balance roundrobin
    server   rqslave1 10.186.21.84:5672 check inter 2000 rise 2 fall 3
#   server   rqslave2 10.186.21.85:5672 check inter 2000 rise 2 fall 3
```
负载均衡器会监听5672端口，轮询内存节点10.186.21.84的5672端口,由于资源问题，此处只配置一个内存节点，应该是配置多个才有意义。10.186.21.85为磁盘节点，只做备份不提供给生产者、消费者使用，当然如果我们服务器资源充足情况也可以配置多个磁盘节点，这样磁盘节点除了故障也不会影响，除非同时出故障。
#### 2）配置策略
使用Rabbit镜像功能，需要基于rabbitmq策略来实现，政策是用来控制和修改群集范围的某个vhost队列行为和Exchange行为。在cluster中任意节点启用策略，策略会自动同步到集群节点
rabbitmqctl set_policy -p hrsystem ha-allqueue"^" '{"ha-mode":"all"}'
这行命令在vhost名称为hrsystem创建了一个策略，策略名称为ha-allqueue,策略模式为 all 即复制到所有节点，包含新增节点，策略正则表达式为 “^” 表示所有匹配所有队列名称。Set_policy语法参看官网：
```bash
set_policy [-p vhost] [--priority priority] [--apply-to apply-to] name pattern definition
```
用法
```bash
Usage:
rabbitmqctl [-n <node>] [-t <timeout>] [-q] set_policy [-p <vhost>] [--priority <priority>] [--apply-to <apply-to>] <name> <pattern>  <definition>
```
通过命令行添加，例如
```bash
[root@10-186-21-84 ~]# rabbitmqctl set_policy delete_ha "^delete" '{"ha-mode":"all"}'
Setting policy "delete_ha" for pattern "^delete" to "{"ha-mode":"all"}" with priority "0" for vhost "/" ..
```
查看web界面
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210023.png "Francis'Blog")
从上图可以看到已添加成功。
也可以通过rabbit控制台添加
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210024.png "Francis'Blog")
下图为以添加的两个policy
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210025.png "Francis'Blog")
下图可以看到在exchanges中已经可以看all_ha已经被应用
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210026.png "Francis'Blog")
#### 3）新加入队列
创建队列时需要指定ha 参数，如果不指定x-ha-prolicy 的话将无法复制。
### 4、单机多节点集群配置
在启动RabbitMQ节点之后，服务器默认的节点名称是Rabbit和监听端口5672，如果想在同一台机器上启动多个节点，那么其他的节点就会因为节点名称和端口与默认的冲突而导致启动失败，可以通过设置环境变量来实现，具体方法如下：
配置三个rabbitmq节点，分别为rabbit1, rabbit2和rabbit3 
主要开启命令如下：
```bash
RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit1 rabbitmq-server -detached
RABBITMQ_NODE_PORT=5673 RABBITMQ_NODENAME=rabbit2 rabbitmq-server -detached
RABBITMQ_NODE_PORT=5674 RABBITMQ_NODENAME=rabbit3 rabbitmq-server -detached
```
结束命令如下：
```bash
rabbitmqctl -n rabbit1 stop
rabbitmqctl -n rabbit2 stop
rabbitmqctl -n rabbit3 stop
```
rabbit1上配置集群
```bash
rabbitmqctl -n rabbit1 stop_app
rabbitmqctl -n rabbit1 reset
rabbitmqctl -n rabbit1 cluster 
rabbitmqctl -n rabbit1 start_app
```
rabbit2加入rabbit1集群
```bash
rabbitmqctl -n rabbit2 stop_app
rabbitmqctl -n rabbit2 reset
rabbitmqctl -n rabbit2 cluster rabbit1@`hostname -s`
rabbitmqctl -n rabbit2 start_app
```
查看集群状态
```bash
rabbitmqctl -n rabbit1 cluster_status
```
将rabbit3加入集群
```bash
rabbitmqctl -n rabbit3 stop_app
rabbitmqctl -n rabbit3 reset
rabbitmqctl -n rabbit3 cluster rabbit1@`hostname -s`
rabbitmqctl -n rabbit3 start_app
```
再查看集群状态
```bash
rabbitmqctl -n rabbit1 cluster_status
```
由于此种集群方案对于并无太大意思，所以只简单讲述方法。
同理可以配置多机多节点集群。















