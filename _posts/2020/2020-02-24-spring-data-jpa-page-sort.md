---
layout: post
title: 【Spring Data 系列学习】Spring Data JPA 自定义查询，分页，排序，条件查询
category: springboot
tags: [springboot]
keywords: Spring Boot,spring data,JPA
excerpt: Spring Data JPA 自定义查询，分页，排序，条件查询
---

# 【Spring Data 系列学习】Spring Data JPA 自定义查询，分页，排序，条件查询

Spring Boot Jpa 默认提供 CURD 的方法等方法，在日常中往往时无法满足我们业务的要求，本章节通过自定义简单查询案例进行讲解。

## 快速上手

**项目中的pom.xml、`application.properties`与 Chapter1 相同**

**实体类映射数据库表**

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

## 自定义简单查询

spring data 内部基础架构中有个根据方法名的查询生成器机制，对于在存储库的实体上构建约束查询很有用。该机制方法的前缀有find…By、read…By、query…By、count…By和get…By，从这些方法可以分析它的其余部分（实体里面的字段）。引入子句可以包含其他表达式，例如在Distinct要创建的查询上设置不同的标志。然而，第一个By作为分隔符来指示实际标准的开始。在一个非常基本的水平上，你可以定义实体性条件，并与它们串联（And和Or）。

> 注：此段来自 《Spring Data JPA 从入门到精通》。

**继承 PagingAndSortingRepository**

```java
public interface UserPagingRepository extends PagingAndSortingRepository<User, Long> {
    // 通过姓名查找
    List<User> findByName(String name);

    // 通过姓名查找
    List<User> queryByName(String name);

    // 通过姓名或者邮箱查找
    List<User> findByNameOrEmail(String name,String email);

    // 计算某一个 age 的数量
    int countByAge(int age);
}

```

**测试类**

> 路径：src/test/java/com/mtcarpenter/chapter2/repository/UserPagingRepositoryTest.java

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserPagingRepositoryTest {

    /**
     * ⽇志对象
     */
    private Logger logger = LoggerFactory.getLogger(UserPagingRepositoryTest.class);

    @Autowired
    private UserPagingRepository userPagingRepository;


    @Before
    public void save() {
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("小米", 9,"a@qq.com")));
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("张三", 16,"b@qq.com")));
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("三哥", 12,"c@qq.com")));
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("米二", 13,"e@qq.com")));
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("阿三", 12,"f@qq.com")));
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("张三", 12,"g@qq.com")));
        logger.info("新增数据 result = {}", userPagingRepository.save(new User("米二", 8,"h@qq.com")));
    }


    @Test
    public void find(){
        logger.info("通过姓名查找(findByName) result = {}", userPagingRepository.findByName("张三"));
        logger.info("通过姓名查找(queryByName) result = {}", userPagingRepository.queryByName("张三"));
        logger.info("通过姓名或者邮箱(findByNameOrEmail)  查找 result = {}", userPagingRepository.findByNameOrEmail("张三","f@qq.com"));
        logger.info("通过某一个 age 的数量(countByAge) result = {}", userPagingRepository.countByAge(12));
    }


}

```

`@Before`会在`@test`之前运行。

输出日志：

```java
Hibernate: select user0_.id as id1_0_, user0_.age as age2_0_, user0_.email as email3_0_, user0_.name as name4_0_ from user user0_ where user0_.name=?
通过姓名查找(findByName) result = [User{id=2, name='张三', age=16, email='b@qq.com'}, User{id=6, name='张三', age=12, email='g@qq.com'}]
    
Hibernate: select user0_.id as id1_0_, user0_.age as age2_0_, user0_.email as email3_0_, user0_.name as name4_0_ from user user0_ where user0_.name=?
通过姓名查找(queryByName) result = [User{id=2, name='张三', age=16, email='b@qq.com'}, User{id=6, name='张三', age=12, email='g@qq.com'}]
    
Hibernate: select user0_.id as id1_0_, user0_.age as age2_0_, user0_.email as email3_0_, user0_.name as name4_0_ from user user0_ where user0_.name=? or user0_.email=?
通过姓名或者邮箱(findByNameOrEmail)  查找 result = [User{id=2, name='张三', age=16, email='b@qq.com'}, User{id=5, name='阿三', age=12, email='f@qq.com'}, User{id=6, name='张三', age=12, email='g@qq.com'}]
    
Hibernate: select count(user0_.id) as col_0_0_ from user user0_ where user0_.age=?
通过某一个 age 的数量(countByAge) result = 3

```

日志比较冗余删除了多余日志，从日志中我们可以发现 JPA 根据我们定义的接口方法自动解析成 SQL

**方法中支持的关键字如下**

| 关键字                 | 示例                                                         | JPQL 表达式                                                  |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `And`                  | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                   | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is`, `Equals`         | `findByFirstname`,<br/>`findByFirstnameIs`,<br/>`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`              | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`             | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`        | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`          | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`     | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`                | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`               | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`, `Null`       | `findByAge(Is)Null`                                          | `… where x.age is null`                                      |
| `IsNotNull`, `NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`                 | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`              | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`         | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`           | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`           | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`              | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`                  | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                   | `findByAgeIn(Collection ages)`                               | `… where x.age in ?1`                                        |
| `NotIn`                | `findByAgeNotIn(Collection ages)`                            | `… where x.age not in ?1`                                    |
| `True`                 | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`                | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`           | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

## 分页和排序

数据分页和排序在日常也是必不可少的，在 Spring Boot Jpa 中使用分页和排序，需要在Repository 接口的方法中，传入`Pageable` 实例 。

```java
public interface UserPagingRepository extends PagingAndSortingRepository<User, Long> {
   // 通过姓名条件查询
    List<User> findByName(String name, Pageable pageable);

}
```

**测试方法**

```java
    @Test
    public void pageAndSort(){
        Sort sort = new Sort(Sort.Direction.DESC, "age");
        int page = 0;
        int size = 10;
        Pageable pageable = PageRequest.of(page, size, sort);
        logger.info("条件查询 result = {}", userPagingRepository.findByName("张三",pageable));
        logger.info("---------------------------------");
        logger.info("根据年龄排序 result = {}", userPagingRepository.findAll(sort));
    }
