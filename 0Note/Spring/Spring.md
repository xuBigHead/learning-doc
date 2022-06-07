# 目录
[TOC]

# Spring总览
Spring框架至今已集成了20多个模块，这些模块分布在以下模块中：

- 核心容器（Core Container）
- 数据访问/集成（Data Access/Integration）层
- Web层
- AOP（Aspect Oriented Programming）模块
- 植入（Instrumentation）模块
- 消息传输（Messaging）
- 测试（Test）模块

![1](../../0Image/2022/1617728594.png)

##核心容器
### Spring-core
提供了框架的基本组成部分，包括控制反转（Inversion of Control，IOC）和依赖注入（Dependency Injection，DI）功能。

---

### Spring-beans
提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean。

---

### Spring-context
建立在Core和Beans模块的基础之上，提供一个框架式的对象访问方式，是访问定义和配置的任何对象的媒介。ApplicationContext接口是Context模块的焦点。

---

### Spring-context-support
持整合第三方库到Spring应用程序上下文，特别是用于高速缓存（EhCache、JCache）和任务调度（CommonJ、Quartz）的支持。

---

### Spring-expression
提供了强大的表达式语言去支持运行时查询和操作对象图。这是对JSP2.1规范中规定的统一表达式语言（Unified EL）的扩展。该语言支持设置和获取属性值、属性分配、方法调用、访问数组、集合和索引器的内容、逻辑和算术运算、变量命名以及从Spring的IOC容器中以名称检索对象。它还支持列表投影、选择以及常用的列表聚合。
 

## AOP和Instrumentation
### Spring-aop
提供了一个符合AOP要求的面向切面的编程实现，允许定义方法拦截器和切入点，将代码按照功能进行分离，以便干净地解耦。

---

### Spring-aspects
提供了与AspectJ的集成功能，AspectJ是一个功能强大且成熟的AOP框架。

---

### Spring-instrument
提供了类植入（Instrumentation）支持和类加载器的实现，可以在特定的应用服务器中使用。

--- 

## 消息
### Spring-messaging
该模块提供了对消息传递体系结构和协议的支持。

---

## 数据访问/集成
### Spring-jdbc
提供了一个JDBC的抽象层，消除了烦琐的JDBC编码和数据库厂商特有的错误代码解析。

---

### Spring-orm
为流行的对象关系映射（Object-Relational Mapping）API提供集成层，包括JPA和Hibernate。使用Spring-orm模块可以将这些O/R映射框架与Spring提供的所有其他功能结合使用，例如声明式事务管理功能。

---
 
### Spring-oxm
提供了一个支持对象/XML映射的抽象层实现，例如JAXB、Castor、JiBX和XStream。

---

### Spring-jms（Java Messaging Service）
指Java消息传递服务，包含用于生产和使用消息的功能。自Spring4.1以后，提供了与Spring-messaging模块的集成。

---

### Spring-tx（事务模块）
支持用于实现特殊接口和所有POJO（普通Java对象）类的编程和声明式事务管理。

---

## Web
### Spring-web
提供了基本的Web开发集成功能，例如多文件上传功能、使用Servlet监听器初始化一个IOC容器以及Web应用上下文。

---

### Spring-webmvc
也称为Web-Servlet模块，包含用于web应用程序的Spring MVC和REST Web Services实现。Spring MVC框架提供了领域模型代码和Web表单之间的清晰分离，并与Spring Framework的所有其他功能集成。

---

### Spring-websocket
Spring4.0以后新增的模块，它提供了WebSocket和SocketJS的实现。

---

### Portlet
类似于Servlet模块的功能，提供了Portlet环境下的MVC实现。

---

## 测试
### Spring-test
支持使用JUnit或TestNG对Spring组件进行单元测试和集成测试。



