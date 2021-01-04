---
layout: post
title: Springboot笔记（九） Springboot原理解析
date: 2020-12-16
tags: Springboot
---

# 09、原理解析

# 1、Profile功能

为了方便多环境适配，springboot简化了profile功能。

## 1、application-profile功能

- 默认配置文件  application.yaml；任何时候都会加载
- 指定环境配置文件  application-{env}.yaml，比如application-dev.yaml
- 激活指定环境

  - 配置文件激活

    ```yaml
    spring:
      profiles:
        active: dev	#只写application-{env}.yaml中{env}的内容即可
    ```

  - 命令行激活

    ```
    可以指定激活环境，并且还可以修改配置文件的值
    java -jar xxx.jar --spring.profiles.active=prod  --person.name=haha
    ```
    
    - 修改配置文件的任意值，命令行优先

- 默认配置与环境配置同时生效
- 如果默认配置与环境配置存在同名配置项，则环境配置（profile配置）优先



## 2、@Profile条件装配功能

比如有两个配置文件，application-test.yaml和application-prod.yaml

```java
===============可以配置在类上==============
//可以配置在类上。当激活的是test环境，则类生效
@Profile("test")
@Component
@ConfigurationProperties("person")
@Data
public class Worker implements Person {
    private String name;
    private Integer age;
}
//可以配置在类上。当激活的是prod环境，则类生效。default代表如果没有指定则默认加载这个
@Profile(value = {"prod","default"})
@Component
@ConfigurationProperties("person")
@Data
public class Boss implements Person {
    private String name;
    private Integer age;
}


===============可以配置在方法上==============
@Configuration
public class MyConfig {
//也可以配置在方法上。当激活的是prod环境，则方法生效
    @Profile("prod")
    @Bean
    public Color red(){
        return new Color();
    }
//也可以配置在方法上。当激活的是test环境，则方法生效
    @Profile("test")
    @Bean
    public Color green(){
        return new Color();
    }
}
```

## 3、profile分组

可以指定组，组中包含几个配置文件，启动组的时候，组中的几个配置文件都会生效

```properties
#分组
spring.profiles.group.myprod[0]=proddb
spring.profiles.group.myprod[1]=prodmq

#激活。当激活myprod组时，组中的proddb，prodmq文件中的配置都会生效
spring.profiles.active=myprod
```



# 2、外部化配置

https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config

## 1、外部配置源

常用：**Java属性文件**、**YAML文件**、**环境变量**、**命令行参数**；

```java
@SpringBootApplication
public class Boot09FeaturesProfileApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(Boot09FeaturesProfileApplication.class, args);
        ConfigurableEnvironment environment = run.getEnvironment();
        //系统的环境变量
        Map<String, Object> systemEnvironment = environment.getSystemEnvironment();
        //系统的属性
        Map<String, Object> systemProperties = environment.getSystemProperties();
        System.out.println(systemEnvironment);
        System.out.println(systemProperties);
    }
}


//系统的环境变量和系统的属性中的东西都可以获取：
//例如，可以获取环境变量的值
@RestController
public class HelloController {
    @Value("${MAVEN_HOME}")
    private String msg;

    //可以获取系统属性中的系统名称
    @Value("${os.name}")
    private String osName;
    
    @GetMapping("/msg")
    public String getMsg(){
        return msg+"==>"+osName;
    }
}
```



## 2、配置文件查找位置

application.yml可以存在于以下位置：优先级越来越高，如果多个位置同时存在，高优先级的配置会覆盖低优先级的

(1) classpath 根路径

(2) classpath 根路径下config目录

(3) jar包当前目录

(4) jar包当前目录的config目录

(5) /config子目录的直接子目录

## 3、配置文件加载顺序：

1. 　当前jar包内部的application.properties和application.yml
2. 　当前jar包内部的application-{profile}.properties 和 application-{profile}.yml
3. 　引用的外部jar包的application.properties和application.yml
4. 　引用的外部jar包的application-{profile}.properties 和 application-{profile}.yml

## 4、总结：

指定环境优先，外部优先（像命令行都属于项目的外部），后面优先级高的可以覆盖前面优先级低的同名配置项



# 3、自定义starter

## 1、starter启动原理

- starter-pom引入 autoconfigurer 包

![11.5](\images\posts\springboot\11.5.jpg)

