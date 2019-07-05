# SpringCloud系列教程 | 第八篇：Spring Cloud Bus 消息总线

> Springboot: 2.1.6.RELEASE

> SpringCloud: Greenwich.SR1

> 如无特殊说明，本系列教程全采用以上版本

前面两篇文章我们聊了Spring Cloud Config配置中心，当我们在更新github上面的配置以后，如果想要获取到最新的配置，需要手动刷新或者利用webhook的机制每次提交代码发送请求来刷新客户端，客户端越来越多的时候，需要每个客户端都执行一遍，这种方案就不太适合了。使用Spring Cloud Bus（国人很形象的翻译为消息总线，我比较喜欢叫消息巴士）可以完美解决这一问题。

## 1. Spring Cloud Bus
Spring cloud bus通过轻量消息代理连接各个分布的节点。这会用在广播状态的变化（例如配置变化）或者其他的消息指令。Spring bus的一个核心思想是通过分布式的启动器对spring boot应用进行扩展，也可以用来建立一个多个应用之间的通信频道。目前唯一实现的方式是用AMQP消息代理作为通道，同样特性的设置（有些取决于通道的设置）在更多通道的文档中。

大家可以将它理解为管理和传播所有分布式项目中的消息既可，其实本质是利用了MQ的广播机制在分布式的系统中传播消息，目前常用的有Kafka和RabbitMQ。利用bus的机制可以做很多的事情，其中配置中心客户端刷新就是典型的应用场景之一，我们用一张图来描述bus在配置中心使用的机制。

![](https://springcloud-oss.oss-cn-shanghai.aliyuncs.com/chapter8/configbus1.jpg)

根据此图我们可以看出利用Spring Cloud Bus做配置更新的步骤:

1. 提交代码触发post给客户端A发送bus/refresh
2. 客户端A接收到请求从Server端更新配置并且发送给Spring Cloud Bus
3. Spring Cloud bus接到消息并通知给其它客户端
4. 其它客户端接收到通知，请求Server端获取最新配置
5. 全部客户端均获取到最新的配置

## 2. 项目示例
我们使用上一篇文章中的config-server和config-client来进行改造，mq使用rabbitmq来做示例。

### 2.1 客户端config-client
#### 2.1.1 添加依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

需要多引入spring-cloud-starter-bus-amqp包，增加对消息总线的支持

#### 2.1.2 配置文件 bootstrap.properties

```
spring.application.name=spring-cloud-config-client
server.port=8081

spring.cloud.config.name=springcloud-config
spring.cloud.config.profile=dev
spring.cloud.config.label=master
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.serviceId=spring-cloud-config-server

eureka.client.service-url.defaultZone=http://localhost:8761/eureka/

management.endpoints.web.exposure.include=*

## 开启消息跟踪
spring.cloud.bus.trace.enabled=true

spring.rabbitmq.host=101.132.124.125
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=wsy@123456
```
配置文件需要增加RebbitMq的相关配置，这样客户端代码就改造完成了。

#### 2.1.3 测试
依次启动eureka，config-serve，config-client

在启动config-client项目的时候我们会发现启动日志会输出这样的一条记录。

```

```