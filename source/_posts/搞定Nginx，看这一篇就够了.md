```yaml
title: 搞定Nginx，看这一篇就够了
date: 2019-11-21 21:55:00 
tags: 
  - Nginx
```

###   

[TOC]



### 1.Nginx 简介  

#### 1.1  Nginx 概述  

​       Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器,特点是占有内存少，并发能
力强，事实上 nginx 的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用 nginx
网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

#### 1.2  Nginx 作为 web 服务器  

​     Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、 php
等。但是不支持 java。 Java 程序只能通过与 tomcat 配合完成。 Nginx 专为性能优化而开发，
性能是其最重要的考量,实现上非常注重效率 ，能经受高负载的考验,有报告表明能支持高
达 50,000 个并发连接数。  

#### 1.3  正向代理  

​      Nginx 不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。
正向代理：如果把局域网外的 Internet 想象成一个巨大的资源库，则局域网中的客户端要访
问 Internet，则需要通过代理服务器来访问，这种代理服务就称为正向代理。  

   那么什么是正向代理呢？如下图所示：

![image-20191121221152368](G:\Blog\source\image\Nginx\image-20191121221038613.png)

​    

  平时一般都会有这样的烦恼，不能不能直接访问Google的服务器，所以我们需要借助代理服务器来实现访问到Google的目的，这就是正向代理。

#### 1.4  反向代理 

​         反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只
需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返
回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器
地址，隐藏了真实服务器 IP 地址。   

那么什么呢又是反向代理呢？如下图所示：

![image-20191121222139652](G:\Blog\source\image\Nginx\image-20191121222039506.png)

#### 1.5  负载均衡  

​        客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服
务器处理完毕后，再将结果返回给客户端。
这种架构模式对于早期的系统相对单一，并发请求相对较少的情况下是比较适合的，成
本也低。但是随着信息数量的不断增长，访问量和数据量的飞速增长，以及系统业务的复杂
度增加，这种架构会造成服务器相应客户端的请求日益缓慢，并发量特别大的时候，还容易
造成服务器直接崩溃。很明显这是由于服务器性能的瓶颈造成的问题，那么如何解决这种情
况呢？
​         我们首先想到的可能是升级服务器的配置，比如提高 CPU 执行频率，加大内存等提高机
器的物理性能来解决此问题，但是我们知道摩尔定律的日益失效，硬件的性能提升已经不能
满足日益提升的需求了。最明显的一个例子，天猫双十一当天，某个热销商品的瞬时访问量
是极其庞大的，那么类似上面的系统架构，将机器都增加到现有的顶级物理配置，都是不能
够满足需求的。那么怎么办呢？

​        上面的分析我们去掉了增加服务器物理配置来解决问题的办法，也就是说纵向解决问题
的办法行不通了，那么横向增加服务器的数量呢？这时候集群的概念产生了，单个服务器解
决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个

服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们
所说的**负载均衡** 。

<img src="G:\Blog\source\image\Nginx\image-20191121222722417.png" alt="image-20191121222722417" style="zoom: 50%;" />

#### 1.6  动静分离  

​       为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速
度。降低原来单个服务器的压力。

<img src="G:\Blog\source\image\Nginx\image-20191121223113026.png" alt="image-20191121223113026" style="zoom:80%;" />  

### 2.Nginx 安装  



#### 2.1  进入 Nginx 官网  查看最新版本

<img src="G:\Blog\source\image\Nginx\image-20191121223410334.png" style="zoom:80%;" />



#### 2.2  安装Nginx

 1.安装依赖包

```bash
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

2.创建目录并进入

```bash
cd /usr/local
mkdir nginx
cd nginx
```

3.下载安装包（也可以直接官网下载）

```bash
wget http://nginx.org/download/nginx-1.17.5.tar.gz
```

4.解压文件

```bash
tar -xvf nginx-1.17.5.tar.gz 
```

5.安装Ngnix

```bash
//进入nginx目录
cd /usr/local/nginx/nginx-1.17.5

//执行命令
./configure

//执行make命令
make

//执行make install命令
make install
```

6.查看开放的端口号  

```bash
firewall-cmd --list-all
```

如果出现“FirewallD is not running”，说明你已经关闭防火墙；以下7，8步骤无需执行。可以直接跳过。

7.设置开放的端口号  

```bash
firewall-cmd --add-service=http –permanent
sudo firewall-cmd --add-port=80/tcp --permanent
```

8.重启防火墙

```bash
firewall-cmd –reload  
```







### 3.Nginx 常用的命令和配置文件

#### 3.1  Nginx 常用的命令

​     （1）启动命令

```bash
 # 进入 /usr/local/nginx/sbin 目录
 cd  /usr/local/nginx/sbin 
 #启动nginx 
  ./nginx  
 #查看是否已经启动
 ps -ef|grep nginx 
