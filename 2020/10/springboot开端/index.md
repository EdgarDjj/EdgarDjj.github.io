# SpringBoot开端


# SpringBoot开端

## SpringBoot项目搭建和启动

New Project中的各字段的含义：

- “ GroupID ” 是项目组织唯一的标识符，实际对应 Java 的包结构，是 main 目录里 Java 的目录结构，本项目就将其设置为 ltd.newbee.mall。
- “ ArtifactID ” 是项目的唯一的标识符，实际对应项目的名称，也就是项目根目录的名称，名为 newbee-mall。
- “ Type ” 可以简单理解为项目管理工具，可以选择 Maven 构建或者 Gradle 构建，本项目选用的是常用的 Maven 方式。
- “ Language ” 表示编程语言的选择，现在支持 Java 、Kotlin 和 Groovy。
- “ Packaging ” 表示项目的打包方式，有两种选择：Jar 和 War，在 Spring Boot 生成后，如果选用的方式不同，那么导入的打包插件也有区别。
- “ Java Version ” 表示 JDK 版本的选择。
- “ Version ” 是项目版本号，IDEA 默认为 0.0.1-SNAPSHOT，也可以自行修改。

#### **Spring Boot**项目的调试和启动的三种方式：

- IDEA直接启动
- Maven插件启动
- 命令行启动



## SpringBoot项目结构

#### 根目录com.example

* 数据类实体类domain
  * jpa项目：com.example.domain
  * mybatis项目：com.example.entity
* 数据额接口访问层service
  * com.example.service
* 数据服务接口实现层
  * com.example.service.impl
* 前端控制层controller
  * com.example.controller
* 工具类库utils
  * com.example.utils
* 配置类config
  * com.example.config
* 数据传输对象dto（Data Transfer Object）用于封装多个实体类和domain之间的关系，不会破坏原有的实体类结构
  * com.example.dto
* 视图包装对象vo（View Object）用与封装客户端请求的数据，防止部分数据泄露：如管理员ID，保证数据安全，不会破坏原有实体类结构
  * com.example.vo
* 常量类constant
  * com.example.constant

#### 根目录resources

* 项目配置文件：resources.application.yml
* 静态资源目录：resources.static
  * 用于存放html、css、js、图片等资源
* 视图模版目录：resources.templates
  * 用于存放jsp、thymeleaf等模版文件
* mybatis映射文件：resources.mappers
* mybatis配置文件：resources.spring-mybatis.xml

## Spring

> Spring是重量级企业开发框架Enterprise JavaBean（EJB）的替代品，Spring通过**依赖注入**和**面向切面编程**，用简单的**Java对象（Plain Old Java Object，POJO）**实现了EJB的功能。

### 什么是Spring？

Spring一般指代Spring Framework，是一个企业级Java应用程序开发框架，提供了一种简易的开发方式，它避免了那些使得开发变得繁琐混乱的大量业务/工具对象，由框架去对Bean进行管理。即从自己买菜做饭变成了（容器）送菜到你面前去使用。

#### **三层架构**

- 表现层 web层 MVC是表层的一个设计模型
- 业务层 service层
- 持久层 dao层

在被管理对象与业务逻辑之间，Spring通过**IOC Inversion of Control（控制反转）**架起使用的桥梁，一个具体的例子就是DI **Dependency Injection（依赖注入）**，有效的实现了**松耦合**。而Spring的核心功能远不止这些，如：

- Spring AOP 
- Spring JDBC 提供了JDBC抽象层，消除了冗长的JDBC代码。
- Spring MVC
- Spring JMS 包含生产（produce）和消费（consume）消息的功能。
- 等等…..

其中Spring AOP，提供了**面向切面编程**的实现，而**AOP****面向切面编程**，一种编程思想并不是Spring的专有。可参考Spring-AOP。纵览Spring的整个结构，其实Spring本身并未提供太多具体的功能，它主要专注于让项目代码更加优雅整洁，使其具有灵活性和拓展性。

## SpringMVC和SpringBoot

### 什么是SpringMVC？

Spring MVC是Spring的一部分，Spring 出来以后，大家觉得很好用，于是按照这种模式设计了一个 MVC框架（一些用Spring 解耦的组件）。SpringMVC是围绕一个DispatcherServlet来设计的，这个Servlet会把请求分发给各个处理器，并支持可配置的处理器映射、视图渲染、本地化、时区与主题渲染等，甚至还能支持文件上传。**主要用于开发WEB应用和网络接口，它是Spring的一个模块，通过Dispatcher Servlet, ModelAndView 和 View Resolver，让应用开发变得很容易**。



### 什么是SpringBoot？

**虽然Spring的组件代码是轻量级的，但它的配置却是重量级的（需要大量XML配置）** 。Spring 2.5引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式XML配置。Spring 3.0引入了基于Java的配置，这是一种类型安全的可重构配置方式，可以代替XML。

尽管如此，我们依旧没能逃脱配置的魔爪。开启某些Spring特性时，比如事务管理和Spring MVC，还是需要用XML或Java进行显式配置。启用第三方库时也需要显式配置，比如基于Thymeleaf的Web视图。配置Servlet和过滤器（比如Spring的DispatcherServlet）同样需要在web.xml或Servlet初始化代码里进行显式配置。组件扫描减少了配置量，Java配置让它看上去简洁不少，但Spring还是需要不少配置。

光配置这些XML文件都够我们头疼的了，占用了我们大部分时间和精力。除此之外，相关库的依赖非常让人头疼，不同库之间的版本冲突也非常常见。

**不过，好消息是：Spring Boot让这一切成为了过去。**



### SpringBoot相较于SSM

一个SSM（Spring SpringMVC Mybatis整合）项目的步骤：

1. 建立maven web项目，并在pom.xml中添加依赖
2. 配置web.xml，引入spring-*.xml配置文件。
3. 配置若干个spring-*.xml文件，例如自动扫描、静态资源映射、默认视图解析、数据库连接池等等。
4. 写业务逻辑代码（dao、service、controller）

而Spring Boot实现了自动配置，减去了繁琐的项目搭建。

com

+- example

 +- myproject

  +- Application.java

  |

  +- domain

  | +- Customer.java

  | +- CustomerRepository.java

  |

  +- service

  | +- CustomerService.java

  |

  +- controller

  | +- CustomerController.java

  |  

  | +- config

  | +- swagerConfig.java

  |

1. Application.java是项目的启动类
2. domain目录主要用于实体（Entity）与数据访问层（Repository）
3. service 层主要是业务类代码
4. controller 负责页面访问控制
5. config 目录主要放一些配置类


