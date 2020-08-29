---
title: 讨论社区02  -  Spring MVC 入门
date: 2019-06-14  21:00:28
tags: [SpringMVC]
category: [讨论社区项目]
---

# HTTP

- HyperText Transfer Protocol
- 用于传输HTML等内容的应用层协议
- 规定了浏览器和服务器之间如何通信，以及通信时的数据格式。
- 学习网站：https://developer.mozilla.org/zh-CN

浏览器服务器通信步骤：

1. 打开一个TCP连接
2. 发生一个HTTP报文 
3. 读取服务器返回的报文信息
4. 关闭连接或为后续请求重用连接

- 按下F12进入调试，在Network下看请求响应（Header和Response）

# Spring MVC

- 三层架构
  - 表现层(mvc)、业务层、数据访问层
- MVC(设计模式)
- Model：模型层
- View：视图层
- Controller：控制层
- 核心组件
  - 前端控制器：DispatcherServlet

浏览器访问服务器，首先访问的时Controller控制层，Controller调用业务层处理，处理完后将得到的数据封装到Model,传给视图层。

 <img src="../../../../java-project/nowcoder-project-master/note/img/image-20191111134655288.png" alt="avatar" style="zoom:60%;" />

# Thymeleaf

- 模板引擎

  - 生成动态的HTML。

- Thymeleaf

  - 倡导自然模板，即以HTML文件为模板。

- 常用语法

  - 标准表达式、判断与循环、模板的布局。

  ```html
  <!--声明thymeleaf模板-->
  <html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
      
  <!--静态资源也需要做路径处理，带url的不需要处理-->
  <link rel="stylesheet" th:href="@{/css/global.css}" />
  <link rel="stylesheet" th:href="@{/css/discuss-detail.css}" />
  ```

  ```html
  <!--标签是否显示demo-->
  <!--usernameMsg为Controller的model传过来的值-->
  <div class="col-sm-10">
  		<input type="text"
  			th:class="|form-control ${usernameMsg!=null?'is-invalid':''}|"
  			th:value="${user!=null?user.username:''}"
  			id="username" name="username" placeholder="请输入您的账号!" required>
  			<div class="invalid-feedback" th:text="${usernameMsg}">
  				该账号已存在!
  			</div>
  </div>
  ```

变量用  `${变量名}`， href 中使用 `@{url地址}`，常量与变量拼接 `|常量+{变量名}|`

```html
<span th:utext="${post.title}">test title</span>

<a class="nav-link" th:href="@{/index}">首页</a>

<form method="post" th:action="@{|/comment/add/${post.id}|}">
```



# 代码部分

底层：

```java
@RequestMapping("/http")
public void http(HttpServletRequest request, HttpServletResponse response) {
    // 获取请求数据
    System.out.println(request.getMethod());
    System.out.println(request.getServletPath());
    Enumeration<String> enumeration = request.getHeaderNames();
    while (enumeration.hasMoreElements()) {
        String name = enumeration.nextElement();
        String value = request.getHeader(name);
        System.out.println(name + ": " + value);
    }
    System.out.println(request.getParameter("code"));

    // 返回响应数据
    response.setContentType("text/html;charset=utf-8");
    try (
        PrintWriter writer = response.getWriter();
    ) {
        writer.write("<h1>xx网</h1>");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

从路径中得到变量GET（两种方法）：

```java
@RequestMapping(path = "/students", method = RequestMethod.GET)
@ResponseBody
public String getStudents(
    @RequestParam(name = "current", required = false, defaultValue = "1") int current,
    @RequestParam(name = "limit", required = false, defaultValue = "10") int limit) {
    System.out.println(current);
    System.out.println(limit);
    return "some students";
}

@RequestMapping(path = "/student/{id}", method = RequestMethod.GET)
@ResponseBody
public String getStudent(@PathVariable("id") int id) {
    System.out.println(id);
    return "a student";
}
```

POST请求:

```java
@RequestMapping(path = "/student", method = RequestMethod.POST)
@ResponseBody
public String saveStudent(String name, int age) {
    System.out.println(name);
    System.out.println(age);
    return "success";
}
```

响应HTML数据(使用ModelAndView或Model):

```java
@RequestMapping(path = "/teacher", method = RequestMethod.GET)
public ModelAndView getTeacher() {
    ModelAndView mav = new ModelAndView();
    mav.addObject("name", "张三");
    mav.addObject("age", 30);
    mav.setViewName("/demo/view");
    return mav;
}

@RequestMapping(path = "/school", method = RequestMethod.GET)
public String getSchool(Model model) {
    model.addAttribute("name", "北京大学");
    model.addAttribute("age", 80);
    return "/demo/view";
}
```

 响应JSON数据(异步请求)：Java对象 -> JSON字符串 -> JS对象,使用`@ResponseBody` 注解

```java
@RequestMapping(path = "/emp", method = RequestMethod.GET)
@ResponseBody
public Map<String, Object> getEmp() {
    Map<String, Object> emp = new HashMap<>();
    emp.put("name", "张三");
    emp.put("age", 23);
    return emp;
}
//转换为json字符串  {"name":"张三","age":"23"}
//也可以返回List<Map<String, Object>>，list集合。
```

## 