# Spring Cloud

> **Spring Cloud Hoxton.SR12**

## 相关概念

**服务提供者 Provider**

实现业务逻辑，同时暴露接口以供外部调用的程序。

**服务消费者 Consumer**

调用另一个服务提供的接口，实现特定任务的程序。

**负载均衡 Load Balance**

根据策略把请求分发给服务的多个提供者，减小单台机器的压力，提高系统的可用性。

**注册中心**

公共平台，用于各个服务注册信息和相互发现，解耦提供者和消费者，提高系统的拓展性。

**服务治理**

服务拆分以后遇到的各种问题和解决方案的集合，比如鉴权、限流、降级、熔断、监控。

**服务降级**

服务失败时的冗余措施，通常返回错误提示，类似异常处理器。

**服务熔断**

根据策略，阻断所有请求，直接返回执行失败。

## CAP 原则

**初步认识**

加州大学的计算机科学家 Eric Brewer 提出分布式系统的 3 个指标：

* Consistency（一致性）：请求无论发送到哪个节点，返回的都是最新数据；
* Availability（可用性）：即使有节点挂掉，仍有其它节点提供服务，但不保证数据最新；
* Partition Tolerance（分区容错）：即使发生分区，系统仍能提供满足一致性或可用性的服务。

> 系统可能发生故障，导致处于不同子网的节点相互隔绝，形成几个独立区域，这些就是分区。

分布式系统无法同时满足这 3 个特性，而且分区容错必须实现，因为分区问题无法避免。

**取舍策略**

* **CP without A**：遇到分区，挂起请求，直到所有节点完成数据同步。Zookeeper，Consul。
* **AP wihtout C**：遇到分区，正常返回，但可能是尚未更新的旧数据。Eureka。

如果看重用户体验，应该选择 AP；如果对数据特别敏感，比如拍卖场景，那就选择 CP；另外，即使 CP 通常也要实现弱一致性或最终一致性。

## 依赖管理

因为 Spring Cloud Netflix 停止更新，Spring Cloud 2020.0.0 开始，停用 Netflix 系列组件，Horton.SR12 是支持这些组件的最高版本，适配 Spring Boot 2.3。

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.12.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<!-- Spring Cloud 版本管理 -->
<dependencyManagement>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>${spring-cloud.version}</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
</dependencyManagement>

<properties>
    <spring-cloud.version>Horton.SR12</spring-cloud.version>
</properties>
```

# 注册中心 Eureka

## 初步认识

Eureka 是 Spring Cloud Netflix 系列组件，用于注册服务。分布式系统的服务向 Eureka 注册，Eureka 保存这些服务的节点信息，消费者向 Eureka 请求服务列表，然后进行远程调用、负载均衡。

**软件架构**

Eureka 基于 C/S 架构设计，服务端是注册中心，客户端是系统的服务。

**心跳机制**

Eureka 客户端注册之后，需要定时向服务端发送心跳，默认周期 30 秒。如果服务端连续多个周期没有收到某个客户端的心跳，默认 3 个周期 90 秒，那就认为这个服务挂掉，把它从注册表剔除。

**自我保护**

如果 Eureka Server 某段时间内收到的心跳总数小于预期，那就进入自我保护状态：

* 拒绝移除长时间没有心跳的服务；
* 仍然支持服务注册和查询请求，但是不会同步到其它节点；
* 直到网络稳定，Eureka Server 把新的注册信息同步到其它节点。

默认情况，统计时间是 15 分钟，预期值是理想值 85%，理想值 = 节点数量 * 30秒/次。

**注册缓慢**

Eureka Client 注册之后，可能持续一段时间无法被服务发现，有以下原因：

* Eureka Server 使用二级缓存：读写缓存 + 只读缓存，前者缓存周期是 180 秒，后者默认每 30 秒从读写缓存同步数据。客户端请求先查只读缓存，没有命中再查读写缓存，还没有就加载存储数据到缓存；
* Eureka Client 缓存服务注册表，默认每 30 秒从 Eureka Server 更新数据；
* Ribbon 维护 ServerList 列表，默认每 30 秒从 Eureka Client 更新数据。

所以，极端情况，新的服务可能要 30 + 30 + 30 秒才会被发现。

## 服务端

引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

启用服务

```
@EnableEurekaServer
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

配置属性，Eureka Server 只是平台，没有必要注册自己或拉取信息。

```
server:
  port: 8091

eureka:
  client:
    register-with-eureka: false # 是否进行注册
    fetch-registry: false # 是否拉取注册表
    service-url: # 注册中心地址
      defaultZone: http://localhost:8091/eureka
```

修改 `eureka.service-url.defaultZone` 属性搭建集群，各个节点平行独立，通过相互注册提高可用性，注册信息将会分享给配置的其它节点。

```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8091/eureka, http://localhost:8092/eureka
```

