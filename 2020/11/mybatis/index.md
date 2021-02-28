# MyBatis


# Mybatis

### Mybatis中#{}和${}的区别

其最大的差别就是 `#{}` 可以防止sql注入，后 `${}` 不可以。

#### SQL注入

一般前台有两个输入框，一个用户名一个密码，传入到后台后会拼接两个参数

```mysql
select USER from table where username=userinput and password=userinput
```

来查询用户和校对，但是用户可以故意输入一个让后台解析失败的字符串，这就是SQL注入

```mysql
select USER from table where username=userinput and password='' or 1=1
```

就无需使用密码了，为了防止SQL注入的问题，首先要对单引号进行过滤，在添加逻辑

#### 关于#{}

1. #{} 表示一个占位符号，相当于JDBC中的 `？` #{} 实现的是向prepareStatement中预处理语句中设置参数值，sql语句中#{}表示一个占位符即?
2. #{}将传入的数据都当成一个字符串，会自动传入的数据加一个双引号。
3. 如果sql语句中只有一个参数时，参数名可以随意定义，若是一个实体或者map集合等，则需写『实体属性名』或者『map集合关键字』

#### 关于${}

1. 将传入的数据直接显示
2. ${}中参数名称不能随意更改
3. 简单来说JDBC不支持使用占位符的地方可以使用${}

#### Mybatis中的#{}和${}的区别

> #{}能防止sql注入

> 在JDBC能使用占位符的地方最好优先使用#{}

> 在JDBC不支持使用占位符的地方就只能使用${}动态参数

例如使用 order by、like动态参数的时候。

例如模糊查询

```mysql
select * from user where user_name link '%${param}%'
```

## MyBatis Generator 配置

### 引入插件

`Mybatis-Generator`的运行方式有很多种：

- 基于`mybatis-generator-core-x.x.x.jar`和其`XML`配置文件，通过命令行运行。
- 通过`Ant`的`Task`结合其`XML`配置文件运行。
- 通过`Maven`插件运行。
- 通过`Java`代码和其`XML`配置文件运行。
- 通过`Java`代码和编程式配置运行。
- 通过`Eclipse Feature`运行。

### 通过编码和配置文件运行

```xml
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

### 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--mybatis的代码生成器相关配置-->
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 引入配置文件 -->
    <properties resource="application-dev.properties"/>

    <!-- 一个数据库一个context,context的子元素必须按照它给出的顺序
        property*,plugin*,commentGenerator?,jdbcConnection,javaTypeResolver?,
        javaModelGenerator,sqlMapGenerator?,javaClientGenerator?,table+
    -->
    <context id="myContext" targetRuntime="MyBatis3" defaultModelType="flat">

        <!-- 这个插件给生成的Java模型对象增加了equals和hashCode方法 -->
        <!--<plugin type="org.mybatis.generator.plugins.EqualsHashCodePlugin"/>-->

        <!-- 注释 -->
        <commentGenerator>
            <!-- 是否不生成注释 -->
            <property name="suppressAllComments" value="true"/>
            <!-- 不希望生成的注释中包含时间戳 -->
            <!--<property name="suppressDate" value="true"/>-->
            <!-- 添加 db 表中字段的注释，只有suppressAllComments为false时才生效-->
            <!--<property name="addRemarkComments" value="true"/>-->
        </commentGenerator>


        <!-- jdbc连接 -->
        <jdbcConnection driverClass="${spring.datasource.driverClassName}"
                        connectionURL="${spring.datasource.url}"
                        userId="${spring.datasource.username}"
                        password="${spring.datasource.password}">
            <!--高版本的 mysql-connector-java 需要设置 nullCatalogMeansCurrent=true-->
            <property name="nullCatalogMeansCurrent" value="true"/>
        </jdbcConnection>

        <!-- 类型转换 -->
        <javaTypeResolver>
            <!--是否使用bigDecimal，默认false。
                false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer
                true，把JDBC DECIMAL 和 NUMERIC 类型解析为java.math.BigDecimal-->
            <property name="forceBigDecimals" value="true"/>
            <!--默认false
                false，将所有 JDBC 的时间类型解析为 java.util.Date
                true，将 JDBC 的时间类型按如下规则解析
                    DATE	                -> java.time.LocalDate
                    TIME	                -> java.time.LocalTime
                    TIMESTAMP               -> java.time.LocalDateTime
                    TIME_WITH_TIMEZONE  	-> java.time.OffsetTime
                    TIMESTAMP_WITH_TIMEZONE	-> java.time.OffsetDateTime
                -->
            <!--<property name="useJSR310Types" value="false"/>-->
        </javaTypeResolver>

        <!-- 生成实体类地址 -->
        <javaModelGenerator targetPackage="com.wqlm.boot.user.po" targetProject="src/main/java">
            <!-- 是否让 schema 作为包的后缀，默认为false -->
            <!--<property name="enableSubPackages" value="false"/>-->
            <!-- 是否针对string类型的字段在set方法中进行修剪，默认false -->
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>


        <!-- 生成Mapper.xml文件 -->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <!--<property name="enableSubPackages" value="false"/>-->
        </sqlMapGenerator>

        <!-- 生成 XxxMapper.java 接口-->
        <javaClientGenerator targetPackage="com.wqlm.boot.user.dao" targetProject="src/main/java" type="XMLMAPPER">
            <!--<property name="enableSubPackages" value="false"/>-->
        </javaClientGenerator>


        <!-- schema为数据库名，oracle需要配置，mysql不需要配置。
             tableName为对应的数据库表名
             domainObjectName 是要生成的实体类名(可以不指定，默认按帕斯卡命名法将表名转换成类名)
             enableXXXByExample 默认为 true， 为 true 会生成一个对应Example帮助类，帮助你进行条件查询，不想要可以设为false
             -->
        <table schema="" tableName="user" domainObjectName="User"
               enableCountByExample="false" enableDeleteByExample="false" enableSelectByExample="false"
               enableUpdateByExample="false" selectByExampleQueryId="false">
            <!--是否使用实际列名,默认为false-->
            <!--<property name="useActualColumnNames" value="false" />-->
        </table>
    </context>
</generatorConfiguration>
```

