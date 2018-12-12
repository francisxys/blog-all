---
title: tomcat基础文档
copyright: true
date: 2018-12-10 14:55:18
abbrlink:
tags:
- tomcat
categories:
- OS
- service
---

## 一、	前言
### 1、Tomcat是什么
Tomcat 是由 Apache 开发的一个 Servlet 容器，实现了对 Servlet 和 JSP 的支持，并提供了作为Web服务器的一些特有功能，如Tomcat管理和控制平台、安全域管理和Tomcat阀等。
由于 Tomcat 本身也内含了一个 HTTP 服务器，它也可以被视作一个单独的 Web 服务器。但是，不能将 Tomcat 和 Apache HTTP 服务器混淆，Apache HTTP 服务器是一个用 C 语言实现的 HTTP Web 服务器；这两个 HTTP web server 不是捆绑在一起的。Tomcat 包含了一个配置管理工具，也可以通过编辑XML格式的配置文件来进行配置。
### 2、Tomcat重要目录
```bash
* /bin - Tomcat 脚本存放目录（如启动、关闭脚本）。 *.sh 文件用于 Unix 系统； *.bat 文件用于 Windows 系统。
* /conf - Tomcat 配置文件目录。
* /logs - Tomcat 默认日志目录。
* /webapps - webapp 运行的目录。
```
### 3、web工程发布目录
一般web项目路径结构
```bash
|-- webapp                         # 站点根目录
    |-- META-INF                   # META-INF 目录
    |   `-- MANIFEST.MF            # 配置清单文件
    |-- WEB-INF                    # WEB-INF 目录
    |   |-- classes                # class文件目录
    |   |   |-- *.class            # 程序需要的 class 文件
    |   |   `-- *.xml              # 程序需要的 xml 文件
    |   |-- lib                    # 库文件夹
    |   |   `-- *.jar              # 程序需要的 jar 包
    |   `-- web.xml                # Web应用程序的部署描述文件
    |-- <userdir>                  # 自定义的目录
    |-- <userfiles>                # 自定义的资源文件
```
* webapp：工程发布文件夹。其实每个 war 包都可以视为 webapp 的压缩包。
* META-INF：META-INF 目录用于存放工程自身相关的一些信息，元文件信息，通常由开发工具，环境自动生成。
* WEB-INF：Java web应用的安全目录。所谓安全就是客户端无法访问，只有服务端可以访问的目录。
* /WEB-INF/classes：存放程序所需要的所有 Java class 文件。
* /WEB-INF/lib：存放程序所需要的所有 jar 文件。
* /WEB-INF/web.xml：web 应用的部署配置文件。它是工程中最重要的配置文件，它描述了servlet和组成应用的其它组件，以及应用初始化参数、安全管理约束等。

## 二、安装
Tomcat需要java环境支持，无论windows server还是linux server都需要先安装JAVA环境，本次安装tomcat8.5.29版本需要最低JAVA SE 7或以上版本支持。具体的JAVA版本支持可以从tomcat官网tomcat.apache.org查看。
### 1、	windwos server安装Tomcat
由于windows server环境安装过于简单，以及实际使用较少，在这里只简述过程。
#### 1）安装JDK
下载JDK版本为1.8.0_161，windows版本可以双击直接安装，或者添加系统环境变量JAVA_HOME，环境变量的值为JDK的路径。
#### 2）安装Tomcat
下载zip压缩版本，下载后解压，可以直接双击bin目录下的startup.bat启动，也可以注册服务，以服务方式启动。
### 2、linux server安装Tomcat
#### 1）安装JDK
检查系统是否已经安装jdk
```bash
java -version
```
如果没有显示java版本信息，则表示没有安装java
```bash
#安装java
tar -zxvf jdk-8u161-linux-x64.tar.gz
mv jdk1.8.0_161 /data/jdk1.8
#修改系统变量
vi /etc/profile
#在文本末尾添加以下内容：
PATH=/data/jdk1.8/bin:$PATH
export PATH
#使添加内容生效  
source /etc/profile
```
再查看java版本   出现如下信息表示安装成功
```bash
# java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```
#### 2）安装Tomcat
```bash
#解压
tar zxvf apache-tomcat-8.5.29.tar.gz
#修改目录名并移动位置
mv apache-tomcat-8.5.29 /data/tomcat
#修改默认启动脚本
cd /usr/local/tomcat/bin
vim catalina.sh
#首先找到CLASSPATH=，改为
CLASSPATH=/data/jdk1.8/lib/tools.jar:/data/jdk1.8/lib/dt.jar
#然后在文件的第二行空行插入以下内容（带#号注释的那行也要）：
#chkconfig: 35 85 15
CATALINA_HOME=/data/tomcat
JAVA_HOME=/data/jdk1.8
JRE_HOME=/data/jdk1.8
#复制脚本到/etc/init.d/
cp catalina.sh /etc/init.d/tomcat 
#给脚本加上可可执行权限
chmod +x /etc/init.d/tomcat
#启动tomcat
service tomcat start
#查看进程
# netstat -lntp|grep java
tcp6       0      0 :::8009                 :::*                    LISTEN      19310/java          
tcp6       0      0 :::8080                 :::*                    LISTEN      19310/java          
tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      19310/java
```
访问ip:8080就能看到欢迎页
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210018.png "")
## 三、配置详解
本节将列举一些重要、常见的配置项。
### 1、Server
Server 元素表示整个 Catalina servlet 容器。
因此，它必须是 conf/server.xml 配置文件中的根元素。它的属性代表了整个 servlet 容器的特性。
属性	描述	备注
className	这个类必须实现org.apache.catalina.Server接口。	默认 org.apache.catalina.core.StandardServer
address	服务器等待关机命令的TCP / IP地址。如果没有指定地址，则使用localhost。	
port	服务器等待关机命令的TCP / IP端口号。设置为-1以禁用关闭端口。	
shutdown	必须通过TCP / IP连接接收到指定端口号的命令字符串，以关闭Tomcat。	
### 2、Service
Service元素表示一个或多个连接器组件的组合，这些组件共享一个用于处理传入请求的引擎组件。Server 中可以有多个 Service。
属性	描述	备注
className	这个类必须实现org.apache.catalina.Service接口。	默认 org.apache.catalina.core.StandardService
name	此服务的显示名称，如果您使用标准 Catalina 组件，将包含在日志消息中。与特定服务器关联的每个服务的名称必须是唯一的。	
conf/server.xml配置文件示例：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8080" shutdown="SHUTDOWN">
  <Service name="xxx">
  ...
  </Service>
