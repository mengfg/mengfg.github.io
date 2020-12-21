---
layout: post
title: Springboot笔记（七） SpringbootWeb开发之SpringMVC配置
date: 2020-12-07
tags: Springboot
---

## SpringMVC自动配置

Spring Boot为Spring MVC提供了自动配置，可与大多数应用程序完美配合。

以下类是SpringBoot对SpringMVC的默认配置

```properties
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
```

自动配置在Spring的默认值之上添加了以下功能：

- 包含`ContentNegotiatingViewResolver`和`BeanNameViewResolver`。--> **视图解析器**
- 支持服务静态资源，包括对WebJars的支持。--> **静态资源文件夹路径**
- 自动注册`Converter`，`GenericConverter`和`Formatter `beans。--> **转换器，格式化器**
- 支持`HttpMessageConverters`。--> **SpringMVC用来转换Http请求和响应的；例如实体类User转换为Json格式**
- 自动注册`MessageCodesResolver`。--> **定义错误代码生成规则**
- 静态`index.html`支持。--> **静态首页访问**
- 定制`Favicon`支持。--> **网站图标**
- 自动使用`ConfigurableWebBindingInitializer`bean。

如果想保留 Spring Boot MVC 的功能，并且需要添加其他 [MVC 配置](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc)（拦截器，格式化程序和视图控制器等），可以添加自己的 `WebMvcConfigurer` 类型的 `@Configuration` 类，但**不能**带 `@EnableWebMvc` 注解。如果您想自定义 `RequestMappingHandlerMapping`、`RequestMappingHandlerAdapter` 或者 `ExceptionHandlerExceptionResolver` 实例，可以声明一个 `WebMvcRegistrationsAdapter` 实例来提供这些组件。

如果您想完全掌控 Spring MVC，可以添加自定义注解了 `@EnableWebMvc` 的 @Configuration 配置类。

以上只是关于SpringMVC的自动配置，**org.springframework.boot.autoconfigure.web**包下是关于web方面所有的自动配置

### 视图解析器

#### 源码分析

视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发？重定向？）

- 自动配置了ViewResolver
- ContentNegotiatingViewResolver：组合所有的视图解析器的；

源码分析：WebMvcAutoConfiguration自动配置类中会通过@Bean向容器中添加一个ContentNegotiatingViewResolver组件

![40](\images\posts\springboot\40.jpg)

添加的ContentNegotiatingViewResolver组件有解析视图的功能：

![41](\images\posts\springboot\41.jpg)

如何获取所有的视图解析器？

![42](\images\posts\springboot\42.jpg)

#### 自定义视图解析器

根据以上源码，我们可以自己给容器中添加一个视图解析器；自动的将其组合进来

```java
import org.springframework.web.servlet.View;
import org.springframework.web.servlet.ViewResolver;

import java.util.Locale;

//自定义视图解析器，需要实现ViewResolver接口
public class MyViewResolver implements ViewResolver {

    @Override
    public View resolveViewName(String viewName, Locale locale) throws Exception {
        return new HelloView();
    }
}
```

```java
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.View;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Map;

//自定义视图，需要实现View接口
@Component
public class HelloView implements View {

    @Override
    public String getContentType() {
        return "text/html";
    }

    @Override
    public void render(Map<String, ?> model, HttpServletRequest request,
                       HttpServletResponse response) throws Exception {
        response.setHeader("Content-type", "text/html;charset=UTF-8");
        //2.让servlet用UTF-8转码
        response.setCharacterEncoding("UTF-8");

        response.getWriter().print("hello view: 自定义视图解析器测试");
    }
}
```

```java
//使用Bean标签将自定义的视图解析器添加至容器中
@Bean
public ViewResolver myViewResolver() {
    return new MyViewResolver();
}
```

通过断点调试发现，我们定义的视图解析器已经加入进来了

![43](\images\posts\springboot\43.jpg)

### 转换器、格式化器

- `Converter`：转换器。如：public String hello(User user)：类型转换使用Converter（表单数据转为user）
- `Formatter` 格式化器；如： 2017.12.17转成Date；

默认的时间日期格式为：

![44](\images\posts\springboot\44.jpg)

所以在使用的时候如果我们没有在application中配置我们自定义的对应的格式，我们就必须遵守这3种格式来配置日期数据。当然我们也可以在application中自定义日期格式

```yaml
spring:
  mvc:
    format:
      date: yyyy-MM-dd
```

自己也可以自定义格式化器、转换器，只需要放在容器中即可

### HttpMessageConverters

- `HttpMessageConverter`：SpringMVC用来转换Http请求和响应的；User---Json；
- `HttpMessageConverters` 是从容器中确定；获取所有的HttpMessageConverter；

自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）

## 拓展SpringMVC

以前的配置文件中的配置

```xml
<mvc:view-controller path="/hello" view-name="success"/>
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

**现在，编写一个配置类（@Configuration），是WebMvcConfigurer类型；不能标注@EnableWebMvc**

这样既保留了所有的自动配置，也能用我们扩展的配置； 

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    ………………
}
```

### 视图映射

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    //视图映射
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        //访问mengfg路径会映射到index页面
        registry.addViewController("/mengfg").setViewName("index");
    }
}
```

### 格式化器

用来可以对请求过来的日期格式化的字符串来做定制化。当然通过application.properties配置也可以办到。

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    //格式化器
    @Override
    public void addFormatters(FormatterRegistry registry) {
        //匿名内部类方式
        registry.addFormatter(new Formatter<Date>() {
            @Override
            public String print(Date date, Locale locale) {
                return null;
            }

            @Override
            public Date parse(String s, Locale locale) throws ParseException {
                return new SimpleDateFormat("yyyy-MM-dd").parse(s);
            }
        });
    }
}
```