## MyBatis Generator 使用

### 什么是example类

Mybatis-Generator会为每个字段产生Criterion，为底层mapper.xml创建动态sql。如果表的字段比较多，产生的example类会非常庞大。理论上通过example类可以构造你想到的任何筛选条件。在Mybatis-Generator中加以配置，配置数据表的生成操作就可以自动生成example。

### 了解example成员变量

```java
 //作用：升序还是降序
 //参数格式：字段+空格+asc(desc)
 protected String orderByClause;  
 //作用：去除重复
 //true是选择不重复记录，false，反之
 protected boolean distinct;
 //自定义查询条件
 //Criteria的集合，集合中对象是由or连接
 protected List<Criteria> oredCriteria;
 //内部类Criteria包含一个Cretiron的集合，
 //每一个Criteria对象内包含的Cretiron之间是由  AND连接的
 public static class Criteria extends GeneratedCriteria {
  protected Criteria() {super();}
 }
 //是mybatis中逆向工程中的代码模型
 protected abstract static class GeneratedCriteria {......}
 //是最基本,最底层的Where条件，用于字段级的筛选
 public static class Criterion {......}
```

### Mapper接口中的方法解析

mapper接口中的函数及方法

| 方法                                                         | 功能说明                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| int countByExample(UserExample example) thorws SQLException  | 按条件计数                                                   |
| int deleteByPrimaryKey(Integer id) thorws SQLException       | 按主键删除                                                   |
| int deleteByExample(UserExample example) thorws SQLException | 按条件查询                                                   |
| String/Integer insert(User record) thorws SQLException       | 插入数据（返回值为ID）                                       |
| User selectByPrimaryKey(Integer id) thorws SQLException      | 按主键查询                                                   |
| ListselectByExample(UserExample example) thorws SQLException | 按条件查询                                                   |
| ListselectByExampleWithBLOGs(UserExample example) thorws SQLException | 按条件查询（包括BLOB字段）。只有当数据表中的字段类型有为二进制的才会产生。 |
| int updateByPrimaryKey(User record) thorws SQLException      | 按主键更新                                                   |
| int updateByPrimaryKeySelective(User record) thorws SQLException | 按主键更新值不为null的字段                                   |
| int updateByExample(User record, UserExample example) thorws SQLException | 按条件更新                                                   |
| int updateByExampleSelective(User record, UserExample example) thorws SQLException | 按条件更新值不为null的字段                                   |

###  example实例解析

mybatis的逆向工程中会生成实例及实例对应的example，example用于添加条件，相当where后面的部分
xxxExample example = new xxxExample();
Criteria criteria = new Example().createCriteria();

| 方法                                       | 说明                                          |
| ------------------------------------------ | --------------------------------------------- |
| example.setOrderByClause(“字段名 ASC”);    | 添加升序排列条件，DESC为降序                  |
| example.setDistinct(false)                 | 去除重复，boolean型，true为选择不重复的记录。 |
| criteria.andXxxIsNull                      | 添加字段xxx为null的条件                       |
| criteria.andXxxIsNotNull                   | 添加字段xxx不为null的条件                     |
| criteria.andXxxEqualTo(value)              | 添加xxx字段等于value条件                      |
| criteria.andXxxNotEqualTo(value)           | 添加xxx字段不等于value条件                    |
| criteria.andXxxGreaterThan(value)          | 添加xxx字段大于value条件                      |
| criteria.andXxxGreaterThanOrEqualTo(value) | 添加xxx字段大于等于value条件                  |
| criteria.andXxxLessThan(value)             | 添加xxx字段小于value条件                      |
| criteria.andXxxLessThanOrEqualTo(value)    | 添加xxx字段小于等于value条件                  |
| criteria.andXxxIn(List<？>)                | 添加xxx字段值在List<？>条件                   |
| criteria.andXxxNotIn(List<？>)             | 添加xxx字段值不在List<？>条件                 |
| criteria.andXxxLike(“%”+value+”%”)         | 添加xxx字段值为value的模糊查询条件            |
| criteria.andXxxNotLike(“%”+value+”%”)      | 添加xxx字段值不为value的模糊查询条件          |
| criteria.andXxxBetween(value1,value2)      | 添加xxx字段值在value1和value2之间条件         |
| criteria.andXxxNotBetween(value1,value2)   | 添加xxx字段值不在value1和value2之间条件       |