Eureka Server 提供一个页面，可以观察注册服务的简单信息。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/netflix-cloud-79cb0354.png)

## 客户端

引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

启用服务

```
@EnableEurekaClient
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

配置属性

```
spring:
  application:
    name: provider # 服务名

server:
  port: 8001

eureka:
  instance:
    hostname: localhost # 主机名，可为IP地址
    instance-id: provider-01 # 节点名，区分相同服务的不同提供者
    prefer-ip-address: true # 使用IP地址，默认主机名
  client:
    service-url:
      defaultZone: http://localhost:8091/eureka
```

# 负载均衡 Ribbon

Ribbon 是 Spring Cloud Netflix 系列组件，用于负载均衡，它在消费者端使用。Ribbon 维护一个列表，保存所有服务节点信息，ServerList 源于注册中心或手动配置，支持动态更新。

Eureka 包含 Ribbon，可以理解，毕竟它们设计之初就是为联合使用。

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

## 原始调用

添加 `RestTemplate` 组件，这个类由 Spring 提供，用于发送 HTTP 请求。

```
@Configuration
public class ConsumerConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

处理器方法，使用 `LoadBalancerClient` 手动向 Eureka 获取节点信息，这个类由 Spring Cloud 提供。

```
@RestController
public class ConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private LoadBalancerClient balancerClient;

    /**
     * 写死服务提供者地址
     */
    @GetMapping("/consumers/origin/{code}")
    public HashMap<String, String> consume01(@PathVariable String code) {
        String url = "http://localhost:8001/providers/" + code;
        return restTemplate.getForObject(url, HashMap.class);
    }

    /**
     * 向Eureka请求提供者信息
     */
    @GetMapping("/consumers/{code}")
    public HashMap<String, String> consume02(@PathVariable String code) {
        ServiceInstance provider = balancerClient.choose("provider");
        String host = provider.getHost();
        int port = provider.getPort();
        String url = "http://" + host + ":" + port + "/providers/" + code;
        return restTemplate.getForObject(url, HashMap.class);
    }
}
```

## 负载均衡

使用 `@LoadBalanced` 标注 `RestTemplate` 组件，启用负载均衡。

```
@Configuration
public class ConsumerConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

处理器方法，使用**服务名**作为调用地址，Ribbon 根据策略和服务列表拼接真实 IP 地址，直接 IP 地址错误。

```
@RestController
public class ConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    /**
     * 使用Ribbon实现负载均衡
     */
    @GetMapping("/consumers/ribbon/{code}")
    public HashMap<String, String> consume03(@PathVariable String code) {
        String uri = "/providers/" + who;
        String url = "http://" + "provider" + uri; // 使用Eureka注册的服务名作为地址
        return restTemplate.getForObject(url, HashMap.class);
    }
}
```

## 负载策略

`IRule` 接口表示负载均衡策略，Ribbon 提供许多它的实现，默认采用轮询。

| 负载均衡策略实现           | 说明                         |
| -------------------------- | ---------------------------- |
| `RoundRobinRule`           | 轮询                         |
| `RandomRule`               | 随机                         |
| `RetryRule`                | 轮询 + 重试                  |
| `WeightedResponseTimeRule` | 优先选择响应速度更快的提供者 |

可以实现 `IRule`，自定义负载均衡策略，注册它为 Bean 进行启用。

```
@Configuration
public class ConsumerConfig {

    @Bean
    public IRule iRule() {
        return new RoundRobinRule();
    }
}
```

## 服务检测

执行负载均衡之前，Ribbon 需要检测节点是否可用，`IPing` 接口表示检测逻辑，Ribbon 提供许多它的实现。

| 服务检测实现        | 说明                          |
| ------------------- | ----------------------------- |
| `NIWSDiscoveryPing` | 使用 Eureka 反馈，默认        |
| `PingUrl`           | 使用 `HttpClient` Ping 功能   |
| `DummyPing`         | 默认返回 `true`，除非发生错误 |
| `NoOpPing`          | 直接返回 `true`               |

可以实现 `IPing`，自定义服务检测，注册它为 Bean 进行启用。

```
@Configuration
public class ConsumerConfig {

    @Bean
    public IPing iPing() {
        return new PingUrl(false, "/"); // 非安全协议，检测URI="/"
    }
}
```

# 服务降级 Hystrix

## 初步认识

Hystrix 是 Spring Cloud Netflix 系列组件，用于服务降级，同时支持熔断、缓存。注意，Hystrix 本身与业务实现无关，有没有它系统都能运行。

**运行流程**

`run()`/`construct()` 表示 Hystrix 修饰的内容，通常是远程服务调用或服务本身，这取决于 Hystrix 位于消费者或提供者。Hystrix 包裹服务，给它一个保底措施，如果失败、超时，返回友好的内容而非异常，这有点类似异常处理器。Hystrix 熔断直接拒绝请求，控制流量，它会动态调整状态，随时恢复正常，这在请求剧增时能保证系统正常运行，而不会被大量请求瞬间压垮。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/netflix-cloud-42d049ed.png)

**引入依赖**

独立 Hystrix，后面再讨论系统集成。

```
<dependency>
    <groupId>com.netflix.hystrix</groupId>
    <artifactId>hystrix-core</artifactId>
    <version>hystrix.version</version>
