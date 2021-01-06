---
layout: post
title: SpringCloud笔记（一） SpringCloud概述
date: 2020-12-26
tags: SpringCloud
---

## 微服务

### 什么是微服务

业界大牛马丁.福勒（Martin Fowler） 这样描述微服务：
论文网址：            https://martinfowler.com/articles/microservices.html


就目前而言，对于微服务业界并没有一个统一的、标准的定义（While there is no precise definition of this architectural style）

但通常而言， 微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行在其独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。服务之间采用轻量级的通信机制互相沟通（通常是基于HTTP的RESTful API）。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务，可以使用不同的语言来编写服务，也可以使用不同的数据存储。

总结来说：

微服务化的核心就是将传统的一站式应用，根据业务拆分成一个一个的服务，彻底地去耦合,每一个微服务提供单个业务功能的服务，一个服务做一件事，从技术角度看就是一种小而独立的处理过程，类似进程概念，能够自行单独启动或销毁，拥有自己独立的数据库。

 ### 微服务与微服务架构

#### 微服务
强调的是服务的大小，它关注的是某一个点，是具体解决某一个问题/提供落地对应服务的一个服务应用,
狭意的看,可以看作Eclipse里面的一个个微服务工程/或者Module

#### 微服务架构

微服务架构是⼀种架构模式，它提倡将单⼀应⽤程序划分成⼀组⼩的服务，服务之间互相协调、互相配合，为⽤户提供最终价值。每个服务运⾏在其独⽴的进程中，服务与服务间采⽤轻量级的通信机制互相协作（通常是基于HTTP协议的RESTful API）。每个服务都围绕着具体业务进⾏构建，并且能够被独⽴的部署到⽣产环境、类⽣产环境等。另外，应当尽量避免统⼀的、集中式的服务管理机制，对具体的⼀个服务⽽⾔，应根据业务上下⽂，选择合适的语⾔、⼯具对其进⾏构建。

### 微服务优缺点

#### 优点

1. 每个服务足够内聚，足够小，代码容易理解这样能聚焦一个指定的业务功能或业务需求
2. 开发简单、开发效率提高，一个服务可能就是专一的只干一件事。
3. 微服务能够被小团队单独开发，这个小团队是2到5人的开发人员组成。
4. 微服务是松耦合的，是有功能意义的服务，无论是在开发阶段或部署阶段都是独立的。
5. 微服务能使用不同的语言开发。
6. 易于和第三方集成，微服务允许容易且灵活的方式集成自动部署，通过持续集成工具，如Jenkins, Hudson, bamboo 。
7. 微服务易于被一个开发人员理解，修改和维护，这样小团队能够更关注自己的工作成果。无需通过合作才能体现价值。
8. 微服务允许你利用融合最新技术。
9. 微服务只是业务逻辑的代码，不会和HTML,CSS 或其他界面组件混合。
10. 每个微服务都有自己的存储能力，可以有自己的数据库。也可以有统一数据库。

#### 缺点

1. 开发人员要处理分布式系统的复杂性
2. 多服务运维难度，随着服务的增加，运维的压力也在增大
3. 系统部署依赖
4. 服务间通信成本
5. 数据一致性
6. 系统集成测试
7. 性能监控……

 ### 微服务技术栈

 多种技术的集合体：

![1](\images\posts\springcloud\1.png)



## SpringCloud

### 入门概述

**SpringCloud=分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合体，俗称微服务全家桶**

SpringCloud，基于SpringBoot提供了一套微服务解决方案，包括服务注册与发现，配置中心，全链路监控，服务网关，负载均衡，熔断器等组件，除了基于NetFlix的开源组件做高度抽象封装之外，还有一些选型中立的开源组件。

SpringCloud利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，SpringCloud为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等,它们都可以用SpringBoot的开发风格做到一键启动和部署。

SpringBoot并没有重复制造轮子，它只是将目前各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过SpringBoot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包

 

 ### Springboot和SpringCloud的关系

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，
为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务

SpringBoot可以离开SpringCloud独立使用开发项目，但是SpringCloud离不开SpringBoot，属于依赖的关系.

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

 

 ### SpringCloud的版本关系

SpringCloud采用了英国伦敦地铁站的名称来命名，并由地铁站名称字母A-Z依次类推的形式来发布迭代版本。

SpringCloud是一个由许多子项目组成的综合项目，各子项目有不同的发布节奏。为了管理SpringCloud与各子项目的版本依赖关系，发布了一个清单，其中包括了某个SpringCloud版本对应的子项目版本。为了避免SpringCloud版本号与子项目版本号混淆，**SpringCloud版本采用了名称而非版本号的命名，这些版本的名字采用了伦敦地铁站的名字，根据字母表的顺序来对应版本时间顺序。**例如Angel是第一个版本, Brixton是第二个版本.当SpringCloud的发布内容积累到临界点或者一个重大BUG被解决后，会发布一个'service releases"版本，简称SRX版本，比如Greenwich.SR2就是SpringCloud发布的Greenwich版本的第2个SRX版本。

