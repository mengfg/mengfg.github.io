---
layout: post
title: SpringCloud笔记（二） 微服务架构编码构建
date: 2020-12-27
tags: SpringCloud
---

## 微服务架构编码构建

现在的代码编写原则：约定 > 配置 > 编码

目前先构建一个	订单支付模块的项目，然后慢慢融合进上一讲中所介绍的技术

### 微服务cloud整体聚合父工程Project

#### 新建项目

这里也可以选择使用Spring Initializr方式创建。此处以maven为例

![5](\images\posts\springcloud\5.jpg)

#### 填写项目信息

![6](\images\posts\springcloud\6.jpg)

#### 设置Maven版本

![7](\images\posts\springcloud\7.jpg)

#### 设置工程名称

![8](\images\posts\springcloud\8.jpg)

项目创建好后，可以将src文件夹直接删除，因为我们只是将项目作为一个聚合父工程，并不需要编码工作

#### 设置字符编码

![9](\images\posts\springcloud\9.jpg)

#### 注解生效激活

![10](\images\posts\springcloud\10.jpg)

#### java编译版本选择

![11](\images\posts\springcloud\11.jpg)

#### File Type过滤

这步可以选做，配置之后，像.idea文件夹，XXX.iml文件就不会显示了

![12](\images\posts\springcloud\12.jpg)

#### pom文件

直接复制以下文件粘贴到IDEA中时，可能会出现空格报错的问题：