</dependency>
```

## 命令对象

根据运行流程图，可知 `HystrixCommand`、`HystrixObservableCommand ` 是 Hystrix 使用入口，这两个命令对象封装 Hystrix 各种特性，它们共有 4 种执行方式，重点掌握 `queue()`。

命令对象实例化时必须设置 GroupKey，这个属性用于分组监控和报警信息。

**HystrixCommand**

继承 `HystrixCommand`，重写构造器和 `run()`。

```
public class MyHystrixCommand extends HystrixCommand<String> {

    private String name;

    public MyHystrixCommand(String groupName) {
    	// 设置GroupKey，name是自定的属性，用于标识对象
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupName)));
        this.name = groupName;
    }

    /**
     * 业务逻辑
     */
    @Override
    protected String run() throws Exception {
        String result = "MyHystrixCommand-" + name;
        System.err.println("[Run] " + result + 
        		", CurrentThread-" + Thread.currentThread().getName());
        Thread.sleep(800); // 模拟业务耗时
        return result;
    }
}
```

**HystrixObservableCommand**

继承 `HystrixObservableCommand`，重写构造器和 `construct()`，后者返回 `Observable` 对象，这个对象用于阻塞或非阻塞调用。对于非阻塞调用，`construct()` 至多调用一次 `onNext()`。

> `HystrixCommand` 默认使用线程池运行，`HystrixObservableCommand` 默认使用调用线程运行。

```
public class MyObserveHystrixCommand extends HystrixObservableCommand<String> {

    private String name;

