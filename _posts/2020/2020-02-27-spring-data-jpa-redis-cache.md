---
layout: post
title: 【Spring Data 系列学习】Spring Data JPA +Redis+cache 实现缓存
category: springboot
tags: [springboot]
keywords: Spring Boot,spring data,JPA
excerpt: Spring Data JPA +Redis+cache 实现缓存
---

# 【Spring Data 系列学习】Spring Data JPA +Redis+cache 实现缓存

## 为什么要用缓存？

主要从“高性能”和“高并发”这两点来看待这个问题。主要来源是用户越多越多，数据量越来越大，对数据库操作的频率次数越高，数据库往往就会成为瓶颈，最常见的方式通过使用缓存将不经常改变的数据存储起来以提高系统性能。

spring boot 项目中可以使用 `Spring Cache ` 进行数据缓存,  `Spring   `对 `Cahce `进行了抽象，提供了 `@Cacheable、@CachePut、@CacheEvict` 等注解。

## 快速上手

创建一个 `Spring boot` 项目，在 `pom.xml ` 加入需要使用的依赖，如下： 

```xml
<!--   引入  jpa-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--mysql 驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!-- redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--commons pool2 创建连接池 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
<!-- jackson 序列化  安全版本：https://www.huaweicloud.com/notice/2018/20190801135923810.html -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.10.3</version>
</dependency>
```

### 配置文件application.properties

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456

# 打印 sql 语句
spring.jpa.show-sql= true
# 自动创建表
spring.jpa.properties.hibernate.hbm2ddl.auto=create
# 默认创建的mysql表为  MyISAM 引擎修改为InnoDB问题
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL55Dialect

# redis
spring.redis.host=localhost
# 连接端口号
spring.redis.port=6379
# 数据库索引 默认 0
spring.redis.database=0
# 连接池最大连接数 （默认：8）
spring.redis.lettuce.pool.max-active=8
# 连接池最小空闲连接 （默认：0）
spring.redis.lettuce.pool.min-idle=0
# 连接池最大阻塞等待时间，赋值表示没有限制 （默认：-1）
spring.redis.lettuce.pool.max-wait=-1ms
# 连接池最大空闲连接 （默认：8）
spring.redis.lettuce.pool.max-idle=8
# log 打印格式
logging.pattern.console=%msg%n
```

### 实体类映射数据库表

**user 实体类**

```java
@Entity
public class User implements Serializable {

    private static final long serialVersionUID = -390763540622907853L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private Integer age;
    
    private String email;

    // 省略构造器 set/get		

}
```

**在启动类增加`@EnableCaching`注解开启缓存功能，如下：**

```java
@SpringBootApplication
@EnableCaching
public class Chapter5Application {

    public static void main(String[] args) {
        SpringApplication.run(Chapter5Application.class, args);
    }

}
```

**UserJpaRepository  **

```java
public interface UserJpaRepository extends JpaRepository<User,Long> {
}
```

**数据实现类 UserServiceImpl**

```java
@Service
@CacheConfig(cacheNames = "user")
public class UserServiceImpl implements UserService {

    /**
     * ⽇志对象
     */
    private Logger logger = LoggerFactory.getLogger(UserServiceImpl.class);

    @Autowired
    private UserJpaRepository userJpaRepository;


    /**
     * 根据 id 查找
     *
     * @param id
     * @return
     */
    @Override
    @Cacheable(key = "#id",unless="#result == null")
    public User findById(Long id) {
        logger.info("id = {} 数据库获取数据 ", id);

        return userJpaRepository.findById(id).orElse(null);
    }

    /**
     * 保存用户
     *
     * @param user
     * @return
     */
    @Override
    @CachePut(key = "#result.id")
    public User save(User user) {
        logger.info("user = {} 保存数据到数据库 ", user);
        return userJpaRepository.save(user);
    }

