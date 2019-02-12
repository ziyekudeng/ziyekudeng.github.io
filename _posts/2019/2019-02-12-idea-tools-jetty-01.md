---
layout: post
title: 【Idea】解决启动jetty后，不能修改js文件的问题
category: tools
tags: [tools]
---




禁止 Jetty 使用映射缓存：

当Jetty服务器正处于启动中时，如果想试图去修改正在被访问的资源文件，例如JS，这时候你修改完成想保存的时候，是保存不了的。
你必须停掉 Jetty 容器，才能保存。保存完成后必须重新启动 Jetty 容器才能看见效果。这样做对修改 .js文件非常麻烦和不爽。

究其原因，这是 Jetty 使用了内存映射文件来缓存静态文件。在Windows下面，使用内存映射文件会导致文件被锁定。
解决方案是不使用内存映射文件来做缓存。步骤如下：

根据所使用 Jetty 版本在本地的 maven 仓库中找到 Jetty 版本对应的jar包。

如：

     <code class=" hljs xml"><plugins>
          <plugin>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>maven-jetty-plugin</artifactId>
            <version>6.1.10</version>
          </plugin>
        </plugins></code>

则需找到( 我的本地的 maven 仓库是在 D:\Repositories\Maven )：
D:\Repositories\Maven\org\mortbay\jetty\jetty\6.1.10\jetty-6.1.10.jar

用解压缩工具打开此jar包，进到：
jetty-6.1.10.jar\org\mortbay\jetty\webapp
找到webdefault.xml文件，即： jetty-6.1.10.jar\org\mortbay\jetty\webapp\webdefault.xml
解压出此文件webdefault.xml，找到：
useFileMappedBuffer
true

将 true 改成 false，以禁止使用映射缓存。

删除原jar包中的webdefault.xml文件，将修改过的webdefault.xml文件压缩进去，OK。搞定。

这样之后就能在 Jetty 运行时修改并保存资源文件。

