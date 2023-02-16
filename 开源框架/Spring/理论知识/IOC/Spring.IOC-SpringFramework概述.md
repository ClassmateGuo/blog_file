# SpringFramework概述

## 引用[官网](https://spring.io/projects/spring-framework#support)的描述:
### 概述

> The Spring Framework provides a comprehensive programming and configuration model for modern Java-based enterprise applications - on any kind of deployment platform.
>
> A key element of Spring is infrastructural support at the application level: Spring focuses on the "plumbing" of enterprise applications so that teams can focus on application-level business logic, without unnecessary ties to specific deployment environments.

翻译

> Spring 框架为任何类型的部署平台上基于Java的现代企业应用程序提供了一个全面的编程和配置模型。
>
> Spring的一个关键要素是应用程序级别的基础设施支持：Spring专注于企业应用程序的“管道”，以便团队可以专注于应用程序级业务逻辑，而无需与特定的部署环境建立不必要的联系。

提取重要因素

- 任何类型的部署平台:无论是操作系统,还是Web容器.都是可以部署基于SpringFramework的应用
- 现代企业应用程序:包含JavaSE和JavaEE在内,被称为一站式解决方案
- 全面的编程和配置模型:基于框架编辑,以及基于框架进行功能和组件的配置
- 应用程序级别的基础设施支持:SpringFramework 不包含任何业务功能,只是一个底层的应用抽象支撑
- 脚手架:使用它可以更快速的构建应用



###  Features(特征)

- [Core technologies](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html): dependency injection, events, resources, i18n, validation, data binding, type conversion, SpEL, AOP.
  - 核心技术:依赖注入，事件，资源，i18n，验证，数据绑定，类型转换，SpEL，AOP.


- [Testing](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html): mock objects, TestContext framework, Spring MVC Test, `WebTestClient`.
  - 测试:模拟对象，TestContext框架，Spring MVC测试，'WebTestClient'.
- [Data Access](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html): transactions, DAO support, JDBC, ORM, Marshalling XML.
  - 数据访问:事务，DAO支持，JDBC，ORM，编组XML
- [Spring MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html) and [Spring WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) web frameworks.
  - Spring MVC和Spring WebFlux web框架.
- [Integration](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html): remoting, JMS, JCA, JMX, email, tasks, scheduling, cache.
  - 集成:远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存.
- [Languages](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/languages.html): Kotlin, Groovy, dynamic languages.
  - 语言:Kotlin, Groovy, dynamic languages.



## Spring 的狭义和广义

### 广义的 Spring：Spring 技术栈

广义上的 Spring 泛指以 Spring Framework 为核心的 Spring 技术栈。

经过多年的发展，Spring 已经不再是一个单纯的应用框架，而是逐渐发展成为一个由多个不同子项目（模块）组成的成熟技术，例如 Spring Framework、Spring MVC、SpringBoot、Spring Cloud、Spring Data、Spring Security 等，其中 Spring Framework 是其他子项目的基础。

这些子项目涵盖了从企业级应用开发到云计算等各方面的内容，能够帮助开发人员解决软件发展过程中不断产生的各种实际问题，给开发人员带来了更好的开发体验。

| 项目名称        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| Spring Data     | Spring 提供的数据访问模块，对 JDBC 和 ORM 提供了很好的支持。通过它，开发人员可以使用一种相对统一的方式，来访问位于不同类型数据库中的数据。 |
| Spring Batch    | 一款专门针对企业级系统中的日常批处理任务的轻量级框架，能够帮助开发人员方便的开发出健壮、高效的批处理应用程序。 |
| Spring Security | 前身为 Acegi，是 Spring 中较成熟的子模块之一。它是一款可以定制化的身份验证和访问控制框架。 |
| Spring Mobile   | 是对 Spring MVC 的扩展，用来简化移动端 Web 应用的开发。      |
| Spring Boot     | 是 Spring 团队提供的全新框架，它为 Spring 以及第三方库一些开箱即用的配置，可以简化 Spring 应用的搭建及开发过程。 |
| Spring Cloud    | 一款基于 Spring Boot 实现的微服务框架。它并不是某一门技术，而是一系列微服务解决方案或框架的有序集合。它将市面上成熟的、经过验证的微服务框架整合起来，并通过 Spring Boot 的思想进行再封装，屏蔽调其中复杂的配置和实现原理，最终为开发人员提供了一套简单易懂、易部署和易维护的分布式系统开发工具包。 |

