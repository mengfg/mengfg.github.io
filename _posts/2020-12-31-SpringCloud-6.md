---
layout: post
title: SpringCloud笔记（六） Ribbon负载均衡服务调用
date: 2020-12-31
tags: SpringCloud
---

## 概述

### 什么是Ribbon

![61](\images\posts\springcloud\61.jpg)

### 官网资料

官网：https://github.com/Netflix/ribbon/wiki/Getting-Started

Ribbon目前也进入维护模式

![62](\images\posts\springcloud\62.jpg)

未来替换方案：Loadbalancer

![63](\images\posts\springcloud\63.jpg)

### 能干嘛

#### LB（负载均衡）

![64](\images\posts\springcloud\64.jpg)

#### 集中式LB

![65](\images\posts\springcloud\65.jpg)

#### 进程内LB

![66](\images\posts\springcloud\66.jpg)

一句话：Ribbon=负载均衡+RestTemplate调用



## Ribbon负载均衡演示

### 架构说明

![67](\images\posts\springcloud\67.jpg)

![68](\images\posts\springcloud\68.jpg)

总结：Ribbon其实就是一个软负载均衡的客户端组件，他可以和其他所需请求的客户端结合使用，和eureka结合只是其中的一个实例。

### POM 

前面我们讲解过了80通过轮询负载访问8001/8002，已经使用过了ribbon的功能

![69](\images\posts\springcloud\69.jpg)

![70](\images\posts\springcloud\70.jpg)

### 二说RestTemplate的使用

官网：https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

![71](\images\posts\springcloud\71.jpg)

他常用的方法有GET请求方法（读取数据，即查询），POST请求方法（写数据，即插入）。

#### getForObject方法/getForEntity方法

![72](\images\posts\springcloud\72.jpg)

#### postForObject/postForEntity

![73](\images\posts\springcloud\73.jpg)



## Ribbon核心组件IRule

IRule:根据特定算法从服务列表中选取一个要访问的服务

![74](\images\posts\springcloud\74.jpg)

### ribbon常用的算法

#### com.netflix.loadbalancer.RoundRobinRule	

轮询。默认方式

#### com.netflix.loadbalancer.RandomRule	

随机

#### com.netflix.loadbalancer.RetryRule

先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试

#### WeightedResponseTimeRule 

对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择

#### BestAvailableRule 

会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

#### AvailabilityFilteringRule 

先过滤掉故障实例，再选择并发较小的实例

#### ZoneAvoidanceRule

默认规则，复合判断server所在区域的性能和server的可用性选择服务器

### 如何替换

#### 注意配置细节

![75](\images\posts\springcloud\75.jpg)

![76](\images\posts\springcloud\76.jpg)

#### 新建package

com.mg.myrule

#### 上面包下新建MySelfRule规则类

```java
package com.mg.myrule;

import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule() {
        return new RandomRule();
    }
}
```

#### 主启动类添加@RibbonClient

