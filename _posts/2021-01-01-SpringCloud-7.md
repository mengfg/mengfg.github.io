---
layout: post
title: SpringCloud笔记（七） OpenFeign服务接口调用
date: 2021-01-01
tags: SpringCloud
---

## 概述

### OpenFeign是什么

![79](\images\posts\springcloud\79.jpg)

![80](\images\posts\springcloud\80.jpg)

Feign是一个声明式的web服务客户端，让编写web服务客户端变得非常容易，只需创建一个接口并在接口上添加注解即可

GitHub的源码地址：https://github.com/spring-cloud/spring-cloud-openfeign

### 能干嘛

![81](\images\posts\springcloud\81.jpg)

### Feign和OpenFeign两者区别

![82](\images\posts\springcloud\82.jpg)



## OpenFeign使用步骤

接口+注解：微服务调用接口+@FeignClient

Feign在消费端使用

![83](\images\posts\springcloud\83.jpg)

#### 建module

新建cloud-consumer-feign-order80模块

#### 改POM

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

    <artifactId>cloud-consumer-feign-order80</artifactId>


    <!--openfeign-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>com.mg.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
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

#### 写yaml

```yaml
server:
  port: 80
eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
```

#### 主启动类

加入@EnableFeignClients注解

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableFeignClients
public class OrderFeignMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignMain80.class,args);
    }
}

```

#### 业务类

##### service层

业务逻辑接口+@FeignClient配置调用provider服务

新建PaymentFeignService接口并新增注解@FeignClient

```java
package com.mg.springcloud.services;

import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import feign.Param;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
	//调用CLOUD-PAYMENT-SERVICE服务下的/payment/get/{id}
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id);
}
```

##### controller层

```java
package com.mg.springcloud.controller;

import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import com.mg.springcloud.services.PaymentFeignService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeignService.getPaymentById(id);
    }
}
```

#### 测试

先启动2个eureka集群7001/7002

再启动2个微服务8001/8002

在启动cloud-consumer-feign-order80模块，让OpenFeign启动

访问：http://localhost/consumer/payment/get/31	发现访问成功，并且有负载均衡

Feign自带负载均衡配置项

#### 总结

![84](\images\posts\springcloud\84.jpg)



## OpenFeign超时控制

超时设置，故意设置超时演示出错情况

### 服务提供方8001controller层故意写暂停程序

```java
@GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout(){
        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (Exception e) {
            e.printStackTrace();
        }
        return port;
    }
```

### 服务消费方80添加超时方法PaymentFeignService

```java
@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeignService {
    @GetMapping(value = "/payment/feign/timeout")
    public String paymentFeignTimeout();
}
```

### 服务消费方80添加超时方法OrderFeignController

```java
@RestController
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/payment/feign/timeout")
    public String paymentFeignTimeout(){
        return paymentFeignService.paymentFeignTimeout();
    }

}
```

### 测试

访问：http://localhost/consumer/payment/feign/timeout

![85](\images\posts\springcloud\85.jpg)

原因在于：OpenFeign默认等待一秒钟，超过后报错

![86](\images\posts\springcloud\86.jpg)

### 解决

YML文件里需要开启OpenFeign客户端超时控制。OpenFeign中会自动引入ribbon的相关依赖，设置超时时间的原理也是通过ribbon去设置的

cloud-consumer-feign-order80模块yaml中增加

```yaml
#设置feign客户端的超时时间（OpenFeign默认支持ribbon）
ribbon:
  #指的是建立连接后从服务器读取到可用资源所用的时间。5秒
  ReadTimeout: 5000
  #指的是建立连接所用的时间，适用于网络状况正常的情况下，两端连接所用的时间。5秒
  ConnectTimeout: 5000
```



## OpenFeign日志打印功能

### 是什么

![87](\images\posts\springcloud\87.jpg)

### 日志级别

![88](\images\posts\springcloud\88.jpg)

### 配置日志bean

```java
package com.mg.springcloud.config;

import feign.Logger;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignConfig {

    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

### YML文件里需要开启日志的Feign客户端

```yaml
logging:
  level:
    #feign日志以什么级别监控那个接口
    com.mg.springcloud.services.PaymentFeignService: debug
```

### 测试

访问：http://localhost/consumer/payment/get/1

![89](\images\posts\springcloud\89.jpg)

