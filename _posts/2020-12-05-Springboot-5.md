---
layout: post
title: Springboot笔记（五） Springboot web开发
date: 2020-12-05
tags: Springboot
---

# 05、Web开发

![4.7](\images\posts\springboot\4.7.jpg)

# 1、SpringMVC自动配置概览

Spring Boot provides auto-configuration for Spring MVC that **works well with most applications.(大多场景我们都无需自定义配置)**

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
  - 内容协商视图解析器和BeanName视图解析器

- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).
  - 静态资源（包括webjars）

- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
  - 自动注册 `Converter，GenericConverter，Formatter `

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).
  - 支持 `HttpMessageConverters` （后来我们配合内容协商理解原理）

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).
  - 自动注册 `MessageCodesResolver` （国际化用）

- Static `index.html` support.
  - 静态index.html 页支持

- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)).
  - 自定义 `Favicon`  

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).
  - 自动使用 `ConfigurableWebBindingInitializer` ，（DataBinder负责将请求数据绑定到JavaBean上）


> If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.2.9.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.
>
> **不用@EnableWebMvc注解。使用** **@Configuration** **+** **WebMvcConfigurer** **自定义规则**



> If you want to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, and still keep the Spring Boot MVC customizations, you can declare a bean of type `WebMvcRegistrations` and use it to provide custom instances of those components.
>
> **声明** **WebMvcRegistrations** **改变默认底层组件**



> If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`, or alternatively add your own `@Configuration`-annotated `DelegatingWebMvcConfiguration` as described in the Javadoc of `@EnableWebMvc`.
>
> **使用** **@EnableWebMvc+@Configuration+DelegatingWebMvcConfiguration 全面接管SpringMVC**





# 2、简单功能分析

## 2.1、静态资源访问

### 1、静态资源目录

将静态资源放在类路径下的这些文件夹中：  `/static` (or `/public` or `/resources` or `/META-INF/resources`即可

访问时 ： 当前项目根路径/ + 静态资源名	如：http://localhost:8080/timg.gif



原理： 静态资源映射/**，也就是拦截所有请求的

请求进来，先去找Controller看能不能处理。不能处理的所有请求又都交给静态资源处理器。静态资源也找不到则响应404页面



可以改变默认的静态资源路径，设置成自定义的

```yaml
spring:
  resources:
  	#是一个数组可以写多个位置
  	#static-locations: classpath:/haha/
    static-locations: [classpath:/haha/,classpath:/hehe/]
    
    这时候就需要将静态资源放在类路径下的haha或者hehe中才能访问到
```



### 2、静态资源访问前缀

默认无前缀，如果想修改需要在配置文件中进行配置

```yaml
spring:
  mvc:
    static-path-pattern: /res/**
```

此时再去访问时：当前项目 + static-path-pattern + 静态资源名	如：http://localhost:8080/res/timg.gif



### 3、webjar

官网：https://www.webjars.org/

自动映射	 /[webjars](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)/**



以jquery为例：需要导入依赖

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

访问时：[http://localhost:8080/webjars/**jquery/3.5.1/jquery.js**](http://localhost:8080/webjars/jquery/3.5.1/jquery.js)   后面地址要按照依赖里面的包路径



## 2.2、欢迎页支持

以下两种情况都会被当做欢迎页面，当访问根目录时会访问到该页面：http://localhost:8080/

- 静态资源路径下放置 index.html

  注意：

  - 可以配置静态资源路径
  - 但是不可以配置静态资源的访问前缀。否则导致 index.html不能被默认访问


```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致welcome page欢迎页功能失效

  resources:
    static-locations: [classpath:/haha/]
```

- controller能处理/index





## 2.3、自定义 `Favicon`

只要将 favicon.ico 图片放在静态资源目录下即可。（可能会存在缓存的问题，注意清空下浏览器缓存）

主要注意：

```yaml
spring:
#  mvc:
#    static-path-pattern: /res/**   这个会导致 Favicon 功能失效
```





## 2.4、静态资源配置原理

- SpringBoot启动默认加载  xxxAutoConfiguration 类（自动配置类）
- SpringMVC功能的自动配置类大多在 WebMvcAutoConfiguration，生效

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
                     ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

- 给容器中配了什么。

```java
@Configuration(proxyBeanMethods = false)
@Import(EnableWebMvcConfiguration.class)
@EnableConfigurationProperties({ WebMvcProperties.class, ResourceProperties.class })
@Order(0)
public static class WebMvcAutoConfigurationAdapter implements WebMvcConfigurer {}
```

- 配置文件的相关属性和xxx进行了绑定。WebMvcProperties和**spring.mvc**下的属性进行绑定ResourceProperties==**spring.resources**下的属性进行绑定





#### 1、配置类只有一个有参构造器

```java
//有参构造器所有参数的值都会从容器中确定
//ResourceProperties resourceProperties：获取和spring.resources绑定的所有的值的对象
//WebMvcProperties mvcProperties： 获取和spring.mvc绑定的所有的值的对象
//ListableBeanFactory beanFactory： Spring的beanFactory
//HttpMessageConverters：找到所有的HttpMessageConverters
//ResourceHandlerRegistrationCustomizer： 找到 资源处理器的自定义器
//DispatcherServletPath  
//ServletRegistrationBean   给应用注册Servlet、Filter....
public WebMvcAutoConfigurationAdapter(ResourceProperties resourceProperties, 
                                      WebMvcProperties mvcProperties,
                                      ListableBeanFactory beanFactory, 		ObjectProvider<HttpMessageConverters> messageConvertersProvider,
                                      ObjectProvider<ResourceHandlerRegistrationCustomizer> resourceHandlerRegistrationCustomizerProvider,
                                      ObjectProvider<DispatcherServletPath> dispatcherServletPath,
                                      ObjectProvider<ServletRegistrationBean<?>> servletRegistrations) {
    this.resourceProperties = resourceProperties;
    this.mvcProperties = mvcProperties;
    this.beanFactory = beanFactory;
    this.messageConvertersProvider = messageConvertersProvider;
    this.resourceHandlerRegistrationCustomizer = resourceHandlerRegistrationCustomizerProvider.getIfAvailable();
    this.dispatcherServletPath = dispatcherServletPath;
    this.servletRegistrations = servletRegistrations;
}
```





#### 2、资源处理的默认规则

```java
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    //判断静态资源规则是否被禁用。（可以在配置文件中配置禁用）
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    //获取缓存策略
    Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
    CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
    
    /**
    * webjars的规则。
    这就是为什么webjars资源访问时，只要写	webjars/层级路径	即可，
    其实是去classpath:/META-INF/resources/webjars/路径下找
    */
    if (!registry.hasMappingForPattern("/webjars/**")) {
        customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
                                             .addResourceLocations("classpath:/META-INF/resources/webjars/")
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));//setCachePeriod设置缓存策略
    }

    //非webjars规则。从静态资源路径中找，如果静态资源路径自定义了就用自定义的，没有自定义就用默认的
    String staticPathPattern = this.mvcProperties.getStaticPathPattern();
    if (!registry.hasMappingForPattern(staticPathPattern)) {
        
        customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
                                             .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
                                             .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));//setCachePeriod设置缓存策略
    }
}



//这里配置类中规定了默认的静态资源路径
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
            "classpath:/resources/", "classpath:/static/", "classpath:/public/" };

    /**
     * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
     * /resources/, /static/, /public/].
     */
    private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

通过源码发现，还可以进行如下的配置来影响静态资源的处理的默认规则

```yaml
spring:
  resources:
    add-mappings: false   #禁用所有静态资源规则
    cache:
      period: 11000	#配置缓存时间，以秒为单位
```



#### 3、欢迎页的处理规则

```java
HandlerMapping：处理器映射。保存了每一个Handler能处理哪些请求。  

    @Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
                                                                   FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
    WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
        new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
        this.mvcProperties.getStaticPathPattern());
    welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
    welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
    return welcomePageHandlerMapping;
}

//构造方法
WelcomePageHandlerMapping(TemplateAvailabilityProviders templateAvailabilityProviders,
                          ApplicationContext applicationContext, Optional<Resource> welcomePage, String staticPathPattern) {
    if (welcomePage.isPresent() && "/**".equals(staticPathPattern)) {
        //要用欢迎页功能，必须是/**，这就是为什么配置静态资源的访问前缀不是/**会导致欢迎页失效
        logger.info("Adding welcome page: " + welcomePage.get());
        setRootViewName("forward:index.html");
    }
    else if (welcomeTemplateExists(templateAvailabilityProviders, applicationContext)) {
        // 调用Controller  /index
        logger.info("Adding welcome page template: index");
        setRootViewName("index");
    }
}
```

#### 4、favicon

和代码没有多大关系。主要是因为浏览器都会默认发	当前项目/favicon.ico	所以当我们配置了静态资源的访问前缀，那么访问路径就会发生变化，需要加上前缀。所以，浏览器再发	当前项目/favicon.ico	请求时自然找不到资源。

# 3、请求参数处理

## 0、请求映射

### 1、rest使用与原理

- @xxxMapping；
- Rest风格支持（*使用**HTTP**请求方式动词来表示对资源的操作*）

  - 以前：**/getUser*   *获取用户*     */deleteUser* *删除用户*    */editUser*  *修改用户*       */saveUser* *保存用户*

  - 现在： /user*    *GET-**获取用户*    *DELETE-**删除用户*     *PUT-**修改用户*      *POST-**保存用户*

  - 核心Filter：HiddenHttpMethodFilter

    - 用法： 表单method=post，隐藏域 _method=put

      ```html
      <form action="/user" method="post">
          <input name="_method" type="hidden" value="PUT"/>
          <input type="submit" value="提交"/>
      </form>
      
      <form action="/user" method="post">
          <input name="_method" type="hidden" value="DELETE"/>
          <input type="submit" value="提交"/>
      </form>
      ```

      ```java
      //java代码中可以使用RequestMapping注解，然后以method参数进行区分
      //也可以使用对应的注解
      //@RequestMapping(value = "/user",method = RequestMethod.GET)
      @GetMapping("/user")
      public String getUser(){
          return "GET-张三";
      }
      
      //@RequestMapping(value = "/user",method = RequestMethod.POST)
      @PostMapping("/user")
      public String saveUser(){
          return "POST-张三";
      }
      
      
      //@RequestMapping(value = "/user",method = RequestMethod.PUT)
      @PutMapping("/user")
      public String putUser(){
          return "PUT-张三";
      }
      
      @DeleteMapping("/user")
      //@RequestMapping(value = "/user",method = RequestMethod.DELETE)
      public String deleteUser(){
          return "DELETE-张三";
      }
      ```

    - SpringBoot中手动开启

      ```yaml
      spring:
        mvc:
          hiddenmethod:
            filter:
              enabled: true
      ```

    - 为什么还需要手动开启？

      ```java
      //WebMvcAutoConfiguration中定义了这个组件
      @Bean
      @ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
      //但是如果你没有配置，默认是false的
      @ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled", matchIfMissing = false)
      public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
          return new OrderedHiddenHttpMethodFilter();
      }
      ```

  - 扩展：如何把_method 这个名字换成我们自己喜欢的。

    ```java
    //自定义filter
        @Bean
        public HiddenHttpMethodFilter hiddenHttpMethodFilter(){
            HiddenHttpMethodFilter methodFilter = new HiddenHttpMethodFilter();
            methodFilter.setMethodParam("_m");//修改成自定义的名字
            return methodFilter;
        }
    ```




