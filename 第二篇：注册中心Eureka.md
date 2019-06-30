# 跟我学SpringCloud | 第二篇：注册中心Eureka

Eureka是Netflix开源的一款提供服务注册和发现的产品，它提供了完整的Service Registry和Service Discovery实现。也是springcloud体系中最重要最核心的组件之一。

## 什么是注册中心
#### 服务中心

管理各种服务功能包括服务的注册、发现、熔断、负载、降级等，比如dubbo admin后台的各种功能。

有了注册中心，调用关系的变化，画几个简图来看一下。

服务A调用服务B

![](https://github.com/meteor1993/image/blob/master/springcloud/chapter2/simple.png?raw=true)

有了注册中心之后，任何一个服务都不在是直连的，都需要通过注册中心去调用。

![](https://github.com/meteor1993/image/blob/master/springcloud/chapter2/simple_register_center.png?raw=true)

如果是一个连续调用：

服务A调用服务B，服务B调用服务C

![](https://github.com/meteor1993/image/blob/master/springcloud/chapter2/simple_two.png?raw=true)

这里如果加上注册中心，整个调用流程就会分为两步，服务A先从注册中心请求服务B，服务B再从注册中心请求服务C

![](https://github.com/meteor1993/image/blob/master/springcloud/chapter2/simple_two_register_center.png?raw=true)

上面的示例只是描述了两三个服务之间的互相调用，可能加上注册中心还会稍显繁琐，如果一条调用链上面有几十个服务（这个丝毫不是开玩笑哦，正常的业务流程中很可能出现这种复杂的调用过程），在工作中我就遇到过超过20个服务的互相调用，这种复杂业务场景的互相调用，如果不使用注册中心，画出来的图会连成一个网状结构，单从图上面已经很难找出服务的上下游关系了。其中如果一个服务有改动，就会牵扯到上下游多台机器的重启，整个架构设计完全耦合在一起，每次改动所需要的工作量完全超出想象。通过注册中心去注册服务，完全不在需要关心上下游机器的ip地址，由几台服务器组成，是否重启才会生效，注册中心已经帮我们把服务的注册和发现做好了，我们只需要知道注册中心在哪里，对应的服务名是什么就ok啦~~

由于各种服务都注册到了服务中心，就有了去做很多高级功能条件。比如几台服务提供相同服务来做均衡负载；监控服务器调用成功率来做熔断，移除服务列表中的故障点；监控服务调用时间来对不同的服务器设置不同的权重等等。

在说Eureka之前我们先了解一下Netflix这家公司：

## 以下介绍来自于百度百科：

> Netflix(Nasdaq NFLX) 成立于1997年，是一家在线影片租赁提供商，主要提供Netflix超大数量的DVD并免费递送，总部位于美国加利福尼亚州洛斯盖图。

> Netflix已经连续五次被评为顾客最满意的网站。可以通过PC、TV及iPad、iPhone收看电影、电视节目，可通过Wii，Xbox360，PS3等设备连接TV。Netflix大奖赛从2006年10月份开始，Netflix公开了大约1亿个1－5的匿名影片评级，数据集仅包含了影片名称。评价星级和评级日期，没有任何文本评价的内容。比赛要求参赛者预测Netflix的客户分别喜欢什么影片，要把预测的效率提高10%以上。

总而言之，这是一家全球最大的（可能要排除国内的，具体不清楚）流媒体公司，最开始见这个单子是在各种美剧或者电影的开头，顺手百度了一下，Netflix拍摄的代表性的美剧有《纸牌屋》、《毒枭》、《怪奇物语》。springcloud的微服务就是基于Netflix这家公司的开源产品来做的。

Netflix的开源框架组件已经在Netflix的大规模分布式微服务环境中经过多年的生产实战验证，正逐步被社区接受为构造微服务框架的标准组件。Spring Cloud开源产品，主要是基于对Netflix开源组件的进一步封装，方便Spring开发人员构建微服务基础框架。对于一些打算构建微服务框架体系的公司来说，充分利用或参考借鉴Netflix的开源微服务组件(或Spring Cloud)，在此基础上进行必要的企业定制，无疑是通向微服务架构的捷径。

## Eureka

按照官方介绍：

> Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers.

> Eureka 是一个基于 REST 的服务，主要在 AWS 云中使用, 定位服务来进行中间层服务器的负载均衡和故障转移。

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务注册和发现。Eureka 采用了 C-S 的设计架构。Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server，并维持心跳连接。这样系统的维护人员就可以通过 Eureka Server 来监控系统中各个微服务是否正常运行。Spring Cloud 的一些其他模块（比如Zuul）就可以通过 Eureka Server 来发现系统中的其他微服务，并执行相关的逻辑。

Eureka由两个组件组成：Eureka服务器和Eureka客户端。Eureka服务器用作服务注册服务器。Eureka客户端是一个java客户端，用来简化与服务器的交互、作为轮询负载均衡器，并提供服务的故障切换支持。Netflix在其生产环境中使用的是另外的客户端，它提供基于流量、资源利用率以及出错状态的加权负载均衡。

用官方的一张图来认识一下：

![](https://github.com/meteor1993/image/blob/master/springcloud/chapter2/eureka-architecture-overview.png?raw=true)

上图简要描述了Eureka的基本架构，由3个角色组成：

1、Eureka Server
* 提供服务注册和发现
2、Service Provider
* 服务提供方
* 将自身服务注册到Eureka，从而使服务消费方能够找到
3、Service Consumer
* 服务消费方
* 从Eureka获取注册服务列表，从而能够消费服务

## 案例实践
终于到了重头戏，开始撸代码~~~
#### Eureka Server