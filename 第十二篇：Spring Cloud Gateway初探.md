# SpringCloud系列教程 | 第十二篇：Spring Cloud Gateway初探

前面我们在聊服务网关Zuul的时候提到了Gateway，那么Zuul和Gateway都是服务网关，这两个有什么区别呢？

## 1. Zuul和Gateway的恩怨情仇

### 1.1 背景
Zuul是Netflix开源的一个项目，Spring只是将Zuul集成在了Spring Cloud中。而Spring Cloud Gateway是Spring Cloud的一个子项目。

还有一个版本的说法是Zuul2的连续跳票和Zuul1的性能并不是很理想，从而催生了Spring Cloud Gateway。

### 1.2 性能比较

网上很多地方都说Zuul是阻塞的，Gateway是非阻塞的，这么说是不严谨的，准确的讲Zuul1.x是阻塞的，而在2.x的版本中，Zuul也是基于Netty，也是非阻塞的，如果一定要说性能，其实这个真没多大差距。

而官方出过一个测试项目，创建了一个benchmark的测试项目：[spring-cloud-gateway-bench](https://github.com/spencergibb/spring-cloud-gateway-bench)，其中对比了：

* Spring Cloud Gateway
* Zuul1.x
* Linkerd

| 组件 | RPS(request per second) |
| -- | -- |
| Spring Cloud Gateway | Requests/sec: 32213.38 |
| Zuul | Requests/sec: 20800.13 |
| Linkerd | Requests/sec: 28050.76 |

从结果可知，Spring Cloud Gateway的RPS是Zuul1.x的1.6倍。

下面，我们进入正题，开始聊聊Spring Cloud Gateway的一些事情。

## 2. Spring Cloud Gateway

Spring Cloud Gateway 是 Spring Cloud 的一个全新项目，该项目是基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Netflix Zuul，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。

### 2.1 特征

* 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0

* 动态路由

* Predicates 和 Filters 作用于特定路由

* 集成 Hystrix 断路器

* 集成 Spring Cloud DiscoveryClient

* 易于编写的 Predicates 和 Filters

* 限流

* 路径重写

###  2.2 术语
* **Route（路由）**：这是网关的基本构建块。它由一个 ID，一个目标 URI，一组断言和一组过滤器定义。如果断言为真，则路由匹配。

* **Predicate（断言）**：这是一个 Java 8 的 Predicate。输入类型是一个 ServerWebExchange。我们可以使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数。

* **Filter（过滤器）**：这是org.springframework.cloud.gateway.filter.GatewayFilter的实例，我们可以使用它修改请求和响应。

### 2.3 流程

![](https://springcloud-oss.oss-cn-shanghai.aliyuncs.com/chapter12/006tKfTcly1fr2q2m5jq7j30cb0gjmxm.jpg)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。

## 3.快速上手