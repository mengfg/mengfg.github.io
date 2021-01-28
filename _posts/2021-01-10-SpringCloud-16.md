---
layout: post
title: SpringCloud笔记（十六） SpringCloud Alibaba Sentinel实现熔断与限流
date: 2021-01-10
tags: SpringCloud
---

## Sentinel

官网：https://github.com/alibaba/Sentinel

中文：https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D

### 是什么

一句话解释，就是之前我们讲解过的Hystrix

![280](\images\posts\springcloud\280.jpg)

### 去哪下

https://github.com/alibaba/Sentinel/releases

此项目我们使用1.7.0版本

![281](\images\posts\springcloud\281.jpg)

### 能干嘛

![282](\images\posts\springcloud\282.jpg)

### 怎么玩

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel

服务使用中的各种问题：

服务雪崩、服务降级、服务熔断、服务限流



## 安装Sentinel控制台

sentinel组件由两部分组成

![283](\images\posts\springcloud\283.jpg)

### 安装步骤

#### 下载

https://github.com/alibaba/Sentinel/releases，下载到本地sentinel-dashboard-1.7.0.jar

#### 运行命令

前提：java8环境OK；8080端口不能被占用

命令：java -jar sentinel-dashboard-1.7.0.jar 

#### 访问sentinel管理界面

访问：http://localhost:8080 登录账号密码均为sentinel



## 初始化演示工程

### 启动Nacos8848成功

直接运行	安装目录下\bin\startup.cmd

访问：http://localhost:8848/nacos/#/login 可以成功

### 构建Module

#### 建module

