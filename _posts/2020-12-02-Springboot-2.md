---
layout: post
title: Springboot笔记（二） Springboot配置文件
date: 2020-12-02
tags: Springboot
---

## 配置文件

### 简介

SpringBoot使用一个全局的配置文件，配置文件名`application`是固定的；

- application.properties
- application.yml
- application.yaml

配置文件的作用：修改SpringBoot自动配置的默认值；SpringBoot在底层都给我们自动配置好；

### YAML简介

YAML（YAML Ain't Markup Language）：**以数据为中心**，比properties、xml等更适合做配置文件

+ yml和xml相比，少了一些结构化的代码，使数据更直接，一目了然。
+ 相比properties文件更简洁

示例：

```yaml
#YAML
server:
	port: 8081

#xml：
<server>
	<port>8081</port>
</server>
```

### YAML语法

+ 以`空格`的缩进来控制层级关系；空格的个数并不重要，只要是左对齐的一列数据，都是同一个层级的

+ 次等级的前面是空格，不能使用制表符(tab)
+ 冒号之后如果有值，那么冒号和值之间至少有一个空格，不能紧贴着
+ 属性和值也是大小写敏感； 

```yaml
server:
    port: 8081
    path: /hello
```

#### 字面量：普通的值（数字，字符串，布尔）

K: V	字面直接来写； 

字符串默认不用加上单引号或者双引号；

""：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思。如：

```yaml
name: "zhangsan \n lisi"

输出；zhangsan 换行 lisi
```

''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据。如：

```yaml
name: ‘zhangsan \n lisi’

输出；zhangsan \n lisi 
```

#### 对象（属性和值）、Map（键值对）

k: v	在下一行来写对象的属性和值的关系；注意缩进

```yaml
friends:
	lastName: zhangsan
	age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```

#### 数组（List、Set）

用- 值表示数组中的一个元素 

```yaml
fruits: 
  - 苹果
  - 桃子
  - 香蕉
```

行内写法:

```yaml
fruits: [苹果,桃子,香蕉]
```

### 配置文件值注入

#### 属性绑定注入

在属性绑定的方式里，我们是通过set方法来完成的。要必须要提供set方法

##### 实体类（Javabean）

```java
public class Pet {
    private String name;
    private Integer age;
    //生成set/get
}


/**
* 将配置文件中配置的每一个属性的值，映射到这个组件中
*@ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
*prefix = "person"：前缀。配置文件中哪个下面的所有属性进行一一映射
*只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能，所以要加入@Component
**/
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Character gender;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Pet pet;
    //生成set/get
}
```

此时会出现这个提示

![13](\images\posts\springboot\13.jpg)

需要导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

###### 引入Lombok

上面实体类中的set/get方法可以通过引入Lombok来简化

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

```java
//引入Lombok后可以通过@Data注解来简化set/get书写

@Data
public class Pet {
    private String name;
    private Integer age;
}


@Data
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Character gender;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Pet pet;
}
```

##### 配置文件

此处以yml格式为例，properties格式示例见后。注意：如果一个项目中同时存在application.properties和application.yml两种格式的配置文件，则优先会选择properties格式配置文件

```yaml
#yml格式：
person:
  name: 张三
  gender: 男
  age: 36
  boss: true
  birth: 1982/10/1
  maps: {k1: v1,k2: v2}
  lists:
    - apple
    - peach
    - banana
  pet:
    name: 小狗
    age: 12
```

##### 测试

```java
import com.mg.domain.Person;
import org.junit.jupiter.api.Test;
//import org.junit.Test;
//import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
//import org.springframework.test.context.junit4.SpringRunner;

//@RunWith(SpringRunner.class)
@SpringBootTest
public class TestMain {
    @Autowired
    private Person person;

    /**
    此处的@Test要注意引入的包，如果引入的是org.junit.jupiter.api.Test则不需要@RunWith注解,
    如果引入的是org.junit.Test，则必须加入@RunWith注解否则注入失败，为null
    **/
    @Test
    public void test1() {
        System.out.println(person);
    }
}
```