### 消息转换器扩展fastjson

#### 在pom.xml中引入fastjson

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>fastjson</artifactId>
   <version>1.2.47</version>
</dependency>
```

#### 配置消息转换器，添加fastjson

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    //消息转换器
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        FastJsonHttpMessageConverter fc = new FastJsonHttpMessageConverter();
        FastJsonConfig fastJsonConfig = new FastJsonConfig();
        fastJsonConfig.setSerializerFeatures(SerializerFeature.PrettyFormat);
        fc.setFastJsonConfig(fastJsonConfig);
        converters.add(fc);
    }
}
```

#### 实体类

```java
//在实体类上可以继续控制
@Data
public class User {
    private String username;
    private String password;
    private int age;
    private int score;
    private int gender;
    @JSONField(format = "yyyy年MM月dd")
    private Date date;
}
```

#### 测试

```java
@ResponseBody
@RequestMapping("/user")
public User user() throws Exception{
    User user = new User();
    user.setUsername("孟繁国");
    user.setPassword("123");
    user.setAge(27);
    user.setGender(1);
    user.setScore(100);
    user.setDate(new SimpleDateFormat("yyyy-MM-dd").parse("1993-01-01"));
    return user;
}

访问输出：
{ "age":27, "date":"1993年01月01", "gender":1, "password":"123", "score":100, "username":"孟繁国" }
```

### 拦截器

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    //拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //拦截除了/hello请求之外的所有请求
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**").excludePathPatterns("/hello");
    }
}
```

```java
//拦截器代码
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("前置拦截");
        return true;
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("后置拦截");
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("最终拦截");
    }
}
```

### 资源映射

```java
@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    
    //资源映射
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        //访问路径为/res来映射类路径下test_static中的文件
        //如：http://localhost:8080/res/1.jpg
        registry.addResourceHandler("/res/**").addResourceLocations("classpath:/test_static/");
    }
}
```



原理：

我们知道`WebMvcAutoConfiguration`是SpringMVC的自动配置类

下面这个类是`WebMvcAutoConfiguration`中的一个内部类

![45](\images\posts\springboot\45.jpg)

看一下`@Import({WebMvcAutoConfiguration.EnableWebMvcConfiguration.class})`中的这个类，他依旧是`WebMvcAutoConfiguration`中的一个内部类

![46](\images\posts\springboot\46.jpg)

重点看一下这个类继承的父类`DelegatingWebMvcConfiguration`

```java
//在该配置类中会通过setConfigurers方法收集所有的WebMvcConfigurer
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {

	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

	//从容器中获取所有的WebMvcConfigurer
	@Autowired(required = false)
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);
		}
	}

    ......
    
        
    /**
     * 以该类中一个方法为例
      * this.configurers：也是WebMvcConfigurer接口的一个实现类(WebMvcConfigurerComposite类)
      * 看一下调用的addViewControllers方法 ↓
      */
    @Override
	protected void addViewControllers(ViewControllerRegistry registry) {
		this.configurers.addViewControllers(registry);
	}
    
   //addViewControllers方法会进入WebMvcConfigurerComposite类实现
    /**
     * 每一个方法的实现，都在遍历所有的WebMvcConfigurer，
     * 从而调用每一个WebMvcConfigurer对应的方法完成每一个WebMvcConfigurer的统一注册添加和配置
     */
    @Override
	public void addViewControllers(ViewControllerRegistry registry) {
        //将所有的WebMvcConfigurer相关配置都来一起调用；
		for (WebMvcConfigurer delegate : this.delegates) {
			delegate.addViewControllers(registry);
		}
	}
```

容器中所有的WebMvcConfigurer都会一起起作用；

我们的配置类也会被调用；

效果：SpringMVC的自动配置和我们的扩展配置都会起作用；

## 全面接管SpringMVC

SpringBoot对SpringMVC的自动配置不需要了，所有都是由我们自己来配置；所有的SpringMVC的自动配置都失效了

**我们只需要在配置类中添加@EnableWebMvc即可；**

```java
@Configuration
@EnableWebMvc
public class MyMvcConfig implements WebMvcConfigurer{…………}
```

原理：

为什么@EnableWebMvc自动配置就失效了；

我们看一下EnableWebMvc注解类

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

重点在于`@Import({DelegatingWebMvcConfiguration.class})`

`DelegatingWebMvcConfiguration`是`WebMvcConfigurationSupport`的子类

```java
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {……}
```

我们再来看一下springmvc的自动配置类`WebMvcAutoConfiguration`

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@ConditionalOnClass({Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class})
//重点是这个注解，只有当容器中没有WebMvcConfigurationSupport这个类型组件的时候该配置类才会生效
@ConditionalOnMissingBean({WebMvcConfigurationSupport.class})
@AutoConfigureOrder(-2147483638)
@AutoConfigureAfter({DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class, ValidationAutoConfiguration.class})
public class WebMvcAutoConfiguration {……}
```

@EnableWebMvc将WebMvcConfigurationSupport组件导入进来，导入的WebMvcConfigurationSupport只是SpringMVC最基本的功能 

## 如何修改Springboot的默认配置

SpringBoot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（如：ViewResolver）将用户配置的和自己默认的组合起来；

- 在SpringBoot中会有非常多的xxxConfigurer帮助我们进行扩展配置
- 在SpringBoot中会有很多的xxxCustomizer帮助我们进行定制配置

