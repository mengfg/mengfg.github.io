---
layout: post
title: SpringCloud笔记（三） Eureka服务注册与发现
date: 2020-12-28
tags: SpringCloud
---

## Eureka基础知识

### 什么是服务治理

Spring Cloud封装了Netflix 公司开发的Eureka模块来实现服务治理

在传统的rpc远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册。

### 什么是服务注册与发现

Eureka采用了CS的设计架构，Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server 来监控系统中各个微服务是否正常运行。

在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息，比如服务地址通讯地址等以别名方式注册到注册中心上。另一方(消费者|服务提供者)，以该别名的方式去注册中心上获取到实际的服务通讯地址，然后再实现本地RPC调用RPC远程调用框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何rpc远程框架中，都会有一个注册中心(存放服务地址相关信息(接口地址)）

![26](\images\posts\springcloud\26.jpg)

### Eureka两组件

Eureka包含两个组件: Eureka Server和Eureka Client

Eureka Server提供服务注册服务

各个微服务节点通过配置启动后,会在EurekaServer中进行注册， 这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

EurekaClient通过注册中心进行访问

是一个Java客户端， 用于简化Eureka Server的交互， 客户端同时也具备一 个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除(默认90秒)。

## 单机Eureka构建步骤

### eurekaServer端服务注册中心

IDEA生成eurekaServer端服务注册中心类似物业公司

#### 建Module

新建模块cloud-eureka-server7001

#### 改POM

在引入eureka时，以前的老版本导入时，现在不要用了

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

现在使用新版本

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

完整文件：

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

    <artifactId>cloud-eureka-server7001</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>

    </dependencies>
</project>

```

#### 写yml

```yaml
server:
  port: 7001

eureka:
  instance:
    #eureka服务端的实例名字
    hostname: localhost
  client:
    #表识不向注册中心注册自己
    register-with-eureka: false
    #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/

```

#### 主启动

```java
package com.mg.springcloud;

import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

import org.springframework.boot.SpringApplication;

//表明这是服务注册中心
@EnableEurekaServer
@SpringBootApplication
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
```

#### 测试

http://localhost:7001/

结果页面：

![27](\images\posts\springcloud\27.jpg)

### cloud-provider-payment8001模块注册进EurekaServer

EurekaClient端cloud-provider-payment8001将注册进EurekaServer成为服务提供者provider，类似商户入驻进大厦

在cloud-provider-payment8001模块的基础上修改：

#### 修改pom

添加进eureka-client相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

之前的版本(现在不要用了)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>
```

#### 修改yml

添加eureka-client相关配置

```yaml
eureka:
  client:
    #表示是否将自己注册进eurekaserver，默认为true
    register-with-eureka: true
    #是否从eurekaserver抓去已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

#### 修改启动类

添加@EnableEurekaClient注解

```java
@EnableEurekaClient
@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

#### 测试

先要启动EurekaServer，再启动cloud-provider-payment8001模块。

http://localhost:7001/

发现服务已经注册上来了，注册的名字就是我们配置文件中spring.application.name配置的名字

```yaml
spring:
  application:
    name: cloud-payment-service
```

![28](\images\posts\springcloud\28.jpg)

### cloud-consumer-order80模块注册进EurekaServer

EurekaClient端cloud-consumer-order80将注册进EurekaServer成为服务消费者consumer,类似顾客来大厦消费

在cloud-consumer-order80模块的基础上修改：

#### 修改pom

添加进eureka-client相关依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

#### 修改yml

添加eureka-client相关配置，并设置应用的名称

```yaml
server:
  port: 80

#应用名称
spring:
  application:
    name: cloud-order-service

#eureka-client相关配置
eureka:
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

#### 修改启动类

添加@EnableEurekaClient注解

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }

}
```

#### 测试

先要启动EurekaServer，再启动cloud-provider-payment8001模块，再启动cloud-consumer-order80模块。

http://localhost/consumer/payment/get/1	发现之前的功能都能正常运行



## 集群Eureka构建步骤

### Eureka集群原理说明

![29](\images\posts\springcloud\29.jpg)

解决办法：搭建Eureka注册中心集群，实现负载均衡+故障容错

原则：互相注册，互相守望

### EurekaServer集群环境构建步骤

#### 建module

新建cloud-eureka-server7002模块

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

    <artifactId>cloud-eureka-server7002</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>

    </dependencies>
</project>
```

#### 修改映射配置

找到C:\Windows\System32\drivers\etc路径下的hosts文件，添加以下内容到文件中

```
127.0.0.1  eureka7001.com
127.0.0.1  eureka7002.com
```

#### 写YML

修改cloud-eureka-server7001模块。注意端口和eureka服务端的实例名字

```yaml
server:
  port: 7001

eureka:
  instance:
    #eureka服务端的实例名字
    hostname: eureka7001.com
  client:
    #表识不向注册中心注册自己
    register-with-eureka: false
    #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://eureka7002.com:7002/eureka/

```

修改cloud-eureka-server7002模块。注意端口和eureka服务端的实例名字

```yaml
server:
  port: 7002

eureka:
  instance:
    #eureka服务端的实例名字
    hostname: eureka7002.com
  client:
    #表识不向注册中心注册自己
    register-with-eureka: false
    #表示自己就是注册中心，职责是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与eureka server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://eureka7001.com:7001/eureka/

```

#### 启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@EnableEurekaServer
@SpringBootApplication
public class EurekaMain7002 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7002.class,args);
    }
}
```

#### 测试

分别启动cloud-eureka-server7001模块和cloud-eureka-server7002模块。

![30](\images\posts\springcloud\30.jpg)

![31](\images\posts\springcloud\31.jpg)

#### 将支付服务8001微服务发布到上面2台Eureka集群配置中

修改yaml

```yaml
service-url:
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
```

#### 将订单服务80微服务发布到上面2台Eureka集群配置中

修改yaml

```yaml
service-url:
	#defaultZone: http://localhost:7001/eureka
	defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
```

#### 测试

先要启动EurekaServer，7001/7002服务

再要启动服务提供者provider，8001服务

再要启动消费者，80

http://localhost/consumer/payment/get/1	发现之前的功能正常运行

### 支付服务提供者8001集群环境构建

#### 建Module

新建cloud-provider-payment8002模块

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

    <artifactId>cloud-provider-payment8002</artifactId>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
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

        <dependency>
            <groupId>com.mg.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

    </dependencies>

</project>
```

#### 写yaml

```yaml
server:
  port: 8002


spring:
  application:
    name: cloud-payment-service
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
      #defaultZone: http://localhost:7001/eureka
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版


mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.mg.springcloud.entities  #所有Entity别名类所在包
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableEurekaClient
@SpringBootApplication
public class PaymentMain8002 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8002.class,args);
    }
}

