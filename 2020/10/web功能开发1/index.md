# Web功能开发（一）


# Web 功能开发（一）

## DispatchServlet 简介

![dispatcherservlet](Web功能开发/dispatcherservlet.png)



## 自动配置

Web开发必不可缺的两个类：

DispatcherServletAutoConfiguration（ DispatchServlet 生效）

WebMvcAutoConfiguration （视图解析器、静态资源处理等）

SpringBoot 为 SpringMVC 提供自动配置：

自动配置在Spring的默认值之上添加了以下功能：

- 包含`ContentNegotiatingViewResolver`和`BeanNameViewResolver` beans。
- 支持提供静态资源，包括对WebJars的支持。
- 自动注册`Converter`，`GenericConverter`和`Formatter` beans。
- 支持`HttpMessageConverters`。
- 自动注册`MessageCodesResolver`。
- 静态`index.html`支持。
- 自定义`Favicon`支持。
- 自动使用`ConfigurableWebBindingInitializer` bean。








