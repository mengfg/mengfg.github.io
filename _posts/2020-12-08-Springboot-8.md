---
layout: post
title: Springboot笔记（八） Springboot示例程序：restful风格的增删改查
date: 2020-12-08
tags: Springboot
---

## restful风格的增删改查

本案例主要是练习之前的知识，代码耦合很大，看懂代码理解即可

### 准备工作

#### 引入静态资源

静态资源文件：https://www.lanzous.com/i7eenib

1. 将静态资源(css,img,js)添加到项目中，放到springboot默认的静态资源文件夹下
2. 将模板文件(html)放到template文件夹下

![47](\images\posts\springboot\47.jpg)

注意：如果你的静态资源明明放到了静态资源文件夹下却无法访问，请检查一下是不是在自定义的配置类上加了@EnableWebMvc注解

#### 引入相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```



### 默认访问首页

template文件加不是静态资源文件夹，默认是无法直接访问的，所以要添加视图映射

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/index").setViewName("login");
        registry.addViewController("/index.html").setViewName("login");
    }
}
此时访问 
http://localhost:8080/
http://localhost:8080/index
http://localhost:8080/index.html
都会跳转到login页面
```

### i18n国际化

#### 编写国际化配置文件，抽取页面需要显示的国际化消息

创建i18n文件夹存放配置文件，文件名格式为`基础名(login)`+`语言代码(zh)`+`国家代码(CN)`

![48](\images\posts\springboot\48.jpg)

**SpringBoot自动配置好了管理国际化资源文件的组件**

```java
@Configuration(
    proxyBeanMethods = false
)
@ConditionalOnMissingBean(
    name = {"messageSource"},
    search = SearchStrategy.CURRENT
)
@AutoConfigureOrder(-2147483648)
@Conditional({MessageSourceAutoConfiguration.ResourceBundleCondition.class})
@EnableConfigurationProperties
public class MessageSourceAutoConfiguration {
    private static final Resource[] NO_RESOURCES = new Resource[0];

    public MessageSourceAutoConfiguration() {
    }

    @Bean
    @ConfigurationProperties(
        prefix = "spring.messages"
    )
    public MessageSourceProperties messageSourceProperties() {
        return new MessageSourceProperties();
    }

