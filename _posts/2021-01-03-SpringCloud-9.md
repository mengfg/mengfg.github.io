---
layout: post
title: SpringCloud笔记（九） Gateway网关
date: 2021-01-03
tags: SpringCloud
---

## 概述简介

### 官网

上一代zuul 1.X：https://github.com/Netflix/zuul/wiki

当前gateway：https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/

### 是什么

![125](\images\posts\springcloud\125.jpg)

#### 概述

![126](\images\posts\springcloud\126.jpg)

![127](\images\posts\springcloud\127.jpg)

![128](\images\posts\springcloud\128.jpg)

#### 总结

Spring Cloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架

**源码架构：**

![129](\images\posts\springcloud\129.jpg)

### 能干嘛

1. 反向代理
2. 鉴权
3. 流量控制
4. 熔断
5. 日志监控

### 微服务架构中网关在哪里

![130](\images\posts\springcloud\130.jpg)

### 有了Zuul了怎么又出来了gateway

#### 我们为什么选择Gatway?

1. neflix不太靠谱，zuul2.0一直跳票,迟迟不发布

   ![131](\images\posts\springcloud\131.jpg)

2. SpringCloud Gateway具有如下特性

   ![132](\images\posts\springcloud\132.jpg)

3. SpringCloud Gateway与Zuul的区别

   ![133](\images\posts\springcloud\133.jpg)

#### Zuul 1.X模型

![134](\images\posts\springcloud\134.jpg)

![135](\images\posts\springcloud\135.jpg)

#### GateWay模型

WebFlux是什么？

![136](\images\posts\springcloud\136.jpg)

![137](\images\posts\springcloud\137.jpg)

![138](\images\posts\springcloud\138.jpg)



## 三大核心概念

### Route(路由)

路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

### Predicate（断言）

参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

### Filter(过滤)

指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。



总结：

![139](\images\posts\springcloud\139.jpg)



## Gateway工作流程

### 官网总结

![140](\images\posts\springcloud\140.jpg)

![141](\images\posts\springcloud\141.jpg)

### 核心逻辑

路由转发+执行过滤器链



## 入门配置

### 建module

新建cloud-gateway-gateway9527模块

### 改POM

引入spring-cloud-starter-gateway，需要注意的是不要引入web和actuator模块。否则启动会出现错误：

Spring MVC found on classpath, which is incompatible with Spring Cloud Gateway at this time. Please remove spring-boot-starter-web dependency.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.mg.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-gateway-gateway9527</artifactId>


    <dependencies>
        <!--新增gateway-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mg.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>
    
</project>
```

### 写yaml

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run( GateWayMain9527.class,args);
    }
}
```

### 9527网关如何做路由映射

cloud-provider-payment8001模块中controller的访问地址有/payment/get/{id}和/payment/lb，我们目前不想暴露8001端口，希望在8001外面套一层，只暴露出9527

9527中YML新增网关配置

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由

        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由


eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka

```

![142](\images\posts\springcloud\142.jpg)

### 测试

启动7001，启动cloud-provider-payment8001，再启动启动9527网关。

添加网关前：http://localhost:8001/payment/get/1	可以正常访问

添加网关后：http://localhost:9527/payment/get/1	也可以正常访问



### Gateway网关路由有两种配置方式

#### 在配置文件yml中配置

就是上面的示例中，直接在yaml中配置路由

#### 代码中注入RouteLocator的Bean

实现访问http://localhost:9527/guonei会跳转到百度国内新闻页面，访问http://localhost:9527/guoji会跳转到百度国际新闻页面

```java
package com.mg.springcloud.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GateWayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_rote_mg", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }

    @Bean
    public RouteLocator customRouteLocator2(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_rote_mg2", r -> r.path("/guoji").uri("http://news.baidu.com/guoji")).build();
        return routes.build();
    }
}
```



## 通过微服务名实现动态路由

默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

以yaml配置的方式为例，修改`spring.cloud.gateway.routes.uri`

需要注意的是uri的协议为lb，表示启用Gateway的负载均衡功能。

lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri

serviceName指的是在注册中心注册的名字

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: lb://CLOUD-PAYMENT-SERVICE
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由

        - id: payment_routh2
          #uri: http://localhost:8001
          uri: lb://CLOUD-PAYMENT-SERVICE
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

测试：

http://localhost:9527/payment/lb	访问成功



## Predicate的使用

启动我们的gateway9527，会发现控制台会打印：

![143](\images\posts\springcloud\143.jpg)

### Route Predicate Factories这个是什么？

![144](\images\posts\springcloud\144.jpg)

![145](\images\posts\springcloud\145.jpg)

### 常用的Route Predicate

![146](\images\posts\springcloud\146.jpg)

需要在yaml文件中的`spring.cloud.gateway.routes.predicates`下配置

#### 1. After Route Predicate

![147](\images\posts\springcloud\147.jpg)

```yaml
- After=2021-01-11T15:35:16.959+08:00[Asia/Shanghai] #在这个时间点之后才能访问成功
```

上面的时间串，可以通过这种方式获得：

```java
public class ZonedDateTimeDemo {
    public static void main(String[] args) {
        ZonedDateTime zonedDateTime = ZonedDateTime.now();
        System.out.println(zonedDateTime);
    }
}
```

注意测试时，我们除了使用浏览器或者postman这种工具外，完全也可以使用cmd命令行下的curl进行测试

打开cmd命令行，直接使用curl进行测试，下面命令相当于发送一个get请求

curl http://localhost:9527/payment/lb

如果加入curl返回中文乱码，可以参考https://blog.csdn.net/leedee/article/details/82685636

![148](\images\posts\springcloud\148.jpg)

#### 2. Before Route Predicate

```yaml
- Before=2021-01-11T15:35:16.959+08:00[Asia/Shanghai]	#在这个时间点之前才能访问成功
```

测试：curl http://localhost:9527/payment/lb

#### 3. Between Route Predicate

```yaml
- Between=2021-01-11T15:35:16.959+08:00[Asia/Shanghai],2021-01-11T16:35:16.959+08:00[Asia/Shanghai] #在这个时间段之间才能访问成功
```

测试：curl http://localhost:9527/payment/lb

#### 4. Cookie Route Predicate

![149](\images\posts\springcloud\149.jpg)

```yaml
- Cookie=username,mengfg	#必须有cookie，cookie名称为username，值为mengfg
```

测试：curl http://localhost:9527/payment/lb --cookie "username=mengfg"

#### 5. Header Route Predicate

![150](\images\posts\springcloud\150.jpg)

```yaml
- Header=X-Request-Id, \d+  #请求头要有X-Request-Id属性，并且值为整数的正则表达式
```

测试：curl http://localhost:9527/payment/lb -H "X-Request-Id:123"

#### 6. Host Route Predicate

```yaml
- Host=**.mg.com	#主机名必须有mg.com才能成功
```

测试：

curl http://localhost:9527/payment/lb -H "Host:www.mg.com"	成功

#### 7. Method Route Predicate

```yaml
- Method=Get	#get请求才能成功
```

测试：curl http://localhost:9527/payment/lb

#### 8. Path Route Predicate

```yaml
- Path=/payment/lb/**   #访问/payment/lb/路径下的才能成功
```

测试：curl http://localhost:9527/payment/lb

#### 9. Query Route Predicate

```yaml
- Query=username, \d+ #要有参数名称username并且是正整数才能路由成功
```

测试：curl http://localhost:9527/payment/lb?username=1

#### 总结

说白了，Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理

全部的配置：

```yaml
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: payment_routh2
          #uri: http://localhost:8001
          uri: lb://CLOUD-PAYMENT-SERVICE
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由
#            - After=2021-01-11T15:35:16.959+08:00[Asia/Shanghai] #在这个时间点之后才能访问成功
#            - Before=2021-01-11T15:35:16.959+08:00[Asia/Shanghai]  #在这个时间点之前才能访问成功
#            - Between=2021-01-11T15:35:16.959+08:00[Asia/Shanghai],2021-01-11T16:35:16.959+08:00[Asia/Shanghai]  #在这个时间段之间才能访问成功
#            - Cookie=username,mengfg #必须有cookie，cookie名称为username，值为mengfg
#            - Header=X-Request-Id, \d+  #请求头要有X-Request-Id属性，并且值为整数的正则表达式
#            - Host=**.mg.com #主机名必须有mg.com才能成功
#            - Method=Get #get请求才能成功
#            - Query=username, \d+ #要有参数名称并且是正整数才能路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```



## Filter的使用

### 是什么

![151](\images\posts\springcloud\151.jpg)

### Spring Cloud Gateway的Filter 

#### 生命周期

分为两种：

1. 在业务逻辑之前	pre
2. 在业务逻辑之后    post

#### 种类

分为两种：

1. GatewayFilter	局部的

   ![152](\images\posts\springcloud\152.jpg)

2. GlobalFilter    全局的

   ![153](\images\posts\springcloud\153.jpg)

可以参照官网进行配置：

https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#the-addrequestparameter-gatewayfilter-factory

例如：

![154](\images\posts\springcloud\154.jpg)

### 自定义过滤器

自定义一个全局GlobalFilter

要实现GlobalFilter ，Ordered两个接口。主要可以用来做全局日志记录，统一网关鉴权等等

```java
package com.mg.springcloud.filter;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Date;

@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter,Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        log.info("*********come in MyLogGateWayFilter: "+new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("username");
        if(StringUtils.isEmpty(uname)){
            log.info("*****用户名为Null 非法用户,(┬＿┬)");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);//给人家一个回应
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```

测试：

正常访问：http://localhost:9527/payment/lb?username=mengfg

如果没有传递username参数，则会访问失败