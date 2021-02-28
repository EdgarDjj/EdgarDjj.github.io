# Spring


# Spring

> 关于《Spring实战》的一些学习

## 前言

任何实际的应用程序都是很多组件组成的，每个组件负责整个应用功能的一部分，这些组件需要与其他的应用元素进行协调以完成自己的任务。当程序需要运行的时候，需要以某种方式创建引入这些组件。

**Spring的核心就是提供一个容器（container），通常称为Spring应用上下文（Spring Application Context），它们会创建和管理应用组件。**而这些组件也称为 **Bean** ，会在Spring Application Context中装配到一起，从而形成一个完整的应用程序。

将 **Bean** 装配到一起的行为称为 **依赖注入（Dependency Injection）的模式** 实现的。此时，组件不会创建它所依赖的组件并管理他们生命周期，使用 **DI模式** 的应用依赖于单独实体（Container）去创建和维护所有组件，并将其注入相应的Bean中。

配置Spring的方式：

* 使用一个或多个XML文件（描述各个组件以及其他组件之间的关系）
* 使用Java配置

相比使用XML方式，基于Java配置有更好的类型安全性和重构能力。

**@Configuration**

会告知Spring这是一个配置类，会为Spring Application Context提供bean

**@Bean**

表示这些方法所返回的对象会以bean的形式添加到Spring Application Context中（默认情况下，这些bean所对应的bean ID与定义它们的方法名相同）

在Spring中，自动配置依赖于 **自动装配（autowiring）和组件扫描（component scanning）**。借助component scanning Spring能自动发现该应用类路径下的组件，并创建到Spring Applicaiton Context的bean中。

```xml
    <!-- 自动扫描所有的注解 -->
    <context:component-scan base-package="com.xxxx"/>
```

而Spring Boot极大地改善了Spring中显式配置，提供了更多增强生产效率的方法。

## Spring核心框架

* Spring：是Spring领域中的一切基础，提供核心容器和DI框架
* SpringMVC：Spring中的web框架
* Spring Data：基本数据的持久化支持
  * 将应用程序的数据respository定义为简单的Java接口，在定义驱动存储和检索数据的方法时使用一种命名约定即可
  * 此外Spring Data能处理多种不同类型的数据库，包含关系型JPA、文档型Mongo、图数据库Neo4j
* Spring Security
  * 解决应用程序通用的安全性需求，认证（Authentication）和授权（Authorization）
* Spring Ingration 和 Spring Batch
  * Spring Ingration 解决了实时集成问题
  * Spring Batch 解决了批处理集成的问题
* **Spring Boot**
* Spring Cloud

## 开始

### Spring项目结构

#### 注意

* mvnw和mvnw.cmd
  * 这是Maven包装器（wrapper）脚本。
  * 借助该脚本，即使主机上没有安装maven也可构建项目。
* maven中 `<packaging>jar</packaging>`默认为jar类型
  * 传统的Java Web应用都是打包成war文件，但jar是所有云平台都能运行可执行jar文件。

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

## Web使用

略

## 使用数据

使用JDBC或JPA

### 最基本的JDBC连接数据库

```java
import java.sql.*;

/**
 * Description:
 *
 * @date: 2020/3/20
 * @author: Edgar_Ding
 */
public class DBUtil {
    private static final String DRIVER = "com.mysql.jdbc.Driver";
    private static final String URL = "jdbc:mysql://localhost:3306/jdbc201910";

    private static final String USER = "root";
    private static final String PASSWORD = "dingjunjie";
    /**
     * 获取数据库连接
     * @return
     * @throws SQLException
     * @throws ClassNotFoundException
     */
    public Connection getConnection() throws SQLException, ClassNotFoundException {
        //1.加载驱动程序
        Class.forName(DRIVER);
        //2. 获得数据库连接
        return DriverManager.getConnection(URL, USER, PASSWORD);
    }
    //关闭资源 增删改
    public static void closeIDU(Connection connection, Statement statement) throws SQLException {
        if(statement != null) {
            statement.close();
        }
        if(connection != null) {
            connection.close();
        }
    }
    //关闭资源 查询
    public static void closeR(ResultSet resultSet, Connection connection, Statement statement) throws SQLException {
        if(resultSet != null) {
            resultSet.close();
        }
        if(connection != null) {
            connection.close();
        }
        if(statement != null) {
            statement.close();
        }
    }
    //关闭资源 无论是增删改还是查询都行
    public static void close(AutoCloseable... autoCloseables) {
        for (AutoCloseable autoCloseable : autoCloseables) {
            try {
                if (autoCloseable != null) {
//                    System.out.println(autoCloseable.getClass().getName());
                    autoCloseable.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
    /**
     * 根据id删除用户
     * @param id
     */
    public void deleteUserById(int id) throws SQLException, ClassNotFoundException {
        //1、注册驱动
        //2、建立连接
        PreparedStatement preparedStatement = null;
        Connection connection = null;
        try {
            connection = new DBUtil().getConnection();
            //3、定义sql
            String sql = "delete from user where id = ?";
            //4、创建preparedStatement
            preparedStatement = connection.prepareStatement(sql);
            //5、设置参数
            preparedStatement.setInt(1, id);
            //6、执行sql （一定不能传入参数，不报错，但不返回值）
            int affectRows = preparedStatement.executeUpdate();
            System.out.println("affectRows : " + affectRows);
        } finally {
            //7、关闭资源
            DBUtil.close(preparedStatement, connection);
        }
    }
```

## 保护Spring

### Spring Security

详情查[SpringSecurity](/2020/10/14/SpringSecurity/)

## 创建REST服务

详情查[RESTfulAPI简明](/2020/10/26/RESTfulAPI简明/)

### 启用超媒体

**超媒体作为应用状态引擎（Hypermedia as the Engine of Application State，HATEOAS）**

创建一种自描述API的方式，API返回的资源中会包含相关资源的连接，客户端只需要最少的API URL信息就能导航整个API。


