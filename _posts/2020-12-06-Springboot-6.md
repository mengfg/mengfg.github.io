---
layout: post
title: Springboot笔记（六） SpringbootWeb开发之模板引擎
date: 2020-12-06
tags: Springboot
---

## 模板引擎

常见的模板引擎有`JSP`、`Velocity`、`Freemarker`、`Thymeleaf`

![36](\images\posts\springboot\36.jpg)

SpringBoot推荐使用Thymeleaf；

## Thymeleaf

### 引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

**如需切换thymeleaf版本：**pom文件中加入

```xml
<properties>
    <!--thymeleaf主程序-->
    <thymeleaf.version>X.X.X.RELEASE</thymeleaf.version>

    <!-- 布局功能的支持程序-->
  	<!-- 要注意的是：thymeleaf3版本主程序，需要layout2以上版本，thymeleaf2版本主程序需要layout1版本-->
    <thymeleaf-layout-dialect.version>XXX</thymeleaf-layout-dialect.version>

</properties>
```

### Thymeleaf使用

![35](\images\posts\springboot\35.jpg)

通过源码发现， 默认只要我们把**HTML页面**放在`classpath:/templates/`，后缀是.html，thymeleaf就能自动渲染；当然这些属性我们都可以通过application.properties来修改

#### 示例

1. 创建模板文件`hello.html`，并导入thymeleaf的名称空间

```html
<!--使用此方法导入命名空间<html lang="en" xmlns:th="http://www.thymeleaf.org">-->

<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>

    </body>
</html>
```

2. 使用模板

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <!--行内写法-->
        <title>[[${title}]]</title>
    </head>
    <body>
        <h1 th:text="${title}"></h1>
        <!--div标签中的文本将会被取出来的值覆盖掉-->
        <div th:text="${info}">这里的文本之后将会被覆盖</div>
    </body>
</html>
```

3. 在controller中准备数据

```java
@Controller
public class HelloT {

    @RequestMapping("/ht")
    public String ht(Model model) {
        model.addAttribute("title","hello Thymeleaf")
            .addAttribute("info","this is first thymeleaf test");
        return "t1";
    }
}
```

### Thymeleaf语法

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

![37](\images\posts\springboot\37.jpg)

**th:utext** 进行文本替换 会解析html：

```html
<p th:utext="utext标签： + ${msg}"></p>
```

![38](\images\posts\springboot\38.jpg)

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

![39](\images\posts\springboot\39.jpg)

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

##### 	th:style

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

##### 	Elvis运算符

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