    /**
     * 更新用户
     *
     * @param user
     * @return
     */
    @Override
    @CachePut(key = "#result.id")
    public User update(User user) {
        logger.info("user = {} 更新数据到数据库 ", user);
        return  userJpaRepository.save(user);
    }

    /**
     * 删除
     *
     * @param id
     */
    @Override
    @CacheEvict(key = "#id")
    public void deleteById(Long id) {
        logger.info("id = {} 数据库数据删除 ", id);
        userJpaRepository.deleteById(id);
    }
}

```

### Cache注解详解

- `@CacheConfig`:全局Cache配置，比如每一个方法中需要配置的集合名  `@Cacheable(value = "**")` ，可通过`cacheNames `实现全局配置（ `@CacheConfig(cacheNames = "**")`）。

- `@Cacheable`:
  - `value`,`cacheNames`: 缓存的集合名，`cacheNames`等同于`value`。
  - `key`：缓存的key，支持SpEL表达式。默认是使用所有参数及其计算的hashCode包装后的对象（SimpleKey）。如：`@Cacheable(key = "#p0")` 表示第一个参数，`@Cacheable(key = "#result")` 返回结果中的参数。
  - `condition`: 非必须。缓存的条件，支持SpEL表达式，当达到满足的条件时才缓存数据。在调用方法前后都会判断。
  - `unless`: 非必须，满足条件时不更新缓存，支持SpEL表达式，与 `condition` 只在调用方法后判断。
  - `keyGenerator`: 缓存key生成器，默认实现是`SimpleKeyGenerator`。与 `key` 互斥。
  - `cacheManager`: 指定使用哪个`CacheManager`。
  - `cacheResolver`:缓存解析器。
  - `sync`：回源到实际方法获取数据时，是否要保持同步，如果为false，调用的是Cache.get(key)方法；如果为true，调用的是Cache.get(key, Callable)方法。

- `@CachePut`:主要针对方法配置，能够根据方法的请求参数对其结果进行缓存，和 `@Cacheable `不同的是，它每次都会触发真实方法的调用。
- `@CacheEvict`: 主要针对方法配置，能够根据一定的条件对缓存进行清空。
  - `allEntries`:是否清空所有缓存内容，缺省为 false，如果指定为 true，则方法调用后将立即清空所有缓存。
  - `beforeInvocation`:是否在方法执行前就清空，缺省为 false，如果指定为 true，则在方法还没有执行的时候就清空缓存，缺省情况下，如果方法执行抛出异常，则不会清空缓存。

**单元测试类**

> 路径：src/test/java/com/mtcarpenter/service/impl/UserServiceImplTest.java

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceImplTest {
    /*
     * ⽇志对象
     */
    private Logger logger = LoggerFactory.getLogger(UserServiceImplTest.class);

    @Autowired
    private UserService userService;

    @Test
    public void findById() {
        logger.info("查询 id = {}", userService.findById(3L));
    }

    @Test
    public void save() {
        logger.info("新增数据 result = {}", userService.save(new User("小米", 9,"a@qq.com")));
    }

    @Test
    public void update() {
        logger.info("修改数据 result = {}", userService.save(new User(1L,"小米", 19,"a@qq.com")));
    }

    @Test
    public void deleteById() {
        userService.deleteById(1L);
    }
}
```



## 文章参考

- *https://docs.spring.io/spring/docs/current/spring-framework-reference/integration.html#cache*

- *https://my.oschina.net/dengfuwei/blog/1616221*

- *https://www.ibm.com/developerworks/cn/opensource/os-cn-spring-cache/*

## 代码示例

本文示例代码访问下面查看仓库：

- *github：* [https://github.com/mtcarpenter/spring-data-chapter](https://github.com/mtcarpenter/spring-data-chapter)
- *gitee* :      [https://gitee.com/mtcarpenter/spring-data-chapter](https://gitee.com/mtcarpenter/spring-data-chapter)

其中，本文示例代码名称：

- `charpter5`： Spring Data JPA +Redis+cache 实现缓存