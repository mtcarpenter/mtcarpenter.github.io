---
layout: post
title: 【Spring Cloud Alibaba 微服务教程】Sentinel 快速入门
category: springcloud
tags: [springcloud]
keywords: Spring Boot,Spring Cloud Alibaba,sentinel
excerpt:  Sentinel 快速入门
---

# Sentinel 快速入门

## Sentinel: 分布式系统的流量防卫兵

## Sentinel 是什么？

随着微服务的流行，服务和服务之间的稳定性变得越来越重要。Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:

- **丰富的应用场景**：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
- **完备的实时监控**：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
- **广泛的开源生态**：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
- **完善的 SPI 扩展点**：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。

Sentinel 的主要特性：

![image-20200418132332932.png](http://mtcarpenter.oss-cn-beijing.aliyuncs.com/2020/image-20200418132332932.png)

Sentinel 的开源生态：

![image-20200418132405126.png](http://mtcarpenter.oss-cn-beijing.aliyuncs.com/2020/image-20200418132405126.png)

Sentinel 分为两个部分:

- 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
- 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

![d5116b71-8c74-dd50-7fc9-298839566a70.png](http://mtcarpenter.oss-cn-beijing.aliyuncs.com/2020/d5116b71-8c74-dd50-7fc9-298839566a70.png)

Sentinel 的核心模块说明如下：

- sentinel-core
  Sentinel 核心模块，实现限流、熔断等基本能力。

- sentinel-dashboard
  Sentinel 可视化控制台，提供基本的管理界面，配置限流、熔断规则等，展示监控数据等。

- sentinel-adapter
  Sentinel 适配，Sentinel-core 模块提供的是限流等基本API，主要是提供给应用自己去显示调用，对代码有侵入性，故该模块对主流框架进行了适配，目前已适配的模块如下：

- - sentinel-apache-dubbo-adapter
    对 Apache Dubbo 版本进行适配，这样应用只需引入 sentinel-apache-dubbo-adapter 包即可对 dubbo 服务进行流控与熔断，大家可以思考会利用 Dubbo 的哪个功能特性。
  - sentinel-dubbo-adapter
    对 Alibaba Dubbo 版本进行适配。
  - sentinel-grpc-adapter
    对 GRPC 进行适配。
  - sentinel-spring-webflux-adapter
    对响应式编程框架 webflux 进行适配。
  - sentinel-web-servlet
    对 servlet 进行适配，例如 Spring MVC。
  - sentinel-zuul-adapter
    对 zuul 网关进行适配。

- sentinel-cluster
  提供集群模式的限流与熔断支持，因为通常一个应用会部署在多台机器上组成应用集群。

- sentinel-transport
  网络通讯模块，提供 Sentinel 节点与 sentinel-dashboard 的通讯支持，主要有如下两种实现。

- - sentinel-transport-netty-http
    基于 Netty 实现的 http 通讯模式。
  - sentinel-transport-simple-http
    简单的 http 实现方式。

- sentinel-extension
  Sentinel 扩展模式。主要提供了如下扩展(高级)功能：

- - sentinel-annotation-aspectj
    提供基于注解的方式来定义资源等。
  - sentinel-parameter-flow-control
    提供基于参数的限流（热点限流）。
  - sentinel-datasource-extension
    限流规则、熔断规则的存储实现，默认是存储在内存中。
  - sentinel-datasource-apollo
    基于 apollo 配置中心实现限流规则、熔断规则的存储，动态推送生效机制。
  - sentinel-datasource-consul
    基于 consul 实现限流规则、熔断规则的存储，动态推送生效机制。
  - sentinel-datasource-etcd
    基于 etcd 实现限流规则、熔断规则的存储，动态推送生效机制。
  - sentinel-datasource-nacos
    基于 nacos 实现限流规则、熔断规则的存储，动态推送生效机制。
  - sentinel-datasource-redis
    基于 redis 实现限流规则、熔断规则的存储，动态推送生效机制。
  - sentinel-datasource-spring-cloud-config
    基于 spring-cloud-config 实现限流规则、熔断规则的存储，动态推送生效机制。
  - sentinel-datasource-zookeeper
    基于 zookeeper 实现限流规则、熔断规则的存储，动态推送生效机制。

## sentinel-dashboard 控制台

### 下载 Sentinel 控制台

- 下载地址：https://github.com/alibaba/Sentinel/releases

- 本系列使用得版本为：1.6.3

> 下载`sentinel-dashboard`控制台，首先看下当前项目引入得 sentinel 版本，线上环境最好一一对应。

### 启动 Sentinel 控制台

- 快速启动

```sh
java -jar sentinel-dashboard-1.6.0.jar
```

默认启动访问地址 http://localhost:8080/#/dashboard/home ，启动成功界面如下:

![image-20200418134615512.png](http://mtcarpenter.oss-cn-beijing.aliyuncs.com/2020/image-20200418134615512.png)

- 配置启动

```
java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```

 `-Dserver.port=8080` 用于指定 Sentinel 控制台端口为 `8080`。

从 Sentinel 1.6.0 起，Sentinel 控制台引入基本的**登录**功能，默认用户名和密码都是 `sentinel`。可以参考 [鉴权模块文档](https://github.com/alibaba/Sentinel/wiki/控制台#鉴权) 配置用户名和密码。

### 鉴权模块如下

从 Sentinel 1.5.0 开始，控制台提供通用的鉴权接口 [AuthService](https://github.com/alibaba/Sentinel/blob/master/sentinel-dashboard/src/main/java/com/alibaba/csp/sentinel/dashboard/auth/AuthService.java)，用户可根据需求自行实现。

从 Sentinel 1.6.0 起，Sentinel 控制台引入基本的**登录**功能，默认用户名和密码都是 `sentinel`。

用户可以通过如下参数进行配置：

- `-Dsentinel.dashboard.auth.username=sentinel` 用于指定控制台的登录用户名为 `sentinel`；
- `-Dsentinel.dashboard.auth.password=123456` 用于指定控制台的登录密码为 `123456`；如果省略这两个参数，默认用户和密码均为 `sentinel`；
- `-Dserver.servlet.session.timeout=7200` 用于指定 Spring Boot 服务端 session 的过期时间，如 `7200` 表示 7200 秒；`60m` 表示 60 分钟，默认为 30 分钟；

同样也可以直接在 Spring properties 文件中进行配置。

> 注意：部署多台控制台时，session 默认不会在各实例之间共享，这一块需要自行改造。



##  创建项目

创建一个命名为： `sentinel-cloud` 的 Spring Boot 应用，

### 1、添加依赖

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.13.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.mtcarpenter</groupId>
    <artifactId>sentinel-cloud-example</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sentinel-cloud</name>
    <properties>
        <java.version>1.8</java.version>
        <alibaba.version>2.1.1.RELEASE</alibaba.version>
        <spring.cloud.version>Greenwich.RELEASE</spring.cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>

        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

        </dependencies>
    </dependencyManagement>

```

### 2、增加配置

在 `bootstrap.properties` 中配置 Nacos server 的地址：

````properties
# 服务名称
spring.application.name=sentinel-example
# 服务端口
server.port=8081
# sentinel dashboard
spring.cloud.sentinel.transport.dashboard=localhost:8080

````

### 3、加注解

在启动类上加入注解，这里暂无注解。

### 4、控制类

```java
@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/echo")
    public String echo(){
        return "mtcarpenter";
    }
}
```

### 5、重启项目

- 访问项目：http://localhost:8081/test/echo

访问项目之后，进入 `sentinel dashboard` 控制台，发现 `sentinel-example` 已经在控制台显示了，如下：

![image-20200418141426547.png](http://mtcarpenter.oss-cn-beijing.aliyuncs.com/2020/image-20200418141426547.png)

## Sentinel 的优势和特性：

- 轻量级，核心库无多余依赖，性能损耗小。

- 方便接入，开源生态广泛。Sentinel 对 Dubbo、Spring Cloud、Web Servlet、gRPC 等常用框架提供适配模块，只需引入相应依赖并简单配置即可快速接入；同时针对自定义的场景 Sentinel 还提供低侵入性的注解资源定义方式，方便自定义接入。

- 丰富的流量控制场景。Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，流控维度包括流控指标、流控效果（塑形）、调用关系、热点、集群等各种维度，针对系统维度也提供自适应的保护机制。

- 易用的控制台，提供实时监控、机器发现、规则管理等能力。

- 完善的扩展性设计，提供多样化的 SPI 接口，方便用户根据需求给 Sentinel 添加自定义的逻辑。

Sentinel 与 Hystrix、resilience4j 的对比：

![](http://mtcarpenter.oss-cn-beijing.aliyuncs.com/2020/ad8d2b43-5d5d-3abd-873c-214a54622aff.png)



## 文章参考

- *https://github.com/alibaba/Sentinel*
- *https://www.cnblogs.com/dingwpmz/p/12482491.html*

## 代码示例

本文示例代码访问下面查看仓库：

- *Github：* https://github.com/mtcarpenter/spring-cloud-learning
- *gitee* :      [https://gitee.com/mtcarpenter/spring-cloud-learning](https://gitee.com/mtcarpenter/spring-cloud-learning)

其中，本文示例代码名称：

- `sentinel-cloud`：sentinel could 入门