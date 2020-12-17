---
layout: post
title: Springboot笔记（三） Springboot自动配置
date: 2020-12-03
tags: Springboot
---

## 自动配置

配置文件到底能写什么？怎么写？自动配置原理；

配置文件能配置的属性请参照官方文档：

https://docs.spring.io/spring-boot/docs/2.3.0.RELEASE/reference/htmlsingle/#common-application-properties

### 自动配置原理

1. #### SpringBoot启动的时候加载主配置类，开启了自动配置功能

```java
@SpringBootApplication
    @EnableAutoConfiguration
```

2. #### @EnableAutoConfiguration 作用： 

   利用AutoConfigurationImportSelector给容器中导入一些组件：

   ```java
   @Import({AutoConfigurationImportSelector.class})
   ```

（1）可以查看selectImports()方法，会调用getAutoConfigurationEntry方法

```java
AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
```

（2）getAutoConfigurationEntry方法中，会调用getCandidateConfigurations方法 

```java
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
```

（3）getCandidateConfigurations方法中，会调用loadFactoryNames方法

```
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
```

（4）loadFactoryNames方法，会调用loadSpringFactories方法

```
return (List)loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
```

（5）loadSpringFactories方法

扫描所有jar包类路径下 META-INF/spring.factories，把扫描到的这些文件的内容包装成properties对象，因为loadFactoryNames会传递一个参数【this.getSpringFactoriesLoaderFactoryClass()】，参数的值为EnableAutoConfiguration.class，所以会从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中 

总结起来就是：

**将 类路径下 META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中** 

![23](\images\posts\springboot\23.jpg)

每一个这样的 xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配
置； 

3. #### 每一个自动配置类进行自动配置功能； 

   ##### 以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置原理 

```java
package org.springframework.boot.autoconfigure.web.servlet;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication.Type;
import org.springframework.boot.autoconfigure.web.ServerProperties;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.filter.OrderedCharacterEncodingFilter;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.boot.web.servlet.server.Encoding;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.web.filter.CharacterEncodingFilter;

//表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@Configuration(
    proxyBeanMethods = false
)
/**
* 启动指定类的ConfigurationProperties功能；
* 将配置文件中对应的值和ServerProperties绑定起来；
* 并把ServerProperties加入到ioc容器中
*/
@EnableConfigurationProperties({ServerProperties.class})

/**
* Spring底层@Conditional注解（Spring注解版）
* 根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效； 
* 判断当前应用是否是web应用，如果是，当前配置类生效
*/
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
//判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；
@ConditionalOnClass({CharacterEncodingFilter.class})
/**
* 判断配置文件中是否存在某个配置server.servlet.encoding.enabled；
* 如果不存在，判断也是成立的,即使我们配置文件中不配置server.servlet.encoding.enabled=true，也是默认生效的
*/
@ConditionalOnProperty(
    prefix = "server.servlet.encoding",
    value = {"enabled"},
    matchIfMissing = true
)
public class HttpEncodingAutoConfiguration {
    
    //他已经和SpringBoot的配置文件映射了
    private final Encoding properties;

    //只有一个有参构造器的情况下，参数的值就会从容器中拿
    public HttpEncodingAutoConfiguration(ServerProperties properties) {
        this.properties = properties.getServlet().getEncoding();
    }

    @Bean//给容器中添加一个组件，这个组件的某些值需要从properties中获取
    @ConditionalOnMissingBean//判断容器没有这个组件(容器中没有才会添加这个组件)
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(org.springframework.boot.web.servlet.server.Encoding.Type.RESPONSE));
        return filter;
    }

    @Bean
    public HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer localeCharsetMappingsCustomizer() {
        return new HttpEncodingAutoConfiguration.LocaleCharsetMappingsCustomizer(this.properties);
    }

    static class LocaleCharsetMappingsCustomizer implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>, Ordered {
        private final Encoding properties;

        LocaleCharsetMappingsCustomizer(Encoding properties) {
            this.properties = properties;
        }

        public void customize(ConfigurableServletWebServerFactory factory) {
            if (this.properties.getMapping() != null) {
                factory.setLocaleCharsetMappings(this.properties.getMapping());
            }

        }

        public int getOrder() {
            return 0;
        }
    }
}
```

+ 根据当前不同的条件判断，决定这个配置类是否生效

+ 一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

**所有在配置文件中能配置的属性都是在xxxxProperties类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类** 

![24](\images\posts\springboot\24.jpg)

### 总结

1. SpringBoot启动会加载大量的自动配置类

2. 我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；

3. 我们再来看这个自动配置类中到底配置了哪些组件（只要我们要用的组件有，我们就不需要再来配置了）

4. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；

   xxxxAutoConfigurartion：自动配置类；给容器中添加组件

   xxxxProperties:封装配置文件中相关属性； 

## 补充：

### @Conditional派生注解 

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效 

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

### 查看哪些自动配置类生效了

自动配置类必须在一定的条件下才能生效；

我们怎么知道哪些自动配置类生效：

我们可以通过配置文件启用 `debug=true`属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效

- `Positive matches ` ：（自动配置类启用的）
- `Negative matches`：（没有启动，没有匹配成功的自动配置类）


  