```

![image-20191122204917552](G:\Blog\source\image\Nginx\image-20191122204917552.png)

​     

（2）关闭命令

```bash
# 进入 /usr/local/nginx/sbin 目录
 cd  /usr/local/nginx/sbin 
#关闭nginx 
  ./nginx  -s stop 
#查看是否已经启动
 ps -ef|grep nginx 
```

![image-20191122205458631](G:\Blog\source\image\Nginx\image-20191122205458631.png)



(3)   重新加载命令  

```bash
# 进入 /usr/local/nginx/sbin 目录
 cd  /usr/local/nginx/sbin 
#重新加载nginx 
  ./nginx  -s reload 
#查看是否已经启动
 ps -ef|grep nginx 
```

![image-20191122205840927](G:\Blog\source\image\Nginx\image-20191122205840927.png)

注意：master 进程并没有改变，但是worker进程却变了。

#### 3.2  nginx.conf 配置文件  

​     nginx 安装目录下，其默认的配置文件都放在这个目录的 conf 目录下，而主配置文件
nginx.conf 也在其中，后续对 nginx 的使用基本上都是对此配置文件进行相应的修改 。 

```bash
# 进入 /usr/local/nginx/conf 目录
cd  /usr/local/nginx/conf
```

![image-20191122210333015](G:\Blog\source\image\Nginx\image-20191122210333015.png)

  配置文件中有很多#， 开头的表示注释内容，配置文件内容如下：  

```bash

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


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

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
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

  根据上述文件，我们可以很明显的将 nginx.conf 配置文件分为三部分：  

  第一部分：全局块



​     从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数， 进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。比如上面第一行配置的：  

```bash
#worker_processes表示可以支持的并发处理量
worker_processes  1;
```

  这是 Nginx 服务器并发处理服务的关键配置， worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约 。

  

第二部分： events 块  

  比如上面的配置：  

```bash
#worker_connections表示每个 work process 支持的最大连接数
events {
    worker_connections  1024;
}
```

  上述例子就表示每个 work process 支持的最大连接数为 1024.      

​          events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 wordprocess 可以同时支持的最大连接数等。这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。  



  第三部分： http 块  

```bash
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
    

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
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

  这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。  

  需要注意的是： http 块也可以包括 http 全局块、 server 块。  



  ①http 全局块  

​        http 全局块配置的指令包括文件引入、 MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。  

  ②server 块  

​       这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。  

​    a.全局 server 块

​       最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或 IP 配置。

​    b. locaton 块

​         一个 server 块可以配置多个 location 块。

​         这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是 IP 别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。  

### 4.Nginx 配置实例-反向代理  

#### 4.1  反向代理实例一  

  实现效果：使用 nginx 反向代理，访问www.sugelamei.com直接跳转到 127.0.0.1:8080  （由于笔者的tomcat部署在tomcat中 但是我在windows上访问，因此访问192.168.10.21:8080）

#### 4.2 操作步骤

```bash
#进入/usr/local目录
cd /usr/local
#新建目录tomcat
mkdir tomcat
#进入tomact目录
cd tomcat
#下载tomcat
wget https://mirrors.cnnic.cn/apache/tomcat/tomcat-8/v8.5.49/bin/apache-tomcat-8.5.49.tar.gz
#解压apache-tomcat-8.5.49.tar.gz
tar -xvf  apache-tomcat-8.5.49.tar.gz
#进入apache-tomcat-8.5.49目录
cd apache-tomcat-8.5.49
#启动tomcat
./bin/startup.sh 