</Server>
```
### 3、Executor
Executor表示可以在Tomcat中的组件之间共享的线程池。
属性	描述	备注
className	这个类必须实现org.apache.catalina.Executor接口。	默认 org.apache.catalina.core.StandardThreadExecutor
name	线程池名称。	要求唯一, 供Connector元素的executor属性使用
namePrefix	线程名称前缀。	
maxThreads	最大活跃线程数。	默认200
minSpareThreads	最小活跃线程数。	默认25
maxIdleTime	当前活跃线程大于minSpareThreads时,空闲线程关闭的等待最大时间。	默认60000ms
maxQueueSize	线程池满情况下的请求排队大小。	默认Integer.MAX_VALUE
示例：
```bash
<Service name="xxx">
  <Executor name="tomcatThreadPool" namePrefix="catalina-exec-" maxThreads="300" minSpareThreads="25"/>
</Service>
```
### 4、Connector
Connector代表连接组件。Tomcat 支持三种协议：HTTP/1.1、HTTP/2.0、AJP。
属性	说明	备注
asyncTimeout	Servlet3.0规范中的异步请求超时	默认30s
port	请求连接的TCP Port	设置为0,则会随机选取一个未占用的端口号
protocol	协议. 一般情况下设置为 HTTP/1.1,这种情况下连接模型会在NIO和APR/native中自动根据配置选择	
URIEncoding	对URI的编码方式.	如果设置系统变量org.apache.catalina.STRICT_SERVLET_COMPLIANCE为true,使用 ISO-8859-1编码;如果未设置此系统变量且未设置此属性, 使用UTF-8编码
useBodyEncodingForURI	是否采用指定的contentType而不是URIEncoding来编码URI中的请求参数	
以下属性在标准的Connector(NIO, NIO2 和 APR/native)中有效:
属性	说明	备注
acceptCount	当最大请求连接maxConnections满时的最大排队大小	默认100,注意此属性和Executor中属性maxQueueSize的区别.这个指的是请求连接满时的堆栈大小,Executor的maxQueueSize指的是处理线程满时的堆栈大小
connectionTimeout	请求连接超时	默认60000ms
executor	指定配置的线程池名称	
keepAliveTimeout	keeAlive超时时间	默认值为connectionTimeout配置值.-1表示不超时
maxConnections	最大连接数	连接满时后续连接放入最大为acceptCount的队列中. 对 NIO和NIO2连接,默认值为10000;对 APR/native,默认值为8192
maxThreads	如果指定了Executor, 此属性忽略;否则为Connector创建的内部线程池最大值	默认200
minSpareThreads	如果指定了Executor, 此属性忽略;否则为Connector创建线程池的最小活跃线程数	默认10
processorCache	协议处理器缓存Processor对象的大小	-1表示不限制.当不使用servlet3.0的异步处理情况下: 如果配置Executor,配置为Executor的maxThreads;否则配置为Connnector的maxThreads. 如果使用Serlvet3.0异步处理, 取maxThreads和maxConnections的最大值
### 5、Context
Context元素表示一个Web应用程序，它在特定的虚拟主机中运行。每个Web应用程序都基于Web应用程序存档（WAR）文件，或者包含相应的解包内容的相应目录，如Servlet规范中所述。
属性	说明	备注
altDDName	web.xml部署描述符路径	默认 /WEB-INF/web.xml
docBase	Context的Root路径	和Host的appBase相结合, 可确定web应用的实际目录
failCtxIfServletStartFails	同Host中的failCtxIfServletStartFails, 只对当前Context有效	默认为false
logEffectiveWebXml	是否日志打印web.xml内容(web.xml由默认的web.xml和应用中的web.xml组成)	默认为false
path	web应用的context path	如果为根路径,则配置为空字符串(""), 不能不配置
privileged	是否使用Tomcat提供的manager servlet	
reloadable	/WEB-INF/classes/ 和/WEB-INF/lib/ 目录中class文件发生变化是否自动重新加载	默认为false
swallowOutput	true情况下, System.out和System.err输出将被定向到web应用日志中	默认为false
### 6、Engine
Engine元素表示与特定的Catalina服务相关联的整个请求处理机器。它接收并处理来自一个或多个连接器的所有请求，并将完成的响应返回给连接器，以便最终传输回客户端。
属性	描述	备注
defaultHost	默认主机名，用于标识将处理指向此服务器上主机名称但未在此配置文件中配置的请求的主机。	这个名字必须匹配其中一个嵌套的主机元素的名字属性。
name	此引擎的逻辑名称，用于日志和错误消息。	在同一服务器中使用多个服务元素时，每个引擎必须分配一个唯一的名称。
### 7、Host
Host元素表示一个虚拟主机，它是一个服务器的网络名称（如“www.mycompany.com”）与运行Tomcat的特定服务器的关联。
属性	说明	备注
name	名称	用于日志输出
appBase	虚拟主机对应的应用基础路径	可以是个绝对路径, 或${CATALINA_BASE}相对路径
xmlBase	虚拟主机XML基础路径,里面应该有Context xml配置文件	可以是个绝对路径, 或${CATALINA_BASE}相对路径
createDirs	当appBase和xmlBase不存在时,是否创建目录	默认为true
autoDeploy	是否周期性的检查appBase和xmlBase并deploy web应用和context描述符	默认为true
deployIgnore	忽略deploy的正则	
deployOnStartup	Tomcat启动时是否自动deploy	默认为true
failCtxIfServletStartFails	配置为true情况下,任何load-on-startup >=0的servlet启动失败,则其对应的Contxt也启动失败	默认为false
### 8、Cluster
Tomcat集群配置。集群的配置比较复杂，默认的集群配置可以满足一般的开发需求。
一个Cluster配置案例：
```bash
<!-- 
    Cluster(集群,族) 节点,如果你要配置tomcat集群,则需要使用此节点.
    className 表示tomcat集群时,之间相互传递信息使用那个类来实现信息之间的传递.
    channelSendOptions可以设置为2、4、8、10，每个数字代表一种方式
    2 = Channel.SEND_OPTIONS_USE_ACK(确认发送)
    4 = Channel.SEND_OPTIONS_SYNCHRONIZED_ACK(同步发送) 
    8 = Channel.SEND_OPTIONS_ASYNCHRONOUS(异步发送)
    在异步模式下，可以通过加上确认发送(Acknowledge)来提高可靠性，此时channelSendOptions设为10
