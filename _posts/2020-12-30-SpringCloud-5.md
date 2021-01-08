---
layout: post
title: SpringCloud笔记（五） Consul服务注册与发现
date: 2020-12-30
tags: SpringCloud
---

## Consul简介

官网：https://www.consul.io/intro/index.html

### 是什么

![45](\images\posts\springcloud\45.jpg)

### 能干嘛

![46](\images\posts\springcloud\46.jpg)

#### 服务发现

提供HTTP和DNS两种发现方式

#### 健康监测

支持多种协议，HTTP、TCP、Docker、Shell脚本定制化

#### KV存储

key , Value的存储方式

#### 多数据中心

Consul支持多数据中心

#### 可视化Web界面

### 去哪下

https://www.consul.io/downloads.html

### 怎么玩

可以参照官网，也可以参考以下中文网站：

https://www.springcloud.cc/spring-cloud-consul.html

## 安装并运行Consul

下载解压完成后只有一个consul.exe文件，cmd命令行下，查看版本信息

![47](\images\posts\springcloud\47.jpg)

### 使用开发模式启动

`consul agent -dev`

![48](\images\posts\springcloud\48.jpg)

启动成功后，通过以下地址可以访问Consul的首页：http;//localhost:8500

![49](\images\posts\springcloud\49.jpg)

## SpringCloud整合consul

### 服务提供者

#### 建module

新建cloud-providerconsul-payment8006模块

#### 改POM

引入consul的依赖

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

    <artifactId>cloud-providerconsul-payment8006</artifactId>

    <dependencies>
        <!--引入consul的依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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
  port: 8006


spring:
  application:
    name: consul-provider-payment
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class,args);
    }
}
```

#### 业务类

为方便测试，只写controller，没有业务逻辑

```java
package com.mg.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.UUID;

@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/consul")
    public String paymentConsul(){
        return "springcloud with consul: "+serverPort+"\t"+ UUID.randomUUID().toString();
    }
}
```

#### 测试

启动cloud-providerconsul-payment8006模块后

##### 测试1

访问：http://localhost:8500/	发现服务已经注册上来了

![50](\images\posts\springcloud\50.jpg)

点击服务，会显示下面所属的实例

![51](\images\posts\springcloud\51.jpg)

点击实例可以显示相应的实例状态

![52](\images\posts\springcloud\52.jpg)

consul这方面和Zookeeper相比，提供了这种图形化的界面

##### 测试2

访问： http://localhost:8006/payment/consul

会显示：springcloud with consul: 8006 f838d086-aefd-4ccb-8aba-1138f3544c53

### 服务消费者

#### 建module

新建cloud-consumerconsul-order80模块

#### 改pom

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

    <artifactId>cloud-consumerconsul-order80</artifactId>



    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
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


spring:
  application:
    name: consul-consumer-order
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderConsulMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderConsulMain80.class,args);
    }
}
```

#### 业务类

##### 配置bean

```java
package com.mg.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @LoadBalanced
    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}

```

##### controller

```java
package com.mg.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderConsulController {

    public static final String INVOME_URL = "http://consul-provider-payment";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/consul")
    public String payment (){
        String result = restTemplate.getForObject(INVOME_URL+"/payment/consul",String.class);
        return result;
    }
}
```

#### 测试

先启动cloud-providerconsul-payment8006模块，再启动cloud-consumerconsul-order80模块。

访问：http://localhost:8500/	发现两个服务都已注册上来

![53](\images\posts\springcloud\53.jpg)

访问：http://localhost/consumer/payment/consul

会显示：springcloud with consul: 8006 ae0f2854-049c-4988-9035-2459b95d5a24



## 三个注册中心异同点

![54](\images\posts\springcloud\54.jpg)

### CAP

C:Consistency(强一致性)

A:Availability(可用性)

P:Partition tolerance(分区容错)

CAP理论关注粒度是数据，而不是整体系统设计的策略

对应分布式微服务架构，P永远都要保证，所以要么是CP，要么是AP

### 经典CAP图

![55](\images\posts\springcloud\55.jpg)

![56](\images\posts\springcloud\56.jpg)

#### AP(Eureka)

![57](\images\posts\springcloud\57.jpg)

![58](\images\posts\springcloud\58.jpg)

#### CP(Zookeeper/Consul)

![59](\images\posts\springcloud\59.jpg)

![60](\images\posts\springcloud\60.jpg)

