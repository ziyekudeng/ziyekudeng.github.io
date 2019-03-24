---
layout: post
title: dubbo：9个你不一定知道，但好用的功能
category: dubbo
tags: [dubbo]
---




**来源：阿飞的博客**

> dubbo功能非常完善，很多时候我们不需要重复造轮子，下面列举一些**你不一定知道，但是很好用**的功能。

## 直连Provider

在开发及测试环境下，可能需要绕过注册中心，只测试指定服务提供者，这时候可能需要点对点直连，点对点直连模式，将以服务接口为单位，忽略注册中心的提供者列表，A 接口配置点对点，不影响 B 接口从注册中心获取列表（说明：官方只建议开发&测试环境使用该功能），用法如下，url指定的地址就是直连地址：

    > `<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" version="1.0.0"  url="dubbo://172.18.1.205:20888/" />`

## 多版本

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用，用法如下：

    > `<dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" version="1.0.0" />`

利用dubbo该特性，我们能够实现一些功能的灰度发布，实现步骤如下：

1.  接口旧的实现定义version="1.0.0"，接口新的实现version="2.0.0"

2.  Consumer端定义version="*"

这样定义Provider和Consumer后，新旧接口实现各承担`50%`的流量；

利用dubbo该特性，还能完成不兼容版本迁移：

1.  在低压力时间段，先升级一半Provider为新版本；

2.  再将所有消费者升级为新版本；

3.  然后将剩下的一半提供者升级为新版本。

## 回声测试

回声测试用于检测服务是否可用，回声测试按照正常请求流程执行，能够测试整个调用是否通畅，可用于监控。
所有服务自动实现**EchoService**接口，只需将任意服务引用强制转型为**EchoService** 即可使用，使用方式（demoService是spring管理的bean）

     EchoService echoService = (EchoService) demoService;
    System.out.println(echoService.$echo("hello")); 
## 隐式参数

可以通过 **RpcContext** 的  `setAttachment()`  和  `getAttachment()`  在Consumer和Provider之间进行参数的隐式传递，例如Controller层拦截登录token，把根据token得到的memberId传给dubbo服务就能使用隐式参数传递的方式，`setAttachment()`设置的 KV 对，在完成一次远程调用会被清空，即多次远程调用要多次设置。使用方式：

1.  服务端set:

     RpcContext.getContext().setAttachment("CRT_MEMBER_ID", "13828886888");  

1.  客户端get：

     RpcContext.getContext().getAttachment("CRT_MEMBER_ID")  
## 上下文

上下文中存放的是当前调用过程中所需的环境信息。所有配置信息都将转换为 URL 的参数
RpcContext 是一个 ThreadLocal 的临时状态记录器，当接收到 RPC 请求，或发起 RPC 请求时，RpcContext 的状态
都会变化。例如：A 调 B，B 再调 C，则 B 机器上，在 B 调 C 之前，RpcContext 记录的是 A 调 B 的信息，在 B 调 C
之后，RpcContext 记录的是 B 调 C 的信息。使用方式：

     boolean isConsumerSide = RpcContext.getContext().isConsumerSide();  
## 本地伪装

本地伪装通常用于服务降级，例如某验权服务，当服务提供方全部挂掉后，客户端不抛出异常，而是通过 Mock 数据
返回授权失败。使用方式如下，mock指定的实现类在Provider抛出RpcException异常时执行（**一定要抛出RpcException异常才执行**），取代远程返回结果：

> `<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService"                 version="1.0.0" mock="com.alibaba.dubbo.demo.consumer.mock.DemoServiceMock"/>`

DemoServiceMock实现源码：

     public class DemoServiceMock implements DemoService {
        public String sayHello(String name) {
            return "mock-value";
        }
    }  
## 泛化调用

泛化接口调用方式主要用于客户端没有 API 接口及模型类元的情况，参数及返回值中的所有 POJO 均用Map表示，通常用于框架集成，例如：实现一个通用的服务测试框架，可通过GenericService调用所有服务实现。使用方式：

    > `<dubbo:reference id="demoService" interface="com.alibaba.dubbo.demo.DemoService" generic="true"/>`

调用源码：
    
     /**
     * @author afei
     * @version 1.0.0
     * @since 2017年11月22日
     */
    public class Main {
    
        public static void main(String[] args) {
            // 引⽤远程服务, 该实例⾥⾯封装了所有与注册中⼼及服务提供⽅连接，请缓存
            ReferenceConfig<GenericService> reference = new ReferenceConfig<GenericService>();
            // 弱类型接⼝名
            reference.setInterface("com.alibaba.dubbo.demo.DemoService");
            reference.setVersion("1.0.0");
            // 声明为泛化接⼝
            reference.setGeneric(true);
            // ⽤com.alibaba.dubbo.rpc.service.GenericService可以替代所有接口引用⽤
            GenericService genericService = reference.get();
            // 基本类型以及Date,List,Map等不需要转换，直接调⽤
            Object result = genericService.$invoke("sayYes", new String[] {"java.lang.String"}, new Object[] {"afei"});
            System.out.println("result --> "+result);
    
            // ⽤Map表示POJO参数，如果返回值为POJO也将自动转成Map
            Map<String, Object> teacher = new HashMap<String, Object>();
            teacher.put("id", "1");
            teacher.put("name", "admin");
            teacher.put("age", "18");
            teacher.put("level", "3");
            teacher.put("remark", "测试");
            // 如果返回POJO将自动转成Map
            result = genericService.$invoke("justTest", new String[]
                    {"com.alibaba.dubbo.demo.bean.HighTeacher"}, new Object[]{teacher});
            System.out.println("result --> "+result);
        }
    }  
## 访问日志

如果想记录每次请求信息，可开启访问日志，类似于Ngnix的访问日志。注意：此日志量比较大，请注意磁盘容量。使用方式（如果配置局部，全局访问日志就会失效）：
配置全局：
    
    > `<dubbo:provider accesslog="/app/dubbo-demo.log"/>`

配置局部：
    
    > `<dubbo:service interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" accesslog="/app/demo.log"/>`
    > `<dubbo:service interface="com.alibaba.dubbo.demo.TestService" ref="testService" accesslog="/app/test.log"/>`

日志格式样式：

> [2017-11-22 10:23:20] 172.18.1.205:56144 -> 172.18.1.205:20886 - com.alibaba.dubbo.demo.DemoService:1.0.0 sayHello(java.lang.String) ["afei"]

## 延迟暴露

如果服务需要预热时间，比如初始化本地缓存，等待相关资源就位等，可以使用delay进行延迟暴露。使Dubbo在Spring容器初始化完后延迟多少毫秒再暴露服务，这个真的很有用。使用方式：

 <dubbo:provider delay="5000"/>  

或者：

<dubbo:service delay="5000" interface="com.alibaba.dubbo.demo.DemoService" ref="demoService" version="1.0.0"/>  