-->
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster" channelSendOptions="8">
    <!--
        Manager决定如何管理集群的Session信息。Tomcat提供了两种Manager：BackupManager和DeltaManager
        BackupManager－集群下的所有Session，将放到一个备份节点。集群下的所有节点都可以访问此备份节点
        DeltaManager－集群下某一节点生成、改动的Session，将复制到其他节点。
        DeltaManager是Tomcat默认的集群Manager，能满足一般的开发需求
        使用DeltaManager，每个节点部署的应用要一样；使用BackupManager，每个节点部署的应用可以不一样.

        className－指定实现org.apache.catalina.ha.ClusterManager接口的类,信息之间的管理.
        expireSessionsOnShutdown－设置为true时，一个节点关闭，将导致集群下的所有Session失效
        notifyListenersOnReplication－集群下节点间的Session复制、删除操作，是否通知session listeners
        maxInactiveInterval－集群下Session的有效时间(单位:s)。
        maxInactiveInterval内未活动的Session，将被Tomcat回收。默认值为1800(30min)
    -->
    <Manager className="org.apache.catalina.ha.session.DeltaManager"
             expireSessionsOnShutdown="false"
             notifyListenersOnReplication="true"/>

    <!--
        Channel是Tomcat节点之间进行通讯的工具。
        Channel包括5个组件：Membership、Receiver、Sender、Transport、Interceptor
    -->
    <Channel className="org.apache.catalina.tribes.group.GroupChannel">
         <!--
            Membership维护集群的可用节点列表。它可以检查到新增的节点，也可以检查到没有心跳的节点
            className－指定Membership使用的类
            address－组播地址
            port－组播端口
            frequency－发送心跳(向组播地址发送UDP数据包)的时间间隔(单位:ms)。默认值为500
            dropTime－Membership在dropTime(单位:ms)内未收到某一节点的心跳，则将该节点从可用节点列表删除。默认值为3000

            注: 组播（Multicast）：一个发送者和多个接收者之间实现一对多的网络连接。
                一个发送者同时给多个接收者传输相同的数据，只需复制一份相同的数据包。
                它提高了数据传送效率，减少了骨干网络出现拥塞的可能性
                相同组播地址、端口的Tomcat节点，可以组成集群下的子集群
         -->
        <Membership className="org.apache.catalina.tribes.membership.McastService"
                    address="228.0.0.4"
                    port="45564"
                    frequency="500"
                    dropTime="3000"/>

        <!--
            Receiver : 接收器，负责接收消息
            接收器分为两种：BioReceiver(阻塞式)、NioReceiver(非阻塞式)

            className－指定Receiver使用的类
            address－接收消息的地址
            port－接收消息的端口
            autoBind－端口的变化区间
            如果port为4000，autoBind为100，接收器将在4000-4099间取一个端口，进行监听
            selectorTimeout－NioReceiver内轮询的超时时间
            maxThreads－线程池的最大线程数
        -->
        <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
                  address="auto"
                  port="4000"
                  autoBind="100"
                  selectorTimeout="5000"
                  maxThreads="6"/>

        <!--
            Sender : 发送器，负责发送消息
            Sender内嵌了Transport组件，Transport真正负责发送消息
        -->
        <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
            <!--
                Transport分为两种：bio.PooledMultiSender(阻塞式)、nio.PooledParallelSender(非阻塞式) 
            -->
            <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
        </Sender>

        <!--
            Interceptor : Cluster的拦截器
            TcpFailureDetector－网络、系统比较繁忙时，Membership可能无法及时更新可用节点列表，
            此时TcpFailureDetector可以拦截到某个节点关闭的信息，
            并尝试通过TCP连接到此节点，以确保此节点真正关闭，从而更新集群可以用节点列表                 
        -->
        <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>

        <!--
            MessageDispatch15Interceptor－查看Cluster组件发送消息的方式是否设置为
            Channel.SEND_OPTIONS_ASYNCHRONOUS(Cluster标签下的channelSendOptions为8时)。
            设置为Channel.SEND_OPTIONS_ASYNCHRONOUS时，
            MessageDispatch15Interceptor先将等待发送的消息进行排队，然后将排好队的消息转给Sender
        -->
        <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
    </Channel>

    <!--
        Valve : 可以理解为Tomcat的拦截器
        ReplicationValve－在处理请求前后打日志；过滤不涉及Session变化的请求                   
        vmRouteBinderValve－Apache的mod_jk发生错误时，保证同一客户端的请求发送到集群的同一个节点
    -->
    <Valve className="org.apache.catalina.ha.tcp.ReplicationValve" filter=""/>
    <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>

    <!--
        Deployer : 同步集群下所有节点的一致性。
     -->
     <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
                   tempDir="/tmp/war-temp/"
                   deployDir="/tmp/war-deploy/"
                   watchDir="/tmp/war-listen/"
                   watchEnabled="false"/>
    <!--
        ClusterListener : 监听器，监听Cluster组件接收的消息
        使用DeltaManager时，Cluster接收的信息通过ClusterSessionListener传递给DeltaManager
    -->
    <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
