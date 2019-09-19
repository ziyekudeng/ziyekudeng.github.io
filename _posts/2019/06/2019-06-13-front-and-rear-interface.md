---
layout: post
title: 聊聊前后端分离接口规范
category: arch
tags: [arch]
keywords: 架构
---
 
来源：https://t.cn/Ai9iAJ4t

# 1. 如何做分离

## 1.1 职责分离

![](https://ziyekudeng.github.io/assets/images/2019/0613/discrete-interface/1.webp)

职责分离

*   前后端仅仅通过异步接口(AJAX/JSONP)来编程

*   前后端都各自有自己的开发流程，构建工具，测试集合

*   关注点分离，前后端变得相对独立并松耦合

![](https://ziyekudeng.github.io/assets/images/2019/0613/discrete-interfac/2.webp)

## 1.2 开发流程

*   后端编写和维护接口文档，在 API 变化时更新接口文档

*   后端根据接口文档进行接口开发

*   前端根据接口文档进行开发 + Mock平台

*   开发完成后联调和提交测试

Mock 服务器根据接口文档自动生成 Mock 数据，实现了接口文档即API：

![](https://ziyekudeng.github.io/assets/images/2019/0613/discrete-interfac/3.webp)

开发流程

## 1.3 具体实施

现在已基本完成了，接口方面的实施：

*   接口文档服务器：可实现接口变更实时同步给前端展示；

*   Mock接口数据平台：可实现接口变更实时Mock数据给前端使用；

*   接口规范定义：很重要，接口定义的好坏直接影响到前端的工作量和实现逻辑；具体定义规范见下节；

![](https://ziyekudeng.github.io/assets/images/2019/0613/discrete-interfac/4.webp)

接口文档+Mock平台服务器

# 2. 接口规范V1.0.0

## 2.1 规范原则

*   接口返回数据即显示：前端仅做渲染逻辑处理；

*   渲染逻辑禁止跨多个接口调用；

*   前端关注交互、渲染逻辑，尽量避免业务逻辑处理的出现；

*   请求响应传输数据格式：JSON，JSON数据尽量简单轻量，避免多级JSON的出现；

## 2.2 基本格式

### 2.2.1 请求基本格式

GET请求、POST请求==必须包含key为body的入参，所有请求数据包装为JSON格式，并存放到入参body中==，示例如下：

**GET请求：**

    xxx/login?body={"username":"admin","password":"123456","captcha":"scfd","rememberMe":1}

**POST请求：**

![](https://ziyekudeng.github.io/assets/images/2019/0613/discrete-interfac/5.webp)

### 2.2.2 响应基本格式

    {
        code: 200,
        data: {
            message: "success"
        }
    }

code : 请求处理状态

*   200: 请求处理成功

*   500: 请求处理失败

*   401: 请求未认证，跳转登录页

*   406: 请求未授权，跳转未授权提示页

data.message: 请求处理消息

*   code=200 且 data.message="success": 请求处理成功

*   code=200 且 data.message!="success": 请求处理成功, 普通消息提示：message内容

*   code=500: 请求处理失败，警告消息提示：message内容

## 2.3 响应实体格式

    {
        code: 200,
        data: {
            message: "success",
            entity: {
                id: 1,
                name: "XXX",
                code: "XXX"
            }
        }
    }

data.entity: 响应返回的实体数据

## 2.4 响应列表格式

data.list: 响应返回的列表数据

## 2.5 响应分页格式
    
    {
        code: 200,
        data: {
            recordCount: 2,
            message: "success",
            totalCount: 2,
            pageNo: 1,
            pageSize: 10,
            list: [
                {
                    id: 1,
                    name: "XXX",
                    code: "H001"
                },
                {
                    id: 2,
                    name: "XXX",
                    code: "H001"
                } ],
            totalPage: 1
        }
    }

*   data.recordCount: 当前页记录数

*   data.totalCount: 总记录数

*   data.pageNo: 当前页码

*   data.pageSize: 每页大小

*   data.totalPage: 总页数

## 2.6 特殊内容规范

### 2.6.1 下拉框、复选框、单选框

由后端接口统一逻辑判定是否选中，通过isSelect标示是否选中，示例如下：
    
    {
        code: 200,
        data: {
            message: "success",
            list: [{
                id: 1,
                name: "XXX",
                code: "XXX",
                isSelect: 1
            }, {
                id: 1,
                name: "XXX",
                code: "XXX",
                isSelect: 0
            }]
        }
    }

禁止下拉框、复选框、单选框判定选中逻辑由前端来处理，统一由后端逻辑判定选中返回给前端展示；

### 2.6.2 Boolean类型

关于Boolean类型，JSON数据传输中一律使用1/0来标示，1为是/True，0为否/False；

### 2.6.3 日期类型

关于日期类型，JSON数据传输中一律使用字符串，具体日期格式因业务而定；

# 3. 未来的大前端

目前我们现在用的前后端分离模式属于第一阶段，由于使用到的一些技术jquery等，对于一些页面展示、数据渲染还是比较复杂，不能够很好的达到复用。对于前端还是有很大的工作量。

下一阶段可以在前端工程化方面，对技术框架的选择、前端模块化重用方面，可多做考量。也就是要迎来“==前端为主的 MV* 时代==”。大多数的公司也基本都处于这个分离阶段。

最后阶段就是==Node 带来的全栈时代==，完全有前端来控制页面，URL，Controller，路由等，后端的应用就逐步弱化为真正的数据服务+业务服务，做且仅能做的是提供数据、处理业务逻辑，关注高可用、高并发等。