    public MyObserveHystrixCommand(String groupName) {
        super(HystrixObservableCommand.Setter
        		.withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupName)));
        this.name = groupName;
    }

    @Override
    protected Observable<String> construct() {
        String result = "MyObserveHystrixCommand-" + name;
        System.err.println("[Construct] " + result + 
        		", CurrentThread-" + Thread.currentThread().getName());
        try {
            Thread.sleep(800);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                // 具体业务逻辑放在onNext方法
                subscriber.onNext("next-1");
                subscriber.onNext("next-2");
                subscriber.onNext("next-3");
                // 业务结束
                subscriber.onCompleted();
            }
        }).subscribeOn(Schedulers.io());
    }
}
```

## 执行方式

### 同步执行 Execute

底层调用 `queue().get()`，所以也是异步执行，只是使用阻塞调用获取结果。

```
@Test
public void executeTest() {
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand myHystrixCommand = new MyHystrixCommand("executeGroup");
    String result = myHystrixCommand.execute();
    long endTime = System.currentTimeMillis();
    System.err.println("[execute end] " + result + 
    		", speeding : " + (endTime - beginTime));
}
```

### 异步执行 Queue

看到 `Future` 对象，知道这是多线程异步运行，底层使用线程池。

```
@Test
public void queueTest() throws ExecutionException, InterruptedException {
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand myHystrixCommand = new MyHystrixCommand("queueGroup");
    // 异步执行，返回Future对象
    Future<String> future = myHystrixCommand.queue();
    long endTime = System.currentTimeMillis();
    System.err.println("[future end] " + "speeding : " + (endTime - beginTime));
    String result = future.get();
    endTime = System.currentTimeMillis();
    System.err.println("[future get] " + result + 
    		", speeding : " + (endTime - beginTime));
}
```

### 观察者 Observe

返回 `Observable` 对象，阻塞或非阻塞调用，非阻塞调用需要设置几个回调方法处理返回结果。

```
@Test
public void observeBlockTest() {
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand myHystrixCommand = new MyHystrixCommand("executeGroup");
    Observable<String> observe = myHystrixCommand.observe();
    // 阻塞式调用，类似Execute
    String blockResult = observe.toBlocking().single();
    long endTime = System.currentTimeMillis();
    System.err.println("[Observe Block] " + blockResult + 
    		", speeding : " + (endTime - beginTime));
}
```

这个 `Observable` 是 Hot 对象，首先运行 `run()`，然后才加载 `Subscriber` 对象并执行回调。

```
@Test
public void observeSubscribe() throws InterruptedException {
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand myHystrixCommand = new MyHystrixCommand("executeGroup");
    Observable<String> observe = myHystrixCommand.observe();
    // 非阻塞式调用
    observe.subscribe(new Subscriber<String>() {
        @Override
        public void onCompleted() {
            System.err.println("[Observe Subscribe OnCompleted]");
        }

        @Override
        public void onError(Throwable throwable) {
            System.err.println("[Observe Subscribe OnError]");
        }

        @Override
        public void onNext(String result) {
            long endTime = System.currentTimeMillis();
            System.err.println("[Observe Subscribe OnNext] " + result + 
            		", speeding : " + (endTime - beginTime));
        }
    });
    Thread.sleep(2000); // 等待非阻塞调用，否则它会随着主线程结束
}
```

### 观察者 ToObservable

`toObservable()` 类似 `observe()`，返回 `Observable` 对象，这个对象只能执行一次。

```
@Test
public void toObserveBlockTest() {
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand myHystrixCommand = new MyHystrixCommand("executeGroup");
    // 阻塞式调用
    Observable<String> toObservable = myHystrixCommand.toObservable();
    String blockResult = toObservable.toBlocking().single();
    long endTime = System.currentTimeMillis();
    System.err.println("[ToObservable Block] " + blockResult + 
    		", speeding : " + (endTime - beginTime));
}
```

这个 `Observable` 是 Cold 对象，首先加载 `Subscriber` 对象，然后才运行 `run()`，最后执行回调。

```
@Test
public void toObserveSubscribe() throws InterruptedException {
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand myHystrixCommand = new MyHystrixCommand("executeGroup");
    Observable<String> toObserve = myHystrixCommand.toObservable();
    // 非阻塞式调用
    toObserve.subscribe(new Subscriber<String>() {
        @Override
        public void onCompleted() {
            System.err.println("[Observe Subscribe OnCompleted]");
        }

        @Override
        public void onError(Throwable throwable) {
            System.err.println("[Observe Subscribe OnError]");
        }

        @Override
        public void onNext(String result) {
            long endTime = System.currentTimeMillis();
            System.err.println("[Observe Subscribe OnNext] " + result + 
            		", speeding : " + (endTime - beginTime));
        }
    });
    Thread.sleep(2000); // 等待非阻塞调用，否则它会随着主线程结束
}
```

## 请求缓存

Hystrix 会缓存请求结果，即 `run()`/`construct()` 返回。首先需要开启缓存，默认开启。

```
public MyHystrixCommand(String groupName) {
    super(Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupName))
            .andCommandPropertiesDefaults(
            	HystrixCommandProperties.
                	defaultSetter().withRequestCacheEnabled(true) // 缓存开关
            )
    );
    this.name = groupName;
}
```

重写 `getCacheKey()`，返回值是缓存的 Key，命令对象使用它存储和查找缓存。

```
@Override
public String getCacheKey() {
    return String.valueOf(this.name);
}
```

只有处于相同请求上下文的请求才能分享缓存，命中缓存后直接返回而不再执行业务。

```
@Test
public void cacheTest() {
    // 开启请求上下文
    HystrixRequestContext requestContext = HystrixRequestContext.initializeContext();
    long beginTime = System.currentTimeMillis();
    MyHystrixCommand command_1 = new MyHystrixCommand("command_1");
    MyHystrixCommand command_2 = new MyHystrixCommand("command_2");
    MyHystrixCommand command_3 = new MyHystrixCommand("command_1");
    String result_1 = command_1.execute();
    System.err.println("[execute end] " + result_1 + 
    		", speeding : " + (System.currentTimeMillis() - beginTime));
    String result_2 = command_2.execute();
    System.err.println("[execute end] " + result_2 + 
    		", speeding : " + (System.currentTimeMillis() - beginTime));
    // 直接返回command_1结果，没有执行run()，耗时非常短
    String result_3 = command_3.execute();
    System.err.println("[execute end] " + result_3 + 
    		", speeding : " + (System.currentTimeMillis() - beginTime));
    // 关闭请求上下文
    requestContext.close();
}
```

## 请求合并

Hystrix 可以把多个请求合并为一个请求，只要它们处于相同请求上下文并且调用时间足够近。

继承 `HystrixCollapser` 对象，重写相关方法，定义批量处理逻辑。

```
// 泛型分别表示：批量处理结果类型、单次处理结果类型、请求参数类型
public class MyCollapserCommand extends HystrixCollapser<List<String>, String, Integer> {

    private Integer id;

    public MyCollapserCommand(Integer id) {
        super(Setter.withCollapserKey(
        		HystrixCollapserKey.Factory.asKey("MyCollapserCommand"))
        );
        this.id = id;
    }

    /**
     * 获取请求参数，这是业务相关的数据
     */
    @Override
    public Integer getRequestArgument() {
        return id;
    }

    /**
     * 批量业务处理
     */
    @Override
    protected HystrixCommand<List<String>> 
    		createCommand(Collection<CollapsedRequest<String, Integer>> collection) {
        return new BatchCommand(collection);
    }

