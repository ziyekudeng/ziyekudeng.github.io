---
layout: post
title: 为什么前后端分离了，我们比从前更痛苦？咋整呢！
category: tools
tags: [tools]
---

来源：https://t.cn/EVBRoQO

*   为什么前后端分离了，你比从前更痛苦？

*   为什么接口会频繁变动？

*   为什么接口文档永远都是不对的？

*   为什么测试工作永远只能临近上线才能开始？

*   怎么破？

*   aml-mocker

*   开始

*   配置 .raml-config.json

*   入门篇：Mock Server

*   高级篇：动态 Server

*   总结

* * *

你有没有遇到过：

*   前端代码刚写完，后端的接口又变了。

*   接口文档永远都是不对的。

*   测试工作永远只能临近上线才能开始。

# 为什么前后端分离了，你比从前更痛苦？

前后端分离早已经不是新闻，当真正分离之后确遇到了更多问题。要想解决现在的痛，就要知道痛的原因：

## 为什么接口会频繁变动？

**设计之初没有想好。** 这需要提高需求的理解能力和接口设计能力。

**变动的成本较低。**

德国有句谚语：“朝汤里吐口水。” 只有这样，才能让人们放弃那碗汤，停止不合理的行为。前后端同学坐在一起工作的时候效率会有提升，当后端同学接口变化时，只需要口头上通知一下即可，我们没有文档，我们很敏捷啊。没错，我们需要承认这样配合开发的效率会很高，但是频繁的变动会导致不断返工，造成了另一种浪费，这种浪费是可以被减少，甚至是被消除的。

## 为什么接口文档永远都是不对的？

接口文档在定接口时起到一定作用，写完接口就没有用了。后面接口的频繁变化，文档必定会永远落后于实际接口，维护文档的带来了一定的成本却没能带来价值。除非对外提供的接口，否则文档谁来看呢？没人看，用处又在哪？

有些公司干脆丢掉接口文档，说我们要拥抱敏捷。

所以接口文档落后的原因在于**没有给我们带来价值**。

## 为什么测试工作永远只能临近上线才能开始？

一个需求，后端开发 4 天，前端开发 4 天，联调 4 天，留给测试同学只有2天时间甚至更少，测不完只能带 bug 上线。

现有开发流程

在开发阶段测试同学无法介入，接口在变，前端也在变， **“提测”** 之前只能喝茶，“提测” 之后又忙的要命。

自动化？想都别想，空有一身好本领，在 “拥抱变化” 之后只能手工测试。偶尔还要拉上前台美眉客串一下测试小妹。手工测试枯燥乏味，乏味的工作就容易出错，而且还不能快速重复，无法对测试过的功能快速回归。

# 怎么破？

解决以上问题要**让接口文档发挥价值，提高变动接口的成本，测试尽早介入。**

接口文档发挥出价值，就要赋予契约的意义，就如同签字画押谁也不许变，来约束我们只认契约不认人。

契约应该由前端同学来驱动，前后端共同协商。由于前端同学与 UX 接触比较紧密，更了解页面所需的数据以及整体的 User Journey，前端同学驱动会更加合理。

契约敲定之后要帮助我们生成 Mock Server（后面我们会介绍一个工具），前后端同学就要依照契约各自开发。Mock Server 可暂时替代后台服务，帮组前端开发，同时，测试同学也可以依照契约文档来编写测试脚本，使用 Mock Server 进行脚本验证。

改进后的开发流程

当后端接口发生变化除了口头通知以外必须修改契约，前端同学和测试同学才能各自修改。如此一来修改契约的成本变高，人们在定契约时则会更加慎重，也会促使我们提高接口的设计能力。

看到图中没有 **“联调”** 的环节，并不是画错了，而是 “联调“ 不再是一项工作，在部署后只需要更改代理的配置即可。甚至使用现代前端框架（如，Vue 或者 React）只要在开发时配置一下，之后都不需要调整任何代码。

**“提测”** 呢？测试一直都在进行，也就不再有一个 ”提测“ 的环节，无论前后端任意一方完成开发，测试同学都可以进行测试。

理论终于扯完了，说起来容易做起来难啊，需要工具来帮助我们。接口描述的工具有很多，比较知名的 Swagger 和 Raml，我个人更倾向于 Raml 。

img

API 文档