**Rest原理（表单提交要使用REST的时候）**：

因为表单中的method默认只能选择POST/GET方式，所以只能通过这种隐藏**_method**方式

- 表单提交会带上**_method=PUT**
- **请求过来被**HiddenHttpMethodFilter拦截
  - 请求是否正常，并且是POST方式
    - 获取到**_method**的值。
    - 判断**_method**的值是否是以下请求之一；**PUT**，**DELETE**，**PATCH**
    - 原生request（post），包装模式requesWrapper重写了getMethod方法，返回的是传入的值（也就是**_method**的值）。
    - 过滤器链放行的时候把wrapper作为request对象放行。所以后面的方法调用getMethod是调用**requesWrapper**的




**Rest使用客户端工具，**

- 如PostMan直接发送Put、delete等方式请求，无需Filter。

  所以Springboot才会把这块变成选择性开启。比如前后端分离可能都不需要这些，直接发送相应的PUT，delete就可以了

```yaml
spring:
  mvc:
    hiddenmethod:
      filter:
        enabled: true   #开启页面表单的Rest功能
```

### 2、请求映射原理



![4.3](\images\posts\springboot\4.3.jpg)

SpringMVC功能分析都从 org.springframework.web.servlet.DispatcherServlet	的 doDispatch()方法开始

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;

        WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

        try {
            ModelAndView mv = null;
            Exception dispatchException = null;

            try {
                processedRequest = checkMultipart(request);
                multipartRequestParsed = (processedRequest != request);

                // 找到当前请求使用哪个Handler（Controller的方法）处理
                mappedHandler = getHandler(processedRequest);
                            
```

```java

//getHandler源码
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    if (this.handlerMappings != null) {
        for (HandlerMapping mapping : this.handlerMappings) {
            HandlerExecutionChain handler = mapping.getHandler(request);
            if (handler != null) {
                return handler;
            }
        }
    }
    return null;
}
```



HandlerMapping：处理器映射。如：/xx请求---》由XX处理 这种映射规则都保存在处理器映射中

默认会存在以下五个：

![4.4](\images\posts\springboot\4.4.jpg)



**RequestMappingHandlerMapping**：保存了所有@RequestMapping 和handler的映射规则。

![4.5](\images\posts\springboot\4.5.jpg)



所有的请求映射都在HandlerMapping中。

- SpringBoot自动配置了欢迎页的 WelcomePageHandlerMapping 。访问 /能访问到index.html；
- SpringBoot自动配置了默认 的 RequestMappingHandlerMapping
- 请求进来，挨个尝试所有的HandlerMapping看是否有请求信息。
  - 如果有就找到这个请求对应的handler
  - 如果没有就是下一个 HandlerMapping

- 我们需要一些自定义的映射处理，我们也可以自己给容器中放**HandlerMapping**。自定义 **HandlerMapping**



## 1、普通参数与基本注解

### 1.1、注解：

@PathVariable（路径变量）、@RequestHeader(获取请求头)、@ModelAttribute、@RequestParam（获取请求参数）、@MatrixVariable（矩阵变量）、@CookieValue（获取cookie的值）、@RequestBody（获取请求体，只有POST请求才有请求体）、@RequestAttribute（获取request域数据）

#### @PathVariable（路径变量）

```java
@RestController
public class ParameterTestController {
    //@PathVariable(路径变量)，一般是用在restful风格的请求中
    @GetMapping("/car/{id}/owner/{username}")
    public Map<String,Object> getCar(@PathVariable("id") Integer id,
                                     @PathVariable("username") String name,
                                     //这里形参是个map，则会将所有的值以key-value形式存放在该map中
                                     @PathVariable Map<String,String> pv){
        Map<String,Object> map = new HashMap<>();
        map.put("id",id);
        map.put("name",name);
        map.put("pv",pv);
        return map;
    }
    //发送请求：http://localhost:8080/car/1/owner/zhangsan
    //返回结果:{"pv":{"id":"1","username":"zhangsan"},"name":"zhangsan","id":1}
}
```

#### @RequestHeader(获取请求头)

```java
//测试@RequestHeader(获取请求头)
@GetMapping("/getHeader")
public Map<String,Object> getHeader(@RequestHeader("User-Agent") String userAgent,
                                    //可以使用map接收所有的请求头信息
                                    @RequestHeader Map<String,String> header){
    Map<String,Object> map = new HashMap<>();
    map.put("userAgent",userAgent);
    map.put("headers",header);
    return map;
}
    //发送请求：http://localhost:8080/getHeader
   //输出结果：{"headers":{"host":"localhost:8080","connection":"keep-alive","upgrade-insecure-requests":"1","user-agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36","accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9","sec-fetch-site":"none","sec-fetch-mode":"navigate","sec-fetch-dest":"document","accept-encoding":"gzip, deflate, br","accept-language":"zh,en-US;q=0.9,en;q=0.8,zh-CN;q=0.7","cookie":"Idea-f36109ca=f9222e91-4f75-49c2-aabd-529bcbf7b48a; JSESSIONID=EDB66560D83FAD1EFB081875A19687FC"},"userAgent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.83 Safari/537.36"}
    
```

#### @RequestParam（获取请求参数）

```java
//测试@RequestParam（获取请求参数）
@GetMapping("/getRequest")
public Map<String,Object> getRequest(@RequestParam("age") Integer age,
                                     @RequestParam("inters") List<String> inters,
                                     //使用map接收所有的参数
                                     @RequestParam Map<String,String> params){
    Map<String,Object> map = new HashMap<>();
    map.put("age",age);
    map.put("inters",inters);
    map.put("params",params);
    return map;
}
//发送请求：http://localhost:8080/getRequest?age=18&inters=football&inters=game
//返回结果:{"inters":["football","game"],"params":{"age":"18","inters":"football"},"age":18}
```

#### @CookieValue（获取cookie的值）

```java
//测试@CookieValue（获取cookie的值）
@GetMapping("/getRequest")
public Map<String,Object> getRequest(@CookieValue("JSESSIONID") String _ga,
                                     //直接使用Cookie对象接收，获取JSESSIONID的所有属性值
                                     @CookieValue("JSESSIONID") Cookie cookie){
    Map<String,Object> map = new HashMap<>();
    map.put("_ga",_ga);
    map.put("cookie",cookie);
    return map;
}
//发送请求：http://localhost:8080/getCookie
//返回结果:{"cookie":{"name":"JSESSIONID","value":"EDB66560D83FAD1EFB081875A19687FC","version":0,"comment":null,"domain":null,"maxAge":-1,"path":null,"secure":false,"httpOnly":false},"_ga":"EDB66560D83FAD1EFB081875A19687FC"}
```

#### @RequestBody（获取请求体，只有POST请求才有请求体）

```java
//测试RequestBody（获取请求体，只有POST请求才有请求体）

@PostMapping("/save")
public Map postMethod(@RequestBody String content){
    Map<String,Object> map = new HashMap<>();
    map.put("content",content);
    return map;
}
/**相关页面：
   <form action="/save" method="post">
        <input name="userName"/></br>
        <input name="email"/></br>
        <input type="submit" value="提交"/>
   </form>
	*/
//会封装成key=value&key=value的方式
//输出结果：{"content":"userName=mengfg&email=aaa"}
```

#### @RequestAttribute（获取request域数据）

```java
//测试@RequestAttribute（获取request域数据）
@Controller
public class RequestController {

    @GetMapping("/goto")
    public String gotoPage(HttpServletRequest request){
        request.setAttribute("msg","成功了……");
        request.setAttribute("code",200);
        //默认是转发过去，相当于return "forward:/success";所以可以获取request中的值
        return "success";
    }

    @ResponseBody
    @GetMapping("/success")
    public Map success(@RequestAttribute("msg") String msg,
                       @RequestAttribute("code") String code,
                       //也可以不使用注解，直接使用request取值
                       HttpServletRequest request){

        Object msg1 = request.getAttribute("msg");

        Map<String,Object> map = new HashMap<>();
        map.put("request_msg",msg1);
        map.put("annotation_msg",msg);
        map.put("code",code);
        return map;
    }
}
//发送请求：http://localhost:8080/goto
//返回结果:{"request_msg":"成功了……","code":"200","annotation_msg":"成功了……"}
```

#### @MatrixVariable（矩阵变量）

之前开发中我们需要传递参数时会采用:

/cars/{path}?XXX=XXX&aaa=cccc	这种方式也就是查询字符串，获取时@RequestParam注解获取即可

还有另一种方式，就是可以使用矩阵变量（以分号来区分）

cars/{path；low=34；brand=byd，Audi，yd} 

例如之前知识的一个问题：在页面开发时，cookie禁用了，session里面的内容怎么使用？

就可以使用这种方式就行重写，也就是URL重写，在访问路径上带上ID的值：

/abc;jsessionid=XXX	把值使用矩阵变量的方式进行传递



SpringBoot默认是禁用了矩阵变量的功能。需要进行手动开启：

原理：

+ 在WebMvcAutoConfiguration中会有configurePathMatch方法
+ configurePathMatch方法中会有一个UrlPathHelper进行解析
+ UrlPathHelper中会有一个removeSemicolonContent（移除分号内容）就是支持矩阵变量的。默认是true

如何开启？

```java
@Configuration
public class MyConfig{
    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void configurePathMatch(PathMatchConfigurer configurer) {
                UrlPathHelper urlPathHelper = new UrlPathHelper();
                //设置不排除分号后面的内容，矩阵变量功能就可以生效了
                urlPathHelper.setRemoveSemicolonContent(false);
                configurer.setUrlPathHelper(urlPathHelper);
            }
        };
    }
}

或者另一种方式：

@Configuration
public class MyConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

Controller中的测试代码：

