---
title: 讨论社区36  -  Spring Security
date: 2019-08-12 22:39:20
tags: [Spring,Spring Security]
category: [讨论社区项目]
---

https://sping.io/projects/spring-security

学习文档：http://www.spring4all.com/article/428

- ### 介绍

  - 简介
    - Spring Security是一个专注与为Java应用程序提供身份认证和授权的框架，它的强大之处在于它可以轻松扩展以满足自定义的需求。
  - 特征
    - 对身份的认证和授权提供全面的、可扩展的支持。
    - 防止各种攻击，如会话固定攻击、点击劫持、csrf攻击等。
    - 支持与Servelt API、Spring MVC等Web技术集成。
  - 原理
    - 底层使用Filter（javaEE标准）进行拦截
    - Filter-->DispatchServlet-->Interceptor-->Controller(后三者属于Spring MVC)
  - 推荐学习网站：www.spring4all.com
    - 看几个核心的Filter源码

  ### 使用

  - 导包：spring-boot-starter-security
  - User实体类实现UserDetails接口，实现接口中各方法（账号、凭证是否可用过期，管理权限）
  - UserService实现UserDetailsService接口,实现接口方法（security检查用户是否登录时用到该接口）
  - 新建SecurityConfig类
    - 继承WebSecurityConfigurerAdapter
    - 配置忽略静态资源的访问
    - 实现认证的逻辑，自定义认证规则（AuthenticationManager: 认证的核心接口）
      - 登录相关配置
      - 退出相关配置
    - 委托模式: ProviderManager将认证委托给AuthenticationProvider.
    - 实现授权的逻辑
      - 授权配置
      - 增加Filter,处理验证码
      - 记住我
  - 重定向，浏览器访问A,服务器返回302，建议访问B.一般不能带数据给B（Session和Cookie）
  - 转发，浏览器访问A，A完成部分请求，存入Request,转发给B完成剩下请求。（有耦合）
> 判断请求在服务端业务是否有耦合，在服务端A，B是独立业务则用重定向；否则使用转发。

  - 在HomeController添加认证逻辑
    - 认证成功后,结果会通过SecurityContextHolder存入SecurityContext中.

