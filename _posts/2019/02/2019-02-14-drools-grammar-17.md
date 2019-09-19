---
layout: post
title: drools6.3+spring+Drools Workbench+activemq搭建风险控制系统
category: drools
tags: [drools]
---
根据业务的需求，从2015年10月份开始模式搭建一个风险控制系统，用于对实时交易的实时控制和非实时监控。当时上网搜索了一下，发现一个不错的框架drools，然后耗费了三个月，终于把drools引入到风控系统中。期间一路遇到的问题无数，如果是新手去搭建drools，估计至少折腾1-2个月。系统最终实现了在一个业务管控台中根据业务配置业务规则，生成决策表和规则的drl文件，然后通过drools workbench控制台构建项目和部署jar包方式发布规则。风控系统通过drools的maven机制，通过maven 的gav方式间隔一定时间扫描kie-workbench，最终实现动态更新drl规则，实现更新业务规则不用停机的目标。

系统采用spring4.2.4、drools6.3、activemq、drools workbench实现。业务逻辑如下：实时交易系统通过MQ异步或同步方式，发送实时交易数据到风控系统。风控系统调用drools框架，判断交易数据是否满足风控监管要求。如果发现，则在drl规则文件中调用相应的spring服务，记录违反风控的案例。

其中的难点有：

1、通过maven pom.xml引入drools的jar包

2、风控系统动态刷新drools风控规则、风控参数；

3、风控系统中的规则drl能够调用spring的服务，调用数据库层完成记录功能；

4、drools决策表的使用；

5、决策表转为drl文件；

6、drl文件打成jar包实现动态部署，风控系统更新规则不需要重启；

7、部署kie-workbench,drools workbench使用；

8 、drools与spring集成；

9、eclipse中安装drools插件；

10、xsi:schemaLocation引入的麻烦。

11、spring无法启动，报java.lang.NoSuchMethodError: com.google.inject.Binder.bindListener

**第一章：关于drools**

drools官网：https://drools.org/

从这里可以下载最新的drools版本和drools workbench。我在这里只下载了drools workbench。实际是把drools workbench当成一个drl规则的编辑、管理、maven版本服务器。

drools的相关jar包通过在项目中的pom引入：
    
    <dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-core</artifactId>
    <version>6.3.0.Final</version>
    </dependency>
    <dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-decisiontables</artifactId>
    <version>6.3.0.Final</version>
    </dependency>
    <dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-api</artifactId>
    <version>6.3.0.Final</version>
    </dependency>
    <dependency>
    <groupId>org.drools</groupId>
    <artifactId>drools-compiler</artifactId>
    <version>6.3.0.Final</version>
    </dependency>
    <dependency>
    <groupId>org.drools</groupId>
    <artifactId>knowledge-api</artifactId>
    <version>6.3.0.Final</version>
    </dependency>
    <dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-spring</artifactId>
    <version>6.3.0.Final</version>
    </dependency>
    <dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-ci</artifactId>
    <version>6.3.0.Final</version>
    </dependency>

通过pom引入drools的jar包，本来是一件很简单的事情，但就是下载不下来，折腾了几天，更换了JRE中jce等安全的jar包，最终下载下来。

但放上jenkins服务器后，始终还是不能下载，于是拼命google，最终发现老外们的谈话中说到一个maven的bug。

果断让公司的同事更新了jenkins对应的nexus公司私服对应maven版本，最终解决了该问题。

原因是maven某个版本在解析依赖路径时，不支持jboss的某种文件方式。该问题也折腾了很久。

**第二章：安装eclipse的drools插件。**

这个插件在eclipse的直接更新即可。自行搜网上的攻略，这个问题不太。安装完这个插件后，就可以创建drools项目、debug drl程序、编辑drl文件等。

貌似还可以创建决策表之类，这个没有试。

**第三章：编写drl文件。**

