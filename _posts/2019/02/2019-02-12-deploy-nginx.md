---
layout: post
title: 使用nginx部署网站
no-post-nav: true
category: nginx
tags: [nginx]
---



### 前面的话

如果服务器只需要放置一个网站程序，解析网站到服务器的网站，网站程序监听80端口就可以了。如果服务器有很多应用，借助nginx不仅可以实现端口的代理，还可以实现负载均衡。本文将详细介绍前端及nodeJS项目在服务器配置时需要用到的nginx配置。

### 安装

#### 【卸载nginx】

在介绍如何安装nginx之前，先要介绍如何卸载nginx。因为nginx不正确的安装，导致无法正常运行，所以需要卸载nginx。

1.  `sudo apt-get remove nginx nginx-common # 卸载删除除了配置文件以外的所有文件`

2.  `sudo apt-get purge nginx nginx-common # 卸载所有东东，包括删除配置文件`

3.  `sudo apt-get autoremove # 在上面命令结束后执行，主要是卸载删除Nginx的不再被使用的依赖包`

4.  `sudo apt-get remove nginx-full nginx-common #卸载删除两个主要的包`

#### 【安装nginx】

首先，更新包列表

1.  `sudo apt-get update`

然后，一定要在sudo下安装nginx

1.  `sudo apt-get install nginx`

### 主机配置

#### 【端口配置】

1.  `listen 127.0.0.1:8000;`

2.  `listen *:8000;`

3.  `listen localhost:8000;`

4.  `# IPV6`

5.  `listen [::]:8000;`

6.  `# other params`

7.  `listen 443 default_serer ssl;`

8.  `listen 127.0.0.1 default_server accept_filter=dataready backlog=1024`

#### 【主机名配置】

1.  `server_name www.xiaohuochai.com xiaohuochai.com`

2.  `server_name *.xiaohuochai.com`

3.  `server_name ~^.xiaohuochai.com$`

### 路径配置

#### 【location】

nginx使用location指令来实现URI匹配

1.  `location = / {`

2.  `   # 完全匹配  =`

3.  `   # 大小写敏感 ~`

4.  `   # 忽略大小写 ~*`

5.  `}`

6.  `location ^~ /images/ {`

7.  `   # 前半部分匹配 ^~`

8.  `   # 可以使用正则，如：`

9.  `   # location ~* .(gif|jpg|png)$ { }`

10.  `}`

11.  `location / {`

12.  `   # 如果以上都未匹配，会进入这里`

13.  `}`

#### 【根目录设置】

1.  `location / {`

2.  `root /home/test/;`

3.  `}`

#### 【别名设置】

1.  `location /blog {`

2.  `   alias /home/www/blog/;`

3.  `}`

4.  `location ~ ^/blog/(d+)/([w-]+)$ {`

5.  `   # /blog/20180402/article-name  `

6.  `   # -> /blog/20180402-article-name.md`

7.  `   alias /home/www/blog/$1-$2.md;`

8.  `}`

#### 【首页设置】

1.  `index /html/index.html /php/index.php;`

#### 【重定向页面设置】

1.  `error_page    404  /404.html;`

2.  `error_page    502 503 /50x.html;`

3.  `error_page    404 =200  /1x1.gif;`

5.  `location / {`

6.  `   error_page  404 @fallback;`

7.  `}`

8.  `location @fallback {`

9.  `   # 将请求反向代理到上游服务器处理`

10.  `   proxy_pass http://localhost:9000;`

11.  `}`

#### 【try_files 设置】

1.  `try_files $uri $uri.html $uri/index.html @other;`

2.  `location @other {`

3.  `   # 尝试寻找匹配 uri 的文件，失败了就会转到上游处理`

4.  `   proxy_pass  http://localhost:9000;`

5.  `}`

6.  `location / {`

7.  `   # 尝试寻找匹配 uri 的文件，没找到直接返回 502`

8.  `   try_files $uri $uri.html =502;`

9.  `}`

### 反向代理

代理分为正向和反向代理，正向代理代理的对象是客户端，反向代理代理的对象是服务端。

反向代理（reserve proxy）方式是指用代理服务器来接受 Internet 上的连接请求，然后将请求转发给内部网络中的上游服务器，并将上游服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器对外的表现就是一个 Web 服务器。

