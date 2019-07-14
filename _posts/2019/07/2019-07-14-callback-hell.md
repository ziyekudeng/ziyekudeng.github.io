---
layout: post
title: 如何避免Java中的回调“地狱”
category: arch
tags: [arch]
keywords: 架构
---

 

## 如何避免Java中的回调“地狱”



 **程序猿DD** 



* * *

本篇文章将简要介绍进行同步远程调用所涉及的代码类型。然后，我们将演示非阻塞 IO 中的分层如何高效使用资源（尤其是线程），引入了称为回调地狱带来的复杂性以及基于反应流方法如何简化编程模型。

**1. 目标服务**

客户端调用表示城市详细信息的目标服务有两个端口。当使用类型为——/cityids 的 URI 调用时，返回城市 id 列表，并且示例结果如下所示：

    [
        1,
        2,
        3,
        4,
        5,
        6,
        7
    ]

一个端口返回给定其 ID 的城市的详细信息，例如，当使用 ID 为1——“/cities/1” 调用时：

    {
        "country": "USA",
        "id": 1,
        "name": "Portland",
        "pop": 1600000
    }

客户端的责任是获取城市 ID 的列表，然后对于每个城市，根据 ID 获取城市的详细信息并将其组合到城市列表中。

**2\. 同步调用**

我正在使用 Spring Framework 的 RestTemplate 进行远程调用。获取 cityId 列表的 Kotlin 函数如下所示：

    private fun getCityIds(): List<String> {
        val cityIdsEntity: ResponseEntity<List<String>> = restTemplate
                .exchange("http://localhost:$localServerPort/cityids",
                        HttpMethod.GET,
                        null,
                        object : ParameterizedTypeReference<List<String>>() {})
        return cityIdsEntity.body!!
    }

获取城市详情：


    private fun getCityForId(id: String): City {
        return restTemplate.getForObject("http://localhost:$localServerPort/cities/$id", City::class.java)!!
    }

鉴于这两个函数，它们很容易组合，以便于轻松返回城市列表 ：
    
    val cityIds: List<String> = getCityIds()
    val cities: List<City> = cityIds
            .stream()
            .map<City> { cityId -> getCityForId(cityId) }
            .collect(Collectors.toList())
    cities.forEach { city -> LOGGER.info(city.toString()) }

代码很容易理解；但是，涉及八个阻塞调用：

1.  获取 7 个城市 ID 的列表，然后获取每个城市的详细信息

2.  获取 7 个城市的详细信息

每一个调用都将在不同的线程上。

**3. 非阻塞 IO 回调**

我将使用 AsyncHttpClient 库来进行非阻塞 IO 调用。

进行远程调用时，AyncHttpClient 返回 ListenableFuture 类型。


    val responseListenableFuture: ListenableFuture<Response> = asyncHttpClient
                    .prepareGet("http://localhost:$localServerPort/cityids")
                    .execute()

