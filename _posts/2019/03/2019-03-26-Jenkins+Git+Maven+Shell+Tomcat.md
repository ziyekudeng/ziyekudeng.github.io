---
layout: post
title: Jenkins+Git+Maven+Shell+Tomcat
category: tools
tags: [tools]
---


 
> 
> 来源地址：segmentfault.com/a/1190000002895521

Jenkins 是一个开源软件项目，旨在提供一个开放易用的软件平台，使软件的持续集成变得可能。现在软件开发追求的是效率以及质量，Jenkins使得自动化成为可能！

**亮点**

1.采用shell自定义脚本,控制集成部署环境更加方便灵活

2.精简war包中的lib包,常驻tomcat里，减少war包传输时间

3.Jenkins 用户权限管理，不让淘气鬼乱动

4.构建失败发邮件通知相关人员解决

5.自动按天备份war包,Jenkins配置备份以及版本控制化

**环境**

Ubuntu 14.10 (GNU/Linux 3.16.0-33-generic x86_64)

**准备工作**

1.Git版本控制服务器

2.Tomcat发布服务器

3.Jenkins服务器(提前安装好Maven,Git,Jdk)

实验时可以在同一台机器配置，但是生产不建议，一台机器挂了，所有服务器都挂了。

废话不多说，直接上干货！

**步骤**

**安装Jenkins**

下载Jenkins War包，Jenkins官网 。

启动Jenkins ，将War包放入Tomcat容器里，启动Tomcat。

提示：

此时Jenkins在初始化配置目录，其默认配置目录路径为当前用户下的.jenkins目录，用户也可以自定义目录，Jenkins默认是把配置文件中的数据读到内存中，如果你替换了之前的配置文件，此时需要点击Jenkins的读取设置或者重启Tomcat,如果此时Jenkins页面无响应，则应该查看Tomcat的Catalina.out,多半是由于内存溢出造成(解决方法增大Tomcat调用Java虚拟机时内存大小,本文不做重点)，运行Jenkins的服务器配置最好内存1G以上，因为后续会加入一些Jenkins插件，有一些会比较占用内存，导致Jenkins启动不了。

**安装Jenkins插件**

1.Email Extension Plugin (邮件通知)

2.GIT plugin (可能已经默认安装了)

3.Publish Over SSH (远程Shell)

**安装方法：**

首页->系统管理->管理插件->可选插件->过滤(搜索插件名)->勾选->点击最下面直接安装即可(需要等待一段时间,详情可以看catalina.out日志变化)

**配置Jenkins**

**配置基本信息**

每个选项后都有个问号解释当前含义(此步新手可以略过，默认不填即可)

配置方法：首页->系统管理->系统设置

**配置邮件**

管理员邮件地址就是邮件的发件人地址(必须和后面邮件配置发件人邮箱一致，否则发不成功邮件)

**配置Jdk**

JAVA_HOME为Jdk路径 其中Jdk也可以从这里下载安装解压(不推荐,需要填写oracle account)

**配置 Maven**

配置 Maven Configuration

路径为maven的setting.xml路径(Maven安装略)

**配置Maven项目**

**配置Maven安装目录**

**配置 Git**

其中Path to Git executable为你git执行的路径 一般默认是/usr/bin/git ,如有差异，可以whereis git

**配置 Publish over SSH**

远程执行shell脚本 采用公钥私钥连接 其中Key里贴的是私钥 远程被管理的主机里贴的是公钥,这2台主机就是相互信任，这样scp等操作就不需要输入用户名和密码。

公钥私钥生成方法：

1.管理主机linux 上 ssh-keygen -t rsa -C "mousycoder@foxmail.com 一路回车 会在/root/.ssh下生成id_rsa(私钥) id_rsa.pub(公钥)。

2.copy 公钥的内容到远程需要通信(被管理)的主机 /root/.ssh/authorized_keys 如无此目录文件则手动创建。

配置完之后可以Test Configuration

**配置 Job**