- autoconfigure包中配置使用 `META-INF/spring.factories` 中 EnableAutoConfiguration 的值，使得项目启动加载指定的自动配置类
- 编写自动配置类 xxxAutoConfiguration 绑定 xxxxProperties
  - **@Configuration**
  - **@Conditional**
  - **@EnableConfigurationProperties**
  - **@Bean**
  - ......


**引入starter** **--- xxxAutoConfiguration --- 容器中放入组件 ---- 绑定xxxProperties ----** **配置项**

## 2、自定义starter

场景：加入我们有一个功能组件是，你传入名字，我会输出打招呼的信息。我们的自定义starter就来抽取这个功能

1. 新建一个空工程，添加两个模块。`mg-hello-spring-boot-starter`，此模块作为启动器，这是说明需要引入的那些依赖。`mg-hello-spring-boot-starter-autoconfigure`此模块作为自动配置包实现真正的自动配置功能

2. `mg-hello-spring-boot-starter`模块中引入`mg-hello-spring-boot-starter-autoconfigure`

   ```xml
   <dependencies>
       <dependency>
           <groupId>com.mg</groupId>
           <artifactId>mg-hello-spring-boot-starter-autoconfigure</artifactId>
           <version>0.0.1-SNAPSHOT</version>
       </dependency>
   </dependencies>
   ```