</Cluster>
```

## 四、配置实例
### 1、Tomcat配置使用manager管理项目
Tomcat在部署新项目时，可以通过/manager/html来上传war包，来完成部署。部署完成的访问路径为：ip:8080/warname。
首先配置tomcat可以访问manager和host-manager
在conf/tomcat-users.xml中<tomcat-users></tomcat-user>标签中添加以下配置信息：
```bash
<role rolename="admin"/>  
<role rolename="admin-gui"/>  
<role rolename="admin-script"/>
<role rolename="manager"/>  
<role rolename="manager-gui"/> 
<role rolename="manager-script"/>  
<role rolename="manager-jmx"/> 
<role rolename="manager-status"/>  
<user username="tomcat" password="tomcat" roles="admin,admin-gui,admin-script,manager,manager-gui,manager-script,manager-jmx,manager-status"/>
```
相关配置注释：
```bash
manager-gui     #允许访问html接口(即URL路径为/manager/html/*)
manager-script   #允许访问纯文本接口(即URL路径为/manager/text/*)
manager-jmx   #允许访问JMX代理接口(即URL路径为/manager/jmxproxy/*)
manager-status   #允许访问Tomcat只读状态页面(即URL路径为/manager/status/*)
特别需要说明的是：manager-gui、manager-script、manager-jmx均具备manager-status的权限，也就是说，manager-gui、manager-script、manager-jmx三种角色权限无需再额外添加manager-status权限，即可直接访问路径”/manager/status/*”。
Tomcat8中还需要增加一段配置才能满足要求，在conf/Catalina/localhost中新建文件名manager.xml，内容为：
<Context privileged="true" antiResourceLocking="false"   
         docBase="${catalina.home}/webapps/manager">  
             <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />  
</Context>
在conf/Catalina/localhost中新建文件名host-manager.xml，内容为：
<Context privileged="true" antiResourceLocking="false"   
         docBase="${catalina.home}/webapps/host-manager">  
             <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />  
</Context>
```  
配置完成后，重启tomcat，即可以访问manager。
### 2、Nginx+Tomcat配置+多Tomcat负载均衡
关于Nginx的配置请查看文档《Nginx应用文档》，Nginx的配置文件如下：
```bash
user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
pid        logs/nginx.pid;


events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';
	
    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
    upstream lvs {
          #配置两台tomcat负载均衡，ip：port地址为tomcat地址。weight为权重，权重越大，访问概率越大，
         server 127.0.0.1:8081 weight=10;
         server 127.0.0.1:8082 weight=10;
         }
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

      #  access_log  logs/host.access.log  main;
    location ~ .*\.(gif|jpg|jpeg|bmp|png|ico|txt|js|css)$
             {
                  root /data/nainx/html/;
                  #expires定义用户浏览器缓存的时间为7天，如果静态页面不常更新，可以设置更长，这样可以节省带宽和缓解服务器的压力
                   expires      7d; 
                                                   
              } 
       location ~ (\.jsp)|(\.do)$
     {    
             #root   html;
             #index  index.jsp index.htm;
         #lvs是 upstream 后面的名字 lvs
             proxy_pass http://lvs;
             #localhost是nginx服务器的主机地址，如果不写此句，会导致静态文件访问路径为http://lvs，导致找不到地址
           proxy_set_header Host localhost;  
                #forwarded信息，用于告诉后端服务器终端用户的ip地址，否则后端服务器只能获取前端代理服务器的ip地址。
            proxy_set_header Forwarded $remote_addr;  
        }

     # / 表示匹配所有地址，默认最大前缀匹配，如果其他没有匹配的才会匹配
        location /{
             root  /data/lvs;
              index  index.html index.htm;
               }
            
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #错误页面地址，500 502 503 504错误的地址  
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```
路径转发配置（可以不用配置）：
```bash
server {
        listen       80;
        server_name   *.*.*.*;#本服务器ip地址

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

      
       location /{
            #root   html;
            #index  index.html index.htm;
            proxy_pass http://10.8.0.66:8090/;
           #当地址最后加上 /时，匹配路径的 yanshi 不会加到转发路径中
            proxy_set_header X-Forwarded-For $remote_addr;
           #当下面这句话不加，Host $host; 会导致post请求参数丢失
            proxy_set_header Host $host;
            proxy_set_header X-Real-Ip $remote_addr;
        }
}
```
以下是关于tomcat的配置：
同一服务器部署多个tomcat时，存在端口号冲突的问题，所以需要修改tomcat配置文件server.xml：
首先了解下tomcat的几个主要端口：
```bash
<Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="60000"  redirectPort="8443" disableUploadTimeout="false"  executor="tomcatThreadPool" URIEncoding="UTF-8"/>
其中8080为HTTP端口，8443为HTTPS端口
<Server port="8005" shutdown="SHUTDOWN">   
8005为远程停服务端口
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
``` 
8009为AJP端口，APACHE能过AJP协议访问TOMCAT的8009端口。
部署多个tomcat主要修改三个端口：
第一个tomcat配置server.xml如下：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8006" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" /> 
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" /> 
  <GlobalNamingResources>    
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <Service name="Catalina">    
    <Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">       
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <Host name="localhost"  appBase="/data/lvs "
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```
三个端口分别为：8006、8081、8010
第二个tomcat配置server.xml如下：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8007" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" /> 
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" /> 
  <GlobalNamingResources>    
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <Service name="Catalina">    
    <Connector port="8082" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8011" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">
      <Realm className="org.apache.catalina.realm.LockOutRealm">       
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <Host name="localhost"  appBase="/data/lvs "
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```
三个端口分别为：8007、8082、8011
如果需要更多的tomcat做负载，端口号依次增加即可。
nginx与tomcat的结合，主要用的是nginx中的upstream,后端可包括有多台tomcat来处ginx的upstream目前支持5种方式的分配
#### 1）轮询（默认）
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
#### 2）weight
指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
例如：
```bash
    upstream bakend {
        server 192.168.0.14 weight=10;
        server 192.168.0.15 weight=10;
   }
```
#### 3）ip_hash
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session 的问题。
例如：
```bash
upstreambakend {
        ip_hash;
        server 192.168.0.14:88;
        server 192.168.0.15:80;
   }
```
#### 4）fair（第三方）
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
```bash
upstream backend {
    server server1;
    server server2;
    fair;
}
```
#### 5）url_hash（第三方）
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
```bash
upstream backend {
    server squid1:3128;
    server squid2:3128;
    hash   $request_uri;
    hash_methodcrc32;
}
```
示例：
```bash
upstream bakend{#定义负载均衡设备的Ip及设备状态
    ip_hash;
    server127.0.0.1:9090 down;
    server127.0.0.1:8080 weight=2;
    server127.0.0.1:6060;
    server127.0.0.1:7070 backup;
}
```
在需要使用负载均衡的server中增加
```bash
proxy_pass http://bakend/;
```
每个设备的状态设置为:
1.down 表示单前的server暂时不参与负载
2.weight 默认为1.weight越大，负载的权重就越大。
3.max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误
4.fail_timeout:max_fails次失败后，暂停的时间。
5.backup： 其它所有的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

nginx支持同时设置多组的负载均衡，用来给不用的server来使用。
client_body_in_file_only 设置为On 可以讲clientpost过来的数据记录到文件中用来做debug
client_body_temp_path 设置记录文件的目录 可以设置最多3层目录
location 对URL进行匹配.可以进行重定向或者进行新的代理 负载均衡
### 3、Apache+Tomcat集群配置
#### 1）集群简述
集群是一组协同工作的服务实体，用以提供比单一服务实体更具扩展性与可用性的服务平台。在客户端看来，一个集群就象是一个服务实体，但 事实上集群由一组服务实体组成。 
与单一服务实体相比较，集群提供了以下两个关键特性： 
·可扩展性－－集群的性能不限于单一的服务实体，新的服 务实体可以动态地加入到集群，从而增强集群的性能。 
·高可用性－－集群通过服务实体冗余使客户端免于轻易遇到out of service的警告。在集群中，同样的服务可以由多个服务实体提供。如果一个服务实体失败了，另一个服务实体会接管失败的服务实体。集群提供的从一个出 错的服务实体恢复到另一个服务实体的功能增强了应用的可用性。 
为了具有可扩展性和高可用性特点，集群的必须具备以下两大能力： 
* 负载均衡－－负载均衡能把任务比较均衡地分布到集群环境下的计算和网络资源。 
* 错误恢复－－由于某种原因，执行某个任务的资源出现故障，另一服 务实体中执行同一任务的资源接着完成任务。这种由于一个实体中的资源不能工作，另一个实体中的资源透明的继续完成任务的过程叫错误恢复。 
负载均衡 和错误恢复都要求各服务实体中有执行同一任务的资源存在，而且对于同一任务的各个资源来说，执行任务所需的信息视图（信息上下文）必须是一样的。 

集群主要分成三大类：高可用集群(High Availability Cluster/HA)， 负载均衡集群(Load Balance Cluster)，高性能计算集群(High Performance Computing Cluster/HPC) 
* 高可用集群(High Availability Cluster/HA)：一般是指当集群中有某个节点失效的情况下，其上的任务会自动转移到其他正常的节点上。还指可以将集群中的某节点进行离线维护再上线，该过程并不影响整个集群的运行。常见的就是2个节点做 成的HA集群，有很多通俗的不科学的名称，比如"双机热备", "双机互备", "双机"，高可用集群解决的是保障用户的应用程序持续对外提供服 务的能力。 
* 负载均衡集群(Load Balance Cluster)：负载均衡集群运行时一般通过一个或者多个前端负载均衡器将工作负载分发到后端的一组服务器上，从而达到将工作负载分发。这样的计算机集群有时也被称为服务器群（Server Farm）。一般web服务器集群、数据库集群 和应用服务器集群都属于这种类型。这种集群可以在接到请求时，检查接受请求较少，不繁忙的服务器，并把请求转到这些服务器 上。从检查其他服务器状态这一点上 看，负载均衡和容错集群很接近，不同之处是数量上更多。 
* 高性能计算集群(High Performance Computing Cluster/HPC)：高性能计算集群采用将计算任务分配到集群的不同计算节点而提高计算能力，因而主要应用在科学计算领域。这类集群致力于提供单个计算机所不能提供的强大的计算能力 

Tomcat集群配置的优缺点：
通常配置tomcat集群有三种方式：使用DNS轮询，使用apache r-proxy代理方式，使用apache mod_jk方式。 
（1）DNS轮询的缺点：当集群中某台服务器停止之后，用户由于dns缓存的缘故，便无法访问服务，必 须等到dns解析更新，或者这台服务器重新启动。还有就是必须把集群中的所有服务端口暴露给外界，没有用apache做前置代理的方式安全，并 且占用大量公网IP地址，而且tomcat还要负责处理静态网页资源，影响效率。优点是集群配置最简单，dns设置也非常简单。 
（2）R- proxy的缺点：当其中一台tomcat停止运行的时候，apache仍然会转发请求过去，导致502网关错误。但是只要服务器再启动就不存 在这个问题。 
（3）mod_jk方式的优点是，Apache 会自动检测到停止掉的tomcat，然后不再发请求过去。缺点就是，当停 止掉的tomcat服务器再次启动的时候，Apache检测不到，仍然不会转发请求过去。 
R-proxy和mod_jk的共同优点是.可 以只将Apache置于公网，节省公网IP地址资源。可以通过设置来实现Apache专门负责处理静态网页，让Tomcat专门负责处理jsp和 servlet等动态请求。共同缺点是：如果前置Apache代理服务器停止运行，所有集群服务将无法对外提供。R-proxy和 mod_jk对静态页面请求的处理，都可以通设置来选取一个尽可能优化的效果。这三种方式对实现最佳负载均衡都有一定不足，mod_jk相对好些，可以通过设置lbfactor参数来分配请求任务。
2）Apache安装
```bash
#下载安装包
wget http://mirrors.hust.edu.cn/apache//httpd/httpd-2.4.33.tar.gz
wget http://archive.apache.org/dist/apr/apr-1.4.5.tar.gz
wget http://archive.apache.org/dist/apr/apr-util-1.3.12.tar.gz
wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.10/pcre-8.42.zip
#apr安装
tar -zxf apr-1.4.5.tar.gz    
cd  apr-1.4.5    
./configure --prefix=/usr/local/apr    
make && make install    
#apr-util安装
tar -zxf apr-util-1.3.12.tar.gz    
cd apr-util-1.3.12    
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr/bin/apr-1-config   
make && make install
#pcre安装
unzip -o pcre-8.42.zip    
cd pcre-8.42
./configure --prefix=/usr/local/pcre    
make && make install  
#apache安装
tar -zxvf httpd-2.4.33.tar.gz
cd httpd-2.4.33
./configure --prefix=/data/apache --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre
make && make install
```
在apache安装过程中会有报错：
差找不到文件apr_escape.h
由于centos7默认是使用的是xfs的硬盘格式，在安装apr的时候由于automake编译根据硬盘格式会默认系统不需要这个文件，如果是其他硬盘格式，比如：ext3、ext4，则不会出现这个问题。
解决方法：从apache官网上查找该文件源码，并放入apr的include/apr-1内，本次环境apr的目录为：/usr/local/apr/include/apr-1
以下为apr_escape.h文件的源码
```bash
/* Licensed to the Apache Software Foundation (ASF) under one or more
* contributor license agreements.  See the NOTICE file distributed with
* this work for additional information regarding copyright ownership.
* The ASF licenses this file to You under the Apache License, Version 2.0
* (the "License"); you may not use this file except in compliance with
* the License.  You may obtain a copy of the License at
*
*     http://www.apache.org/licenses/LICENSE-2.0
*
* Unless required by applicable law or agreed to in writing, software
* distributed under the License is distributed on an "AS IS" BASIS,
* WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
* See the License for the specific language governing permissions and
* limitations under the License.
*/
/**
* @file apr_escape.h
* @brief APR-UTIL Escaping
*/
#ifndef APR_ESCAPE_H
#define APR_ESCAPE_H
#include "apr.h"
#include "apr_general.h"
#ifdef __cplusplus
extern "C" {
#endif

/**
* @defgroup APR_Util_Escaping Escape functions
* @ingroup APR
* @{
*/

/* Simple escape/unescape functions.
*
*/

/**
* When passing a string to one of the escape functions, this value can be
* passed to indicate a string-valued key, and have the length computed
* automatically.
*/
#define APR_ESCAPE_STRING     (-1)

/**
* Perform shell escaping on the provided string.
*
* Shell escaping causes characters to be prefixed with a '\' character.
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param str The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or the string was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_shell(char *escaped, const char *str,
apr_ssize_t slen, apr_size_t *len);

/**
* Perform shell escaping on the provided string, returning the result
* from the pool.
*
* Shell escaping causes characters to be prefixed with a '\' character.
*
* If no characters were escaped, the original string is returned.
* @param p Pool to allocate from
* @param str The original string
* @return the encoded string, allocated from the pool, or the original
* string if no escaping took place or the string was NULL.
*/
APR_DECLARE(const char *) apr_pescape_shell(apr_pool_t *p, const char *str)
__attribute__((nonnull(1)));

/**
* Unescapes a URL, leaving reserved characters intact.
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param url String to be unescaped
* @param slen The length of the original url, or APR_ESCAPE_STRING
* @param forbid Optional list of forbidden characters, in addition to
* 0x00
* @param reserved Optional list of reserved characters that will be
* left unescaped
* @param plus If non zero, '+' is converted to ' ' as per
* application/x-www-form-urlencoded encoding
* @param len If set, the length of the escaped string will be returned
* @return APR_SUCCESS on success, APR_NOTFOUND if no characters are
* decoded or the string is NULL, APR_EINVAL if a bad escape sequence is
* found, APR_BADCH if a character on the forbid list is found.
*/
APR_DECLARE(apr_status_t) apr_unescape_url(char *escaped, const char *url,
apr_ssize_t slen, const char *forbid, const char *reserved, int plus,
apr_size_t *len);

/**
* Unescapes a URL, leaving reserved characters intact, returning the
* result from a pool.
* @param p Pool to allocate from
* @param url String to be unescaped in place
* @param forbid Optional list of forbidden characters, in addition to
* 0x00
* @param reserved Optional list of reserved characters that will be
* left unescaped
* @param plus If non zero, '+' is converted to ' ' as per
* application/x-www-form-urlencoded encoding
* @return A string allocated from the pool on success, the original string
* if no characters are decoded, or NULL if a bad escape sequence is found
* or if a character on the forbid list is found, or if the original string
* was NULL.
*/
APR_DECLARE(const char *) apr_punescape_url(apr_pool_t *p, const char *url,
const char *forbid, const char *reserved, int plus)
__attribute__((nonnull(1)));

/**
* Escape a path segment, as defined in RFC1808.
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param str The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or the string was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_path_segment(char *escaped,
const char *str, apr_ssize_t slen, apr_size_t *len);

/**
* Escape a path segment, as defined in RFC1808, returning the result from a
* pool.
* @param p Pool to allocate from
* @param str String to be escaped
* @return A string allocated from the pool on success, the original string
* if no characters are encoded or the string is NULL.
*/
APR_DECLARE(const char *) apr_pescape_path_segment(apr_pool_t *p,
const char *str) __attribute__((nonnull(1)));

/**
* Converts an OS path to a URL, in an OS dependent way, as defined in RFC1808.
* In all cases if a ':' occurs before the first '/' in the URL, the URL should
* be prefixed with "./" (or the ':' escaped). In the case of Unix, this means
* leaving '/' alone, but otherwise doing what escape_path_segment() does. For
* efficiency reasons, we don't use escape_path_segment(), which is provided for
* reference. Again, RFC 1808 is where this stuff is defined.
*
* If partial is set, os_escape_path() assumes that the path will be appended to
* something with a '/' in it (and thus does not prefix "./").
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param path The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param partial If non zero, suppresses the prepending of "./"
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or if the string was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_path(char *escaped, const char *path,
apr_ssize_t slen, int partial, apr_size_t *len);

/**
* Converts an OS path to a URL, in an OS dependent way, as defined in RFC1808,
* returning the result from a pool.
*
* In all cases if a ':' occurs before the first '/' in the URL, the URL should
* be prefixed with "./" (or the ':' escaped). In the case of Unix, this means
* leaving '/' alone, but otherwise doing what escape_path_segment() does. For
* efficiency reasons, we don't use escape_path_segment(), which is provided for
* reference. Again, RFC 1808 is where this stuff is defined.
*
* If partial is set, os_escape_path() assumes that the path will be appended to
* something with a '/' in it (and thus does not prefix "./").
* @param p Pool to allocate from
* @param str The original string
* @param partial If non zero, suppresses the prepending of "./"
* @return A string allocated from the pool on success, the original string
* if no characters are encoded or if the string was NULL.
*/
APR_DECLARE(const char *) apr_pescape_path(apr_pool_t *p, const char *str,
int partial) __attribute__((nonnull(1)));

/**
* Urlencode a string, as defined in
* http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.1.
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param str The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or if the stirng was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_urlencoded(char *escaped, const char *str,
apr_ssize_t slen, apr_size_t *len);

/**
* Urlencode a string, as defined in
* http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.1, returning
* the result from a pool.
* @param p Pool to allocate from
* @param str String to be escaped
* @return A string allocated from the pool on success, the original string
* if no characters are encoded or if the string was NULL.
*/
APR_DECLARE(const char *) apr_pescape_urlencoded(apr_pool_t *p,
const char *str) __attribute__((nonnull(1)));

/**
* Apply entity encoding to a string. Characters are replaced as follows:
* '<' becomes '&lt;', '>' becomes '&gt;', '&' becomes '&amp;', the
* double quote becomes '&quot;" and the single quote becomes '&apos;'.
*
* If toasc is not zero, any non ascii character will be encoded as
* '%\#ddd;', where ddd is the decimal code of the character.
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param str The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param toasc If non zero, encode non ascii characters
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or the string was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_entity(char *escaped, const char *str,
apr_ssize_t slen, int toasc, apr_size_t *len);

/**
* Apply entity encoding to a string, returning the result from a pool.
* Characters are replaced as follows: '<' becomes '&lt;', '>' becomes
* '&gt;', '&' becomes '&amp;', the double quote becomes '&quot;" and the
* single quote becomes '&apos;'.
* @param p Pool to allocate from
* @param str The original string
* @param toasc If non zero, encode non ascii characters
* @return A string allocated from the pool on success, the original string
* if no characters are encoded or the string is NULL.
*/
APR_DECLARE(const char *) apr_pescape_entity(apr_pool_t *p, const char *str,
int toasc) __attribute__((nonnull(1)));

/**
* Decodes html entities or numeric character references in a string. If
* the string to be unescaped is syntactically incorrect, then the
* following fixups will be made:
* unknown entities will be left undecoded;
* references to unused numeric characters will be deleted.
* In particular, &#00; will not be decoded, but will be deleted.
* @param unescaped Optional buffer to write the encoded string, can be
* NULL
* @param str The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or the string was NULL
*/
APR_DECLARE(apr_status_t) apr_unescape_entity(char *unescaped, const char *str,
apr_ssize_t slen, apr_size_t *len);

/**
* Decodes html entities or numeric character references in a string. If
* the string to be unescaped is syntactically incorrect, then the
* following fixups will be made:
* unknown entities will be left undecoded;
* references to unused numeric characters will be deleted.
* In particular, &#00; will not be decoded, but will be deleted.
* @param p Pool to allocate from
* @param str The original string
* @return A string allocated from the pool on success, the original string
* if no characters are encoded or the string is NULL.
*/
APR_DECLARE(const char *) apr_punescape_entity(apr_pool_t *p, const char *str)
__attribute__((nonnull(1)));

/**
* Escape control characters in a string, as performed by the shell's
* 'echo' command. Characters are replaced as follows:
* \\a alert (bell), \\b backspace, \\f form feed, \\n new line, \\r carriage
* return, \\t horizontal tab, \\v vertical tab, \\ backslash.
*
* Any non ascii character will be encoded as '\\xHH', where HH is the hex
* code of the character.
*
* If quote is not zero, the double quote character will also be escaped.
* @param escaped Optional buffer to write the encoded string, can be
* NULL
* @param str The original string
* @param slen The length of the original string, or APR_ESCAPE_STRING
* @param quote If non zero, encode double quotes
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if no changes to the string were
* detected or the string was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_echo(char *escaped, const char *str,
apr_ssize_t slen, int quote, apr_size_t *len);

/**
* Escape control characters in a string, as performed by the shell's
* 'echo' command, and return the results from a pool. Characters are
* replaced as follows: \\a alert (bell), \\b backspace, \\f form feed,
* \\n new line, \\r carriage return, \\t horizontal tab, \\v vertical tab,
* \\ backslash.
*
* Any non ascii character will be encoded as '\\xHH', where HH is the hex
* code of the character.
*
* If quote is not zero, the double quote character will also be escaped.
* @param p Pool to allocate from
* @param str The original string
* @param quote If non zero, encode double quotes
* @return A string allocated from the pool on success, the original string
* if no characters are encoded or the string is NULL.
*/
APR_DECLARE(const char *) apr_pescape_echo(apr_pool_t *p, const char *str,
int quote);

/**
* Convert binary data to a hex encoding.
* @param dest The destination buffer, can be NULL
* @param src The original buffer
* @param srclen The length of the original buffer
* @param colon If not zero, insert colon characters between hex digits.
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if the string was NULL
*/
APR_DECLARE(apr_status_t) apr_escape_hex(char *dest, const void *src,
apr_size_t srclen, int colon, apr_size_t *len);

/**
* Convert binary data to a hex encoding, and return the results from a
* pool.
* @param p Pool to allocate from
* @param src The original buffer
* @param slen The length of the original buffer
* @param colon If not zero, insert colon characters between hex digits.
* @return A zero padded buffer allocated from the pool on success, or
* NULL if src was NULL.
*/
APR_DECLARE(const char *) apr_pescape_hex(apr_pool_t *p, const void *src,
apr_size_t slen, int colon) __attribute__((nonnull(1)));

/**
* Convert hex encoded string to binary data.
* @param dest The destination buffer, can be NULL
* @param str The original buffer
* @param slen The length of the original buffer
* @param colon If not zero, ignore colon characters between hex digits.
* @param len If present, returns the length of the string
* @return APR_SUCCESS, or APR_NOTFOUND if the string was NULL, or APR_BADCH
* if a non hex character is present.
*/
APR_DECLARE(apr_status_t) apr_unescape_hex(void *dest, const char *str,
apr_ssize_t slen, int colon, apr_size_t *len);

/**
* Convert hex encoding to binary data, and return the results from a pool.
* If the colon character appears between pairs of hex digits, it will be
* ignored.
* @param p Pool to allocate from
* @param str The original string
* @param colon If not zero, ignore colon characters between hex digits.
* @param len If present, returns the length of the final buffer
* @return A buffer allocated from the pool on success, or NULL if src was
* NULL, or a bad character was present.
*/
APR_DECLARE(const void *) apr_punescape_hex(apr_pool_t *p, const char *str,
int colon, apr_size_t *len);

/** @} */
#ifdef __cplusplus
}
#endif