```

#### 业务类

直接将cloud-provider-payment8001模块中的业务类复制过来即可。包括dao，service，controller，mapper

修改cloud-provider-payment8001模块和cloud-provider-payment8002模块中的controller，方便看出效果：

获取到配置文件的端口，用于查看到底调用的是哪个服务提供者

```java
@RestController
@Slf4j
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    //获取到配置文件的端口，用于查看到底调用的是哪个服务提供者
    @Value("${server.port}")
    private String port;

    @PostMapping(value = "/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        int result = paymentService.create(payment);
        log.info("*****插入结果："+result);
        if (result>0){  //成功
            //返回数据时加入端口信息
            return new CommonResult(200,"插入数据库成功"+port,result);
        }else {
            return new CommonResult(444,"插入数据库失败",null);
        }
    }
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("*****查询结果："+payment);
        if (payment!=null){  //说明有数据，能查询成功
            //返回数据时加入端口信息
            return new CommonResult(200,"查询成功"+port,payment);
        }else {
            return new CommonResult(444,"没有对应记录，查询ID："+id,null);
        }
    }
}
```

#### 负载均衡

修改cloud-consumer-order80中的配置

配置类ApplicationContextConfig中添加RestTemplate组件时加入@LoadBalanced，开启负载均衡

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

controller中调用的地址修改成在注册中心注册服务名称

```java
@RestController
@Slf4j
public class OrderController {