```

​      访问页面192.168.10.21:8080，测试是否启动成功，其中192.168.10.21是笔者centos7的ip；上面部署了tomcat ，端口为默认的8080；

![image-20191122220625595](G:\Blog\source\image\Nginx\image-20191122220625595.png)

​          通过修改本地 hosts 文件，将 sugelamei.github.io映射到 192.168.10.21；![image-20191122220852680](G:\Blog\source\image\Nginx\image-20191122220852680.png)

​      添加如下内容：

![image-20191122221836182](G:\Blog\source\image\Nginx\image-20191122221001774.png)

​       在 nginx.conf 配置文件中增加如下配置  

![image-20191122222532831](G:\Blog\source\image\Nginx\image-20191122221649159.png)

   访问  http://www.sugelamei.com/ ![image-20191122222349435](G:\Blog\source\image\Nginx\image-20191122222349435.png)

#### 4.3  反向代理实例二  

  实现效果：使用 nginx 反向代理， 根据访问的路径跳转到不同端口的服务中
nginx 监听端口为 9001，
访问 http://192.168.10.21:9001/edu/ 直接跳转到 192.168.10.21:8081
访问 http://192.168.10.21:9001/vod/ 直接跳转到 192.168.10.21:8082  

#### 4.4  操作步骤

```bash
#进入/usr/local目录
 cd /usr/local/tomcat/
#复制2个tomcat并修改服务端口
cp  -r apache-tomcat-8.5.49 apache-tomcat-8081
cp  -r apache-tomcat-8.5.49 apache-tomcat-8082
#修改服务端口(不会的同学可以网上查一下哟)
vi apache-tomcat-8081/conf/server.xml
vi apache-tomcat-8082/conf/server.xml

#启动tomcat
./apache-tomcat-8081/bin/startup.sh 
./apache-tomcat-8082/bin/startup.sh 

```

测试192.168.10.21:8081和192.168.10.21:8082   

![image-20191122224736171](G:\Blog\source\image\Nginx\image-20191122224736171.png)

![image-20191122224833755](G:\Blog\source\image\Nginx\image-20191122224805925.png)

为了方便测试，把8082的tomcat中

 随意准备2个html文件如下：名字均叫a.html

```html
<h1>8081</h1>
```

放在/usr/local/tomcat/apache-tomcat-8081/webapps/server01/中；

```html
<h1>8082</h1>
```

放在/usr/local/tomcat/apache-tomcat-8082/webapps/server02/中；

修改 nginx 的配置文件 ， 在 http 块中添加 server{}  


```bash
server {
    listen       9001 ;
    server_name  localhost;

    location ~/server01/ {
        proxy_pass http://192.168.10.21:8081;
        
    }
    location ~/server02/ {
        proxy_pass http://192.168.10.21:8082   
      
    }
}
```
 注意:一定要先关闭nginx，再启动nginx；

location 指令说明  

   该指令用于匹配 URL,语法如下：  

```
location [ = | ~ | ~* | ^~] uri {

 }
```

1、 = ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。
2、 ~：用于表示 uri 包含正则表达式，并且区分大小写。
3、 ~*：用于表示 uri 包含正则表达式，并且不区分大小写。
4、 ^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location块中的正则 uri 和请求字符串做匹配。


注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。  



测试

1. http://192.168.10.21:9001/server01/a.html 

![image-20191123231733274](G:\Blog\source\image\Nginx\image-20191123231733274.png)

2.  http://192.168.10.21:9001/server02/a.html 

![image-20191123231754023](G:\Blog\source\image\Nginx\image-20191123231754023.png)



### 5.Nginx配置实例-负载均衡

   实现效果：配置负载均衡

   操作步骤 ：

​    1.首先准备两个同时启动的Tomcat

​    2.在 **nginx.conf** 中进行配置

```bash
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
    ###############此次添加内容的开始标志########################
    #javacloud可以随意指定只要和下面对应就行
    upstream  javacloud{
       server 192.168.10.21:8081;
       server 192.168.10.21:8082;
    }

    server {
          location / {
             proxy_pass   http:javacloud;
             proxy_connect_timeout 10;
            }
         }   
    ###############此次添加内容的结束标志########################
        
    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
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

随着互联网信息的爆炸性增长，负载均衡（load balance）已经不再是一个很陌生的话题， 顾名思义，负载均衡即是将负载分摊到不同的服务单元，既保证服务的可用性，又保证响应 足够快，给用户很好的体验。快速增长的访问量和数据流量催生了各式各样的负载均衡产品， 很多专业的负载均衡硬件提供了很好的功能，但却价格不菲，这使得负载均衡软件大受欢迎， nginx 就是其中的一个，在 linux 下有 Nginx、LVS、Haproxy 等等服务可以提供负载均衡服 务，而且 Nginx 提供了几种分配方式(策略)： 

1.**轮询（默认）**

 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器 down 掉，能自动剔除

 2.**weight(权重)**

