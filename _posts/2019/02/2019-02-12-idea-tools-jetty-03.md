---
layout: post
title: 【Idea】下解决IDEA使用jetty跑项目js、css文件被占用无法修改的问题
category: tools
tags: [tools]
---



用IDEA开发web项目使用maven的jetty插件跑的时候经常遇到项目启动后，无法编辑js文件和css文件。
最初以为是Idea的问题，但是这么严重的一个问题怎么就没有人注意呢？
后来又上网查了好多资料，原来才发现不是IDEA的问题，是jetty本身的问题：原因是如果NIO被支持的话，Jetty会使用内存映射文件来缓存静态文件，其中包括.js文件。在Windows下面，使用内存映射文件会导致文件被锁定。
解决方案是不使用内存映射文件来做缓存。

到maven本地仓库的org/eclipse/jetty/jetty-webapp/下，找到对应版本的jetty插件修改webdefault.xml
将：

    <code class="xml"><init-param>
        <param-name>useFileMappedBuffer</param-name>
        <param-value>true</param-value>
    </init-param></code> 

改为：

    <code class="xml"><init-param>
        <param-name>useFileMappedBuffer</param-name>
        <param-value>false</param-value>
    </init-param></code> 

即可搞定！
也可以将此文件拷贝到项目中，在jetty插件配置中引入：

    <code class="xml"><plugin>
      <groupId>org.mortbay.jetty</groupId>
      <artifactId>jetty-maven-plugin</artifactId>
      <version>7.5.1.v20110908</version>
      <configuration>
        <connectors>
          <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">
            <port>8088</port>
            <maxIdleTime>60000</maxIdleTime>
          </connector>
        </connectors>
        <systemProperties>
          <systemProperty>
            <name>org.eclipse.jetty.util.URI.charset</name>
            <value>UTF-8</value>
          </systemProperty>
        </systemProperties>
        <webAppConfig>
          <defaultsDescriptor>src/main/resources/webdefault.xml</defaultsDescriptor>
        </webAppConfig>
      </configuration>
    </plugin></code> 




 
