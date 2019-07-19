---
layout: post
title: UidGenerator：百度开源的 Java 实现的、基于 Snowflake 算法的唯一 ID 生成器
category: arch
tags: [arch]
keywords: 架构
---

 

## 开源 | UidGenerator：百度开源的 Java 实现的、基于 Snowflake 算法的唯一 ID 生成器



> **项目介绍**

UidGenerator 是 Java 实现的，基于 Snowflake 算法的唯一 ID 生成器。

UidGenerator 以组件形式工作在应用项目中，支持自定义 workerId 位数和初始化策略，从而适用于 docker 等虚拟化环境下实例自动重启、漂移等场景。

在实现上，UidGenerator 通过借用未来时间来解决 sequence 天然存在的并发限制；采用 RingBuffer 来缓存已生成的 UID，并行化 UID 的生产和消费，同时对 CacheLine 补齐，避免了由 RingBuffer 带来的硬件级「伪共享」问题。最终单机 QPS 可达 600 万。

> **GitHub 地址**


# [UidGenerator GitHub 地址](https://github.com/baidu/uid-generator)