expected START_TAG or END_TAG not TEXT (position: TEXT seen ...

可以先粘贴至Word文档，将空格替换，然后再粘贴至IDEA

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.mg.springcloud</groupId>
    <artifactId>cloud2020</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--修改为pom-->
    <packaging>pom</packaging>

    <!--统一管理jar包版本-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <mysql.version>5.1.47</mysql.version>
        <druid.version>1.1.16</druid.version>
        <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
    </properties>

    <!--子模块继承之后，提供作用：锁定版本+子modlue不用写groupId和version-->
    <dependencyManagement>
        <dependencies>
            <!--springboot2.2.2-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.2.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springcloudHoxton.SR1-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Hoxton.SR1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--springcloudalibaba2.1.0.RELEASE-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2.1.0.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>druid</artifactId>
                <version>${druid.version}</version>
            </dependency>
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>${mybatis.spring.boot.version}</version>
            </dependency>
            <dependency>
                <groupId>junit</groupId>
                <artifactId>junit</artifactId>
                <version>${junit.version}</version>
            </dependency>
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <optional>true</optional>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

#### 知识点回顾

##### 1. Maven中的dependencyManagement和dependencies

![13](\images\posts\springcloud\13.jpg)

例如在父项目中：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.2</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

然后在子项目中添加mysql-connector时可以不指定版本号，例如：

```xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```

![14](\images\posts\springcloud\14.jpg)

##### 2. Maven中跳过单元测试

![15](\images\posts\springcloud\15.jpg)

### Rest微服务工程构建-支付模块

#### 创建module

新建支付模块cloud-provider-payment8001

![16](\images\posts\springcloud\16.jpg)

![17](\images\posts\springcloud\17.jpg)

![18](\images\posts\springcloud\18.jpg)

![19](\images\posts\springcloud\19.jpg)

创建完成后请回到父工程查看pom文件变化，会新增module

![20](\images\posts\springcloud\20.jpg)

#### 改pom文件

支付模块的pom文件中引入依赖

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

    <artifactId>cloud-provider-payment8001</artifactId>

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

    </dependencies>

</project>

```

#### 写yaml

在类路径下新建application.yml

```yaml
server:
  port: 8001


spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource  #当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver  #mysql驱动包
    url: jdbc:mysql://localhost:3306/db2019?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.mg.springcloud.entities  #所有Entity别名类所在包
```

#### 写主启动类

在java文件夹下新建com.mg.springcloud包，并新建PaymentMain8001启动类

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

#### 业务类

##### 建表SQL

```sql
CREATE TABLE payment(
  id bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
  serial varchar(200) DEFAULT '',
  PRIMARY KEY (id)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

##### entities

主实体Payment，注意要放在com.mg.springcloud.entities包中，因为前面配置文件中配置了生成别名的类包

```java
package com.mg.springcloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

Json封装体CommonResult：作为返回给前端的统一数据形式，用于规范返回给前端的数据

```java
package com.mg.springcloud.entities;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T>{

    private Integer code;
    private String message;
    private T data;

    public CommonResult(Integer code,String message){
        this(code,message,null);
    }
}
```

##### dao

新建dao包

接口：

```java
package com.mg.springcloud.dao;

import com.mg.springcloud.entities.Payment;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

@Mapper
public interface PaymentDao {

    public int create(Payment payment);
    public Payment getPaymentById(@Param("id") Long id);
}
```

mybatis的映射文件PaymentMapper.xml：

在src\main\resources\mapper文件夹下新建PaymentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.mg.springcloud.dao.PaymentDao">

    <insert id="create" parameterType="Payment" useGeneratedKeys="true" keyProperty="id">
        insert  into payment(serial) values (#{serial})
    </insert>


    <resultMap id="BaseResultMap" type="com.mg.springcloud.entities.Payment">
        <id property="id" column="id" jdbcType="VARCHAR"></id>
        <result property="serial" column="serial"></result>
    </resultMap>

    <select id="getPaymentById" parameterType="long" resultMap="BaseResultMap">
        select * from payment where id=#{id}
    </select>

</mapper>
```

##### service

新建service包

接口PaymentService:

```java
package com.mg.springcloud.service;

import com.mg.springcloud.entities.Payment;
import org.apache.ibatis.annotations.Param;

public interface PaymentService {

    public int create(Payment payment); //写

    public Payment getPaymentById(@Param("id") Long id);  //读取
}
```

实现类:

```java
package com.mg.springcloud.service.impl;

import com.mg.springcloud.dao.PaymentDao;
import com.mg.springcloud.entities.Payment;
import com.mg.springcloud.service.PaymentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class PaymentServiceImpl implements PaymentService {

    //也可以使用次注解，他是javax.annotation.Resource提供的.和Autowired作用一致
    //@Resource	
    @Autowired
    private PaymentDao paymentDao;

    public int create(Payment payment){
        return paymentDao.create(payment);
    }

    public Payment getPaymentById( Long id){
        return paymentDao.getPaymentById(id);
    }
}
```

##### controller

```java
package com.mg.springcloud.controller;

import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import com.mg.springcloud.service.PaymentService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class PaymentController {

    @Autowired
    private PaymentService paymentService;

    @PostMapping(value = "/payment/create")
    public CommonResult create(Payment payment){
        int result = paymentService.create(payment);
        log.info("*****插入结果："+result);
        if (result>0){  //成功
            return new CommonResult(200,"插入数据库成功",result);
        }else {
            return new CommonResult(444,"插入数据库失败",null);
        }
    }
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("*****查询结果："+payment);
        if (payment!=null){  //说明有数据，能查询成功
            return new CommonResult(200,"查询成功",payment);
        }else {
            return new CommonResult(444,"没有对应记录，查询ID："+id,null);
        }
    }

}

```

#### 测试

先将目前搭建的代码跑通。

浏览器发送get请求测试查询方法：http://localhost:8001/payment/get/1

postman模拟post请求测试插入方法：http://localhost:8001/payment/create?serial=测试数据002



### 热部署Devtools

开发阶段的可以选择开启，但是正式的生产环境要禁用掉

#### 支付模块cloud-provider-payment8001的pom.xml中添加：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

#### 父工程中添加

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### 开启自动编译选项

![21](\images\posts\springcloud\21.jpg)

#### 更新值

按住Ctrl+shift+Alt+/

![22](\images\posts\springcloud\22.jpg)

![23](\images\posts\springcloud\23.jpg)

#### 重启IDEA



### Rest微服务工程构建-消费者模块

#### 创建module

新建消费者模块cloud-consumer-order80	过程同上支付模块

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

    <artifactId>cloud-consumer-order80</artifactId>

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

#### 写YML

```yaml
server:
  port: 80
```

#### 主启动

```java
package com.mg.springcloud;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }

}
```

#### 业务类

此模块作为消费者，其实只是调用支付模块，所以只需要controller层和必要的实体类即可

##### entities

同上支付模块。

创建entities(将cloud-provider-payment8001工程下的entities包下的两个实体类复制过来)

##### 配置类

###### RestTemplate

RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集

###### 官网及使用

**官网地址：**
https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html

**使用：**

使用restTemplate访问restful接口非常的简单粗暴无脑。

(url, requestMap, ResponseBean.class)这三个参数分别代表REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

```java
package com.mg.springcloud.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextConfig {

    @Bean
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

}
```

##### controller

```java
package com.mg.springcloud.controller;

import com.mg.springcloud.entities.CommonResult;
import com.mg.springcloud.entities.Payment;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class OrderController {

    public static final String PAYMENT_URL = "http://localhost:8001";

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

将两个模块分别启动。

测试查询：http://localhost/consumer/payment/get/1

测试插入：http://localhost/consumer/payment/create?serial=测试数据4

测试插入时需要注意的是，支付模块controller插入方法中不要忘记@RequestBody注解

```java
@PostMapping(value = "/payment/create")
public CommonResult create(@RequestBody Payment payment){
    int result = paymentService.create(payment);
    log.info("*****插入结果："+result);
    if (result>0){  //成功
        return new CommonResult(200,"插入数据库成功",result);
    }else {
        return new CommonResult(444,"插入数据库失败",null);
    }
}
```

### 开启Run DashBoard

正常情况下，如果你有多个module，IDEA会自动检测到给你开启DashBoard。如果没有自动开启，开启使用以下方法开启

1. 找到自己的项目路径下的workspace.xml	例如我本地的路径是

   E:\myProject\cloud2020\\.idea\workspace.xml

   在RunDashboard标签下添加以下代码

   ```xml
   <option name="configurationTypes">
       <set>
           <option value="SpringBootApplicationConfigurationType" />
       </set>
   </option>
   ```

   ![24](\images\posts\springcloud\24.jpg)

然后就可以出现如下效果了：如果未出现可以考虑重启下IDEA

![25](\images\posts\springcloud\25.jpg)



### 项目重构

系统中有重复部分，比如两个模块中存在着相同的实体类，需要进行重构

#### 创建module

新建一个公共模块cloud-api-commons

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

    <artifactId>cloud-api-commons</artifactId>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
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

        <!-- https://mvnrepository.com/artifact/cn.hutool/hutool-all -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>
    </dependencies>

</project>
```

#### entities

将之前两个模块中的重复的entities放置在此模块中

#### install至Maven仓库中

将公共模块cloud-api-commons通过maven命令clean install，安装到Maven仓库中

#### 订单80和支付8001分别改造

1. 删除各自的原先有过的entities文件夹

2. 各自黏贴POM内容

   ```xml
   <dependency>
       <groupId>com.atguigu.springcloud</groupId>
       <artifactId>cloud-api-commons</artifactId>
       <version>${project.version}</version>
   </dependency>
   ```

重新测试，项目正常运行