weight 代表权,重默认为 1,权重越高被分配的客户端越多 指定轮询几率，weight 和访问比率成正比，用于后端服务器性能不均的情况。例如：

```bash
upstream server_pool{ 
server 192.168.10.21 weight=10; 
server 192.168.10.22 weight=10; 
}
```

3.**ip_hash** 

每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。 例如： 

```bash
upstream server_pool{ 
ip_hash; 
server 192.168.10.21:80; 
server 192.168.10.22:80; 
}
```

4.**fair（第三方）**

按后端服务器的响应时间来分配请求，响应时间短的优先分配。例如： 

```bash
upstream server_pool{ 
server 192.168.10.21:80; 
server 192.168.10.22:80;  
fair; 
}
```



### 6.Nginx配置实例**-**动静分离

​          Nginx 动静分离简单来说就是把动态跟静态请求分开，不能理解成只是单纯的把动态页面和 

静态页面物理分离。严格意义上说应该是动态请求跟静态请求分开，可以理解成使用 Nginx  

处理静态页面，Tomcat 处理动态页面。动静分离从目前实现角度来讲大致分为两种， 一种是纯粹把静态文件独立成单独的域名，放在独立的服务器上，也是目前主流推崇的方案； 另外一种方法就是动态跟静态文件混合在一起发布，通过 nginx 来分开。 

​          通过 location 指定不同的后缀名实现不同的请求转发。通过 expires 参数设置，可以使用

浏览器缓存过期时间，减少与服务器之前的请求和流量。

具体 Expires 定义：是给一个资源设定一个过期时间，也就是说无需去服务端验证，直接通过浏览器自身确认是否过期即可， 所以不会产生额外的流量。此种方法非常适合不经常变动的资源。（如果经常更新的文件， 不建议使用 Expires 来缓存），我这里设置 3d，表示在这 3 天之内访问这个 URL，发送 一个请求，比对服务器该文件最后更新时间没有变化，则不会从服务器抓取，返回状态码 304，如果有修改，则直接从服务器重新下载，返回状态码 200。

1.准备静态资源

2.配置Nginx配置

```bash
 server {
        listen       80;
        server_name  192.168。10.21;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        #data/www下的资源
        location /www/{
            root   /data/;
            index  index.html index.htm;
        }
         #data/image下的资源
        location /image/{
            root   /data/;
            index  index.html index.htm;
        }
```

  最后检查 Nginx 配置是否正确即可，然后测试动静分离是否成功，之需要删除后端 tomcat  

服务器上的某个静态文件，查看是否能访问，如果可以访问说明静态资源 nginx 直接返回 

了，不走后端 tomcat 服务器。





### 7.nginx原理与优化参数配置

整个nginx的架构图：

![image-20191124175229547](G:\Blog\source\image\Nginx\image-20191124175229547.png)

![image-20191124175315545](G:\Blog\source\image\Nginx\image-20191124175315545.png)





**master-workers 的机制的好处**

​         首先，对于每个 worker 进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销， 

同时在编程以及问题查找时，也会方便很多。其次，采用独立的进程，可以让互相之间不会 

影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快启动新的 

worker 进程。当然，worker 进程的异常退出，肯定是程序有 bug 了，异常退出，会导致当 

前 worker 上的所有请求失败，不过不会影响到所有请求，所以降低了风险。



**需要设置多少个** **worker** 

Nginx 同 redis 类似都采用了 io 多路复用机制，每个 worker 都是一个独立的进程，但每个进 

程里只有一个主线程，通过异步非阻塞的方式来处理请求， 即使是千上万个请求也不在话 

下。每个 worker 的线程可以把一个 cpu 的性能发挥到极致。所以 worker 数和服务器的 cpu 

数相等是最为适宜的。设少了会浪费 cpu，设多了会造成 cpu 频繁切换上下文带来的损耗。 



**设置worker 数量** 

```bash
worker_processes 4 

#work 绑定 cpu(4 work 绑定 4cpu)。 

worker_cpu_affinity 0001 0010 0100 1000 

#work 绑定 cpu (4 work 绑定 8cpu 中的 4 个) 。 

worker_cpu_affinity 0000001 00000010 00000100 00001000 
```



**连接数** **worker_connection** 

​         这个值是表示每个 worker 进程所能建立连接的最大值，所以，一个 nginx 能建立的最大连接 

数，应该是 worker_connections *worker_processes。