#### 【负载均衡设置】

upstream，定义一个上游服务器集群

1.  `upstream backend {`

2.  `   # ip_hash;`

3.  `   server s1.barretlee.com;`

4.  `   server s2.barretlee.com;`

5.  `}`

6.  `server {`

7.  `location / {`

8.  `       proxy_pass http://backend;`

9.  `   }`

10.  `}`

#### 【反向代理设置】

proxy_pass 将请求转发到有处理能力的端上，默认不会转发请求中的 Host 头部

1.  `location /blog {`

2.  `   prox_pass http://localhost:9000;`

4.  `   ### 下面都是次要关注项`

5.  `proxy_set_header Host $host;`

6.  `   proxy_method POST;`

7.  `   # 指定不转发的头部字段`

8.  `proxy_hide_header Cache-Control;`

9.  `proxy_hide_header Other-Header;`

10.  `   # 指定转发的头部字段`

11.  `proxy_pass_header Server-IP;`

12.  `proxy_pass_header Server-Name;`

13.  `   # 是否转发包体`

14.  `proxy_pass_request_body on | off;`

15.  `   # 是否转发头部`

16.  `proxy_pass_request_headers on | off;`

17.  `   # 显形/隐形 URI，上游发生重定向时，Nginx 是否同步更改 uri`

18.  `proxy_redirect on | off;`

19.  `}`

### HTTPS配置

1.  `server{`

2.  `listen 80;`

3.  `       server_name api.xiaohuochai.cc;`

4.  `       return 301 https://api.xiaohuochai.cc$request_uri;`

5.  `}`

6.  `server{`

7.  `listen 443;`

8.  `       server_name api.xiaohuochai.cc;`

9.  `       ssl on;`

10.  `ssl_certificate /home/www/blog/crt/api.xiaohuochai.cc.crt;`

11.  `ssl_certificate_key /home/www/blog/crt/api.xiaohuochai.cc.key;`

12.  `ssl_session_timeout 5m;`

13.  `       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;`

14.  `ssl_protocols TLSv1 TLSv1.1 TLSv1.2;`

15.  `       ssl_prefer_server_ciphers on;`

16.  `       if ($ssl_protocol = "") {`

17.  `rewrite ^(.*)https://$host$1 permanent;`

18.  `       }`

20.  `}`

#### 【HTTP2】

开启HTTP2服务非常简单，只需要在端口443后面添加http2即可。

1.  `server{`

2.  `listen 443 http2;`

3.  `...`

4.  `}`

### gzip配置

开启网站的 gzip 压缩功能，通常可以高达70%，也就是说，如果网页有30K，压缩之后就变成9K， 对于大部分网站，显然可以明显提高浏览速度。

gzip配置在nginx.conf文件中已经存在，只不过默认是注释的状态，只需将注释符号去掉即可

1.  `##`

2.  `   # Gzip Settings`

3.  `   ##`

5.  `   gzip on;`

6.  `gzip_disable "msie6";`

7.  `   gzip_vary on;`

8.  `   gzip_proxied any;`

9.  `gzip_comp_level 6;`

10.  `gzip_buffers 16 8k;`

11.  `gzip_http_version 1.1;`

12.  `   gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;`

### 缓存配置

 如果服务器中存在静态资源，可设置本地强缓存。expires 7d表示在本地缓存7天

1.  `location / {`

2.  `expires 7d;`

3.  `...` 

4.  `}`

设置完成后，浏览器会自动添加expires和cache-control字段，而对于协商缓存Etag和Last-Modified，nginx默认开启，无需配置。

### CSP配置

跨域脚本攻击 XSS 是最常见、危害最大的网页安全漏洞。为了防止它们，要采取很多编程措施，非常麻烦。很多人提出，能不能根本上解决问题，浏览器自动禁止外部注入恶意脚本？这就是"网页安全政策"（Content Security Policy，缩写 CSP）的来历。

CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。它的实现和执行全部由浏览器完成，开发者只需提供配置

目前，CSP有如下指令

1.  `指令 指令值示例 说明`

2.  `default-src    'self' cnd.a.com    定义针对所有类型（js、image、css、web font，ajax 请求，iframe，多媒体等）资源的默认加载策略，某类型资源如果没有单独定义策略，就使用默认的。`

