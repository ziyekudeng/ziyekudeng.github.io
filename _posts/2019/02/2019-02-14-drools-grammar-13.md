---
layout: post
title: Jboss rules规则引擎 Drools 6.4.0 Final 教程(3)
category: drools
tags: [drools]
---


# Drools 6.4.0 Final

 1.新建maven项目

 ![](https://img-blog.csdn.net/20160621201643225?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

 2.pom.xml

 <code class="language-java"><?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <groupId>com.sample</groupId>
  <artifactId>Drools-test</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <name>Drools :: Sample Maven Project</name>
  <description>A sample Drools Maven project</description>

  <properties>
    <runtime.version>6.4.0.Final</runtime.version>
  </properties>

  <repositories>
    <repository>
      <id>jboss-public-repository-group</id>
      <name>JBoss Public Repository Group</name>
      <url>http://repository.jboss.org/nexus/content/groups/public/</url>
      <releases>
        <enabled>true</enabled>
        <updatePolicy>never</updatePolicy>
      </releases>
      <snapshots>
        <enabled>true</enabled>
        <updatePolicy>daily</updatePolicy>
      </snapshots>
    </repository>
  </repositories>

  <dependencies>
    <dependency>
      <groupId>org.kie</groupId>
      <artifactId>kie-api</artifactId>
      <version>${runtime.version}</version>
    </dependency>
	 <dependency>
             <groupId>org.kie</groupId>
             <artifactId>kie-spring</artifactId>
            <version>${runtime.version}</version>
      </dependency>
    <dependency>
      <groupId>org.drools</groupId>
      <artifactId>drools-core</artifactId>
      <version>${runtime.version}</version>
    </dependency>
    <dependency>
      <groupId>org.drools</groupId>
      <artifactId>drools-decisiontables</artifactId>
      <version>${runtime.version}</version>
    </dependency>

    <dependency>
      <groupId>org.jbpm</groupId>
      <artifactId>jbpm-test</artifactId>
      <version>${runtime.version}</version>
    </dependency>
  </dependencies>
</project></code> <code class="language-java">3./Drools-test/src/main/resources/META-INF/kmodule.xml</code><code class="language-java"><pre name="code" class="java"><?xml version="1.0" encoding="UTF-8"?>
<kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">
    <kbase name="rules" packages="rules">
        <ksession name="ksession-rules"/>
    </kbase>
    <kbase name="dtables" packages="dtables">
        <ksession name="ksession-dtables"/>
    </kbase>
    <kbase name="process" packages="process">
        <ksession name="ksession-process"/>
    </kbase>
</kmodule></code>

<code class="language-java">4./Drools-test/src/main/resources/rules/Sample.drl</code><code class="language-java"><pre name="code" class="java">package com.sample

import com.sample.DroolsTest.Message;

rule "Hello World"
    when
    <span style="white-space:pre">	</span>//判断status是否与常量Message.HELLO相同   赋值message到myMessage
        m : Message( status == Message.HELLO, myMessage : message )
    then
    <span style="white-space:pre">	</span>//输出myMessage
        System.out.println( myMessage );
        //更改Message
        m.setMessage( "Goodbye world" );
        m.setStatus( Message.GOODBYE );
        //更改后继续执行规则
        update( m );
end

rule "GoodBye"
    when
   <span style="white-space:pre">		</span> //判断status是否与常量Message.GOODBYE相同   赋值message到myMessage
        Message( status == Message.GOODBYE, myMessage : message )
    then
    <span style="white-space:pre">	</span>//输出myMessage
        System.out.println( myMessage );
end</code> 

<code class="language-java">5./Drools-test/src/main/java/com/sample/Message.java</code><code class="language-java"><pre name="code" class="java">package com.sample;

public class Message {

    //常量 HELLO 值0
    public static final int HELLO = 0;
    //常量 GOODBYE 值0
    public static final int GOODBYE = 1;

    //消息
    private String message;
    //状态
    private int status;

    public String getMessage() {
        return this.message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public int getStatus() {
        return this.status;
    }

    public void setStatus(int status) {
        this.status = status;
    }

}</code> 

<code class="language-java">6./Drools-test/src/main/java/com/sample/DroolsTest.java</code><code class="language-java"><pre name="code" class="java">package com.sample;

import org.kie.api.KieServices;
import org.kie.api.runtime.KieContainer;
import org.kie.api.runtime.KieSession;

/**
 * This is a sample class to launch a rule.
 */
public class DroolsTest {

    public static final void main(String[] args) {
        try {
            // load up the knowledge base
        	//得到一个KieService 
	        KieServices ks = KieServices.Factory.get();
	        //工作kieService加载默认路径下的kmodule.xml 得到kContainer场所
    	    KieContainer kContainer = ks.getKieClasspathContainer();
    	    //kContainer得到一个会话
        	KieSession kSession = kContainer.newKieSession("ksession-rules");

            // go !
            Message message = new Message();</code> <code class="language-java">message.setMessage("Hello World");
            message.setStatus(Message.HELLO);

            //将事实数据传入kSession中
            kSession.insert(message);
            //执行所有的规则
            kSession.fireAllRules();
        } catch (Throwable t) {
            t.printStackTrace();
        }
    }

}</code> 
