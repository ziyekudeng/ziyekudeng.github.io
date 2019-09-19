---
layout: post
title: 使用Nginx实现反向代理
category: tools
tags: [tools]
---



转：https://blog.csdn.net/Daybreak1209/article/details/51549031

一、代理服务器

1、什么是代理服务器

代理服务器，客户机在发送请求时，不会直接发送给目的主机，而是先发送给代理服务器，代理服务接受客户机请求之后，再向主机发出，并接收目的主机返回的数据，存放在代理服务器的硬盘中，再发送给客户机。

![](https://img-blog.csdn.net/20160531205353826?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

2、为什么要使用代理服务器

1）提高访问速度

由于目标主机返回的数据会存放在代理服务器的硬盘中，因此下一次客户再访问相同的站点数据时，会直接从代理服务器的硬盘中读取，起到了缓存的作用，尤其对于热门站点能明显提高请求速度。

2）防火墙作用

由于所有的客户机请求都必须通过代理服务器访问远程站点，因此可在代理服务器上设限，过滤某些不安全信息。

3）通过代理服务器访问不能访问的目标站点

互联网上有许多开发的代理服务器，客户机在访问受限时，可通过不受限的代理服务器访问目标站点，通俗说，我们使用的翻墙浏览器就是利用了代理服务器，虽然不能出国，但也可直接访问外网。

二、反向代理 VS 正向代理
1、什么是正向代理？什么是反向代理？

正向代理，架设在客户机与目标主机之间，只用于代理内部网络对Internet的连接请求，客户机必须指定代理服务器,并将本来要直接发送到Web服务器上的http请求发送到代理服务器中。

![](https://img-blog.csdn.net/20160531205420201?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

反向代理服务器架设在服务器端，通过缓冲经常被请求的页面来缓解服务器的工作量，将客户机请求转发给内部网络上的目标服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器与目标主机一起对外表现为一个服务器。

![](https://img-blog.csdn.net/20160531205433342?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

2、反向代理有哪些主要应用？

现在许多大型web网站都用到反向代理。除了可以防止外网对内网服务器的恶性攻击、缓存以减少服务器的压力和访问安全控制之外，还可以进行负载均衡，将用户请求分配给多个服务器。

三、方向代理服务器Nginx

Nginx作为近年来较火的反向代理服务器，安装在目的主机端，主要用于转发客户机请求，后台有多个http服务器提供服务，nginx的功能就是把请求转发给后面的服务器，决定哪台目标主机来处理当前请求。下面演示如何进行配置使Nginx发挥作用。

1、模拟n个http服务器作为目标主机

用作测试，简单的使用2个tomcat实例模拟两台http服务器，分别将tomcat的端口改为8081和8082

2、配置IP域名

192.168.72.49 8081.max.com

192.168.72.49 8082.max.com

3、配置nginx.conf

 **[html]** [view plain](https://blog.csdn.net/Daybreak1209/article/details/51549031 "view plain") [copy](https://blog.csdn.net/Daybreak1209/article/details/51549031 "copy")

1.  upstream tomcatserver1 {
2.  server 192.168.72.49:8081;
3.  }
4.  upstream tomcatserver2 {
5.  server 192.168.72.49:8082;
6.  }
7.  server {
8.  listen 80;
9.  server_name 8081.max.com;

11.  #charset koi8-r;

13.  #access_log logs/host.access.log main;

15.  location / {
16.  proxy_pass https://tomcatserver1;
17.  index index.html index.htm;
18.  }
19.  }
20.  server {
21.  listen 80;
22.  server_name 8082.max.com;

24.  #charset koi8-r;

26.  #access_log logs/host.access.log main;

28.  location / {
29.  proxy_pass https://tomcatserver2;
30.  index index.html index.htm;
31.  }
32.  }

流程：

1）浏览器访问8081.max.com，通过本地host文件域名解析，找到192.168.72.49服务器（安装nginx）

2）nginx反向代理接受客户机请求，找到server_name为8081.max.com的server节点，根据proxy_pass对应的http路径，将请求转发到upstream tomcatserver1上，即端口号为8081的tomcat服务器。

4、效果展示

请求8081.max.com，tomcat1接收返回首页

![](https://img-blog.csdn.net/20160531210753161?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

请求8082.max.com，tomcat2接收返回首页

![](https://img-blog.csdn.net/20160531210810439?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

四、总结

通过分析我们不难得出，以百度为例，如果客户机的IP和百度服务器（目标主机）的IP在同一个网段，那就形同局域网内部发送请求，速度极快。

但如果满足不了这种需求还想到达到一个较好的请求响应时，百度服务器就可以对外提供一个与目标服务器在一个网段的公网IP,也就是反向代理服务的IP，通过代理服务器转发客户机请求，决定幕后的N台服务器谁来处理这个请求，并且由于反向代理服务器与目标主机在一个网段，访问速度也会很快。

Nginx用作反向代理服务器时，它就是众多反向代理服务器中的一种，通过简单的配置，指定到服务器IP或域名地址便可将客户机请求转发给指定服务器处理请求。