3.  `script-src    'self' js.a.com    定义针对 JavaScript 的加载策略。`

4.  `style-src    'self' css.a.com    定义针对样式的加载策略。`

5.  `img-src    'self' img.a.com    定义针对图片的加载策略。`

6.  `connect-src    'self' 针对 Ajax、WebSocket 等请求的加载策略。不允许的情况下，浏览器会模拟一个状态为 400 的响应。`

7.  `font-src    font.a.com    针对 WebFont 的加载策略。`

8.  `object-src    'self' 针对 <object>、<embed> 或 <applet> 等标签引入的 flash 等插件的加载策略。`

9.  `media-src    media.a.com    针对 <audio> 或 <video> 等标签引入的 HTML 多媒体的加载策略。`

10.  `frame-src    'self' 针对 frame 的加载策略。`

11.  `sandbox    allow-forms    对请求的资源启用 sandbox（类似于 iframe 的 sandbox 属性）。`

12.  `report-uri    /report-uri    告诉浏览器如果请求的资源不被策略允许时，往哪个地址提交日志信息。 特别的：如果想让浏览器只汇报日志，不阻止任何内容，可以改用 Content-Security-Policy-Report-Only 头。`

指令值可以由下面这些内容组成：

1.  `指令值 指令示例 说明`

2.  `img-src    允许任何内容。`

3.  `'none' img-src 'none' 不允许任何内容。`

4.  `'self' img-src 'self' 允许来自相同来源的内容（相同的协议、域名和端口）。`

5.  `data: img-src data: 允许 data: 协议（如 base64 编码的图片）。`

6.  `www.a.com    img-src img.a.com    允许加载指定域名的资源。`

7.  `.a.com    img-src .a.com    允许加载 a.com 任何子域的资源。`

8.  `https://img.com    img-src https://img.com    允许加载 img.com 的 https 资源（协议需匹配）。`

9.  `https: img-src https: 允许加载 https 资源。`

10.  `'unsafe-inline' script-src 'unsafe-inline' 允许加载 inline 资源（例如常见的 style 属性，onclick，inline js 和 inline css 等等）。`

11.  `'unsafe-eval' script-src 'unsafe-eval' 允许加载动态 js 代码，例如 eval()。`

admin.xiaohuochai.cc中的CSP配置如下

1.  `add_header Content-Security-Policy "default-src 'self';`

2.  `script-src 'self' 'unsafe-inline' 'unsafe-eval';`

3.  `img-src 'self' data: https://pic.xiaohuochai.site https://static.xiaohuochai.site;`

4.  `style-src 'self' 'unsafe-inline';`

5.  `frame-src https://demo.xiaohuochai.site https://xiaohuochai.site;";`

### 隐藏信息

在请求响应头中，有这么一行 server: nginx，说明用的是 Nginx 服务器，但并没有具体的版本号。由于某些 Nginx 漏洞只存在于特定的版本，隐藏版本号可以提高安全性。这只需要在配置里加上这个就可以了：

1.  `server_tokens   off;`

### 配置流程

下面在/etc/nginx/conf.d下新建一个配置文件，命名为test-8081.conf，内容如下

注意：一般以域名-端口号来命名配置文件

1.  `upstream xiaohuochai {`

2.  `server 127.0.0.1:8081;`

3.  `}`

4.  `server{`

5.  `listen 80;`

6.  `server_name 1.2.3.4;`

7.  `location / {`

8.  `               proxy_set_header X-Real-IP $remote_addr;`

9.  `               proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;`

10.  `proxy_set_header Host $http_host;`

11.  `               proxy_set_header X-Nginx-Proxy true;`

12.  `               proxy_pass http://test;`

13.  `               proxy_redirect off;`

15.  `       }`

17.  `}`

下面使用sudo nginx -t来测试配置文件是否格式正确

如果不想让报文显示server的详细信息，需要将/etc/nginx/nginx.conf主配置文件中的server_tockens off前面的注释取消即可

接着，重启nginx服务

1.  `sudo nginx -s reload`

### 后端项目

下面来部署后端的nodejs项目，在/etc/nginx/conf.d目录下新建文件，该项目占用3000端口，则起名为api-xiaohuochai-cc-3000.conf

1.  `upstream api {`

2.  `server 127.0.0.1:3000;`