```java
//测试@MatrixVariable（矩阵变量）
//矩阵变量必须有url路径变量才能被解析
    @GetMapping("/cars/{path}")
    public Map carsSell(@MatrixVariable("low") Integer low,
                        @MatrixVariable("brand") List<String> brand,
                        @PathVariable("path") String path){
        Map<String,Object> map = new HashMap<>();

        map.put("low",low);
        map.put("brand",brand);
        //注意看下他的路径到底是什么，注意是不包含分号后面内容的
        map.put("path",path);
        return map;
    }
/**发送请求：
http://localhost:8080/cars/sell;low=34;brand=byd,audi,yd
	或者
http://localhost:8080/cars/sell;low=34;brand=byd;brand=audi;brand=yd
两种写法表达的意思相同
*/
//返回结果:{"path":"sell","low":34,"brand":["byd","audi","yd"]}


    // /boss/1;age=20/2;age=10

//如果存在相同的变量，需要使用pathVar来进行区分
    @ResponseBody
    @GetMapping("/boss/{bossId}/{empId}")
    public Map boss(@MatrixVariable(value = "age",pathVar = "bossId") Integer bossAge,
                    @MatrixVariable(value = "age",pathVar = "empId") Integer empAge){
        Map<String,Object> map = new HashMap<>();
        map.put("bossAge",bossAge);
        map.put("empAge",empAge);
        return map;
    }
//发送请求：http://localhost:8080//boss/1;age=20/2;age=10	存在相同的变量age
//返回结果：{"bossAge":20,"empAge":10}
```





### 1.2、Servlet API：

WebRequest、ServletRequest、MultipartRequest、 HttpSession、javax.servlet.http.PushBuilder、Principal、InputStream、Reader、HttpMethod、Locale、TimeZone、ZoneId

像这种ServletAPI可以在方法定义的时候直接作为形参

**ServletRequestMethodArgumentResolver  解析以上的部分参数**

```java
@Override
    public boolean supportsParameter(MethodParameter parameter) {
        Class<?> paramType = parameter.getParameterType();
        return (WebRequest.class.isAssignableFrom(paramType) ||
                ServletRequest.class.isAssignableFrom(paramType) ||
                MultipartRequest.class.isAssignableFrom(paramType) ||
                HttpSession.class.isAssignableFrom(paramType) ||
                (pushBuilder != null && pushBuilder.isAssignableFrom(paramType)) ||
                Principal.class.isAssignableFrom(paramType) ||
                InputStream.class.isAssignableFrom(paramType) ||
                Reader.class.isAssignableFrom(paramType) ||
                HttpMethod.class == paramType ||
                Locale.class == paramType ||
                TimeZone.class == paramType ||
                ZoneId.class == paramType);
    }
```



### 1.3、复杂参数：

**Map**、**Model（map、model里面的数据会被放在request的请求域，相当于 request.setAttribute）、**Errors/BindingResult、**RedirectAttributes（ 重定向携带数据）**、**ServletResponse（response）**、SessionStatus、UriComponentsBuilder、ServletUriComponentsBuilder

```java
Map<String,Object> map,  Model model, HttpServletRequest request 都是可以给request域中放数据，
使用request.getAttribute()可以获取值;
```

**Map、Model类型的参数**，会返回 mavContainer.getModel（）；他是一个 BindingAwareModelMap类型， 是Model 也是Map 

**mavContainer**.getModel(); 获取到值的

![4.8](\images\posts\springboot\4.8.jpg)

![4.9](\images\posts\springboot\4.9.jpg)

![5.1](\images\posts\springboot\5.1.jpg)

### 1.4、自定义对象参数： 

可以自动类型转换与格式化，可以级联封装。

```java
/**页面：
 *     姓名： <input name="userName"/> <br/>
 *     年龄： <input name="age"/> <br/>
 *     生日： <input name="birth"/> <br/>
 *     宠物姓名：<input name="pet.name"/><br/>
 *     宠物年龄：<input name="pet.age"/>
 */
@Data
public class Person {
    
    private String userName;
    private Integer age;
    private Date birth;
    private Pet pet;
    
}

@Data
public class Pet {

    private String name;
    private String age;

}

result
```



## 2、POJO封装过程

- **ServletModelAttributeMethodProcessor**



## 3、参数处理原理

- HandlerMapping中找到能处理请求的Handler（也就是Controller中的方法）
- 为当前Handler 找一个适配器 HandlerAdapter；
- 适配器执行目标方法并确定方法参数的每一个值



### 1、HandlerAdapter

![4.6](\images\posts\springboot\4.6.jpg)

0 - 支持方法上标注@RequestMapping 

1 - 支持函数式编程的

一般只会用到这两个

### 2、执行目标方法

```java
//DispatcherServlet类中doDispatch方法中：
// 真正执行handler
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

//RequestMappingHandlerAdapter类handleInternal方法中
mav = invokeHandlerMethod(request, response, handlerMethod); //执行目标方法


//ServletInvocableHandlerMethod中的invokeForRequest方法：真正执行目标方法
Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);


//InvocableHandlerMethod中invokeForRequest方法：
//获取方法的参数值
Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
```

### 3、参数解析器-HandlerMethodArgumentResolver

确定将要执行的目标方法的每一个参数的值是什么;

SpringMVC目标方法能写多少种参数类型。取决于参数解析器。

![5.2](\images\posts\springboot\5.2.jpg)

![5.3](\images\posts\springboot\5.3.jpg)

- 当前解析器是否支持解析这种参数
- 支持就调用 resolveArgument



### 4、返回值处理器

![5.4](\images\posts\springboot\5.4.jpg)



### 5、如何确定目标方法每一个参数的值

```java
============InvocableHandlerMethod==========================
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {

        MethodParameter[] parameters = getMethodParameters();
        if (ObjectUtils.isEmpty(parameters)) {
            return EMPTY_ARGS;
        }

        Object[] args = new Object[parameters.length];
        for (int i = 0; i < parameters.length; i++) {
            MethodParameter parameter = parameters[i];
            parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
            args[i] = findProvidedArgument(parameter, providedArgs);
            if (args[i] != null) {
                continue;
            }
            if (!this.resolvers.supportsParameter(parameter)) {
                throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
            }
            try {
                args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
            }
            catch (Exception ex) {
                // Leave stack trace for later, exception may actually be resolved and handled...
                if (logger.isDebugEnabled()) {
                    String exMsg = ex.getMessage();
                    if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                        logger.debug(formatArgumentError(parameter, exMsg));
                    }
                }
                throw ex;
            }
        }
        return args;
    }
```

#### 5.1、挨个判断所有参数解析器那个支持解析这个参数

```java
    @Nullable
    private HandlerMethodArgumentResolver getArgumentResolver(MethodParameter parameter) {
        HandlerMethodArgumentResolver result = this.argumentResolverCache.get(parameter);
        if (result == null) {
            for (HandlerMethodArgumentResolver resolver : this.argumentResolvers) {
                if (resolver.supportsParameter(parameter)) {
                    result = resolver;
                    this.argumentResolverCache.put(parameter, result);
                    break;
                }
            }
        }
        return result;
    }
```

#### 5.2、解析这个参数的值

```
调用各自 HandlerMethodArgumentResolver 的 resolveArgument 方法即可
```

#### 5.3、自定义类型参数 封装POJO 

**ServletModelAttributeMethodProcessor  这个参数处理器支持**

 **判断是否为简单类型：**

```java
public static boolean isSimpleValueType(Class<?> type) {
        return (Void.class != type && void.class != type &&
                (ClassUtils.isPrimitiveOrWrapper(type) ||
                Enum.class.isAssignableFrom(type) ||
                CharSequence.class.isAssignableFrom(type) ||
                Number.class.isAssignableFrom(type) ||
                Date.class.isAssignableFrom(type) ||
                Temporal.class.isAssignableFrom(type) ||
                URI.class == type ||
                URL.class == type ||
                Locale.class == type ||
                Class.class == type));
    }
```

###  

**核心代码：**

```java
@Override
@Nullable
public final Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                                    NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

    Assert.state(mavContainer != null, "ModelAttributeMethodProcessor requires ModelAndViewContainer");
    Assert.state(binderFactory != null, "ModelAttributeMethodProcessor requires WebDataBinderFactory");

    String name = ModelFactory.getNameForParameter(parameter);
    ModelAttribute ann = parameter.getParameterAnnotation(ModelAttribute.class);
    if (ann != null) {
        mavContainer.setBinding(name, ann.binding());
    }

    Object attribute = null;
    BindingResult bindingResult = null;

    if (mavContainer.containsAttribute(name)) {
        attribute = mavContainer.getModel().get(name);
    }
    else {
        // Create attribute instance
        try {
            attribute = createAttribute(name, parameter, binderFactory, webRequest);
        }
        catch (BindException ex) {
            if (isBindExceptionRequired(parameter)) {
                // No BindingResult parameter -> fail with BindException
                throw ex;
            }
            // Otherwise, expose null/empty value and associated BindingResult
            if (parameter.getParameterType() == Optional.class) {
                attribute = Optional.empty();
            }
            bindingResult = ex.getBindingResult();
        }
    }

    if (bindingResult == null) {
        // Bean property binding and validation;
        // skipped in case of binding failure on construction.
        WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
        if (binder.getTarget() != null) {
            if (!mavContainer.isBindingDisabled(name)) {
                bindRequestParameters(binder, webRequest);
            }
            validateIfApplicable(binder, parameter);
            if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
                throw new BindException(binder.getBindingResult());
            }
        }
        // Value type adaptation, also covering java.util.Optional
        if (!parameter.getParameterType().isInstance(attribute)) {
            attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
        }
        bindingResult = binder.getBindingResult();
    }

    // Add resolved attribute and BindingResult at the end of the model
    Map<String, Object> bindingResultModel = bindingResult.getModel();
    mavContainer.removeAttributes(bindingResultModel);
    mavContainer.addAllAttributes(bindingResultModel);

    return attribute;
}
```



**WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);**

**WebDataBinder :web数据绑定器，将请求参数的值绑定到指定的JavaBean里面**

**WebDataBinder 利用它里面的 Converters 将请求数据转成指定的数据类型。再次封装到JavaBean中**



**GenericConversionService：在设置每一个值的时候，找它里面的所有converter那个可以将这个数据类型（request带来参数的字符串）转换到指定的类型（比如JavaBean -- Integer或者byte -- > file）**

![5.5](\images\posts\springboot\5.5.jpg)



全部的converter：

![5.6](\images\posts\springboot\5.6.jpg)



Converter的总接口：@FunctionalInterface**public interface** Converter<S, T>

例如StringToNumber就实现了该接口

**private static final class** StringToNumber<T **extends** Number> **implements** Converter<String, T>

未来我们可以给WebDataBinder里面放自己的Converter；



自定义 Converter：

```html
比如有这么一个场景，在页面中,宠物的传参方式变为以逗号分隔

姓名： <input name="userName"/> <br/>
年龄： <input name="age"/> <br/>
生日： <input name="birth"/> <br/>
<!--
宠物姓名：<input name="pet.name"/><br/>
宠物年龄：<input name="pet.age"/>
-->
宠物：<input name="pet" value="啊猫,3"/><br/>

```

```java
//WebMvcConfigurer定制化SpringMVC的功能
@Bean
public WebMvcConfigurer webMvcConfigurer(){
    @Override
    public void addFormatters(FormatterRegistry registry) {
        //Converter<源类型, 要转成的类型>
        registry.addConverter(new Converter<String, Pet>() {
            @Override
            public Pet convert(String source) {
                // 啊猫,3
                if (!StringUtils.isEmpty(source)) {
                    Pet pet = new Pet();
                    String[] split = source.split(",");
                    pet.setName(split[0]);
                    pet.setAge(Integer.parseInt(split[1]));
                    return pet;
                }
                return null;
            }
        });
    }
}
```