```java
@SpringBootApplication
@EnableEurekaClient
//CLOUD-PAYMENT-SERVICE服务采用我们MySelfRule类中的算法规则
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

完整的项目层级结构为：

![77](\images\posts\springcloud\77.jpg)

#### 测试

访问：http://localhost/consumer/payment/getForEntity/1

多刷新几次发现，并不是我们之前的轮询方式了，而是替换成了随机方式



## Ribbon负载均衡算法

### 原理

以轮询算法为例：

![78](\images\posts\springcloud\78.jpg)

### 手写本地负载均衡器

#### 8001/8002微服务改造

controller层加入方法，用于返回端口信息，方便测试时查看是否轮询调用

```java
@GetMapping(value = "/payment/lb")
public String getPaymentLB(){
    return port;
}
```

#### 80订单微服务改造

1. ApplicationContextConfig去掉@LoadBalanced。取消默认的ribbon的负载均衡策略

   ```java
   @Configuration
   public class ApplicationContextConfig {
   
       @Bean
       //@LoadBalanced
       public RestTemplate getRestTemplate(){
           return new RestTemplate();
       }
   
   }
   ```

2. LoadBalancer接口

   在com.mg.springcloud下新建lb包，然后新建接口LoadBalancer

   ```java
   package com.mg.springcloud.lb;
   
   import org.springframework.cloud.client.ServiceInstance;
   
   import java.util.List;
   
   public interface LoadBalancer {
       //收集服务器总共有多少台能够提供服务的机器，并放到list里面
       ServiceInstance instances(List<ServiceInstance> serviceInstances);
   }
   
   ```

3. 新建LoadBalancer接口的实现类MyLB

   ```java
   package com.mg.springcloud.lb;
   
   import org.springframework.cloud.client.ServiceInstance;
   
   import java.util.List;
   import org.springframework.stereotype.Component;
   import java.util.concurrent.atomic.AtomicInteger;
   
   @Component
   public class MyLB implements LoadBalancer {
   
       private AtomicInteger atomicInteger = new AtomicInteger(0);
   
       //坐标
       private final int getAndIncrement(){
           int current;
           int next;
           do {
               current = this.atomicInteger.get();
               next = current >= 2147483647 ? 0 : current + 1;
               //第一个参数是期望值，第二个参数是修改值
           }while (!this.atomicInteger.compareAndSet(current,next)); 
           System.out.println("*******第几次访问，次数next: "+next);
           return next;
       }
   
       @Override
       //得到机器的列表
       public ServiceInstance instances(List<ServiceInstance> serviceInstances) {  
           //得到服务器的下标位置
           int index = getAndIncrement() % serviceInstances.size(); 
           return serviceInstances.get(index);
       }
   }
   ```

4. OrderController

   注入自定义接口loadBalancer，注入discoveryClient。新建getPaymentLB方法测试

   ```java
   package com.mg.springcloud.controller;
   
   import com.mg.springcloud.entities.CommonResult;
   import com.mg.springcloud.entities.Payment;
   import com.mg.springcloud.lb.LoadBalancer;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.cloud.client.ServiceInstance;
   import org.springframework.cloud.client.discovery.DiscoveryClient;
   import org.springframework.http.ResponseEntity;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   import org.springframework.web.bind.annotation.RestController;
   import org.springframework.web.client.RestTemplate;
   
   import javax.annotation.Resource;
   import java.net.URI;
   import java.util.List;
   
   @RestController
   @Slf4j
   public class OrderController {
   
   //    public static final String PAYMENT_URL = "http://localhost:8001";
       public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
   
       @Resource
       private RestTemplate restTemplate;
   
       @Autowired
       private LoadBalancer loadBalancer;
   
       @Autowired
       private DiscoveryClient discoveryClient;
   
   
       @GetMapping("/consumer/payment/create")
       public CommonResult<Payment>   create(Payment payment){
           return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment,CommonResult.class);  //写操作
       }
   
       @GetMapping("/consumer/payment/get/{id}")
       public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
           return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
       }
   
       @GetMapping("/consumer/payment/getForEntity/{id}")
       public CommonResult<Payment> getPayment2(@PathVariable("id") Long id){
           ResponseEntity<CommonResult> entity = restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
           if(entity.getStatusCode().is2xxSuccessful()) {
               return entity.getBody();
           }
   
           return new CommonResult<>(444,"操作失败");
       }
   
   
       @GetMapping(value = "/consumer/payment/lb")
       public String getPaymentLB(){
           List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
           if (instances == null || instances.size() <= 0){
               return null;
           }
           ServiceInstance serviceInstance = loadBalancer.instances(instances);
           URI uri = serviceInstance.getUri();
           return restTemplate.getForObject(uri+"/payment/lb",String.class);
       }
   }
   ```

5. 测试

   访问：http://localhost/consumer/payment/lb	发现端口是不断轮询的