步骤:首页->新建->构建一个maven项目(输入item名称)->进入该项目->配置

JOB基本信息

项目源码管理

Repository UR 项目地址 Credentials授权可以是SSH也可以是用户名密码(SSH方法同上)

选择需要构建的分支，我们项目采用git工作流 默认master和develop 平时开发构建develop分支，正式上线构建master并且打标签(前公司git提交标准化相当复杂，分支相当多,这里大家可以根据实际情况来)。

**构建触发器**

这里我们选择poll轮询每隔1分钟去检测git仓库代码库版本,如果有更改则立刻构建，这里大家可以根据自己团队实际情况去制定，当然还有另外一个插件gitlab-hook可以主动去通知jenkins构建,不过插件所占内存比较大，需要增大tomcat虚拟机内存配置，不然会内存溢出，个人觉得如果一个团队人很多的话，选择poll更适合并且时间间隔设置长一些，避免频繁构建,gitlab-hook 适合人很少甚至一个人的情况。

**构建命令**

我们采用最简单的clean install 当然这里可以根据各自需求

例如 部署后的产物上传到nexus等,详情参考 Maven命令

    clean install deploy:deploy-file -DgroupId=com.weitoo -DartifactId=common -Dversion=0.1-SNAPSHOT -Dpackaging=jar -Dfile=D:workspaceserver-aggregatorcommon  argetcommon-0.1-SNAPSHOT.jar -Durl=https://192.168.0.200:8081/nexus/content/repositories/thirdparty/ -DrepositoryId=thirdparty
    
    Add post-build step

构建成功后执行shell命令

该shell的目的是取出war包lib中其他所有lib包 只留下common-0.1-SNAPSHOT.jar 大大减少war包大小(完整war包30M 传包到阿里云服务器需要2分多,精简后2M，10秒多,大大提高构建速度)。