3.  `}`

4.  `server{`

5.  `listen 80;`

6.  `       server_name api.xiaohuochai.cc;`

7.  `       return 301 https://api.xiaohuochai.cc$request_uri;`

8.  `}`

9.  `server{`

10.  `listen 443 http2;`

11.  `       server_name api.xiaohuochai.cc;`

12.  `       ssl on;`

13.  `ssl_certificate /home/www/blog/crt/api.xiaohuochai.cc.crt;`

14.  `ssl_certificate_key /home/www/blog/crt/api.xiaohuochai.cc.key;`

15.  `ssl_session_timeout 5m;`

16.  `       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;`

17.  `ssl_protocols TLSv1 TLSv1.1 TLSv1.2;`

18.  `       ssl_prefer_server_ciphers on;`

19.  `       if ($ssl_protocol = "") {`

20.  `rewrite ^(.*)https://$host$1 permanent;`

21.  `       }`

22.  `location / {`

23.  `       　　　　proxy_set_header X-Real-IP $remote_addr;`

24.  `               proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;`

25.  `proxy_set_header Host $http_host;`

26.  `               proxy_set_header X-Nginx-Proxy true;`

27.  `               proxy_pass http://api;`

28.  `               proxy_redirect off;`

29.  `       }`

30.  `}`

### 后台项目

后台项目起名为admin-xiaohuochai-cc-3001.conf。由于项目采用react构建，与普通的静态网站有些不同

#### 1、前端路由

由于使用前端路由，项目只有一个根入口。当输入类似/posts的url时，找不到这个页面，这是，nginx会尝试加载index.html，加载index.html之后，react-router就能起作用并匹配我们输入的/posts路由，从而显示正确的posts页面。

1.  `try_files $uri $uri/ /index.html = 404;`

#### 2、反向代理

由于该项目需要向后端api.xiaohuochai.cc获取数据，但是后台占用的是3000端口，相当于跨域访问，这时就需要进行反向代理。

1.  `location /api/ {`

2.  `       proxy_pass http://api/;`

3.  `   }`

> 注意：一定要在api后面添加/，否则不生效

#### 3、配置缓存及CSP

1.  `expires 7d;`

2.  `add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; img-src 'self' data: https://pic.xiaohuochai.site https://static.xiaohuochai.site; style-src 'self' 'unsafe-inline'; frame-src https://demo.xiaohuochai.site https://xiaohuochai.site;";`

### 下面是详细的配置文件

1.  `upstream admin {`

2.  `server 127.0.0.1:3001;`

3.  `}`

4.  `server{`

5.  `listen 80;`

6.  `   server_name admin.xiaohuochai.cc;`

7.  `   return 301 https://admin.xiaohuochai.cc$request_uri;`

8.  `root /home/www/blog/admin/build;`

9.  `   index index.html;`

10.  `}`

11.  `server{`

12.  `listen 443 http2;`

13.  `       server_name admin.xiaohuochai.cc;`

14.  `       ssl on;`

15.  `ssl_certificate /home/www/blog/crt/admin.xiaohuochai.cc.crt;`

16.  `ssl_certificate_key /home/www/blog/crt/admin.xiaohuochai.cc.key;`

17.  `ssl_session_timeout 5m;`

18.  `       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;`

19.  `ssl_protocols TLSv1 TLSv1.1 TLSv1.2;`

20.  `       ssl_prefer_server_ciphers on;`

21.  `       if ($ssl_protocol = "") {`

22.  `rewrite ^(.*)https://$host$1 permanent;`

23.  `       }`

24.  `location /api/ {`

25.  `       proxy_pass http://api/;`

26.  `   }`

27.  `location / {`

28.  `       index index.html;`

29.  `root /home/www/blog/admin/build;`

30.  `expires 7d;`

31.  `add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; img-src 'self' data: https://pic.xiaohuochai.site https://static.xiaohuochai.site; style-src 'self' 'unsafe-inline'; frame-src https://demo.xiaohuochai.site https://xiaohuochai.site;";`

32.  `       try_files $uri $uri/ /index.html = 404;`

33.  `   }`

34.  `}`

### 前台项目

前台项目起名为www-xiaohuochai-cc-3002.conf。项目采用vue构建。该项目与后台项目类似，但稍有些不同。不同之处在于，使用主域名xiaohuochai.cc或二级域名www.xiaohuochai.cc都需要跳转。