3. `mg-hello-spring-boot-starter-autoconfigure`模块中（test包、主启动类、配置文件等都可以删除，因为我们只是一个自动配置的类，不需要启动所以用不到这些东西）

   pom文件：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.4.1</version>
           <relativePath/> <!-- lookup parent from repository -->
       </parent>
       <groupId>com.mg</groupId>
       <artifactId>mg-hello-spring-boot-starter-autoconfigure</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       <name>mg-hello-spring-boot-starter-autoconfigure</name>
       <description>Demo project for Spring Boot</description>
   
       <properties>
           <java.version>1.8</java.version>
       </properties>
   
       <!--只需要保留spring-boot-starter即可，其他无用-->
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter</artifactId>
           </dependency>
       </dependencies>
   </project>
   ```

   配置文件类：

   ```java
   @ConfigurationProperties(prefix = "mg.hello")
   public class HelloProperties {
       private String prefix;
       private String suffix;
       //生成set/get
   }
   ```

   Service接口：

   ```java
   public class HelloService {
   
       @Autowired
       private HelloProperties helloProperties;
   
       public String sayHello(String userName) {
           return helloProperties.getPrefix() + ","+userName + helloProperties.getSuffix();
       }
   }
   ```

   自动配置：

   ```java
   @Configuration
   //当容器中没有HelloService组件时才生效
   @ConditionalOnMissingBean(HelloService.class)
   //绑定配置文件，并且HelloProperties会自动放入容器中
   @EnableConfigurationProperties(HelloProperties.class)
   public class HelloServiceAutoConfiguration {
       @Bean
       public HelloService helloService() {
           HelloService helloService = new HelloService();
           return helloService;
       }
   }
   ```

   EnableAutoConfiguration 的值，使得项目启动加载指定的自动配置类：

   在类路径下创建 `META-INF/spring.factories`

   ```properties
   # Auto Configure
   org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
   com.mg.hello.auto.HelloServiceAutoConfiguration
   ```

4. 创建项目进行测试

   要引入自定义的启动器

   ```xml
   <dependency>
       <groupId>com.mg</groupId>
       <artifactId>mg-hello-spring-boot-starter</artifactId>
       <version>1.0-SNAPSHOT</version>
   </dependency>
   ```

   配置文件可以自定义HelloProperties中的前缀和后缀

   ```yaml
   mg:
     hello:
       prefix: 你好
       suffix: 同学
   ```

   测试

   ```java
   @RestController
   public class HelloController {
   
       @Autowired
       private HelloService helloService;
   
       @GetMapping("/hello")
       public String hello() {
           return helloService.sayHello("小明");
       }
   }
   
   访问http://localhost:8080/hello会显示	你好,小明同学
   ```

   

# 4、SpringBoot原理

Spring原理【[Spring注解](https://www.bilibili.com/video/BV1gW411W7wy?p=1)】、**SpringMVC**原理、**自动配置原理**、SpringBoot原理

## 1、SpringBoot启动过程

- 创建 **SpringApplication**

  - 保存一些信息。

  - 判定当前应用的类型。（是使用ClassUtils这个工具类判定。我们的应用一般都是一个Servlet）

  - **bootstrappers**：初始启动引导器（List<Bootstrapper>）：去spring.factories文件中找`org.springframework.boot.Bootstrapper`类型的

  - 找 **ApplicationContextInitializer**；去**spring.factories**找 **ApplicationContextInitializer**

    - List<ApplicationContextInitializer<?>> initializers

      ![11.7](\images\posts\springboot\11.7.jpg)

  - **找** **ApplicationListener  ；应用监听器。**去**spring.factories****找** **ApplicationListener**
    
    - List<ApplicationListener<?>> **listeners**

- 运行 **SpringApplication**

- - **StopWatch**
  - **记录应用的启动时间**
  - 创建引导上下文（Context环境）`createBootstrapContext()`

- - - 获取到所有之前的 **bootstrappers 挨个执行** intitialize() 来完成对引导启动器上下文环境设置

      ```java
      //Bootstrapper类
      public interface Bootstrapper {
      
          /**
           * Initialize the given {@link BootstrapRegistry} with any required registrations.
           * @param registry the registry to initialize
           */
          void intitialize(BootstrapRegistry registry);
      }
      ```

- - 让当前应用进入**headless**模式。**java.awt.headless**
  - **获取所有** **RunListener**（运行监听器）【为了方便所有Listener进行事件感知】

- - - getSpringFactoriesInstances 去**spring.factories**找 **SpringApplicationRunListener**

      ![11.6](\images\posts\springboot\11.6.jpg)

- - 遍历 **SpringApplicationRunListener 调用 starting 方法；**

- - - **相当于通知所有感兴趣系统正在启动过程的人，项目正在 starting。**

- - 保存命令行参数；ApplicationArguments
  - 准备环境 prepareEnvironment（）;

- - - 返回或者创建基础环境信息对象。**StandardServletEnvironment**
    - **配置环境信息对象。**

- - - - **读取所有的配置源的配置属性值。**

- - - 绑定环境信息
    - 监听器调用 listener.environmentPrepared()；通知所有的监听器当前环境准备完成

- - 创建IOC容器（createApplicationContext（））

- - - 根据项目类型（Servlet）创建容器，
    - 当前会创建 **AnnotationConfigServletWebServerApplicationContext**

- - **准备ApplicationContext IOC容器的基本信息**   **prepareContext()**

- - - 保存环境信息
    - IOC容器的后置处理流程。
    - 应用初始化器；applyInitializers；

- - - - 遍历所有的 **ApplicationContextInitializer 。调用** **initialize.。来对ioc容器进行初始化扩展功能**

      - 遍历所有的 listener 调用 **contextPrepared。EventPublishRunListenr；通知所有的监听器contextPrepared**

        ![11.8](\images\posts\springboot\11.8.jpg)

- - - **所有的监听器 调用** **contextLoaded。通知所有的监听器 contextLoaded；**

- - **刷新IOC容器。**refreshContext

- - - 创建容器中的所有组件（Spring注解）

- - 容器刷新完成后工作：afterRefresh
  - 所有监听 器 调用 listeners.**started**(context); **通知所有的监听器** **started**
  - **调用所有runners；**callRunners()

- - - **获取容器中的** **ApplicationRunner** 

      ```java
      @FunctionalInterface
      public interface ApplicationRunner {
          /**
           * Callback used to run the bean.
           * @param args incoming application arguments
           * @throws Exception on error
           */
          void run(ApplicationArguments args) throws Exception;
      }
      ```

    - **获取容器中的**  **CommandLineRunner**

      ```java
      @FunctionalInterface
      public interface CommandLineRunner {
          /**
           * Callback used to run the bean.
           * @param args incoming main method arguments
           * @throws Exception on error
           */
          void run(String... args) throws Exception;
      }
      ```

    - **合并所有runner并且按照@Order进行排序**

    - **遍历所有的runner。调用 run** **方法**

- - **如果以上有异常，**

- - - **调用Listener 的 failed**

- - **调用所有监听器的 running 方法**  listeners.running(context); **通知所有的监听器** **running** 
  - **running如果有问题。继续通知 failed 。****调用所有 Listener 的** **failed；****通知所有的监听器** **failed**



## 2、Application Events and Listeners

https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners

**ApplicationContextInitializer**

**ApplicationListener**

**SpringApplicationRunListener**



## 3、ApplicationRunner 与 CommandLineRunner