新建cloudalibaba-sentinel-service8401模块

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

    <artifactId>cloudalibaba-sentinel-service8401</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.mg.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        #nacos服务注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置sentinel dashboard地址
        dashboard: localhost:8080
        port: 8719  #默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class, args);
    }
}
```

#### 业务类

```java
package com.mg.springcloud.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FlowLimitController
{
    @GetMapping("/testA")
    public String testA() {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB() {

        return "------testB";
    }

}
```

### 启动Sentinel8080

java -jar sentinel-dashboard-1.7.0

### 启动微服务8401

### 启动8401微服务后查看sentienl控制台

空空如也，啥都没有。这是因为Sentinel采用的懒加载。执行一次访问即可。例如

访问：http://localhost:8401/testA或者http://localhost:8401/testB

效果：

![284](\images\posts\springcloud\284.jpg)

发现：sentinel8080正在监控微服务8401



## 流控规则

### 基本介绍

![285](\images\posts\springcloud\285.jpg)

相关解释：

![286](\images\posts\springcloud\286.jpg)

![287](\images\posts\springcloud\287.jpg)

### 流控模式

#### 直接（默认）

直接->快速失败，系统默认选项

##### 配置及说明

![288](\images\posts\springcloud\288.jpg)

##### 测试

快速点击访问http://localhost:8401/testA	会显示页面：

![289](\images\posts\springcloud\289.jpg)

思考一个问题：

直接调用默认报错信息，技术方面OK but，是否应该有我们自己的后续处理？是否类似有一个fallback的兜底方法？

#### 关联

##### 是什么？

当关联的资源达到阈值时，就限流自己。例如，当与A关联的资源B达到阈值后，就限流自己，即B惹事，A挂了

##### 配置A

![290](\images\posts\springcloud\290.jpg)

##### postman模拟并发密集访问testB

###### 访问testB成功

![291](\images\posts\springcloud\291.jpg)

###### postman里新建多线程集合组

![292](\images\posts\springcloud\292.jpg)

###### 将访问地址添加进新线程组

其实就是模拟一段时间内资源B达到阈值后，资源A会限流。比如下订单会调用支付宝的接口，当支付宝接口挂掉了，那么下订单的接口也必须要限流

![293](\images\posts\springcloud\293.jpg)

![294](\images\posts\springcloud\294.jpg)

![295](\images\posts\springcloud\295.jpg)

![296](\images\posts\springcloud\296.jpg)

运行postman的同时，点击访问http://localhost:8401/testA，发现服务A也挂了

#### 链路

链路流控模式指的是，当从某个接口过来的资源达到限流条件时，开启限流。它的功能有点类似于针对来源配置项，区别在于：**针对来源是针对上级微服务，而链路流控是针对上级接口，也就是说它的粒度更细。**

##### 编写service

@SentinelResource注解指定资源名

```java
package com.mg.springcloud.services;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import org.springframework.stereotype.Service;

@Service
public class TestService {

    @SentinelResource(value = "service")
    public void test() {
        System.out.println("service");
    }
}

```

##### 修改Controller中的两个方法，分别调用service中的方法

```java
package com.mg.springcloud.controller;

import com.mg.springcloud.services.TestService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FlowLimitController {

    @Autowired
    private TestService testService;

    @GetMapping("/testA")
    public String testA() {
        testService.test();
        return "------testA";
    }
    
    @GetMapping("/testB")
    public String testB() {
        testService.test();
        return "------testB";
    }
    
}
```

##### 禁止收敛URL的入口context

`从1.6.3 版本开始，Sentinel Web filter默认收敛所有URL的入口context，因此链路限流不生效。
1.7.0 版本开始（对应SCA的2.1.1.RELEASE)，官方在CommonFilter 引入了WEB_CONTEXT_UNIFY 参数，用于控制是否收敛context。将其配置为 false 即可根据不同的URL 进行链路限流。SCA 2.1.1.RELEASE之后的版本,可以通过配置spring.cloud.sentinel.web-context-unify=false即可关闭收敛我们当前使用的版本是SpringCloud Alibaba 2.1.0.RELEASE，无法实现链路限流。目前官方还未发布SCA2.1.2.RELEASE，所以我们只能使用2.1.1.RELEASE，需要写代码的形式实现`

上面这段是从百度的，经过测试，SCA换成最新版本2.2.1.RELEASE仍然无效：**配置spring.cloud.sentinel.web-context-unify=false无效！！！**

推荐做法仍然是关闭官方的CommonFilter实例化，自己手动实例化CommonFilter，设置WEB_CONTEXT_UNIFY=false

步骤：

1. 将SpringCloud Alibaba的版本调整2.1.1RELEASE。目前使用的2.1.0版本没有WEB_CONTEXT_UNIFY属性

2. 配置文件中关闭sentinel官方的CommonFilter实例化

   ```yaml
   spring:
       cloud:
           sentinel:
               filter:
                   enabled: false
   ```

3. 添加配置类，自己构建CommonFilter实例

   ```java
   package com.mg.springcloud.config;
   
   import com.alibaba.csp.sentinel.adapter.servlet.CommonFilter;
   import org.springframework.boot.web.servlet.FilterRegistrationBean;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   @Configuration
   public class FilterContextConfig {
   
       @Bean
       public FilterRegistrationBean sentinelFilterRegistration() {
           FilterRegistrationBean registrationBean = new FilterRegistrationBean();
           registrationBean.setFilter(new CommonFilter());
           registrationBean.addUrlPatterns("/*");
           // 入口资源关闭聚合
           registrationBean.addInitParameter(CommonFilter.WEB_CONTEXT_UNIFY, "false");
           registrationBean.setName("sentinelFilter");
           registrationBean.setOrder(1);
   
           return registrationBean;
       }
   }
   ```

4. 控制台配置限流规则

   ![297](\images\posts\springcloud\297.jpg)

5. postman模拟并发密集访问testA

   访问http://localhost:8401/testA发现已被限流，而http://localhost:8401/testB则不受影响

### 流控效果

#### 快速失败（默认的流控处理）

直接失败，抛出异常：Blocked by Sentinel (flow limiting)

源码：`com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController`

#### Warm Up （预热）

##### 说明

公式：阈值除以coldFactor（默认值为3），经过预热时长后才会达到阈值

##### 官网

限流 冷启动：https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8

![298](\images\posts\springcloud\298.jpg)

![299](\images\posts\springcloud\299.jpg)

##### 源码

![300](\images\posts\springcloud\300.png)

##### Warmup配置

![301](\images\posts\springcloud\301.jpg)

##### 测试

多次快速点击http://localhost:8401/testB，刚开始如果超过一秒三次就会出现问题，超过5秒后，一秒超过十次才会出现问题

##### 应用场景

![302](\images\posts\springcloud\302.jpg)

#### 排队等待

匀速排队，阈值必须设置为QPS

![303](\images\posts\springcloud\303.jpg)

##### 官网

![304](\images\posts\springcloud\304.jpg)

![305](\images\posts\springcloud\305.jpg)

##### 源码

`com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController`



## 降级规则

### 官网

https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7

### 基本介绍

![306](\images\posts\springcloud\306.jpg)

![307](\images\posts\springcloud\307.jpg)

进一步说明：

![308](\images\posts\springcloud\308.jpg)

Sentinel的断路器是没有半开状态的。半开的状态系统自动去检测是否请求有异常，没有异常就关闭断路器恢复使用，有异常则继续打开断路器不可用。具体可以参考Hystrix

![309](\images\posts\springcloud\309.jpg)

![310](\images\posts\springcloud\310.jpg)



## 降级策略实战

### RT

#### 是什么

![311](\images\posts\springcloud\311.jpg)

![312](\images\posts\springcloud\312.jpg)

#### 测试

##### 代码

8401模块的controller中加入以下代码

```java
@GetMapping("/testD")
public String testD()
{
    try { 
        TimeUnit.SECONDS.sleep(1); 
    } catch (InterruptedException e) { 
        e.printStackTrace(); 
    }
    log.info("testD 测试RT");

    return "------testD";
}
```

##### 配置

![313](\images\posts\springcloud\313.jpg)

##### 压测

jmeter或者postman都可以

![314](\images\posts\springcloud\314.jpg)

![315](\images\posts\springcloud\315.jpg)

### 异常比例

#### 是什么

![316](\images\posts\springcloud\316.jpg)

![317](\images\posts\springcloud\317.jpg)

#### 测试

##### 代码

8401模块的controller中修改testD代码

```java
@GetMapping("/testD")
    public String testD()
    {
//        try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
//        log.info("testD 测试RT");
        log.info("testD 测试异常比例");
        int age = 10/0;
        return "------testD";
    }