    /**
     * 批量处理结果和请求之间的映射
     */
    @Override
    protected void mapResponseToRequests(
    					List<String> strings, // 批量处理结果
                        Collection<CollapsedRequest<String, Integer>> collection) {
        int idx = 0;
        for (CollapsedRequest<String, Integer> reqest : collection) {
            reqest.setResponse(strings.get(idx++));
        }
    }
}

/**
 * 处理批量请求逻辑
 */
class BatchCommand extends HystrixCommand<List<String>> {

    private Collection<HystrixCollapser.CollapsedRequest<String, Integer>> list;

    protected BatchCommand(
    		Collection<HystrixCollapser.CollapsedRequest<String, Integer>> list) {
        super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("batchCommand")));
        this.list = list;
    }

    @Override
    protected List<String> run() throws Exception {
        System.err.println("[BatchCommand Run] CurrentThread-" 
        		+ Thread.currentThread().getName());
        List<String> result = new ArrayList<String>();
        for (HystrixCollapser.CollapsedRequest<String, Integer> item : list) {
            Integer arg = item.getArgument();
            result.add("Argument_" + arg);
        }
        return result;
    }
}
```

测试请求合并，如果成功，`run()` 应该运行较少次数，但是成功打印所有结果。

```
@Test
public void collapserTest() throws ExecutionException, InterruptedException {
    // 开启请求上下文
    HystrixRequestContext requestContext = HystrixRequestContext.initializeContext();

    MyCollapserCommand c1 = new MyCollapserCommand(1);
    MyCollapserCommand c2 = new MyCollapserCommand(2);
    MyCollapserCommand c3 = new MyCollapserCommand(3);
    MyCollapserCommand c4 = new MyCollapserCommand(4);
    MyCollapserCommand c5 = new MyCollapserCommand(5);

    Future<String> q1 = c1.queue();
    Future<String> q2 = c2.queue();
    Future<String> q3 = c3.queue();
    Future<String> q4 = c4.queue();
    Future<String> q5 = c5.queue();

    String r1 = q1.get();
    String r2 = q2.get();
    String r3 = q3.get();
    String r4 = q4.get();
    String r5 = q5.get();

    System.err.println(r1 + "," + r2 + "," + r3 + "," + r4 + "," + r5);
    // 关闭请求上下文
    requestContext.close();
}
```

可在实例化时进行配置，比如请求之间距离多近可以被合并。

```
public MyCollapserCommand(Integer id) {
    super(Setter
            .withCollapserKey(HystrixCollapserKey.Factory.asKey("MyCollapserCommand"))
            .andCollapserPropertiesDefaults(
                    HystrixCollapserProperties.defaultSetter()
                            .withTimerDelayInMilliseconds(1000) // 请求间隔1秒之内可被合并
            )
    );
    this.id = id;
}
```

## 运行隔离

命令对象的运行应该相互隔离，否则单个服务失败就有可能导致系统失败。Hystrix 支持线程和信号量两种隔离方式，线程天然具有隔离性，信号量类似计数器，相对轻量，因为没有线程开销，适合没有网络开销的调用。

默认使用线程隔离，底层依靠线程池实现。属性 ThreadPoolKey 指定线程池实例的名字，默认 GroupKey，线程池可配置，比如 CoreSize、MaximumSize，如下所示。

```
public MyHystrixCommand(String groupName) {
    super(Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupName))
            .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey(groupName)) // 线程池名字
            .andThreadPoolPropertiesDefaults( // 配置线程池
                    HystrixThreadPoolProperties.defaultSetter()
                            .withCoreSize(3)
                            .withx(8)
                            .withMaxQueueSize(16)
                            .withKeepAliveTimeMinutes(1)
            )
            .andCommandPropertiesDefaults(
	            HystrixCommandProperties.defaultSetter()
                	.withExecutionIsolationStrategy( // 启用线程隔离
                    	HystrixCommandProperties.ExecutionIsolationStrategy.THREAD)
            )
    );
    this.name = groupName;
}
```

设置信号量隔离，命令对象使用调用线程运行。

```
public MyHystrixCommand(String groupName) {
    super(Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupName))
            .andCommandPropertiesDefaults(
            	HystrixCommandProperties.defaultSetter()
                	.withExecutionIsolationStrategy( // 启用信号量隔离
                    	HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE)
            )
    );
    this.name = groupName;
}
```

## 服务降级

重写 `getFallback()`，定义冗余处理逻辑。

```
public class MyHystrixCommand extends HystrixCommand<String> {

	//...
    
