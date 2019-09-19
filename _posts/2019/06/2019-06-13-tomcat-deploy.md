---
layout: post
title: Tomcat 部署知识
category: tools
tags: [tools]
---

## Context标签

Context标签可用于Tomcat部署web项目，配置项目信息，设置项目的浏览器访问路径，让项目修改后自动重新编译部署等。

### 如何使用：

找到tomcat安装目录的conf/server.xml文件，在server.xml里的标签中加入：


### 属性涵义：

#### path表示浏览器访问的地址路径

例如：



浏览器访问地址为：”[https://IP](https://IP)地址或域名:端口/test”
paht可以为空字符串，为空字符串时表示此项目为Tomcat默认的项目。
例如：



浏览器访问地址为：”[https://IP](https://IP)地址或域名:端口”
path的值”test”和”/test”和”/test/”都是等同的配置一样的效果。

#### docBase表示本地项目WebRoo绝对路径

docBase除了可以为本地项目WebRoot绝对路径，也可以是相对Tomcat的webapps目录的成品项目（可以理解为“war文件解压后的项目”）路径。
例如：
Tomcat的webapps目录下有一个成品项目，项目的文件夹为“Test”，那么可以有以下写法：



浏览器访问地址为：”[https://IP](https://IP)地址或域名:端口/test”

#### reloadable表示项目修改时是否自动重新编译和装载项目。

也就是如果为true，你可以不用每次修改代码后都在eclipse上重新部署，或者说不用修改java类文件之后重启tomcat。

## 常用脚本

### 部署

    rm -rf /home/show/webapps/backmanage/backmanage
    echo "remove application finish"
    unzip /home/show/webapps/backmanage/backmanage.zip -d /home/show/webapps/backmanage/backmanage

### 启动

    TOMCAT_HOME="/home/show/tomcat/apache-tomcat_7004_backmanage"
    JAVA_HOME="/home/show/jdk1.7.0_80"
    export JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m "
    sh ${TOMCAT_HOME}/bin/startup.sh
    echo "start server finish"


### 停止

    pid=`ps -ef | grep tomcat | grep 7004 | grep java | awk '{print $2}'`
    echo $pid
    #循环所有的pid执行kill -9的命令
    for id in $pid
    do
      kill -9 $pid
      echo "killed$pid"  
    done
    #结束 杀掉进程
    echo "杀掉进程"


### 日志

    TOMCAT_HOME="/home/show/tomcat/apache-tomcat_7004_backmanage"
    tail -f ${TOMCAT_HOME}/logs/catalina.out



