# Spring Security

SpringSecurity 是一个基于 Spring 开发的非常强大的权限验证框架，核心功能包括：

> - 认证（用户登录）
> - 授权（此用户可以进行的操作）
> - 攻击防护（防止伪造身份攻击）

Security 也有其他安全框架 如更加易用的 Shiro 等等。

## 开发配置环境

~~~ xml
<!--  处理 Web 安全性基础设施，包括 Web 应用安全相关的基本功能，如处理 Http、Filter等  -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-web</artifactId>
    <version>6.1.1</version>
</dependency>
<!--   提供了 Spring Security 配置支持，允许使用 Java 或 XML 进行安全性规则和策略配置  -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>6.1.1</version>
</dependency>
~~~



## ##内部原理

### 过滤器

Spring Security 中一共提供了 32 个过滤器，其中默认使用的有 15 个，其中13个如下图所示（少了 CsrfFilter 和 BasicAuthenticationFilter 这两个），首先我们就先来看看过滤器的配置问题。

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231217220549.png)

> - 0-3 和 CsrfFilter 这几个过滤器是 功能性的前置过滤器，提供了SpringSecurity的基础必要能力。
> - 4-12 和 BasicAuthenticationFilter 则与认证和授权过程相关
>
> 详情可查看博客：https://zhuanlan.zhihu.com/p/646662714

在一个 Web 项目中，请求流程大概如下图所示：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231217220936.png)

请求从客户端发起（例如浏览器），然后穿过层层 Filter，最终来到 Servlet 上，被 Servlet 所处理。

那么 Spring Security 中默认的 15 个过滤器就是这样嵌套在 Client 和 Servlet 之间吗？并非如此。

上图中的 Filter 我们可以称之为 Web Filter，Spring Security 中的 Filter 我们可以称之为 Security Filter，它们之间的关系如下图：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231217220716.png)

可以看到，Spring Security Filter 并不是直接嵌入到 Web Filter 中的，而是通过 FilterChainProxy 来统一管理 Spring Security Filter，FilterChainProxy 本身则通过 Spring 提供的 DelegatingFilterProxy 代理过滤器嵌入到 Web Filter 之中。

### 2.过滤器链

上面介绍的是单个过滤器链，实际上，在 Spring Security 中，可能存在多个过滤器链。

有人会问，下面这种配置是不是就是多个过滤器链？

~~~java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .antMatchers("/admin/**").hasRole("admin")
            .antMatchers("/user/**").hasRole("user")
            .anyRequest().authenticated()
            ...
            .csrf().disable();
}
~~~

这个配置非常常见，但是这并不是多个过滤器链而是一个过滤器，因为不管是 `/admin/**` 还是 `/user/**` ，走过的过滤器都是一样的，只是不同的路径判断条件不一样而已。

如果系统存在多个过滤器链，多个过滤器链会在 FilterChainProxy 中进行划分，如下图：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231217221457.png)

可以看到，当请求到达 FilterChainProxy 之后，FilterChainProxy 会根据请求的路径，将请求转发到不同的 Spring Security Filters 上面去，不同的 Spring Security Filters 对应了不同的过滤器，也就是不同的请求将经过不同的过滤器。

正常情况下，我们配置的都是一个过滤器链，多个过滤器链怎么配置呢？举一个简单的例子：
~~~java
@Configuration
public class SecurityConfig {
    @Configuration
    @Order(1)
    static class DefaultWebSecurityConfig extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.antMatcher("/foo/**")
                    .authorizeRequests()
                    .anyRequest().hasRole("admin")
                    .and()
                    .csrf().disable();
        }
    }
    
    @Configuration
    @Order(2)
    static class DefaultWebSecurityConfig2 extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http.antMatcher("/bar/**")
                    .authorizeRequests()
                    .anyRequest().hasRole("user")
                    .and()
                    .formLogin()
                    .permitAll()
                    .and()
                    .csrf().disable();
        }
    }
}
~~~

* 首先，SecurityConfig 不再需要继承自 WebSecurityConfigurerAdapter 了，只是作为一个普通的配置类，加上 @Configuration 注解即可。

* 提供 UserDetailsService 实例，相当于是我们的数据源。

* 创建静态内部类继承 WebSecurityConfigurerAdapter 类，同时用 @Configuration 注解标记静态内部类是一个配置类，配置类里边的代码就和之前的一样了，无需赘述。

* 每一个静态内部类相当于就是一个过滤器链的配置。

* 注意在静态内部类里边，我没有使用 http.authorizeRequests() 开始，http.authorizeRequests() 配置表示该过滤器链过滤的路径是 /**。在静态内部类里边，我是用了 http.antMatcher("/bar/**") 开启配置，表示将当前过滤器链的拦截范围限定在 /bar/**。

* 当存在多个过滤器链的时候，必然会有一个优先级的问题，所以每一个过滤器链的配置类上通过 @Order(2) 注解来标记优先级。

从上面这段代码中大家可以看到，configure(HttpSecurity http) 方法似乎就是在配置过滤器链？是的没错！我们在该方法中的配置，都是在添加/移除/修改 Spring Security 默认提供的过滤器，所以该方法就是在配置 Spring Security 中的过滤器链。

