# SpringMVC原理


# SpringMVC原理

* M 代表 Model

model 就是数据，dao、bean

* V 代表 View

网页、JSP，用来展示模型中的数据

* C 代表 Controller

把不同的数据（Model），显示在不同的视图（View）上

MVC模式主要解决了页面代码和后台代码的分离。

## Spring MVC原理

1. 客户端发送请求，request请求url
2. 前端控制器（DispatcherServlet）接收客户端请求，找到处理器映射HandlerMapping解析请求对应的Handle
3. HandlerAdaptor会根据Handler来调用真正的处理器controller处理请求，并处理相应的业务逻辑
4. 处理器返回一个模型视图ModeAndView
5. 视图解析器View resolver进行解析
6. 返回一个视图对象
7. 前端控制器DispatcherServlet渲染数据（Model）
8. 将得到的视图返回个客户端

#### 前端控制器DispatcherServlet

Servlet会拦截并处理HTTP请求，DispatcherServlet会拦截所有这些请求，并发送给Spring MVC控制器

```xml
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <!-- 拦截所有的请求 -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### 处理器映射HandlerMapping

DispatcherServlet会查询一个或多个处理器映射来确定请求的下一站在哪

### Spring Bean




