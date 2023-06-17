# SpringCloud

> SpringCloud Hoxton.SR1
>
> ​	官档：https://cloud.spring.io/spring-cloud-static/Hoxton.SR1/reference/htmlsingle/
>
> ​	中文：https://www.bookstack.cn/read/spring-cloud-docs/docs-project-QuickStart.md
>
> SpringBoot 2.2.2.RELEASE
>
> ​	官档：https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/htmlsingle/

## 1. 基本概述

### 1.1 微服务架构

微服务是一种软件架构模式。它将单体应用程序根据业务能力拆分为一个个小的程序（微服务），每个微服务可以使用不同的编程语言，不同的数据库等技术，只需保证完成业务逻辑。微服务分别部署在不同的机器上，各自运行在自己的进程内，服务间使用轻量级的机制通信（HTTP RESTFUL  API）。单个微服务的异常、宕机，不会对其他微服务造成影响。

微服务架构的优点：

1. 将复杂的业务拆分成多个业务，彻底的去耦合，利于分工。增加新业务时，只需关注业务逻辑，而不用担心耦合问题
2. 微服务都是独立部署的，如果其中一个宕机了，不会影响整个系统。这方便
3. 各个微服务可以使用不同的编程语言和不同的数据库，提高业务实现的灵活度

微服务架构的缺点：

1. 微服务构建比单体项目复杂
2. 部署比单体项目复杂
3. 服务间使用HTTP 协议通信，通信成本比单体项目高
4. 数据一致性问题

### 1.2 SpringCloud

SpringCloud 简单说是多种微服务架构技术落地实现的集合体，是微服务架构下的一站式解决方案，即微服务全家桶

旧版SpringCloud 全家桶几乎全部组件（Eureka、Feign 等）都停止更新，H 版以后出现许多替代组件，以SpringCloud Alibaba（Nacos，sentinel） 为代表

| 微服务功能 | 技术实现                                        |
| :--------- | :---------------------------------------------- |
| 注册中心   | ~~Eureka~~、Zookeeper、Consul、Nacos*           |
| 服务调用   | ~~Ribbon~~、LoadBalancer、~~Feign~~、OpenFeign* |
| 服务降级   | ~~Hystrix~~、resilience4j、sentinel*            |
| 服务网关   | ~~Zuul~~、~~Zuul2~~、gateway*                   |
| 服务配置   | ~~Config~~、Apolo、Nacos*                       |
| 服务总线   | ~~Bus~~、Nacos*                                 |

> 1. SpringCloud 和SpringBoot 的关系
>
>    SpringBoot 是组成微服务架构的一个微服务单位，SpringCloud 是将由SpringBoot 开发的一个一个的单体微服务整合并管理起来，为他们提供服务发现、断路器、路由等能力。SpringCloud 依赖于SpringBoot，SpringBoot可以单体运行，但是SpringCloud 却不可以
>
> 2. SpringCloud 和SpringBoot 版本匹配
>
>    | Release Train                                                | Boot Version                          |
>    | ------------------------------------------------------------ | ------------------------------------- |
>    | [2020.0.x](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes) aka Ilford | 2.4.x, 2.5.x (Starting with 2020.0.3) |
>    | [Hoxton](https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-Hoxton-Release-Notes) | 2.2.x, 2.3.x (Starting with SR5)      |
>    | [Greenwich](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes) | 2.1.x                                 |
>    | [Finchley](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes) | 2.0.x                                 |
>    | [Edgware](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes) | 1.5.x                                 |
>    | [Dalston](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes) | 1.5.x                                 |

### 1.3 服务治理

所谓服务治理，简单理解就是管理微服务，保证平台整体正常、平稳地运行。服务治理涉及的内容较多，比如鉴权、限流、降级、熔断、监控告警等。这些服务治理功能的实现，底层依赖大量的数据结构和算法。

传统的RPC 远程调用框架中，每个服务与服务之间依赖关系比较复杂，管理困难，需要使用服务治理框架，管理服务与服务之间依赖关系，辅助实现服务调用、负载均衡、容错、服务发现与注册等。

### 1.4 服务注册

微服务架构将功能模块拆分成尽量小的粒度，一个完成的微服务程序必然包含大量的节点，各微服务之间的依赖、调用关系如何保存管理是微服务实现的重要问题。结合之前的Dubbo 学习，服务注册是将所有微服务的信息保存在一个节点或一个集群服务中（zookeeper），这个节点或集群是注册中心。服务发现是微服务客户端连接注册中心，请求获得需要远程调用的微服务的信息，然后使用信息实现远程调用。

### 1.5 服务发现

#### DNS 模式

浏览器通过域名获取网站服务的流程中，DNS 服务器根据域名解析出一个IP 地址，返回IP 地址中对应链接包含的内容。根据特定的标志（域名）来获取我们所需要的服务，这就是服务发现。在微服务的领域，将应用拆分成一个个的微服务之后，服务发现，则变成微服务之间相互获取彼此的信息

然而，在微服务的场景下，使用DNS 服务器作为服务发现的实现存在以下问题

- DNS 服务器不支持动态变更，不能够随着服务的状态变更（上线、下线、故障）而对域名映射变更
- DNS 只支持域名和IP 地址的一一映射，但在微服务的中，很多微服务会部署多个实例，这也就要求标志与服务要有一对多的映射
- DNS 服务无法解决多数据中心的问题

总之，服务发现就是程序如何通过一个标志获取服务列表，并且这个服务列表是能够随着服务的状态而动态变更的

#### 服务发现的模式

目前，服务发现主要有两种模式：客户端模式与服务端模式，两者的本质区别在于，客户端是否保存服务列表信息

客户端模式调用微服务，首先从服务注册中心获取服务列表，再根据调用端本地的负载均衡策略，进行服务调用

服务端模式，调用方直接向服务注册中心进行请求，服务注册中心通过自身负载均衡策略，对微服务进行调用。这个模式下，调用方不需要在自身节点维护服务发现逻辑以及服务注册信息，这个模式相对来说比较类似DNS 模式

### 1.6 CAP原则

CAP 原则又称CAP 定理，指的是在一个分布式系统中， Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼。CAP 原则是NOSQL 数据库的基石。

分布式系统的CAP 理论：理论首先把分布式系统中的三个特性进行了如下归纳：

- 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
- 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
- 分区容忍性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

#### 一致性与可用性的决择编辑

CAP 理论就是说在分布式存储系统中，最多只能实现上面的两点。而由于当前的网络硬件肯定会出现延迟丢包等问题，所以分区容忍性是我们必须需要实现的。所以我们只能在一致性和可用性之间进行权衡，没有NoSQL 系统能同时保证这三点。对于web2.0网站来说，关系数据库的很多主要特性却往往无用武之地。

1. 数据库事务一致性需求

   ​	很多web 实时系统并不要求严格的数据库事务，对读一致性的要求很低，有些场合对写一致性要求并不高，允许实现最终一致性。

2. 数据库的写实时性和读实时性需求

   ​	对关系数据库来说，插入一条数据之后立刻查询，是肯定可以读出来这条数据的，但是对于很多web应用来说，并不要求这么高的实时性，比方说发一条消息之 后，过几秒乃至十几秒之后，我的订阅者才看到这条动态是完全可以接受的。

3. 对复杂的SQL 查询，特别是多表关联查询的需求

   ​	任何大数据量的web 系统，都非常忌讳多个大表的关联查询，以及复杂的数据分析类型的报表查询，特别是SNS 类型的网站，从需求以及产品设计角 度，就避免了这种情况的产生。往往更多的只是单表的主键查询，以及单表的简单条件分页查询，SQL 的功能被极大的弱化了。

#### 取舍策略

CAP三个特性只能满足其中两个，那么取舍的策略就共有三种：

1. **CA without P：**如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但放弃P 的同时也就意味着放弃了系统的扩展性，也就是分布式节点受限，没办法部署子节点，这是违背分布式系统设计的初衷的。传统的关系型数据库RDBMS：Oracle、MySQL 就是CA。

2. **CP without A：**如果不要求A（可用），相当于每个请求都需要在服务器之间保持强一致，而P（分区）会导致同步时间无限延长(也就是等待数据同步完才能正常访问服务)，一旦发生网络故障或者消息丢失等情况，就要牺牲用户的体验，等待所有数据全部一致了之后再让用户访问系统。设计成CP的系统其实不少，最典型的就是分布式数据库，如Redis、HBase等。对于这些分布式数据库来说，数据的一致性是最基本的要求，因为如果连这个标准都达不到，那么直接采用关系型数据库就好，没必要再浪费资源来部署分布式数据库。

3. **AP wihtout C：**要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。典型的应用就如某米的抢购手机场景，可能前几秒你浏览商品的时候页面提示是有库存的，当你选择完商品准备下单的时候，系统提示你下单失败，商品已售完。这其实就是先在 A（可用性）方面保证系统可以正常的服务，然后在数据的一致性方面做了些牺牲，虽然多少会影响一些用户体验，但也不至于造成用户购物流程的严重阻塞。

AP：Eureka

CP：Zookeeper、Consul

### 1.7 负载均衡

* LB 负载均衡(Load Balance)是什么？

  在考虑后端机器承载的情况下保证请求分配的平衡和合理，将请求根据指定的策略平摊地分配到多个服务器，达到系统的HA（高可用），常见的负载均衡有软件Nginx，LVS，硬件F5 等

* 本地负载均衡客户端和Nginx 服务端负载均衡

  Nginx 是服务器负载均衡，客户端所有请求会先交给Nginx ，然后由Nginx 实现转发请求，即负载均衡是由服务端实现

  Ribbon 本地负载均衡，在调用微服务接口时，从注册中心上获取服务列表信息并缓存到JVM 本地，在本地根据策略选择服务器，再进行RPC 远程服务调用

* 集中式和进程内

  集中式LB：在服务的消费方和提供方之间使用独立的LB 设施(硬件如F5 或软件如nginx ),，由该设施负责把访问请求通过某种策略转发至服务的提供方

  进程内LB：将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址

## 2. 项目准备

### 2.1 依赖管理

```xml
<!-- 项目父工程 -->
<modules>
    <module>cloud_provider_payment8001</module>
    <module>cloud_consumer_order80</module>
    <module>cloud_api_commons</module>
    <module>cloud_eureka_server7001</module>
    <module>cloud_eureka_server7002</module>
    <module>cloud_provider_payment8002</module>
</modules>

<!-- 继承提供作用：锁定版本+锁定子模块groupId 和version  -->
<dependencyManagement>
    <dependencies>
        <!-- spring boot 2.2.2 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- SpringCloud Hoxton.SR1 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- SpringCloud alibaba 2.1.0.RELEASE -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.spring.boot.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <optional>true</optional>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 2.2 公共接口

对于多个模块（微服务）都需使用的公共接口，可以将其抽取收集到一个公共模块中，打包后发送至需要的机器用于被微服务依赖。

### 2.3 基本框架

使用支付、消费demo 辅助学习

#### 支付模块

cloud_provider_payment8001 模块

1. 创建module

2. 修改pom

   ```xml
   <parent>
       <artifactId>springCloud</artifactId>
       <groupId>org.pochaiyi</groupId>
       <version>1.0-SNAPSHOT</version>
   </parent>
   <modelVersion>4.0.0</modelVersion>
   
   <!-- 继承pom 文件，不需要标注groupId 和version -->
   <artifactId>cloud_provider_payment8001</artifactId>
   
   <properties>
       <maven.compiler.source>8</maven.compiler.source>
       <maven.compiler.target>8</maven.compiler.target>
   </properties>
   
   <dependencies>
       <!--eureka-client-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
       <!-- commons_api -->
       <dependency>
           <groupId>org.pochaiyi</groupId>
           <artifactId>cloud_api_commons</artifactId>
           <version>${project.version}</version>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
       </dependency>
       <dependency>
           <groupId>org.mybatis.spring.boot</groupId>
           <artifactId>mybatis-spring-boot-starter</artifactId>
       </dependency>
       <dependency>
           <groupId>com.alibaba</groupId>
           <artifactId>druid-spring-boot-starter</artifactId>
       </dependency>
       <!-- mysql-connector-java -->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
       </dependency>
       <!-- jdbc -->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-jdbc</artifactId>
       </dependency>
       <!-- 不需要，使用插件JReBel 代替实现热部署 -->
       <!--<dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>-->
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

3. application 配置

   ```yaml
   server:
     # 微服务端口号
     port: 8001
   
   spring:
     application:
       name: cloud-provider-payment
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://localhost:3306/cloud?useUnicode=true&characterEncoding=utf-8&useSSL=false
       username: root
       password: root
   
   mybatis:
     mapperLocations: classpath:mapper/*.xml
     # 别名类包
     type-aliases-package: com.pochaiyi.entities
   ```
   
