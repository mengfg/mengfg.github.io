---
layout: post
title: SpringCloud笔记（十一） SpringCloud Bus 消息总线
date: 2021-01-05
tags: SpringCloud
---

## 概述

### 是什么

![164](\images\posts\springcloud\164.jpg)

Bus支持两种消息代理：RabbitMQ和Kafka

### 能干嘛

![165](\images\posts\springcloud\165.jpg)

### 为何被称为总线

![166](\images\posts\springcloud\166.jpg)



## RabbitMQ环境配置

### 安装Erlang

RabbitMQ服务端代码是使用并发式语言Erlang编写的，安装Rabbit MQ的前提是安装Erlang。

下载地址：http://erlang.org/download/otp_win64_21.3.exe

#### 安装步骤

![167](\images\posts\springcloud\167.jpg)

![168](\images\posts\springcloud\168.jpg)

### 安装RabbitMQ

下载地址：https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.14/rabbitmq-server-3.7.14.exe

#### 安装步骤

![169](\images\posts\springcloud\169.jpg)

![170](\images\posts\springcloud\170.jpg)

安装完成后，进入RabbitMQ安装目录下的sbin目录，可以看到会有很多的命令

如例我自己本机：D:\Program Files\RabbitMQ Server\rabbitmq_server-3.7.14\sbin

![171](\images\posts\springcloud\171.jpg)

#### 启动管理功能

输入以下命令启动管理功能

rabbitmq-plugins enable rabbitmq_management

![172](\images\posts\springcloud\172.jpg)

运行以上命令后，会出现以下插件：

![173](\images\posts\springcloud\173.jpg)

访问地址：http://localhost:15672/

默认用户名、密码都是guest，登录成功后显示以下界面

![174](\images\posts\springcloud\174.jpg)



## SpringCloud Bus动态刷新全局广播

必须先具备RabbitMQ环境

演示广播效果，增加复杂度，再以3355为模板再制作一个3366

### 构建环境

#### 建module

新建cloud-config-client-3366模块

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

    <groupId>com.mg.springcloud</groupId>
    <artifactId>cloud-config-client-3366</artifactId>

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

#### 写yaml

```yaml
server:
  port: 3366

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
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableEurekaClient
@SpringBootApplication
public class ConfigClientMain3366 {
    public static void main(String[] args) {
        SpringApplication.run( ConfigClientMain3366.class,args);
    }
}
```

#### 业务类

controller：

```java
package com.mg.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${server.port}")
    private String serverPort;

    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/configInfo")
    public String getConfigInfo(){
        return "serverPort:"+serverPort+"\t\n\n configInfo: "+configInfo;
    }
}
```

### 设计思想

有两种设计思想：

第一种：利用消息总线触发一个客户端/bus/refresh,而刷新所有客户端的配置

![175](\images\posts\springcloud\175.png)

第二种： 利用消息总线触发一个服务端ConfigServer的/bus/refresh端点,而刷新所有客户端的配置（更加推荐）

![176](\images\posts\springcloud\176.png)

#### 对比

图二的架构显然更加合适，图一不适合的原因如下：

1. 打破了微服务的职责单一性，因为微服务本身是业务模块，它本不应该承担配置刷新职责
2. 破坏了微服务各节点的对等性
3. 有一定的局限性。例如，微服务在迁移时，它的网络地址常常会发生变化，此时如果想要做到自动刷新，那就会增加更多的修改

### 给cloud-config-center-3344配置中心服务端添加消息总线支持

#### 修改POM

加入`spring-cloud-starter-bus-amqp`和`spring-boot-starter-actuator`

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

        <!--监控依赖-->
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

        <!--amqp依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### 修改yaml

加入rabbitmq相关配置，并暴露bus刷新配置的端点

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
          uri: https://github.com/mengfg/sprincloud-config.git
          search-paths:
            - springcloud-config
      label: master
      
#加入rabbitmq相关配置
rabbitmq:
  host: localhost
  port: 5672
  username: guest
  password: guest


eureka:
  client:
    service-url:
      defaultZone:  http://localhost:7001/eureka

#暴露bus刷新配置的端点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

### 给cloud-config-center-3355客户端添加消息总线支持

#### 修改pom

加入`spring-cloud-starter-bus-amqp`和`spring-boot-starter-actuator`

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

         <!--监控依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!--amqp依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
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

#### 修改yaml

加入rabbitmq相关配置，并暴露端点

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
      
  #rabbitmq相关配置
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

#暴露端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 给cloud-config-center-3366客户端添加消息总线支持

加入`spring-cloud-starter-bus-amqp`和`spring-boot-starter-actuator`

#### 修改pom

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

    <groupId>com.mg.springcloud</groupId>
    <artifactId>cloud-config-client-3366</artifactId>

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

        <!--监控依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!--amqp依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
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

#### 修改yaml

加入rabbitmq相关配置，并暴露端点

```yaml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344

  #rabbitmq相关配置
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka

#暴露端点
management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 测试

启动7001，启动3344，启动3355,启动3366

修改Github上配置文件。

以POST方式访问ConfigServer的/bus/refresh端点，以curl方式测试：

curl -X POST "http://localhost:3344/actuator/bus-refresh"

访问配置中心：http://config-3344.com:3344/config-dev.yml	访问正常

访问客户端：http://localhost:3355/configInfo	发现不需要重启服务即可生效

访问客户端：http://localhost:3366/configInfo	发现不需要重启服务即可生效

结论：通过上面的配置，可以实现一次修改，广播通知，处处生效



## SpringCloud Bus动态刷新定点通知

不想全部通知，只想定点通知。比如只通知3355，不通知3366。也就是指定具体某一个实例生效而不是全部

之前所有的配置都和上面相同，只是在以POST方式访问ConfigServer的/bus/refresh端点时，需要有所改变：

公式：http://localhost:配置中心的端口号/actuator/bus-refresh/{destination}

/bus/refresh请求不再发送到具体的服务实例上，而是发给config server并通过destination参数类指定需要更新配置的服务或实例

如：

curl -X POST "http://localhost:3344/actuator/bus-refresh/config-client:3355"

就会实现只通知3355，不通知3366



## 总结

![177](\images\posts\springcloud\177.jpg)

