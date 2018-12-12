hexo
## 一、本地环境部署
安装nodejs和gitbash
切换到hexo根目录后，进行初始化
``` bash
hexo init
```
## 二、安装相关的插件
``` bash
#搜索插件
npm install hexo-generator-searchdb --save
#上传deploy插件
npm install hexo-deployer-git --save
```
## 三、hexo命令
``` bash
#初始化
hexo init
#创建post页面
hexo new post "name"
#清除历史缓存
hexo clean
#生成页面
hexo generate
#本地运行
hexo server
#上传到github
hexo d -g
#上传之前将CNAME和READNE.md拷贝进public目录下
```
## 四、markdown语法
插入图片
图片链接后跟图片提示
![Alt text](http://pjakaipln.bkt.clouddn.com/20181210019.png "Francis'Blog")
同样支持html语法，居中显示
<center>
<img src="http://pjakaipln.bkt.clouddn.com/%E6%94%AF%E4%BB%98%E5%AE%9D%E6%89%AB%E7%A0%81%E9%A2%86%E7%BA%A2%E5%8C%85.jpg" width="25%" height="25%" />
</center>
添加链接格式
前面为文字，后面位文字所附带的链接
[阿里云产品优惠](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=taqd2wlp)

表格模板
| 名称                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| RABBITMQ_BASE                 | 此基础目录包含了RabbitMQ   server的数据库，日志文件的子目录. 另外，也可以独立设置RABBITMQ_MNESIA_BASE 和 RABBITMQ_LOG_BASE 目录. |
| RABBITMQ_CONFIG_FILE          | 用于配置文件的路径，无.config扩展名. 如果 **configuration file** 存在，服务器将使用它来配置RabbitMQ组件. 参考 **Configuration   guide** 来了解更多信息. |