### 6、目标方法执行完成

将所有的数据都放在 **ModelAndViewContainer**；包含要去的页面地址View。还包含Model数据。

![5.7](\images\posts\springboot\5.7.jpg)

### 7、处理派发结果

**processDispatchResult**(processedRequest, response, mappedHandler, mv, dispatchException);



renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);



```java
InternalResourceView：
@Override
    protected void renderMergedOutputModel(
            Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {

        // Expose the model object as request attributes.
        exposeModelAsRequestAttributes(model, request);

        // Expose helpers as request attributes, if any.
        exposeHelpers(request);

        // Determine the path for the request dispatcher.
        String dispatcherPath = prepareForRendering(request, response);

        // Obtain a RequestDispatcher for the target resource (typically a JSP).
        RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
        if (rd == null) {
            throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
                    "]: Check that the corresponding file exists within your web application archive!");
        }

        // If already included or response already committed, perform include, else forward.
        if (useInclude(request, response)) {
            response.setContentType(getContentType());
            if (logger.isDebugEnabled()) {
                logger.debug("Including [" + getUrl() + "]");
            }
            rd.include(request, response);
        }

        else {
            // Note: The forwarded resource is supposed to determine the content type itself.
            if (logger.isDebugEnabled()) {
                logger.debug("Forwarding to [" + getUrl() + "]");
            }
            rd.forward(request, response);
        }
    }
```



```java
暴露模型作为请求域属性
// Expose the model object as request attributes.
        exposeModelAsRequestAttributes(model, request);
```



```java
protected void exposeModelAsRequestAttributes(Map<String, Object> model,
            HttpServletRequest request) throws Exception {

    //model中的所有数据遍历挨个放在请求域中
        model.forEach((name, value) -> {
            if (value != null) {
                request.setAttribute(name, value);
            }
            else {
                request.removeAttribute(name);
            }
        });
    }
```



# 4、数据响应与内容协商

![5.8](\images\posts\springboot\5.8.jpg)



## 1、响应JSON

### 1.1、jackson.jar+@ResponseBody

```xml
要引入web开发场景
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
web场景自动引入了json场景
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-json</artifactId>
      <version>2.3.4.RELEASE</version>
      <scope>compile</scope>
    </dependency>
```

web场景自动引入的json场景是Jackson的：

![5.9](\images\posts\springboot\5.9.jpg)

然后在方法上使用@ResponseBody注解即可。

就会给前端自动返回json数据；



#### 1、返回值解析器

![6.1](\images\posts\springboot\6.1.jpg)

```java
//ServletInvocableHandlerMethod中
try {
    this.returnValueHandlers.handleReturnValue(
        returnValue, getReturnValueType(returnValue), mavContainer, webRequest);
}

//HandlerMethodReturnValueHandlerComposite中
@Override
public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
                              ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {

    HandlerMethodReturnValueHandler handler = selectHandler(returnValue, returnType);
    if (handler == null) {
        throw new IllegalArgumentException("Unknown return value type: " + returnType.getParameterType().getName());
    }
    handler.handleReturnValue(returnValue, returnType, mavContainer, webRequest);
}

RequestResponseBodyMethodProcessor      
@Override
    public void handleReturnValue(@Nullable Object returnValue, MethodParameter returnType,
            ModelAndViewContainer mavContainer, NativeWebRequest webRequest)
            throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {

        mavContainer.setRequestHandled(true);
        ServletServerHttpRequest inputMessage = createInputMessage(webRequest);
        ServletServerHttpResponse outputMessage = createOutputMessage(webRequest);

        // Try even with null return value. ResponseBodyAdvice could get involved.
        // 使用消息转换器进行写出操作
        writeWithMessageConverters(returnValue, returnType, inputMessage, outputMessage);
    }
```





#### 2、返回值解析器原理

![6.2](\images\posts\springboot\6.2.jpg)

- 返回值处理器判断是否支持这种类型返回值 supportsReturnType
- 如果支持。返回值处理器调用 handleReturnValue 进行处理
- RequestResponseBodyMethodProcessor 可以处理返回值标了@ResponseBody 注解的。

  - 利用 MessageConverters 进行处理 将数据写为json

    - 内容协商（浏览器默认会以请求头的方式告诉服务器他能接受什么样的内容类型）

      ![6.3](\images\posts\springboot\6.3.jpg)

    - 服务器最终根据自己自身的能力，决定服务器能生产出什么样内容类型的数据，

    - SpringMVC会挨个遍历所有容器底层的 HttpMessageConverter ，看谁能处理？

      - 得到MappingJackson2HttpMessageConverter就可以将对象写为json
      - 利用MappingJackson2HttpMessageConverter将对象转为json再写出去。 




### 1.2、SpringMVC到底支持哪些返回值

```properties
ModelAndView
Model
View
ResponseEntity 
ResponseBodyEmitter
StreamingResponseBody
HttpEntity
HttpHeaders
Callable
DeferredResult
ListenableFuture
CompletionStage
WebAsyncTask
标注有 @ModelAttribute 且为对象类型的
标注@ResponseBody 注解 （他的是使用RequestResponseBodyMethodProcessor处理）
```

### 1.3、HTTPMessageConverter原理



#### 1、MessageConverter规范

![6.4](\images\posts\springboot\6.4.jpg)

HttpMessageConverter: 看是否支持将 此 Class类型的对象，转为MediaType类型的数据。

例子：Person对象转为JSON。或者 JSON转为Person



#### 2、默认的MessageConverter

![6.5](\images\posts\springboot\6.5.jpg)

0 - 只支持Byte类型的

1 - String

2 - String

3 - Resource

4 - ResourceRegion

5 - DOMSource.class、 SAXSource.class 、 StAXSource.class 、StreamSource.class 、Source.class

6 - MultiValueMap

7 - true（能将任意对象转换）

8 - true（能将任意对象转换）

9 - 支持注解方式xml处理的



最终 MappingJackson2HttpMessageConverter  把对象转为JSON（利用底层的jackson的objectMapper转换的）

![6.6](\images\posts\springboot\6.6.jpg)



## 2、内容协商

根据客户端接收能力不同，返回不同媒体类型的数据。

### 1、引入xml依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

### 2、postman分别测试返回json和xml

只需要改变请求头中Accept字段。Http协议中规定的，告诉服务器本客户端可以接收的数据类型。

![6.7](\images\posts\springboot\6.7.jpg)



### 3、开启浏览器参数方式内容协商功能

为了方便内容协商，开启基于请求参数的内容协商功能。

```yaml
spring:
    contentnegotiation:
      favor-parameter: true  #开启请求参数内容协商模式
```

发请求： http://localhost:8080/test/person?format=json

