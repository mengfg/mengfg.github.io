---
layout: post
title: SpringCloud笔记（十三） SpringCloud Sleuth分布式请求链路追踪
date: 2021-01-07
tags: SpringCloud
---

## 概述

### 为什么会出现这个技术？需要解决哪些问题？

![196](\images\posts\springcloud\196.jpg)

![197](\images\posts\springcloud\197.jpg)

### 是什么

github地址：https://github.com/spring-cloud/spring-cloud-sleuth

Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案，在分布式系统中提供追踪解决方案并且兼容支持了zipkin

![198](\images\posts\springcloud\198.jpg)



## 搭建链路监控步骤

### zipkin

#### 下载

SpringCloud从F版起已不需要自己构建Zipkin server了，只需要调用jar包即可。

下载地址：https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

本项目以下载的 zipkin-server-2.12.9.exec.jar为例

#### 运行jar

使用如下命令	java -jar zipkin-server-2.12.9-exec.jar

![199](\images\posts\springcloud\199.jpg)

#### 运行控制台

访问：http://localhost:9411/zipkin/

#### 相关术语

Trace：类似于树结构的Span集合，表示一条调用链路，存在唯一标识

span：表示调用链路来源，通俗的理解span就是一次请求信息

完整的链路调用

![200](\images\posts\springcloud\200.jpg)

上图精简表示为：

![201](\images\posts\springcloud\201.jpg)

### 服务提供者

以cloud-provider-payment8001为例

#### 修改POM

添加以下依赖

```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

#### 修改yaml

添加`spring.zipkin.base-url`以及`spring.sleuth.sampler.probability`

```yaml
server:
  port: 8001
  
spring:
  application:
    name: cloud-payment-service
  zipkin:
    #监控的数据查看地址
    base-url: http://localhost:9411
  sleuth:
    sampler:
    #采样率，值介于0到1之间，1表示全部采集。一般设置为0.5即可
    probability: 1

  datasource:
    type: com.alibaba.druid.pool.DruidDataSource  #当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver  #mysql驱动包
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root

eureka:
  client:
    #表示是否将自己注册进eurekaserver，默认为true
    register-with-eureka: true
    #是否从eurekaserver抓去已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    instance-id: payment8001
    prefer-ip-address: true
#    #Eureka客户端向服务器端发送心跳的时间间隔，单位为秒（默认30秒）
#    lease-renewal-interval-in-seconds: 1
#    #Eureka服务端在收到最后一次心跳等待时间上限，单位为秒（默认90秒），超时将剔除服务
#    lease-expiration-duration-in-seconds: 2


mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.mg.springcloud.entities  #所有Entity别名类所在包
```

#### 业务类

业务类PaymentController中添加测试方法

```java
@GetMapping("/payment/zipkin")
public String paymentZipkin() {
    return "hi ,i'am paymentzipkin server fall back，welcome to atguigu，O(∩_∩)O哈哈~";
}
```

### 服务消费者（调用方）

以cloud-consumer-order80模块为例

#### 修改POM

添加以下依赖

```xml
<!--包含了sleuth+zipkin-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

#### 修改yaml

添加`spring.zipkin.base-url`以及`spring.sleuth.sampler.probability`

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-order-service
  zipkin:
    #监控的数据查看地址
    base-url: http://localhost:9411
  sleuth:
    sampler:
    #采样率，值介于0到1之间，1表示全部采集。一般设置为0.5即可
    probability: 1

eureka:
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
```

#### 业务类

业务类OrderController中添加测试方法

```java
@GetMapping("/consumer/payment/zipkin")
public String paymentZipkin() {
    String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
    return result;
}
```

### 测试

依次启动eureka7001/8001/80，

访问：http://localhost/consumer/payment/zipkin     80调用8001几次测试下

打开浏览器访问:http:localhost:9411，会出现以下界面

![202](\images\posts\springcloud\202.jpg)

点击某个链路会显示详细信息：

![203](\images\posts\springcloud\203.jpg)

查看依赖关系：

![204](\images\posts\springcloud\204.jpg)

### 原理

![205](\images\posts\springcloud\205.jpg)