```

**测试结果**

```java
2020-02-29 17:02:37.431  INFO 48944 --- [           main] c.m.c.r.UserPagingRepositoryTest         : 条件查询 result = [User{id=2, name='张三', age=16, email='b@qq.com'}, User{id=6, name='张三', age=12, email='g@qq.com'}]
2020-02-29 17:02:37.431  INFO 48944 --- [           main] c.m.c.r.UserPagingRepositoryTest         : ---------------------------------
Hibernate: select user0_.id as id1_0_, user0_.age as age2_0_, user0_.email as email3_0_, user0_.name as name4_0_ from user user0_ order by user0_.age desc
2020-02-29 17:02:37.459  INFO 48944 --- [           main] c.m.c.r.UserPagingRepositoryTest         : 根据年龄排序 result = [User{id=2, name='张三', age=16, email='b@qq.com'}, User{id=4, name='米二', age=13, email='e@qq.com'}, User{id=3, name='三哥', age=12, email='c@qq.com'}, User{id=5, name='阿三', age=12, email='f@qq.com'}, User{id=6, name='张三', age=12, email='g@qq.com'}, User{id=1, name='小米', age=9, email='a@qq.com'}, User{id=7, name='米二', age=8, email='h@qq.com'}]
```

## 复杂条件查询

前面演示了 `CrudRepository` 和 `PagingAndSortingRepository` ，下面通过继承 `JpaRepository` 和`JpaSpecificationExecutor` 操作更复杂的语句。

> JpaSpecificationExecutor 是 JPA 2.0 提供的Criteria API，可以用于动态生成query。Spring Data JPA 支持 Criteria 查询，可以很方便地使用，足以应付工作中的所有复杂查询的情况了，可以对 JPA 实现最大限度的扩展。《spring data Jpa 从入门到精通》                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             

```java
public interface JpaSpecificationExecutor<T> {
    // 根据 Specification 条件查询单个对象
    Optional<T> findOne(@Nullable Specification<T> var1);
	// 根据 Specification 条件查询 返回 List 结果
    List<T> findAll(@Nullable Specification<T> var1);
	// 根据 Specification 条件 和 分页条件查询
    Page<T> findAll(@Nullable Specification<T> var1, Pageable var2);
	// 根据 Specification 条件 和 排序条件查询 返回 List 结果
    List<T> findAll(@Nullable Specification<T> var1, Sort var2);
	// 根据 Specification 条件查询数量
    long count(@Nullable Specification<T> var1);
}

```

**UserJpaRepository 数据层接口**

```java
public interface UserJpaRepository extends JpaRepository<User,Long>, JpaSpecificationExecutor {
}
```

**测试类**

> 路径：src/test/java/com/mtcarpenter/chapter2/repository/UserJpaRepositoryTest.java

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserJpaRepositoryTest {


    @Test
    public void specification() {
        Specification specification = new Specification<User>() {
            @Override
            public Predicate toPredicate(Root root, CriteriaQuery query, CriteriaBuilder cb) {
                // like 模糊查询 ， root.get("name") 属性名  "%三%" 为三
                Predicate p1 = cb.like(root.get("name"), "%三%");
                // greaterThan 表示 age 大于 10
                Predicate p2 = cb.greaterThan(root.get("age"), 10);
                // cb.and(p1, p2) ，and 则表示 p1 和 p2 并且关系，除了 and 还有or, not等。点击  CriteriaBuilder 可进行查看
                return cb.and(p1, p2);
            }
        };

    }


    @Test
    public void ConditionalQuery() {
        int page = 0;
        int size = 10;
        Pageable pageable = PageRequest.of(page, size);
        // 模拟传入的条件
        User user = new User("三", 10, "b@qq.com");
        Specification specification = new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
                List<Predicate> predicates = new ArrayList<>();
                // 判断传入的值是否为空
                if (!"".equals(user.getName())) {
                    predicates.add(cb.like(root.get("name"), "%" + user.getName() + "%"));
                }
                // 判断年龄是否为空
                if (user.getAge() != null) {
                    predicates.add(cb.greaterThan(root.get("age"), user.getAge()));
                }

                return cb.and(predicates.toArray(new Predicate[predicates.size()]));
            }
        };
        Page result = userJpaRepository.findAll(specification, pageable);
        logger.info("条件查询 result = {}", result.getContent());
    }

}
```

`specification()` 方法更容易理解，如果看懂了此方法，有利于更了解`ConditionalQuery()` 方法。`ConditionalQuery()` 方法这种模式也是在实际开发中，使用的频率比较高的方法。

JpaSpecificationExecutor 通过 CriteriaQuery 几乎可以实现任何逻辑了。

## 代码示例

本文示例代码访问下面查看仓库：

- *Github：* https://github.com/mtcarpenter/spring-data-chapter

其中，本文示例代码名称：

- `charpter2`：Spring Data JPA 自定义查询，分页，排序，条件查询