    @Override
    protected String getFallback() {
        String result = "MyHystrixCommand-" + name;
        System.err.println("[Fallback] " + result + 
        		", CurrentThread-" + Thread.currentThread().getName());
        return result;
    }
}
```

服务降级默认拦截所有异常，除了 `HystrixBadRequestException`，当然，这些都可以修改。

## 请求熔断

Hystrix 以时间窗为单位统计两个指标：请求总数、失败数量。如果这个时间段内请求数量达到阈值，然后失败数量也达到一定比例，Hystrix 就会开启熔断。

熔断状态，Hystrix 拒绝所有请求，直接执行 `fallback()`，参看运行流程图。

默认熔断 5 秒之后，Hystrix 进入半熔断状态，放行一次请求，如果请求成功执行，恢复正常，否则继续熔断。

```
public MyHystrixCommand(String groupName) {
    super(Setter
            .withGroupKey(HystrixCommandGroupKey.Factory.asKey(groupName))
            .andCommandPropertiesDefaults(
            	HystrixCommandProperties.defaultSetter()
					// 配置熔断
                    .withCircuitBreakerEnabled(true) // 开启熔断
                    //.withCircuitBreakerForceOpen(false) // 强制熔断
                    //.withCircuitBreakerForceClosed(false) // 强制关闭
                    .withCircuitBreakerRequestVolumeThreshold(2) // 请求总数阈值
                    .withCircuitBreakerErrorThresholdPercentage(50) // 失败比例阈值
                    .withCircuitBreakerSleepWindowInMilliseconds(5) // 半熔断开启时间
			)
    );
    this.name = groupName;
}
```

## 提供者集成

消费者或提供者两个端都可以使用 Hystrix，Feign 和 Zuul 可以集成 Hystrix，这是消费者端应用，它把远程调用操作当作 `run()` 修饰。这里学习提供者端使用，把处理器方法当作 `run()` 修饰。

引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

启用服务

```
@EnableHystrix
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

使用 `@HystrixCommand` 标注处理器方法，表示对其进行修饰，这里配置服务降级。

```
/**
 * 服务降级，它的方法签名应该和处理器方法相同
 */
public BaserResponseVO<Map<String, Object>> fallbackMethod(BasePageVO basePageVO) {
    HashMap<String, Object> map = new HashMap<>();
    map.put("error", "not found");
    return BaserResponseVO.success(map);
}

/**
 * 处理器方法
 */
@HystrixCommand(
		fallbackMethod = "fallbackMethod", // 指定服务降级
        ignoreExceptions = CommonServiceException.class // 忽略拦截异常
)
@GetMapping("/")
public BaserResponseVO<Map<String, Object>> getSomething(BasePageVO basePageVO) {
    //...
}
```

注解 `@HystrixCommand` 还能配置熔断、隔离这些功能。

```
@HystrixCommand(
	commandProperties = {
		@HystrixProperty(name = "execution.isolation.strategy", 
			value = "THREAD"), // 线程隔离
		@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", 
			value = "1000"), // 超时时间
		@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", 
			value = "10"), // 请求总数阈值
		@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", 
			value = "50") // 失败比例阈值
	},
	threadPoolProperties = { // 线程池配置
        @HystrixProperty(name = "coreSize", value = "1"),
        @HystrixProperty(name = "maxQueueSize", value = "10"),
        @HystrixProperty(name = "keepAliveTimeMinutes", value = "1000"),
        @HystrixProperty(name = "queueSizeRejectionThreshold", value = "8")
	}
)
@GetMapping("/")
public BaserResponseVO<Map<String, Object>> getSomething(BasePageVO basePageVO) {
    //...
}
```

# 远程调用 Feign

Feign 是 Spring Cloud Netflix 系列组件，用于远程调用，相比 `RestTemplate`，它的代码简单易读。

## 初步使用

引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

启用服务

```
@EnableFeignClients
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

使用接口定义远程调用，每个接口方法表示一种 HTTP 请求，属性 `name` 是标识符，`url` 是服务地址。

> 接口方法和对应的处理器方法几乎相同，Feign 支持 Spring MVC 部分注解，为了避免错误，所有请求参数都应该使用 `@RequestParam` 注解。
>
> 可用注解：`@RequestMapping`、`@RequestParam`、`@PathVariable`、`@RequestBody`。

```
@FeignClient(name = "feignClient", url = "http://localhost:8001")
public interface ConsumeFeign {

    @GetMapping("/providers/{who}")
    Map<String, String> provider01(@PathVariable("who") String who);
}
```

处理器方法，使用 `@FeignClient` 接口类型进行远程调用，Feign 使用动态代理技术创建实例。

```
@Slf4j
@RestController
public class ConsumerController {

    @Resource
    private ConsumeFeign consumeFeign;

