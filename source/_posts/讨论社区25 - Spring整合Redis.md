---
title: 讨论社区25  -  Spring整合Redis
date: 2019-07-27 22:15:41
tags: [Redis,Spring]
category: [讨论社区项目,编程笔记]
---

- 引入依赖

  - `spring-boot-starter-data-redis`

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
      <!--包含在spring boot中可以不用写VersionId，已做好兼容-->
  </dependency>
  ```

- 配置Redis

  - 配置数据库参数
  - 编写配置类，构造RedisTemplate

  ```xml
  # RedisProperties
  spring.redis.database=1
  spring.redis.host=localhost
  spring.redis.port=6379
  ```

  ```java
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.data.redis.connection.RedisConnectionFactory;
  import org.springframework.data.redis.core.RedisTemplate;
  import org.springframework.data.redis.serializer.RedisSerializer;
  
  @Configuration
  public class RedisConfig {
  
      @Bean
      public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory){
          RedisTemplate<String, Object>  template = new RedisTemplate<String, Object>();
          template.setConnectionFactory(factory);
  
          // 设置key的序列化方式
          template.setKeySerializer(RedisSerializer.string());
          // 设置value的序列化方式
          template.setValueSerializer(RedisSerializer.json());
          // 设置hash的key的序列化方式
          template.setHashKeySerializer(RedisSerializer.string());
          // 设置hash的value的序列化方式
          template.setHashValueSerializer(RedisSerializer.json());
  
          template.afterPropertiesSet();
          return template;
  
      }
  }
  ```

- 访问Redis

  - `redisTemplate.opsForValue()`
  - `redisTemplate.opsForHash()`
  - `redisTemplate.opsForList()`
  - `redisTemplate.opsForSet()`
  - `redisTemplate.opsForZSet()`

多次访问一个redisKey可以做绑定操作

```java
// 多次访问同一个key
@Test
public void testBoundOperations() {
    String redisKey = "test:count";
    BoundValueOperations operations = redisTemplate.boundValueOps(redisKey);
    operations.increment();
    operations.increment();
    operations.increment();
    operations.increment();
    operations.increment();
    System.out.println(operations.get());
}
```

Redis的事务处理

```java
// 编程式事务
@Test
public void testTransactional() {
    Object obj = redisTemplate.execute(new SessionCallback() {
    @Override
    public Object execute(RedisOperations operations) throws DataAccessException {
        String redisKey = "test:tx";

        operations.multi();

        operations.opsForSet().add(redisKey, "zhangsan");
        operations.opsForSet().add(redisKey, "lisi");
        operations.opsForSet().add(redisKey, "wangwu");
        
		//立即查询不会得出结果
        System.out.println(operations.opsForSet().members(redisKey));
        // 提交事务
        return operations.exec();
        }
    });
    
    System.out.println(obj);
}
```

