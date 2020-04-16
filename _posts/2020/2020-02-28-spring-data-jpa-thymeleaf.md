---
layout: post
title: 【Spring Data 系列学习】Spring Data JPA + Thymeleaf 简单使用 
category: springboot
tags: [springboot]
keywords: Spring Boot,spring data,JPA
excerpt: Spring Data JPA + Thymeleaf 简单使用 
---

# 【Spring Data 系列学习】Spring Data JPA + Thymeleaf 简单使用 

## Thymeleaf 简介

Thymeleaf 是一个跟 Velocity、FreeMarker 类似的模板引擎，它可以完全替代 JSP 。相较与其他的模板引擎，它有如下三个极吸引人的特点：

- `Thymeleaf ` 在有网络和无网络的环境下皆可运行，即它可以让美工在浏览器查看页面的静态效果，也可以让程序员在服务器查看带数据的动态页面效果。这是由于它支持 html 原型，然后在 html 标签里增加额外的属性来达到模板 + 数据的展示方式。浏览器解释 html 时会忽略未定义的标签属性，所以 thymeleaf 的模板可以静态地运行；当有数据返回到页面时，Thymeleaf 标签会动态地替换掉静态内容，使页面动态显示。
- `Thymeleaf ` 开箱即用的特性。它提供标准和 Spring 标准两种方言，可以直接套用模板实现 JSTL、 OGNL 表达式效果，避免每天套模板、改 JSTL、改标签的困扰。同时开发人员也可以扩展和创建自定义的方言。
- `Thymeleaf` 提供 Spring 标准方言和一个与 SpringMVC 完美集成的可选模块，可以快速的实现表单绑定、属性编辑器、国际化等功能。

## 搭建项目

创建一个 `Spring boot` 项目，在 `pom.xml ` 加入需要使用的依赖，如下： 

```xml
<!--web-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--   引入  jpa-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<!--mysql 驱动  -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!--thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

## 配置文件 application.properties

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/test?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=123456
# 打印 sql 语句
spring.jpa.show-sql= true
# 自动创建表
spring.jpa.properties.hibernate.hbm2ddl.auto=update
# 默认创建的mysql表为  MyISAM 引擎修改为InnoDB问题
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL55Dialect

# thymeleaf 基本配置
# 是否开启缓存，开发时关闭缓存,不然没法看到实时页面
spring.thymeleaf.cache=false
# 模板默认文件位置，可修改
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.servlet.content-type=text/html
# 模板后缀
spring.thymeleaf.suffix=.html
```

## 实体类映射数据库表

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

## 服务层

**UserJpaRepository  **

```java
public interface UserJpaRepository extends JpaRepository<User,Long> {
}
```

**数据接口 UserService**

```java
public interface UserService {

    /**
     * 查询所有数据
     * @return
     */
    List<User> list();

    /**
     * 根据 id 查找
     * @param id
     * @return
     */
    User findById(Long id);

    /**
     * 保存用户
     * @param user
     * @return
     */
    User save(User user);

    /**
     * 更新用户
     * @param user
     * @return
     */
    User update(User user);


    /**
     * 删除
     * @param id
     */
    void deleteById(Long id);


}

```

**数据实现类 UserServiceImpl**

```java
@Service
public class UserServiceImpl implements UserService {

    /**
     * ⽇志对象
     */
    private Logger logger = LoggerFactory.getLogger(UserServiceImpl.class);

    @Autowired
    private UserJpaRepository userJpaRepository;


    /**
     * 查询所有数据
     *
     * @return
     */
    @Override
    public List<User> list() {
        return userJpaRepository.findAll();
    }

    /**
     * 根据 id 查找
     *
     * @param id
     * @return
     */
    @Override
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
    public void deleteById(Long id) {
        logger.info("id = {} 数据库数据删除 ", id);
        userJpaRepository.deleteById(id);
    }
}

```

## 表现层

