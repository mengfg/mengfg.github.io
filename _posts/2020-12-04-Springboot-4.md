---
layout: post
title: Springboot笔记（四） Springboot配置文件
date: 2020-12-04
tags: Springboot
---

# 04、配置文件

SpringBoot使用一个全局的配置文件，配置文件名`application`是固定的；

可以写成以下几种拓展名：

- application.properties
- application.yml
- application.yaml

注意：如果一个项目中同时存在application.properties和application.yml两种格式的配置文件，他们会相互配合，但如果里面出现相同的配置，则优先会选择properties格式配置文件里的配置

# 1、文件类型

## 1.1、properties

同以前的properties用法

## 1.2、yaml

### 1.2.1、简介

YAML 是 "YAML Ain't Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。 



非常适合用来做以数据为中心的配置文件



### 1.2.2、基本语法

- key: value；kv之间有空格
- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格（IDEA开发时未发现该问题）
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释
- 字符串无需加引号，如果要加，''与""表示字符串内容 会被 转义/不转义



### 1.2.3、数据类型



- 字面量：单个的、不可再分的值。date、boolean、string、number、null

  k: v

  - 字符串默认不用加上单引号或者双引号；
  - ”“：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思。如：

  ```yaml
  name: "zhangsan \n lisi"
  
  输出；zhangsan 换行 lisi
  ```

  - ’‘：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据。如：

  ```yaml
  name: ‘zhangsan \n lisi’
  
  输出；zhangsan \n lisi 
  ```

  



- 对象：键值对的集合。map、hash、set、object 

```yaml
行内写法：  k: {k1:v1,k2:v2,k3:v3}
#或
k: 
  k1: v1
  k2: v2
  k3: v3
```

- 数组：一组按次序排列的值。array、list、queue

```yaml
行内写法：  k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3
```

### 1.2.4、示例

```java
Data
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String userName;
    private Boolean boss;
    private Date birth;
    private Integer age;
    private Pet pet;
    private String[] interests;
    private List<String> animal;
    private Map<String, Object> score;
    private Set<Double> salarys;
    private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
    private String name;
    private Double weight;
}
```



新建文件	application.yaml	或者	application.yml

```yaml
# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]
```

注意：Spring Boot支持松散绑定，因此环境属性名和bean属性名之间不需要完全匹配。

如Person中有`lastName`属性，在配置文件中可以写成`lastName`或`lastname`或`last-name`或`last_name`等等

# 2、配置提示

自定义的类和配置文件绑定一般没有提示。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>


<!--打包的时候排除掉配置处理器-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-configuration-processor</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
```