使用``@SpringBootTest``注解，需要在pom中引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

##### 使用Properties配置文件方式

```properties
#properties格式
person.name=李四
person.age=34
person.birth=1986/09/12
person.boss=true
person.gender=女
person.lists=cat,dog,pig
person.maps.k1=v1
person.maps.k2=v2
person.pet.name="小黑"
person.pet.age=10
```

当使用properties格式配置文件测试时，发现中文会乱码，而且char类型还会抛出Failed to bind properties under 'person.gender' to java.lang.Character异常

##### 中文乱码解决办法

IDEA中 `Settings`--->  `File Encodings`，将配置文件字符集改为`UTF-8`，并勾选：

![14](\images\posts\springboot\14.jpg)

#### 构造器绑定注入

通过构造方法绑定。springboot2.2.X之后才能支持

##### 实体类

```java
public class Pet {
    private String name;
    private Integer age;
	//生成set/get，不是必须的操作
    
    //提供构造方法
    public Pet(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}

//在@ConfigurationProperties 注解的基础上再添加一个@ConstructorBinding。注意不要添加@Component,因为次注解不支持像 `@Component`、`@Bean`、`@Import` 等方式创建 bean 的构造器参数绑定
@ConstructorBinding
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Character gender;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Pet pet;
//提供构造方法
    public Person(String name, Character gender, Integer age, boolean boss, Date birth, Map<String, Object> maps, List<Object> lists, Pet pet) {
        this.name = name;
        this.gender = gender;
        this.age = age;
        this.boss = boss;
        this.birth = birth;
        this.maps = maps;
        this.lists = lists;
        this.pet = pet;
    }
}
```

##### 配置文件

```yaml
person:
  name: 张三
  gender: 男
  age: 36
  boss: true
  birth: 1982/10/1
  maps: {k1: v1,k2: v2}
  lists:
    - apple
    - peach
    - banana
  pet:
    name: 小狗
    age: 12
```

##### 测试

```java
import com.mg.domain.Person;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
@EnableConfigurationProperties(Person.class)
public class TestMain {
    @Autowired
    private Person person;

    @Test
    public void test1() {
        System.out.println(person);
    }
}

细节：
@EnableConfigurationProperties(Person.class):
如果一个配置类只配置@ConfigurationProperties注解，而没有使用@Component，那么在IOC容器中是获取不到properties或yml配置文件转化的bean。说白了 @EnableConfigurationProperties 相当于把使用 @ConfigurationProperties 的类进行了启用注入
```

##### @ConstructorBinding总结

1. 用了 `@ConstructorBinding` 这个注解，就标识这个类的参数优先通过带参数的构造器注入，如果没有带参数的构造器则再通过  setters  注入；
2. 当 `@ConstructorBinding` 用在类上时，该类只能有一个带参数的构造器；如果有多个构造器时，可以把 `@ConstructorBinding` 直接绑定到具体的构造方法上；如

```java
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Character gender;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Pet pet;
    //生成set/get

    //存在构造方法的重载时，需要将@ConstructorBinding绑定到方法上
    @ConstructorBinding
    public Person(String name, Character gender, Integer age, boolean boss, Date birth, Map<String, Object> maps, List<Object> lists, Pet pet) {
        this.name = name;
        this.gender = gender;
        this.age = age;
        this.boss = boss;
        this.birth = birth;
        this.maps = maps;
        this.lists = lists;
        this.pet = pet;
    }

    public Person(String name, Character gender, Integer age, boolean boss) {
        this.name = name;
        this.gender = gender;
        this.age = age;
        this.boss = boss;
    }
}
```

3. 不支持像 `@Component`、`@Bean`、`@Import` 等方式创建 bean 的构造器参数绑定
4. 成员变量可以是 `final` 不可变；
5. 支持该类的内部类构造器注入的形式；
6. 支持默认值 `@DefaultValue`、`@DateTimeFormat` 时间格式等注解配合使用；
7. 需要配合 `@ConfigurationProperties`、`@EnableConfigurationProperties` 注解使用；