```

##### 配置

![318](\images\posts\springcloud\318.jpg)

##### 压测

此处已postman为例

![319](\images\posts\springcloud\319.jpg)

##### 结论

![320](\images\posts\springcloud\320.jpg)

如果不采用这种降级处理方式的话，直接访问testD则会抛出一个error Page的页面

![321](\images\posts\springcloud\321.jpg)

### 异常数

#### 是什么

![322](\images\posts\springcloud\322.jpg)

![323](\images\posts\springcloud\323.jpg)

异常数是按照分钟统计的

#### 测试

##### 代码

8401模块的controller中加入以下代码

```java
@GetMapping("/testE")
public String testE(){
    log.info("testE 测试异常数");
    int age = 10/0;
    return "------testE 测试异常数";
}
```

##### 配置

![324](\images\posts\springcloud\324.jpg)

##### 压测

发现在一分钟内访问达到配置的五次报错后，服务进行熔断降级



## 热点key限流

### 官网

https://github.com/alibaba/Sentinel/wiki/热点参数限流

![325](\images\posts\springcloud\325.jpg)

### 基本介绍

![326](\images\posts\springcloud\326.jpg)

### 承上启下复习start

使用@SentinelResource自定义兜底的方法

![327](\images\posts\springcloud\327.jpg)

### 代码

源码可以查看`com.alibaba.csp.sentinel.slots.block.BlockException`

8401模块的controller中加入以下代码

```java
@GetMapping("/testHotKey")
//value属性值可以随便定义，他会出现在sentinel后台，作为资源名。blockHandler值指定兜底的方法
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {
    return "------testHotKey";
}

//兜底方法
public String deal_testHotKey (String p1, String p2, BlockException exception){
    return "------deal_testHotKey,o(╥﹏╥)o";
}
```

### 配置

![328](\images\posts\springcloud\328.jpg)

结合上面代码：`@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")`

方法testHostKey里面第一个参数只要QPS超过每秒1次，马上降级处理，并且降级处理的方法用了我们自己定义的

但是如果上面代码改为`@SentinelResource(value = "testHotKey")`，则会直接将异常打到了前台用户界面看不到，不友好。并不会调用系统自带的方法显示Blocked by Sentinel (flow limiting)。所以使用@SentinelResource注解自定义之后一定要有一个自定义的兜底的方法

![329](\images\posts\springcloud\329.jpg)

### 测试

访问：http://localhost:8401/testHotKey?p1=abc	违背配置规则会熔断降级

访问：http://localhost:8401/testHotKey?p1=abc&p2=33	违背配置规则会熔断降级

访问：http://localhost:8401/testHotKey?p2=abc	违背配置规则不会熔断降级，因为没有p1参数

### 参数例外项

上述案例演示了第一个参数p1,当QPS超过1秒1次点击后马上被限流