drools drl文件是一种类java语言，基本规则是：

    package com.test.risk.epay.vo.risk;
    //generated from Decision Table
    import com.test.risk.xxx.RiskRecordService;
    import com.test.common.utils.RiskEcoUtils;
    global RiskRecordService riskRecordService;
    
    // rule values at C13, header at C8
    rule "Epay Base Risk Rule_13"
    when
    $epayCommonEcoRiskVo:EpayCommonEcoRiskVo(passwd_error_cnt == "5", open_cancel_cnt == "5", pay_passwd_mod_cnt == "5", find_passwd_cnt == "5", bank_card_mod_cnt == "5", phoneNo_mod_cnt == "5")
    
    then
    riskRecordService.insertKieRecord($epayCommonEcoRiskVo,"base_00000001");
    end

需要定义跟java类似的package,import 类，还有可以定义global的参数。其中EpayCommonEcoRiskVo是放到package com.test.risk.epay.vo.risk的类。

**第四章：安装drools workbench**

从drools官网下载tomcat7版本的Drools Workbench,实际就是一个war包，需要严格按照里面的readme的要求，配置好tomcat才可以运行起来。我尝试安装在本地的windows环境，但总是不成功，最后找了一台linux主机安装好了。具体步骤：通过maven可以下载相关的jar包，然后扔到tomcat的lib目录。需要按要求修改tomcat的一些配置。readme没有要求修改h2中sa的默认密码，但我多次尝试后，发现必须设置一个密码。可以通过命令：

java -cp h2-1.3.161.jar org.h2.tools.Shell -url jdbc:h2:~/jbpm2 -user sa -password 123456创建sa的账号密码，然后修改resources.properties中的sa的密码。 

特别注意一点：由于资源有限，我尝试把风控系统与drools workbench部署到同一台机器，这个会出现问题。理由是：风控系统需要依赖本地的maven服务，需要配置~/.m2目录下的settting.xml，需要把setting.xml的url指向drools workbench的机器。但如果drools workbench启动时，它也会检查本地的setting.xml，而这个setting.xml需要指向自身，最终会导致drools workbench无法启动。因此：drools workbench必须部署在一台远程机器上。~/.m2/setting.xml配置如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <settings xmlns="https://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="https://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
    <server>
    <id>guvnor-m2-repo</id>
    <username>domcat</username>
    <password>xxxxxx</password>
    <configuration>
    <wagonProvider>httpclient</wagonProvider>
    <httpConfiguration>
    <all>
    <usePreemptive>true</usePreemptive>
    </all>
    </httpConfiguration>
    </configuration>
    </server>
    </servers>
    <profiles>
    <profile>
    <id>guvnor-m2-repo</id>
    <repositories>
    <repository>
    <id>guvnor-m2-repo</id>
    <name>BRMS Repository</name>
    <url>https://xxx.xxx.xxx.xxx:8080/guvnor/maven2/</url>
    <layout>default</layout>
    <releases>
    <enabled>true</enabled>
    <updatePolicy>always</updatePolicy>
    </releases>
    <snapshots>
    <enabled>true</enabled>
    <updatePolicy>always</updatePolicy>
    </snapshots>
    </repository>
    </repositories>
    <activation>
    <activeByDefault>true</activeByDefault>
    </activation>
    </profile>
    </profiles>
    <activeProfiles>
    <activeProfile>guvnor-m2-repo</activeProfile>
    </activeProfiles>
    </settings> 

其中上述风控系统中主机的setting.xml中的账号和密码就是drools workbench所在tomcat的tomcat-user的配置账号密码，也是drools workbench的登录账号密码。

**第五章：spring与drools6.3集成**

网上大部分的描述都是spring与drools5版本的集成。drools6版本出来后，在更新drl规则上做了很大的改动，实现了根据类似maven的groupId、artifactId、version更新规则。网上资料甚少，加上官网的文档也说得不够清楚，最后也折腾了一番

首先，在风控系统和drools workbench所在的linux主机不需要安装maven，也不需要设置maven环境变量，但需要有~/.m2及下面的setting.xml，setting.xml参考上面。

