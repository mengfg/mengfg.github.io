---
layout: post
title: SpringCloud笔记（十五） SpringCloud Alibaba Nacos服务注册和配置中心
date: 2021-01-09
tags: SpringCloud
---

## Nacos简介

### 为什么叫Nacos

前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service

### 是什么

一个更易于构建云原生应用的动态服务发现，配置管理和服务管理中心

Nacos：Dynamic Naming and Configuration Service

Nacos就是注册中心+配置中心的组合，Nacos = Eureka+Config+Bus

### 能干嘛

替代Eureka做服务注册中心

替代Config做服务配置中心

### 去哪下

https://github.com/alibaba/Nacos

官方文档：

https://nacos.io/zh-cn/index.html

https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_nacos_discovery

### 各种注册中心比较

![216](\images\posts\springcloud\216.jpg)



## 安装并运行Nacos

本地Java8+Maven环境已经OK

先从官网下载Nacos：https://github.com/alibaba/nacos/releases/tag/1.1.4 （本次示例以1.1.4为例）

解压安装包，直接运行bin目录下的startup.cmd

命令运行成功后直接访问http://localhost:8848/nacos  默认账号密码都是nacos

结果页面：

![217](\images\posts\springcloud\217.jpg)



## Nacos作为服务注册中心演示

### 基于Nacos的服务提供者

#### 建Module

新建cloudalibaba-provider-payment9001模块

#### 改POM

父POM

```xml
<!--springcloudalibaba2.1.0.RELEASE-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.1.0.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

本模块POM，加入`spring-cloud-starter-alibaba-nacos-discovery`

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
    <artifactId>cloudalibaba-provider-payment9001</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 9001

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

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class,args);
    }
}
```

#### 业务类

```java
package com.mg.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PaymentController
{
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id)
    {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}
```

#### 测试

启动9001

访问nacos控制台，发现服务已经注册上来了

![218](\images\posts\springcloud\218.jpg)

访问：http://localhost:9001/payment/nacos/1	发现可以正常访问

以上说明nacos服务注册中心+服务提供者9001都ok了



为了下一章节演示nacos的负载均衡，参照9001新建9002。此处想取巧不想新建重复体力劳动，直接拷贝虚拟端口映射：

![219](\images\posts\springcloud\219.jpg)

![220](\images\posts\springcloud\220.jpg)

此处为了保证质量，我们还是按部就班的新建9002，创建步骤和9001相同，只是端口和主启动类名需要注意下

#### 建module

新建cloudalibaba-provider-payment9002模块

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
    <artifactId>cloudalibaba-provider-payment9002</artifactId>

    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
  port: 9002

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

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9002 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9002.class,args);
    }
}

```

#### 业务类

```java
package com.mg.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class PaymentController
{
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id)
    {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}
```

启动后，同样会注册到nacos

![221](\images\posts\springcloud\221.jpg)

点击详情后能看到详细信息：

![222](\images\posts\springcloud\222.jpg)

访问：http://localhost:9002/payment/nacos/1	同样可以访问成功

### 基于Nacos的服务消费者

#### 建module

新建cloudalibaba-consumer-nacos-order83模块

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
    <artifactId>cloudalibaba-consumer-nacos-order83</artifactId>

    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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

为什么nacos支持负载均衡？是因为底层集成了ribbon的依赖

![223](\images\posts\springcloud\223.jpg)

#### 写yaml

```yaml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

service-url:
  nacos-user-service: http://nacos-payment-provider
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class OrderNacosMain83 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
}
```

#### 业务类

ApplicationContextConfig：用于创建RestTemplate对象

```java
package com.mg.springcloud.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

controller：

```java
package com.mg.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;


@RestController
@Slf4j
public class OrderNacosController
{
    @Resource
    private RestTemplate restTemplate;

    //读取配置文件中的内容，不在代码中硬编码了
    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping(value = "/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id)
    {
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }
}
```

#### 测试

nacos控制台：

![224](\images\posts\springcloud\224.jpg)

访问：http://localhost:83/consumer/payment/nacos/1	83轮询访问9001/9002，轮询负载OK

### 服务注册中心对比

Nacos全景图所示：

![225](\images\posts\springcloud\225.jpg)

Nacos和CAP：

![226](\images\posts\springcloud\226.jpg)

![227](\images\posts\springcloud\227.jpg)

切换：

Nacos支持AP和CP模式的切换

![228](\images\posts\springcloud\228.jpg)

