---
layout: post
title: APM系统SkyWalking介绍
category: arch
tags: [arch]
keywords: 架构
---



> 公司最近在构建服务化平台，需要上线APM系统，本篇文章简单的介绍SkyWalking

## APM

APM全称Application Performance Management应用性能管理，目的是通过各种探针采集数据，收集关键指标，同时搭配数据呈现以实现对应用程序性能管理和故障管理的系统化解决方案

Zabbix、Premetheus、open-falcon等监控系统主要关注服务器硬件指标与系统服务运行状态等，而APM系统则更重视**程序内部执行过程**指标和**服务之间链路调用**情况的监控，APM更有利于深入代码找到请求响应“慢”的根本问题，与Zabbix之类的监控是互补关系

目前市面上开源的APM系统主要有CAT、Zipkin、Pinpoint、SkyWalking，大都是参考Google的Dapper实现的

**CAT：** 是由国内美团点评开源的，基于Java语言开发，目前提供Java、C/C++、Node.js、Python、Go等语言的客户端，监控数据会全量统计，国内很多公司在用，例如美团点评、携程、拼多多等，CAT跟下边要介绍的Zipkin都需要在应用程序中埋点，对代码侵入性强，我们倾向于选择对代码无侵入的产品，所以淘汰了CAT

**Zipkin：** 由Twitter公司开发并开源，Java语言实现，侵入性相对于CAT要低一点，需要对web.xml之类的配置文件做修改，但依然对代码有侵入，也没有选择

**Pinpoint：** 一个韩国团队开源的产品，运用了字节码增强技术，只需要在启动时添加启动参数即可，对代码**无侵入**，目前支持Java和PHP语言，底层采用HBase来存储数据，探针收集的数据粒度非常细，但性能损耗大，因其出现的时间较长，完成度也很高，应用的公司较多

**SkyWalking：** 国人开源的产品，主要开发人员来自于华为，2019年4月17日Apache董事会批准SkyWalking成为顶级项目，支持Java、.Net、NodeJs等探针，数据存储支持Mysql、Elasticsearch等，跟Pinpoint一样采用字节码注入的方式实现代码的**无侵入**，探针采集数据粒度粗，但性能表现优秀，且对云原生支持，目前增长势头强劲，社区活跃，中文文档没有语言障碍

综合考虑，我们选择了SkyWalking

## SkyWalking

官方有两句话介绍SkyWalking：

SkyWalking是分布式系统的应用程序性能监视工具，专为微服务、云原生架构和基于容器（Docker、K8S、Mesos）架构而设计

SkyWalking是观察性分析平台和应用性能管理系统。提供分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案

### SkyWalking架构

SkyWalking采用组件式开发，易于扩展，主要组件作用如下：

**Skywalking Agent：** 采集`tracing`（调用链数据）和`metric`（指标）信息并上报，上报通过HTTP或者gRPC方式发送数据到Skywalking Collector

**Skywalking Collector ：** 链路数据收集器，对agent传过来的`tracing`和`metric`数据进行整合分析通过`Analysis Core`模块处理并落入相关的数据存储中，同时会通过`Query Core`模块进行二次统计和监控告警

**Storage：** Skywalking的存储，支持以`ElasticSearch`、`Mysql`、`TiDB`、`H2`等作为存储介质进行数据存储

**UI：** Web可视化平台，用来展示落地的数据，目前官方采纳了RocketBot作为SkyWalking的主UI

### SkyWalking界面

- 仪表盘

仪表盘主要包含Service Dashboard和Database Dashboard

Service Dashboard内分别有Global、Service、Endpoint、Instance面板，展示了全局以及服务、端点、实例的详细信息

Database Dashboard内可以展示数据库的响应时间、响应时间分布、吞吐量、SLA、慢SQL等详细信息，便于直观展示数据库状态

- 拓扑图

SkyWalking能够根据获取的数据自动绘制服务之间的调用关系图，并能识别常见的服务显示在图标上，例如图上的kafka、H2服务

每条连线的颜色反应了服务之间的调用延迟情况，可以非常直观的看到服务与服务之间的调用状态，连线中间的点能点击，可显示两个服务之间链路的平均响应时间、吞吐率以及SLA等信息

- 追踪面板

能够显示请求的代码内部执行情况，一个完整的请求都经过了哪些服务、执行了哪些代码方法、每个方法的执行时间、执行状态等详细信息，快速定位代码问题

- 告警面板

## 写在最后

SkyWalking目前还处在高速发展的阶段，我们在生产环境部署，遇到了一系列的问题，例如数据量过大图像断点，图像显示慢，与elasticsearch版本不兼容等，遇到问题所能查找的资料也有限，还需谨慎上线生产，但从Github上可以看到产品仍在快速更新不断完善，相信未来SkyWalking的发展也会越来越好，感谢开源

来源：[https://mp.weixin.qq.com/s/F-IPkfo6jp6Wkb4ql-jaLg](https://mp.weixin.qq.com/s/F-IPkfo6jp6Wkb4ql-jaLg)
 © 著作权归作者所有