#endif  /* !APR_ESCAPE_H */
```
由于Apache和Nginx同台安装，所以修改Apache的端口为88，配置ServerName
启动apache
```bash
./bin/apachectl start
```
访问apache服务器：http://ip:88
响应结果：It works！  #apache服务器安装成功
#### 3）Tomcat安装配置
同一服务器部署多个tomcat时，存在端口号冲突的问题，所以需要修改tomcat配置文件server.xml：
首先了解下tomcat的几个主要端口：
```bash
<Connector port="8080" protocol="HTTP/1.1"  connectionTimeout="60000"  redirectPort="8443" disableUploadTimeout="false"  executor="tomcatThreadPool" URIEncoding="UTF-8"/>
#其中8080为HTTP端口，8443为HTTPS端口
<Server port="8005" shutdown="SHUTDOWN">   
#8005为远程停服务端口
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" /> 
#8009为AJP端口，APACHE能过AJP协议访问TOMCAT的8009端口。
```
部署多个tomcat主要修改三个端口：
第一个tomcat配置server.xml如下：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8006" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" /> 
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" /> 
  <GlobalNamingResources>    
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <Service name="Catalina">    
    <Connector port="8081" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">
      <Realm className="org.apache.catalina.realm.LockOutRealm">       
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <Host name="localhost"  appBase="webapps "
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```
三个端口分别为：8006、8081、8010
第二个tomcat配置server.xml如下：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8007" shutdown="SHUTDOWN">
  <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
  <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" /> 
  <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
  <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
  <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" /> 
  <GlobalNamingResources>    
    <Resource name="UserDatabase" auth="Container"
              type="org.apache.catalina.UserDatabase"
              description="User database that can be updated and saved"
              factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
              pathname="conf/tomcat-users.xml" />
  </GlobalNamingResources>
  <Service name="Catalina">    
    <Connector port="8082" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <Connector port="8011" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat2">
      <Realm className="org.apache.catalina.realm.LockOutRealm">       
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
               resourceName="UserDatabase"/>
      </Realm>
      <Host name="localhost"  appBase="webapps "
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
      </Host>
    </Engine>
  </Service>