切换时需要使用PUT请求发送：

curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'



## Nacos作为服务配置中心演示

### Nacos作为配置中心-基础配置

#### 建module

新建cloudalibaba-config-nacos-client3377模块

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
    <artifactId>cloudalibaba-config-nacos-client3377</artifactId>


    <dependencies>
        <!--nacos-config-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--nacos-discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--web + actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般基础配置-->
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

需要写两个yaml，bootstrap.yml和application.yml。他们两个配置文件的内容会组合生效，如果存在相同配置在bootstrap.yml优先级高

为什么需要写两个yaml？

![229](\images\posts\springcloud\229.jpg)

bootstrap.yml：

```yaml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #服务注册中心地址
      config:
        server-addr: localhost:8848 #配置中心地址
        file-extension: yaml #指定yaml格式的配置
```

application.yml

```yaml
spring:
  profiles:
    active: dev
```

#### 主启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;


@EnableDiscoveryClient
@SpringBootApplication
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```

#### 业务类

需要加入@RefreshScope注解，通过springcloud原生注解@RefreshScope实现配置的自动更新

```java
package com.mg.springcloud.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RefreshScope
public class ConfigClientController
{
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```

#### 在Nacos中添加配置信息

##### 理论

Nacos中的dataid的组成格式与SpringBoot配置文件中的匹配规则：

官网介绍：https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html

![230](\images\posts\springcloud\230.jpg)

![231](\images\posts\springcloud\231.jpg)

![234](\images\posts\springcloud\234.jpg)

##### 实操

本项目中

spring.application.name 为 nacos-config-client

spring.profiles.active 为 dev

spring.cloud.nacos.config.file-extension 为 yaml

所以在nacos中新建配置的Data Id为nacos-config-client-dev.yaml

![232](\images\posts\springcloud\232.jpg)

![233](\images\posts\springcloud\233.jpg)

#### 测试

运行cloud-config-nacos-client3377的主启动类

调用接口查看配置信息：http://localhost:3377/config/info

#### 自带动态刷新

修改下Nacos中的yaml配置文件，再次调用查看配置的接口，就会发现配置已经刷新



### Nacos作为配置中心-分类配置

#### 存在问题

![235](\images\posts\springcloud\235.jpg)

#### Nacos的图形化管理界面

配置管理：

![236](\images\posts\springcloud\236.jpg)

命名空间：

![237](\images\posts\springcloud\237.jpg)

#### Namespace、Group、Data ID三者关系？为什么这么设计？

![238](\images\posts\springcloud\238.jpg)

![239](\images\posts\springcloud\239.jpg)

#### 示例

##### DataID方案

指定spring.profile.active和配置文件的DataID来使不同环境下读取不同的配置。

###### 步骤

默认空间+默认分组+新建dev和test两个DataID：

1. 新建dev配置DataID

![240](\images\posts\springcloud\240.jpg)

2. 新建test配置DataID

![241](\images\posts\springcloud\241.jpg)

最终nacos的配置列表中会存在两个配置文件

![242](\images\posts\springcloud\242.jpg)

3. 通过spring.profile.active属性就能进行多环境下配置文件的读取

![243](\images\posts\springcloud\243.jpg)

4. 测试

   访问：http://localhost:3377/config/info	发现配置文件配置是什么就加载什么

##### Group方案

通过Group实现环境区分

新建TEST_GROUP

![244](\images\posts\springcloud\244.jpg)

新建DEV_GROUP

![245](\images\posts\springcloud\245.jpg)

最终nacos的配置列表中会存在两个data ID相同但分组不同的配置文件

![246](\images\posts\springcloud\246.jpg)

修改bootstrap.yml和application.yml，在config下增加一条group的配置即可。可配置为DEV_GROUP或TEST_GROUP

![247](\images\posts\springcloud\247.jpg)

##### Namespace方案

新建dev和test的Namespace

![248](\images\posts\springcloud\248.png)

新建完成后，命名空间列表中会出现新建的命名空间，并且会有相应的ID号

![249](\images\posts\springcloud\249.png)

回到服务管理-服务列表查看

![250](\images\posts\springcloud\250.jpg)

dev的命名空间下新建三个配置文件，dataID相同，但group不同

![251](\images\posts\springcloud\251.jpg)

修改bootstrap.yml和application.yml

![252](\images\posts\springcloud\252.jpg)



## Nacos集群和持久化配置（重要）

### 官网说明

https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html

官网架构图：

![253](\images\posts\springcloud\253.jpg)

上图翻译成便于理解的示意图为：

![254](\images\posts\springcloud\254.png)

按照上述，我们需要mysql数据库：

https://nacos.io/zh-cn/docs/deployment.html

![255](\images\posts\springcloud\255.jpg)

![256](\images\posts\springcloud\256.jpg)

![257](\images\posts\springcloud\257.jpg)

### Nacos持久化配置解释

Nacos默认自带的是嵌入式数据库derby：

https://github.com/alibaba/nacos/blob/develop/config/pom.xml	文件中会有derby的依赖坐标

#### derby到mysql切换配置步骤

1. nacos安装目录下\conf目录下找到sql脚本nacos-mysql.sql

   ![258](\images\posts\springcloud\258.jpg)

   新建一个数据库名叫nacos_config，直接粘贴nacos-mysql.sql中的内容执行即可

2. nacos安装目录下\conf目录下找到application.properties

   ![259](\images\posts\springcloud\259.jpg)

   将如下内容粘贴过去，并将用户名、密码等改成自己机器的

   ```properties
   spring.datasource.platform=mysql
    
   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
   db.user=root
   db.password=root
   ```

   重新启动nacos，发现看到的是个全新的空记录界面，之前的记录就都没有了，以前是记录进derby

### Linux版Nacos+MySQL生产环境配置

预计需要，1个nginx+3个nacos注册中心+1个mysql

![260](\images\posts\springcloud\260.jpg)

#### Nacos下载linux版本

https://github.com/alibaba/nacos/releases/tag/1.1.4

![261](\images\posts\springcloud\261.jpg)

解压后安装。将解压后的文件复制一份到mynacos中，之后操作就操作mynacos中的文件

#### 集群配置步骤（重点）

##### Linux服务器上mysql数据库配置

新建nacos_config数据库，将mynacos/nacos/conf目录下的nacos-mysql.sql文件的内容粘贴执行

![262](\images\posts\springcloud\262.jpg)

执行后结果

![263](\images\posts\springcloud\263.jpg)

##### application.properties配置

先将原先文件复制一份备份

![264](\images\posts\springcloud\264.jpg)

将如下内容粘贴到application.properties中，并将用户名、密码等改成自己机器的

```properties
spring.datasource.platform=mysql
 
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=root
```

##### Linux服务器上nacos的集群配置cluster.conf

梳理出3台nacos机器的不同服务端口号

复制出cluster.conf：

![265](\images\posts\springcloud\265.jpg)

将cluster.conf内容修改为：

![266](\images\posts\springcloud\266.jpg)

注意：这个IP不能写127.0.0.1,必须是Linux命令hostname -i能够识别的IP

![267](\images\posts\springcloud\267.jpg)

##### 编辑Nacos的启动脚本startup.sh，使它能够接受不同的启动端

/mynacos/nacos/bin目录下有startup.sh

![268](\images\posts\springcloud\268.jpg)

修改内容：

![269](\images\posts\springcloud\269.jpg)

![270](\images\posts\springcloud\270.jpg)

![271](\images\posts\springcloud\271.jpg)

执行方式：

![272](\images\posts\springcloud\272.jpg)

##### Nginx的配置，由它作为负载均衡器

修改nginx的配置文件：

![273](\images\posts\springcloud\273.jpg)

nginx.conf：

```
upstream cluster{                                                        
 
    server 127.0.0.1:3333;
    server 127.0.0.1:4444;
    server 127.0.0.1:5555;
}


 server{                     
    listen 1111;
    server_name localhost;
    location /{
         proxy_pass http://cluster;                                               
    }
....省略
```

按照指定启动：

![274](\images\posts\springcloud\274.jpg)

##### 截止到此处，1个Nginx+3个nacos注册中心+1个mysql

测试通过nginx访问nacos：https://写你自己虚拟机的ip:1111/nacos/#/login

新建一个配置测试：

![275](\images\posts\springcloud\275.jpg)

此时，linux服务器的mysql会插入一条记录

![276](\images\posts\springcloud\276.jpg)

#### 测试

微服务cloudalibaba-provider-payment9002启动注册进nacos集群

##### 修改yaml

server-addr:  写你自己的虚拟机ip:1111

![277](\images\posts\springcloud\277.jpg)

##### 结果：会成功注册到nacos

![278](\images\posts\springcloud\278.jpg)

#### 高可用小总结

![279](\images\posts\springcloud\279.jpg)