### @Value获取值和@ConfigurationProperties获取值比较 

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

配置文件yml还是properties他们都能获取到值；
如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；
如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties； 

#### 松散绑定

Spring Boot使用一些宽松的规则将环境属性绑定到@ConfigurationProperties bean，因此环境属性名和bean属性名之间不需要完全匹配。

如Person中有`lastName`属性，在配置文件中可以写成`lastName`或`lastname`或`last-name`或`last_name`等等

![15](\images\posts\springboot\15.jpg)

松散绑定时可以使用以下这些格式：

| 属性文件中的配置 | 说明                                     |
| ---------------- | ---------------------------------------- |
| person.last-name | 羊肉串模式case, 推荐使用                 |
| person.lastName  | 标准驼峰模式                             |
| person.last_name | 下划线模式                               |
| PERSON_FIRSTNAME | 大写下划线，如果使用系统环境时候推荐使用 |

#### SPEL

```yaml
##　properties配置文件
person.age=#{2019-1986+1}
```

```java
//Person类
//使用@ConfigurationProperties注解，不支持，会抛出异常
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private Integer age;


//使用@value注解 OK
@Component
public class Person {
    @Value("${person.age}")
    private Integer age;
```

#### JSR303数据校验

需要确保类路径上有一个兼容的JSR-303实现，此处我们用的是hibernate的实现

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.0.Final</version>
</dependency>
```

`@ConfigurationProperties`支持校验，如果校验不通过，会抛出异常

![15](\images\posts\springboot\16.jpg)

`@value`注解不支持数据校验

#### 复杂类型封装

```java
//这样程序运行是直接报错的。@Value不支持复杂类型封装
@Value("${person.pet}")
private Pet pet;
```

### @PropertySource&@ImportResource&@Configuration

#### @PropertySource

`@PropertySource`注解的作用是加载指定的配置文件，值可以是数组，也就是可以加载多个配置文件

springboot默认会从全局配置文件（application.properties或application.yml）中获取值，如果想加载其他配置文件的内容，则需要此注解。

如 将person中相关的属性抽取为person.properties，使用`@PropertySource({"classpath:person.properties"})`指定加载`person.properties`配置文件

![17](\images\posts\springboot\17.jpg)

注意：

1. `@PropertySource`注解不支持yaml，只能是properties
2. 使用`@PropertySource`注解加载其他配置文件时，要保证全局配置文件（application.properties或application.yml）中没有相同配置，如果有相同配置还是会被全局配置文件覆盖

#### @ImportResource

`@ImportResource`注解用于导入Spring的配置文件，让配置文件里面的内容生效；(就是以前写的springmvc.xml、applicationContext.xml)

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；比如编写了一个Spring的配置文件bean.xml，此时直接使用是不能被识别的

![18](\images\posts\springboot\18.jpg)

此时需要在**主入口函数的类上**加入`@ImportResource`注解

```java
@ImportResource("classpath:beans.xml")
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

注意：`@ImportResource`这个注解是放在主入口函数的类上，而不是测试类上

#### @Configuration

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式

配置类**@Configuration** 相当于Spring配置文件

##### @Bean

使用**@Bean**给容器中添加组件

```java
@Configuration
public class BeanConfiguration {
    //相当于在配置文件中用<bean><bean/>标签添加组件
    @Bean
    public Pet createPet() {
        Pet pet = new Pet();
        pet.setName("小狗dsada");
        pet.setAge(12);
        return pet;
    }
}
```

此时就可以直接使用了

```java
@SpringBootTest
public class TestMain {
    @Autowired
    private Pet pet;

    @Test
    public void test1() {
        System.out.println(pet.getName()+"---"+pet.getAge());//输出	小狗dsada---12
    }
}
```