    @Bean
    public MessageSource messageSource(MessageSourceProperties properties) {
        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
        if (StringUtils.hasText(properties.getBasename())) {
           /**
           * setBasenames方法设置国际化资源文件的基础名（去掉语言国家代码的）
           * properties.getBasename()通过源码追踪知道返回 messages 所以我们的配置文件可以直接放在类路径下叫messages.properties 而这里我们并没有按照默认规则命名，所以需要在配置文件中通过spring.messages.basename=i18n.login设置
           **/
            messageSource.setBasenames(
                StringUtils.commaDelimitedListToStringArray(
                StringUtils.trimAllWhitespace(properties.getBasename())
                )
            );
        }

        if (properties.getEncoding() != null) {
            messageSource.setDefaultEncoding(properties.getEncoding().name());
        }

        messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
        Duration cacheDuration = properties.getCacheDuration();
        if (cacheDuration != null) {
            messageSource.setCacheMillis(cacheDuration.toMillis());
        }

        messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
        messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
        return messageSource;
    }
```

#### 在配置文件中添加国际化文件的位置和基础名

```yaml
spring:
  messages:
    basename: i18n.login
```

如果配置文件中没有配置基础名，就在类路径下找基础名为`message`的配置文件

#### 将页面文字改为获取国际化配置，格式`#{key}`

```html
<!--login.html-->
<!--注意要先引入thymeleaf的命名空间	xmlns:th="http://www.thymeleaf.org"-->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">
    <title>Signin Template for Bootstrap</title>
    <!-- Bootstrap core CSS -->
    <link href="asserts/css/bootstrap.min.css" rel="stylesheet">
    <!-- Custom styles for this template -->
    <link href="asserts/css/signin.css" rel="stylesheet">
</head>

<body class="text-center">
<form class="form-signin" action="dashboard.html">
    <img class="mb-4" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
    <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
    <label class="sr-only">Username</label>
    <input type="text" class="form-control" th:placeholder="#{login.username}" placeholder="Username" required="" autofocus="">
    <label class="sr-only">Password</label>
    <input type="password" class="form-control" th:placeholder="#{login.password}" placeholder="Password" required="">
    <div class="checkbox mb-3">
        <label>
            <input type="checkbox" value="remember-me"> [[#{login.remember}]]
        </label>
    </div>
    <button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
    <p class="mt-5 mb-3 text-muted">© 2017-2018</p>
    <a class="btn btn-sm">中文</a>
    <a class="btn btn-sm">English</a>
</form>
</body>

</html>
```

#### 然后就可以更改浏览器语言，页面就会使用对应的国际化配置文件

![49](\images\posts\springboot\49.jpg)

#### 注意：显示的中文乱码问题

页面显示时，国际化的操作可能会出现中文乱码问题。需要按照以下设置

![50](\images\posts\springboot\50.jpg)

![51](\images\posts\springboot\51.jpg)

#### 原理

国际化Locale（区域信息对象）；

LocaleResolver（获取区域信息对象的组件）；

在springmvc配置类`WebMvcAutoConfiguration`中注册了该组件

```java
@Bean
//前提是容器中不存在这个组件.所以使用自己的对象就要配置@Bean让这个条件不成立（实现LocaleResolver 即可）
@ConditionalOnMissingBean
/**
* 如果在application.properties中有配置国际化就用配置文件的
* 没有配置就用AcceptHeaderLocaleResolver 默认request获取请求头中的
*/
@ConditionalOnProperty(
    prefix = "spring.mvc",
    name = {"locale"}
)
public LocaleResolver localeResolver() {
    if (this.mvcProperties.getLocaleResolver() == org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    } else {
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
        return localeResolver;
    }
}
```

默认的就是根据请求头带来的区域信息获取Locale进行国际化

![52](\images\posts\springboot\52.jpg)

### 点击链接切换语言

实现点击连接切换语言，而不是更改浏览器的语言

通过上面的源码，知道只要自己实现区域信息解析器，就可以覆盖默认从请求头获取的配置

#### 修改页面，点击链接携带语言参数

```html
<!--login.html-->
<a class="btn btn-sm" th:href="@{/index.html(l='zh_CN')}">中文</a>
<a class="btn btn-sm" th:href="@{/index.html(l='en_US')}">English</a>
```

#### 自己实现区域信息解析器

```java
import org.springframework.util.StringUtils;
import org.springframework.web.servlet.LocaleResolver;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Locale;

public class MyLocaleResolver implements LocaleResolver {

    @Override
    public Locale resolveLocale(HttpServletRequest httpServletRequest) {
        //获取请求参数中的语言
        String language = httpServletRequest.getParameter("l");
        //没带区域信息参数就用系统默认的
        Locale locale = Locale.getDefault();
        if (!StringUtils.isEmpty(language)) {
            //提交的参数是zh_CN （语言代码_国家代码）
            String[] s = language.split("_");

            locale = new Locale(s[0], s[1]);

        }

        return locale;
    }
    @Override
    public void setLocale(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Locale locale) {

    }
}
```

#### 在配置类中将其注册到容器中

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/index").setViewName("login");
        registry.addViewController("/index.html").setViewName("login");
    }

    //注册区域信息解析器
    @Bean
    public LocaleResolver localeResolver() {
        return new MyLocaleResolver();
    }
}
```

注意：如果没有生效，请检查`@Bean`的那个方法的名称是否为`localeResolver`

### 实现登录功能

#### 提供登录的controller

```java
@Controller
public class UserController {
    @PostMapping("/user/login")
    public String login(@RequestParam String username, @RequestParam String password, HttpSession session, Model model) {
        if (!StringUtils.isEmpty(username) && "123456".equals(password)) {

            //登录成功，把用户信息方法放在session中，防止表单重复提交，重定向到后台页面
            session.setAttribute("loginUser", username);
            return "redirect:/main.html";
        }
        //登录失败,返回到登录页面
        model.addAttribute("msg", "用户名或密码错误！");
        return "login";
    }
}
```

#### 修改表单提交地址，输入框添加name值与参数名称对应

```html
<!--login.html-->
<!--修改表单提交地址，提交方式-->
<form class="form-signin" action="dashboard.html" th:action="@{/user/login}" method="post">
    <img class="mb-4" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
    <h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
    <label class="sr-only">Username</label>
    <!--添加name值与controller参数名称对应-->
    <input type="text" name="username" class="form-control" th:placeholder="#{login.username}" placeholder="Username" autofocus="">
    <label class="sr-only">Password</label>
    <!--添加name值与controller参数名称对应-->
    <input type="password" name="password" class="form-control" th:placeholder="#{login.password}" placeholder="Password" required="">
    <div class="checkbox mb-3">
        <label>
            <input type="checkbox" value="remember-me"> [[#{login.remember}]]
        </label>
    </div>
    <button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
    <p class="mt-5 mb-3 text-muted">© 2017-2018</p>
    <a class="btn btn-sm" href="?l=zh_CN">中文</a>
    <a class="btn btn-sm" href="?l=en_US">English</a>
</form>
```

#### 由于登录失败是转发，所以页面的静态资源请求路径会不正确，使用模板引擎语法替换

```html
<!--login.html-->
CSS：
<link href="asserts/css/bootstrap.min.css" th:href="@{/asserts/css/bootstrap.min.css}" rel="stylesheet">
<link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">

图片：
<img class="mb-4" src="asserts/img/bootstrap-solid.svg" th:src="@{/asserts/img/bootstrap-solid.svg}" alt="" width="72" height="72">
```

#### 添加登录失败页面显示

```html
<!--login.html-->
<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
<!--msg存在才显示该p标签-->
<p th:text="${msg}" th:if="${not #strings.isEmpty(msg)}" style="color: red"></p>
```

#### 修改页面立即生效

```yaml
#禁用缓存
spring:
  thymeleaf:
    cache: false
```

页面修改完成以后ctrl+f9：重新编译； 

### 拦截器进行登录检查

#### 实现拦截器

```java
package com.mg.springboot_crud.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class LoginHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object loginUser = request.getSession().getAttribute("loginUser");
        if (loginUser == null) {
            //未登录，拦截，并转发到登录页面
            request.setAttribute("msg", "您还没有登录，请先登录！");
            request.getRequestDispatcher("/index").forward(request, response);
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}
```

#### 注册拦截器

```java
package com.mg.springboot_crud.config;


import com.mg.springboot_crud.interceptor.LoginHandlerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MyMvcConfig implements WebMvcConfigurer {
    /**
    * 注意：在spring2.X的版本中，只要用户自定义了拦截器，则静态资源会被拦截。
    * 但是在spring1.X的版本中，是不会拦截静态资源的。
    * 因此，在使用spring2.X时，配置拦截器之后，我们要把静态资源的路径加入到不拦截的路径之中。
    */
    //定义不拦截路径
    private static final String[] excludePaths = {"/", "/index", "/index.html", "/user/login", "/asserts/**"};

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("login");
        registry.addViewController("/index").setViewName("login");
        registry.addViewController("/index.html").setViewName("login");
    }

    @Bean
    public LocaleResolver localeResolver() {
        return new MyLocaleResolver();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //添加不拦截的路径，SpringBoot已经做好了静态资源映射，所以我们不用管
        registry.addInterceptor(new LoginHandlerInterceptor())
                .excludePathPatterns(excludePaths);
    }
}
```

### CRUD-员工列表

使用rest风格

| 实验功能                             | 请求URI | 请求方式 |
| ------------------------------------ | ------- | -------- |
| 查询所有员工                         | emps    | GET      |
| 查询某个员工(来到修改页面)           | emp/1   | GET      |
| 来到添加页面                         | emp     | GET      |
| 添加员工                             | emp     | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/1   | GET      |
| 修改员工                             | emp     | PUT      |
| 删除员工                             | emp/1   | DELETE   |

为了页面结构清晰，在template文件夹下新建emp文件夹，将list.html移动到emp文件夹下

#### DAO层代码

新建dao包，在dao包中创建如下代码

```java
//DepartmentDao
package com.mg.springboot_crud.dao;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import com.mg.springboot_crud.entities.Department;
import org.springframework.stereotype.Repository;


@Repository
public class DepartmentDao {

    private static Map<Integer, Department> departments = null;

    static{
        departments = new HashMap<Integer, Department>();

        departments.put(101, new Department(101, "D-AA"));
        departments.put(102, new Department(102, "D-BB"));
        departments.put(103, new Department(103, "D-CC"));
        departments.put(104, new Department(104, "D-DD"));
        departments.put(105, new Department(105, "D-EE"));
    }

    public Collection<Department> getDepartments(){
        return departments.values();
    }

    public Department getDepartment(Integer id){
        return departments.get(id);
    }

}
```

```java
//EmployeeDao
package com.mg.springboot_crud.dao;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import com.mg.springboot_crud.entities.Department;
import com.mg.springboot_crud.entities.Employee;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

@Repository
public class EmployeeDao {

    private static Map<Integer, Employee> employees = null;

    @Autowired
    private DepartmentDao departmentDao;

    static{
        employees = new HashMap<Integer, Employee>();

        employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1, new Department(101, "D-AA")));
        employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1, new Department(102, "D-BB")));
        employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0, new Department(103, "D-CC")));
        employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0, new Department(104, "D-DD")));
        employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1, new Department(105, "D-EE")));
    }

    private static Integer initId = 1006;

    public void save(Employee employee){
        if(employee.getId() == null){
            employee.setId(initId++);
        }

        employee.setDepartment(departmentDao.getDepartment(employee.getDepartment().getId()));
        employees.put(employee.getId(), employee);
    }

    public Collection<Employee> getAll(){
        return employees.values();
    }

    public Employee get(Integer id){
        return employees.get(id);
    }

    public void delete(Integer id){
        employees.remove(id);
    }
}
```

#### 实体类代码

新建entities包，然后创建如下代码

```java
package com.mg.springboot_crud.entities;