1.  `server{`

2.  `listen 443 http2;`

3.  `       server_name www.xiaohuochai.cc xiaohuochai.cc;`

4.  `...`

### 详细配置如下

1.  `upstream client {`

2.  `server 127.0.0.1:3002;`

3.  `}`

4.  `server{`

5.  `listen 80;`

6.  `   server_name www.xiaohuochai.cc xiaohuochai.cc;`

7.  `   return 301 https://www.xiaohuochai.cc$request_uri;`

8.  `root /home/www/blog/client/dist;`

9.  `   index index.html;`

10.  `}`

11.  `server{`

12.  `listen 443 http2;`

13.  `       server_name www.xiaohuochai.cc xiaohuochai.cc;`

14.  `       ssl on;`

15.  `ssl_certificate /home/www/blog/client/crt/www.xiaohuochai.cc.crt;`

16.  `ssl_certificate_key /home/www/blog/client/crt/www.xiaohuochai.cc.key;`

17.  `ssl_session_timeout 5m;`

18.  `       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;`

19.  `ssl_protocols TLSv1 TLSv1.1 TLSv1.2;`

20.  `       ssl_prefer_server_ciphers on;`

21.  `       if ($ssl_protocol = "") {`

22.  `rewrite ^(.*)https://$host$1 permanent;`

23.  `       }`

24.  `location /api/ {`

25.  `       proxy_pass http://api/;`

27.  `   }`

28.  `location / {`

29.  `       index index.html;`

30.  `root /home/www/blog/client/source/dist;`

31.  `expires 7d;`

32.  `add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://static.xiaohuochai.site ; img-src 'self' data: https://pic.xiaohuochai.site https://static.xiaohuochai.site; style-src 'self' 'unsafe-inline' https://static.xiaohuochai.site; frame-src https://demo.xiaohuochai.site https://xiaohuochai.site https://www.xiaohuochai.site;";`

33.  `       try_files $uri $uri/ /index.html = 404;`

34.  `   }`

35.  `}`

### SSR项目

如果前端项目是服务器端渲染的SSR项目，则与普通的前端项目有很大不同，它不仅需要守护后端程序，还有前端静态资源的处理，如果是首页，还需要处理www

详细配置如下

1.  `upstream client {`

2.  `server 127.0.0.1:3002;`

3.  `}`

4.  `server{`

5.  `listen 80;`

6.  `       server_name www.xiaohuochai.cc xiaohuochai.cc;`

7.  `   return 301 https://www.xiaohuochai.cc$request_uri;`

8.  `}`

9.  `server{`

10.  `listen 443 http2;`

11.  `       server_name www.xiaohuochai.cc xiaohuochai.cc;`

12.  `       ssl on;`

13.  `ssl_certificate /home/blog/client/crt/www.xiaohuochai.cc.crt;`

14.  `ssl_certificate_key /home/blog/client/crt/www.xiaohuochai.cc.key;`

15.  `ssl_session_timeout 5m;`

16.  `       ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;`

17.  `ssl_protocols TLSv1 TLSv1.1 TLSv1.2;`

18.  `       ssl_prefer_server_ciphers on;`

19.  `   if ($host = 'xiaohuochai.cc'){`

20.  `rewrite ^/(.*)$ http://www.xiaohuochai.cc/$1 permanent;`

21.  `   }`

22.  `location / {`

23.  `expires 7d;`

24.  `add_header Content-Security-Policy "default-src 'self' https://static.xiaohuochai.site; connect-src https://api.xiaohuochai.cc; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://static.xiaohuochai.site ; img-src 'self' data: https://pic.xiaohuochai.site https://static.xiaohuochai.site; style-src 'self' 'unsafe-inline' https://static.xiaohuochai.site; frame-src https://demo.xiaohuochai.site https://xiaohuochai.site https://www.xiaohuochai.site;";`

25.  `       proxy_set_header X-Real-IP $remote_addr;`

26.  `               proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;`

27.  `proxy_set_header Host $http_host;`

28.  `               proxy_set_header X-Nginx-Proxy true;`

29.  `               proxy_pass http://client;`

30.  `               proxy_redirect off;`

32.  `   }`

33.  `}`


 