    //    public static final String PAYMENT_URL = "http://localhost:8001";
    //修改成在注册中心注册服务名称。负载均衡会采用轮询机制分别调用两个服务提供者
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment>   create(Payment payment){
        return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment,CommonResult.class);  //写操作
    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
        return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
    }
}
```

#### 测试

先要启动EurekaServer，7001/7002服务

再要启动服务提供者provider，8001/8002服务

再启动服务消费者，80服务

http://localhost/consumer/payment/get/1 不断刷新访问，发现会采用轮询机制分别调用两个服务提供者

Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还有负载功能了



## actuator微服务信息完善

目前服务注册在注册中心后，会有两个小问题

![32](\images\posts\springcloud\32.jpg)

### 主机名称：服务名称修改

以8001服务和8002服务为例。yaml文件中添加

```yaml
eureka:
  instance:
      instance-id: payment8001
```

```yaml
eureka:
  instance:
      instance-id: payment8002
```

### 访问信息有ip信息提示

以8001服务和8002服务为例。yaml文件中添加

```yaml
eureka:
  instance:
      prefer-ip-address: true
```

### 效果

修改完成后效果

![33](\images\posts\springcloud\33.jpg)



## 服务发现Discovery

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

以cloud-provider-payment8001为例：

### controller

修改cloud-provider-payment8001的Controller，加入以下代码，注意导入的是org.springframework.cloud.client.discovery包下的DiscoveryClient

```java
@RestController
@Slf4j
public class PaymentController {

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping(value = "/payment/discovery")
    public Object discovery(){
        //获取所有service名称
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("***** element:"+element);
        }
        
        //获取CLOUD-PAYMENT-SERVICE下的所有实例
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }
        return this.discoveryClient;
    }
}
```

### 启动类

加入@EnableDiscoveryClient注解

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@EnableEurekaClient
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

### 测试

先要启动EurekaServer，7001/7002服务

再启动8001主启动类

http://localhost:8001/payment/discovery

会发现后台打印

![34](\images\posts\springcloud\34.jpg)



## Eureka自我保护

### 故障现象

![35](\images\posts\springcloud\35.jpg)

### 导致原因

一句话：某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存

属于CAP里面的AP分支

![36](\images\posts\springcloud\36.jpg)

![37](\images\posts\springcloud\37.jpg)

![38](\images\posts\springcloud\38.jpg)

![39](\images\posts\springcloud\39.jpg)

### 怎么禁止自我保护

一般生产环境中不会禁止自我保护

#### 注册中心

以eureakeServer端7001为例，修改yaml文件，加入`eureka.server.enable-self-preservation = false`，该选项默认是true

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      #为方便测试暂时改为单机版
      #defaultZone: http://eureka7002.com:7002/eureka/
      defaultZone: http://eureka7001.com:7001/eureka/
  server:
    #关闭自我保护机制，保证不可用服务及时被剔除
    enable-self-preservation: false
    #eureka server清理无效节点的时间间隔
    eviction-interval-timer-in-ms: 2000
```

关闭后注册中心的效果：

![40](\images\posts\springcloud\40.jpg)

#### 客户端

以生产者客户端eureakeClient端8001为例。修改yaml文件，加入`lease-renewal-interval-in-seconds`和`lease-expiration-duration-in-seconds`选项

```yaml
server:
  port: 8001


spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: org.gjt.mm.mysql.Driver
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root

eureka:
  client:
    register-with-eureka: true
    fetchRegistry: true
    service-url:
      #为方便测试暂时改为单机版
      defaultZone: http://localhost:7001/eureka
      #defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka  #集群版
  instance:
    instance-id: payment8001
    prefer-ip-address: true
    #Eureka客户端向服务器端发送心跳的时间间隔，单位为秒（默认30秒）
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳等待时间上限，单位为秒（默认90秒），超时将剔除服务
    lease-expiration-duration-in-seconds: 2


mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.mg.springcloud.entities
```

#### 测试

先启动7001再启动8001，发现8001已经注册在7001了，此时关闭掉8001，再次刷新7001，发现服务会立马被清理掉