---
layout: post
title: 【Spring Data 系列学习】Spring Data JPA @Query 注解查询
category: springboot
tags: [springboot]
keywords: Spring Boot,spring data,JPA
---



# 【Spring Data 系列学习】Spring Data JPA @Query 注解查询

前面的章节讲述了 Spring Data Jpa 通过声明式对数据库进行操作，上手速度快简单易操作。但同时 JPA 还提供通过注解的方式实现，通过将 `@Query`  注解在继承 repository 的接口类方法上 。

**Query 源码讲解**

```java
public @interface Query {
    /**
    * 指定 JPQL 的查询语句。（nativeQuery = true）是原生的 SQL 语句.
	*/
    String value() default "";
	/**
	* 指定 count 的 JPQL 语句，如果不指定将根据 query 自动生成。
	* （nativeQuery = true 的时候，是原生查询的 SQL 语句）
	*/
    String countQuery() default "";
    /**
    *根据那个字段来 count，一般默认即可。
    */
    String countProjection() default "";
    /**
    * 默认是 false，表示 value 里面是不是原生的 SQL 语句
    */
    boolean nativeQuery() default false;
    /**
    * 可以指定一个 query 的名字，必须是唯一的。
    * 如果不指定，默认的生成规则是
    * {$domainClass}.${queryMethodName}
    */
    String name() default "";
    /**
    * 可以指定一个 count 的query 名字，必须是唯一的。
    * 如果不指定，默认的生成规则是：
    * {$domainClass}.${queryMethodName}.count
    */
    String countName() default "";
}

```

## 快速上手

**项目中的`pom.xml`、`application.properties`与 Chapter1 相同**

**实体类映射数据库表**

**user 实体类**

```java
@Entity
@Table(name = "t_user")
public class User implements Serializable {

    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "u_name")
    private String name;

    @Column(name ="u_age")
    private Integer age;
    
    @Column(name ="u_email")
    private String email;

    // 省略构造器 set/get		

}
```

`@Entity`:定义对象将会成为被JPA管理的实体，将映射到指定的数据库表。

`@Table` :指定数据库的表名。

`@Column`:定义该属性对应数据库中的列名。

`@Id` 定义属性为数据库的主键，一个实体里面必须有一个。

`@GeneratedValue(strategy = GenerationType.IDENTITY)` 自增长 ID 策略

生成如下：

![image-20200301143239501](https://mtcarpenter.oss-cn-beijing.aliyuncs.com/images/image-20200301143239501.png)



## @Query 查询

### 基本使用

**继承 UserQueryRepository**

```java

public interface UserQueryRepository extends JpaRepository<User, Long> {

    /**
     *  语句中 User 查询数据表的类名，?1 括号代表第一个参数
     */
    @Query(name = "select  * from User where name = ?1")
    List<User> findByName(String name);

     /**
     * Sort 排序
     *  根据姓名模糊查询排序
     */
    @Query("select u from User u where u.name like ?1%")
    List<User> findByAndSort(String name, Sort sort);
    
    /**
     * @Transactional 事务的支持 ，@Modifying 用于修改查询
     * @param name 对应 ?1
     * @param id 对应 ?2
     * @return
     */
    @Transactional
    @Modifying
    @Query("update User u set u.name = ?1 where u.id = ?2")
    int updateById(String  name, Long id);


}
```

### @Param用法

```java
    /**
     *  param 对象
     * @param name
     * @param age
     * @return
     */
    @Query(value = "select  u from User u where u.name = :name and u.age = :age")
    List<User> queryParamByNameAndAge(@Param("name") String name,@Param("age") Integer age);


    /**
     *  传一个对象
     * @param user
     * @return
     */
    @Query(value = "select  u from User u  where u.name = :#{#user.name} and u.age = :#{#user.age}")
    List<User> queryObjectParamByNameAndAge(@Param("user") User user);
```

- `:name ` 对应 `@Param`中的 name。
- `:age` 对应 `@Param`中的 age。

- `:#{#user.name}` : 对象中的参数使用方法

### SpEL表达式

```java
@Query("select u from #{#entityName} u where u.lastname = ?1")
 List<User> findByLastname(String lastname);
```

- `entityName`: 根据指定的 Repository 自动插入相关的 entityName。有两种方式能被解析出来：
  - 如果定义了 `@Entity` 注解，直接用其属性名。
  - 如果没有定义，直接用实体类的名称。

### 原生 SQL

```java
    @Query(value = "select * from t_user  where u_name = :name",nativeQuery = true)
    List<User> queryNativeByName(@Param("name") String name);
```

- `nativeQuery`: 为 true 开启。开启之后字段则需要对应的数据库中的表名和字段。

**CURD 测试类**

> 路径：src/test/java/com/mtcarpenter/repository/UserQueryRepositoryTest.java

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserQueryRepositoryTest {

    /**
     * ⽇志对象
     */
    private Logger logger = LoggerFactory.getLogger(UserQueryRepositoryTest.class);

    @Autowired
    private UserQueryRepository userQueryRepository;


    @Before
    public void save() {
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("小米", 9, "a@qq.com")));
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("张三", 16, "b@qq.com")));
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("三哥", 12, "c@qq.com")));
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("米二", 13, "e@qq.com")));
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("阿三", 12, "f@qq.com")));
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("张三", 12, "g@qq.com")));
        logger.info("新增数据 result = {}", userQueryRepository.save(new User("米二", 8, "h@qq.com")));
    }

    /**
     * 基本使用
     */
    @Test
    public void test() {
        logger.info("@query 查询张三 result = {}", userQueryRepository.findByName("张三"));
        logger.info("根据姓名模糊查询排序 result = {}", userQueryRepository.findByNameAndSort("米", new Sort(Sort.Direction.ASC,"age")));
        logger.info("修改 id = 1 的name ，result ={ }", userQueryRepository.updateById("红米", 1L));
    }

    /**
     *  param 参数使用
     */
    @Test
    public void paramTest(){
        logger.info("@param 使用方法  result = {}",userQueryRepository.queryParamByNameAndAge("张三", 12));
        User user = new User();
        user.setName("张三");
        user.setAge(12);
        logger.info("@Param 对象 result = {}", userQueryRepository.queryObjectParamByNameAndAge(user));
    }

    /**
     * SpEl 使用
     */
    @Test
    public void spELTest(){
        logger.info("SpEL 使用方法  result = {}",userQueryRepository.queryELByName("张三"));
    }

    /**
     * 原生查询
     */
    @Test
    public void nativeTest(){
        logger.info("原生查询 使用方法  result = {}",userQueryRepository.queryNativeByName("张三"));

    }
}

```

## 代码示例

本文示例代码访问下面查看仓库：

- *github：* [https://github.com/mtcarpenter/spring-data-chapter](https://github.com/mtcarpenter/spring-data-chapter)
- *gitee* :      [https://gitee.com/mtcarpenter/spring-data-chapter](https://gitee.com/mtcarpenter/spring-data-chapter)

其中，本文示例代码名称：

- `charpter3`：Spring Data JPA @Query 注解查询