4. 业务逻辑

   bean、mapper、service、controller、configuration、springBoot 启动类

   注意实体类需要实现Serializable 接口，实体类传输依赖序列号、反序列化技术

   ```java
   @RestController
   @Slf4j
   public class PaymentController {
       @Value("${server.port}")
       private String serverPort;
   
       @Autowired
       private PaymentService paymentService;
   
       // 接受json 传入
       @PostMapping("/payment/create")
       public CommonResult create(@RequestBody Payment payment)
       {
           int result = paymentService.create(payment);
           log.info("return id: " + result);
           if(result > 0)
           {
               return new CommonResult(200,"insert succeed,port:"+serverPort,result);
           }else{
               return new CommonResult(402,"insert failed",null);
           }
       }
   
       @GetMapping(value = "/payment/get/{id}")
       public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id)
       {
           Payment payment = paymentService.getPaymentById(id);
           log.info("return payment: {}",payment);
           if (payment != null) {
               return new CommonResult(200,"query succeed,port:"+serverPort,payment);
           }else{
               return new CommonResult(402,"query failed: "+id,null);
           }
       }
   }
   ```

#### 消费模块

cloud_consumer_order80 模块

1. 创建module

2. 修改pom

3. application 配置

   ```yaml
   server:
     port: 80
   
   spring:
     application:
       name: cloud_consumer_order
   ```

4. 业务逻辑

   此时未引入服务调用组件，使用spring 组件RestTemplate 直接远程服务接口

   RestTemplate：提供多种便捷访问远程Http 服务的API，是一种简单便捷的访问restful 服务模板类，是Spring 提供的用于访问Rest 服务的客户端模板工具集
   
   ```java
   @Bean
   public RestTemplate restTemplate() {
       return new RestTemplate();
   }
   ```
   
   ```java
   @RestController
   @Slf4j
   public class OrderController {
       public static final String PaymentSrv_URL = "http://localhost:8001";
       
       // RestTemplate：提供多种便捷访问远程Http 服务的API，是一种简单便捷的访问restful 服务模板类，是Spring 提供的用于访问Rest 服务的客户端模板工具集
       @Autowired
       private RestTemplate restTemplate;
   
       // 浏览器客户端发送get 请求，底层实质又发送post 请求调用服务端8001
       @GetMapping("/consumer/payment/create")
       public CommonResult create(Payment payment)
       {
           log.info(payment.getSerial());
           // 参数：(url,请求参数(requestBody),返回类型))
           return restTemplate.postForObject(PaymentSrv_URL + "/payment/create",payment,CommonResult.class);
       }
   
       @GetMapping("/consumer/payment/get/{id}")
       public CommonResult getPayment(@PathVariable Long id)
       {
           return restTemplate.getForObject(PaymentSrv_URL + "/payment/get/"+id, CommonResult.class, id);
       }
   }
   ```

## 3. 注册中心

### 3.1 Eureka（SpringCloud）

SpringCloud 封装Netflix 公司开发的Eureka 模块来实现服务治理，H 版后SpringCloud 不再维护Eureka，但仍可使用，目前主流注册中心为Nacos

Eureka 包含两个组件：EurekaServer、EurekaClient

1. EurekaServer 提供服务注册服务，即注册中心

   各微服务节点通过配置启动后，在EurekaServer 中进行注册，EurekaServer 的服务注册表中会存储所有可用服务节点的信息，服务节点的信息可在web 页面直接浏览

2. EurekaClient通过注册中心进行访问

   是Java 客户端，用于简化与EurekaServer 的交互，客户端内置使用轮询（round-robin）负载算法的负载均衡器。应用启动后，将会向EurekaServer 发送心跳（默认周期30秒）。如果EurekaServer 在多个心跳周期（默认90秒）内没有接收到某个节点的心跳，EurekaServer 将会从服务注册表中把这个服务节点删除

#### 单机搭建

##### 服务端

1. 创建模块

2. 引入依赖（SpringCloud H 版后不再包含Eureka）

   ```xml
   <!-- eureka-server -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
   </dependency>
   ```

3. application 配置

   ```yaml
   server:
     port: 7001
   
   spring:
     application:
       name: cloud_eureka_server
   
   eureka:
     instance:
       # eureka 服务端主机名称，默认localhost，可直接配置为IP 地址
       hostname: localhost
     client:
       # server 不需要注册自己
       register-with-eureka: false
       # server 不需要检索服务
       fetch-registry: false
       service-url:
         # 设置与eurekaServer 交互或者查看web 页面的地址
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

3. 主启动类注解

   @EnableEurekaServer，表明此微服务是EurekaServer

##### 提供者

1. 创建模块

2. 引入依赖

   ```xml
   <!--eureka-client-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