可以将回调附加到 ListenableFuture 以在可用时对响应进行操作。

    
    responseListenableFuture.addListener(Runnable {
        val response: Response = responseListenableFuture.get()
        val responseBody: String = response.responseBody
        val cityIds: List<Long> = objectMapper.readValue<List<Long>>(responseBody,
                object : TypeReference<List<Long>>() {})
        ....
    }

鉴于 cityIds 的列表，我想获得城市的详细信息，因此从响应中，我需要进行更多的远程调用并为每个调用附加回调以获取城市的详细信息：
    
    val responseListenableFuture: ListenableFuture<Response> = asyncHttpClient
            .prepareGet("http://localhost:$localServerPort/cityids")
            .execute()
    responseListenableFuture.addListener(Runnable {
        val response: Response = responseListenableFuture.get()
        val responseBody: String = response.responseBody
        val cityIds: List<Long> = objectMapper.readValue<List<Long>>(responseBody,
                object : TypeReference<List<Long>>() {})
        cityIds.stream().map { cityId ->
            val cityListenableFuture = asyncHttpClient
                    .prepareGet("http://localhost:$localServerPort/cities/$cityId")
                    .execute()
            cityListenableFuture.addListener(Runnable {
                val cityDescResp = cityListenableFuture.get()
                val cityDesc = cityDescResp.responseBody
                val city = objectMapper.readValue(cityDesc, City::class.java)
                LOGGER.info("Got city: $city")
            }, executor)
        }.collect(Collectors.toList())
    }, executor)

这是一段粗糙的代码；回调中又包含一组回调，很难推理和理解 - 因此它被称为“回调地狱”。

**4\. 在 Java CompletableFuture 中使用非阻塞 IO**

通过将 Java 的 CompletableFuture 作为返回类型而不是 ListenableFuture 返回，可以稍微改进此代码。CompletableFuture 提供允许修改和返回类型的运算符。

例如，考虑获取城市 ID 列表的功能：
    
    private fun getCityIds(): CompletableFuture<List<Long>> {
        return asyncHttpClient
                .prepareGet("http://localhost:$localServerPort/cityids")
                .execute()
                .toCompletableFuture()
                .thenApply { response ->
                    val s = response.responseBody
                    val l: List<Long> = objectMapper.readValue(s, object : TypeReference<List<Long>>() {})
                    l
                }
    }

在这里，我使用 thenApply 运算符将 CompletableFuture<Response> 转换为 CompletableFuture<List<Long>> 。

同样的，获取城市详情：

    
    private fun getCityDetail(cityId: Long): CompletableFuture<City> {
        return asyncHttpClient.prepareGet("http://localhost:$localServerPort/cities/$cityId")
                .execute()
                .toCompletableFuture()
                .thenApply { response ->
                    val s = response.responseBody
                    LOGGER.info("Got {}", s)
                    val city = objectMaper.readValue(s, City::class.java)
                    city
                }
    }

这是基于回调的方法的改进。但是，在这个特定情况下，CompletableFuture 缺乏有用的运算符，例如，所有城市细节都需要放在一起：
    
    val cityIdsFuture: CompletableFuture<List<Long>> = getCityIds()
    val citiesCompletableFuture: CompletableFuture<List<City>> =
            cityIdsFuture
                    .thenCompose { l ->
                        val citiesCompletable: List<CompletableFuture<City>> =
                                l.stream()
                                        .map { cityId ->
                                            getCityDetail(cityId)
                                        }.collect(toList())
                        val citiesCompletableFutureOfList: CompletableFuture<List<City>> = CompletableFuture.allOf(*citiesCompletable.toTypedArray())
                                        .thenApply { _: Void? ->
                                            citiesCompletable
                                                    .stream()
                                                    .map { it.join() }
                                                    .collect(toList())
                                        }
                        citiesCompletableFutureOfList
                    }

使用了一个名为 CompletableFuture.allOf 的运算符，它返回一个“Void”类型，并且必须强制返回所需类型的 CompletableFuture<List<City>>。

**5\. 使用 Reactor 项目**

Project Reactor 是 Reactive Streams 规范的实现。它有两种特殊类型可以返回 0/1 项的流和 0/n 项的流 - 前者是 Mono，后者是 Flux。

Project Reactor 提供了一组非常丰富的运算符，允许以各种方式转换数据流。首先考虑返回城市 ID 列表的函数：

    
    private fun getCityIds(): Flux<Long> {
            return webClient.get()
                    .uri("/cityids")
                    .exchange()
                    .flatMapMany { response ->
                        LOGGER.info("Received cities..")
                        response.bodyToFlux<Long>()
                    }
        }

我正在使用 Spring 优秀的 WebClient 库进行远程调用并获得 Project Reactor Mono <ClientResponse> 类型的响应，可以使用 flatMapMany 运算符将其修改为 Flux<Long> 类型。

根据城市 ID，沿着同样的路线获取城市的详情：

        
    
    
    private fun getCityDetail(cityId: Long?): Mono<City> {
        return webClient.get()
                .uri("/cities/{id}", cityId!!)
                .exchange()
                .flatMap { response ->
                    val city: Mono<City> = response.bodyToMono()
                    LOGGER.info("Received city..")
                    city
                }
    }
在这里，Project Reactor Mono<ClientResponse> 类型正在使用 flatMap 运算符转换为 Mono<City> 类型。

以及从中获取 cityIds，这是 City 的代码：
    
    val cityIdsFlux: Flux<Long> = getCityIds()
    val citiesFlux: Flux<City> = cityIdsFlux
            .flatMap { this.getCityDetail(it) }
    return citiesFlux

这非常具有表现力 - 对比基于回调的方法的混乱和基于 Reactive Streams 的方法的简单性。

**6\. 结束语**

在我看来，这是使用基于反应流的方法的最大原因之一，特别是 Project Reactor，用于涉及跨越异步边界的场景，例如在此实例中进行远程调用。它清理了回调和回调的混乱，提供了一种使用丰富的运算符进行修改/转换类型的自然方法。