从SpringBoot2.2.1.RELEASE版本开始我们不再需要添加@Configuration，我们可以在扫描范围的bean的内部之间定义bean。如：

```java
@Component//需要让容器扫描到
public class BeanConfiguration {

    @Bean
    public Pet createPet() {
        Pet pet = new Pet();
        pet.setName("小狗dsada");
        pet.setAge(12);
        return pet;
    }
}

//或者这种情况都是可以的：
@RestController
public class YamlController {
    @Bean
    public Dep getDep(){
        return new Dep();
    }
}

```

### 配置文件占位符

#### 随机数

```properties
${random.value}
${random.int}
${random.long}
${random.int(10)}
${random.int[1024,65536]}
```

![19](\images\posts\springboot\19.jpg)

#### 占位符获取之前配置的值，如果没有可以是用:指定默认值 

```yml
person:
  last-name: 张三
  gender: 男
  age: ${random.int(100)}
  boss: true
  birth: 1982/10/1
  maps: {k1: v1,k2: v2}
  lists:
    - apple
    - peach
    - banana
  pet:
    name: ${person.last-name:小明}的小狗
    age: 12
    
此时如果person.last-name有值，如值为张三，则会输出：张三的小狗；如果没有值则会输出：小明的小狗
```

### Profile

Profile是Spring对不同环境提供不同配置功能的支持，可以通过激活、指定参数等方式快速切换环境

#### 多profile文件形式

文件名格式：application-{profile}.properties/yml，例如：

- application-dev.properties
- application-prod.properties

程序启动时会默认加载`application.properties`，启动的端口就是8080

可以在主配置文件中指定激活哪个配置文件

```yml
spring:
  profiles:
    active: dev
```

#### yml支持多文档块方式

每个文档块使用`---`分割

```yml
server:
  port: 8080
spring:
  profiles:
    active: prod
---
server:
  port: 8081
spring:
  profiles: dev
---
server:
  port: 8082
spring:
  profiles: prod
```

#### 激活指定profile的三种方式

1. 在配置文件中指定 spring.profiles.active=dev（如上）
2. 项目打包后在命令行启动

```
java -jar xxx.jar --spring.profiles.active=dev；
```

3. 虚拟机参数

```
-Dspring.profiles.active=dev
```

![20](\images\posts\springboot\20.jpg)

![21](\images\posts\springboot\21.jpg)

### 配置文件加载位置 

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件:

–file：./config/
–file：./
–classpath：/config/
–classpath：/

以上位置，优先级由高到底，高优先级的配置会覆盖低优先级的配置（优先级低的先加载）；

SpringBoot会从这四个位置全部加载主配置文件；互补配置；

![22](\images\posts\springboot\22.jpg)

注意：这里项目根路径下的配置文件maven编译时不会打包过去，需要修改pom

```xml
<resources>
    <resource>
        <directory>.</directory>
        <filtering>true</filtering>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.yaml</include>
        </includes>
    </resource>
</resources>
```

我们还可以通过spring.config.location来改变默认的配置文件位置

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

```
java -jar XXX.jar --spring.config.location=D:/application.properties
```

### 外部配置加载顺序 

**SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置**

1. 命令行参数所有的配置都可以在命令行上进行指定

```
java -jar XXX.jar --server.port=8087 --server.context-path=/abc

多个配置用空格分开； --配置项=值
```

2. 来自java:comp/env的JNDI属性

3. Java系统属性（System.getProperties()）

4. 操作系统环境变量

5. RandomValuePropertySource配置的random.*属性值

   **由jar包外向jar包内进行寻找**

   **优先加载带profile**

6. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件

7. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件

   **再来加载不带profile**

8. jar包外部的application.properties或application.yml(不带spring.profile)配置文件

9. jar包内部的application.properties或application.yml(不带spring.profile)配置文件

10. @Configuration注解类上的@PropertySource

11. 通过SpringApplication.setDefaultProperties指定的默认属性

    还有很多，具体可以参考官方文档：

    https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config