描述工具生成文档还不够，还要生成 Mock Server，如果描述工具和 Mock Server 是分离又带来了额外的工作，好在有她——raml-mocker。

# raml-mocker

    raml-mocker (https://github.com/xbl/raml-mocker) 是一个基于 Raml 使用 Nodejs 开发的 Mock Server 工具，使用 Raml 描述接口中设置 response 的 `example` 指令即可，raml-mocker 会解析 Raml 文件，并启动一个 Mock Server，将 `example` 的内容返回给浏览器。

## 开始

### 初始化项目

    git clone https://github.com/xbl/raml-mocker-starter.git raml-api
    cd raml-api
    git remote rm origin
### 安装

yarn
# or
    npm install
### 启动 mock server

    yarn start
    # or
    npm start
### 测试

    curl -i https://localhost:3000/api/v1/users/1/books/
    # or
    curl -i https://localhost:3000/api/v1/users/1/books/1
### 生成 API 可视化文档

    yarn run build
    # or
    npm run build

此功能使用了raml2html。

## 配置 .raml-config.json
    
    {
      "controller": "./controller",
      "raml": "./raml",
      "main": "api.raml",
      "port": 3000,
      "plugins": []
    }

*   controller: controller 目录路径，在高级篇中会有更详细说明

*   raml: raml 文件目录

*   main: raml 目录下的入口文件

*   port: mock server 服务端口号

*   plugins: 插件

## 入门篇：Mock Server

raml-mocker 只需要在response 添加 `example`:

    /books:
      /:id:
        post:
          body:
            application/json:
              type: abc
          responses:
            200:
              body:
                application/json:
                  type: song
                  # 返回的 Mock 数据
                  example: !include ./books_200.json
    
    books_200.json
    
    {
      "code": 200,
      "data": [
        {
          "id": 1,
          "title": "books title",
          "description": "books desccription1"
        },
        {
          "id": 2,
          "title": "books title",
          "description": "books desccription2"
        }
      ]
    }

通过 curl 请求：

    curl -i https://localhost:3000/api/v1/users/1/books

就会得到 `example` 的数据，唯一不足是无法根据参数动态返回不同数据。别急，请往下看。

## 高级篇：动态 Server

如果静态的 Mock 数据不能满足你的需求，Raml-mocker 还提供了动态的功能。

在 raml 文档中添加 `(controller)` 指令，即可添加动态的 Server，如：
    
    /books:
      type:
        resourceList:
      get:
        description: 获取用户的书籍
        (controller): user#getBook
        responses:
          200:
            body:
              type: song[]
              example: !include ./books_200.json

在文档中 `(controller)` 表示 controller 目录下 user.js 中 getBook 函数。
    
    controller/user.js
    
    exports.getBook = (req, res, webApi) => {
      console.log(webApi);
      res.send('Hello World!');
    }

Raml-mocker 是在 expressjs 基础上进行开发，req、res 可以参考 express 文档。

webApi 会返回文档中的配置：
    
    {
      "absoluteUri": "/api/:version/users/:user_id/books",
      "method": "get",
      "controller": "user#getBook",
      "responses": [
        {
          "code": "200",
          "body": "... example ...",
          "mimeType": "application/json"
        }
      ]
    }

如此，raml-mocker 提供了更多可扩展空间，我们甚至可以在 controller 中实现一定的逻辑。

### 插件

Raml-mocker 提供了插件机制，允许我们在不使用 `controller` 指令的时候对 response 的内容进行处理，例如使用 Mockjs。

    .raml-config.json
    
    {
      "controller": "./controller",
      "raml": "./raml",
      "main": "api.raml",
      "port": 3000,
      "plugins": ["./plugins/mock.js"]
    }

    ./plugins/mock.js
    
    var { mock } = require('mockjs');
    
    module.exports = (body) => {
      try {
        return mock(JSON.parse(body));
      } catch(e) {}
      return body;
    }
    
    Enjoy it！

# 总结

前后端分离可以让我们的职责更清晰，打破前端发挥的局限，工作解耦之后能更好的提高开发效率。然而因为没有规划好开发流程，导致了我们没有发挥出其应有的价值，造成了更多的浪费。

raml-mocker 能够帮助我们在工具上解决一定的问题，更重要的是持续改进的思想，只有团队的思想是统一的才有可能达到快速交付。

希望能对你有所帮助，谢谢！