#### 特殊情况

普通情况下，超过1秒钟一个后，达到阈值1后马上被限流。我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样，假如当p1的值等于5时，它的阈值可以达到200

#### 配置

![330](\images\posts\springcloud\330.jpg)

添加按钮不能忘记点

![331](\images\posts\springcloud\331.jpg)

#### 测试

访问：http://localhost:8401/testHotKey?p1=5	当p1等于5的时候，阈值变为200

访问：http://localhost:8401/testHotKey?p1=3	当p1不等于5的时候，阈值就是平常的1

#### 前提条件

热点参数的注意点，参数必须是基本类型或者String

### 其他

添加异常看看，如修改testHotKey方法，加入int age = 10/0;让其产生异常

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey",blockHandler = "deal_testHotKey")
public String testHotKey(@RequestParam(value = "p1",required = false) String p1,
                         @RequestParam(value = "p2",required = false) String p2) {
    int age = 10/0;
    return "------testHotKey";
}
```

结果发现会显示error page，并不会进入到兜底方法。此知识点后面会讲解

![332](\images\posts\springcloud\332.jpg)



## 系统规则

### 是什么

https://github.com/alibaba/Sentinel/wiki/%E7%B3%BB%E7%BB%9F%E8%87%AA%E9%80%82%E5%BA%94%E9%99%90%E6%B5%81

![335](\images\posts\springcloud\335.jpg)

### 各项配置参数说明

![333](\images\posts\springcloud\333.jpg)

### 示例

以配置全局QPS为例

![334](\images\posts\springcloud\334.jpg)



## @SentinelResource

### 按资源名称限流+后续处理

#### 启动Nacos成功

#### 启动Sentinel成功

#### Module

修改cloudalibaba-sentinel-service8401模块

##### POM

添加以下依赖

```xml
<dependency>
    <groupId>com.mg.springcloud</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>${project.version}</version>
</dependency>
```

##### yaml

不变

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        #nacos服务注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置sentinel dashboard地址
        dashboard: localhost:8080
        port: 8719  #默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口
      filter:
        enabled: false
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

业务类

增加业务类RateLimitController，便于测试

```java
package com.mg.springcloud.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.BlockException;

import com.mg.springcloud.entities.*;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class RateLimitController {
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource", blockHandler = "handleException")
    public CommonResult byResource() {
        return new CommonResult(200, "按资源名称限流测试OK", new Payment(2020L, "serial001"));
    }

    public CommonResult handleException(BlockException exception) {
        return new CommonResult(444, exception.getClass().getCanonicalName() + "\t 服务不可用");
    }
}
```

##### 主启动类

不变。

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@EnableDiscoveryClient
@SpringBootApplication
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class, args);
    }
}
```

#### 配置流控规则

1秒钟内查询次数大于1，就跑到我们自定义限流处进行处理

##### 配置步骤

![336](\images\posts\springcloud\336.jpg)

#### 测试

1秒钟点击1下，OK

疯狂点击，返回了自己定义的限流处理信息，限流发送

#### 额外问题

此时关闭微服务8401看看。Sentinel控制台，流控规则消失了，并不能持久化配置，后面会讲



### 按照Url地址限流+后续处理

通过访问的URL来限流，会返回Sentinel自带默认的限流处理信息

#### 业务类

业务类RateLimitController中增加以下代码

```java
@GetMapping("/rateLimit/byUrl")
@SentinelResource(value = "byUrl")
public CommonResult byUrl() {
    return new CommonResult(200,"按url限流测试OK",new Payment(2020L,"serial002"));
}
```

#### Sentinel控制台配置

先访问一次http://localhost:8401/rateLimit/byUrl	访问正常

![337](\images\posts\springcloud\337.jpg)

#### 测试

疯狂点击http://localhost:8401/rateLimit/byUrl

![338](\images\posts\springcloud\338.jpg)



### 上面兜底方法面临的问题

![339](\images\posts\springcloud\339.jpg)



### 客户自定义限流处理逻辑

#### 自定义限流处理类

创建customerBlockHandler类用于自定义限流处理逻辑

```java
package com.mg.springcloud.myhandler;

import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.mg.springcloud.entities.*;

public class CustomerBlockHandler {

    public static CommonResult handleException(BlockException exception) {
        return new CommonResult(2020, "自定义限流处理信息....CustomerBlockHandler");

    }
}
```

