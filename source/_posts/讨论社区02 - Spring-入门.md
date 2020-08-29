---
title: 讨论社区02 - Spring 入门
date: 2020-08-29 20:56:39
tags: [Spring]
category: [讨论社区项目]
---



# Spring全家桶

- Spring Framework 
- Spring Boot
- Spring Cloud (微服务,大项目拆分成若干子项目)
- Spring Cloud Data Flow(数据集成)
- 官网: https://spring.io

# Spring Framework

- Spring Core
- IoC、AOP  (管理对象的思想,spring管理的对象叫做Bean.)
- Spring Data Access
  - Transactions(事务)、Spring MyBatis
- Web Servlet
  - Spring MVC
- Integration(集成)
  - Email、Scheduling(定时任务)、AMQP(消息队列)、Security(安全控制)

# Spring IoC

- Inversion of Control
  - 控制反转，是一种面向对象编程的设计思想。
- Dependency Injection
- 依赖注入，是IoC思想的实现方式。
- IoC Container
- IoC容器，是实现依赖注入的关键，本质上是一个工厂。
- 容器管理Bean的前提:提供Bean的类型,通过配置文件配置Bean之间的关系.
- 降低Bean之间的耦合度

# 代码部分

- 主动获取:

```java
@SpringBootApplication
public class TalkingApplication {
	public static void main(String[] args) {
		SpringApplication.run(CommunityApplication.class, args);
	}
}
配置类,启动时自动扫描,扫描配置类所在的包以及子包下的Bean.
@Component @Repository @Service @Controller
```

测试代码要以其为配置类,需加上注解:

```java
@ContextConfiguration(classes = TalkingApplication.class)
```

想要使用spring容器需要实现接口,ApplicationContextAware,实现接口中set方法.传入参数applicationContext(spring容器),他是一个接口,继承自BeanFactory.

获取Bean:applicationContext.getBean(test.class);

```java
public class TalkingApplicationTests implements ApplicationContextAware {

	private ApplicationContext applicationContext;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}
}
```

给Bean自定义名字:@Component("名字")

初始化方法@PostConstruct,在构造器之后调用.销毁对象之前调用,@PreDestroy.

@Scope()指定单例多例

@Configuration配置类,用以装载使用第三方类.

- 自动注入:
  - @Autowired