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

