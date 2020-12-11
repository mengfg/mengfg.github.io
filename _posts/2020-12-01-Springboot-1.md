---
layout: post
title: Springboot笔记（一） Springboot入门
date: 2020-12-01
tags: Springboot
---

## Springboot简介

### 什么是Springboot

是用来简化Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。Spring Boot 是所有基于 Spring 开发的项目的起点。SpringBoot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，spring boot整合了很多的技术，提供了JavaEE的大整合。

### SpringBoot主要特性

spring官方的网站：https://spring.io/

1. SpringBoot Starter：他将常用的依赖分组进行了整合，将其合并到一个依赖中，这样就可以一次性添加到项目的Maven或Gradle构建中；
2. 使编码变得简单，SpringBoot采用 JavaConfig的方式对Spring进行配置，并且提供了大量的注解，极大的提高了工作效率。
3. 自动配置：SpringBoot的自动配置特性利用了Spring对条件化配置的支持，合理地推测应用所需的bean并自动化配置他们；
4.  使部署变得简单，SpringBoot内置了三种Servlet容器，Tomcat，Jetty,undertow.我们只需要一个Java的运行环境就可以跑SpringBoot的项目了，SpringBoot的项目可以打成一个jar包。
5. 现在流行微服务与分布式系统，springboot就是一个非常好的微服务开发框架，你可以使用它快速的搭建起一个系统。同时，你也可以使用spring cloud（Spring Cloud是一个基于Spring Boot实现的云应用开发工具）来搭建一个分布式的架构。

## Spring入门程序

### 环境准备

jdk1.8：Spring Boot 推荐jdk1.8及以上；

maven3.x：maven 3.3以上版本；

IntelliJIDEA2018

SpringBoot 2.3.0.RELEASE

#### Maven设置

1. **配置阿里云镜像**

```xml
<mirror>  
    <id>nexus-aliyun</id>  
    <mirrorOf>central</mirrorOf>    
    <name>Nexus aliyun</name>  
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>  
</mirror>
```

2. **给maven 的settings.xml配置文件的profiles标签添加**

```xml
<!--指定使用JDK1.8来编译项目-->
<profile>
    <id>jdk-1.8</id>
    <activation>
        <activeByDefault>true</activeByDefault>
        <jdk>1.8</jdk>
    </activation>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
</profile>
```

#### IDEA设置

指定maven环境

![3](\images\posts\springboot\1.jpg)

![4](\images\posts\springboot\2.jpg)

### 入门程序

#### 创建Maven工程（jar）

#### 导入Springboot的相关依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

#### 编写一个主程序，启动Springboot应用

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

//@SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
@SpringBootApplication
public class HelloApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }
}
```

#### 创建测试的Controller

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "hello world";
    }
}
```

运行主程序，启动成功后浏览器输入   http://localhost:8080/hello   会得到    helloworld

### 将应用打成jar包

我们可以将以上应用打成一个jar包，然后使用java  -jar 的命令直接运行

#### 在pom.xml中引入构建工具

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

#### 打jar包

在命令行使用mvn  package打包	或者	使用IDEA中Maven Project中使用package打包

![3](\images\posts\springboot\3.jpg)

#### 使用java   -jar命令直接运行jar包即可

打完的jar包默认在项目下的target文件夹下

