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