### 狭义的 Spring：Spring Framework

狭义的 Spring 特指 Spring Framework，通常我们将它称为 Spring 框架。

Spring 框架是一个分层的、面向切面的 Java 应用程序的一站式轻量级解决方案，它是 Spring 技术栈的核心和基础，是为了解决企业级应用开发的复杂性而创建的。

Spring 有两个核心部分： IOC 和 AOP。

| 核心 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| IOC  | Inverse of Control 的简写，译为“控制反转”，指把创建对象过程交给 Spring 进行管理。 |
| AOP  | Aspect Oriented Programming 的简写，译为“面向切面编程”。  AOP 用来封装多个类的公共行为，将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，减少系统的重复代码，降低模块间的耦合度。另外，AOP 还解决一些系统层面上的问题，比如日志、事务、权限等。 |


Spring 是一种基于 Bean 的编程技术，它深刻地改变着 Java 开发世界。Spring 使用简单、基本的 Java Bean 来完成以前只有 EJB 才能完成的工作，使得很多复杂的代码变得优雅和简洁，避免了 EJB 臃肿、低效的开发模式，极大的方便项目的后期维护、升级和扩展。

在实际开发中，服务器端应用程序通常采用三层体系架构，分别为表现层（web）、业务逻辑层（service）、持久层（dao）。

Spring 致力于 Java EE 应用各层的解决方案，对每一层都提供了技术支持。

- 在表现层提供了对 Spring MVC、Struts2 等框架的整合；
- 在业务逻辑层提供了管理事务和记录日志的功能；
- 在持久层还可以整合 MyBatis、Hibernate 和 JdbcTemplate 等技术，对数据库进行访问。


这充分地体现了 Spring 是一个全面的解决方案，对于那些已经有较好解决方案的领域，Spring 绝不做重复的事情。



## 关键词

- IOC & AOP：SpringFramework 的两大核心特性：**Inverse of Control 控制反转、Aspect Oriented Programming 面向切面编程**.
- 轻量级：对比于重量级框架，它的规模更小（可能只有几个 jar 包）、消耗的资源更少.
- 一站式：覆盖企业级开发中的所有领域.
- 第三方整合：SpringFramework 可以很方便的整合进其他的第三方技术（如持久层框架 MyBatis / Hibernate ，表现层框架 Struts2 ，权限校验框架 Shiro 等）.
- 容器：SpringFramework 的底层有一个管理对象和组件的容器，由它来支撑基于 SpringFramework 构建的应用的运行.



## SpringFramework的版本

[某博主自己根据SpringBoot依赖包码出了一份版本对应关系](https://blog.csdn.net/java_zjn/article/details/108711513)

| SpringFramework版本 | 对应jdk版本 | 重要特性                                                     |
| ------------------- | ----------- | ------------------------------------------------------------ |
| SpringFramework 1.x | jdk 1.3     | 基于 xml 的配置                                              |
| SpringFramework 2.x | jdk 1.4     | 改良 xml 文件、初步支持注解式配置                            |
| SpringFramework 3.x | Java 5      | 注解式配置、JavaConfig 编程式配置、Environment 抽象          |
| SpringFramework 4.x | Java 6      | SpringBoot 1.x、核心容器增强、条件装配、WebMvc 基于 Servlet3.0 |
| SpringFramework 5.x | Java 8      | SpringBoot 2.x、响应式编程、SpringWebFlux、支持 Kotlin       |



## 面试题

### 什么是SpringFramework?
SpringFramework是一个开源的,松耦合的,可配置的一站式企业级Java开发框架.它的核心是IOC和AOP,它可以很容易的构建出企业级Java应用,并且可以根据应用开发的组件需要,整合对应的技术.

### 为什么使用SpringFramework?(对应上文的特征)

- IOC:组件之间的解耦.
- AOP:切面编程可以将应用业务做统一或者特定的功能增强,能实现应用业务与增强逻辑的解耦(例如记录每个管理员的操作日志).
- 容器与事件:管理应用中使用的组件Bean,托管Bean的生命周期,事件与坚挺的驱动机制.
- Web.事务控制,测试与其他技术的整合.

### SpringFramework包含哪些模块?
- beans,core,context,expression (核心包)
- aop (切面编程)
- jdbc (整合jdbc)
- orm (整合ORM框架)
- tx (事务控制)
- web (Web层技术)
- test (整合测试)
- .... (也对应了上文的特征)