#### 修改RateLimitController

添加如下代码

```java
@GetMapping("/rateLimit/customerBlockHandler")
@SentinelResource(value = "customerBlockHandler",
                  //指定调用哪个类
                  blockHandlerClass = CustomerBlockHandler.class,
                  //指定调用哪个方法
                  blockHandler = "handlerException2")
public CommonResult customerBlockHandler() {
    return new CommonResult(200,"按客戶自定义",new Payment(2020L,"serial003"));
}
```

#### 启动微服务后先调用一次

http://localhost:8401/rateLimit/customerBlockHandler

#### Sentinel控制台配置

![340](\images\posts\springcloud\340.jpg)

#### 测试

访问：http://localhost:8401/rateLimit/customerBlockHandler	超过一秒一次的频率之后就会调用CustomerBlockHandler的handlerException2()方法来处理降级

#### 进一步说明

![341](\images\posts\springcloud\341.jpg)



### 更多注解属性说明

![342](\images\posts\springcloud\342.jpg)

![343](\images\posts\springcloud\343.jpg)

#### 编码方式

除了使用注解的方式，也可以使用编码的方式来实现。编码方式只要了解即可

![345](\images\posts\springcloud\345.jpg)

##### Sentinel主要有三个核心API

SphU定义资源

Tracer定义统计

ContextUtil定义了上下文



## 服务熔断功能

sentinel整合ribbon+openFeign+fallback

### Ribbon系列

#### 提供者9003/9004

##### 建module

新建cloudalibaba-provider-payment9003和cloudalibaba-provider-payment9004模块。两个模块只是端口不同

##### 改POM

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
    <artifactId>cloudalibaba-provider-payment9003</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.mg.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
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

##### 写yml

注意两个项目的端口

```yaml
server:
  port: 9003	#9004项目改为9004

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

##### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9003.class, args);
    }
}
```

##### 业务类

为了简便此处不连接数据库了，使用hashMap来模拟

```java
package com.mg.springcloud.controller;
import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;


@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    public static HashMap<Long, Payment> hashMap = new HashMap<>();
    static{
        hashMap.put(1L,new Payment(1L,"28a8c1e3bc2742d8848569891fb42181"));
        hashMap.put(2L,new Payment(2L,"bba8c1e3bc2742d8848569891ac32182"));
        hashMap.put(3L,new Payment(3L,"6ua8c1e3bc2742d8848569891xt92183"));
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id){
        Payment payment = hashMap.get(id);
        CommonResult<Payment> result = new CommonResult(200,"from mysql,serverPort:  "+serverPort,payment);
        return result;
    }
}
```

##### 测试地址

启动nacos和sentinel

http://localhost:9003/paymentSQL/1

http://localhost:9004/paymentSQL/1

#### 消费者84

##### 建module

新建cloudalibaba-consumer-nacos-order84模块

##### 改POM

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

    <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
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

##### 写yml

```yaml
server:
  port: 84


spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719

service-url:
  nacos-user-service: http://nacos-payment-provider
```

##### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;


@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class OrderNacosMain84
{
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class, args);
    }
}
```

##### 业务类

###### ApplicationContextConfig：

```java
package com.mg.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;


