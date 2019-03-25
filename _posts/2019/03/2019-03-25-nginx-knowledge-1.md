---
layout: post
title: Linux下nginx的安装和配置 
category: nginx
tags: [nginx]
---


# Linux下nginx的安装和配置

## 第一部分 nginx安装

## 1.1软件环境

操作系统：centOS 7

## 1.2 nginx下载

下载页面地址：[http://nginx.org/en/download.html](http://nginx.org/en/download.html)
稳定版本nignx 1.8.0为例
[http://nginx.org/download/nginx-1.8.0.tar.gz](http://nginx.org/download/nginx-1.8.0.tar.gz)

## 1.3 linux下安装nginx

    tar zxvf nginx-1.8.0.tar.gz
    cd nginx-1.8.0
    ./configure 
    make
    make install

提示：执行./configure时，可能会提示很多插件找不到，使用命令

    yum install gcc-c++
    yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel

如果需要ssl功能需要openssl库

 #yum -y install openssl openssl—devel

要是这样编译的时候还是找不到openssl库，就需要下载openssl源文件,解压后,将路径指定到解压的路径。

    ./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-openssl=/usr/local/src/openssl-xxxx --with-pcre --with-http_stub_status_module 
## 1.4 nginx启动

    /usr/local/nginx/sbin/nginx –c /usr/local/nginx/conf/nginx.conf

参数“-c”指定配置文件的路径，不加“-c”参数，Nginx会默认加载其安装目录conf子目录中的nginx.conf文件

## 1.5 nginx停止

查询nginx主进程号：

    <code class=" hljs 1c">ps –ef|grep nginx

从容停机：

    <code class=" hljs livecodeserver">kill –quit nginx主进程号

快速关机：

    kill –term nginx主进程号
    kill –int nginx主进程号

强制关机：

    pkill -9 nginx

提示：如果在nginx.conf配置文件中指定pid文件存放路径，该文件中存放nginx当前主进程号。nginx.pid文件默认存放nginx安装目录的logs目录下。所以以上命令可以如下使用：

    kill –信号类型 ‘usr/local/nginx/logs/nginx.pid’
## 1.6 nginx平滑重启

kill –HUP nginx主进程号

## 1.7 配置检验

    nginx –t –c /usr/local/nginx/conf/nginx.conf
## 1.8 tomcat安装

下载tomcat二级制版本

    tar zxvf apache-tomcat-xxxx.tar.gz
    mv apache-tomcat-xxxx /usr/local/tomcat
    mkdir /data0
    mkdir /data0/htdocs
    mkdir /data0/htdocs/www
    cp -rf /usr/local/tomcat/webapps/* /data0/htdocs/www
    vi /usr/local/tomcat/conf/server.xml

修改端口号，如果多个tomcat服务器在同一台机器，使用的端口号不能重复
在最后前一行加一下内容：

## 1.9 nginx对tomcat做负载均衡配置

    user www www;
    worker_processes 8;
    pid /usr/local/nginx/nginx.pid;
    worker_rlimit_nofile 102400;
    events   
    {   
    use epoll;   
    worker_connections 102400;   
    }   
    http   
    {   
      include       mime.types;   
      default_type  application/octet-stream;   
      fastcgi_intercept_errors on;   
      charset  utf-8;   
      server_names_hash_bucket_size 128;   
      client_header_buffer_size 4k;   
      large_client_header_buffers 4 32k;   
      client_max_body_size 300m;   
      sendfile on;   
      tcp_nopush     on;   
    
      keepalive_timeout 60;   
    
      tcp_nodelay on;   
      client_body_buffer_size  512k;   
    
      proxy_connect_timeout    5;   
      proxy_read_timeout       60;   
      proxy_send_timeout       5;   
      proxy_buffer_size        16k;   
      proxy_buffers            4 64k;   
      proxy_busy_buffers_size 128k;   
      proxy_temp_file_write_size 128k;   
    
      gzip on;   
      gzip_min_length  1k;   
      gzip_buffers     4 16k;   
      gzip_http_version 1.1;   
      gzip_comp_level 2;   
      gzip_types       text/plain application/x-javascript text/css application/xml application/json;   
      gzip_vary on;   
    
    upstream tomcat_server {   
     server 127.0.0.1:8080 weight=1 max_fails=2 fail_timeout=30s;   
     server 127.0.0.1:8081 weight=1 max_fails=2 fail_timeout=30s;   
    }   
    
    server {   
        listen 80;   
        server_name  chinaapp.sinaapp.com;   
        index index.jsp index.html index.htm;   
        #发布目录/data/www 
        root  /data/www;   
    
    location /   
    {   
    default_type 'application/json';
              proxy_next_upstream http_502 http_504 error timeout invalid_header;   
          proxy_set_header Host  $host;   
          proxy_set_header X-Real-IP $remote_addr;   
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;   
          proxy_pass http://tomcat_server;   
          expires      3d;   
        }   
      }   
    } 
## 1.10 nginx配置https（ssl）

## 1.10.1自行颁发不受浏览器信任的证书

可以通过以下步骤生成一个简单的证书：
进入你想创建证书和私钥的目录，例如：

    $ cd /usr/local/nginx/conf

创建服务器私钥，命令会让你输入一个口令：

    $ openssl genrsa -des3 -out server.key 1024

创建签名请求的证书（CSR）：

    $ openssl req -new -key server.key -out server.csr

在加载SSL支持的Nginx并使用上述私钥时除去必须的口令：

    $ openssl rsa -in server.key -out server.nopass.key

注：server.key和server.nopass.key都是私钥文件，不同的是前者需要输入密码，后者不需要。如果私钥server.key用于nginx，那么在nginx启动的时候会提示输入私钥文件的密码，nginx则无法完成自动重启。使用server.nopass.key就不需要输入密码。

最后标记证书使用上述私钥和CSR：

    $ openssl x509 -req -days 365 -in server.csr -signkey server.nopass.key -out server.no.pass.crt

通过自行颁发的私钥文件server.nopass.key和CA证书server.nopass.crt，就可以搭建安全的nginx web服务器。

## 1.10.2配置nginx

修改Nginx配置文件，让其包含新标记的证书和私钥：

    server {
      server_name YOUR_SERVER_NAME;
        listen 443;
        ssl on;
        ssl_certificate /usr/local/nginx/conf/server.nopass.crt;
        ssl_certificate_key /usr/local/nginx/conf/server.nopass.key;
    }

重启nginx。
这样就可以通过以下方式访问：
[https://YOUR_SERVER_NAME](https://YOUR_SERVER_NAME)
另外还可以加入如下代码实现80端口重定向到443.

    server {
    listen 80;
    server_name localhost;
    rewrite ^(.*) https://$server_name$1 permanent;
    }
## 1.10.3 ssl性能优化

Nginx默认设置的DH算法（译注：Diffie-Hellman key exchange algorithm）是影响SSL性能的最大因素，因此采用如下设置能增加SSL性能：

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-SHA384:AES256-SHA256:RC4:HIGH:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!AESGCM;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

参考资料：
[Nginx SSL性能调优](http://blog.jobbole.com/44844/)
[nginx使用ssl模块配置HTTPS支持](http://www.cnblogs.com/yanghuahui/archive/2012/06/25/2561568.html)
[nginx实现ssl反向代理实战](http://www.cnblogs.com/yanghuahui/p/3641195.html)