public class Department {

    private Integer id;
    private String departmentName;

    public Department() {
    }

    public Department(int i, String string) {
        this.id = i;
        this.departmentName = string;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getDepartmentName() {
        return departmentName;
    }

    public void setDepartmentName(String departmentName) {
        this.departmentName = departmentName;
    }

    @Override
    public String toString() {
        return "Department [id=" + id + ", departmentName=" + departmentName + "]";
    }

}
```

```java
package com.mg.springboot_crud.entities;

import java.util.Date;

public class Employee {

	private Integer id;
    private String lastName;

    private String email;
    //1 male, 0 female
    private Integer gender;
    private Department department;
    private Date birth;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Integer getGender() {
        return gender;
    }

    public void setGender(Integer gender) {
        this.gender = gender;
    }

    public Department getDepartment() {
        return department;
    }

    public void setDepartment(Department department) {
        this.department = department;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }
    public Employee(Integer id, String lastName, String email, Integer gender,
                    Department department) {
        super();
        this.id = id;
        this.lastName = lastName;
        this.email = email;
        this.gender = gender;
        this.department = department;
        this.birth = new Date();
    }

    public Employee() {
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                ", gender=" + gender +
                ", department=" + department +
                ", birth=" + birth +
                '}';
    }
	
}
```

#### 添加员工controller，实现查询员工列表的方法

```java
package com.mg.springboot_crud.controller;

import com.mg.springboot_crud.dao.EmployeeDao;
import com.mg.springboot_crud.entities.Employee;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.Collection;

@Controller
public class EmpController {

