---
layout: post
title: Springboot笔记（一） Springboot入门
date: 2020-12-01
tags: Springboot
---

## Springboot简介

### 什么是Springboot

是用来简化Spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。Spring Boot 是所有基于 Spring 开发的项目的起点。SpringBoot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，spring boot整合了很多的技术，提供了JavaEE的大整合。

核心思想：约定大于配置

### SpringBoot优点

- 快速创建独立运行的Spring项目以及与主流框架集成
- 使用嵌入式的Servlet容器，应用无需打成WAR包
- starters自动依赖与版本控制
- 大量的自动配置，简化开发，也可修改默认值
- 无需配置XML，无代码生成，开箱即用
- 准生产环境的运行时应用监控
- 与云计算的天然集成

### 微服务

2014，martin fowler

微服务：架构风格（服务微化）

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

单体应用：ALL IN ONE

微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

[详细参照微服务文档](https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa)

[集群、分布式、微服务概念和区别](https://blog.csdn.net/qq_37788067/article/details/79250623)

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
public class HelloWorldMainApplication {
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

运行主程序Main方法测试，启动成功后浏览器输入   http://localhost:8080/hello   会得到    helloworld

### 简化部署：将应用打成jar包

我们可以将以上应用打成一个jar包，然后使用java  -jar 的命令直接运行

#### 在pom.xml中引入构建工具

```xml
 <!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
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

进入打包好的jar包所在目录

使用  `java  -jar  jar包名称 ` 运行

## HelloWorld探究

### POM文件依赖

```xml
<!--Hello World项目的父工程是org.springframework.boot-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.0.RELEASE</version>
</parent>


<!--org.springframework.boot他的父项目是spring-boot-dependencies他来真正管理Spring Boot应用里面的所有依赖版本；Spring Boot的版本仲裁中心；以后我们导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖自然需要声明版本号）-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.0.RELEASE</version>
</parent>
```

### 启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring-boot-starter-***<u>web</u>***：

- spring-boot-starter：spring-boot场景启动器；帮我们导入了web模块正常运行所依赖的组件；
- Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

### 主程序类（主入口类）

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

//@SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

`@SpringBootApplication`: Spring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；

看一下`@SpringBootApplication`这个注解的源码

```java
@Target({ElementType.TYPE}) //可以给一个类型进行注解，比如类、接口、枚举
@Retention(RetentionPolicy.RUNTIME) //可以保留到程序运行的时候，它会被加载进入到 JVM 中
@Documented //将注解中的元素包含到 Javadoc 中去。
@Inherited //继承，比如A类上有该注解，B类继承A类，B类就也拥有该注解
@SpringBootConfiguration
@EnableAutoConfiguration
/*
*创建一个配置类，在配置类上添加 @ComponentScan 注解。
*该注解默认会扫描该类所在的包下所有的配置类，相当于之前的 <context:component-scan>。
*/
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {…………}
```

`@SpringBootApplication`中有两个重要的注解：`@SpringBootConfiguration`和`@EnableAutoConfiguration`

#### `@SpringBootConfiguration`：

Spring Boot的配置类；标注在某个类上，表示这是一个Spring Boot的配置类；

```java
//@SpringBootConfiguration注解源代码为：
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {…………}
```

##### @Configuration

配置类上来标注这个注解；配置类（用于替换配置文件），配置类也是容器中的一个组件所以也需要@Component标注

```java
//@Configuration注解源码：
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {…………}
```

#### `@EnableAutoConfiguration`

开启自动配置功能。以前我们需要配置的东西，Spring Boot帮我们自动配置；这个注解告诉SpringBoot开启自动配置功能；这样自动配置才能生效；

```java
//@EnableAutoConfiguration注解源代码为：
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {…………}
```

##### @AutoConfigurationPackage

自动配置包

```java
//@AutoConfigurationPackage源码
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {…………}
```

###### @Import({Registrar.class})

Spring的底层注解@Import，给容器中导入一个组件，导入的组件由org.springframework.boot.autoconfigure.AutoConfigurationPackages.Registrar将主配置类（@SpringBootApplication标注的类）所在包及下面所有子包里面的所有组件扫描到Spring容器

所以说springboot的主启动类所在的package就是扫描器默认扫描的basepackage

比如：当前项目的主配置类所在包为com.mg	此时controller包是在主程序所在的包下，所以会被扫描到

![5](\images\posts\springboot\5.jpg)

如果此时我们新建一个test包，将主程序放在test包下，这样启动就只会去扫描test包下的内容而controller包就不会被扫描到，再访问开始的hello就是404

![6](\images\posts\springboot\6.jpg)

##### @Import({AutoConfigurationImportSelector.class})

AutoConfigurationImportSelector.class将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；

有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

![4](\images\posts\springboot\4.jpg)

Spring Boot在启动的时候从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；以前我们需要自己配置的东西，自动配置类都帮我们完成了；

![7](\images\posts\springboot\7.jpg)

## 其他方式创建Springboot程序

### 使用Spring Initializer快速创建Spring Boot项目 

IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；

选择我们需要的模块；向导会联网创建Spring Boot项目；

前提：需要联网

#### 创建时选择Spring Initializr

![8](\images\posts\springboot\8.jpg)

#### 完善项目信息

![9](\images\posts\springboot\9.jpg)

#### 选择需要的依赖及Springboot的版本

![10](\images\posts\springboot\10.jpg)

#### 存放位置

![11](\images\posts\springboot\11.jpg)

#### 创建完成后 不要的文件可以删除

![12](\images\posts\springboot\12.jpg)

### 使用Spring Initializr 的 Web页面创建项目

#### 打开  https://start.spring.io/

#### 填写项目信息（和IDEA创建时的信息相同）

#### 点击”Generate“按钮生成项目；下载此项目

#### 解压项目包，并用IDEA以Maven项目导入，一路下一步即可，直到项目导入完毕。

注意：如果是第一次使用，可能速度会比较慢，包比较多、需要耐心等待一切就绪。

### 注意：

使用上述工具默认生成的Spring Boot项目中：

1. 主程序已经生成好了，我们只需要完成我们自己的逻辑

2. `resources`文件夹中目录结构

- static：保存所有的静态资源； js、css、images；
- templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）；
- application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；