---
layout: post
title: SpringCloud笔记（十） SpringCloud config分布式配置中心
date: 2021-01-04
tags: SpringCloud
---

## 概述

### 分布式系统面临的配置问题

![155](\images\posts\springcloud\155.jpg)

### 是什么

![156](\images\posts\springcloud\156.jpg)

![157](\images\posts\springcloud\157.jpg)

### 能干嘛

1. 集中管理配置文件
2. 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
3. 当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
4. 将配置信息以REST接口的形式暴露。post、curl访问刷新均可....

### 与Github整合配置

Config默认使用Git来存储配置文件（也有其它方式，比如支持svn和本地文件，但最推荐的还是Git，而且使用的是http/https访问的形式）

### 官网

https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/

![158](\images\posts\springcloud\158.jpg)



## Config服务端配置与测试

### 准备工作

本地新建文件夹并新建三个文件config-dev.yml，config-test.yml，config-prod.yml，Github上新建一个名为springcloud-config的新Repository。创建一个分支dev，本地文件夹与GItHub同步。

### 构建配置中心模块

#### 建module

新建cloud-config-center-3344模块

#### 改pom

加入`spring-cloud-config-server`表示service端

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

    <artifactId>cloud-config-center-3344</artifactId>

    <dependencies>


        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
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
  port: 3344
spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          #github上新建仓库的地址
          uri: https://github.com/mengfg/springcloud-config.git
          search-paths:
            - springcloud-config
      label: master
eureka:
  client:
    service-url:
      defaultZone:  http://localhost:7001/eureka
```

#### 主启动类

加入@EnableConfigServer注解

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigCenterMain3344 .class,args);
    }
}
```

#### windows下修改hosts文件，增加映射

这步可以选做，在C:\Windows\System32\drivers\etc下的hosts中添加

```
127.0.0.1  config-3344.com
```

#### 测试

启动微服务7001，启动微服务3344

访问：http://localhost:3344/master/config-prod.yml	可以读取出github中相关文件的内容

### 配置读取规则

#### 官网

![159](\images\posts\springcloud\159.jpg)

![160](\images\posts\springcloud\160.jpg)

![161](\images\posts\springcloud\161.jpg)

#### /{label}/{application}-{profile}.yml

推荐使用这种方式：

master分支：

http://config-3344.com:3344/master/config-test.yml

dev分支：
http://config-3344.com:3344/dev/config-test.yml

#### /{application}-{profile}.yml

默认会去读取master分支。也可以通过修改微服务3344的yaml配置文件指定

```yaml
spring:
  cloud:
    config:
      label: master
```

http://config-3344.com:3344/config-test.yml

#### /{application}/{profile}[/{label}]

http://config-3344.com:3344/config/test/master

这种方式会以json串的方式返回



## Config客户端配置与测试

### 建module

新建cloud-config-client-3355模块

### 改pom

加入，`spring-cloud-starter-config`表示客户端

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

    <artifactId>cloud-config-client-3355</artifactId>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
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

### 写yml

注意：新建的是bootstrap.yml

![162](\images\posts\springcloud\162.jpg)

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
  	#config客户端配置
  	#以下配置综合起来就是读取master分支上的config-dev.yml 即http://localhost:3344/master/config-dev.yml
    config:
      label: master	#分支名称
      name: config	#配置文件名称
      profile: dev	#读取环境
      uri: http://localhost:3344	#配置中心地址
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
```

配置说明：

![163](\images\posts\springcloud\163.jpg)

### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ConfigClientMain3355 {
    public static void main(String[] args) {
        SpringApplication.run( ConfigClientMain3355.class,args);
    }
}
```

### 业务类

```java
package com.mg.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

### 测试

启动7001，启动Config配置中心3344微服务，启动3355作为Client，

访问：http://config-3344.com:3344/master/config-dev.yml	发现自测是正常的

访问：http://localhost:3355/configInfo	客户端也是可以正常访问到

### 存在问题

Linux运维修改GitHub上的配置文件内容做调整，

刷新3344，自测发现ConfigServer配置中心立刻响应，

刷新3355，发现ConfigServer客户端没有任何响应

3355没有变化除非自己重启或者重新加载。



## Config客户端之动态刷新

避免上面提到的问题：每次更新配置都要重启客户端微服务3355

### 修改3355模块

#### POM引入actuator监控

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### 修改YML，暴露监控端口

```yaml
server:
  port: 3355

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

#加入如下配置
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

#### 业务类Controller修改

加入@RefreshScope注解

```java
package com.mg.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
```

#### POST请求刷新3355

必须是post请求

curl -X POST "http://localhost:3355/actuator/refresh"

#### 存在的问题

假如有多个微服务客户端3355/3366/3377。。。。每个微服务都要执行一次post请求，手动刷新？

可否广播，一次通知，处处生效？或者有100个微服务，我们只想通知其中的90个。

这就是涉及到下面一讲：消息总线