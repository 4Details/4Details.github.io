---
title: java反射机制01：概述
date: 2020-09-20 22:26:51
tags: [反射]
category: [Java基础]
---

打算系统学习一些java的注解与反射的知识，要了解反射与注解，首先得弄清楚注解、反射的概念是什么。

这一节主要讲一下**注解**。

#  什么是注解

Annotation是从JDK 5.0开始引入的新技术

**Annotation的作用：** 

- 不是程序本身，可以对程序做出解释（这一点和注释没有区别）
- 可以被其他程序（比如：编译器等）读取。

**Annotation的格式：**

注解是以“@注释名”在代码中存在的，还可以添加一些参数值，例如： @SuppressWarnings(value = "unchecked")

**Annotation在哪里使用？**

可以附加在package，class，method，field等上面，相当于给他们添加了额外的辅助信息，我们可以通过反射机制编程实现对元数据的访问。

#  有哪些注解



