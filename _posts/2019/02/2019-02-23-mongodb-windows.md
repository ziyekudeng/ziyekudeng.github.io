---
layout: post
title: windows环境下启动mongodb服务
category: mongodb
tags: [mongodb]
---


# windows环境下启动mongodb服务

 2018年07月26日 17:03:53 [XCCS_澍](https://me.csdn.net/u011692780) 阅读数：1987

转载于：[https://blog.csdn.net/hh12211221/article/details/78902596](https://blog.csdn.net/hh12211221/article/details/78902596)

### 方法一

1、打开命令窗口，切换到mongodb安装目录下的“bin”目录中。

输入命令：cd E:\software\MongoDB\Server\3.4\bin

![](https://img-blog.csdn.net/20171226152417506)

2、启动服务。输入命令：”mongod --dbpath E:\software\MongoDB\data

![](https://img-blog.csdn.net/20171226152656644)

注：--dbpath是指定数据库存放目录，要注意dbpath前有两个“-”。

3、命令窗口中打印一些启动信息，则表示启动成功。如下所示：

![](https://img-blog.csdn.net/20171226153025925)

4、在浏览器中输入[http://localhost:27017/](http://localhost:27017/)即可看到显示信息为：

![](https://img-blog.csdn.net/20171226153156099)

到此为止，mongodb服务已启动成功，关闭命令窗口即可关闭mongodb服务。

### <a name="t1"></a>方法二

上述启动mongodb的方法操作不方便，每次启动否需要输入命令，因此我们需要建立一个永久性的服务，即将mongo加入到windows本地服务中。

1、打开命令窗口，切换到mongodb安装目录下的“bin”目录中。

输入命令：cd E:\software\MongoDB\Server\3.4\bin

![](https://img-blog.csdn.net/20171226152417506)

2、输入命令：mongod.exe --logpath E:\software\MongoDB\data\log\mongodb.log --logappend --dbpath E:\software\MongoDB\data --directoryperdb --serviceName MongoDB --install

![](https://img-blog.csdn.net/20171226154106286)

3、开启服务。输入命令“net start MongoDB”。（若不生效，也可以打开任务管理器，找到相关服务，手动打开）

![](https://img-blog.csdn.net/20171226160812863)

### <a name="t2"></a>问题解决

无法创建服务

问题：

若在方法二中执行第二步后，在输入命令后提示“服务名无效”或者在任务管理器中没有找到该服务，则可查看“E:\software\MongoDB\data\log”下的mongodb.log日志。

![](https://img-blog.csdn.net/20171226161139905)

原因：

cmd没哟管理员权限。

解决方法：

以管理员身份运行cmd，再重新按照方法二的操作步骤执行即可。

启动服务报错

1、问题：

启动服务报“发生系统错误 5。拒绝访问。”

![](https://img-blog.csdn.net/20171226162356053)

原因：

cmd没哟管理员权限。

解决方法：

以管理员身份运行cmd，再重新按照方法二的操作步骤执行即可。

2、问题：

启动服务报“MongoDB 服务正在启动 .MongoDB 服务无法启动。”

![](https://img-blog.csdn.net/20171226162549686)

解决方法：

在“E:\software\MongoDB\data”下找到‘mongod.lock’和‘storage.bson’这两个文件，删除重启即可。

![](https://img-blog.csdn.net/20171226162738566)

 