    /**
     * Feign
     */
    @GetMapping("/consumers/feign/{who}")
    public HashMap<String, String> consume04(@PathVariable String who) {
        log.error("feign consumer");
        return (HashMap<String, String>) consumeFeign.provider01(who);
    }
}
```

## 注解参数

| `@FeignClient` 属性           | 说明               |
| ----------------------------- | ------------------ |
| `name`、`value`               | FeignClient 标识符 |
| `url`                         | 调用地址           |
| `Path`                        | URI 前缀           |
| `primary`                     | 优先级             |
| `configuration`               | 局部配置           |
| `Fallback`、`FallbackFactory` | 服务降级           |

### 优先实现

动态代理创建的 `@FeignClient` 接口类型实例默认被 `@Primary` 标注，修改属性 `primary = false` 表示移除优先注解，然后就能自动装配自己的接口实现。

```
@FeignClient(name = "provider", url = "http://localhost:8001", primary = false)
public interface ConsumeFeign {
	...
}
```

### 自定配置

默认行为已经足以解决大部分问题，Feign 提供可配置项以支持局部定制。

| 配置选项      | 说明和默认                                                   |
| ------------- | ------------------------------------------------------------ |
| FeignDecoder  | 解析器，默认 `ResponseEntityDecoder`                         |
| FeignEncoder  | 编码器，默认 `SpringEncoder`                                 |
| FeignLogger   | 日志，默认 `Slf4jLogger`                                     |
| FeignContract | 默认 `SpringMVCContract`，它是 Feign 支持 Spring MVC 注解的关键。 |
| FeignBuilder  | `HystrixFeign.Builder`                                       |
| FeignClient   | `LoadBalancerFeignClient` 或者 `FeignClient`，结合 Ribbon 则使用前者。 |

创建 `@Configuration` 类，通过注册组件修改 Feign 配置，这个类不能被 Spring 扫描，否则报错。

```
@Configuration
public class FeignConfig {

    /**
     * 使用默认Contract，现在Feign无法使用Spring MVC注解
     */
    @Bean
    public Contract contract() {
        return new Contract.Default();
    }
}
```

使用注解属性 `configuration` 指定 Feign 配置类。

```
@FeignClient(
	name = "provider", url = "http://localhost:8001", configuration = FeignConfig.class
)
public interface ConsumeFeign {
	...
}
```

## 组件整合

### 负载均衡 Ribbon

Feign 可与 Ribbon 整合，修改 `@FeignClient` 属性 `name`/`value` 为服务名，删除 `url` 属性。现在，这个接口的远程调用就会使用 Ribbon 负载均衡。

```
@FeignClient(name = "provider")
public interface ConsumeFeign {
	...
}
```

### 服务降级 Hystrix

Feign 可与 Hystrix 整合，定义远程调用的服务降级，首先开启支持。

```
feign.hystrix.enabled = true
```

**Fallback**

实现 `@FeignClient` 接口，重写方法定义服务降级逻辑。

```
@Component
public class ConsumeFeignFallback implements ConsumeFeign {

    @Override
    public Map<String, String> provider01(String who) {
        return new HashMap<String, String>(); // 降级返回空内容
    }
}
```

使用 `fallback` 属性为 `@FeignClient` 接口的所有远程调用设置服务降级。

```
@FeignClient(..., fallback = ConsumeFeignFallback.class)
public interface ConsumeFeign {
	...
}
```

**FallbackFactory**

另一种方式，实现 `FallbackFactory` 接口，重写方法，返回服务降级实现类。

```
@Component
public class ConsumeFeignFallbackFactory implements FallbackFactory<ConsumeFeign> {

    @Override
    public ConsumeFeign create(Throwable throwable) {
        return new ConsumeFeign() {
            @Override
            public Map<String, String> provider01(String who) {
                return new HashMap<String, String>(); // 降级返回空内容
            }
        };
    }
}
```

修改 `@FeignClient` 属性 `fallbackFactory`，指定服务降级的工厂类，这是工厂模式的实现。

```
@FeignClient(..., fallbackFactory = ConsumeFeignFallbackFactory.class)
public interface ConsumeFeign {
	...
}
```

## 性能优化

对于服务器，网络是比较珍贵的资源，怎样在有限带宽之下提高传输效率，可以考虑两个方面：优化客户端、压缩传输数据。

**传输方式**

Feign 默认使用 JDK 工具进行 HTTP 通信，它的性能不高，Feign 支持更改底层通信工具，以下是简单演示。

引入依赖，Apache HttpClient 目前是性能最高的 HTTP 客户端，它能显著提升通信效率。

```
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

启用相关支持

```
feign:
  httpclient:
    enabled: true # HttpClient
```

**压缩数据**

Feign 内置支持 Gzip 解压缩传输数据，压缩不一定提升性能，因为解压缩的耗时可能大于节省的网络传输时间。

```
feign:
  compression:
    request: # 压缩请求
      enabled: true # 开启
      mime-types: text/xml,application/xml,application/json # 支持压缩的数据类型
      min-request-size: 2048 # 使用压缩的最小数据大小
    response: # 压缩响应
      enabled: true
```