</Server>
```
三个端口分别为：8007、8082、8011
如果需要更多的tomcat做负载，端口号依次增加即可。
tomcat1测试  
http://ip:8081
tomcat2 测试  
http://ip:8082
结果：显示tomcat首页  
#### 4）集群配置
创建mod_jk.so文件
```bash
#下载
wget http://archive.apache.org/dist/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.41-src.tar.gz
tar -zxvf tomcat-connectors-1.2.41-src.tar.gz 
cd tomcat-connectors-1.2.41-src
cd native/
#进行编译，生成文件，但是不需要make install
./configure --with-apxs=/data/apache/bin/apxs
make
cd apache-2.0/
#拷贝到apache目录下
cp mod_jk.so /data/apache/modules/
/data/apache/modules/
在httpd.conf中加入配置
Include conf/mod_jk.conf
/data/apache/conf下创建mod_jk.conf文件，文件内容为
#mod_jk 配置mod_jk包  
LoadModule jk_module modules/mod_jk.so  
#workers 配置工作负责文件  
JkWorkersFile conf/workers.properties  
#jk log 配置jk日志文件  
JkLogFile logs/mod_jk.log  
#jk log leve 配置日志级别  
JkLogLevel info  
#配置jk日志内存共享  
JkShmFile logs/mod_jk.shm  
#balancer 配置负载均衡模式  
JkMount /*.jsp balancer  
在/data/apache/conf下创建workers.properties文件，文件内容为
#tomcat1的配置  
worker.tomcat1.port=8010
worker.tomcat1.host=127.0.0.1
worker.tomcat1.reference=worker.template  
worker.tomcat1.activation=A  
#worker.tomcat1.lbfactor=1  
#tomcat2 的配置  
worker.tomcat2.port=8011  
worker.tomcat2.host=127.0.0.1
worker.tomcat2.reference=worker.template  
worker.tomcat2.activation=A  
#worker.tomcat2.lbfactor=1  
worker.list=balancer  
#balancer 负载配置  
worker.balancer.type=lb  
worker.balancer.balance_workers=tomcat1,tomcat2  
worker.balancer.sticky_session=1  
#tempalte 负载模板配置  
worker.template.type=ajp13
```  
重启apache、tomcat1、tomcat2验证，可以访问apache地址http://ip:88/testjsp.jsp，可以看到不同的客户端或刷新后显示的可以是tomcat1，也可以是tomcat2，验证成功。
#### 5）Session复制
在Tomcat集群中实现session同步，可以通过session共享和复制来实现，下面以session复制来实现session同步。
Session复制需要修改server.xml配置
Tomcat1中在<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">后面加上以下配置
```bash
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"    
                 channelSendOptions="8">    
          
          <Manager className="org.apache.catalina.ha.session.DeltaManager"    
                   expireSessionsOnShutdown="false"    
                   notifyListenersOnReplication="true"/>    
    
          <Channel className="org.apache.catalina.tribes.group.GroupChannel">    
            <Membership className="org.apache.catalina.tribes.membership.McastService"    
                        address="228.0.0.4"    
                        port="45564"    
                        frequency="500"    
                        dropTime="3000"/>    
            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"    
                      address="auto"   #默认为auto，改为自己的IP  
                      port="4001"    #同一台服务器上的tomcat必须修改为不同的端口，tomcat1修改为4001，tomcat2修改为4002。  
                      autoBind="100"    
                      selectorTimeout="5000"    
                      maxThreads="6"/>    
    
            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">    
              <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>    
            </Sender>    
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>    
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>    
          </Channel>    
    
          <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"    
                 filter=""/>    
          <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>    
    
          <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"    
                    tempDir="/tmp/war-temp/"    
                    deployDir="/tmp/war-deploy/"    
                    watchDir="/tmp/war-listen/"    
                    watchEnabled="false"/>    
    
          <ClusterListener className="org.apache.catalina.ha.session.JvmRouteSessionIDBinderListener"/>    
          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>    
        </Cluster>
```  
Tomcat2中在<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat1">后面加上以下配置
```bash	
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"    
                 channelSendOptions="8">    
          
          <Manager className="org.apache.catalina.ha.session.DeltaManager"    
                   expireSessionsOnShutdown="false"    
                   notifyListenersOnReplication="true"/>    
    
          <Channel className="org.apache.catalina.tribes.group.GroupChannel">    
            <Membership className="org.apache.catalina.tribes.membership.McastService"    
                        address="228.0.0.4"    
                        port="45564"    
                        frequency="500"    
                        dropTime="3000"/>    
            <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"    
                      address="auto"   #默认为auto，改为自己的IP  
                      port="4002"    #同一台服务器上的tomcat必须修改为不同的端口，tomcat1修改为4001，tomcat2修改为4002。  
                      autoBind="100"    
                      selectorTimeout="5000"    
                      maxThreads="6"/>    
    
            <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">    
              <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>    
            </Sender>    
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>    
            <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>    
          </Channel>    
    
          <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"    
                 filter=""/>    
          <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>    
    
          <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"    
                    tempDir="/tmp/war-temp/"    
                    deployDir="/tmp/war-deploy/"    
                    watchDir="/tmp/war-listen/"    
                    watchEnabled="false"/>    
    
          <ClusterListener className="org.apache.catalina.ha.session.JvmRouteSessionIDBinderListener"/>    
          <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>    
        </Cluster> 
```         
在集群中所有tomcat的应用项目中web.xml中的配置 ：在WEB-INF/web.xml中加入以下内容
```bash
<!--此应用将与群集服务器复制Session-->  
<distributable/>  
```