​      当然，这里说的是最大连接数，对于 HTTP 请 求 本 地 资 源 来 说 ， 能 够 支 持 的 最 大 并 发 数 量 是 worker_connections *worker_processes，如果是支持 http1.1 的浏览器每次访问要占两个连接，所以普通的静态访 问最大并发数是： worker_connections *worker_processes /2，而如果是 HTTP 作 为反向代 理来说，最大并发数量应该是 worker_connections *worker_processes/4。因为作为反向代理服务器，每个并发会建立与客户端的连接和与后端服 务的连接，会占用两个连接。

**nginx.conf结构**

![image-20191124180006121](G:\Blog\source\image\Nginx\image-20191124180006121.png)

### 8.Nginx搭建高可用集群

#### 8.1  Keepalived+Nginx 高可用集群（主从模式）

   1.什么是 nginx高可用

 ![image-20191124190852505](G:\Blog\source\image\Nginx\image-20191124190852505.png)

 (1)需要两台 nginx服务器

 (2)需要 keepalived

 (3)需要虚拟 ip



2.配置高可用的准备工作
（1）需要两台服务器 192.168.10.21 和 192.168.10.66
（2）在两台服务器安装 nginx
（3）在两台服务器安装 keepalived



3.在两台服务器安装 keepalived
（1）使用 yum 命令进行安装

```bash
yum install –y keepalived 
```

（2）安装之后，在 etc 里面生成目录 keepalived，有文件 keepalived.conf

4、完成高可用配置（主从配置）
（1）查看/etc/keepalived/keepalivec.conf 配置文件

```bash
! Configuration File for keepalived

global_defs {
   #故障发生时给谁发邮件通知
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   #通知邮件从哪个地址发出
   notification_email_from Alexandre.Cassen@firewall.loc
   #通知邮件的smtp地址
   smtp_server 192.168.200.1
   #连接smtp服务器的超时时间
   smtp_connect_timeout 30
   #标志本节点的字符串，通常为ip地址，故障发生时邮件会通知到
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}

virtual_server 192.168.200.100 443 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.201.100 443 {
        weight 1
        SSL_GET {
            url {
              path /
              digest ff20ad2481f97b1754ef3e12ecd3a9cc
            }
            url {
              path /mrtg/
              digest 9b3a0c85a887a256d6939da88aabd8cd
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.2 1358 {
    delay_loop 6
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    sorry_server 192.168.200.200 1358

    real_server 192.168.200.2 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.3 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334c
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

virtual_server 10.10.10.3 1358 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 192.168.200.4 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

    real_server 192.168.200.5 1358 {
        weight 1
        HTTP_GET {
            url { 
              path /testurl/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl2/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            url { 
              path /testurl3/test.jsp
              digest 640205b7b0fc66c1ea91c463fac6334d
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }
}

```

keepalived只有一个配置文件keepalived.conf，里面主要包括以下几个配置区域，分别是global_defs、static_ipaddress、vrrp_script、vrrp_instance和virtual_server.

**global_defs区域**

主要是配置故障发生时的通知对象以及机器标志

 

```bash
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id 192.168.224.206
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
```

- notification_email  故障发生时给谁发邮件通知
- notification_email_from  通知邮件从哪个地址发出
- smtp_server 通知邮件的smtp地址
- smtp_connect_timeout 连接smtp服务器的超时时间
- enable_traps开启SNMP（Simple Network Management Protocol）陷阱
- router_id 标志本节点的字符串，通常为ip地址，故障发生时邮件会通知到

**vrrp_script区域**

用来做健康检查的，当检查失败时会将vrrp_instance的priority减少相应的值.

```bash
vrrp_script chk_nginx {
       script "/usr/local/keepalived-1.3.4/nginx_check.sh"
       interval 2 
       weight -20
}
```

-   script:自己写的监测脚本。
- interval 2:每2s监测一次
- weight -20：监测失败，则相应的vrrp_instance的优先级会减少20个点



**vrrp_instance** 

```bash
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    mcast_src_ip 192.168.224.206
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.224.208
    }
    track_script{
    chk_nginx
  }
}
```

- state:只有BACKUP和MASTER。MASTER为工作状态，BACKUP是备用状态

- interface:为网卡接口：可通过ip addr查看自己的网卡接口

- virtual_router_id:虚拟路由标志。同组的virtual_router_id应该保持一致。它将决定多播的MAC地址。

- priority：设置本节点的优先级，优先级高的为master

- advert_int：MASTER与BACKUP同步检查的时间间隔

- virtual_ipaddress：这就是传说中的虚拟ip