    @Autowired
    private EmployeeDao employeeDao;

    @GetMapping("/emps")
    public String emps(Model model) {
        Collection<Employee> empList = employeeDao.getAll();
        model.addAttribute("emps", empList);
        return "emp/list";
    }

}
```

#### 修改后台页面，更改左侧侧边栏，将`customer`改为`员工列表`，并修改请求路径

dashboard.html页面：

```html
<li class="nav-item">
    <a class="nav-link" th:href="@{/emps}">
        <svg .....>
            ......
        </svg>
        员工列表
    </a>
</li>
```

同样emp/list页面的左边侧边栏是和后台dashboard页面一模一样的，每个都要修改很麻烦，接下来，抽取公共片段

### thymeleaf公共页面元素抽取

#### 语法

```html
<!--公共代码片段-->
<footer th:fragment="copy">
    &copy; 2011 The Good Thymes Virtual Grocery
</footer>

<!--引用代码片段-->
<div th:insert="~{footer :: copy}"></div>
~{templatename::selector}：模板名::选择器
~{templatename::fragmentname}:模板名::片段名

<!-- 〜{...}包围是完全可选的，所以上⾯的代码 将等价于：-->
<div th:insert="footer :: copy"></div>
```

具体参考官方文档：https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#including-template-fragments

三种引入公共片段的th属性：

- `th:insert`：将公共片段整个插入到声明引入的元素中
- `th:replace`：将声明引入的元素替换为公共片段
- `th:include`：将被引入的片段的内容包含进这个标签中

```html
/*公共片段*/
<footer th:fragment="copy">
    &copy; 2011 The Good Thymes Virtual Grocery
</footer>

/*引入方式*/
<div th:insert="footer :: copy"></div>
<div th:replace="footer :: copy"></div>
<div th:include="footer :: copy"></div>

/*效果*/
insert的公共片段在div标签中：
<div>
    <footer>
        &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>

<footer>
    &copy; 2011 The Good Thymes Virtual Grocery
</footer>

<div>
    &copy; 2011 The Good Thymes Virtual Grocery
</div>
```

#### 后台页面抽取

1. 将后台主页中的顶部导航栏作为片段，在list页面引入

dashboard.html：

```html
<nav th:fragment="topbar" class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0">
    <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Company name</a>
    <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
    <ul class="navbar-nav px-3">
        <li class="nav-item text-nowrap">
            <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
        </li>
    </ul>
</nav>
```

**list.html：**

```html
<!--需要先引入thymeleaf命名空间-->
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    ……………………
<body>

 <!--引入抽取的topbar-->
 <!--模板名会使用thymeleaf的前后缀配置规则进行解析，所以直接写dashboard就可以不用写HTML的后缀-->
<div th:replace="dashboard::topbar"></div>