[http://localhost:8080/test/person?format=](http://localhost:8080/test/person?format=json)xml



![6.8](\images\posts\springboot\6.8.jpg)

确定客户端接收什么样的内容类型；

1、Parameter策略优先确定是要返回json数据（获取请求头中的format的值）

![6.9](\images\posts\springboot\6.9.jpg)

2、最终进行内容协商返回给客户端json即可。

### 4、内容协商原理

- 1、判断当前响应头中是否已经有确定的媒体类型。MediaType

- **2、获取客户端（PostMan、浏览器）支持接收的内容类型。（获取客户端Accept请求头字段，例如application/xml）**

  - **contentNegotiationManager 内容协商管理器 默认使用基于请求头的策略**

    ![7.1](\images\posts\springboot\7.1.jpg)

  - **HeaderContentNegotiationStrategy  确定客户端可以接收的内容类型** 

    ![7.2](\images\posts\springboot\7.2.jpg)

- 3、遍历循环所有当前系统的 **MessageConverter**，看谁支持操作这个对象（Person）

  ![7.3](\images\posts\springboot\7.3.jpg)

- 4、找到支持操作Person的converter，把converter支持的媒体类型统计出来。

- 5、客户端需要【application/xml】。服务端能力【10种、json、xml】

  ![7.4](\images\posts\springboot\7.4.jpg)

- 6、进行内容协商的最佳匹配媒体类型

- 7、用 支持 将对象转为 最佳匹配媒体类型 的converter。调用它进行转化 。



导入了jackson处理xml的包，xml的converter就会自动进来

```java
WebMvcConfigurationSupport
jackson2XmlPresent = ClassUtils.isPresent("com.fasterxml.jackson.dataformat.xml.XmlMapper", classLoader);


if (jackson2XmlPresent) {
    Jackson2ObjectMapperBuilder builder = Jackson2ObjectMapperBuilder.xml();
    if (this.applicationContext != null) {
        builder.applicationContext(this.applicationContext);
    }
    messageConverters.add(new MappingJackson2XmlHttpMessageConverter(builder.build()));
}
```



### 5、自定义 MessageConverter

**实现多协议数据兼容。json、xml、x-guigu**

**0、**@ResponseBody 响应数据出去 调用 **RequestResponseBodyMethodProcessor** 处理

1、Processor 处理方法返回值。通过 **MessageConverter** 处理

2、所有 **MessageConverter** 合起来可以支持各种媒体类型数据的操作（读、写）

3、内容协商找到最终的 **messageConverter**；



步骤：

1. 自定义MessageConverter

   ```java
   public class MyConverter implements HttpMessageConverter<Person> {
       //是否支持可读
       //例如public Person getPerson(@RequestBody Person person),传递的参数可以直接使用@RequestBody注解读取匹配
       @Override
       public boolean canRead(Class<?> clazz, MediaType mediaType) {
           return false;
       }
   
       //是否可写
       @Override
       public boolean canWrite(Class<?> clazz, MediaType mediaType) {
           //只要是Person类型就可以写
           return clazz.isAssignableFrom(Person.class);
       }
   
       //服务器要统计所有的MessageConverter都能写出那些内容类型，调用的就是这个方法
       @Override
       public List<MediaType> getSupportedMediaTypes() {
           return MediaType.parseMediaTypes("application/x-guigu");
       }
   
       @Override
       public Person read(Class<? extends Person> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
           return null;
       }
   
       //自定义数据的写出
       @Override
       public void write(Person person, MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
           //自定义协议数据的写出。此处我们把各个属性以逗号分隔
           String data = person.getUserName()+";"+person.getAge()+";"+person.getBirth();
   
           //写出去
           OutputStream body = outputMessage.getBody();
           body.write(data.getBytes());
       }
   }
   ```

2. 添加进系统底层

   ```java
   @Configuration
   public class MyConfig {
   
       @Bean
       public WebMvcConfigurer webMvcConfigurer() {
           return new WebMvcConfigurer() {
               @Override
               public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                   converters.add(new MyConverter());
               }
           };
       }
   
   }
   ```

3. 使用postman模拟请求并修改请求头

   ![7.5](\images\posts\springboot\7.5.png)

   



需要修改SpringMVC的什么功能，都需要从一个入口给容器中添加一个： WebMvcConfigurer

```java
 @Bean
    public WebMvcConfigurer webMvcConfigurer(){
        return new WebMvcConfigurer() {

            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {

            }
        }
    }
```





以上的设置在Postman等客户端发送访问请求时有效，但是再浏览器中通过修改参数format无效。因为此时内容协商中并未加入。

![7.6](\images\posts\springboot\7.6.jpg)

需要加入自定义的内容协商策略：

```java
@Configuration
public class MyConfig {

    @Bean
    public WebMvcConfigurer webMvcConfigurer() {

        return new WebMvcConfigurer() {
            @Override
            public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
                converters.add(new MyConverter());
            }


            //自定义内容协商策略
            @Override
            public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
                Map<String,MediaType> mediaTypes = new HashMap<>();
                mediaTypes.put("json",MediaType.APPLICATION_JSON);
                mediaTypes.put("xml",MediaType.APPLICATION_XML);
                mediaTypes.put("gg",MediaType.parseMediaType("application/x-guigu"));
                //指定支持解析哪些参数对应的哪些媒体类型
                ParameterContentNegotiationStrategy parameterstrategy = new ParameterContentNegotiationStrategy(mediaTypes);

				//将请求头的方式也加入进去，否则这种方式会被覆盖掉
                HeaderContentNegotiationStrategy headerStrategy = new HeaderContentNegotiationStrategy();
                configurer.strategies(Arrays.asList(parameterstrategy,headerStrategy));

            }
        };

    }
}
```

经过以上的操作，内容协商中已经加入了

![7.7](\images\posts\springboot\7.7.jpg)

**有可能我们添加的自定义的功能会覆盖默认很多功能，导致一些默认的功能失效。（例如自定义之后会覆盖请求头的方式，所以要注意加上）**

**大家考虑，上述功能除了我们完全自定义外？SpringBoot有没有为我们提供基于配置文件的快速修改媒体类型功能？怎么配置呢？【提示：参照SpringBoot官方文档web开发内容协商章节】**



# 5、视图解析与模板引擎

视图解析：**SpringBoot默认不支持 JSP，需要引入第三方模板引擎技术实现页面渲染。**

## 1、视图解析

![7.8](\images\posts\springboot\7.8.jpg)

### 1、视图解析原理流程

1、目标方法处理的过程中，所有数据都会被放在 **ModelAndViewContainer 里面。包括数据和视图地址**

**2、方法的参数是一个自定义类型对象（从请求参数中确定的），把他重新放在** **ModelAndViewContainer** 

**3、任何目标方法执行完成以后都会返回 ModelAndView（数据和视图地址）。**

**4、processDispatchResult  处理派发结果（页面改如何响应）**

- 1、**render**(**mv**, request, response); 进行页面渲染逻辑

  - 1、根据方法的String返回值得到 **View** 对象【定义了页面的渲染逻辑】

    - 1、所有的视图解析器尝试是否能根据当前返回值得到**View**对象

      ![7.9](\images\posts\springboot\7.9.jpg)

    - 2、根据返回值得到了  **redirect:/main.html** --> Thymeleaf new **RedirectView**()

    - 3、ContentNegotiationViewResolver 里面包含了下面所有的视图解析器，内部还是利用下面所有视图解析器得到视图对象。

      ![8.1](\images\posts\springboot\8.1.jpg)

    - 4、view.render(mv.getModelInternal(), request, response);   视图对象调用自定义的render进行页面渲染工作

      ![8.2](\images\posts\springboot\8.2.jpg)

      - **RedirectView 如何渲染【重定向到一个页面】**
      - 1、获取目标url地址
      - 2、response.sendRedirect(encodedURL);




**视图解析：**

- - **返回值以 forward: 开始： new InternalResourceView(forwardUrl); -->  转发****request.getRequestDispatcher(path).forward(request, response);** 
  - **返回值以** **redirect: 开始：** **new RedirectView() --》 render就是重定向** 
  - **返回值是普通字符串： new ThymeleafView（）--->** 



我们也可以 自定义视图解析器+自定义视图



## 2、模板引擎-Thymeleaf

### 1、thymeleaf简介

Thymeleaf is a modern server-side Java template engine for both web and standalone environments, capable of processing HTML, XML, JavaScript, CSS and even plain text.

**现代化、服务端Java模板引擎**



### 2、基本语法

#### 1、表达式

| 表达式名字 | 语法   | 用途                               |
| ---------- | ------ | ---------------------------------- |
| 变量取值   | ${...} | 获取请求域、session域、对象等值    |
| 选择变量   | *{...} | 获取上下文对象值                   |
| 消息       | #{...} | 获取国际化等值                     |
| 链接       | @{...} | 生成链接                           |
| 片段表达式 | ~{...} | jsp:include 作用，引入公共页面片段 |



#### 2、字面量

文本值: **'one text'** **,** **'Another one!'** **,…**数字: **0** **,** **34** **,** **3.0** **,** **12.3** **,…**布尔值: **true** **,** **false**

空值: **null**

变量： one，two，.... 变量不能有空格

#### 3、文本操作

字符串拼接: **+**

变量替换: **|The name is ${name}|** 



#### 4、数学运算

运算符: + , - , * , / , %



#### 5、布尔运算

运算符:  **and** **,** **or**

一元运算: **!** **,** **not** 





#### 6、比较运算

比较: **>** **,** **<** **,** **>=** **,** **<=** **(** **gt** **,** **lt** **,** **ge** **,** **le** **)**等式: **==** **,** **!=** **(** **eq** **,** **ne** **)** 



#### 7、条件运算

If-then: **(if) ? (then)**

If-then-else: **(if) ? (then) : (else)**

Default: (value) **?: (defaultvalue)** 



#### 8、特殊操作

无操作： _





### 3、设置属性值-th:attr

设置单个值

```html
<form action="subscribe.html" th:attr="action=@{/subscribe}">
  <fieldset>
    <input type="text" name="email" />
    <input type="submit" value="Subscribe!" th:attr="value=#{subscribe.submit}"/>
  </fieldset>
</form>
```

设置多个值

```html
<img src="../../images/gtvglogo.png"  th:attr="src=@{/images/gtvglogo.png},title=#{logo},alt=#{logo}" />
```



以上两个的代替写法 th:xxxx

```html
<input type="submit" value="Subscribe!" th:value="#{subscribe.submit}"/>
<form action="subscribe.html" th:action="@{/subscribe}">
```



所有h5兼容的标签写法

https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#setting-value-to-specific-attributes



### 4、迭代

```html
<tr th:each="prod : ${prods}">
        <td th:text="${prod.name}">Onions</td>
        <td th:text="${prod.price}">2.41</td>
        <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```



```html
<tr th:each="prod,iterStat : ${prods}" th:class="${iterStat.odd}? 'odd'">
  <td th:text="${prod.name}">Onions</td>
  <td th:text="${prod.price}">2.41</td>
  <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
</tr>
```



### 5、条件运算

```html
<a href="comments.html"
th:href="@{/product/comments(prodId=${prod.id})}"
th:if="${not #lists.isEmpty(prod.comments)}">view</a>
```



```html
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
  <p th:case="*">User is some other thing</p>
</div>
```

#  

### 6、属性优先级

![8.3](\images\posts\springboot\8.3.jpg)



## 3、thymeleaf使用

#### 1、引入Starter

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

#### 2、自动配置好了thymeleaf

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(ThymeleafProperties.class)
@ConditionalOnClass({ TemplateMode.class, SpringTemplateEngine.class })
@AutoConfigureAfter({ WebMvcAutoConfiguration.class, WebFluxAutoConfiguration.class })
public class ThymeleafAutoConfiguration { }
```

#  

自动配好的策略

- 1、所有thymeleaf的配置值都在 ThymeleafProperties
- 2、配置好了模板引擎 **SpringTemplateEngine** 
- **3、配好了视图解析器 ThymeleafViewResolver** 
- 4、我们只需要直接开发页面，因为默认前缀是类路径下的templates，默认后缀是.html。所以我们需要在类路径下的templates文件夹下创建HTML页面

```java
    public static final String DEFAULT_PREFIX = "classpath:/templates/";

    public static final String DEFAULT_SUFFIX = ".html";  //xxx.html
```

#### 3、控制器开发

```java
@Controller
public class ViewController {
    @GetMapping("/view")
    public String view(Model model) {
        model.addAttribute("msg","你好，world");
        model.addAttribute("link","http://www.baidu.com");
        return "success";
    }
}
```



#### 4、页面开发

success.html

```html
<!DOCTYPE html>
<!--需要引入名称空间，写标签就会有提示了-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">哈哈</h1>
<h2>
    <!--会替换成http://www.baidu.com-->
    <a href="www.atguigu.com" th:href="${link}">去百度</a>  <br/>
    <!--会直接拼接@中的内容，例如http://localhost:8080/link-->
    <a href="www.atguigu.com" th:href="@{link}">去百度2</a>
</h2>
</body>
</html>
```

### Thymeleaf语法示例

#### 基础语法

##### 文本标签 th:text/th:utext

用于文本内容的显示操作。

1. **th:text** 进行文本替换 不会解析html
2. **th:utext** 进行文本替换 会解析html

```java
@RequestMapping("/th")
public String th(Model model){
    String msg = "<h1>我是h1</h1>";
    model.addAttribute("msg",msg);
    return "/course/th";
}
```

**th:text** 进行文本替换 不会解析html：

```html
<p th:text="text标签：  + ${msg}"></p>
```

![37](\images\posts\springboot\8.4.jpg)

**th:utext** 进行文本替换 会解析html：

```html
<p th:utext="utext标签： + ${msg}"></p>
```

![38](\images\posts\springboot\8.5.jpg)

使用 + 和 | | 效果是一样的，如下代码所示：

```html
<p th:utext="utext标签： + ${msg}"></p>
<p th:utext="|utext标签： ${msg}|"></p>
```

##### 字符串拼接

拼接字符串通过 + 或者 | 进行拼接，并能进行算术和逻辑运算

```java
@RequestMapping("/th")
public String th(Model model){
    model.addAttribute("a",1);
    model.addAttribute("b",2);
    model.addAttribute("flag",true);
    return "/course/th";
}
```

```html
<p th:text="${a}+${b}"></p>		输出：3
<p th:text="|${a} ${b}|"></p>	输出：12
<p th:text="${a} > ${b}"></p>	输出：false
<p th:text="!${flag}"></p>	输出：false
```

##### *{...}和${...}表达式

正常情况下 *{...} 和 ${...}是一样的，但是 *{...} 一般和 **th:object** 进行一起使用来完成对象属性的简写。

```java
@RequestMapping("/th")
public String th(Model model){
    User user = new User("ljk"，18);
    model.addAttribute("user",user);
    return "/course/th";
}
```

使用 ${...}操作：

```html
<p th:text="${user.name}"></p>
<p th:text="${user.age}"></p>

输出：
ljk
18
```

使用 *{...}操作

```html
<p th:text="*{user.name}"></p>
<p th:text="*{user.age}"></p>

输出：
ljk
18

```

使用 *{...}操作特有操作：

```html
<div th:object="${user}" >
	<p th:text="*{name}"></p>
	<p th:text="*{age}"></p>
</div>

输出：
ljk
18

```

##### @{...}链接网址表达式

一般和 th:href、th:src进行结合使用，用于显示Web 应用中的URL链接。通过@{...}表达式Thymeleaf 可以帮助我们拼接上web应用访问的全路径，同时我们可以通过（）进行参数的拼接

```html
<img th:src="@{/images/gtvglogo.png}"  />
页面生成结果：<img src="/images/gtvglogo.png"  />
访问：http://localhost:8080/images/gtvglogo.png

<a th:href="@{/product/comments(prodId=${prod.id})}" >查看</a>
页面生成结果：<a href="/product/comments?prodId=2" >查看</a>
访问：http://localhost:8080/product/comments?prodId=2

<a  th:href="@{/product/comments(prodId=${prod.id},prodId2=${prod.id})}" >查看</a>
页面生成结果：<a  href="/product/comments?prodId=2&amp;prodId2=2" >查看</a>
访问：http://localhost:8080/product/comments?prodId=2&prodId2=2

```

##### 条件判断 th:if/th:unless

**th:if** 当条件为true则显示。
**th:unless** 当条件为false 则显示。

```java
@RequestMapping("/thif")
public String thif(Model model){
    model.addAttribute("flag",true);
    return "/course/thif";
}

```

```html
<p th:if="${flag}">if判断</p>		${flag}为true，输出：if判断

<p th:unless="!${flag}">unless 判断</p>	!${flag}为false，输出：unless 判断

```

##### switch

**th:switch** 我们可以通过switch来完成类似的条件表达式的操作。

```java
@RequestMapping("/thswitch")
public String thswitch(Model model){
    User user = new User("ljk",23);
    model.addAttribute("user",user);
    return "/course/thswitch";
}

```

```html
<div th:switch="${user.name}">
    <p th:case="'ljk'">User is  ljk</p>
    <p th:case="ljk1">User is ljk1</p>
</div>

输出：
User is ljk

```

##### for循环

**th:each** 遍历集合

```java
@RequestMapping("/th")
	public String theach(Model model){	
		List<User> userList = new ArrayList<User>();
		User user1 = new User("ljk",18);
		User user2 = new User("ljk2",19);
		User user3 = new User("ljk3",20);
		User user4 = new User("lj4",21);
		userList.add(user1);
		userList.add(user2);
		userList.add(user3);
		userList.add(user4);
		model.addAttribute("userList",userList);
		List<String> strList = new ArrayList<String>();
		strList.add("ljk");
		strList.add("ljk2");
		strList.add("ljk3");
		strList.add("lj4");
		model.addAttribute("strList",strList);
		return "/course/theach";
}

```

```html
<table>
      <thead>
        <tr>
          <th>用户名称</th>
          <th>用户年龄</th>
        </tr>
      </thead>
      <tbody>
        <tr th:each="user : ${userList}" th:class="${userStat.odd}? 'odd'">
          <td th:text="${user.name}">Onions</td>
          <td th:text="${user.age}">2.41</td>
        </tr>
      </tbody>
    </table>
----------------------------------------------------------------------
    <table>
      <thead>
        <tr>
          <th>用户名称</th>
        </tr>
      </thead>
      <tbody>
        <tr th:each="str : ${strList}" th:class="${strStat.odd}? 'odd'">
          <td th:text="${str}">Onions</td>
        </tr>
      </tbody>
    </table>

我们可以通过便利的变量名+Stat 来获取索引 是否是第一个或最后一个等。如 ${strStat.odd}
 便利的变量名+Stat称作状态变量，其属性有：

1. index:当前迭代对象的迭代索引，从0开始，这是索引属性；

2. count:当前迭代对象的迭代索引，从1开始，这个是统计属性；

3. size:迭代变量元素的总量，这是被迭代对象的大小属性；

4. current:当前迭代变量；

5. even/odd:布尔值，当前循环是否是偶数/奇数（从0开始计算）；

6. first:布尔值，当前循环是否是第一个；

7. last:布尔值，当前循环是否是最后一个

```

![39](\images\posts\springboot\8.6.jpg)

#### 属性设置

th：任意html属性；来替换原生属性的值 

##### th:href

用于声明在a 标签上的href属性的链接 ，该语法会和@{..} 表达式一起使用。

```html
<a href="../home.html" th:href="@{/}">返回首页</a>
页面生成结果：<a href="/">返回首页</a>

```

##### th:class

用于声明在标签上class 属性信息。

```html
<p th:class=" 'even'? 'even' : 'odd'" th:text=" 'even'? 'even' : 'odd'"></p>
页面生成结果：<p class="even">even</p>
输出：even

```

##### th:attr

用于声明html中或自定义属性信息。

```html
<img  th:attr="src=@{/images/gtvglogo.png}" />
页面生成结果：<img src="/images/gtvglogo.png" />
如果设置了contexPath则会自动加上，如：<img src="/springboot/images/gtvglogo.png" />

```

##### th:value

用于声明html中value属性信息

```java
@RequestMapping("/thvalue")
public String thvalue(Model model){
    model.addAttribute("name", "ljk");
    return "/course/thvalue";
}

```

```html
<input type="text" th:value="${name}" />
页面生成结果：<input type="text" value="ljk">

```

##### th:action

用于声明html from标签中action属性信息

```html
<form action="subscribe.html" th:action="@{/subscribe}">
    <input type="text" name="name" value="abc"/>
</form>

页面生成结果：
<form action="/subscribe">
    <input type="text" name="name" value="abc">
</form>
如果设置了contexPath则会自动加上，如
<form action="/springboot/subscribe">
    <input type="text" name="name" value="abc">
</form>

```

##### th:id

用于声明htm id属性信息。

```java
@RequestMapping("/thid")
public String thid(Model model){
    model.addAttribute("id", 123);
    return "/course/thid";
}

```

```html
<p th:id="${id}"></p>
页面生成结果：<p id="123"></p>

```

##### th:onclick

用于声明htm 中的onclick事件。

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>Insert title here</title>
        <script type="text/javascript">
            function showUserInfo(){
                alert("i am zhuoqianmingyue!")
            }
        </script>
    </head>
    <body>
        <p th:onclick="'showUserInfo()'">点我</p>
    </body>
</html>

页面生成结果：<p onclick="showUserInfo()">点我</p>

```

##### th:selected

用于声明htm 中的selected属性信息。

```java
@RequestMapping("/thselected")
public String thselected(Model model){
    model.addAttribute("sex", 1);
    return "/course/thselected";
}

```

```html
<select>
	<option name="sex"></option>
	<option th:selected="1 == ${sex}">男</option>
	<option th:selected="0 == ${sex}">女</option>
</select>

页面生成结果：
<select>
    <option name="sex"></option>
    <option selected="selected">男</option>
    <option>女</option>
</select>

```

##### th:src

用于声明html中的img中src属性信息

```html
<img  title="GTVG logo" th:src="@{/images/gtvglogo.png}" />

页面生成结果：
<img title="GTVG logo" src="/sbe/images/gtvglogo.png">
如果设置了contexPath则会自动加上，如
<img title="GTVG logo" src="/springboot/images/gtvglogo.png">

```

##### th:style

用于声明htm中的标签 css的样式信息

```java
RequestMapping("/thstyle")
    public String thstyle(Model model){
    model.addAttribute("isShow", true);
    return "/course/thstyle";
}

```

```html
<p th:style="'display:' + @{(${isShow} ? 'none' : 'block')} + ''"></p>

页面生成结果：
<p style="display:none"></p>

```

##### th:with

用于thymeleaf模版页面中局部变量定义的使用

```java
@RequestMapping("/thwith")
public String thwith(Model model){
    model.addAttribute("today", new Date());
    List<User> users = new ArrayList<User>();
    users.add(new User("ljk",18));
    users.add(new User("ljk2",18));
    model.addAttribute("users",users);
    return "/course/thwith";
}

```

```html
<p th:with="df='dd/MMM/yyyy HH:mm'">
    Today is: <span th:text="${#dates.format(today,df)}">13 February 2011</span>
</p>

页面生成结果：
<p>
    Today is: <span>19/十二月/2020 18:15</span>
</p>



<div th:with="firstEle=${users[0]}">
    <p>
        第一个用户的名称是： <span th:text="${firstEle.name}"></span>.
    </p>
</div>

页面生成结果：
<div>
    <p>
        第一个用户的名称是： <span>ljk</span>.
    </p>
</div>

```

##### Elvis运算符

Elvis运算可以理解成简单的判断是否为null的三元运算的简写，如果值为null则显示默认值，如果不为null 则显示原有的值。

```java
@RequestMapping("/elvis")
public String elvis(Model model){
    model.addAttribute("age", null);
    model.addAttribute("age2", 18);
    return "/course/elvis";
}

```

```html
<p>Age: <span th:text="${age}?: '年龄为null'"></span></p>
<p>Age2: <span th:text="${age2}?: '年龄为null'"></span></p>
页面生成结果：
<p>Age: <span>年龄为null</span></p>
<p>Age2: <span>18</span></p>

```

##### 三元表达式

我们可以在thymeleaf的语法中使用三元表达式 具体使用方法是在th:x中通过 表达式？1选项：2选项。

```java
@RequestMapping("/threeElementOperation")
public String threeElementOperation(Model model){
    model.addAttribute("name", "ljk");
    return "/course/threeElementOperation";
}

```

```html
<p th:class=" 'even'? 'even' : 'odd'" th:text=" 'even'? 'even' : 'odd'"></p>
<p th:value="${name eq 'ljk' ? '帅哥':'丑男'}" th:text="${name eq 'ljk' ? '帅哥':'丑男'}"></p>
页面生成结果：
<p class="even">even</p>
<p value="帅哥">帅哥</p>

条件表达式操作字符：
gt：great than（大于）
ge：great equal（大于等于）
eq：equal（等于）
lt：less than（小于）
le：less equal（小于等于）
ne：not equal（不等于）

```

##### No-Operation（_）什么都不做

Elvis运算符 的一种特殊简写操作，当显示的值为null 时就什么都不做。

```java
@RequestMapping("/noOperation")
public String noOperation(Model model){
    model.addAttribute("name", null);
    return "/course/noOperation";
}

```

```html
<span th:text="${name} ?: _">no user authenticated</span>
页面生成结果：
<span>no user authenticated</span>

```

更多配置参考官方文档：https://www.thymeleaf.org/documentation.html

中文参考书册：https://www.lanzous.com/i7dzr2j



## 4、构建后台管理系统

### 1、项目创建

引入thymeleaf、web-starter、devtools、lombok相关依赖



### 2、静态资源处理

自动配置好，我们只需要把所有静态资源放到 static 文件夹下

### 3、路径构建

th:action="@{/login}"



### 4、模板抽取

th:insert/replace/include



### 5、页面跳转

```java
    @PostMapping("/login")
    public String main(User user, HttpSession session, Model model){

        if(StringUtils.hasLength(user.getUserName()) && "123456".equals(user.getPassword())){
            //把登陆成功的用户保存起来
            session.setAttribute("loginUser",user);
            //登录成功重定向到main.html;  重定向防止表单重复提交
            return "redirect:/main.html";
        }else {
            model.addAttribute("msg","账号密码错误");
            //回到登录页面
            return "login";
        }

    }
```



### 6、数据渲染

```html
    @GetMapping("/dynamic_table")
    public String dynamic_table(Model model){
        //表格内容的遍历
        List<User> users = Arrays.asList(new User("zhangsan", "123456"),
                new User("lisi", "123444"),
                new User("haha", "aaaaa"),
                new User("hehe ", "aaddd"));
        model.addAttribute("users",users);

        return "table/dynamic_table";
    }
        <table class="display table table-bordered" id="hidden-table-info">
        <thead>
        <tr>
            <th>#</th>
            <th>用户名</th>
            <th>密码</th>
        </tr>
        </thead>
        <tbody>
        <tr class="gradeX" th:each="user,stats:${users}">
            <td th:text="${stats.count}">Trident</td>
            <td th:text="${user.userName}">Internet</td>
            <td >[[${user.password}]]</td>
        </tr>
        </tbody>
        </table>
```



# 6、拦截器

1、编写一个拦截器实现HandlerInterceptor接口

2、拦截器注册到容器中（实现WebMvcConfigurer的addInterceptors）

3、指定拦截规则【如果是拦截所有，静态资源也会被拦截】

## 1、HandlerInterceptor 接口

```java
/**
 * 登录检查
 * 1、配置好拦截器要拦截哪些请求
 * 2、把这些配置放在容器中
 */
@Slf4j
public class LoginInterceptor implements HandlerInterceptor {

   //目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

        String requestURI = request.getRequestURI();
        log.info("preHandle拦截的请求路径是{}",requestURI);

        //登录检查逻辑
        HttpSession session = request.getSession();

        Object loginUser = session.getAttribute("loginUser");

        if(loginUser != null){
            //放行
            return true;
        }

        //拦截住。未登录。跳转到登录页
        request.setAttribute("msg","请先登录");
//        re.sendRedirect("/");
        request.getRequestDispatcher("/").forward(request,response);
        return false;
    }
    

    //目标方法执行完成以后
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle执行{}",modelAndView);
    }

    // 页面渲染以后
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        log.info("afterCompletion执行异常{}",ex);
    }
}
```



## 2、配置拦截器

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor())
                .addPathPatterns("/**")  //所有请求都被拦截包括静态资源
                .excludePathPatterns("/","/login","/css/**","/fonts/**","/images/**","/js/**"); //放行的请求
    }
}
```



## 3、拦截器原理

1、根据当前请求，找到**HandlerExecutionChain【**可以处理请求的handler以及handler的所有 拦截器】

![8.7](\images\posts\springboot\8.7.jpg)



2、先来**顺序执行** 所有拦截器的 preHandle方法

- 1、如果当前拦截器prehandler返回为true。则执行下一个拦截器的preHandle
- 2、如果当前拦截器返回为false。直接    倒序执行所有已经执行了的拦截器的  afterCompletion；

**3、如果任何一个拦截器返回false。直接跳出不执行目标方法**

**4、所有拦截器都返回True。执行目标方法**

**5、倒序执行所有拦截器的postHandle方法。**

**6、前面的步骤有任何异常都会直接倒序触发** afterCompletion

7、页面成功渲染完成以后，也会倒序触发 afterCompletion

![8.8](\images\posts\springboot\8.8.jpg)





# 7、文件上传

## 1、页面表单

```html
<!--注意 method，enctype-->
<form role="form" th:action="@{/upload}" method="post" enctype="multipart/form-data">
    <div class="form-group">
        <label for="exampleInputEmail1">邮箱</label>
        <input type="email" name="email" class="form-control" id="exampleInputEmail1" placeholder="Enter email">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">名字</label>
        <input type="text" name="username" class="form-control" id="exampleInputPassword1" placeholder="Password">
    </div>
    <!--单文件-->
    <div class="form-group">
        <label for="exampleInputFile">头像</label>
        <input type="file" name="headerImg" id="exampleInputFile">
    </div>
    <!--多文件，注意添加 multiple-->
    <div class="form-group">
        <label for="exampleInputFile">生活照</label>
        <input type="file" name="photos" multiple>
    </div>
    <button type="submit" class="btn btn-primary">提交</button>