@Configuration
public class ApplicationContextConfig
{
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate()
    {
        return new RestTemplate();
    }
}
```

###### CircleBreakerController：

1. 修改后请重启微服务：因为热部署对java代码级生效及时，对@SentinelResource注解内属性，有时效果不好

2. 目的：fallback管运行异常；blockHandler管配置违规

3. 测试地址：http://localhost:84/consumer/fallback/1

4. 没有任何配置

   ```java
   package com.mg.springcloud.controller;
   
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.mg.springcloud.entities.CommonResult;
   import com.mg.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class CircleBreakerController {
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @Resource
       private RestTemplate restTemplate;
   
       @RequestMapping("/consumer/fallback/{id}")
       @SentinelResource(value = "fallback")
   
       public CommonResult<Payment> fallback(@PathVariable Long id) {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
   
           if (id == 4) {
               throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
           } else if (result.getData() == null) {
               throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
           }
           return result;
       }
   }
   ```

   访问：http://localhost:84/consumer/fallback/1	访问正常

   访问：http://localhost:84/consumer/fallback/4	会直接抛出错误页面

5. 只配置fallback

   fallback只负责业务异常

   ```java
   package com.mg.springcloud.controller;
   
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.mg.springcloud.entities.CommonResult;
   import com.mg.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class CircleBreakerController {
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @Resource
       private RestTemplate restTemplate;
   
       @RequestMapping("/consumer/fallback/{id}")
       @SentinelResource(value = "fallback",fallback = "handlerFallback")
       public CommonResult<Payment> fallback(@PathVariable Long id) {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
   
           if (id == 4) {
               throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
           } else if (result.getData() == null) {
               throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
           }
   
           return result;
       }
   
       //fallback
       public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
       }
   }
   ```

   访问：http://localhost:84/consumer/fallback/1	访问正常

   访问：http://localhost:84/consumer/fallback/4	会调用自己定义的handlerFallback方法处理

6. 只配置blockHandler

   blockHandler只负责sentinel控制台的配置违规

   ```java
   package com.mg.springcloud.controller;
   
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.alibaba.csp.sentinel.slots.block.BlockException;
   import com.mg.springcloud.entities.CommonResult;
   import com.mg.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class CircleBreakerController {
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @Resource
       private RestTemplate restTemplate;
   
       @RequestMapping("/consumer/fallback/{id}")
       @SentinelResource(value = "fallback",blockHandler = "blockHandler")
       public CommonResult<Payment> fallback(@PathVariable Long id) {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
   
           if (id == 4) {
               throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
           } else if (result.getData() == null) {
               throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
           }
   
           return result;
       }
   
       //blockHandler
       public CommonResult blockHandler(@PathVariable  Long id,BlockException blockException) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
       }
   
   }
   ```

   配置sentinel的降级规则

   ![346](\images\posts\springcloud\346.jpg)

   访问：http://localhost:84/consumer/fallback/1	访问正常

   访问：http://localhost:84/consumer/fallback/4	第一访问会报errorpage，如果多次访问并且符合配置的sentinel规则的话则会调用自定义的blockHandler方法处理

7. fallback和blockHandler都配置

   如果两个属性都配置，超过sentinel配置的规则还是以控制台规则的blockHandler为准

   ```java
   package com.mg.springcloud.controller;
   
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.alibaba.csp.sentinel.slots.block.BlockException;
   import com.mg.springcloud.entities.CommonResult;
   import com.mg.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class CircleBreakerController {
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @Resource
       private RestTemplate restTemplate;
   
       @RequestMapping("/consumer/fallback/{id}")
       @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler")
       public CommonResult<Payment> fallback(@PathVariable Long id) {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
   
           if (id == 4) {
               throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
           } else if (result.getData() == null) {
               throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
           }
   
           return result;
       }
   
   //    fallback
       public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
       }
   
       //blockHandler
       public CommonResult blockHandler(@PathVariable  Long id,BlockException blockException) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
       }
   
   }
   ```

   配置sentinel控制台规则

   ![348](\images\posts\springcloud\348.jpg)

   访问：http://localhost:84/consumer/fallback/1	访问正常，如果多次访问并且符合配置的sentinel规则的话则会调用自定义的blockHandler方法处理

   访问：http://localhost:84/consumer/fallback/4	第一访问会调用自定义的fallback方法处理，如果多次访问并且符合配置的sentinel规则的话则会调用自定义的blockHandler方法处理

   ![347](\images\posts\springcloud\347.jpg)

8. exceptionsToIgnore：忽略属性

   ```java
   package com.mg.springcloud.controller;
   
   import com.alibaba.csp.sentinel.annotation.SentinelResource;
   import com.alibaba.csp.sentinel.slots.block.BlockException;
   import com.mg.springcloud.entities.CommonResult;
   import com.mg.springcloud.entities.Payment;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   
   @RestController
   @Slf4j
   public class CircleBreakerController {
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @Resource
       private RestTemplate restTemplate;
   
       @RequestMapping("/consumer/fallback/{id}")
       @SentinelResource(value = "fallback",fallback = "handlerFallback",blockHandler = "blockHandler",exceptionsToIgnore = {IllegalArgumentException.class})
   
       public CommonResult<Payment> fallback(@PathVariable Long id) {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);
   
           if (id == 4) {
               throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
           } else if (result.getData() == null) {
               throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
           }
   
           return result;
       }
   
   //    fallback
       public CommonResult handlerFallback(@PathVariable  Long id,Throwable e) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(444,"兜底异常handlerFallback,exception内容  "+e.getMessage(),payment);
       }
   
       //blockHandler
       public CommonResult blockHandler(@PathVariable  Long id,BlockException blockException) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(445,"blockHandler-sentinel限流,无此流水: blockException  "+blockException.getMessage(),payment);
       }
   
   }
   ```

   ![349](\images\posts\springcloud\349.jpg)

访问：http://localhost:84/consumer/fallback/4	如果报异常则不会调用自定义的fallback方法了，会直接抛出error page



### Feign系列

修改84模块。84消费者调用提供者9003，Feign组件一般在消费侧

#### POM

添加以下依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

#### yaml

添加sentinel对feign的支持

```yaml
#对Feign的支持
feign:
  sentinel:
    enabled: true