    <!--这段代码可以直接删掉了，用上面的引入即可
    <nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0">
        <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Company name</a>
        <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
        <ul class="navbar-nav px-3">
            <li class="nav-item text-nowrap">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
            </li>
        </ul>
    </nav>
-->
......
```

2. 使用选择器的方式 抽取左侧边栏代码

**dashboard.html：**

```html
<div class="container-fluid">
    <div class="row">
        <!--加入id属性，使用Id选择器-->
        <nav id="sidebar" class="col-md-2 d-none d-md-block bg-light sidebar" ......
```

**list.html：**

```html
<div class="container-fluid">
    <div class="row">
        <!--引入，之前代码也可以删除了-->
        <div th:replace="dashboard::#sidebar"></div>
```

#### 引入片段传递参数

实现点击当前项高亮

将`dashboard.html`中的公共代码块抽出为单独的html文件，放到commos文件夹下

在引入代码片段的时候可以传递参数，然后在sidebar代码片段模板中判断当前点击的链接

语法：

```
~{templatename::selector(变量名=值)}

/*或者在定义代码片段时，定义参数*/
<nav th:fragment="topbar(A,B)"
/*引入时直接传递参数*/
~{templatename::fragmentname(A值,B值)}
```

**topbar.html**

```html
<!doctype html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<body>

<nav th:fragment="topbar" class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0">
    <a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Company name</a>
    <input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
    <ul class="navbar-nav px-3">
        <li class="nav-item text-nowrap">
            <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
        </li>
    </ul>
</nav>
</body>
</html>
```

**sidebar.html**

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<nav id="sidebar" class="col-md-2 d-none d-md-block bg-light sidebar">
    <div class="sidebar-sticky">
        <ul class="nav flex-column">
            <li class="nav-item">
                <!--通过传递currentURI的值来判断是否高亮显示，如果传递的main.html则Dashboard高亮-->
                <a class="nav-link active" th:class="${currentURI}=='main.html'?'nav-link active':'nav-link'" th:href="@{/main.html}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                        <polyline points="9 22 9 12 15 12 15 22"></polyline>
                    </svg>
                    Dashboard <span class="sr-only">(current)</span>
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file">
                        <path d="M13 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9z"></path>
                        <polyline points="13 2 13 9 20 9"></polyline>
                    </svg>
                    Orders
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-shopping-cart">
                        <circle cx="9" cy="21" r="1"></circle>
                        <circle cx="20" cy="21" r="1"></circle>
                        <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"></path>
                    </svg>
                    Products
                </a>
            </li>
            <li class="nav-item">
                <!--通过传递currentURI的值来判断是否高亮显示,如果传递的emps则员工列表高亮-->
                <a class="nav-link" th:class="${currentURI}=='emps'?'nav-link active':'nav-link'" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#" th:href="@{/emps}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users">
                        <path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
                        <circle cx="9" cy="7" r="4"></circle>
                        <path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
                        <path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
                    </svg>
                    员工列表
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-bar-chart-2">
                        <line x1="18" y1="20" x2="18" y2="10"></line>
                        <line x1="12" y1="20" x2="12" y2="4"></line>
                        <line x1="6" y1="20" x2="6" y2="14"></line>
                    </svg>
                    Reports
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-layers">
                        <polygon points="12 2 2 7 12 12 22 7 12 2"></polygon>
                        <polyline points="2 17 12 22 22 17"></polyline>
                        <polyline points="2 12 12 17 22 12"></polyline>
                    </svg>
                    Integrations
                </a>
            </li>
        </ul>

        <h6 class="sidebar-heading d-flex justify-content-between align-items-center px-3 mt-4 mb-1 text-muted">
            <span>Saved reports</span>
            <a class="d-flex align-items-center text-muted" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-plus-circle"><circle cx="12" cy="12" r="10"></circle><line x1="12" y1="8" x2="12" y2="16"></line><line x1="8" y1="12" x2="16" y2="12"></line></svg>
            </a>
        </h6>
        <ul class="nav flex-column mb-2">
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                        <polyline points="14 2 14 8 20 8"></polyline>
                        <line x1="16" y1="13" x2="8" y2="13"></line>
                        <line x1="16" y1="17" x2="8" y2="17"></line>
                        <polyline points="10 9 9 9 8 9"></polyline>
                    </svg>
                    Current month
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                        <polyline points="14 2 14 8 20 8"></polyline>
                        <line x1="16" y1="13" x2="8" y2="13"></line>
                        <line x1="16" y1="17" x2="8" y2="17"></line>
                        <polyline points="10 9 9 9 8 9"></polyline>
                    </svg>
                    Last quarter
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                        <polyline points="14 2 14 8 20 8"></polyline>
                        <line x1="16" y1="13" x2="8" y2="13"></line>
                        <line x1="16" y1="17" x2="8" y2="17"></line>
                        <polyline points="10 9 9 9 8 9"></polyline>
                    </svg>
                    Social engagement
                </a>
            </li>
            <li class="nav-item">
                <a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-file-text">
                        <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                        <polyline points="14 2 14 8 20 8"></polyline>
                        <line x1="16" y1="13" x2="8" y2="13"></line>
                        <line x1="16" y1="17" x2="8" y2="17"></line>
                        <polyline points="10 9 9 9 8 9"></polyline>
                    </svg>
                    Year-end sale
                </a>
            </li>
        </ul>
    </div>
</nav>
</body>
</html>
```

**然后在`dashboard.html`和`list.html`中引入**

dashboard.html

```html
<body>
    <div th:replace="commons/topbar::topbar"></div>

    <div class="container-fluid">
        <div class="row">
            <div th:replace="commons/sidebar::#sidebar(currentURI='main.html')"></div>
                ………………
```

list.html

```html
<body>
    <div th:replace="commons/topbar::topbar"></div>


    <div class="container-fluid">
        <div class="row">
            <div th:replace="commons/sidebar::#sidebar(currentURI='emps')"></div>
            ………………
```

#### 显示员工数据，添加增删改按钮

```html
<main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
    <h2>
        <button class="btn btn-sm btn-success">添加员工</button>
    </h2>
    <div class="table-responsive">
        <table class="table table-striped table-sm">
            <thead>
                <tr>
                    <th>员工号</th>
                    <th>姓名</th>
                    <th>邮箱</th>
                    <th>性别</th>
                    <th>部门</th>
                    <th>生日</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody>
                <tr th:each="emp:${emps}">
                    <td th:text="${emp.id}"></td>
                    <td th:text="${emp.lastName}"></td>
                    <td th:text="${emp.email}"></td>
                    <td th:text="${emp.gender}==1?'男':'女'"></td>
                    <td th:text="${emp.department.departmentName}"></td>
                    <td th:text="${#dates.format(emp.birth,'yyyy-MM-dd')}"></td>
                    <td>
                        <button class="btn btn-sm btn-primary">修改</button>
                        <button class="btn btn-sm btn-danger">删除</button>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</main>
```

#### 员工添加

1. 创建员工添加页面`add.html`

```html
<!DOCTYPE html>
<!-- saved from url=(0052)http://getbootstrap.com/docs/4.0/examples/dashboard/ -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">

<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Dashboard Template for Bootstrap</title>
    <!-- Bootstrap core CSS -->
    <link href="asserts/css/bootstrap.min.css" rel="stylesheet">

    <!-- Custom styles for this template -->
    <link href="asserts/css/dashboard.css" rel="stylesheet">
    <style type="text/css">
        /* Chart.js */

        @-webkit-keyframes chartjs-render-animation {
            from {
                opacity: 0.99
            }
            to {
                opacity: 1
            }
        }

        @keyframes chartjs-render-animation {
            from {
                opacity: 0.99
            }
            to {
                opacity: 1
            }
        }

        .chartjs-render-monitor {
            -webkit-animation: chartjs-render-animation 0.001s;
            animation: chartjs-render-animation 0.001s;
        }
    </style>
</head>

<body>
<div th:replace="commons/topbar::topbar"></div>

<div class="container-fluid">
    <div class="row">
        <div th:replace="commons/sidebar::#sidebar(currentURI='emps')"></div>
        <main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
            <form>
                <div class="form-group">
                    <label>LastName</label>
                    <input name="lastName" type="text" class="form-control" placeholder="zhangsan">
                </div>
                <div class="form-group">
                    <label>Email</label>
                    <input  name="email" type="email" class="form-control" placeholder="zhangsan@atguigu.com">
                </div>
                <div class="form-group">
                    <label>Gender</label><br/>
                    <div class="form-check form-check-inline">
                        <input class="form-check-input" type="radio" name="gender" value="1">
                        <label class="form-check-label">男</label>
                    </div>
                    <div class="form-check form-check-inline">
                        <input class="form-check-input" type="radio" name="gender" value="0">
                        <label class="form-check-label">女</label>
                    </div>
                </div>
                <div class="form-group">
                    <label>department</label>
                    <select class="form-control">
                        <option>1</option>
                        <option>2</option>
                        <option>3</option>
                        <option>4</option>
                        <option>5</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>Birth</label>
                    <input name="birth" type="text" class="form-control" placeholder="zhangsan">
                </div>
                <button type="submit" class="btn btn-primary">添加</button>
            </form>
        </main>
    </div>
</div>

<!-- Bootstrap core JavaScript
================================================== -->
<!-- Placed at the end of the document so the pages load faster -->
<script type="text/javascript" src="asserts/js/jquery-3.2.1.slim.min.js"></script>
<script type="text/javascript" src="asserts/js/popper.min.js"></script>
<script type="text/javascript" src="asserts/js/bootstrap.min.js"></script>

<!-- Icons -->
<script type="text/javascript" src="asserts/js/feather.min.js"></script>
<script>
    feather.replace()
</script>

<!-- Graphs -->
<script type="text/javascript" src="asserts/js/Chart.min.js"></script>
<script>
    var ctx = document.getElementById("myChart");
    var myChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
            datasets: [{
                data: [15339, 21345, 18483, 24003, 23489, 24092, 12034],
                lineTension: 0,
                backgroundColor: 'transparent',
                borderColor: '#007bff',
                borderWidth: 4,
                pointBackgroundColor: '#007bff'
            }]
        },
        options: {
            scales: {
                yAxes: [{
                    ticks: {
                        beginAtZero: false
                    }
                }]
            },
            legend: {
                display: false,
            }
        }
    });
</script>

</body>

</html>
```

2. 修改`list.html`点击链接跳转到添加页面

```html
<a href="/emp" th:href="@{/emp}" class="btn btn-sm btn-success">添加员工</a>
```

3. `EmpController`添加映射方法

```java
@Autowired
    private DepartmentDao departmentDao;

    @GetMapping("/emp")
    public String toAddPage(Model model) {
        //准备部门下拉框数据
        Collection<Department> departments = departmentDao.getDepartments();
        model.addAttribute("departments",departments);
        return "emp/add";
    }
```

4. 修改`add.html`页面遍历添加下拉选项

```html
<select class="form-control" name="department.id">
    <option th:each="dept:${departments}" 
            th:text="${dept.departmentName}" 
            th:value="${dept.id}">		
    </option>
</select>
```

5. 表单提交，添加员工

```html
修改add.html
<form th:action="@{/emp}" method="post">
```

```java
@PostMapping("/emp")
public String add(Employee employee) {
    System.out.println(employee);
    //模拟添加到数据库
    employeeDao.save(employee);
    //添加成功重定向到列表页面
    return "redirect:/emps";
}
```

#### 日期格式修改

表单提交的日期格式必须是`dd/MM/yyyy`的格式，可以在配置文件中修改格式

```yaml
spring:
  mvc:
    format:
      date: yyyy-MM-dd
```

#### 员工修改

1. 点击按钮跳转到编辑页面

```html
<!--修改list.html页面-->
<a th:href="@{/emp/}+${emp.id}" class="btn btn-sm btn-primary">修改</a>
```

2. 添加编辑edit.html页面，表单的提交要为post方式，提供`_method`参数

```html
<!DOCTYPE html>
<!-- saved from url=(0052)http://getbootstrap.com/docs/4.0/examples/dashboard/ -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">

<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="description" content="">
    <meta name="author" content="">

    <title>Dashboard Template for Bootstrap</title>
    <!-- Bootstrap core CSS -->
    <link href="asserts/css/bootstrap.min.css" th:href="@{/asserts/css/bootstrap.min.css}" rel="stylesheet">

    <!-- Custom styles for this template -->
    <link href="asserts/css/dashboard.css" th:href="@{/asserts/css/dashboard.css}" rel="stylesheet">
    <style type="text/css">
        /* Chart.js */

        @-webkit-keyframes chartjs-render-animation {
            from {
                opacity: 0.99
            }
            to {
                opacity: 1
            }
        }

        @keyframes chartjs-render-animation {
            from {
                opacity: 0.99
            }
            to {
                opacity: 1
            }
        }

        .chartjs-render-monitor {
            -webkit-animation: chartjs-render-animation 0.001s;
            animation: chartjs-render-animation 0.001s;
        }
    </style>
</head>

<body>
<div th:replace="commons/topbar::topbar"></div>

<div class="container-fluid">
    <div class="row">
        <div th:replace="commons/sidebar::#sidebar(currentURI='emps')"></div>
        <main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
            <form th:action="@{/emp}" method="post">
                <!--员工id-->
                <input type="hidden" name="id" th:value="${emp.id}">
                <!--http请求方式-->
                <input type="hidden" name="_method" value="put">
                <div class="form-group">
                    <label>LastName</label>
                    <input name="lastName" th:value="${emp.lastName}" type="text" class="form-control" placeholder="zhangsan">
                </div>
                <div class="form-group">
                    <label>Email</label>
                    <input  name="email" th:value="${emp.email}" type="email" class="form-control" placeholder="zhangsan@atguigu.com">
                </div>
                <div class="form-group">
                    <label>Gender</label><br/>
                    <div class="form-check form-check-inline">
                        <input class="form-check-input" type="radio" name="gender" value="1" th:checked="${emp.gender==1}">
                        <label class="form-check-label">男</label>
                    </div>
                    <div class="form-check form-check-inline">
                        <input class="form-check-input" type="radio" name="gender" value="0" th:checked="${emp.gender==0}">
                        <label class="form-check-label">女</label>
                    </div>
                </div>
                <div class="form-group">
                    <label>department</label>
                    <select name="department.id" class="form-control">
                        <option th:each="dept:${departments}" th:value="${dept.id}" th:selected="${dept.id}==${emp.department.id}" th:text="${dept.departmentName}"></option>
                    </select>
                </div>
                <div class="form-group">
                    <label>Birth</label>
                    <input name="birth" type="text" class="form-control" placeholder="zhangsan" th:value="${#dates.format(emp.birth,'yyyy-MM-dd')}">
                </div>
                <button type="submit" class="btn btn-primary">添加</button>
            </form>
        </main>
    </div>
</div>


<!-- Bootstrap core JavaScript
================================================== -->
<!-- Placed at the end of the document so the pages load faster -->
<script type="text/javascript" src="asserts/js/jquery-3.2.1.slim.min.js"></script>
<script type="text/javascript" src="asserts/js/popper.min.js"></script>
<script type="text/javascript" src="asserts/js/bootstrap.min.js"></script>

<!-- Icons -->
<script type="text/javascript" src="asserts/js/feather.min.js"></script>
<script>
    feather.replace()
</script>

<!-- Graphs -->
<script type="text/javascript" src="asserts/js/Chart.min.js"></script>
<script>
    var ctx = document.getElementById("myChart");
    var myChart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
            datasets: [{
                data: [15339, 21345, 18483, 24003, 23489, 24092, 12034],
                lineTension: 0,
                backgroundColor: 'transparent',
                borderColor: '#007bff',
                borderWidth: 4,
                pointBackgroundColor: '#007bff'
            }]
        },
        options: {
            scales: {
                yAxes: [{
                    ticks: {
                        beginAtZero: false
                    }
                }]
            },
            legend: {
                display: false,
            }
        }
    });
</script>

</body>

</html>
```

3. Controller转发到编辑页面，回显员工信息

```java
@GetMapping("/emp/{id}")
public String toEditPage(@PathVariable Integer id, Model model) {
    Employee employee = employeeDao.get(id);
    //准备部门下拉框数据
    Collection<Department> departments = departmentDao.getDepartments();
    model.addAttribute("emp", employee).addAttribute("departments", departments);
    return "emp/edit";
}
```

4. 提交表单修改员工信息

```java
@PutMapping("/emp")
public String update(Employee employee) {
    employeeDao.save(employee);
    return "redirect:/emps";
}
```

#### 员工删除

1. 修改list页面

```html
<!DOCTYPE html>
<!-- saved from url=(0052)http://getbootstrap.com/docs/4.0/examples/dashboard/ -->
<html lang="en" xmlns:th="http://www.thymeleaf.org">

    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
        <meta name="description" content="">
        <meta name="author" content="">

        <title>Dashboard Template for Bootstrap</title>
        <!-- Bootstrap core CSS -->
        <link href="asserts/css/bootstrap.min.css" rel="stylesheet">

        <!-- Custom styles for this template -->
        <link href="asserts/css/dashboard.css" rel="stylesheet">
        <style type="text/css">
            /* Chart.js */

            @-webkit-keyframes chartjs-render-animation {
                from {
                    opacity: 0.99
                }
                to {
                    opacity: 1
                }
            }

            @keyframes chartjs-render-animation {
                from {
                    opacity: 0.99
                }
                to {
                    opacity: 1
                }
            }

            .chartjs-render-monitor {
                -webkit-animation: chartjs-render-animation 0.001s;
                animation: chartjs-render-animation 0.001s;
            }
        </style>
    </head>

    <body>
        <div th:replace="commons/topbar::topbar"></div>

        <div class="container-fluid">
            <div class="row">
                <div th:replace="commons/sidebar::#sidebar(currentURI='emps')"></div>

                <main role="main" class="col-md-9 ml-sm-auto col-lg-10 pt-3 px-4">
                    <h2>
                        <a href="/emp" th:href="@{/emp}" class="btn btn-sm btn-success">添加员工</a>
                    </h2>
                    <div class="table-responsive">
                        <table class="table table-striped table-sm">
                            <thead>
                                <tr>
                                    <th>员工号</th>
                                    <th>姓名</th>
                                    <th>邮箱</th>
                                    <th>性别</th>
                                    <th>部门</th>
                                    <th>生日</th>
                                    <th>操作</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr th:each="emp:${emps}">
                                    <td th:text="${emp.id}"></td>
                                    <td th:text="${emp.lastName}"></td>
                                    <td th:text="${emp.email}"></td>
                                    <td th:text="${emp.gender}==1?'男':'女'"></td>
                                    <td th:text="${emp.department.departmentName}"></td>
                                    <td th:text="${#dates.format(emp.birth,'yyyy-MM-dd')}"></td>
                                    <td>
                                        <a th:href="@{/emp/}+${emp.id}" class="btn btn-sm btn-primary">修改</a>
										<!--删除按钮，并添加属性-->
                                        <button th:attr="del_uri=@{/emp/}+${emp.id}" class="btn btn-sm btn-danger deleteBtn">删除</button>

                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </main>
                <!--删除按钮的form表单-->
                <form id="deleteEmpForm" method="post">
                    <input type="hidden" name="_method" value="delete">
                </form>
            </div>
        </div>



        <!-- Bootstrap core JavaScript
================================================== -->
        <!-- Placed at the end of the document so the pages load faster -->
        <script type="text/javascript" src="asserts/js/jquery-3.2.1.slim.min.js"></script>
        <script type="text/javascript" src="asserts/js/popper.min.js"></script>
        <script type="text/javascript" src="asserts/js/bootstrap.min.js"></script>

        <!-- Icons -->
        <script type="text/javascript" src="asserts/js/feather.min.js"></script>
        <script>
            feather.replace()
        </script>

        <!--使用js提交删除-->
        <script>
            $(".deleteBtn").click(function(){
                $("#deleteEmpForm").attr("action",$(this).attr("del_uri")).submit();
                return false;
            });
        </script>

        <!-- Graphs -->
        <script type="text/javascript" src="asserts/js/Chart.min.js"></script>
        <script>
            var ctx = document.getElementById("myChart");
            var myChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"],
                    datasets: [{
                        data: [15339, 21345, 18483, 24003, 23489, 24092, 12034],
                        lineTension: 0,
                        backgroundColor: 'transparent',
                        borderColor: '#007bff',
                        borderWidth: 4,
                        pointBackgroundColor: '#007bff'
                    }]
                },
                options: {
                    scales: {
                        yAxes: [{
                            ticks: {
                                beginAtZero: false
                            }
                        }]
                    },
                    legend: {
                        display: false,
                    }
                }
            });
        </script>

    </body>

</html>

```

2. controller层添加方法

```java
@DeleteMapping("/emp/{id}")
public String delete(@PathVariable String id){
    employeeDao.delete(Integer.parseInt(id));
    return "redirect:/emps";
}
```

注意：如果提示不支持POST请求，在确保代码无误的情况下查看是否配置启动`HiddenHttpMethodFilter`，需要在配置文件中配置，2.X版本默认是false，1.X版本默认是true

```yaml
spring:
    hiddenmethod:
      filter:
        enabled: true
```

