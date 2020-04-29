---
layout: post
title: 【Spring Data 系列学习】Spring Data Redis 入门
category: springboot
tags: [springboot]
keywords: Spring Boot,spring data,JPA
---

# 【Spring Data 系列学习】Spring Data Redis 入门

## redis 简介
Redis是一个基于内存的键（key）值（value) 类型的数据结构存储容器,因为数据是存在内存中的，所以读写速度非常快，因此 redis 被广泛应用于缓存方向。另外，redis 也经常用来做分布式锁。redis 目前提供了5 种数据类型来支持不同的业务场景。除此之外，redis 支持事务 、持久化、LUA脚本、LRU驱动事件、多种集群方案。
## 应用场景 
缓存系统
排行版
计数器
社交网络
消息队列系统
实时系统

## 为什么要用 redis/为什么要用缓存
主要从“高性能”和“高并发”这两点来看待这个问题。
说起缓存框架，我们最常用的缓存框架有 memcached、Redis 这两个，但它们之间其实是有差异的。
Memcached 只支持单一的 key-value 存储，所以这里面存储的数据类型单一，无法适应多样化的业务发展。

在 Redis 缓存框架中，它支持多达 5 种类型的数据存储，并且提供了多个原子命令操作。并且 Redis 还支持了将数据持久化到本地文件，这样当发生意外时就不需要再从数据库读取一遍数据了，直接读取本地文件恢复即可。
从两个缓存框架的发展历程来看，我们可以知道 Redis 是 Memcached 的升级版本，Memcached 具有的功能 Redis 基本上都具备了。

所以很多时候我们都是使用 Redis 作为首选的缓存框架，当然了 Memcached 也有一些比 Redis 好一些的性能，比如在存储完全静态的小量 key-value 数据时，Memcached 会比 Redis快一些。

但只要数据量稍微大一点，或者数据是动态的，那么 Memcached 的性能就会直线下降。


## 数据类型

以下是 redis 常见的 5 种类型，表格描述来日《Redis实战》

| 结构类型 | 结构存储的值                                                 | 结构的读写能力                                               |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| STRING   | 可以是字符串、整数或者浮点数                                 | 对整个字符串或者字符串的其中一部分执行操作，对整数和浮点数执行自增或自减操作 |
| LIST     | 一个链表，链表上的每个节点都包含了一个字符串                 | 从两端压入或者弹出元素对单个或者多个元素进行修剪，只保留一个范围内的元素 |
| SET      | 包含字符串的无序收集器（unordered collection），并且被包含的每个字符串都是独一无二、各不相同的 | 添加、获取、移除单个元素检查一个元素是否存在于集合中计算交集、并集、差集从集合里面随机获取元素 |
| HAST     | 包含键值对的无序散列表                                       | 添加、获取、移除单个键值对 获取所有键值对 检查某个键是否存在 |
| ZSET     | 字符串成员（member）与浮点数分值（score ）之间的有序映射，元素的排列顺序由分值的大小决定 | 添加、获取、删除元素根据分值范围或者成员来获取元素 计算一个键的排名 |

## spring data redis

创建一个 Spring boot 项目，创建项⽬在 Chapter1项⽬ pom.xml 加入需要使⽤的依赖，如下所示： 

```xml
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
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.10.3</version>
</dependency>
```

> jackson-databind 安全版本参看:https://www.huaweicloud.com/notice/2018/20190801135923810.html

### 配置文件application.properties

```properties
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

**redis 配置类**

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper mapper = new ObjectMapper();
        //设置输入时忽略JSON字符串中存在而Java对象实际没有的属性
        mapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        // mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(mapper);
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        template.setValueSerializer(serializer);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        template.afterPropertiesSet();
        return template;
    }

}
```

配置主要解决 `RedisTemplate` 序列化问题， `RedisTemplate `使用默认 `JdkSerializationRedisSerializer`的序列d对象，在存入数据会将数据先序列化成字节数组然后在存入 `Redis` 数据库，通过工具查询也是字节数组显示，如下：

key = test ，value = test

字节数组显示如下：

key =  `\xAC\xED\x00\x05t\x00\x04test`  ,value = `\xAC\xED\x00\x05t\x00\x04test`

- `RedisTemplate`:使用的是 `JdkSerializationRedisSerializer`
- `StringRedisTemplate`:使用的是 `StringRedisSerializer`,字符串类型可以直接使用不用序列化。

Spring Data Redis 有如下几种序列化：

- `GenericToStringSerializer`: 可以将任何对象泛化为字符串并序列化
- `Jackson2JsonRedisSerializer`: 跟JacksonJsonRedisSerializer实际上是一样的
- `JacksonJsonRedisSerializer`: 序列化object对象为json字符串
- `JdkSerializationRedisSerializer`: 序列化java对象（其被序列化的对象必须实现Serializable接口）
- `StringRedisSerializer`: 简单的字符串序列化
- `GenericToStringSerializer`:类似StringRedisSerializer的字符串序列化
- `GenericJackson2JsonRedisSerializer`:类似Jackson2JsonRedisSerializer，但使用时构造函数不用特定的类参考以上序列化,自定义序列化类; 