分享我的Shell

    mv ~/.jenkins/jobs/server/workspace/server/target/server/WEB-INF/lib/common-0.1-SNAPSHOT.jar ~/.jenkins/jobs/server/workspace/server/target/
    rm -rf ~/.jenkins/jobs/server/workspace/server/target/server/WEB-INF/lib/*
    rm -rf ~/.jenkins/jobs/server/workspace/server/target/server.war 
    mv ~/.jenkins/jobs/server/workspace/server/target/common-0.1-SNAPSHOT.jar ~/.jenkins/jobs/server/workspace/server/target/server/WEB-INF/lib/
    cd ~/.jenkins/jobs/server/workspace/server/target/server/
    zip -r ~/.jenkins/jobs/server/workspace/server/target/server.war * -r
    scp /root/.jenkins/jobs/server/workspace/server/target/server.war root@123.56.xxx.xx:/opt/war/

构建成功远程执行shell脚本

exec command 是远程sh的路径

**分享我的publish.sh文件**

作用是备份每次上传的war包 重启Tomcat。
    
    export JAVA_HOME=/opt/software/jdk1.7.0_25
    TOMCAT_HOME="/opt/software/apache-tomcat-7.0.59"
    TOMCAT_PORT=80
    PROJECT="server"
    BAK_DIR=/opt/war/bak/$PROJECT/`date +%Y%m%d`
    
    mkdir -p "${BAK_DIR}"
    cp /opt/war/"${PROJECT}".war "${BAK_DIR}"/"${PROJECT}"_`date +%Y%m%d%H%M%S`.war
    
    #shutdown tomcat
    /opt/sh/kill-tomcat-force.sh
    
    #publish project 
    rm -rf "${TOMCAT_HOME}"/webapps/${PROJECT}
    cp /opt/war/"${PROJECT}".war "${TOMCAT_HOME}"/webapps/${PROJECT}.war
    
    #remove tmp
    rm -rf /opt/war/${PROJECT}.war
    
    #unzip war
    unzip "${TOMCAT_HOME}"/webapps/${PROJECT}.war -d "${TOMCAT_HOME}"/webapps/${PROJECT}
    
    rm -rf "${TOMCAT_HOME}"/webapps/${PROJECT}.war
    
    ##copy lib
    cp /opt/lib/* "${TOMCAT_HOME}"/webapps/${PROJECT}/WEB-INF/lib/
    
    ## start tomcat
    
    sleep 3
    
    #start tomcat
    /opt/software/apache-tomcat-7.0.59/bin/startup.sh
    echo "tomcat is starting!"   

**分享我的kill-tomcat-force.sh文件**

作用是强制关闭tomcat进程
    
    set fileformat=unix
    
    path=/opt/software/apache-tomcat-7.0.59/bin
    
    ps -ef|grep $path|grep tomcat|awk '{print $2}'
    
    echo "exec $path/shutdown.sh"
    $path/shutdown.sh
    
    sleep 3s
    
    #kill -9 pid
    ps -ef|grep $path|grep tomcat|awk '{print $2}'|xargs kill -9
    
    #success msg
    echo "shutdown success"
    
    ps -ef|grep $path|grep java|awk '{print $2}'

**分享我的Tomcat精简方法**

1.在tomcat_home/lib下新建自定义jar包文件，导入项目所需其他jar包(以后有新增的话，单独再导一次)

2.修改tomcat_home/conf/catalina.properties 搜索=shared.loader加上路径

    shared.loader=${catalina.base}/lib/server,${catalina.base}/lib/server/*.jar,${catalina.home}/lib/server,${catalina.home}/lib/server/*.jar

此时Tomcat运行前会加载server下的lib包，如果是多个项目公用一个tomcat的时候，就需要这里放公共的lib包，避免tomcat加载多余的jar包,消耗内存。

构建后邮件设置

邮件主题收件人配置

**邮件触发器**

局部配置会覆盖掉全局配置,我们之前在全局配置里配置了构建失败邮件触发器,这里是更加精细的配置，

我们选择构建失败Failure-1st触发器，失败以后发邮件给开发者，(这里可以根据实际需要，配置，可以配置多个触发器)开发者的邮件在Recipient List里配置。

**Jenkins用户权限管理**

步骤：首页-> 系统管理-> Configure Global Security

基本配置:

只有注册的用户才能操作,当然如果是大企业的话，可以采用项目矩阵授权策略,详情可以Google。

**Jenkins配置的备份和版本控制**

很多情况下稍不注意改变了Jenkins的配置，把平台弄坏了，又想去恢复，这个时候就得把Jenkins的配置文件进行配置或者版本化，只需要把/root/.jenkins/加入git版本库里即可,该目录下包含Jenkins所有信息,包括每次构建历史信息和历史jar包

进行全备份然后覆盖掉该文件夹的时候，重新构建JOB会出现文件夹已经存在等exception，只需要手动删掉这些目录即可，不会丢失数据。(这是Jenkins的一个bug,参考 JENKINS-21330)

【推荐阅读】

[初级Java程序员需要掌握哪些主流技术才能拿20K？](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455546009&idx=1&sn=a5318af96efb926f1d20a5a97e96a670&chksm=fb9cb4f9cceb3def2e0bde8df943c477f269e40c913f46e09db9983cec39f8f67c36ec1b3789&scene=21#wechat_redirect)

[学习如何使用Shiro，从架构谈起，到框架集成！](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455546041&idx=1&sn=55c581351a85fcb357c6914565fe3ed6&chksm=fb9cb4d9cceb3dcfcc99bdcbff66358bbef8699ad428da9adf121bb0aa7d3f579c7c05c32177&scene=21#wechat_redirect)

[单点登录原理与简单实现](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455546053&idx=1&sn=c56f001614b83913ee51a0f3893270c6&chksm=fb9cb4a5cceb3db3419a263d1fd6d3028a4b21c0da2a94d16455b4144bf27604545a43ef67b0&token=1308182523&lang=zh_CN&scene=21#wechat_redirect)

（完）

