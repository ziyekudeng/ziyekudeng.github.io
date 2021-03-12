---
layout: post
title: 安装SkyWalking作为监控工具
category: tools
tags: [tools]
keywords: 监控工具
---
## **写在前面**

以下文章来源于[Java学习者社区](https://urlify.cn/Zfy2ia)


## **正文**



SkyWalking 是一个应用性能监控系统，特别为微服务、云原生和基于容器（Docker, Kubernetes, Mesos）体系结构而设计。除了应用指标监控以外，它还能对分布式调用链路进行追踪。类似功能的组件还有：Zipkin、**Pinpoint**、CAT等。



### 1.  概念与架构

SkyWalking是一个开源监控平台，用于从服务和云原生基础设施收集、分析、聚合和可视化数据。SkyWalking提供了一种简单的方法来维护分布式系统的清晰视图，甚至可以跨云查看。它是一种现代APM，专门为云原生、基于容器的分布式系统设计。

SkyWalking从三个维度对应用进行监视：service（服务）, service instance（实例）, endpoint（端点）

服务和实例就不多说了，端点是服务中的某个路径或者说URI

> SkyWalking allows users to understand the topology relationship between Services and Endpoints, to view the metrics of every Service/Service Instance/Endpoint and to set alarm rules.

SkyWalking允许用户了解服务和端点之间的拓扑关系，查看每个服务/服务实例/端点的度量，并设置警报规则。

#### 1.1.  架构

![](https://ziyekudeng.github.io/assets/images/2021/03/12/1.1架构.png)

SkyWalking逻辑上分为四个部分：Probes（探针）, Platform backend（平台后端）, Storage（存储）, UI

这个结构就很清晰了，探针就是Agent负责采集数据并上报给服务端，服务端对数据进行处理和存储，UI负责展示

![](https://ziyekudeng.github.io/assets/images/2021/03/12/1.1UI.png)

### 2.  下载与安装

SkyWalking有两中版本，ES版本和非ES版。如果我们决定采用ElasticSearch作为存储，那么就下载es版本。

https://skywalking.apache.org/downloads/

https://archive.apache.org/dist/skywalking/

![](https://ziyekudeng.github.io/assets/images/2021/03/12/2.1下载地址.png)
![](https://ziyekudeng.github.io/assets/images/2021/03/12/2.1安装指示.png)

*   agent目录将来要拷贝到各服务所在机器上用作探针
*   bin目录是服务启动脚本
*   config目录是配置文件
*   oap-libs目录是oap服务运行所需的jar包
*   webapp目录是web服务运行所需的jar包

接下来，要选择存储了，支持的存储有：

*   H2
*   ElasticSearch 6, 7
*   MySQL
*   TiDB
*   **InfluxDB**

作为监控系统，首先排除H2和MySQL，这里推荐InfluxDB，它本身就是时序数据库，非常适合这种场景

但是InfluxDB我不是很熟悉，所以这里先用ElasticSearch7

https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/backend-storage.md

#### 2.1.  安装ElasticSearch

https://www.elastic.co/guide/en/elasticsearch/reference/7.10/targz.html

##### 启动
./bin/elasticsearch -d -p pid
##### 停止
pkill -F pid

![](https://ziyekudeng.github.io/assets/images/2021/03/12/2.1安装命令.png)

ElasticSearch7.x需要Java 11以上的版本，但是如果你设置了环境变量JAVA_HOME的话，它会用你自己的Java版本

通常，启动过程中会报以下三个错误：

[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]

[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

[3]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured


**解决方法：**

在 /etc/security/limits.conf 文件中追加以下内容：

* soft nofile 65536
* hard nofile 65536
* soft nproc  4096
* hard nproc  4096

可通过以下四个命令查看修改结果：

* ulimit -Hn
* ulimit -Sn
* ulimit -Hu
* ulimit -Su

修改 /etc/sysctl.conf 文件，追加以下内容：

* vm.max_map_count=262144 

修改es配置文件 elasticsearch.yml 取消注释，保留一个节点

* cluster.initial_master_nodes: ["node-1"] 

为了能够ip:port方式访问，还需修改网络配置

* network.host: 0.0.0.0 

修改完是这样的：

![](https://ziyekudeng.github.io/assets/images/2021/03/12/2.1安装命令2.png)

至此，ElasticSearch算是启动成功了

接下来，在 config/application.yml 中配置es地址即可

* storage:
    * selector: ${SW_STORAGE:elasticsearch7}
    * elasticsearch7:
        * clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:192.168.100.19:9200}
#### 2.2.  安装Agent

https://github.com/apache/skywalking/blob/v8.2.0/docs/en/setup/service-agent/java-agent/README.md

将agent目录拷贝至各服务所在的机器上

* scp -r ./agent chengjs@192.168.100.12:~/ 

这里，我将它拷贝至各个服务目录下

![](https://ziyekudeng.github.io/assets/images/2021/03/12/2.2安装指示.png)

plugins是探针用到各种插件，SkyWalking插件都是即插即用的，可以把optional-plugins中的插件放到plugins中

修改 agent/config/agent.config 配置文件，也可以通过命令行参数指定

主要是配置服务名称和后端服务地址

* agent.service_name=${SW_AGENT_NAME:user-center}
* collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:192.168.100.17:11800} 

当然，也可以通过环境变量或系统属性的方式来设置，例如：

* export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800 

最后，在服务启动的时候用命令行参数 -javaagent 来指定探针

*  java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -jar yourApp.jar 

例如：

* java -javaagent:./agent/skywalking-agent.jar -Dspring.profiles.active=dev -Xms512m -Xmx1024m -jar demo-0.0.1-SNAPSHOT.jar 
### 3.  启动服务

修改 webapp/webapp.yml 文件，更改端口号及后端服务地址

* server:
    * port: 8080

* collector:
    * path: /graphql
    * ribbon:
    * ReadTimeout: 10000
  ###### #Point to all backend's restHost:restPort, split by ,
    * listOfServers: 127.0.0.1:12800 

启动服务

*  bin/startup.sh 

或者分别依次启动

*  bin/oapService.sh
* bin/webappService.sh 

查看logs目录下的日志文件，看是否启动成功

* 浏览器访问 http://127.0.0.1:8080

### 4\. 告警

![](https://ziyekudeng.github.io/assets/images/2021/03/12/4告警.png)

编辑 alarm-settings.yml 设置告警规则和通知

https://github.com/apache/skywalking/blob/v8.2.0/docs/en/setup/backend/backend-alarm.md

**重点说下告警通知**

![](https://ziyekudeng.github.io/assets/images/2021/03/12/4告警通知.png)
![](https://ziyekudeng.github.io/assets/images/2021/03/12/4告警通知2.png)

为了使用钉钉机器人通知，接下来，新建一个项目

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.wt.monitor</groupId>
    <artifactId>skywalking-alarm</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>skywalking-alarm</name>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.aliyun</groupId>
            <artifactId>alibaba-dingtalk-service-sdk</artifactId>
            <version>1.0.1</version>
        </dependency>

        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.15</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

**可选依赖（不建议引入）**

    <dependency <groupId>org.apache.skywalking</groupId>
    <artifactId>server-core</artifactId>
    <version>8.2.0</version>
    </dependency>

**定义告警消息实体类**

     package com.wt.monitor.skywalking.alarm.domain; import lombok.Data; import java.io.Serializable; /** * @author ChengJianSheng
    * @date 2020/12/1 */ @Data public class AlarmMessageDTO implements Serializable { private int scopeId; private String scope; /** * Target scope entity name */
    private String name; private String id0; private String id1; private String ruleName; /** * Alarm text message */
    private String alarmMessage; /** * Alarm time measured in milliseconds */
    private long startTime;
    
    } 

**发送钉钉机器人消息**

    package com.wt.monitor.skywalking.alarm.service; import com.dingtalk.api.DefaultDingTalkClient; import com.dingtalk.api.DingTalkClient; import com.dingtalk.api.request.OapiRobotSendRequest; import com.taobao.api.ApiException; import lombok.extern.slf4j.Slf4j; import org.apache.commons.codec.binary.Base64; import org.springframework.beans.factory.annotation.Value; import org.springframework.stereotype.Service; import javax.crypto.Mac; import javax.crypto.spec.SecretKeySpec; import java.io.UnsupportedEncodingException; import java.net.URLEncoder; import java.security.InvalidKeyException; import java.security.NoSuchAlgorithmException; /** * https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq
    * @author ChengJianSheng
    * @data 2020/12/1 */ @Slf4j
      @Service public class DingTalkAlarmService {
    
      @Value("${dingtalk.webhook}") private String webhook;
      @Value("${dingtalk.secret}") private String secret; public void sendMessage(String content) { try {
      Long timestamp = System.currentTimeMillis();
      String stringToSign = timestamp + "\n" + secret;
      Mac mac = Mac.getInstance("HmacSHA256");
      mac.init(new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256")); byte[] signData = mac.doFinal(stringToSign.getBytes("UTF-8"));
      String sign = URLEncoder.encode(new String(Base64.encodeBase64(signData)),"UTF-8");
    
               String serverUrl = webhook + "&timestamp=" + timestamp + "&sign=" + sign;
               DingTalkClient client = new DefaultDingTalkClient(serverUrl);
               OapiRobotSendRequest request = new OapiRobotSendRequest();
               request.setMsgtype("text");
               OapiRobotSendRequest.Text text = new OapiRobotSendRequest.Text();
               text.setContent(content);
               request.setText(text);
    
               client.execute(request);
           } catch (ApiException e) {
               e.printStackTrace();
               log.error(e.getMessage(), e);
           } catch (NoSuchAlgorithmException e) {
               e.printStackTrace();
               log.error(e.getMessage(), e);
           } catch (UnsupportedEncodingException e) {
               e.printStackTrace();
               log.error(e.getMessage(), e);
           } catch (InvalidKeyException e) {
               e.printStackTrace();
               log.error(e.getMessage(), e);
           }
      }
      }

**AlarmController.java**

    package com.wt.monitor.skywalking.alarm.controller; import com.alibaba.fastjson.JSON; import com.wt.monitor.skywalking.alarm.domain.AlarmMessageDTO; import com.wt.monitor.skywalking.alarm.service.DingTalkAlarmService; import lombok.extern.slf4j.Slf4j; import org.springframework.beans.factory.annotation.Autowired; import org.springframework.web.bind.annotation.PostMapping; import org.springframework.web.bind.annotation.RequestBody; import org.springframework.web.bind.annotation.RequestMapping; import org.springframework.web.bind.annotation.RestController; import java.text.MessageFormat; import java.util.List; /** * @author ChengJianSheng
    * @date 2020/12/1 */ @Slf4j
      @RestController
      @RequestMapping("/skywalking") public class AlarmController {
    
      @Autowired private DingTalkAlarmService dingTalkAlarmService;
    
      @PostMapping("/alarm") public void alarm(@RequestBody List<AlarmMessageDTO> alarmMessageDTOList) {
      log.info("收到告警信息: {}", JSON.toJSONString(alarmMessageDTOList)); if (null != alarmMessageDTOList) {
      alarmMessageDTOList.forEach(e->dingTalkAlarmService.sendMessage(MessageFormat.format("-----来自SkyWalking的告警-----\n【名称】: {0}\n【消息】: {1}\n", e.getName(), e.getAlarmMessage())));
      }
      }
      }

![](https://ziyekudeng.github.io/assets/images/2021/03/12/4告警通知示例.png)

### 5\.  文档

https://skywalking.apache.org/

https://skywalking.apache.org/zh/

https://github.com/apache/skywalking/tree/v8.2.0/docs

https://archive.apache.org/dist/

https://www.elastic.co/guide/en/elasticsearch/reference/master/index.html


