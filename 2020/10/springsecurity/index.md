# SpringSecurity


# SpringSecurity

## 简介

SpringSecurity是一个强大的可高度定制的认证和授权框架，对于Spring应用来说它是一套Web安全标准。SpringSecurity注重于为Java应用提供认证和授权功能，像所有的Spring项目一样，它对自定义需求具有强大的扩展性。

### Shiro、SpringSecurity

不同：功能很像

相同：认证、授权

* 功能权限
* 访问权限
* 菜单权限
* …拦截器，过滤器：大量原生代码，荣誉

几个重要的类：

* WebSecurityConfigurerAdapter：自定义Security策略
* AuthenticationManagerBuilder：自定义认证策略
* @EnableWebSecurity：开启WebSecurity模式，@EnableXXX开启某个功能

其主要的两个目标

- “认证” （Authentication）

  - 身份验证是关于验证您的凭据，如用户名/用户ID和密码，以验证您的身份。

    身份验证通常通过用户名和密码完成，有时与身份验证因素结合使用。

- “授权” （Authorization）

  - 授权发生在系统成功验证您的身份后，最终会授予您访问资源（如信息，文件，数据库，资金，位置，几乎任何内容）的完全权限。

    这个概念是通用的，而不是只在Spring Security 中存在。

## 使用

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter
{
  // 授权
  @Override
  protected void configure(HttpSecurity http) throws Exception {

    // @formatter:off
    http.cors()
      // 关闭 CSRF
      .and().csrf().disable()
      // 登录行为由自己实现，参考 AuthController#login
      .formLogin().disable()
      .httpBasic().disable()

      // 认证请求
      .authorizeRequests()
      // 所有请求都需要登录访问
      .anyRequest()
      .authenticated()
      // RBAC 动态 url 认证
      .anyRequest()
      .access("@rbacAuthorityService.hasPermission(request,authentication)")

      // 登出行为由自己实现，参考 AuthController#logout
      .and().logout().disable()
      // Session 管理
      .sessionManagement()
      // 因为使用了JWT，所以这里不管理Session
      .sessionCreationPolicy(SessionCreationPolicy.STATELESS)

      // 异常处理
      .and().exceptionHandling().accessDeniedHandler(accessDeniedHandler);
    // @formatter:on

    // 添加自定义 JWT 过滤器
    http.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
  }
  
  // 认证
      @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception
    {
        auth.userDetailsService(adminUserServiceImpl).passwordEncoder(new BCryptPasswordEncoder());
    }

  
    @Override
    public void configure(WebSecurity web) throws Exception
    {
//        静态资源放行
        web.ignoring().antMatchers("/static/**");
    }
}
```

AOP：拦截器

```java
// 首页是所有人可以访问，功能页只有对应权限的人才能进入
			// 请求授权的规则
					http.authorizeRequests()
                .antMatchers("/").permitAll()//"/"所有角色都可以通過
                .antMatchers("/teacher/**").hasAnyRole("teacher")//"/teacher/**"只有角色為teacher才可以通過
                .antMatchers("/student/**").hasAnyRole("student")
                .antMatchers("/admin/**").hasAnyRole("admin")
                .antMatchers("/user/**").hasAnyRole("user");

//      非授权访问时跳转的方法
        http.formLogin();

//        安全
//关闭csrf功能:跨站请求伪造,默认只能通过post方式提交logout请求
        http.csrf().disable();

//      用户注销后返回的界面
        http.logout().logoutSuccessUrl("/");

// 记住我 信息会保存到cookie
//定制请求的授权规则
@Override
protected void configure(HttpSecurity http) throws Exception {
//。。。。。。。。。。。
   //记住我
   http.rememberMe();
}
```

如果注销404了，就是因为它默认防止csrf跨站请求伪造，因为会产生安全问题，我们可以将请求改为post表单提交，或者在spring security中关闭csrf功能；我们试试：在 配置中增加 `http.csrf().disable();`

## 补充

CSRF（Cross-site request forgery）跨站请求伪造。在spring security中的跨域保护。是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。跟跨网站脚本（XSS）相比，**XSS** 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。


