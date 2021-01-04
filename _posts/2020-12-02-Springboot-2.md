---
layout: post
title: Springboot笔记（二） Springboot入门程序
date: 2020-12-02
tags: Springboot
---

# 02、SpringBoot2入门

## 1、系统要求

- [Java 8](https://www.java.com/) & 兼容java14 .
- Maven 3.3+
- idea 
- springboot 2.3.4



### 1.1、maven设置

maven安装目录下\config\settings.xml中增加如下内容

```xml
<!--添加阿里云的镜像地址-->
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
</mirrors>

<!--使用jdk.1.8编译-->
<profiles>
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
</profiles>
```

### 1.2、IDEA设置

指定maven环境

![2.8](\images\posts\springboot\2.8.jpg)

![2.9](\images\posts\springboot\2.9.jpg)

## 2、HelloWorld

需求：浏览发送/hello请求，响应 Hello，Spring Boot 2 

### 2.1、创建maven工程



### 2.2、引入依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>


<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

</dependencies>
```

### 2.3、创建主程序

```java
/**
 * 主程序类
 * @SpringBootApplication：表明这是一个SpringBoot应用
 */
@SpringBootApplication
public class MainApplication {

    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class,args);
    }
}
```

注意主程序创建的位置（原因后面会讲）

![3.2](\images\posts\springboot\3.2.jpg)

### 2.4、编写业务

```java
//@RestController注解功能 = @ResponseBody + @Controller
@RestController
public class HelloController {
    
    @RequestMapping("/hello")
    public String handle01(){
        return "Hello, Spring Boot 2!";
    }

}
```

### 2.5、测试

直接运行main方法

### 2.6、简化配置

可以在application.properties配置文件中修改配置

```properties
server.port=8888
```

### 2.7、简化部署

我们可以将以上应用打成一个jar包，然后使用java  -jar 的命令直接运行

添加依赖：

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

添加上述依赖后，可以把项目打成jar包

![3.3](\images\posts\springboot\3.3.jpg)

打完的jar包默认在项目下的target文件夹下

直接在目标服务器使用 	`java -jar jar包名称`	执行即可。

注意点：

- 取消掉cmd的快速编辑模式

如果选中了快速编辑模式，在启动时如果在命令行窗口点击鼠标程序则会停止住，这时需要按下Ctrl+C，程序才会继续执行。或者直接取消掉cmd的快速编辑模式也可以。

![3.4](\images\posts\springboot\3.4.jpg)