</form>
```



## 2、文件上传代码



```java
    /**
     * MultipartFile 自动封装上传过来的文件
     */
    @PostMapping("/upload")
    public String upload(@RequestParam("email") String email,
                         @RequestParam("username") String username,
                         //注意使用的是@RequestPart注解
                         @RequestPart("headerImg") MultipartFile headerImg,
                         @RequestPart("photos") MultipartFile[] photos) throws IOException {

        log.info("上传的信息：email={}，username={}，headerImg={}，photos={}",
                email,username,headerImg.getSize(),photos.length);

        //保存单文件
        if(!headerImg.isEmpty()){
            //保存到文件服务器，OSS服务器
            String originalFilename = headerImg.getOriginalFilename();
            headerImg.transferTo(new File("H:\\cache\\"+originalFilename));
        }

        //保存多文件
        if(photos.length > 0){
            for (MultipartFile photo : photos) {
                if(!photo.isEmpty()){
                    String originalFilename = photo.getOriginalFilename();
                    photo.transferTo(new File("H:\\cache\\"+originalFilename));
                }
            }
        }

        return "main";
    }
```

如果想修改默认的文件上传配置，比如修改文件上传的大小，可以直接在配置文件中进行修改

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB	#上传的单个文件不能超过10M
      max-request-size: 100MB #上传的所有文件大小不能超过100M
```