在spring的applicationContext.xml中，加入：

    <kie:import releaseId-ref="ksession-releaseId" enableScanner="true" scannerInterval="300000"/>
    <kie:releaseId id="ksession-releaseId" groupId="com.test" artifactId="epay-risk" version="LATEST"/>
    
    <bean id="kiePostProcessor"
    class="org.kie.spring.annotations.KModuleAnnotationPostProcessor"/> 

引入的目的是实现动态刷新规则jar包的规则。配置的时间是5分钟刷新一遍，最终实现业务规则的更新，不需要重启机器。加入以上配置后，eclipse报错，无法辨认kie:import等标签，需要在xml中加入：xmlns:kie="https://drools.org/schema/kie-spring" 还有：https://drools.org/schema/kie-spring https://raw.githubusercontent.com/droolsjbpm/droolsjbpm-integration/master/kie-spring/src/main/resources/org/kie/spring/kie-spring-6.0.0.xsd

这里碰到一个非常烦人的问题，就是kie-spring-6.0.0.xsd无法下载，想尝试使用classpath的kie-spring-6.0.0.xsd，但不成功。官方正式xsd的地址无效，最后使用上面的github的地址，不过该地址有时候会发傻，用不了。

在spring 的serviceImpl中，加入：
    
    /**
    *使用kie-spring注入服务
    */
    @KSession("defaultKieSession")
    @KReleaseId( groupId = "com.test", artifactId = "epay-risk", version = "LATEST")
    private KieSession kSession; 

完成kieSession的初始化。需要注意的是：配置的version是LATEST，一旦每隔5分钟扫描一次的kieScan侦查到有新的版本后，就会自动更新为最新版本。按这样的LATEST写，每次都需要在drools workbench发布增量的新的版本号，不能使用固定的版本号。如果使用固定的版本号，采用SNAPSHOP方式实现。

spring启动时，会自动到drools workbench服务器下载对应的jar包，并且每五分钟到drools workbench扫描一遍，看看有没有最新的jar包版本。如果有，则下载到本地的~/.m2目录下。

测试：在drools workbench中更新drl规则，更新jar包的版本，部署jar包后，风控系统每5分钟（可以配置）扫描到有新的版本，将完成更新，然后按新的业务风控规则运行。

**第六章：drl调用spring服务**

    drl的规则如下：
    
    package com.test.risk.epay.vo.risk;
    //generated from Decision Table
    import com.test.risk.xxx.RiskRecordService;
    import com.test.common.utils.RiskEcoUtils;
    global RiskRecordService riskRecordService; 
    
    // rule values at C13, header at C8
    rule "test Epay Base Risk Rule_13"
    when
    $epayCommonEcoRiskVo:EpayCommonEcoRiskVo(passwd_error_cnt == "5", open_cancel_cnt == "5", pay_passwd_mod_cnt == "5", find_passwd_cnt == "5", bank_card_mod_cnt == "5", phoneNo_mod_cnt == "5")
    then
    riskRecordService.insertKieRecord($epayCommonEcoRiskVo,"base_00000001");
    end

其中注意一点riskRecordService是spring的一个服务。那么怎么让drl认识到riskRecordService这个spring服务呢？折腾半天，最终找到方法：

在ksession初始化时，通过kSession.setGlobal("riskRecordService", riskRecordService);设置：

    /**
    *使用kie-spring注入服务
    */
    @KSession("defaultKieSession")
    @KReleaseId( groupId = "com.test", artifactId = "epay-risk", version = "LATEST")
    private KieSession kSession;
    
    /**
    * 数据库层
    */
    @Autowired
    RiskRecordService riskRecordService; 
    
    /**
    * 初始时完成
    */
    @PostConstruct
    public void init(){
    
    //当需要读取classpath时，放开注释。
    /*
    KieServices ks = KieServices.Factory.get();
    KieContainer kContainer = ks.getKieClasspathContainer();
    kSession = kContainer.newKieSession("ksession-rules");*/
    kSession.setGlobal("riskRecordService", riskRecordService);
    } 

通过setGlobal的方式可以让drl认得spring的服务，从而可以在drl规则中调用spring服务。