**UserController**

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @Autowired
    private UserService userService;

    @GetMapping("/list")
    public String list(Model model){
        model.addAttribute("userList", userService.list());
        return "list";
    }


    @GetMapping("/toAdd")
    public String toAdd(){
        return "add";
    }


    @GetMapping("/toUpdate")
    public String toUpdate(Long id,Model model){
        model.addAttribute("user", userService.findById(id));
        return "update";
    }

    @PostMapping("/add")
    public String add(User user){
        userService.save(user);
        return "redirect:/user/list";
    }

    @PostMapping("/update")
    public String update(User user){
        userService.save(user);
        return "redirect:/user/list";
    }

    @GetMapping("/delete")
    public String delete(Long id){
        userService.deleteById(id);
        return "redirect:/user/list";
    }

}

```

- `return"list";` 代表会直接去 resources 目录下找相关的页面。
- `return"redirect:/user/list";`  重定向到到对应的 Controller 路径。

## 页面展示

**list.html**

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户列表</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}">
</head>
<body>
<div >
    <table class="table table-striped center-block">
        <h1>用户列表</h1>
        <a th:href="@{/user/toAdd}" class="btn btn-info">新增</a>
        <thead>
        <tr>
            <th>id</th>
            <th>用户名</th>
            <th>年龄</th>
            <th>邮箱</th>
            <th>操作</th>
        </tr>
        </thead>
        <tbody>
            <tr th:each="user:${userList}">
                <td scope="row" th:text="${user.id}"></td>
                <td th:text="${user.name}"></td>
                <td th:text="${user.age}"></td>
                <td th:text="${user.email}"></td>
                <td ><a th:href="@{/user/toUpdate(id=${user.id})}" >编辑</a></td>
                <td ><a th:href="@{/user/delete(id=${user.id})}" >删除</a></td>
            </tr>

        </tbody>
    </table>

</div>
</body>
</html>
```

运行效果图：

![image]( https://mtcarpenter.oss-cn-beijing.aliyuncs.com/images/image-202003011432395012.png)

**add.html**

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户新增</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}">
</head>
<body>
<div style="margin-left: 10%;">
    <form  class="form-horizontal col-sm-4"   th:action="@{/user/add}"  method="post">
        <div class="form-group" >
            <label for="name" >name</label>
            <input type="text" class="form-control" id="name" name="name" placeholder="用户名">
        </div>
        <div class="form-group">
            <label for="age">age</label>
            <input type="number" class="form-control" id="age" name="age" placeholder="年龄">
        </div>
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" class="form-control" id="email" name="email" placeholder="Email">
        </div>
        <button type="submit" class="btn btn-default">提交</button>
    </form>

</div>
</body>
</html>
```

![image-20200308163656618]( https://mtcarpenter.oss-cn-beijing.aliyuncs.com/images/image-20200308163656618.png)

**update.html**

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org">
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户修改</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.css}">
</head>
<body>
<div style="margin-left: 10%;">
    <form  class="form-horizontal col-sm-4"   th:action="@{/user/update}" th:object="${user}"  method="post">
        <div class="form-group" >
            <label for="name" >name</label>
            <input type="text" class="form-control" id="name" name="name" th:value="*{name}" placeholder="用户名">
            <input type="hidden" class="form-control"  name="id" th:value="*{id}">
        </div>
        <div class="form-group">
            <label for="age">age</label>
            <input type="number" class="form-control" id="age" name="age" th:value="*{age}"  placeholder="年龄">
        </div>
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" class="form-control" id="email" name="email" th:value="*{email}" placeholder="Email">
        </div>
        <button type="submit" class="btn btn-default">提交</button>
    </form>

</div>
</body>
</html>
```

![image-20200308163636986]( https://mtcarpenter.oss-cn-beijing.aliyuncs.com/images/image-20200308163636986.png)



## 文章参考

- *http://www.thymeleaf.org*

## 代码示例

本文示例代码访问下面查看仓库：

- *Github：* *https://github.com/mtcarpenter/spring-data-chapter*

其中，本文示例代码名称：

- `charpter6`：Spring Data JPA + Thymeleaf 简单使用