**接口继承**

某个服务可能被多个消费者使用，但它的调用方式无论在哪都差不多，如果每个消费者都要编写一套 Feign 调用接口，实在有点代码冗余，并且也不方便后续的统一修改。

`@FeignClient` 接口可以继承**一个**普通接口，所以，用户可以在一个公共普通接口声明远程调用方法，这些方法以及它们的 Spring MVC 注解都会被继承。现在定义远程调用，创建 `@FeignClient` 接口并且继承对应的公共普通接口即可。这会省去大量重复代码，并且方便统一修改。

# 服务网关 Zuul

Zuul 是 Spring Cloud Netflix 系列组件，用于网关，负责筛选和路由请求，可以把它看作过滤器的集合。

用户使用 URL 表达式定义路由规则，规定匹配的请求应该路由到哪个服务，Zuul 使用 Ribbon 调用远程服务并封装返回作为响应。所以，Zuul 需要集成 Eureka 和 Ribbon。Zuul 还能集成 Hystrix，设置调用端服务降级。

Zuul 会过滤请求的一些非安全信息，比如 Cookie、Authorization 字段，可用 `zuul.sensitive-headers` 属性自定义屏蔽字段，覆盖默认行为。

## 初步使用

引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>

<!-- Eureka、Ribbon、Hystrix -->
```

启用服务，不需开启 Eureka Client

```
@EnableZuulProxy
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

配置路由，规定使用 `film-service` 服务响应 */prefix/film-api* 开头的请求。

```
zuul:
  prefix: "/prefix" # URI前缀
  routes: # 路由规则
    film-service: # 服务名字
      path: "/film-api/**" # URI匹配
  ignored-services:
    film-service
```

属性 `zuul.routes` 表示路由规则，每条规则以服务名开始，`path` 是 URI 表达式，支持 ?、*、** 通配符，它是前缀匹配，Zuul 使用前缀之后的内容进行远程调用。

除了 `path` 映射，还能使用服务名匹配服务，但是这会暴露服务名，可用 `zuul.ignored-services` 禁用。

## 运行流程

Zuul 基于 Servlet + Filter 结构，非常简单。Zuul Servlet 负责接收请求并管理请求处理流程，Zuul Runner 管理过滤器，它把请求按序交给匹配的过滤器处理。Zuul 有 3 种类型过滤器，Routing Filters 负责路由，它会调用远程服务并封装响应，Pre 用于路由前处理，Post 用于路由后处理，它们都能影响请求和响应。

`RequestContext` 是 Zuul 对请求的进一步封装，它的重要特性是线程安全。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/netflix-cloud-b415b12f.png)

Zuul 过滤器的执行顺序和生命周期，Origin Server 表示远程服务。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/netflix-cloud-9cd1484e.png)

Zuul 内置许多过滤器，用于支持各种功能。

| 过滤器                    | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `PreDecorationFilter`     | 根据 RouteLocator 确定在哪和如何路由                         |
| `RibbonRoutingFilter`     | 通过 Ribbon 调用服务，只对有 ServiceID 参数的请求上下文生效。 |
| `SimpleHostRoutingFilter` | 通过 HttpClient 调用服务，只对 URL 配置的路由生效。          |

## 自定过滤

继承 `ZuulFilter` 实现自己的过滤器，把它注册为 Bean 就能进行启用。可以发现，`RequestContext` 类似线程本地变量，难怪它是线程安全。

```
@Slf4j
@Component
public class MyFilter extends ZuulFilter {

    /**
     * 过滤器类型
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * 过滤器优先级
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 是否启用
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 执行逻辑
     */
    @Override
    public Object run() throws ZuulException {
        // 获取当前请求上下文
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        Enumeration<String> headerNames = request.getHeaderNames();
        // 打印所有请求头字段
        while (headerNames.hasMoreElements()) {
            String name = headerNames.nextElement();
            log.error("[MyFilter] {}:{}", name, request.getHeader(name));
        }
        return null;
    }
}
```

## 服务降级 Hystrix

通过实现 `FallbackProvider` 接口定义降级服务，把它注册为 Bean 进行启用。

```
@Component
public class MyFallBack implements FallbackProvider {

    /**
     * 针对哪些路由修饰
     */
    @Override
    public String getRoute() {
        // return "film-service";
        return "*";
    }

    /**
     * 降级返回内容
     */
    @Override
    public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
        return new ClientHttpResponse() {

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }

            @Override
            public InputStream getBody() throws IOException {
                BaserResponseVO<Object> baserResponseVO = BaserResponseVO.fail(new CommonServiceException(404, "error happened"));
                String result = JSONObject.toJSONString(baserResponseVO);
                return new ByteArrayInputStream(result.getBytes());
            }

            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {
            }
        };
    }
}
```
