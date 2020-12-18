---
layout: post
title: Springboot笔记（五） Springboot中Web开发
date: 2020-12-05
tags: Springboot
---

### SpringBoot Web开发

1. 创建SpringBoot应用，选中我们需要的模块
2. SpringBoot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来
3. 自己编写业务代码

### Springboot对静态资源的映射规则

`WebMvcAutoConfiguration`类的`addResourceHandlers`方法：（添加资源映射）

![32](\images\posts\springboot\32.jpg)

可以通过ResourceProperties资源配置类设置和静态资源有关的参数，如缓存时间等 ：

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {…………}
```

#### webjars方式

通过源码发现，所有 `/webjars/**` ，都去 `classpath:/META-INF/resources/webjars/` 找资源

`webjars`：以jar包的方式引入静态资源；

webjars官网：https://www.webjars.org/ 

例如：添加jquery的webjars

![29](\images\posts\springboot\29.jpg)

需要引入相关依赖：

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

在项目中便会引入相关文件：

![30](\images\posts\springboot\30.jpg)

相应的访问地址为：http://localhost:8080/webjars/jquery/3.5.1/jquery.js

#### 非webjars，自己的静态资源怎么访问

`WebMvcAutoConfiguration`类的`addResourceHandlers`方法：（添加资源映射）

![32](\images\posts\springboot\32.jpg)

资源配置类ResourceProperties源码：

![31](\images\posts\springboot\31.jpg)

上图中添加的映射访问路径`staticPathPattern`值是`/**`，对应的资源文件夹就是配置类`ResourceProperties`中的`CLASSPATH_RESOURCE_LOCATIONS`数组中的文件夹：

"/**" 访问当前项目的任何资源，都去（静态资源的文件夹）找映射

| 数组中的值                     | 在项目中的位置                         |
| ------------------------------ | -------------------------------------- |
| classpath:/META-INF/resources/ | src/main/resources/META-INF/resources/ |
| classpath:/resources/          | **src/main/resources/resources/**      |
| classpath:/static/             | src/main/resources/static/             |
| classpath:/public/             | src/main/resources/public/             |

localhost:8080/abc ---> 去静态资源文件夹里面找abc

### 首页映射

![33](\images\posts\springboot\33.jpg)

`location`就是静态资源路径，所以首页就是上面静态资源下的`index.html`，被`/**`映射，因此直接访问项目就是访问欢迎页

### 网站图标映射（favicon.ico）

所有的 favicon.ico 都是在静态资源文件下找。直接将favicon.ico放在静态资源文件下即可。

![34](\images\posts\springboot\34.jpg)