**从 Spring Cloud 2020.0.0-M1 开始，Spring Cloud 废除了这种英国伦敦地铁站的命名方式，而使用了全新的 "日历化" 版本命名方式。**

 

 ### SpringCloud和Springboot之间的依赖关系如何看

官网：https://spring.io/projects/spring-cloud

依赖关系：

![2](\images\posts\springcloud\2.jpg)

 更详细的版本对应查看方法：

https://start.spring.io/actuator/info会得到一个json串，格式化后

```json
{
	"git": {
		"branch": "2d1145cf248875e601bdc1568a7fa1961506cb54",
		"commit": {
			"id": "2d1145c",
			"time": "2021-01-05T13:26:05Z"
		}
	},
	"build": {
		"version": "0.0.1-SNAPSHOT",
		"artifact": "start-site",
		"versions": {
			"spring-boot": "2.4.1",
			"initializr": "0.10.0-SNAPSHOT"
		},
		"name": "start.spring.io website",
		"time": "2021-01-05T13:28:26.513Z",
		"group": "io.spring.start"
	},
	"bom-ranges": {
		"azure": {
			"2.0.10": "Spring Boot >=2.0.0.RELEASE and <2.1.0.RELEASE",
			"2.1.10": "Spring Boot >=2.1.0.RELEASE and <2.2.0.M1",
			"2.2.4": "Spring Boot >=2.2.0.M1 and <2.3.0.M1",
			"2.3.5": "Spring Boot >=2.3.0.M1"
		},
		"codecentric-spring-boot-admin": {
			"2.0.6": "Spring Boot >=2.0.0.M1 and <2.1.0.M1",
			"2.1.6": "Spring Boot >=2.1.0.M1 and <2.2.0.M1",
			"2.2.4": "Spring Boot >=2.2.0.M1 and <2.3.0.M1",
			"2.3.1": "Spring Boot >=2.3.0.M1 and <2.5.0-M1"
		},
		"solace-spring-boot": {
			"1.0.0": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1",
			"1.1.0": "Spring Boot >=2.3.0.M1 and <2.5.0-M1"
		},
		"solace-spring-cloud": {
			"1.0.0": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1",
			"1.1.1": "Spring Boot >=2.3.0.M1 and <2.5.0-M1"
		},
        ======================springcloud和springboot的版本对照====================
		"spring-cloud": {
			"Finchley.M2": "Spring Boot >=2.0.0.M3 and <2.0.0.M5",
			"Finchley.M3": "Spring Boot >=2.0.0.M5 and <=2.0.0.M5",
			"Finchley.M4": "Spring Boot >=2.0.0.M6 and <=2.0.0.M6",
			"Finchley.M5": "Spring Boot >=2.0.0.M7 and <=2.0.0.M7",
			"Finchley.M6": "Spring Boot >=2.0.0.RC1 and <=2.0.0.RC1",
			"Finchley.M7": "Spring Boot >=2.0.0.RC2 and <=2.0.0.RC2",
			"Finchley.M9": "Spring Boot >=2.0.0.RELEASE and <=2.0.0.RELEASE",
			"Finchley.RC1": "Spring Boot >=2.0.1.RELEASE and <2.0.2.RELEASE",
			"Finchley.RC2": "Spring Boot >=2.0.2.RELEASE and <2.0.3.RELEASE",
			"Finchley.SR4": "Spring Boot >=2.0.3.RELEASE and <2.0.999.BUILD-SNAPSHOT",
			"Finchley.BUILD-SNAPSHOT": "Spring Boot >=2.0.999.BUILD-SNAPSHOT and <2.1.0.M3",
			"Greenwich.M1": "Spring Boot >=2.1.0.M3 and <2.1.0.RELEASE",
			"Greenwich.SR6": "Spring Boot >=2.1.0.RELEASE and <2.1.999.BUILD-SNAPSHOT",
			"Greenwich.BUILD-SNAPSHOT": "Spring Boot >=2.1.999.BUILD-SNAPSHOT and <2.2.0.M4",
			"Hoxton.SR9": "Spring Boot >=2.2.0.M4 and <2.3.8.BUILD-SNAPSHOT",
			"Hoxton.BUILD-SNAPSHOT": "Spring Boot >=2.3.8.BUILD-SNAPSHOT and <2.4.0.M1",
			"2020.0.0-M3": "Spring Boot >=2.4.0.M1 and <=2.4.0.M1",
			"2020.0.0-M4": "Spring Boot >=2.4.0.M2 and <=2.4.0-M3",
			"2020.0.0": "Spring Boot >=2.4.0.M4 and <2.4.2-SNAPSHOT",
			"2020.0.1-SNAPSHOT": "Spring Boot >=2.4.2-SNAPSHOT"
		},
		"spring-cloud-alibaba": {
			"2.2.1.RELEASE": "Spring Boot >=2.2.0.RELEASE and <2.3.0.M1"
		},
		"spring-cloud-gcp": {
			"2.0.0-RC2": "Spring Boot >=2.4.0-M1 and <2.5.0-M1"
		},
		"spring-cloud-services": {
			"2.0.3.RELEASE": "Spring Boot >=2.0.0.RELEASE and <2.1.0.RELEASE",
			"2.1.8.RELEASE": "Spring Boot >=2.1.0.RELEASE and <2.2.0.RELEASE",
			"2.2.6.RELEASE": "Spring Boot >=2.2.0.RELEASE and <2.3.0.RELEASE",
			"2.3.0.RELEASE": "Spring Boot >=2.3.0.RELEASE and <2.4.0-M1"
		},
		"spring-geode": {
			"1.2.12.RELEASE": "Spring Boot >=2.2.0.M5 and <2.3.0.M1",
			"1.3.7.RELEASE": "Spring Boot >=2.3.0.M1 and <2.4.0-M1",
			"1.4.0": "Spring Boot >=2.4.0-M1"
		},
		"spring-statemachine": {
			"2.0.0.M4": "Spring Boot >=2.0.0.RC1 and <=2.0.0.RC1",
			"2.0.0.M5": "Spring Boot >=2.0.0.RC2 and <=2.0.0.RC2",
			"2.0.1.RELEASE": "Spring Boot >=2.0.0.RELEASE"
		},
		"vaadin": {
			"10.0.17": "Spring Boot >=2.0.0.M1 and <2.1.0.M1",
			"14.4.5": "Spring Boot >=2.1.0.M1 and <2.5.0-M1"
		},
		"wavefront": {
			"2.0.2": "Spring Boot >=2.1.0.RELEASE and <2.4.0-M1",
			"2.1.0-RC1": "Spring Boot >=2.4.0-M1 and <2.4.2-SNAPSHOT",
			"2.1.0-SNAPSHOT": "Spring Boot >=2.4.2-SNAPSHOT"
		}
	},
	"dependency-ranges": {
		"okta": {
			"1.2.1": "Spring Boot >=2.1.2.RELEASE and <2.2.0.M1",
			"1.4.0": "Spring Boot >=2.2.0.M1 and <2.4.0-M1",
			"1.5.1": "Spring Boot >=2.4.0-M1 and <2.4.1",
			"2.0.0": "Spring Boot >=2.4.1 and <2.5.0-M1"
		},
		"mybatis": {
			"2.0.1": "Spring Boot >=2.0.0.RELEASE and <2.1.0.RELEASE",
			"2.1.4": "Spring Boot >=2.1.0.RELEASE and <2.5.0-M1"
		},
		"camel": {
			"2.22.4": "Spring Boot >=2.0.0.M1 and <2.1.0.M1",
			"2.25.2": "Spring Boot >=2.1.0.M1 and <2.2.0.M1",
			"3.3.0": "Spring Boot >=2.2.0.M1 and <2.3.0.M1",
			"3.5.0": "Spring Boot >=2.3.0.M1 and <2.4.0-M1",
			"3.7.0": "Spring Boot >=2.4.0.M1 and <2.5.0-M1"
		},
		"open-service-broker": {
			"2.1.3.RELEASE": "Spring Boot >=2.0.0.RELEASE and <2.1.0.M1",
			"3.0.4.RELEASE": "Spring Boot >=2.1.0.M1 and <2.2.0.M1",
			"3.1.1.RELEASE": "Spring Boot >=2.2.0.M1 and <2.4.0-M1"
		}
	}
}
```

打开SpringCloud相关版本文档：已Hoxton.SR9为例

 https://docs.spring.io/spring-cloud/docs/Hoxton.SR9/reference/html/

会提示我们版本的选择：

![3](\images\posts\springcloud\3.jpg)



## 此次笔记的使用版本

cloud：**Hoxton.SR1**

boot：**2.2.2.RELEASE**

cloud alibaba：2.1.0.RELEASE

java：JAVA8

maven：3.5及以上

mysql：5.7及以上

## 关于Cloud各种组件的停更/升级/替换

![4](\images\posts\springcloud\4.png)

SpringCloud官网：https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/

SpringCloud中文文档：https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md

Springboot官网：https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/	