## 3、自动配置原理

**文件上传自动配置类：MultipartAutoConfiguration**

**文件上传自动配置的属性类：MultipartProperties**

- 自动配置好了 **StandardServletMultipartResolver   【文件上传解析器】**
- **原理步骤**

  - **1、请求进来使用文件上传解析器判断（**isMultipart**）并封装（**resolveMultipart方法，**返回**MultipartHttpServletRequest**）文件上传请求**

  - **2、参数解析器来解析请求中的文件内容封装成MultipartFile**

    ![8.9](\images\posts\springboot\8.9.jpg)

  - **3、将request中文件信息封装为一个Map；**MultiValueMap<String, MultipartFile>


**FileCopyUtils**。实现文件流的拷贝



# 8、异常处理

## 1、错误处理

#### 1、默认规则

- 默认情况下，Spring Boot提供`/error`处理所有错误的映射

- 对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，响应一个“ whitelabel”错误视图，以HTML格式呈现相同的数据

  ![9.1](\images\posts\springboot\9.1.jpg)

  ![9.2](\images\posts\springboot\9.2.jpg)

- **要对其进行自定义，添加View解析为error**

- 要完全替换默认行为，可以实现 `ErrorController `并注册该类型的Bean定义，或添加`ErrorAttributes类型的组件`以使用现有机制但替换其内容。

- error/下的4xx，5xx页面会被自动解析；

  ![9.3](\images\posts\springboot\9.3.jpg)

#### 2、定制错误处理逻辑

- 自定义错误页

  - error/404.html   error/5xx.html；有精确的错误状态码页面就匹配精确，没有就找 4xx.html；如果都没有就触发白页

- @ControllerAdvice+@ExceptionHandler处理全局异常；底层是 **ExceptionHandlerExceptionResolver 支持的**

  ```java
  @Slf4j
  @ControllerAdvice
  public class GlobalExceptionHandler {
  
      //指定能处理的异常类型
      @ExceptionHandler({ArithmeticException.class,NullPointerException.class})
      public String handleArithException(Exception e){
  
          log.error("异常是：{}",e);
          return "login"; //视图地址
      }
  
      //也可以使用ModelAndView方式
  //    @ExceptionHandler({ArithmeticException.class,NullPointerException.class})
  //    public ModelAndView handleArithException1(Exception e){
  //
  //        ModelAndView mv = new ModelAndView();
  //        mv.setViewName("login");
  //        log.error("异常是：{}",e);
  //        return mv; //视图地址
  //    }
  }
  ```

  

- @ResponseStatus+自定义异常 ；底层是 **ResponseStatusExceptionResolver ，把responsestatus注解的信息底层调用** **response.sendError(statusCode, resolvedReason)；tomcat发送的/error**

  ```java
  //自定义异常，ResponseStatus注解定义异常的状态
  @ResponseStatus(value = HttpStatus.FORBIDDEN,reason = "请求不被允许")
  public class MyException extends RuntimeException{
  
      public MyException() {
  
      }
  
      public MyException(String message) {
          super(message);
      }
  }
  
  
  
  @Controller
  public class ExceptionController {
      @ResponseBody
      @GetMapping("/ex")
      public String ex(){
          if(4>3) {
              //使用时，抛出的异常状态就是上面自定义的
              throw new MyException();
          }
          return "haha";
      }
  }
  ```

  

- Spring底层的异常，如 参数类型转换异常；**DefaultHandlerExceptionResolver 处理框架底层的异常。**

  - response.sendError(HttpServletResponse.**SC_BAD_REQUEST**, ex.getMessage()); 

- 自定义实现 HandlerExceptionResolver 处理异常；可以作为默认的全局异常处理规则

  ```java
  //优先级，数字越小优先级越高。调成最高优先级后就作为默认的全局异常处理规则了
  @Order(value= Ordered.HIGHEST_PRECEDENCE) 
  @Component
  public class CustomerHandlerExceptionResolver implements HandlerExceptionResolver {
      @Override
      public ModelAndView resolveException(HttpServletRequest request,
                                           HttpServletResponse response,
                                           Object handler, Exception ex) {
  
          try {
              response.sendError(511,"我喜欢的错误");
          } catch (IOException e) {
              e.printStackTrace();
          }
          return new ModelAndView();
      }
  }
  ```

  ![9.4](\images\posts\springboot\9.4.jpg)

- **ErrorViewResolver**  实现自定义处理异常；

  - response.sendError 。error请求就会转给controller
  - 你的异常没有任何人能处理。tomcat底层 response.sendError。error请求就会转给controller
  - **basicErrorController 要去的页面地址是** **ErrorViewResolver**  ；






#### 3、异常处理自动配置原理