**第七章：drools workbench的规则编写**

本来以为drools workbench编写drl文件是一件简单的事情，但当我把drl文件写进去时，却发现无法编译通过，也就是说import进来的类必须存在。我觉得非常奇怪,drl文件中import来的类是存在于spring的项目中，编译drl时不应该检查import的类是否存在drools workbench。但是，规则是人家定的，最终的解决方法就是把所有drl中使用的的java类封装成一个底层的jar包，然后在drools workbench的项目中通过artifact repository中安装该底层jar包，然后在drools workbench项目中通过add dependency加进来，最后编译、部署和发布jar包成功。

**第八章：关于drools决策表的使用及转化为drl文件**

建议参考官网的例子。值得注意的一点是，在drl中global RiskRecordService riskRecordService;对应决策表的写法是：variables RiskRecordService riskRecordService。通过：

    File file = new File("路径+riskRule.xls");
    InputStream is;
    try {
    is = new FileInputStream(file);
    SpreadsheetCompiler converter = new SpreadsheetCompiler();
    String drl = converter.compile(is, InputType.XLS);
    System.out.println( drl); 
    
    } catch (FileNotFoundException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
    }
    } 

可以将excel的决策表生成drl文件。

第九章：Spring无法启动。

参考我在另外网站写的两个问题：https://stackoverflow.com/questions/34916936/spring-integration-with-drools和https://stackoverflow.com/questions/34890616/kie-workbench-integration-in-spring-throw-exception

报错：

    <code style="border: 0px; font-family: Consolas, Menlo, Monaco, 'Lucida Console', 'Liberation Mono', 'DejaVu Sans Mono', 'Bitstream Vera Sans Mono', 'Courier New', monospace, sans-serif;">Caused by: java.lang.NoSuchMethodError: com.google.inject.Binder.bindListener(Lcom/google/inject/matcher/Matcher;[Lcom/google/inject/spi/ProvisionListener;)V
        at org.eclipse.sisu.plexus.PlexusBindingModule.configure(PlexusBindingModule.java:75) at com.google.inject.spi.Elements$RecordingBinder.install(Elements.java:223) at com.google.inject.spi.Elements.getElements(Elements.java:101) at com.google.inject.spi.Elements.getElements(Elements.java:92) at org.eclipse.sisu.wire.WireModule.configure(WireModule.java:75) at com.google.inject.spi.Elements$RecordingBinder.install(Elements.java:223) at com.google.inject.spi.Elements.getElements(Elements.java:101) at com.google.inject.internal.InjectorShell$Builder.build(InjectorShell.java:133) at com.google.inject.internal.InternalInjectorCreator.build(InternalInjectorCreator.java:103) at com.google.inject.Guice.createInjector(Guice.java:95) at com.google.inject.Guice.createInjector(Guice.java:72) at com.google.inject.Guice.createInjector(Guice.java:62) at org.codehaus.plexus.DefaultPlexusContainer.addPlexusInjector(DefaultPlexusContainer.java:477) at org.codehaus.plexus.DefaultPlexusContainer.<init>(DefaultPlexusContainer.java:203)</code>后面的解决方法是在pom.xml中更新：<code style="border: 0px; font-family: Consolas, Menlo, Monaco, 'Lucida Console', 'Liberation Mono', 'DejaVu Sans Mono', 'Bitstream Vera Sans Mono', 'Courier New', monospace, sans-serif;"><dependency>
            <groupId>com.google.inject</groupId>
            <artifactId>guice</artifactId>
            <version>4.0</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.aether</groupId>
            <artifactId>aether-impl</artifactId>
            <version>0.9.0.M4</version>
        </dependency></code>

总结：使用drools框架不容易，让drools6.3与spring集成实现动态更新规则非常不容易，沿途处处陷阱，网上的资料非常稀缺，出现一个问题不折腾半天解决不了。期待drools后续的版本能够继续优化。使用drools6.3和spring的童鞋可以参考下这篇文章，少走弯路。
