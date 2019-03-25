---
layout: post
title: linux下重启weblogic（关闭和启动）
category: nginx
tags: [nginx]
---





# [linux下重启weblogic（关闭和启动）](https://www.cnblogs.com/yadongliang/p/8097433.html)

本文转自:[http://blog.sina.com.cn/s/blog_4b5bc011010110nq.html](http://blog.sina.com.cn/s/blog_4b5bc011010110nq.html)

ssh远程连接Linux服务器！

 开启weblogic：

 1、找到/Oracle/Middleware/user_projects/domains/用户_domain目录，

 2、执行nohup ./startWebLogic.sh &（&的作用是让weblogic启动在后台运行），

 3、使用命令tail -f 文本文件名即可监视远程文件的变动情况，例如要监视Weblogic某一域的日志输出只需要输入命令：tail -f nohup.out

 停止weblocgic：

 命令 ./stopWebLogic 一般情况很难关闭，需要杀掉后台进程（经常这样）

 查看后台进程
    
     ＃ps -ef|grep weblogic 如：
    
     root    28596 28558  2 16:10 pts/1    00:00:18 /opt/Oracle/Middleware/jr。。。。。。。。。
    
     root    28880 28778  0 16:22 pts/2    00:00:00 grep weblogic

 杀后台进程 ：# kill -9 28596 即可