```

#### 业务类

带@FeignClient注解的业务接口

```java
package com.mg.springcloud.services;

import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import com.mg.springcloud.services.impl.PaymentFallbackService;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;


@FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
public interface PaymentService {
    //实际上调用的是9003的方法
    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}
```

PaymentFallbackService实现类：用于定义兜底方法

```java
package com.mg.springcloud.services.impl;

import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import com.mg.springcloud.services.PaymentService;
import org.springframework.stereotype.Component;

@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id) {
        return new CommonResult<>(44444,"服务降级返回,---PaymentFallbackService",new Payment(id,"errorSerial"));
    }
}
```

Controller中添加：

```java
@Resource
private PaymentService paymentService;

@GetMapping(value = "/consumer/paymentSQL/{id}")
public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
    return paymentService.paymentSQL(id);
}
```

#### 主启动类

添加@EnableFeignClients启动Feign的功能

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;


@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class OrderNacosMain84
{
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class, args);
    }
}
```

#### 测试

启动nacos

启动sentinel

启动84 和 9003

访问：http://localhost:84/consumer/paymentSQL/1	84访问9003正常

测试84调用9003，此时故意关闭9003微服务提供者，看84消费侧自动降级，不会被耗死

访问：http://localhost:84/consumer/paymentSQL/1	会调用PaymentFallbackService类中定义的方法来处理降级



### 熔断框架比较

![350](\images\posts\springcloud\350.jpg)

![351](\images\posts\springcloud\351.jpg)



## 规则持久化

### 是什么

一旦我们重启应用，Sentinel规则将消失，生产环境需要将配置规则进行持久化

### 怎么玩

将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台的流控规则就能看到，只要Nacos里面的配置不删除，针对8401上Sentinel上的流控规则持续有效

### 步骤

修改cloudalibaba-sentinel-service8401

#### POM

添加以下依赖

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

#### yaml

添加Nacos数据源配置

```yaml
spring:
   cloud:
    sentinel:
    	datasource:
            ds1:
              nacos:
                server-addr: localhost:8848
                dataid: ${spring.application.name}
                groupid: DEFAULT_GROUP
                data-type: json
                rule-type: flow
```

全部配置

```yaml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        #nacos服务注册中心地址
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置sentinel dashboard地址
        dashboard: localhost:8080
        port: 8719  #默认8719，假如被占用了会自动从8719开始依次+1扫描。直至找到未被占用的端口
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataid: ${spring.application.name}
            groupid: DEFAULT_GROUP
            data-type: json
            rule-type: flow
      filter:
        enabled: false
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

#### 添加Nacos业务规则配置

![352](\images\posts\springcloud\352.jpg)

![353](\images\posts\springcloud\353.jpg)

##### 内容解析

```json
[
    {
         "resource": "/rateLimit/byUrl",
         "limitApp": "default",
         "grade":   1,
         "count":   1,
         "strategy": 0,
         "controlBehavior": 0,
         "clusterMode": false    
    }
]
```

![354](\images\posts\springcloud\354.jpg)

#### 测试

启动8401后，访问：http://localhost:8401/rateLimit/byUrl

刷新sentinel发现业务规则有了

![355](\images\posts\springcloud\355.jpg)

快速访问测试接口http://localhost:8401/rateLimit/byUrl	QPS超过1秒1次则显示默认的

![356](\images\posts\springcloud\356.jpg)

停止8401再看sentinel

![357](\images\posts\springcloud\357.jpg)

重新启动8401再看sentinel

扎一看还是没有，稍等一会儿。多次调用：http://localhost:8401/rateLimit/byUrl

重新配置出现了，持久化验证通过

