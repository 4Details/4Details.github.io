---
title: 讨论社区04  -  Mybatis  入门
date: 2019-06-18 21:00:28
tags: [Mybatis,Spring]
category: [讨论社区项目]
---

# 安装数据库

- 安装MySQL Server
- 安装MySQL Workbench

# MyBatis

- 核心组件
  - SqlSessionFactory：用于创建SqlSession的工厂类。
  - SqlSession：MyBatis的核心组件，用于向数据库执行SQL。
  - 主配置文件：XML配置文件，可以对MyBatis的底层行为做出详细的配置。
  - Mapper接口：就是DAO接口，在MyBatis中习惯性的称之为Mapper。
  - Mapper映射器：用于编写SQL，并将SQL和实体类映射的组件，采用XML、注解均可实现。
- 示例
  - 使用MyBatis对用户表进行CRUD操作。
- 在application.properties中配置数据库、Mybatis相关。

```xml
# DataSourceProperties
#数据库驱动，5.x版本和8.x版本不一样
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
#数据库连接url，jdbc:mysql://数据库ip地址:端口/数据库名？字符编码&是否启用安全连接&时区
spring.datasource.url=jdbc:mysql://localhost:3306/{database-name}?characterEncoding=utf-8&useSSL=false&serverTimezone=Hongkong
#数据库用户名和密码
spring.datasource.username={username}
spring.datasource.password={password}
#连接池类型，最大连接数，最小空闲和超时时间
spring.datasource.type=com.zaxxer.hikari.HikariDataSource
spring.datasource.hikari.maximum-pool-size=15
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=30000

# MybatisProperties
#mybatis相关配置，mapper的位置，实体类的包，自动生成key和下划线命名驼峰命名的匹配
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package={entity的包路径}
mybatis.configuration.useGeneratedKeys=true
mybatis.configuration.mapUnderscoreToCamelCase=true
```