**String**

**Java 实现**

```java
 /**
   * redis String 操作
   */
@Test
public void redisStringTest() {
    // 1 字符串操作
    // set hello world
    stringRedisTemplate.opsForValue().set("hello", "world");
    // set user name  10 秒失效
    /**
         * TimeUnit.DAYS          //天
         * TimeUnit.HOURS         //小时
         * TimeUnit.MINUTES       //分钟
         * TimeUnit.SECONDS       //秒
         * TimeUnit.MILLISECONDS  //毫秒
         */
    stringRedisTemplate.opsForValue().set("user", "name", 10, TimeUnit.SECONDS);
    // 查询字符串
    logger.info("result = {}", stringRedisTemplate.opsForValue().get("hello"));
    // 删除
    stringRedisTemplate.delete("hello");
    // 查询删除之后的
    logger.info("result = {}", stringRedisTemplate.opsForValue().get("hello"));
}
// result = world
// result = null
```

**String 数字操作**

```java
    /**
     * redis String 数字操作
     */
    @Test
    public void redisStringNumberTest() {
        // increment 自增
        // num 默认自增 1
        redisTemplate.opsForValue().increment("num");
        logger.info("result = {}", redisTemplate.opsForValue().get("num"));
        redisTemplate.opsForValue().increment("num", 2);
        logger.info("result = {}", redisTemplate.opsForValue().get("num"));
        // decrement 减
        redisTemplate.opsForValue().decrement("num");
        logger.info("result = {}", redisTemplate.opsForValue().get("num"));
    }

// result = 1
// result = 3
// result = 2
```

**hash**

```java
 /**
   * redis hash
   */
@Test
public void redisHashTest() {
    redisTemplate.opsForHash().put("userInfo", "name", "姓名");
    redisTemplate.opsForHash().put("userInfo", "age", 20);
    redisTemplate.opsForHash().put("userInfo", "sex", 0);
    // 删除
    redisTemplate.opsForHash().delete("userInfo", "sex");
    // 获取 userInfo name value
    logger.info("result = {}", redisTemplate.opsForHash().get("userInfo", "name"));
    // 查询 哈希键中的 fields
    logger.info("result = {}", redisTemplate.opsForHash().keys("userInfo"));
    // 查询哈希键中的所有 field 的 value
    logger.info("result = {}", redisTemplate.opsForHash().values("userInfo"));
}
// result = 姓名
// result = [name, age]
// result = [姓名, 20]
```

**list**

```java
    /**
     * redis list
     */
    @Test
    public void redisListTest() {
        redisTemplate.opsForList().leftPush("list-user", "list1");
        redisTemplate.opsForList().leftPush("list-user", "list2");
        redisTemplate.opsForList().leftPush("list-user", "list3");
        redisTemplate.opsForList().rightPush("list-user", new User("list4", 12, "123@qq.com"));
        // 获取全部
        logger.info("result = {}", redisTemplate.opsForList().range("list-user", 0, -1));
        // 索引为 1
        logger.info("result = {}", redisTemplate.opsForList().index("list-user", 1));
    }
// result = [list3, list2, list1, {id=null, name=list4, age=12, email=123@qq.com}]
// result = list2
```

**set**

```java
@Test
public void redisSetTest() {
    redisTemplate.opsForSet().add("set-key", "key1", "key2");
    redisTemplate.opsForSet().add("set-key", "key3");
    // 获取 key 的所有值
    logger.info("result = {}", redisTemplate.opsForSet().members("set-key"));
    // 判断 value 是否值是否在 key 中
    logger.info("result = {}", redisTemplate.opsForSet().isMember("set-key", "key3"));
}
// result = [key2, key1, key3]
// result = true
```

**zset**

```java
    /**
     * redis zset 
     */
    @Test
    public void redisZSetTest() {
        redisTemplate.opsForZSet().add("zset-key", "key1", 90);
        redisTemplate.opsForZSet().add("zset-key", "key2", 92);
        redisTemplate.opsForZSet().add("zset-key", "key3", 82);
        // 获取 key 中的 value 的分数
        logger.info("result = {}", redisTemplate.opsForZSet().score("zset-key", "key3"));
        // 移除
        redisTemplate.opsForZSet().remove("zset-key", "key1");
        logger.info("result = {}", redisTemplate.opsForZSet().range("zset-key", 0, -1));
    }
//  result = 82.0
//  result = [key3, key2]
```

> 路径：src/test/java/com/mtcarpenter/Chapter4ApplicationTests.java

## 代码示例

本文示例代码访问下面查看仓库：

- *github：* [https://github.com/mtcarpenter/spring-data-chapter](https://github.com/mtcarpenter/spring-data-chapter)
- *gitee* :      [https://gitee.com/mtcarpenter/spring-data-chapter](https://gitee.com/mtcarpenter/spring-data-chapter)

其中，本文示例代码名称：

- `charpter4`：Spring Data Redis 入门