# Starter

## Nginx基础知识

Nginx是一个高性能的HTTP和反向代理服务器，特点是占有内存少，并发能力强。

Nginx支持热部署，可以长时间不间断运行，它具有以下特点：

* 正向代理：在客户端（浏览器）配置代理服务器，通过代理服务器进行互联网访问（类似VPN）
* 反向代理：客户端不知道代理操作，将请求发送至反向代理服务器中，反向代理服务器会选择目标服务器获取数据，然后返回给客户端。在客户端眼里，反向代理服务器和目标服务器是相同的，只暴露了反向代理服务器，隐藏了真实的目标服务器。
* 负载均衡：可以将请求进行分发，原先集中到一个服务器的请求可以分发至多个服务器，负载也会分发至多个服务器。
* 动静分离：可以加快网站解析速度，动态页面和静态页面交给不同的服务器进行解析，降低原来单个服务器的压力。

## Nginx配置文件及常用命令

使用Nginx操作命令时，需要进入到Nginx的根目录中，一般在/usr/local/nginx （存疑）

* 打开Nginx：service Nginx start
* 关闭Nginx：service Nginx stop
* 重加载Nginx：service Nginx reload

Nginx的配置文件地址：

以下路径基于宝塔Linux面板安装的Nginx

​ nginx.conf文件路径：**/www/server/nginx/conf/nginx.conf**

​ 多域名Nginx配置文件：**/www/server/panel/vhost/nginx/你的域名.conf**

#### 配置文件组成部分

**全局块**

从配置文件开始到events块的内容

主要会设置一些影响nginx服务器整体运行的配置

worker\_processes代表nginx并发处理的数量（会受到软硬件的约束）

**Events块**

Events块涉及的指令主要影响Nginx服务器与用户的网络连接

worker\_connections用于配置nginx与用户网络连接的最大数量

**Http块**

包括文件引入、MIME-TYPE定义、日志自定义、连接超时时间、单连接请求数上限等内容

**Server块**

跟虚拟主机有较大关系

* listen表示Nginx目前正在监听的端口
* location 目录

### 试图达成的目标

1、将gitbook生成的网站放到服务器上，通过Nginx进行外网访问

2、了解Nginx的基本内容，包括配置文件格式、如何将网站放上服务器、如何输入对应的网址后能够跳转到特定的网页

3、了解Nginx配置文件的配置方法，多个网站是否需要多个配置文件