（2）修改/etc/keepalived/keepalivec.conf 配置文件，此为192.168.10.21（主）

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
   
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
################此处修改开始########################
   vrrp_strict  chk_http_port{
        script "/usr/local/src/nginx_check.sh"
        interval 2 #（检测脚本执行的间隔）
        weight 2
   }
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER  # 备份服务器上将 MASTER 改为 BACKUP
    interface ens33 #网卡
    virtual_router_id 51  # 主、备机的 virtual_router_id 必须相同
    priority 100  # 主、备机取不同的优先级，主机值较大，备份机值较小
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.55 //  虚拟地址
    }
}
################此处修改结束########################

```

（3）修改/etc/keepalived/keepalivec.conf 配置文件，此为192.168.10.66（备）

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
   
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1 
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
################此处修改开始########################
   vrrp_strict  chk_http_port{
        script "/usr/local/src/nginx_check.sh"
        interval 2 #（检测脚本执行的间隔）
        weight 2
   }
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP  # 备份服务器上将 MASTER 改为 BACKUP
    interface ens33 #网卡
    virtual_router_id 51  # 主、备机的 virtual_router_id 必须相同
    priority 90  # 主、备机取不同的优先级，主机值较大，备份机值较小
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.55 //  虚拟地址
    }
}
################此处修改结束########################

```

（4）在/usr/local/src 添加检测脚本（nginx_check.sh）

```shell
#!/bin/bash
A=`ps -C nginx –no-header |wc -l`
if [ $A -eq 0 ];then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx --no-header |wc -l` -eq 0 ];then
        killall keepalived
    fi
fi
```



（5）把两台服务器上 nginx 和 keepalived 启动
启动 nginx：

```bash
./nginx
```

启动 keepalived：

```bash
systemctl start keepalived.service
```

到此，Keepalived+Nginx 高可用集群就搭建完成了。

#### 8.2 Keepalived+Nginx 高可用集群（双主模式）

![image-20191124203253315](G:\Blog\source\image\Nginx\image-20191124203253315.png)

（1）修改/etc/keepalived/keepalivec.conf 配置文件，此为192.168.10.21

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
   
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
################此处修改开始########################
   vrrp_strict  chk_http_port{
        script "/usr/local/src/nginx_check.sh"
        interval 2 #（检测脚本执行的间隔）
        weight 2
   }
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state MASTER  # 备份服务器上将 MASTER 改为 BACKUP
    interface ens33 #网卡
    virtual_router_id 51  # 主、备机的 virtual_router_id 必须相同
    priority 100  # 主、备机取不同的优先级，主机值较大，备份机值较小
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.55 //  虚拟地址
    }
}

vrrp_instance VI_2 {
 state BACKUP
 interface ens33
 virtual_router_id 52
 priority 90
 advert_int 1
 authentication {
 auth_type PASS
 auth_pass 2222
 }
 virtual_ipaddress {
  192.168.10.44 //  虚拟地址
 } }
################此处修改结束########################

```

（3）修改/etc/keepalived/keepalivec.conf 配置文件，此为192.168.10.66

```bash
! Configuration File for keepalived

global_defs {
   notification_email {
   
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1 
   smtp_connect_timeout 30
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
################此处修改开始########################
   vrrp_strict  chk_http_port{
        script "/usr/local/src/nginx_check.sh"
        interval 2 #（检测脚本执行的间隔）
        weight 2
   }
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_instance VI_1 {
    state BACKUP  # 备份服务器上将 MASTER 改为 BACKUP
    interface ens33 #网卡
    virtual_router_id 51  # 主、备机的 virtual_router_id 必须相同
    priority 90  # 主、备机取不同的优先级，主机值较大，备份机值较小
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.10.55 //  虚拟地址
    }
}

vrrp_instance VI_2 {
 state MASTER
 interface ens33
 virtual_router_id 52
 priority 100
 advert_int 1
 authentication {
 auth_type PASS
 auth_pass 2222
 }
 virtual_ipaddress {
  192.168.10.44 //  虚拟地址
 } }
################此处修改结束########################

```

//重新启动 keepalived  

```
systemctl restart keepalived 
```

到此，keepalived+nginx 高可用集群（双主模式）就搭建完成了

### 9.彩蛋

请大家持续关注公众号：Java橙序猿

 ![img](G:\Blog\source\image\common\Java橙序猿.png)

关注博客：

```
 http://superdevops.cn
```

