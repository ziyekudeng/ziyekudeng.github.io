---
layout: post
title: Jboss rules规则引擎 Drools 6.4.0 Final 教程(1)
category: drools
tags: [drools]
---



# 前言

目前世面上中文的KIE DROOLS Workbench（JBOSS BRMS）的教程几乎没有，有的也只有灵灵碎碎的使用机器来翻译的（翻的不知所云）或者是基于老版本的JBOSS Guvnor即5.x的一些教程，而且这些教程都是”缺胳膊少腿“的，初学者看后不知道它到底在干吗？能干吗？能够解决自己系统中什么问题。

所以笔者自己写了几个例子，把整个最新的英文版的KIE DROOLS 6.3.0.Final的官方教程给串了起来，用于供读者使用并以此来作为入门以及相关SOA理念的推广的第一步。

本教程共分为”三“集。

# <a name="t1"></a>什么是规则引擎

![](https://img-blog.csdn.net/20160413153536700?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](https://img-blog.csdn.net/20160413153705659?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**规则是让业务人士驱动整个企业过程的最佳实践**

![](https://img-blog.csdn.net/20160413153828285?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t2"></a>业务规则在实现上的矛盾

![](https://img-blog.csdn.net/20160413154035457?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t3"></a>业务规则技术

![](https://img-blog.csdn.net/20160413154231600?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t4"></a>引入业务规则技术的目的

**对系统的使用人员**

*   把业务策略（规则）的创建、修改和维护的权利交给业务经理
*   提高业务灵活性
*   加强业务处理的透明度，业务规则可以被管理
*   减少对IT人员的依赖程度
*   避免将来升级的风险

**对IT开发人员**

*   简化系统架构，优化应用
*   提高系统的可维护性和维护成本
*   方便系统的整合
*   减少编写“硬代码”业务规则的成本和风险

## <a name="t5"></a>何为规则引擎

*   可以将一个或多个的事实映射到一个或多个规则上
*   接受数据输入，解释业务规则，并根据业务规则做出业务决策

### <a name="t6"></a>一个简单的例子

![](https://img-blog.csdn.net/20160413154457541?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t7"></a>从IT技术人员的角度看为什么使用规则引擎?

*   从应用逻辑和数据中将业务逻辑分离
*   简单！ -规则有一个非常简单的结构
*   让业务用户开发和维护规则以降低成本
*   声明式编程
*   性能和可伸缩性
*   解决复杂的和复合的问题，其中有大量细粒度的规则和事实互动

## <a name="t8"></a>DEMO-人寿新卓越变额万能寿险投保规则

![](https://img-blog.csdn.net/20160413154637600?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t9"></a>DEMO-人寿新卓越变额万能寿险投保规则的IT实现

免体检累积最高限额表在规则引擎中的实现：

![](https://img-blog.csdn.net/20160413154733241?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# <a name="t10"></a>什么叫BRMS

## <a name="t11"></a>

## <a name="t12"></a>什么是BRMS-考虑两个问题（IT管理者角度）

![](https://img-blog.csdn.net/20160413154833398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t13"></a>什么是BRMS-考虑两个问题（开发人员易用性角度）

![](https://img-blog.csdn.net/20160413154908124?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t14"></a>BRMS-Business Rules Management System

![](https://img-blog.csdn.net/20160413155004492?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t15"></a>一个优秀的BRMS应该具有的特点

![](https://img-blog.csdn.net/20160413155035860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t16"></a>BRMS中两个重要的概念：因子、公式

![](https://img-blog.csdn.net/20160413155117540?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t17"></a>从业务的角度看因子与公式间的关系

![](https://img-blog.csdn.net/20160413155150727?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t18"></a>从IT的角度看因子与公式间的关系

![](https://img-blog.csdn.net/20160413155225832?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t19"></a>基于BRMS的系统逻辑架构

这个逻辑图有点复杂，很多人看了都会感觉“不知所云”，OK，不急！我们在后文中会来“回溯”它。

![](https://img-blog.csdn.net/20160413155404428?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# <a name="t20"></a>JBOSS Drools & Guvnor

世面上成熟的规则引擎有很多，著名的如：IBM 的iLog，pegga rulz（飞马），我们在这边要介绍的也是开源中最著名的jboss rulz。

Jboss Rulz最早是只有基于.drools的规则文件的一个内嵌式规则引擎，后来它发展成了“规则管理系统”即BRMS，它的BRMS被称为Guvnor。后来在JBOSS Guvnor5.x后它又改名叫"KIE Drools WorkBench“。

![](https://img-blog.csdn.net/20160413160619023?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](https://img-blog.csdn.net/20160413160639507?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

目前世面上中文的KIE DROOLS Workbench（JBOSS BRMS）的教程几乎没有，有的也只有灵灵碎碎的使用机器来翻译的（翻的不知所云）或者是基于老板的JBOSS Guvnor即5.x的一些教程，而且这些教程都是”缺胳膊少腿“的，初学者看后不知道它到底在干吗？能干吗？能够解决自己系统中什么问题。

所以笔者自己写了几个例子，把整个最新的英文版的KIE DROOLS 6.3.0.Final给串了起来，用于供读者使用并以此来作为入门SOA理念的推广的第一步。

## <a name="t21"></a>Guvnor核心功能-最好的开源规则引擎

![](https://img-blog.csdn.net/20160413160711639?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

# <a name="t22"></a>KIE Drools6.3.0.Final的安装与使用

## <a name="t23"></a>准备安装文件与环境-环境

*   CentOS 6
*   mysql5.5.x or above
*   apache-tomcat-7.0.67.zip（Tomcat7.0.4 or above）

这些环境，读者应该自己会安装了。

## <a name="t24"></a>准备安装文件与环境-必须软件

*   kie-drools-wb-6.3.0.Final-tomcat7.war
*   drools-distribution-6.3.0.Final.zip
*   给Tomcat7的lib目录下用的jar文件，其中包括：

| jboss-jacc-api_1.4_spec-1.0.3.Final.jar |
| kie-tomcat-integration-6.3.0.Final.jar |
| slf4j-log4j12-1.7.7.jar |
| log4j-core-2.1.jar |
| log4j-api-2.1.jar |
| log4j-slf4j-impl-2.1.jar |
| slf4j-api-1.7.7.jar |
| javax.security.jacc-api-1.5-javadoc.jar |
| btm-2.1.4.jar |
| btm-tomcat55-lifecycle-2.1.4.jar |
| jta-1.1.jar |
| 数据库驱动（mysql-connector-java-5.1.38.jar） |

 以上12个依赖文件如果读者一时搜不到，不要紧我都把它们上传在此了：[droos6.3.0在tomcat布署时的缺失包.zip](https://download.csdn.net/detail/lifetragedy/9491249)

## <a name="t25"></a>开始安装

### <a name="t26"></a>1\. 把下列文件全部copy至tomcat的lib目录下

 ![](https://img-blog.csdn.net/20160413163210854?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### <a name="t27"></a>2\. 打开eclipse后按照Help->install new software输入以下地址

 **[plain]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  https://download.jboss.org/drools/release/6.3.0.Final/org.drools.updatesite/

![](https://img-blog.csdn.net/20160413163437526?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### <a name="t28"></a>3\. 把drools-distribution-6.3.0.Final.zip解压在当前目录

![](https://img-blog.csdn.net/20160413163543375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### <a name="t29"></a>
4\. 把kie-drools-wb-6.3.0.Final-tomcat7.war解压在当前目录

 并改名成kie-drools后拷贝入tomcat的webapps目录下。

### <a name="t30"></a>
5\. 修改D:\tomcat7\webapps\kie-drools\WEB-INF\classes\META-INF目录下的persistence.xml文件

 ![](https://img-blog.csdn.net/20160413163816987?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 把该项目原来使用的**H2Dialect**改成**MySQL5Dialect**

 **[plain]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  <properties>
2.  <!--property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/-->
3.  <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
4.  <property name="hibernate.max_fetch_depth" value="3"/>
5.  <property name="hibernate.hbm2ddl.auto" value="update"/>
6.  <property name="hibernate.show_sql" value="false"/>
7.  <property name="hibernate.transaction.manager_lookup_class" value="org.hibernate.transaction.BTMTransactionManagerLookup"/>
8.  <!-- BZ 841786: AS7/EAP 6/Hib 4 uses new (sequence) generators which seem to cause problems -->
9.  <property name="hibernate.id.new_generator_mappings" value="false"/>
10.  </properties>

### <a name="t31"></a>6\. 在tomcat的conf目录下增加一个文件名为：btm-config.properties的文件

 使内容如下：

 **[plain]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  bitronix.tm.serverId=tomcat-btm-node0
2.  bitronix.tm.journal.disk.logPart1Filename=${btm.root}/work/btm1.tlog
3.  bitronix.tm.journal.disk.logPart2Filename=${btm.root}/work/btm2.tlog
4.  bitronix.tm.resource.configuration=${btm.root}/conf/resources.properties

### <a name="t32"></a>7\. 在tomcat的conf目录下增加一个文件名为：resources.properties的文件

 **[plain]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  resource.ds1.className=bitronix.tm.resource.jdbc.lrc.LrcXADataSource
2.  resource.ds1.uniqueName=jdbc/jbpm
3.  resource.ds1.minPoolSize=10
4.  resource.ds1.maxPoolSize=20
5.  resource.ds1.driverProperties.driverClassName=com.mysql.jdbc.Driver
6.  resource.ds1.driverProperties.url=jdbc:mysql://192.168.0.101:3306/drools?useUnicode=true&characterEncoding=UTF-8
7.  resource.ds1.driverProperties.user=kie
8.  resource.ds1.driverProperties.password=aaaaaa
9.  resource.ds1.allowLocalTransactions=true

### <a name="t33"></a>8\. 在tomcat的conf目录下修改context.xml

 增加如下内容：

 **[html]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  <Resource name="jdbc/jbpm" auth="Container" type="javax.sql.DataSource"
2.  driverClassName="com.mysql.jdbc.Driver"
3.  url="jdbc:mysql://192.168.0.101:3306/drools?useUnicode=true&characterEncoding=UTF-8"
4.  username="kie"
5.  password="aaaaaa"
6.  maxActive="20"
7.  maxIdle="1"
8.  maxWait="5000" />

### <a name="t34"></a>9\. 在tomcat的conf目录下修改server.xml

增加如下内容：

 **[html]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  <Valve className="org.kie.integration.tomcat.JACCValve" />

**记得一定要在</host>上部加入**

### <a name="t35"></a>10\. 在tomcat的conf目录下修改tomcat-users.xml

 增加如下内容：

 **[html]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  <user username="tomcat" password="tomcat" roles="admin,manager,manager-gui"/>

该用户用于访问drools kie

### <a name="t36"></a>11\. 在mysql中建立一个schema，名为drools

 并为该schema分配一个用户，该用户如果是通过远程访问mysql记得该用户要建成%（或者是username@ip地址）这样的格式，因为drools在第一次运行时会通过JPA在相应的DB内建立39张表。

 ![](https://img-blog.csdn.net/20160413164710250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### <a name="t37"></a>12\. 修改tomcat目录bin下的catalina.sh文件

 增加如下内容

 **[plain]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  export CATALINA_HOME="/opt/tomcat1"
2.  export CATALINA_OPTS="-Dbtm.root=$CATALINA_HOME \
3.  -Dbitronix.tm.configuration=$CATALINA_HOME/conf/btm-config.properties \
4.  -Djbpm.tsr.jndi.lookup=java:comp/env/TransactionSynchronizationRegistry \
5.  -Djava.security.auth.login.config=$CATALINA_HOME/webapps/kie-drools/WEB-INF/classes/login.config \
6.  -Dorg.jboss.logging.provider=jdk"
7.  export JAVA_OPTS="-d64 -server -showversion -Xms1024m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=256m -XX:CICompilerCount=8 -XX:+UseCompressedOops -XX:-DontCompileHugeMethods -Xss256k -XX:+AggressiveOpts -XX:+UseBiasedLocking -XX:MaxTenuringThreshold=31 -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:+UseFastAccessorMethods -Djava.awt.headless=true -XX:+UseCMSInitiatingOccupancyOnly -XX:+UseGCOverheadLimit -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:MaxGCPauseMillis=200 -Dorg.kie.demo=false"

 **注意：**

1\. 此处我的tomcat是放在CentOS的/opt/tomcat1下的，因此我的CATALINA_HOME的设置要换成你的tomcat所在的路径

2\. 参数 –Dorg.kie.demo=false的作用是在无互联网环境下去运行kie-drools时，如果不加此参数kie-drools会在每次运行时去GIT试图加载kie-drools的demo，如果你的服务器为虚拟机或者是无互联网环境时它会因为建立internet连接超时而抛出一个疑似memory leak的exception而导致整个war工程加载失败。

 3\. \ 这个符号的前后都要有空格，同时每行启始处也有有空格，这个符号的作用是在LINUX的CONSOLE界面中一行太长了，分成多行写但可以连成一行执行的作用。

## <a name="t38"></a>启动

 在tomcat/bin目录下键入： ./catalina.sh start启动tomcat，在tomcat/logs目录下观察日志文件：

 ![](https://img-blog.csdn.net/20160413165027425?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

打开一个IE，输入如下地址：[https://192.168.0.101:8080/kie-drools](https://192.168.0.101:8080/kie-drools)即可看到kie-drools的主界面了

## <a name="t39"></a>KIE的使用

 使用tomcat/tomcat（在tomcat-users.xml文件中配置的具有admin/analyst角色的用户即可登录)

 ![](https://img-blog.csdn.net/20160413165130486?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## <a name="t40"></a>
新建一个Project

 ![](https://img-blog.csdn.net/20160413170952498)

 ![](https://img-blog.csdn.net/20160413171012978)‘

 可以看到，在KIE-DROOLS里的project其实就是一个maven工程

 如果在新建Project时碰到让你必须"Select a repository”，请按照下面步骤操作

 ![](https://img-blog.csdn.net/20160415125635539?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 ![](https://img-blog.csdn.net/20160415125701790?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## <a name="t41"></a>在project里新建一条规则

 为了练习，我们将新建一条规则，这条规则很简单:

假设有一报销流程，需要经过部门经理审批后到财务，如果员工的报销经额大于5,000那么除部门经理需要审批外还要报总经理再审批。

这是一条业务规则，假设哪天总经理说“大于5,000就要我批，我太烦了，改成大于10,000块才需要我审核吧”，想一下我们传统的做法。

 ![](https://img-blog.csdn.net/20160413171107509)

### <a name="t42"></a>利用KIE-DROOLS书写规则-建立因子

 该规则涉及到一个公式 money>X
该规则涉及到一个因子，money

按照上述思想，我们先建立因子

 ![](https://img-blog.csdn.net/20160413171153672)

 这个Data Object就是因子

 ![](https://img-blog.csdn.net/20160413171220947)

 你可以在eclipse里把这个POJO写好后直接复制到KIE-DROOLS的Data Object编辑界面中去：

 **[java]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  package org.sky.threshholdrulz;

3.  public class PaymentInfo implements java.io.Serializable {

5.  static final long serialVersionUID = 1L;

7.  public PaymentInfo() {
8.  }

10.  private int moneyAmount = 0;
11.  private String decisionPath = "";

13.  public void setMoneyAmount(int amount) {
14.  this.moneyAmount = amount;
15.  }

17.  public int getMoneyAmount() {
18.  return this.moneyAmount;
19.  }

21.  public void setDecisionPath(String path) {
22.  this.decisionPath = path;
23.  }

25.  public String getDecisionPath() {
26.  return this.decisionPath;
27.  }
28.  }

![](https://img-blog.csdn.net/20160413171331752)

 不要忘了点”SAVE“按钮。

### <a name="t43"></a>利用KIE-DROOLS书写规则-规则

 ![](https://img-blog.csdn.net/20160413171426019?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 规则内容如下：

 **[plain]** [view plain](https://blog.csdn.net/lifetragedy/article/details/51143914 "view plain") [copy](https://blog.csdn.net/lifetragedy/article/details/51143914 "copy") [![在CODE上查看代码片](https://code.csdn.net/assets/CODE_ico.png)](https://code.csdn.net/snippets/1646176 "在CODE上查看代码片")[![派生到我的代码片](https://code.csdn.net/assets/ico_fork.svg)](https://code.csdn.net/snippets/1646176/fork "派生到我的代码片")

1.  package org.sky.threshholdrulz;

3.  no-loop

5.  rule "approval decision by general manager"
6.  when
7.  m : PaymentInfo( moneyAmount>5000 );
8.  then
9.  modify (m) { setDecisionPath("GM") };
10.  end

12.  rule "approval decision by manager"
13.  when
14.  m : PaymentInfo( moneyAmount<=5000 );
15.  then
16.  modify (m) { setDecisionPath("M") };
17.  end

### <a name="t44"></a>利用KIE-DROOLS书写规则-测试规则

 我们这条规则其实很简单：

如果PaymentInfo因子中的paymentAmount>5000，那么PaymentInfo中的decisionPath返回就是字符串“GM”。

如果PaymentInfo因子中的paymentAmount<=5000，那么PaymentInfo中的decisionPath返回就是字符串“M”。

![](https://img-blog.csdn.net/20160413171528714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### <a name="t45"></a>利用KIE-DROOLS书写规则-向导式的Test Scenario

![](https://img-blog.csdn.net/20160413171559370?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### <a name="t46"></a>利用KIE-DROOLS书写规则-给测试用例创建数据

 ![](https://img-blog.csdn.net/20160413171628167?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171644413?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171700226?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171717590?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### <a name="t47"></a>利用KIE-DROOLS书写规则-运行测试

 ![](https://img-blog.csdn.net/20160413171751434?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## <a name="t48"></a>使用JAVA程序调用规则-规则存本地

 ![](https://img-blog.csdn.net/20160413171835246?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171851122?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171908403?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171927184?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 ![](https://img-blog.csdn.net/20160413171946464?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# <a name="t49"></a>结束本次教程

 后面的教程会讲述如何把.drools文件建到我们自己搭建的kie-drools workbench 6.3.0.Final上，然后用JAVA代码远程访问的内容，这块内容目前在国内的博文和论坛中几乎无资料，笔者也是通过看源码和看官方文档后总结出来的。

 其远程访问共分为两种：stateful（有状态）和stateless（无状态）2种，文中也会给出相应的对比和讲解。