3. application 配置

   ```yaml
   # 追加
   eureka:
     instance:
     	# 微服务主机名称
       hostname: host8001
     	# 微服务实例ID，一个微服务可能有多个实例
       instance-id: payment8001
       # 隐含信息包含IP 地址，默认显示为主机名称
       prefer-ip-address: true
     client:
       # 注册到eurekaServer，默认为true。
       register-with-eureka: true
       # 从eurekaServer 抓取已有注册信息，默认为true
       # 单节点无所谓，集群必须设置为true 才能配合ribbon 使用负载均衡
       fetchRegistry: true
       # eurekaServer 注册地址
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

4. 主启动类注解

   @EnableEurekaClient，表明此微服务是EurekaClient

##### 消费者

步骤同提供者相同

#### 集群搭建

##### 服务端集群

Eureka 服务端集群的节点互相注册，相互守望

1. 创建第二个服务模块

2. application 配置

   ```yaml
   server:
     port: 7002
   
   spring:
     application:
       name: cloud-eureka-server
   
   eureka:
     instance:
       hostname: eureka7002
     client:
       register-with-eureka: false
       fetch-registry: false
       service-url:
         # 向eureka 服务端集群其他节点注册
         defaultZone: http://eureka7001:7001/eureka/
   ```

3. 主启动类注解

   @EnableEurekaServer，表明此微服务是EurekaServer

4. 修改单机搭建的配置

   修改第一台Eureka 服务端application 配置

   ```yaml
   eureka:
     instance:
     	# hosts 配置eureka7001，eureka7002 都映射localhost，以达到在单机中配置两个eurekaServer 并设置不同主机名的目的
       hostname: eureka7001
     client:
       service-url:
         # 向eureka 服务端集群其他节点注册
         defaultZone: http://eureka7002:7002/eureka/
         defaultZone: http://eureka7002:7002/eureka/
   ```

   修改客户端application 的注册地址

   ```yaml
   eureka:
     client:
       service-url:
         # 向服务端集群所有节点注册
         defaultZone: http://eureka7001:7001/eureka,http://eureka7002:7002/eureka
   ```

##### 提供者集群

服务提供者可能发生错误而无法提供服务，为微服务搭建集群（即提供相同服务的微服务实例）可实现高可用性

1. 创建第二个提供者模块

2. 设置步骤同第一个提供者模块相同

   ```yaml
   server:
     port: 8002
   
   spring:
     application:
     	# 相同功能的微服务集群，微服务名称应该相同，实现微服务集群的负载均衡
       name: cloud-provider-payment
     datasource:
       type: com.alibaba.druid.pool.DruidDataSource
       driver-class-name: com.mysql.cj.jdbc.Driver
       url: jdbc:mysql://localhost:3306/cloud?useUnicode=true&characterEncoding=utf-8&useSSL=false
       username: root
       password: root
   
   mybatis:
     mapperLocations: classpath:mapper/*.xml
     type-aliases-package: com.pochaiyi.entities
   
   eureka:
     client:
       register-with-eureka: true
       fetchRegistry: true
       service-url:
         defaultZone: http://eureka7001:7001/eureka,http://eureka7002:7002/eureka
   ```

##### 消费者修改

1. 此时将调用单个微服务修改为调用整个微服务集群

   CLOUD-PROVIDER-PAYMENT 是微服务集群的微服务名称

   ```java
   public class OrderController {
       public static final String PaymentSrv_URL = "http://CLOUD-PROVIDER-PAYMENT";
       ...
   ```

   负载均衡设置

   ```java
   @Bean
   // Ribbon 负载均衡功能，调用集群微服务时必须设置，默认采用轮询方式
   @LoadBalanced
   public RestTemplate restTemplate()
   {
       return new RestTemplate();
   }
   ```

#### 其他功能

##### web 信息

Eureka 服务端web 页面查看注册微服务信息，默认显示格式：主机名称 : 微服务名称

修改客户端微服务的application 配置，更换显示信息为：自定义微服务实例ID，并且连接显示IP地址

```yaml
eureka:
  instance:
    # 微服务主机名称
    hostname: host8001
    # 微服务实例ID，一个微服务可能有多个实例
    instance-id: payment8001
    # 连接显示IP 地址，默认显示为主机名称
    prefer-ip-address: true
```

##### 服务发现

主启动类标注@EnableDiscoveryClient

客户端能使用Eureka 提供的DiscoveryClient 组件获取注册微服务的信息

```java
public class PaymentController {
    // 使用此API 可获取在服务端注册的微服务的信息
    @Autowired
    private DiscoveryClient discoveryClient;

	...

    @GetMapping(value = "/payment/discovery")
    public Object discovery(){
        // 查看所有服务
        List<String> services = discoveryClient.getServices();
        for (String service : services) {
            log.info(service);
        }
        // 查看指定微服务名称的所有实例
        List<ServiceInstance> instances = discoveryClient.getInstances("cloud-provider-payment".toUpperCase());
        for (ServiceInstance instance : instances) {
            log.info(instance.getInstanceId()+"; "+instance.getUri());
        }
        return this.discoveryClient;
    }
}
```

##### 自我保护

自我保护模式正是一种针对网络异常波动的安全保护措施，使用自我保护模式能使Eureka 集群更加的健壮、稳定的运行，目的是防止因为网络不通等原因注销正常运行的微服务

自我保护机制的工作机制：如果在15分钟内超过85%的客户端节点都没有正常的心跳，那么Eureka 就认为客户端与注册中心出现了网络故障，EurekaServer 自动进入自我保护机制，此时会出现以下几种情况：

1. EurekaServer 不再从注册列表中移除因为长时间没收到心跳而应该过期的服务
2. EurekaServer 仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节点依然可用
3. 当网络稳定时，当前EurekaServer 新的注册信息会被同步到其它节点中

自我保护设置（服务端）

关闭自我保护机制，不代表服务端不保存注册信息，服务端注销服务的机制将变为最常见的心跳机制

```yaml
eureka:
  server:
    # 关闭自我保护机制，不可用服务会被注销
    enable-self-preservation: false
    # 清理无效服务的时间间隔，默认60000 毫秒
    eviction-interval-timer-in-ms: 200
```

心跳机制（服务端）

```shell
# eureka 客户端向服务端发送心跳的时间间隔（单位为秒，默认30秒）
    lease-renewal-interval-in-seconds: 1
    # eureka 服务端收到最后一次心跳后等待的时间上限（单位为秒，默认90秒），超时将注销服务
    lease-expiration-duration-in-seconds: 2
```

### 3.2 Zookeeper

在模块cloud_provider_zkTest 实验zookeeper 作为注册中心，注册信息在zookeeper 中存放为临时节点

1. 引入依赖（将eureka-client 替换为zookeeper-discovery）

   ```xml
   <!-- SpringBoot 整合zookeeper 客户端 -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
       <exclusions>
           <exclusion>
               <groupId>org.apache.zookeeper</groupId>
               <artifactId>zookeeper</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <!-- zookeeper 3.5.7 -->
   <dependency>
       <groupId>org.apache.zookeeper</groupId>
       <artifactId>zookeeper</artifactId>
       <version>3.5.7</version>
       <exclusions>
           <exclusion>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
           </exclusion>
       </exclusions>
   </dependency
   ```

2. application 配置

   ```yaml
   spring:
     application:
       name: cloud-provider-payment
     cloud:
       zookeeper:
         connect-string: 192.168.178.100:2181
   ```

3. 主启动类标注@EnableDiscoveryClient

4. 启动微服务，在zookeeper 中查看到新的临时节点

   /services/cloud-provider-payment/***

   ```shell
   [zk: localhost:2181(CONNECTED) 0] ls /services/cloud-provider-payment/
   
   Path must not end with / character
   
   [zk: localhost:2181(CONNECTED) 1] get /services/cloud-provider-payment/595b5cdc-4f0b-41f5-8841-8970ac82470f
   
   # 微服务节点信息
   {"name":"cloud-provider-payment","id":"595b5cdc-4f0b-41f5-8841-8970ac82470f","address":"thinkpad","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1631093993353,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
   [zk: localhost:2181(CONNECTED) 2]
   ```

### 3.3 Consul

Consul 是一套开源的分布式服务发现和配置管理系统，由HashiCorp 公司用Go 语言开发。

提供微服务系统中的服务治理、配置中心、控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用以构建全方位的服务网格，总之Consul 提供了完整的服务网格解决方案。

它具有很多优点。包括： 基于 raft 协议，比较简洁； 支持健康检查, 同时支持HTTP 和DNS 协议 支持跨数据中心的WAN 集群 提供图形界面跨平台，支持Linux、Mac、Windows。

Consul 操作和注册中心配置与zookeeper 相似

1. 引入依赖

   ```shell
    <!-- SpringCloud consul-server -->
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    	<dependency>
    	<groupId>org.springframework.boot</groupId>
   	 <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
   ```
   
2. application 配置

   ```yaml
   spring:
     application:
       name: cloud-consumer-order
     cloud:
       consul:
         host: localhost
         port: 8500
         discovery:
           service-name: ${spring.application.name}
   ```

3. 主启动类标注@EnableDiscoveryClient
4. 浏览器登录consul web 页面http://localhost:8500，查看微服务注册信息

## 4. 服务调用

### 4.1 Ribbon

SpringCloud Ribbon 是基于Netflix Ribbon 实现的客户端负载均衡调用工具，主要提供客户端的软件负载均衡算法和服务调用，还提供一系列完善的配置项如连接超时，重试等

Ribbon 已经进入停更，但仍在大量项目中部署使用。SpringCloud loadbalancer 是SpringCloud 提供的替代产品，但目前仍待优化，现在主要使用OpenFeign 实现服务调用

Ribbon = 负载均衡 + RestTemplate 调用，Ribbon 其实就是一个软负载均衡的客户端组件，可以和其他发送请求的客户端结合使用，与eureka 结合只是其中的一个使用实例

#### RestTemplate

RestTemplate 常用API

* getForObject

  返回指定对象

  ```java
  getForObject(String url,Class<T> responseType,Object... uriVariables)
  
  url：url地址
  responseType：响应实体类型
  uriVariables：url地址参数，没有url 参数时可以不填
  ```

* getForEntity

  参数与getForObject 相同，但其返回对象ResponseEntity<T> 包含响应体、状态码等信息

  可使用ResponseEntity.getBody() 获取与getForObject 相同的信息

* postForObject

  ```java
  postForObject(String url, @Nullable Object request, Class<T> responseType)
      
  url：url地址
  request：请求实体对象
  responseType：响应实体类型
  ```

* postForEntity

#### 负载均衡策略

Ribbon 负载均衡默认使用轮询策略，现实验使用指定策略

Ribbon 全部策略（包路径 com.netflix.loadbalancer）

| 策略                                                         | 包名                      |
| ------------------------------------------------------------ | ------------------------- |
| 轮询                                                         | RoundRobinRule            |
| 随机                                                         | RandomRule                |
| 先轮询，失败则重试获取可用服务                               | RetryRule                 |
| 轮询扩展，响应速度越快的实例选择权重越大                     | WeightedResponseTimeRule  |
| 先过滤多次访问故障而处于断路器跳闸状态的服务，再选择并发量较小的服务 | BestAvailableRule         |
| 先过滤故障服务，再选择并发较小的服务                         | AvailabilityFilteringRule |
| 默认规则，根据server 所在区域的性能和server 的可用性选择     | ZoneAvoidanceRule         |

1. 引入依赖（eureka-client 包含ribbon）

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
   </dependency>
   ```

2. 创建策略配置类

   自定义配置类不要放在spring 包扫描范围内，否则这个配置类会被所有Ribbon 客户端共享，无法达到特殊化定制的目的

   ```java
   package com.ribbonRule;
   
   @Configuration
   public class MyRule {
       @Bean
       public IRule ribbonRule(){
           // 随机策略
           return new RandomRule();}}
   ```

3. 标注主配置类

   设置调用指定服务使用指定的负载均衡策略

   @RibbonClient(name = 服务名称,configuration = 配置类class 对象)

   ```java
   @RibbonClient(name = "CLOUD-PROVIDER-PAYMENT",configuration = MyRule.class)
   public class CloudConsumerOrder80 {
       ...
   ```

### 4.2 OpenFeign

Feign 是SpringCloud 的一个轻量级RESTful 风格的HTTP 服务客户端，Feign 内置Ribbon，用来做客户端负载均衡和服务调用。Feign 使用方式：使用Feign 的注解标注接口，与服务提供的接口绑定，便可通过调用本地接口间接调用远程服务

OpenFeign 是SpringCloud 在Feign 基础上增加对Spring MVC 注解的支持，如@RequesMapping 等。OpenFeign 的注解@FeignClient 可以解析Spring MVC 注解标注的接口，并通过动态代理方式产生实现类，实现类负载均衡并调用服务

#### 基本使用

1. 引入依赖

   ```xml
   <!-- openfeign -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   <!-- eureka client -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. application 配置

   ```yaml
   server:
     port: 80
   
   spring:
     application:
       name: cloud-consumer-openFeign-order80
   
   eureka:
   #  instance:
   #    instance-id: consumerOpenFeignOrder80
   #    prefer-ip-address: true
   #    lease-renewal-interval-in-seconds: 1
   #    lease-expiration-duration-in-seconds: 2
     client:
       register-with-eureka: false
       service-url:
         defaultZone: http://eureka7001:7001/eureka,http://eureka7002:7002/eureka
   ```

3. 标注启动类，开启Feign

   ```java
   @SpringBootApplication
   @EnableEurekaClient
   @EnableFeignClients
   public class CloudConsumerOpenFeignOrder80 {
       ...
   ```

4. 接口绑定：新建接口类 -> 声明方法与提供者暴露接口相同 -> 标注注解并注明服务名称

   基于HTTP 的调用，需要注明服务的ULR，OpenFeign 支持Spring MVC 的注解

   ```java
   @Service
   @FeignClient(value = "CLOUD-PROVIDER-PAYMENT")
   public interface FeignPaymentService {
       @GetMapping(value = "/payment/get/{id}")
       public CommonResult<Payment> getPaymentByIdMap(@PathVariable("id") Long id);
   }
   ```

5. controller

   ```java
   @GetMapping(value = "/feign/payment/getMap/{id}")
   public CommonResult<Payment> getPaymentByIdMap(@PathVariable("id") Long id){
       log.info(serverPort);
       return feignPaymentService.getPaymentByIdMap(id);
   }
   ```

#### 超时控制

默认Feign 客户端调用服务只等待1秒，但是服务端处理请求可能超时，导致Feign 客户端直接返回报错，为避免这样的情况，需要设置Feign 客户端的超时控制

application 配置

```yaml
ribbon:
  # 建立连接的时间，适用于网络状况正常的情况下，两端连接所用的时间
  ReadTimeout: 5000
  # 建立连接后从服务端获取资源所用的时间
  ConnectTimeout: 5000
```

#### 日志打印

Feign 提供日志打印功能，可以通过配置调整日志输出级别，获取Feign 中Http 请求的细节，即对Feign 接口的调用情况进行监控和输出

| 日志级别 | 日志内容                                       |
| -------- | ---------------------------------------------- |
| NONE     | 不显示任何日志，默认                           |
| BASIC    | 记录请求方法、URL、响应状态码及执行时间        |
| HEADERS  | 除BASIC 的信息外，还有请求和响应的头信息       |
| FULL     | 除HEADERS 信息外，还有请求和响应的正文及元数据 |

##### 配置日志

1. 创建@Bean 组件，返回日志级别类

   ```java
   import feign.Logger;
   
   ...
   
   @Bean
   Logger.Level feignLoggerLevel()
   {
       return Logger.Level.FULL;
   }
   ```

2. application 配置

   ```yaml
   logging:
     level:
       # feign 日志以什么级别监控哪个接口
       com.pochaiyi.service.FeignPaymentService: debug
   ```

## 5. 服务降级

* 服务雪崩

  多个微服务之间相互调用，微服务A 调用微服务B 和微服务C，微服务B 和微服务C 又调用其它的微服务，这就是所谓的”扇出“。如果扇出的链路上某个微服务的调用响应时间过长或不可用，对微服务A 的调用就会占用越来越多的系统资源，进而引起系统崩溃，即所谓的“雪崩效应”

* “断路器”是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩

* 什么是服务降级？

  为避免调用方线程被长时间、不必要地占用，故障在分布式系统中的蔓延，乃至雪崩，程序需要做特定设置，在无法即时返回正常的程序结果时，向调用方返回一个符合预期的、可处理的备选响应（FallBack），这便是服务降级

* 必要性

  服务降级在分布式系统中是必要的设置，因为分布式系统中各个微服务的复杂繁多的调用关系，某个服务出现异常或卡顿几乎是不可避免，如果没有即时合理的返回处理，很容易造成系统的崩溃

### 5.1 Hystrix

hystrix 能提供服务降级、服务熔断、即时监控，现在已经停止更新，官方推荐resilience4j（SpringCloud），当前流行sentinel（Alibabab）

服务降级触发场景：程序运行异常、请求等待超时、服务熔断、线程池/信号量打满

#### 基本使用

provider 端与consumer 端都可配置服务降级，service 层和controller 层都可配置降级

一对一的服务与fallback 方法时，返回类型与参数类型必须一致

**provider service **

1. 搭建基本的eureka+feign 的单注册中心，单提供者，单消费者框架

2. 引入依赖

   ```xml
   <!-- hystrix -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

3. 主启动类标注@EnableCircuitBreaker，开启服务降级

   ```java
   import org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker;
   
   @SpringBootApplication
   @EnableEurekaClient
   @EnableDiscoveryClient
   @EnableCircuitBreaker
   public class CloudProviderHystrixPayment {
       ...
   ```

4. 为服务方法单独设置降级措施（service 层）

   @HystrixCommand 开启服务降级

   fallbackMethod 指定同类的某个方法处理服务降级

   @HystrixProperty...,valuecommandProperties={@HystrixProperty(...,value=... 设置服务时间峰值

   ```java
   @HystrixCommand(fallbackMethod = "hystrixTimeOutHandler",
                   commandProperties={@HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="3000")})
   public String hystrixTimeOut(Integer id)
   {
       try {
           TimeUnit.SECONDS.sleep(2);
       } catch (InterruptedException e) {
           e.printStackTrace();
       }
       return Thread.currentThread().getName()+"paymentTimeOut,id: "+id;
   }
   
   public String hystrixTimeOutHandler(Integer id){
       return "hystrixTimeOutHandler 专属服务降级处理方法";
   }
   ```

5. 修改服务方法，添加超时或异常逻辑，浏览器访问可发现返回内容为指定的降级方法

**consumer controller**

1. 引入依赖

2. 主启动类标注@EnableHystrix

3. 使用feign 服务调用时，需要在application 配置中设置支持hystrix

   ```yaml
   feign:
     hystrix:
       enabled: true
   ```

4. controller 层标注@HystrixCommand 和设置fallback 方法

##### 局部处理

1）使用@DefaultProperties 处理类中全部标注@HystrixCommand 的服务降级

2）如果服务方法已配置特定fallback 方法则不修改，否则使用defaultFallback 方法处理降级

3）此时fallback 方法处理类中所有服务的服务降级，没有参数

```java
@RestController
@DefaultProperties(defaultFallback = "HystrixFeignControllerHandler")
public class HystrixFeignController {
    @Autowired
    HystrixFeignService hystrixFeignService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    @HystrixCommand
    public CommonResult<String> paymentOK(@PathVariable("id") Integer id)
    {
        int i = 1/0;
        return hystrixFeignService.paymentOK(id);
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
//    @HystrixCommand(fallbackMethod = "paymentTimeOutHandler",
//            commandProperties={@HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="2000")})
    @HystrixCommand
    public CommonResult<String> paymentTimeOut(@PathVariable("id") Integer id)
    {
        int i = 1/0;
        return hystrixFeignService.paymentTimeOut(id);
    }

    public CommonResult<String> paymentTimeOutHandler(Integer id)
    {
        return new CommonResult(400,
                "consumer controller paymentTimeOutHandler 专属服务降级处理方法");
    }

    public CommonResult<String> HystrixFeignControllerHandler()
    {
        return new CommonResult(400,
                "consumer controller HystrixFeignController 类全局专属服务降级处理方法");
    }
}
```

##### 代码分离

将业务代码与服务降级代码分离，此操作适用在consumer 端

1. 为映射服务的接口创建实现类，在重写方法中编写服务降级代码，实现类标注spring 组件注解

   ```java
   @Service
   public class HystrixFeignServiceFallback implements HystrixFeignService {
       @Override
       public CommonResult<String> paymentOK(Integer id) {
           return new CommonResult(400,"HystrixFeignServiceFallback paymentOK");
       }
   
       @Override
       public CommonResult<String> paymentTimeOut(Integer id) {
           return new CommonResult(400,"HystrixFeignServiceFallback paymentTimeOut");
       }
   }
   ```

2. 服务接口的@FeignClient 注解添加属性fallback 用于指定其处理降级的接口实现类、

   可使用注解@HystrixCommand 为降级服务设置属性

   ```java
   @Service
   @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT",fallback = HystrixFeignServiceFallback.class)
   public interface HystrixFeignService {
       ...
       
       @HystrixCommand(commandProperties={@HystrixProperty(name="execution.isolation.thread.timeoutInMilliseconds",value="5000")})
       @GetMapping("/payment/hystrix/timeout/{id}")
       public CommonResult<String> paymentTimeOut(@PathVariable("id") Integer id);
   ```

#### 服务熔断

* 什么是服务熔断？

  如果短时间内服务降级次数过多，或许可以认为程序已经出现严重问题无法正常提供服务，那么这段时间内进入服务熔断，切断服务的请求，即不经过任何处理直接返回fallback 方法，即使请求内容正确且此时可以正常处理请求。一段时间后将会尝试关闭服务熔断，如果成功，则程序恢复正常提供服务。

  熔断机制是应对雪崩效应的一种微服务链路保护机制，当扇出链路的某个微服务出错或不可用或响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息，检测到该节点微服务调用响应正常后，再恢复调用链路

##### 运作流程

在SpringCloud 框架中，熔断机制通过Hystrix 实现。Hystrix 会监控微服务间调用的状况，当失败的调用达到一定阈值，缺省是5 秒内20 次调用失败，便启动熔断机制

服务降级频繁 -> 熔断 -> 尝试恢复调用链路

* 熔断状态

  熔断打开：请求不再调用服务，打开时长达到所设时钟（默认MTTR 即平均故障处理时间）则进入半熔断状态

  熔断关闭：熔断关闭不会对服务进行熔断

  熔断半开：部分请求根据规则调用服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断

* 熔断的自动关闭？

  hystrix 自动恢复，当断路器打开，对主逻辑进行熔断之后，hystrix 会启动一个休眠时间窗，在这个时间窗内，降级逻辑临时成为主逻辑。当休眠时间窗到期，断路器将进入半开状态，释放一次请求给原来的主逻辑，如果请求正常返回，断路器将闭合，主逻辑恢复，如果依然有问题，断路器继续进入打开状态，休眠时间窗重新计时

##### 基本使用

cloud_provider_hystrix_payment8001 模块

1. service 添加

   时间窗：断路器打开需要统计一些请求和错误数据，而统计的时间范围就是时间窗，默认最近10 秒

   请求总数阀值：在时间窗内，必须满足请求总数阀值才有资格熔断，默认20。意味在10秒内，如果该hystrix 命令的调用不足20次，即使所有的请求都失败，断路器也不会打开

   错误百分比阀值：当请求总数在快照时间窗内超过阀值，比如30次调用中有15次发生异常，即超过50%的错误百分比，在默认设定50%阀值的情况下，断路器将打开

   ```java
   // 服务熔断
   @HystrixCommand(fallbackMethod = "paymentCircuitBreakerHandler",commandProperties = {
       @HystrixProperty(name = "circuitBreaker.enabled",value = "true"), //开启熔断
       @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"), //请求次数阈值
       @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"), //窗口时间
       @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"), //降级百分比阈值
   })
   public String paymentCircuitBreaker(@PathVariable("id") Integer id)
   {
       if(id < 0)
       {
           throw new RuntimeException("id不能是负数");
       }
       String serialNumber = IdUtil.simpleUUID();
       return Thread.currentThread().getName()+"paymentTimeOut id "+serialNumber;
   }
   
   public String paymentCircuitBreakerHandler(@PathVariable("id") Integer id)
   {
       return "id 不能为负 " +id;
   }
   ```

2. controller 添加

   ```java
   @GetMapping("/payment/hystrix/circuit/{id}")
   public CommonResult paymentCircuitBreaker(@PathVariable("id") Integer id)
   {
       String result = hystrixService.paymentCircuitBreaker(id);
       log.info(result);
       return new CommonResult(200,result);
   }
   ```

3. 测试

   http://localhost:8001/payment/hystrix/circuit/{id}，id为负数方法报错调用fallback，短时间内多次调用fallback，再尝试id 为正数的请求，发现仍返回fallback，则成功开启熔断

#### HystrixCommand

@HystrixCommand 属性配置，包含降级、熔断属性

```java
commandProperties = {
    // 隔离策略，THREAD 线程池隔离，SEMAPHORE 信号池隔离
    @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
    // 隔离策略为信号池隔离时，设置信号池大小（最大并发数）
    @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
    // 配置命令执行的超时时间，单位秒
    @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
    // 是否启用超时时间
    @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
    // 执行超时时是否中断
    @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
    // 执行被取消时是否中断
    @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
    // 允许回调方法执行的最大并发数
    @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
    // 服务降级是否启用，是否执行回调函数
    @HystrixProperty(name = "fallback.enabled", value = "true"),
    // 是否启用断路器
    @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
    // 滚动时间窗中，断路器熔断的最小请求数，默认20
    	// 滚动时间窗（默认10秒）内收到19个请求，即使全部失败，断路器也不会打开
    @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
    // 滚动时间窗内，服务降级请求的百分比阈值（已经超过最小请求数），达到则打开断路器
    @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
    // 断路器打开后的休眠时间窗，休眠时间窗结束后断路器进入半开状态，尝试熔断的请求命令，成功则关闭断路器，否则继续打开状态
    @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
    // 断路器强制打开
    @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
    // 断路器强制关闭
    @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
    // 滚动时间窗设置，该时间用于断路器判断健康度时收集信息的持续时间
    @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
    // 设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候根据时间窗长度拆分成多个 “桶”累计各度量值，每个“桶”记录一段时间内的采集指标。比如10 秒内拆分10 个“桶”收集，所以timeinMilliseconds 必须能被numBuckets 整除，否则抛异常
    @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
    // 设置对命令执行的延迟是否使用百分位数跟踪和计算，设置为false，所有的概要统计都将返回 -1。
    @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
    // 设置百分位统计的滚动窗口的持续时间，单位为毫秒。
    @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
    // 设置百分位统计滚动窗口中使用“桶”的数量
    @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
    // 设置执行过程中每个“桶”保留的最大执行次数，如果滚动时间窗内发生超过该设定值的执行数，就从最初的位置开始重写。将该值设置为100, 滚动窗口为10秒，若在10秒内一个“桶”中发生了500次执行，那么该“桶”中只保留最后的100次执行的统计
    // 此外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间
    @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
    // 设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间
    @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
    // 开启请求缓存
    @HystrixProperty(name = "requestCache.enabled", value = "true"),
    // HystrixCommand 的执行和事件是否打印日志到HystrixRequestLog
    @HystrixProperty(name = "requestLog.enabled", value = "true"),
},
threadPoolProperties = {
    // 设置执行命令线程池的核心线程数，也就是命令执行的最大并发量
    @HystrixProperty(name = "coreSize", value = "10"),
    // 设置线程池的最大队列大小，-1时，线程池将使用SynchronousQueue 实现的队列否则使用LinkedBlockingQueue 实现的队列
    @HystrixProperty(name = "maxQueueSize", value = "-1"),
    // 为队列设置拒绝阈值，通过该参数，即使队列没有达到最大值也能拒绝请求。该参数主要是对 LinkedBlockingQueue 队列的补充，因为LinkedBlockingQueue 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小
    @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
}
```

#### Hystrix 执行步骤

1. 创建HystrixCommand（用在依赖的服务返回单个操作结果的时候）或HystrixObserableCommand（用于依赖的服务返回多个操作结果时）对象

2. 命令执行，其中HystrixComand 实现下面前两种执行方式，而HystrixObservableCommand 实现后两种执行方式

   execute()：同步执行，从依赖的服务返回一个单一的结果对象， 或是在发生错误时抛出异常

   queue()：异步执行，直接返回一个Future 对象，其中包含服务执行结束时要返回的单一结果对象

   observe()：返回Observable 对象，它代表操作的多个结果，是一个Hot Obserable（不论"事件源"是否有"订阅者"，都会在创建后对事件进行发布，所以Hot Observable 的每一个"订阅者" 都有可能是从"事件源" 的中途开始的，并可能只看到整个操作的局部）

   toObservable()：同样返回Observable 对象，也代表操作的多个结果，但它返回的是一个Cold Observable（没有 "订阅者" 时不会发布事件，而是等待，直到有 "订阅者" 后再发布事件，对于Cold Observable 的订阅者，可以保证看到操作的全部过程）

3. 若当前命令的请求缓存功能是被启用的， 并且该命令缓存命中， 那么缓存的结果会立即以Observable 对象的形式返回

4. 检查断路器是否为打开状态，如果断路器打开，Hystrix 不会执行命令，而是转接到fallback 处理逻辑（第 8 步），如果断路器关闭，检查是否有可用资源来执行命令（第 5 步）

5. 线程池/请求队列/信号量是否占满，如果命令依赖服务的专有线程池和请求队列，或者信号量（不使用线程池的时候）已被占满， 那么Hystrix 不会执行命令， 而是转接到fallback 处理逻辑（第8步）

6. Hystrix 根据编写的方法决定采取什么方式去请求依赖服务

   HystrixCommand.run()：返回一个单一的结果，或者抛出异常

   HystrixObservableCommand.construct()： 返回一个Observable 对象发射多个结果，或通过onError 发送错误通知

7. Hystrix 将"成功"、"失败"、"拒绝"、"超时"等信息报告给断路器，断路器维护着一组计数器统计这些数据，断路器根据统计数据决定是否打开断路器，对某个依赖服务的请求进行"熔断/短路"

8. 命令执行失败，Hystrix 进入fallback 尝试回退处理，通常称该操作为"服务降级"，能够引起服务降级处理的情况有以下几种：

   第4步：当前命令处于"熔断/短路"状态，且断路器打开

   第5步：当前命令的线程池、请求队列或者信号量被占满

   第6步：HystrixObservableCommand.construct() 或HystrixCommand.run() 抛出异常

9. Hystrix 命令执行成功，将处理结果直接返回或以Observable 形式返回

注意：如果没有为命令实现降级逻辑或在降级处理逻辑中抛出异常，Hystrix 依然会返回一个Observable 对象， 但它不会发射任何结果数据，而是通过onError 方法通知命令立即中断请求，并通过onError 方法将引起命令失败的异常发送给调用者

#### 服务监控

除隔离依赖服务的调用外，Hystrix 还提供准实时的调用监控（Hystrix Dashboard），Hystrix 持续地记录所有通过Hystrix 发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。Netflix 通过hystrix-metrics-event-stream 项目实现了对以上指标的监控，SpringCloud 也提供Hystrix Dashboard 的整合，将监控内容转化成可视化界面

1. 创建模块，cloud_consumer_hystrix_dashboard9001

2. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
   </dependency>
   ```

3. application 配置

   ```yaml
   server:
     port: 9001
   ```

4. 启动类标注@EnableHystrixDashboard

   ```java
   @SpringBootApplication
   @EnableHystrixDashboard
   public class CloudConsumerHystrixDashboard9001 {
   ```

5. 监控的provider 需要依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

6. 启动服务，本地dashboard 页面路径http://localhost:9001/hystrix

7. 监控provider 在主启动类添加@bean 组件

   ```java
   // 为服务监控而配置，与服务容错无关，SpringCloud 升级后，ServletRegistrationBean 因为springBoot 的默认路径不是"/hystrix.stream"，只要在自己的项目里配置上下面的servlet 即可
   @Bean
   public ServletRegistrationBean getServlet() {
       HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
       ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
       registrationBean.setLoadOnStartup(1);
       registrationBean.addUrlMappings("/hystrix.stream");
       registrationBean.setName("HystrixMetricsStreamServlet");
       return registrationBean;
   }
   ```

8. dashboard 页面填写监控地址：http://localhost:8001/hystrix.stream

9. 发送正常和异常请求于provider，观察dashboard 图像显示

   实心圆：两种含义。颜色代表实例的健康程度，绿色<黄色<橙色<红色递减，大小也会根据实例的请求流量发生变化，流量越大该实心圆就越大

   曲线：记录2分钟内流量的相对变化，通过它可观察到流量的上升下降趋势

## 6. 服务网关

服务网关 = 路由转发 + 过滤器

​	路由转发：接收外界请求，转发到匹配微服务

​	过滤器：在请求的微服务执行前后增加操作，如权限校验、限流、监控等

作用：反向代理、鉴权、流量控制、熔断、日志监控等

### 6.1 Geteway

Gateway 基于Spring 5、SpringBoot 2、Project Reactor 等技术，旨在提供一种简单有效的方式对API 进行路由，目的是替代Zuul

SpringCloud 2.0 以后没有对Zuul 2.0 等新版本进行集成，而为提升网关性能，SpringCloud Gateway 基于WebFlux 框架实现，WebFlux 框架底层使用高性能的Reactor 模式和Netty 通信框架

Getway 于Zuul 1.x 区别：Zuul 1.x 基于Servlet 2. 5使用阻塞架构，不支持长连接，Zuul 的设计模式和Nginx 相似，每次I/ O 操作都从工作线程中选择一个执行，请求线程被阻塞直到工作线程完成，差别是Nginx 使用C++ 实现，Zuul 使用 Java 实现，JVM 本身有第一次加载较慢的情况，使得Zuul 的性能相对较差

* Getway 特性
  1. 动态路由：能匹配任何请求属性
  2. 对路由指定Predicate（断言）和Filter（过滤器）
  3. 集成Hystrix 断路器功能
  4. 集成SpringCloud 服务发现功能
  5. 易于编写的Predicate（断言）和 Filter（过滤器）
  6. 请求限流功能
  7. 支持路径重写

#### 路由（Route）

##### 基本配置

1. 创建模块cloud_gateway_gateway9527

2. 引入依赖

   ```xml
   <dependencies>
       <!-- gateway -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-gateway</artifactId>
       </dependency>
       <!-- eureka-client -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
       <!-- commons_api -->
       <dependency>
           <groupId>org.pochaiyi</groupId>
           <artifactId>cloud_api_commons</artifactId>
           <version>${project.version}</version>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

3. application 配置

   ```yaml
   server:
     port: 8001
   
   spring:
     application:
       name: cloud-gateway-gateway9527
   
   eureka:
     instance:
       #hostname: gateway
       instance-id: gateway9527
       prefer-ip-address: true
       lease-renewal-interval-in-seconds: 1
       lease-expiration-duration-in-seconds: 2
     client:
       register-with-eureka: true
       service-url:
         defaultZone: http://localhost:7001/eureka
         #defaultZone: http://eureka7001:7001/eureka,http://eureka7002:7002/eureka
   ```

4. 网关设置（方式1：配置文件）

   ```yaml
   spring:
     cloud:
       gateway:
         routes:
           # 路由规则ID
           - id: provider_payment_getPaymentById
             # 服务路由地址
             uri: http://localhost:8001
             # 断言条件，请求URL 必须符合条件才可通行
             predicates:
               # 请求路径断言，同时也是请求转发RequestMapping 路径
               - Path=/payment/get/**
   ```

5. 网关设置（方式2：RouteLocator Bean）

   向spring 注入路由组件

   ```java
   @Configuration
   public class RouteLocatorConfig {
       @Bean
       public RouteLocator paymentRouteLocator(RouteLocatorBuilder builder)
       {
           RouteLocatorBuilder.Builder routes = builder.routes();
   
           routes.route("provider_payment_getPaymentById", //路由规则ID
                   r -> r.path("/payment/get/**") //请求路径断言
                           .uri("http://localhost:8001")).build(); //服务路由地址
           return routes.build();
       }
   }
   ```

6. 此时基本配置完成

   可通过地址http://localhost:9527/payment/get/1 访问http://localhost:8001/payment/get/1

##### 动态路由

转发请求至服务集群，不能将服务地址写死，Gateway 默认根据注册中心服务列表，以微服务名称为路径创建动态路由进行转发，从而实现动态路由的功能

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 修改application 配置

   ```yaml
   spring:
     cloud:
       gateway:
         discovery:
           locator:
             # 开启从注册中心动态创建路由，默认false
             enabled: true
         routes:
           - id: provider_payment_getPaymentById
             # 服务协议lb 表示启用Gateway 负载均衡功能
             uri: lb://CLOUD-PROVIDER-PAYMENT
             predicates:
               - Path=/payment/get/**
   ```

3. 再次访问http://localhost:9527/payment/get/1，观察提供服务的微服务的ip:port 动态变化

#### 断言（Predicate）

即请求必须满足的条件，否则无法正常转发至微服务，有多种断言规则，请求URL 断言最常见

时间表示为ZoneDateTime 实例：`ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));`

| 断言                          | application 配置示例（predicates 子级）                      |
| ----------------------------- | ------------------------------------------------------------ |
| After（请求必须在某时间之后） | - After=2020-02-02T17:45:06.206+08:00[Asia/Shanghai]         |
| Before                        | - Before=2020-02-02T17:45:06.206+08:00[Asia/Shanghai]        |
| Between                       | - Between=2020-02-02T17:45:06.206+08:00[Asia/Shanghai],2020-03-25T18:59:06.206+08:00[Asia/Shanghai] |
| Cookie                        | - Cookie=key, value                                          |
| Header（请求头）              | - Header=key, value                                          |
| Host                          | - Host=**.com（可在请求头中设置request host）                |
| Method                        | - Method=GET                                                 |
| Path                          | - Path=/payment/get/**                                       |
| Query（请求参数）             | - Query=name[, \w+]（参数值）                                |

#### 过滤器（Filter）

路由过滤器可用于修改进入的HTTP 请求和返回的HTTP 响应，路由过滤器只能指定路由使用，Gateway 内置多种路由过滤器，它们都由GatewayFilter 的工厂类创建实例

过滤器分类：服务前执行（pre），服务后执行（post），单个过滤、全局过滤

##### 单个过滤

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: provider_payment_getPaymentById
          filters:
            # 过滤器工厂在匹配请求的请求头中添加信息 filterHead=855
            - AddRequestParameter=filterHead,855
          uri: lb://CLOUD-PROVIDER-PAYMENT
          predicates:
            - Path=/payment/get/**
```

##### 全局过滤

作用：全局日志记录、统一网关鉴权等

注入组件，实现接口GlobalFilter,Ordered 的实例

```java
@Component
@Slf4j
public class GlobalFilter_01 implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String name = exchange.getRequest().getQueryParams().getFirst("name");
        log.info(name);
        if(name == null){
            log.info("name = null");
            // 设置返回响应码
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            // 关闭请求响应，请求结束
            return exchange.getResponse().setComplete();
        }
        // 执行过滤器链，即放行执行后续过滤器或服务方法
        return chain.filter(exchange);
    }

    // 过滤器优先级，{-2147483648，2147483647}，值越小优先级越高
    @Override
    public int getOrder() {
        return 0;
    }
}
```

## 7. 配置中心

微服务架构将单体应用拆分为粒度十分小的大量微服务，每个微服务都有配置文件，其中包含大量一致的、可能需要修改的信息，当微服务数量众多时，对这些信息的修改将十分不方便，配置中心便是一套集中式的、动态的配置管理工具，它具有以下优点

1. 集中管理配置信息
2. 不同环境不同配置，动态化的配置更新，分环境部署
3. 运行期间调整配置，不需在每台部署机器修改配置，服务会向配置中心统一拉取配置信息
4. 配置发生变动，服务不需要重启即可感知到配置的变化并应用新的配置
5. 将配置信息以REST 接口的形式暴露

### 7.1 Config

SpringCloud Config 为微服务提供集中化的外部配置支持，配置服务器为各个不同的微服务应用的所有环境提供中心化的外部配置

SpringCloud Config 分为服务端和客户端两部分

1. 服务端也称分布式配置中心，是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口

2. 客户端则是通过指定的配置中心来管理应用资源和与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息，配置服务器默认采用Git 存储配置信息，这有助于对环境配置进行版本管理，且可以通过Git 客户端管理和访问配置内容

#### 服务端

1. 创建Gitee 仓库springcloud_config，并在仓库中创建不同环境的application 配置文件

2. 创建模块cloud_config_center3344

3. 引入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-config-server</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

4. application 配置

   ```yaml
   server:
     port: 3344
   spring:
     application:
       name:  cloud-config-center
     cloud:
       config:
         server:
           git:
             uri: https://gitee.com/pochaiyi/spring-cloud-config
             # 搜索目录
             search-paths:
               - spring-cloud-config
         # 读取分支
         label: master
   
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

5. 主启动类标注注解@EnableConfigServer

   ```java
   @SpringBootApplication
   @EnableConfigServer
   public class CloudConfigCenter3344 {
       public static void main(String[] args) {
           SpringApplication.run(CloudConfigCenter3344.class,args);
       }
   }
   ```

6. http://localhost:3344/master/{文件名} 通过配置中心访问远程配置信息

##### 配置访问规则

文件命名规则：{application}-{profile}.yaml

1. /{label}/{application}-{profile}.yaml
2. /{application}-{profile}.yaml（默认为application 配置分支或master）
3. /{application}/{profile}[/{label}]

* label：分支(branch)
* name ：服务名
* profiles：环境(dev/test/prod)

#### 客户端

1. 创建模块

2. 引入依赖

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-config</artifactId>
       </dependency>
   	...
   </dependencies>
   ```

3. bootstrap 配置

   ```yaml
   server:
     port: 3355
   
   spring:
     application:
       name: cloud-config-client
     cloud:
       # Config 客户端配置
       config:
         # 分支
         label: master
         # 配置文件application
         name: config
         # 配置文件profile
         profile: prod
         # 配置中心地址k
         uri: http://localhost:3344
         # 综上，将读取配置信息 http://config:3344/master/config-prod.yaml
   
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   ```

   > applicaiton.yaml 是用户级资源配置项，bootstrap.yaml 是系统级，优先级更高
   >
   > SpringBoot 启动时会创建`Bootstrap Context` 和`Application Context` ，前者是后者的父上下文，初始化时，`Bootstrap Context` 从外部源加载配置属性并解析，这两个上下文共享一个从外部获取的`Environment`
   >
   > `bootstrap.yml` 对应`Bootstrap Context` 配置，与`Application Context` 配置分离，默认情况，它不会被本地配置覆盖

4. 主启动类标注@EnableEurekaClient

5. 业务类

   ```java
   @RestController
   @Slf4j
   public class ConfigClientController {
       @Value("${config.info}")
       private String configInfo;
   
       @GetMapping("/configInfo")
       public String getConfigInfo()
       {
           log.info("getConfigInfo");
           return configInfo;
       }
   }
   ```

6. 启动客户端微服务

   访问http://localhost:3355/configInfo 返回http://config:3344/master/config-prod.yaml config: info: 的值

##### 动态刷新

修改Gitee 远程仓库中config-prod.yaml 文件config: info 属性的值，不重启服务，服务端配置信息更新，客户端不变，重启客户端服务，配置信息变化

修改客户端程序，设置不重启更新配置信息

**此方法需要手动刷新配置信息，且一条命令只能刷新一个客户端的配置信息**

1. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 修改application 配置

   ```yaml
   # 暴露监控端口
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

3. 业务类标注注解@RefreshScope

4. 测试

   修改Gitee 远程仓库中config-prod.yaml 文件config: info 属性的值

   查看客户端配置信息，未更新

   发送Post 请求刷新配置信息：`curl -X POST "http://localhost:3355/actuator/refresh"`

   查看客户端配置信息，已经更新

## 8. 消息总线

在微服务架构的系统中，通常使用轻量级的消息代理构建共用的消息主题，并让系统中所有微服务实例都连接上，由于该主题产生的消息会被所有实例监听和消费，所以称它为消息总线，总线上的各实例，都可以方便地广播一些需要让其他连接在该主题上的实例都需要获取的消息

ConfigClient 所有实例都监听MQ 中同一个Topic （默认springCloudBus），当一个服务刷新数据时，它会把这个信息放入Topic，其它监听同个Topic 的服务就能得到通知，然后更新自身的配置

### 8.1 Bus

SpringCloud Bus 是能将分布式系统的节点与轻量级消息系统链接起来的框架，它整合Java 事件处理机制和消息中间件的功能，SpringClud Bus 目前支持RabbitMQ 和Kafka

SpringCloud Bus 能管理和传播分布式系统间的消息，就像一个分布式执行器，可用于广播状态更改、事件推送等，也可以当作微服务间的通信通道，SpringCloud Bus 配合 SpringCloud Config 可以实现配置的动态刷新

#### RabbitMQ 安装

##### erlang 环境配置

1. 安装依赖

   ```shell
   yum -y install gcc glibc-devel make ncurses-devel openssl-devel xmlto perl wget gtk2-devel binutils-devel
   ```

2. 下载

   ```shell
   wget http://erlang.org/download/otp_src_22.0.tar.gz
   ```

3. 解压

   ```shell
   tar -zxvf otp_src_22.0.tar.gz
   ```

4. 配置安装路径

   ```shell
   /opt/otp_src_22.0/configure --prefix=/opt/erlang
   ```

5. 安装

   ```shell
   cd /opt/otp_src_22.0/configure
   
   make install
   ```

6. 检查安装成功

   ```shell
   ll /opt/erlang/bin/
   ```

7. 配置环境变量

   ```shell
   echo 'export PATH=$PATH:/opt/erlang/bin/' >> /etc/profile
   
   source /etc/profile
   ```

8. erlang 使用

   ```shell
   # 进入
   erl
   # 退出
   halt().
   ```

##### RabbitMQ 安装

1. 下载

   ```shell
   wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.15/rabbitmq-server-generic-unix-3.7.15.tar.xz
   ```

2. 安装xz 工具解压tar.xz 文件

   ```shell
   yum install -y xz
   ```

3. 第一次解压

   ```shell
   /bin/xz -d rabbitmq-server-generic-unix-3.7.15.tar.xz

4. 第二次解压

   ```shell
   tar -xvf rabbitmq-server-generic-unix-3.7.15.tar -C /opt/

5. 配置环境变量

   ```shell
   echo 'export PATH=$PATH:/opt/rabbitmq_server-3.7.15/sbin' >> /etc/profile
   
   source /etc/profile

##### RabbitMQ 开始

1. 启动

   ```shell
   rabbitmq-server -detached

2. 停止

   ```shell
   rabbitmqctl stop

3. 查看状态

   ```shell
   rabbitmqctl status

4. 开启web 插件

   ```shell
   rabbitmq-plugins enable rabbitmq_management
   ```

   访问地址：http://192.168.178.100:15672/

   账号密码默认：guest guest（默认账号只能本地登录）

##### RabbitMQ 用户管理

1. 查看所有用户

   ```shell
   rabbitmqctl list_users

2. 添加用户

   ```shell
   rabbitmqctl add_user username password

3. 配置权限

   ```shell
   rabbitmqctl set_permissions -p "/" username ".*" ".*" ".*"

4. 查看用户权限

   ```shell
   rabbitmqctl list_user_permissions username

5. 设置tag

   ```shell
   rabbitmqctl set_user_tags username administrator

6. 删除用户

   ```shell
   rabbitmqctl delete_user username

#### 全局动态刷新

**此方法仍需要手动刷新配置信息，但一条命名可刷新所有client 端配置信息**

原理：server 和client 都订阅Rabbit 相同的Topic，使用Post 请求发送刷新配置信息的指令，已经更新配置信息的server 端将新的配置发送至MQ，监视到消息更新的所有client 从MQ 获取更新配置信息，从来达到一条命令刷新全局client 配置信息的目的

1. 根据cloud_config_client3355 创建新的模板cloud_config_client3366

2. ConfigServer 和client 添加消息支持

   引入依赖

   ```xml
   <!-- 添加消息总线RabbitMQ 支持 -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bus-amqp</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

   application 配置

   management.endpoints.web.exposure.include 是Actuator 暴露端点的配置

   ```yaml
   spring:
     ...
     rabbitmq:
       host: 192.168.178.100
       port: 5672
       username: rabbit
       password: rabbit
   
   # rabbitmq 相关配置，暴露bus刷新配置的端点
   management:
     # 暴露bus 刷新配置的端点
     endpoints:
       web:
         exposure:
           include: 'bus-refresh'
   ```

3. 测试

   启动config  Server 和两个ConfigClient

   分别访问配置信息：

   ​		http://config:3344/master/config-prod.yaml

   ​		http://localhost:3355/configInfo

   ​		http://localhost:3366/configInfo

   修改Gitee 远程仓库中config-prod.yaml 文件config: info 属性的值

   再次查看配置信息，服务端已更新，客户端未更新

   发送Post 请求刷新配置信息：`curl -X POST "http://config:3344/actuator/bus-refresh"`

   查看两个客户端配置信息，都已经更新

#### 定点刷新通知

刷新指定的ConfigClient 节点，刷新请求发送给ConfigServer 并通过destination 参数类指定需要更新配置的服务或实例

Post 请求

```shell
curl -X POST http://localhost:配置中心端口号/actuator/bus-refresh/{微服务名称:port}

# eg
curl -X POST http://localhost:3344/actuator/bus-refresh/cloud-config-client:3366
```

## 9. 消息驱动

思想参考JDBC，将对MQ 的操作封装为抽象的API，对不同的MQ 产品使用特定的驱动程序，使得对所有的MQ，操作的代码都几乎相同，即使更换底层MQ 也不需要大量修改代码

### 9.1 Stream

SpringCloud Stream 是一个构建消息驱动微服务的框架。微服务通过binding（input/output）与SpringCloud Stream 的binder 对象交互，binder 对象再与消息中间件交互

SpringCloud Stream 构建的应用程序与MQ 通过binder 联系，**绑定器对应用程序起到隔离作用**， 使得MQ 的实现细节对应用程序是透明的。binder 可以生成binding，binding 即消息通道，对应消息的消费者和生产者有input 和output 两种类型，消息通道具体可对应到MQ 的exchange 或topic

1. output：发送消息Channel，内置Source 接口
2. input：接收消息Channel，内置Sink 接口
3. binding：通过配置将微服务和binder 绑定

微服务通过Spring Integration 连接消息代理中间件以实现消息事件驱动，SpringCloud Stream 为供应商的消息中间件产品提供个性化的自动化配置实现，引用发布-订阅、消费组、分区的三个核心概念，目前仅支持RabbitMQ、Kafka

#### 消息生产者

1. 创建模块 cloud_stream_rabbitmq_provider8801

2. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
   </dependency>
   ```

3. application 配置

   ```yaml
   server:
     port: 8801
   
   spring:
     application:
       name: cloud-stream-provider
     cloud:
       stream:
         # 配置绑定的rabbitmq 的服务信息
         binders:
           # 定义的名称，用于binding 整合
           defaultRabbit:
             # MQ 类型
             type: rabbit
             # rabbitmq 相关的环境配置
             environment:
               spring:
                 rabbitmq:
                   host: 192.168.178.100
                   port: 5672
                   username: rabbit
                   password: rabbit
         # 服务的整合处理
         bindings:
           # 通道名称
           output:
             # 订阅或发布的exchange 名称
             destination: cloudStreamMessage
             # 消息类型，文本设置为 text/plain
             content-type: application/json
             # 设置要绑定的消息服务的具体设置，即对应的binder
             binder: defaultRabbit
   
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
     instance:
       lease-renewal-interval-in-seconds: 2
       lease-expiration-duration-in-seconds: 5
       instance-id: messageProducer8801
       prefer-ip-address: true
   ```

4. 发布类

   ```java
   // 为当前类注入消息通道的实例
   @EnableBinding(Source.class)
   public class MessageProviderService {
       // 定义消息通道
       @Resource
       private MessageChannel output;
   
       public String send(){
           String serial = UUID.randomUUID().toString().substring(0,8);
           // 根据随机值创建一个message 并将其发布
           this.output.send(MessageBuilder.withPayload(serial).build());
           return serial;
       }
   }
   ```

5. web 交互

   ```java
   @RestController
   public class SendMessageController {
       @Resource
       private MessageProviderService messageProviderService;
   
       @GetMapping(value = "/sendMessage")
       public String sendMessage()
       {
           return messageProviderService.send();
       }
   }

运行会有MQ 连接报错，暂时无视

#### 消息消费者

1. 创建模块 cloud_stream_rabbitmq_consumer8802

2. 引入依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
   </dependency>
   ```

3. application 配置

   ```yaml
   server:
     port: 8802
   
   spring:
     application:
       name: cloud-stream-consumer
     cloud:
       stream:
         binders:
           defaultRabbit:
             type: rabbit
             environment:
               spring:
                 rabbitmq:
                   host: 192.168.178.100
                   port: 5672
                   username: rabbit
                   password: rabbit
         bindings:
           input:
             # 订阅或发布的exchange 名称
             destination: cloudStreamMessage
             content-type: application/json
             binder: defaultRabbit
   
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
     instance:
       lease-renewal-interval-in-seconds: 2
       lease-expiration-duration-in-seconds: 5
       instance-id: messageConsumer8802
       prefer-ip-address: true

4. 订阅类，监听消息通道

   ```java
   @Service
   // 为当前类注入消息通道的实例，binding.Sink 接口用于接收消息
   @EnableBinding(Sink.class)
   @Slf4j
   public class ReceiveMessageListener {
   
       @Value("${server.port}")
       private String serverPort;
   
       // 监听消息通道，当接收到消息便执行此方法
       @StreamListener(Sink.INPUT)
       public void input(Message<String> message)
       {
           log.info("message: " + message.getPayload()+"; port: "+serverPort);
       }
   }
   ```

5. 启动rabbit、生产者、消费者，访问消费者发布消息URL，可观察到消费者执行监听通道方法执行

##### 消费者分组

Rabbit 中被订阅的Exchange，一条消息可被多个消费者组消费，一组消费者中只能由一个消费者消费此消息，即队外全量消费，队内竞争消费（与RocketMQ Queue 消息消费类似）

* application 配置分组

  ```yaml
  spring:
    cloud:
      stream:
        bindings:
          input:
            group: 分组名称
  ```

## 10. 链路监控

微服务框架中，客户端发起的一个请求在后端系统中可能会经过多个不同的的服务节点调用协同产生最后的响应结果，每一个前段请求都会形成一条复杂的分布式服务调用链，链路中的任何一环出现高延时或错误都会引起整个请求最后的失败。

### 10.1 Sleuth

为能即时发现出现问题的节点，需要监控整个调用链，SpringCloud Sleuth 在分布式系统中提供追踪请求调用的解决方案并且兼容支持zipkin（dashboard）

SpringCloud F 版后不再需要专门构建Zipkin Server，只需调用JAR 包即可

#### Zipkin 安装

1. 下载

   https://repo1.maven.org/maven2/io/zipkin/zipkin-server/

   zipkin-server-2.20.0-exec.jar

2. 运行

   ```shell
   java -jar zipkin-server-2.20.0-exec.jar
   ```

3. 访问dashboard 页面，http://localhost:9411/zipkin/

#### 微服务整合

cloud_provider_payment8001 服务提供者

1. 引入依赖

   ```xml
   <!-- sleuth+zipkin -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zipkin</artifactId>
   </dependency>
   ```

2. application 配置

   ```yaml
   spring:
     zipkin:
     	# zipkin 服务地址
       base-url: http://localhost:9411
     sleuth:
       sampler:
         # 采样率值（0，1），1 表示全部采集，但性能会下降
        probability: 1
   ```

cloud-consumer-order80 服务消费者

与提供者做相同修改

# SpringCloud Alibaba

**SpringCloud Netflix 维护**

SpringCloud Netflix 项目进入维护模式，以下模块和相应的启动器（starter）将被置于维护模式：

1. spring-cloud-netflix-archaius
2. spring-cloud-netflix-hystrix-contract
3. spring-cloud-netflix-hystrix-dashboard
4. spring-cloud-netflix-hystrix-stream
5. spring-cloud-netflix-hystrix
6. spring-cloud-netflix-ribbon
7. spring-cloud-netflix-turbine-stream
8. spring-cloud-netflix-turbine
9. spring-cloud-netflix-zuul

不包括Eureka 和concurrency-limits 模块

> 将模块置于维护模式意味着SpringCloud 团队将不再向模块添加新功能，将修复拦截器错误和安全问题，将考虑和审查来自社区的小型拉取请求

SpringCloud 建议替换的新组件（当然，目前应该更多使用Alibaba 的新组件）

| Current                     | Replacement                                     |
| --------------------------- | ----------------------------------------------- |
| Hystrix                     | Resilience4j                                    |
| Hystrix Dashboard / Turbine | Micrometer + Monitoring System                  |
| Ribbon                      | SpringCloud Loadbalancer                        |
| Zuul 1                      | SpringCloud Gateway                             |
| Archaius 1                  | SpringBoot external config + SpringCloud Config |

**SpringCloud Alibaba 诞生**

所以，Alibaba 开发了属于自己的一站式分布式系统解决方案（正如曾经Dubbo 被SpringCloud 替代一般），2018.10.31，Spring Cloud Alibaba 入驻Spring Cloud 官方孵化器，并在Maven 中央库发布了第一个版本

Github：https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md

**主要功能**

1. 服务限流降级：默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控
2. 服务注册与发现：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持
   分布式配置管理：支持分布式系统中的外部化配置，配置更改时自动刷新
3. 消息驱动能力：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力
4. 阿里云对象存储：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据
5. 分布式任务调度：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行

## 1. Nacos（Alibaba）

Nacos = 服务注册 + 配置管理 = Eureka + Config + Bus，Nacos 可以设置AP 或CP 模式，拥有内置控制台管理

官网：https://nacos.io/zh-cn/index.html

下载：https://github.com/alibaba/nacos/releases

运行环境：Java8 + Maven

### 1.1 服务注册

#### 启动Nacos

1. 解压压缩包

2. 单机启动

   ```shell
   sh /opt/nacos/bin/startup.sh -m standalone

3. 查看dashboard 页面，默认账号密码都为nacos

#### 服务提供者模块

1. 创建模块

2. 引入依赖

   ```xml
   <!-- SpringCloud Alibaba nacos -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. application 配置

   ```yaml
   server:
     port: 9001
   
   spring:
     application:
       name: nacos-payment-provider
     cloud:
       nacos:
         discovery:
           server-addr: 192.168.178.100:8848
   
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   ```

3. 主启动类标注@EnableDiscoveryClient

   ```java
   @SpringBootApplication
   @EnableDiscoveryClient
   public class CloudAlibabaProviderPayment9001 {
       public static void main(String[] args) {
           SpringApplication.run(CloudAlibabaProviderPayment9001.class,args);
       }
   }
   ```

#### 服务消费者模块

1. 创建模块

2. 引入依赖

   ```xml
   <!-- SpringCloud ailibaba nacos -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

3. application 配置

   ```yaml
   server:
     port: 83
   
   spring:
     application:
       name: nacos-order-consumer
     cloud:
       nacos:
         discovery:
           server-addr: 192.168.178.100:8848
   
   # 消费者将要访问的微服务名称（成功注册nacos 的微服务提供者）
   service-url:
     nacos-user-service: http://nacos-payment-provider
   ```

4. 主启动类标注@EnableDiscoveryClient

5. 注入RestTemplate 组件

   ```java
   @Configuration
   public class ApplicationContextBean {
       @Bean
       // Nacos 含有并默认开启负载均衡（轮询），必须标注@LoadBalanced 开启负载均衡功能
       @LoadBalanced
       public RestTemplate getRestTemplate() {
           return new RestTemplate();
       }
   }
   ```

6. 业务类，Nacos 服务消费者也使用RestTemplate 远程调用服务

   ```java
   @RestController
   public class OrderNacosController {
       @Resource
       private RestTemplate restTemplate;
   
       @Value("${service-url.nacos-user-service}")
       private String serverURL;
   
       @GetMapping("/consumer/payment/nacos/{id}")
       public String paymentInfo(@PathVariable("id") Long id) {
           return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
       }
   }
   ```

### 1.2 配置中心

Nacos 会记录配置文件历史版本，默认保留30天

控制台还回滚功能，回滚操作将触发配置更新

Nacos 自动动态刷新，不需像Config+Bus 那样手动发送Post 请求更新配置信息

#### 基本使用

1. 创建模块 cloudalibaba_nacos_config_client3377

2. 引入依赖

   ```xml
   <!-- nacos-config -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
   </dependency>
   <!-- nacos-discovery -->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

3. bootstrap、application 配置

   bootstrap 是系统级配置，优先级高于application（应用级），微服务根据dataId 从Nacos 获取配置信息

   ```yaml
   # 优先级高于application，用于从配置中心获取配置信息
   # nacos配置
   server:
     port: 3377
   
   spring:
     application:
       name: nacos-config-client
     cloud:
       nacos:
         discovery:
           # Nacos 注册中心地址
           server-addr: 192.168.178.100:8848
         config:
           # Nacos 配置中心地址
           server-addr: 192.168.178.100:8848
           # 指定yaml 格式的配置
           file-extension: yaml
   
   # 此时配置对应的远程配置信息为：nacos-config-client-prod.yaml
   ```

   ```yaml
   spring:
     profiles:
       active: prod
   ```

4. 主启动类标注@EnableDiscoveryClient

5. 业务类，标注@RefreshScope，实时更新配置信息

   ```java
   @RestController
   // @RefreshScope 使当前控制器类的配置支持Nacos 动态刷新功能
   @RefreshScope
   public class ConfigClientController {
       @Value("${config.info}")
       private String configInfo;
   
       @GetMapping("/config/info")
       public String getConfigInfo() {
           return configInfo;
       }
   }
   ```

#### DataId

`dataId` 完整格式：`${prefix}-${spring.profiles.active}.${file-extension}`

* `prefix`默认为`spring.application.name`，也可以通过`spring.cloud.nacos.config.prefix`配置

- `spring.profiles.active`即当前环境对应的profile

  若`spring.profiles.active`为空，对应的连接符`-`也将不存在，dataId 的拼接格式变为 `${prefix}.${file-extension}`

- `file-exetension` 为配置信息的数据格式，可以通过`spring.cloud.nacos.config.file-extension` 配置，目前只支持`properties` 和 `yaml` 类型，`yaml` 与`yml` 不匹配

#### 分类配置

分布式系统常有多种环境（生产、开发、测试等），且大型项目又有许多子项目，为合理的管理并区分不同环境的配置信息，Nacos 引入分类配置的概念

DataId + 名称空间（namespace）、分组（group）= {namespace{group{DataId}}} 便是一个配置文件在Nacos 中的的精确定位，前面没有配置namespace 和group 的原因是使用默认的public 名称空间和DEFAULT_GROUP 分组

application 中配置namespace 和group

```yaml
spring:
    nacos:
      config:
        group: 分组
        namespace: 名称空间（Id序号）
```

### 1.3 集群搭建

1个Ningx + 2个Nacos + 1个MySql

Nacos 默认使用内置的数据库derby，集群使用默认存储方式存在数据一致性问题，需要设置为集中式存储集群化部署（目前只支持MySql）

集群中每个Nacos 服务节点的IP:Port 都不同，使用Ningx 统一接收请求，再根据负载均衡策略转发至特定的Nacos 节点

#### 持久化切换

1. 执行sql 脚本，创建Nacos 数据库、表，sql 脚本位置`解压目录/conf/nacos-mysql.sql`

2. 修改nacos 配置，设置存储方式为MySql，配置文件位置`解压目录/conf/application.properties`

   使用Linux 中的MySQL 5.7，8.0 版本出错

   ```shell
   # 文件末尾追加
   spring.datasource.platform=mysql
   
   db.num=1
   econnect=true
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=10000&socketTimeout=30000&autoReconnect=true
   #db.url.0=jdbc:mysql://192.168.3.4:3306/nacos_config?characterEncoding=utf8&connectTimeout=10000&socketTimeout=30000&autoReconnect=true&serverTimezone=UTC
   db.user=root
   db.password=root
   ```

#### 集群启动

1. centos01 安装第二台Nacos 服务

2. 修改所有Nacos 节点的集群配置

   ```shell
   # 创建集群配置文件，可以直接拷贝example 文件
   cp /opt/nacos/conf/cluster.conf.example conf/cluster.conf
   ```

   ```shell
   # 追加集群节点IP：Port
   192.168.178.100:8848
   192.168.178.101:8848
   ```

3. 启动集群服务

   ```shell
   # Nacos 默认集群启动
   /opt/nacos/bin/startup.sh
   
   # 响应信息（部分）
   2021-09-15 14:59:04,484 INFO The server IP list of Nacos is [192.168.178.100:8848, 192.168.178.101:8848]
   ```

4. 分别登录集群节点的控制台，在其中一台创建配置信息，可在另一控制台查看到

#### 反向代理

使用Nginx 作为Nacos 节点的反向代理（代理服务器），同时提供负载均衡功能

##### Nginx 安装

下载：http://nginx.org/en/download.html

1. 解压

   ```shell
   tar -zxvf nginx-1.20.1.tar.gz /opt
   ```

2. 进入解压目录 /opt/，源码安装 

   ```shell
   ./configure --prefix=/opt/nginx
   ```

   ```shell
   make
   ```

   ```shell
   make install
   ```

* nginx 默认配置文件在解压目录，路径`/opt/nginx-1.20.1/conf/nginx.conf`

* 启动nginx

  ```shell
  /opt/nginx/sbin/nginx [选项]
  
  -c ：指定配置文件路径
  ```

* 停止nginx

  ```shell
  /opt/nginx/sbin/nginx -s stop
  ```

* 安全停止nginx

  ```shell
  /opt/nginx/sbin/nginx -s quit
  ```

* 重新加载配置

  ```shell
  /opt/nginx/sbin/nginx -s reload
  ```

##### Nginx 配置

位置：/opt/nginx/conf/nginx.conf

1. 修改配置，监听8890 端口，转发至Nacos 服务集群

   ```shell
   33     #gzip  on;
   34 
   35     upstream cluster{
   36         server 192.168.178.100:8848;
   37         server 192.168.178.101:8848;
   38     }
   39 
   40     server {
   41         listen       8890;
   42         server_name  localhost;
   43 
   44         #charset koi8-r;
   45 
   46         #access_log  logs/host.access.log  main;
   47 
   48         location / {
   49             #root   html;
   50             #index  index.html index.htm;
   51             proxy_pass http://cluster;
   52         }
   ```

2. 开启Nginx 服务

   ```shell
   /opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf
   ```

3. 访问http://192.168.178.100:8890/nacos，便可进入进群某节点的控制台

4. 最终测试

   通过Nginx 访问控制台，添加配置消息，可分别在集群节点控制台查看到数据，在MySql 数据库中也可以查看到新插入数据

#### 微服务修改

1. 将Nacos 服务地址修改为Nginx 暴露的地址

   ```shell
   spring:
     application:
       name: nacos-config-client
     cloud:
       nacos:
         discovery:
           #server-addr: 192.168.178.100:8848
           server-addr: 192.168.178.100:8890
   ```

## 2. Sentinel（Alibaba）

Sentinel 是一个轻量级流量控制、熔断降级的Java 库，下载：https://github.com/alibaba/Sentinel/releases

Sentinel 分为两部分：核心库（Java 客户端）、控制台（Dashboard）

Dashboard 是可直接运行的JAR 包，占用与Tomcat 相同的8080 端口，链接地址：http://localhost:8080/，默认账户密码均为sentinel

### 2.1 演示模块

1. 创建模块 cloudalibaba_sentinel_service8401

2. 引入依赖

   ```xml
   <dependencies>
       <!-- SpringCloud ailibaba nacos -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
       <!-- SpringCloud ailibaba sentinel-datasource-nacos 用于持久化 -->
       <dependency>
           <groupId>com.alibaba.csp</groupId>
           <artifactId>sentinel-datasource-nacos</artifactId>
       </dependency>
       <!-- SpringCloud ailibaba sentinel -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
       </dependency>
       <!-- openfeign -->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-openfeign</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
       </dependency>
       <dependency>
           <groupId>cn.hutool</groupId>
           <artifactId>hutool-all</artifactId>
           <version>4.6.3</version>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <optional>true</optional>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   ```

3. application 配置

   ```yaml
   server:
     port: 8401
   
   spring:
     application:
       name: cloudalibaba-sentinel-service
     cloud:
       nacos:
         discovery:
           # Nacos 服务注册中心地址
           server-addr: 192.168.178.100:8848
       sentinel:
         transport:
           # Sentinel dashboard 地址
           dashboard: localhost:8080
           # 默认8719端口，如被占用自动从8719开始依次+1 扫描，直至找到未被占用的端口
           port: 8719
   
   management:
     endpoints:
       web:
         exposure:
           include: '*'
   ```

4. 主启动类标注@EnableDiscoveryClient

5. 业务类

   ```java
   @RestController
   public class FlowLimitController
   {
   
       @GetMapping("/testA")
       public String testA()
       {
           return "testA";
       }
   
       @GetMapping("/testB")
       public String testB()
       {
           return "testB";
       }
   }
   ```

6. 启动Sentinel Dashboard

   ```shell
   java -jar sentinel-dashboard-1.8.2.jar
   ```

   http://localhost:8080/

7. 启动Nacos

   ```shell
   sh /opt/nacos/bin/startup.sh -m standalone
   ```

   http://192.168.178.100:8890/nacos

8. 启动演示模块

   Dashboard 采用懒加载，首次访问请求http://localhost:8401/testA 后才在Dashboard 查看到服务信息

### 2.2 流量控制

服务 -> 流控规则 -> 新增流控规则

* 资源名：唯一名称，默认请求路径

* 针对来源：Sentinel 可以对调用者进行限流，填写微服务名称，默认default（不区分来源）

* 阈值类型/单机阈值

  * QPS（每秒请求数量）：当该资源的请求达到阈值，进行限流
  * 线程数：当处理该资源请求的线程达到阈值，进行限流

* 是否集群：不需要集群

* 流控模式

  * 直接：达到限流条件直接限流

  * 关联：关联资源达到限流条件，该资源限流（此时设置的阈值属于关联资源）

    eg：/testA 关联/testB，/testB 请求达到阈值，/testA 限流

* 流控效果

  * 快速失败：直接失败，抛出异常

  * Warm Up：codeFactor（冷加载因子，默认3）的值，从阈值/codeFactor 经过指定预热时间达到QPS 阈值，防止低流量的服务突然需要处理大量请求导致故障

    eg：QPS=10，当大量请求进入，阈值初始为10/3=3，并逐渐增加至10

  * 排队等待：匀速排队，严格控制请求间隔时间（漏桶算法）让请求匀速通过，只支持QPS 阈值类型，等待超时请求失败，这种方式用于处理间隔性突发的流量，避免突然大流量导致许多请求被限流 

### 2.3 服务熔断

Sentinel 熔断降级在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。资源被降级后，接下来的降级时间窗口内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

Sentinel 的断路器没有半开状态，半开状态时系统自动检测请求是否仍有异常，没有则关闭断路器恢复使用，有则继续打开断路器不可用

#### 熔断策略

* 慢调用比例 (`SLOW_REQUEST_RATIO`)：选择以慢调用比例作为阈值，需要设置允许的慢调用RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用RT 则结束熔断，若大于设置的慢调用RT 则会再次被熔断。
* 异常比例 (`ERROR_RATIO`)：当单位统计时长（`statIntervalMs`）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 `[0.0, 1.0]`，代表 0% - 100%。
* 异常数 (`ERROR_COUNT`)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。

### 2.4 热点参数限流

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。

Sentinel 利用LRU 策略统计最近最常访问的热点参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式

1. 配置业务方法并标注@SentinelResource

   ```java
   // 热点参数限流
   @GetMapping("/testC")
   // value = 唯一标识符 = dashboard 热点规则资源名，blockHandler = 处理配置错误的方法名，默认返回错误白页
   //@SentinelResource(value = "testC",blockHandler = "testC_handler")
   @SentinelResource(value = "testC")
   public String testC(@RequestParam(value = "param1",required = false)String param1,
                       @RequestParam(value = "param2",required = false)String param2){
       log.info(param1);
       log.info(param2);
       return "testC";
   }
   // 错误处理方法参数 = 源方法参数 + BlockException 参数，顺序不可变
   public String testC_handler(String param1, String param2, BlockException e){
       return "testC_handler";
   }
   ```

2. dashboard 配置热点规则
   * 资源名：@SentinelResource 注解value 值
   * 只支持QPS 限流模式
   * 参数索引，由0 开始

3. 参数例外项

   特定参数值另外设置限流规则，比如可设置param1 参数值为1时，它限流阈值为100

* 参数必须是基本类型或者String

### 2.5 系统自适应限流

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的Load、CPU 使用率、总体平均RT、入口QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

目的：保证系统不被拖垮；在系统稳定的前提下，保持系统的吞吐量

#### 系统规则配

系统保护规则是从应用级别的入口流量进行控制，从单台机器的load、CPU 使用率、平均RT、入口QPS 和并发线程数等几个维度监控应用指标，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且仅对入口流量生效。入口流量指的是进入应用的流量，比如Web 服务或Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

* Load 自适应（仅对Linux/Unix-like 机器生效）：系统的load 作为启发指标，进行自适应系统保护。当系统load 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的`maxQps * minRt` 估算得出。设定参考值一般是`CPU cores * 2.5`
* CPU usage（1.5.0+ 版本）：当系统CPU 使用率超过阈值即触发系统保护（取值范围0.0-1.0），比较灵敏
* 平均RT：当单台机器上所有入口流量的平均RT 达到阈值即触发系统保护，单位是毫秒
* 并发线程数：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护
* 入口QPS：当单台机器上所有入口流量的QPS 达到阈值即触发系统保护

### 2.6 @SentinelResource

#### BlockException

Sentinel 对微服务进行服务时（流量监控、热点参数限流等），当服务调用违背配置的规则，会抛出BlockException，默认返回白页，可在代码中配置专门处理BlockException 的方法

##### URL地址

前面Dashboard 配置规则，皆使用URL 作为资源名

##### 资源名称

使用@SentinelResource 注解标注业务方法

@SentinelResource 属性

| name              | description                       |
| ----------------- | --------------------------------- |
| value             | Id 值，Dashboard 配置规则的资源名 |
| blockHandler      | 处理方法名                        |
| blockHandlerClass | 处理方法类Class 对象，默认本地类  |

1. 业务类

   ```java
   @RestController
   public class BlockExceptionController {
       @GetMapping("/byResource")
       @SentinelResource(value = "byResource",
           blockHandlerClass = BlockExceptionHandler.class,blockHandler = "byResourceHandle")
       public CommonResult byResource() {
           return new CommonResult(200,"byResource");
       }
   }
   ```

2. 处理类，分离业务逻辑与异常处理逻辑

   ```java
   public class BlockExceptionHandler {
       public static CommonResult byResourceHandle(BlockException exception){
           return new CommonResult(402,"BlockExceptionHandler.byResourceHandle");
       }
   }
   ```

3. Dashboard 配置规则（熔断、限流、热点等）

#### OriginalException

BlockHandler 不能处理BlockException 以外的其他Java 原生异常或SpringCloud 定义的异常，这些代码逻辑的异常需要配置fallback 方法处理

**sentinel 整合ribbon + openFeign + fallback**

1. 创建服务提供模块cloudalibaba_provider_payment9003、cloudalibaba_provider_payment9003，过程参考演示模块

   1）业务类

   ```java
   @RestController
   public class PaymentController {
       @Value("${server.port}")
       private String serverPort;
   
       public static HashMap<Long, Payment> hashMap = new HashMap<>();
       static
       {
           hashMap.put(1L,new Payment(1L,"28a8c1e3bc2742d8848569891fb42181"));
           hashMap.put(2L,new Payment(2L,"bba8c1e3bc2742d8848569891ac32182"));
           hashMap.put(3L,new Payment(3L,"6ua8c1e3bc2742d8848569891xt92183"));
       }
   
       @GetMapping(value = "/getPayment/{id}")
       public CommonResult<Payment> getPayment(@PathVariable("id") Long id) {
           Payment payment = hashMap.get(id);
           CommonResult<Payment> result = new CommonResult(200,"serverPort: "+serverPort,payment);
           return result;
       }
   }
   ```

2. 创建服务消费模块cloudalibaba_consumer_nacos_order84

   1） 引入依赖

   ```xml
   <dependencies>
       <!-- SpringCloud ailibaba nacos -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
       <!-- SpringCloud ailibaba sentinel -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
       </dependency>
       ...
   ```

   2）application 配置

   ```yaml
   server:
     port: 84
   
   spring:
     application:
       name: nacos-order-consumer
     cloud:
       nacos:
         discovery:
           server-addr: 192.168.178.100:8848
       sentinel:
         transport:
           # 配置Sentinel dashboard 地址
           dashboard: localhost:8080
           # 默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
           port: 8719
   
   
   # 消费者将要访问的微服务名称（nacos 注册的微服务）
   service-url:
     nacos-user-service: http://nacos-payment-provider
   ```

   3）主启动类标注@EnableDiscoveryClient

##### 结合 Ribbon 

1. 注入RestTemplate 组件

   ```java
   @Configuration
   public class ApplicationContextConfig {
       @Bean
       @LoadBalanced
       public RestTemplate getRestTemplate() {
           return new RestTemplate();
       }
   }

2. 业务类，配置blockHandler 和fallback

   ```yaml
   @RestController
   public class CircleBreakerController {
       @Resource
       private RestTemplate restTemplate;
       public static final String SERVICE_URL = "http://nacos-payment-provider";
   
       @RequestMapping("/ribbon/payment/{id}")
       // blockHandler 处理sentinel 规则异常，fallback 处理业务异常
       @SentinelResource(value = "getPayment",blockHandler = "blockHandler",fallback = "fallbackHandler")
       public CommonResult<Payment> getPayment(@PathVariable Long id) {
           CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/getPayment/"+id,CommonResult.class,id);
           if (id<1 || id>3) {
               throw new IllegalArgumentException ("IllegalArgumentException,there is no data");
           }else if (result.getData() == null) {
               throw new NullPointerException ("NullPointerException,id = null");
           }
           return result;
       }
   
       public CommonResult fallbackHandler(@PathVariable Long id, Throwable e) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(400,"fallbackHandler",payment);
       }
   
       public CommonResult blockHandler(@PathVariable  Long id, BlockException blockException) {
           Payment payment = new Payment(id,"null");
           return new CommonResult<>(400,"blockHandler",payment);
       }
   }

4. Sentinel Dashboard 配置规则
5. 不断发出请求或设置id 值引发异常，观察响应结果，普通异常由fallback 处理，但设置熔断规则后，异常达到熔断条件，直接返回blockHandler 结果

##### 结合 OpenFeign

1. cloudalibaba_consumer_nacos_order84 引入依赖

   ```shell
   <!-- SpringCloud openfeign -->
   <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. application 配置

   ```yaml
   # 激活Feign 对Sentinel 的支持
   feign:
     sentinel:
       enabled: true
   ```

3. 服务调用映射和降级方法

   ```java
   @FeignClient(value = "nacos-payment-provider",fallback = PaymentFallbackService.class)
   public interface PaymentService {
       @GetMapping(value = "/getPayment/{id}")
       public CommonResult<Payment> getPayment(@PathVariable("id") Long id);
   }
   ```

   ```java
   @Component
   public class PaymentFallbackService implements PaymentService {
       @Override
       public CommonResult<Payment> getPayment(Long id) {
           return new CommonResult<>(400,"PaymentFallbackService OpenFeign 降级",new Payment(id, ""));
       }
   }
   ```

4. 增加业务方法

   ```java
   // OpenFeign
   @Resource
   private PaymentService paymentService;
   
   @GetMapping(value = "/feign/getPayment/{id}")
   public CommonResult<Payment> feignGetPayment(@PathVariable("id") Long id) {
       if(id<1 || id>3) {
           throw new RuntimeException("没有该id");
       }
       return paymentService.getPayment(id);
   }
   ```

5. 主启动类标注@EnableFeignClients
6. 设置Sentinel Dashboard 规则

### 2.7 规则持久化

被Sentinel 监控的微服务重启后，之前设置的规则将被清除，为避免大量的重置操作，需要将规则持久化

Sentinel 可以将规则持久化到数据库、Nacos 等中，现操作将其持久化至Nacos

ps：只能手动将配置写入Nacos，Sentinel 再从中读取，不能将Sentinel 中规则保存至Nacos

1. 引入依赖

   ```xml
   <!--SpringCloud ailibaba sentinel-datasource-nacos -->
   <dependency>
       <groupId>com.alibaba.csp</groupId>
       <artifactId>sentinel-datasource-nacos</artifactId>
   </dependency>
   ```

2. application 配置数据源

   ```yaml
   spring:
     cloud:
       sentinel:
         datasource:
           ds1:
             nacos:
               server-addr: 192.168.178.100:8848
               # Nacos 注册的微服务名称，同时也是Nacos 配置信息的Data Id
               dataId: cloudalibaba-sentinel-service
               groupId: DEFAULT_GROUP
               # 配置信息的数据格式
               data-type: json
               rule-type: flow

3. 在Nacos 配置规则数据

   ```json
   [
       {
           "resource": "/byResource",
           "limitApp": "default",
           "grade": 1,
           "count": 1,
           "strategy": 0,
           "controlBehavior": 0,
           "clusterMode": false
       }
   ]
   
   # resource：资源名称
   # limitApp：来源应用
   # grade：阈值类型，0表示线程数，1表示QPS
   # count：单机阈值
   # strategy：流控模式，0表示直接，1表示关联，2表示链路
   # controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待
   # clusterMode：是否集群
   ```

4. 启动模块，访问数次/byResource 后可在Dashboard 查看到设置的规则，停止微服务，规则又消失，重启又出现

## 3. Seata（Alibaba）

官网：http://seata.io/zh-cn/

版本：seata-server-0.9.0.zip

Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

### 3.1 基本概述

#### 重要概念

* Transaction ID

  全局唯一的事务ID

* Transaction Coordinator (TC)

  事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚

* Transaction Manager (TM)

  控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议

* Resource Manager (RM)

  控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚

#### 执行步骤

1. TM 向TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID
2. XID 在微服务调用链路的上下文中传播
3. RM 向TC 注册分支事务，将其纳入XID 对应全局事务的管辖
4. TM 向TC 发起针对XID 的全局提交或回滚决议
5. TC 调度XID 下管辖的全部分支事务完成提交或回滚请求

### 3.2 Seata Server

1. 下载：http://seata.io/zh-cn/blog/download.html

2. 解压

3. 修改配置文件 `seata-server-0.9.0\seata\conf\file.conf`

   ```shell
   # 自定义事务组名称，默认default 分组
   service {
     #vgroup->rgroup
     vgroup_mapping.my_test_tx_group = "fsp_tx_group"
     #only support single node
     default.grouplist = "127.0.0.1:8091"
     ...
   }
   
   # 修改事务日志存储模式为db + 配置数据库连接信息
   ## transaction log store
   store {
     ## store mode: file、db
     mode = "db"
     
     ...
     
     ## database store
     db {
       ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
       datasource = "dbcp"
       ## mysql/oracle/h2/oceanbase etc.
       db-type = "mysql"
       driver-class-name = "com.mysql.jdbc.Driver"
       url = "jdbc:mysql://192.168.178.100:3306/seata"
       user = "root"
       password = "root"
   	...
   }
   ```

4. 建库（seata）建表，其中建表sql 脚本位于`\seata-server-0.9.0\seata\conf\db_store.sql`，结果创建branch_table、global_table、lock_table 三个表

5. 修改配置文件`\seata-server-0.9.0\seata\conf\registry.conf`

   ```shell
   # 指明注册中心为nacos 并设置nacos 连接地址:端口号
   registry {
     # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
     type = "nacos"
   
     nacos {
       serverAddr = "192.168.178.100:8848"
       namespace = ""
       cluster = "default"
     }
     ...
   ```

6. 先启动Nacos，再启动seata-server.bat（0.9 版本只适配JDK1.8）

### 3.3 数据库准备

seata_order：存储订单的数据库

seata_storage：存储库存的数据库

seata_account：存储账户信息的数据库

1. 建库

   ```sql
   CREATE DATABASE seata_order;
   CREATE DATABASE seata_storage;
   CREATE DATABASE seata_account;
   ```

2. seata_order 库建表 t_order

   ```sql
   CREATE TABLE t_order (
     `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
     `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
     `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
     `count` INT(11) DEFAULT NULL COMMENT '数量',
     `money` DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
     `status` INT(1) DEFAULT NULL COMMENT '订单状态：0：创建中；1：已完结' 
   ) ENGINE=INNODB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

3. seata_storage库下建表 t_storage

   ```sql
   CREATE TABLE t_storage (
    `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
    `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
    `total` INT(11) DEFAULT NULL COMMENT '总库存',
    `used` INT(11) DEFAULT NULL COMMENT '已用库存',
    `residue` INT(11) DEFAULT NULL COMMENT '剩余库存'
   ) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
   ```

4. seata_account库下建表 t_account 

   ```sql
   CREATE TABLE t_account (
     `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'id',
     `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
     `total` DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
     `used` DECIMAL(10,0) DEFAULT NULL COMMENT '已用余额',
     `residue` DECIMAL(10,0) DEFAULT '0' COMMENT '剩余可用额度'
   ) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

5. 业务数据库都执行`\seata-server-0.9.0\seata\conf\db_undo_log.sql` 脚本，建立回滚日志表 undo_log

### 3.4 微服务准备

#### seata_order_service2001

1. 引入依赖

   ```xml
   <dependencies>
       <!-- nacos -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
       <!-- seata -->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
           <exclusions>
               <exclusion>
                   <artifactId>seata-all</artifactId>
                   <groupId>io.seata</groupId>
               </exclusion>
           </exclusions>
       </dependency>
       <dependency>
           <groupId>io.seata</groupId>
           <artifactId>seata-all</artifactId>
           <version>0.9.0</version>
       </dependency>
       <!-- feign-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-openfeign</artifactId>
       </dependency>
       <!-- web-actuator-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-actuator</artifactId>
       </dependency>
       <!-- mysql-druid-->
       <dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>5.1.37</version>
       </dependency>
   	...
   </dependencies>
   ```
   
2. application 配置

   ```yaml
   server:
     port: 2001
   
   spring:
     application:
       name: seata-order-service
     cloud:
       alibaba:
         seata:
           # 自定义事务组名称，需要与配置文件修改信息相匹配
           tx-service-group: fsp_tx_group
       nacos:
         discovery:
           server-addr: 192.168.178.100:8848
     datasource:
       driver-class-name: com.mysql.jdbc.Driver
       url: jdbc:mysql://localhost:3306/seata_order
       username: root
       password: root
   
   ...
   ```

3. 复制Seata 配置文件file.conf、registry.conf 至Resource 文件夹

4. mapper、config、service、controller、主启动类

5. 相同步骤设置seata_storage_service2002、seata_account_service2003

#### 事务注解

1. 问题：service 远程调用出错，这个调用之前的操作应该失效，但未经处理的代码将保存出错前的修改

2. 标注service 方法@GlobalTransactional

   ```java
   @Override
   // name=标识符，rollbackFor=执行回滚的异常
   @GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
   public void create(Order order) {
           ...