- **ErrorMvcAutoConfiguration  自动配置异常处理规则**

  - **容器中的组件：类型：DefaultErrorAttributes ->** **id为errorAttributes**

  ```java
  public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver
  ```

  - **DefaultErrorAttributes**：定义错误页面中可以包含哪些数据。

    ![9.5](\images\posts\springboot\9.5.jpg)

    ![9.6](\images\posts\springboot\9.6.jpg)

  - **容器中的组件：类型：BasicErrorController --> id为basicErrorController（json+白页 适配响应）**
    - **处理默认** **/error 路径的请求；页面响应** **new** ModelAndView(**"error"**, model)；
    - **容器中有组件 View**->**id是error**；（响应默认错误页）
    - 容器中放组件 **BeanNameViewResolver（视图解析器）；按照返回的视图名作为组件的id去容器中找View对象。**

  - **容器中的组件：**类型：**DefaultErrorViewResolver -> id：**conventionErrorViewResolver
    - 如果发生错误，会以HTTP的状态码 作为视图页地址（viewName），找到真正的页面
    - error/404、5xx.html




如果想要返回页面；就会找error视图【**StaticView**】。(默认是一个白页)





写出去json

![9.8](\images\posts\springboot\9.8.jpg)

 错误页

![9.7](\images\posts\springboot\9.7.jpg)



#### 4、异常处理步骤流程

1、执行目标方法，目标方法运行期间有任何异常都会被catch、而且标志当前请求结束；并且用 **dispatchException** 

2、进入视图解析流程（页面渲染？） 

processDispatchResult(processedRequest, response, mappedHandler, **mv**, **dispatchException**);

3、**mv** = **processHandlerException**；处理handler发生的异常，处理完成返回ModelAndView；

- 1、遍历所有的 **handlerExceptionResolvers，看谁能处理当前异常【HandlerExceptionResolver处理器异常解析器】**

  ![9.9](\images\posts\springboot\9.9.jpg)

- **2、系统默认的  异常解析器；**

  ![10.1](\images\posts\springboot\10.1.jpg)

  - **1、DefaultErrorAttributes先来处理异常。把异常信息保存到rrequest域，并且返回null；**

  - **2、默认没有任何人能处理异常，所以异常会被抛出**

    - 1、如果没有任何人能处理最终底层就会发送 /error 请求。会被底层的BasicErrorController处理
    - 2、解析错误视图；遍历所有的ErrorViewResolver  看谁能解析。

    ![10.2](\images\posts\springboot\10.2.jpg)

    - 3、默认的DefaultErrorViewResolver ,作用是把响应状态码作为错误页的地址，error/500.html
    - 4、模板引擎最终响应这个页面 **error/500.html** 

# 9、Web原生组件注入（Servlet、Filter、Listener）

## 1、使用Servlet API

### Servlet

```java
@WebServlet(urlPatterns = "/my")//配置能处理的请求
public class MyServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().write("66666");
    }
}


//必须要配合@ServletComponentScan注解使用，指定原生Servlet组件都放在那里
@ServletComponentScan(basePackages = "com.atguigu.admin")
//@ServletComponentScan()不指定值的话默认就是主类所在的包
@SpringBootApplication
public class Boot05WebAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot05WebAdminApplication.class, args);
    }
}
```

### Filter

```java
@Slf4j
@WebFilter(urlPatterns={"/css/*","/images/*"}) //拦截路径
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("MyFilter初始化完成");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("MyFilter工作");
        chain.doFilter(request,response);
    }

    @Override
    public void destroy() {
        log.info("MyFilter销毁");
    }
}


//必须要配合@ServletComponentScan注解使用，指定原生Servlet组件都放在那里
@ServletComponentScan(basePackages = "com.atguigu.admin")
@SpringBootApplication
public class Boot05WebAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot05WebAdminApplication.class, args);
    }
}
```

### Listener

```java
@Slf4j
@WebListener
public class MySwervletContextListener implements ServletContextListener {


    @Override
    public void contextInitialized(ServletContextEvent sce) {
        log.info("MySwervletContextListener监听到项目初始化完成");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        log.info("MySwervletContextListener监听到项目销毁");
    }
}


//必须要配合@ServletComponentScan注解使用，指定原生Servlet组件都放在那里
@ServletComponentScan(basePackages = "com.atguigu.admin")
@SpringBootApplication
public class Boot05WebAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot05WebAdminApplication.class, args);
    }
}
```



推荐使用以上这些方式来使用servlet的原生组件





扩展：DispatchServlet 如何注册进来

- 容器中自动配置了  DispatcherServlet  属性绑定到 WebMvcProperties；对应的配置文件配置项是 **spring.mvc。**
- 通过 `ServletRegistrationBean<DispatcherServlet> `把 DispatcherServlet  配置进来。
- 默认映射的是 / 路径。

![10.3](\images\posts\springboot\10.3.jpg)

Tomcat-Servlet；

多个Servlet都能处理到同一层路径，精确优选原则，例如有两个拦截路径，

A： /my/

B： /my/1

此时发送my/1会被B拦截，发送my/2会被A拦截





## 2、使用RegistrationBean

```java
ServletRegistrationBean`, `FilterRegistrationBean`, and `ServletListenerRegistrationBean

@Configuration
// (proxyBeanMethods = true)：保证依赖的组件始终是单实例的
public class MyRegistConfig {

    @Bean
    public ServletRegistrationBean myServlet(){
        MyServlet myServlet = new MyServlet();

        return new ServletRegistrationBean(myServlet,"/my","/my02");
    }


    @Bean
    public FilterRegistrationBean myFilter(){

        MyFilter myFilter = new MyFilter();
        /**
        * 拦截上面的myServlet定义的路径。
        * 此时需要加上(proxyBeanMethods = true)：保证依赖的组件始终是单实例的
        * 虽然不加不会引起功能上的问题，但是每次都需要调用myServlet()，则会new MyServlet()会造成冗余的对象
        */
        //return new FilterRegistrationBean(myFilter,myServlet());
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter);
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/my","/css/*"));
        return filterRegistrationBean;
    }

    @Bean
    public ServletListenerRegistrationBean myListener(){
        MySwervletContextListener mySwervletContextListener = new MySwervletContextListener();
        return new ServletListenerRegistrationBean(mySwervletContextListener);
    }
}



//使用这种方式，此时我们自定义的servlet组件就不需要加相应的注解了
//@WebServlet(urlPatterns = "/my")
public class MyServlet extends HttpServlet {……}

@Slf4j
//@WebFilter(urlPatterns={"/css/*","/images/*"}) //拦截路径
public class MyFilter implements Filter {……}

@Slf4j
//@WebListener
public class MySwervletContextListener implements ServletContextListener {……}

```



# 10、嵌入式Servlet容器

## 1、切换嵌入式Servlet容器

- 默认支持的webServer

- - `Tomcat`, `Jetty`, or `Undertow`
  - `ServletWebServerApplicationContext 容器启动寻找ServletWebServerFactory 并引导创建服务器`

- 切换服务器

![10.4](\images\posts\springboot\10.4.jpg)

```xml
<!--使用undertow-->
<!--先排除tomcat的-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>   
</dependency>

<!--再引入undertow的依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```





- 原理

- - SpringBoot应用启动发现当前是Web应用。web场景包-导入tomcat
  - web应用会创建一个web版的ioc容器 `ServletWebServerApplicationContext` 
  - `ServletWebServerApplicationContext`  启动的时候寻找 **ServletWebServerFactory**`（Servlet 的web服务器工厂---生产--> Servlet 的web服务器）`  
  - SpringBoot底层默认有很多的WebServer工厂；`TomcatServletWebServerFactory`, `JettyServletWebServerFactory`, or `UndertowServletWebServerFactory`
  - `底层直接会有一个自动配置类。ServletWebServerFactoryAutoConfiguration`
  - `ServletWebServerFactoryAutoConfiguration导入了ServletWebServerFactoryConfiguration（配置类）`
  - `ServletWebServerFactoryConfiguration 配置类 根据动态判断系统中到底导入了那个Web服务器的包。（默认是web-starter导入tomcat包），容器中就有 TomcatServletWebServerFactory`
  - `TomcatServletWebServerFactory 创建出Tomcat服务器并启动；TomcatWebServer 的构造器拥有初始化方法initialize---this.tomcat.start();`
  - `内嵌服务器，就是手动把启动服务器的代码调用（tomcat核心jar包存在）`


## 2、定制Servlet容器

- 实现  **WebServerFactoryCu**stomizer<ConfigurableServletWebServerFactory> 

  - 把配置文件的值和ServletWebServerFactory 进行绑定

  **xxxxxCustomizer**：定制化器，可以改变xxxx的默认规则

  ```java
  import org.springframework.boot.web.server.WebServerFactoryCustomizer;
  import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
  import org.springframework.stereotype.Component;
  
  @Component
  public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
  
      @Override
      public void customize(ConfigurableServletWebServerFactory server) {
          server.setPort(9000);
      }
  
  }
  ```

- 修改配置文件 **server.xxx**

  ```yaml
  server:
    port: 9000
  ```

- 直接自定义 **ConfigurableServletWebServerFactory** 

  ```java
  @Bean
  public ConfigurableServletWebServerFactory webServerFactory() {
      TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
      factory.setPort(9000);
      factory.setSessionTimeout(10, TimeUnit.MINUTES);
      factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
      return factory;
  }
  ```

  



# 11、定制化原理

## 1、定制化的常见方式 

- 修改配置文件；
- **xxxxxCustomizer；**
- **编写自定义的配置类   xxxConfiguration；+** **@Bean替换、增加容器中默认组件，例如视图解析器** 
- **Web应用 编写一个配置类实现 WebMvcConfigurer 即可定制化web功能；+ @Bean给容器中再扩展一些组件**

```java
@Configuration
public class AdminWebConfig implements WebMvcConfigurer
```

- @EnableWebMvc + WebMvcConfigurer —— @Bean  可以全面接管SpringMVC，所有规则全部自己重新配置； 实现定制和扩展功能

  - 原理

  - 1、WebMvcAutoConfiguration  默认的SpringMVC的自动配置功能类。静态资源、欢迎页.....
  - 2、一旦使用 @EnableWebMvc 会 导入@Import(DelegatingWebMvcConfiguration.**class**)
  - 3、**DelegatingWebMvcConfiguration** 的 作用，只保证SpringMVC最基本的使用
    - 把所有系统中的 WebMvcConfigurer 拿过来。所有功能的定制都是这些 WebMvcConfigurer  合起来一起生效
    - 自动配置了一些非常底层的组件。**RequestMappingHandlerMapping**、这些组件依赖的组件都是从容器中获取
    - **public class** DelegatingWebMvcConfiguration **extends** **WebMvcConfigurationSupport**

  - 4、**WebMvcAutoConfiguration** 里面的配置要能生效 必须  @ConditionalOnMissingBean(**WebMvcConfigurationSupport**.**class**)
  - 5、@EnableWebMvc  导致了 **WebMvcAutoConfiguration  没有生效。**



## 2、原理分析套路

**场景starter** **- xxxxAutoConfiguration - 导入xxx组件 - 绑定xxxProperties --** **绑定配置文件项** 