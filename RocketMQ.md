# RocketMQ

# 消息队列概念

消息队列（Message Queue）是一种软件，提供生产消息、传递消息、消费消息等功能，消息就是数据，生产者向 MQ 发送消息，消费者从 MQ 读取消息。

## 消息队列作用

MQ 最基本的用途是作为服务间的一种通信方式，相比 RPC，它有什么不同呢？

**程序解耦**

服务间使用 MQ 进行通信，上游服务把数据封装成消息发送给 MQ，然后返回。下游服务从 MQ 拉取消息并进行处理。这样，虽然上下游服务间没有直接联系，但却能执行完整的业务流程。

解耦使得分布式服务更易编写和维护，增加业务，只需新增服务订阅消息，删除业务则停止消费消息即可。

**异步执行**

生产者把消息发送给 MQ 后就直接返回，流程后续的步骤不需立即执行，至于什么时候执行？这取决于下游服务何时拉取消息。

因此，MQ 使业务的上下游能够异步运行，这能显著减少请求的返回时间，增加系统吞吐量。相比之下，RPC 是同步通信，远程调用需要获得返回才能继续执行。

**削峰填谷**

在高并发场景，服务可能来不及处理，这将导致大量请求堆积阻塞，甚至压垮服务。可以先把请求封装成消息发送到 MQ，然后返回。然后运行异步线程或子服务，尽可能快地拉取消息进行处理。这样，可以延缓请求处理，降低系统的瞬时压力。

## 消息传输模型

什么是消息传输模型？可以理解为消息的组织、使用方式，主流的传输模型有：队列模型，发布订阅模型。

**队列模型**

消息组织为队列结构，尾进头出，消息一旦被拉取，就会出队删除，所以，这种模型的消息只能被消费一次。正因如此，队列模型的消费者不需身份标识，反正消息只会消费一次，管它是被谁消费。

**发布订阅模型**

使用主题作为消息容器，消费者都有明确的身份标识，通常是订阅组，不同订阅组相互独立。主题通常支持持久化消息，并基于订阅组记录消费进度。单个主题可被多个订阅组消费，每个订阅组都能够消费主题的全部消息，必要时还能重置消费进度，以消费历史消息。

多个订阅组可以同时消费相同主题的全部消息，所以需要区别各个消费者的身份，分别记录消费进度。

# 领域模型概念

RocketMQ 基于发布订阅模型，它的消息的生命周期可以分为生产、存储、消费三个部分。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/rmq-0dfe6afa.png)

## 主题 Topic

**模型定义**

主题是 RocketMQ 消息的顶级容器，通过 TopicName 在集群内唯一标识和区分。主题的主要作用有两个：

* 消息分类：根据业务把消息划分到不同的主题，以此实现消息的存储隔离和订阅隔离。
* 批量管理：根据主题，对一类消息进行统一管理，比如身份识别、权限设置。


**自动创建**

主题的创建和管理需要消耗资源，RocketMQ 默认允许自动创建主题，但在生产环境应该禁用。

**版本差异**

每个主题应该只存储一种类型的消息。在 5.x 版本，服务端会校验接收的消息的类型；但 4.x 版本不支持消息类型检验，需要生产者端自己保证类型一致。

## 队列 MessageQueue

**模型定义**

队列是 RocketMQ 消息的实际容器，通过 QueueId 在主题内唯一标识和区分。可以这么理解：队列是主题的具体实现，同理 Kafka 使用分区实现主题。队列的主要作用有两个：

* 顺序存储：队列内部，消息按照接收的顺序进行存储，使用消息位点 offset 标记位置。
* 流式操作：根据位点，可以从队列的任意位置读取若干条消息。

**读写权限**

队列可以设置读写权限：6 默认，可以读写；4 只读；2 只写；0 不可读写（两位二进制数）

使用 Dashboard 创建主题时，底部 perm 选项用于设置这个主题的所有队列的读写权限。

**使用建议**

在 4.x 版本的集群消费模式，消息以队列位粒度分配给消费者，所以增加队列数量可以拓展主题的消费能力。队列数量可在定义或修改主题时设置，应遵循少用够用原则，如果队列太多，不但增加服务端的管理负担，还会给客户端增加压力，因为客户端是轮询访问队列，无效访问太多只会徒增消耗。

## 消息 Message

**模型定义**

消息是 RocketMQ 最小的数据传输单元，每条消息必定属于某个主题。消息不可变，其一旦被服务端接收，就不会变化。RocketMQ 会持久化消息。

**内部属性**

| 属性              | 必要 | 说明                                                       |
| ----------------- | ---- | ---------------------------------------------------------- |
| Topic             | 必须 | 所属主题的名字                                             |
| Body              | 必须 | 负载，二进制存储，存放业务数据。                           |
| Tag               | 可选 | 标签，主题之下的二级分类，用于订阅筛选。                   |
| Key               | 可选 | MQ 基于 Key 建立哈希索引，方便快速查询，也能用于订阅筛选。 |
| Properties        | 可选 | 属性，key-value 格式，存放扩展信息。                       |
| DelayTimeLevel    | 可选 | 延迟级别，消息到达服务端指定时长之后才能被消费。           |
| DeliveryTimestamp | 可选 | 毫秒级时间戳，5.x 新增，补充延迟级别，表示消息的生效时间。 |

## 生产者 Producer

**模型定义**

生产者是创建并发送消息到服务端的运行实体，通常集成在业务系统。生产者能向多个主题发送消息，单个主题也能接收多个生产者的消息。

生产者客户端可定义以下传输行为：

* 发送方式：同步，或异步。
* 批量发送：消息条数，消息大小。
* 事务行为：事务消息需要生产者配合进行事务检查等行为，以保证最终一致性。

**内部属性**

| 属性         | 必要 | 说明                                                     |
| ------------ | ---- | -------------------------------------------------------- |
| 客户端 ID    | 必须 | 客户端标识，客户端 SDK 自动生成，不可修改。              |
| 接入点信息   | 必须 | 服务端连接地址                                           |
| 身份认证信息 | 可选 | 身份验证的凭证信息，仅在服务端开启身份识别和认证时需要。 |
| 请求超时时间 | 可选 | 网络请求的超时时间                                       |
| 发送重试策略 | 可选 | 消息发送失败时的重试策略                                 |

**生产者组**

生产者使用生产者组进行身份标识，从 5.x 版本开始，生产者匿名，不再使用生产者组。

## 消费者组 ConsumerGroup

**模型定义**

消费者组是订阅组的实现，它由多个消费行为一致的消费者组成，支持负载均衡。消费者组是一个逻辑资源，不是运行实体，基于消费者组，可以实现消费性能的水平扩展和高可用。

**内部属性**

| 属性         | 说明                                           |
| ------------ | ---------------------------------------------- |
| 消费者组名字 | 集群内的唯一标识                               |
| 投递方式     | 拉取消息的方式：客户端拉取、服务端推送         |
| 消费重试策略 | 消费失败时的重试策略，仅对 PushConsumer 有效。 |
| 订阅关系     | 消费规则                                       |

**自动创建**

RocketMQ 默认允许自动创建消费者组，与主题相同，为了避免浪费资源，应在生产环境禁用。

**消费一致**

同组消费者的消费行为应该一致，在 5.x 版本，消费者会从服务端请求相关信息，保证消费行为一致。但 4.x 版本并不支持，需要消费者端自己保证行为一致。

## 消费者 Producer

**模型定义**

消费者是从服务端拉取消息并进行处理的运行实体，通常集成在业务调用链的下游系统。

消费者客户端可定义以下传输行为：

- 消费者身份：关联某个消费者组。
- 消费者类型：PushConsumer、PullConsumer 和 SimpleConsumer（5.x 新增）。
- 本地运行配置：比如消费并发度、客户端线程等。

**内部属性**

| 属性         | 必要 | 说明                                                     |
| ------------ | ---- | -------------------------------------------------------- |
| 客户端 ID    | 必须 | 客户端标识，客户端 SDK 自动生成，不可修改。              |
| 消费者组名字 | 必须 | 标识属于哪个消费者组                                     |
| 接入点信息   | 必须 | 服务端连接地址                                           |
| 身份认证信息 | 可选 | 身份验证的凭证信息，仅在服务端开启身份识别和认证时需要。 |
| 请求超时时间 | 可选 | 网络请求的超时时间                                       |
| 消费监听器   | 可选 | 监听服务端推送的消息并进行处理，仅 PushConsumer 需要。   |

**行为一致**

RocketMQ 要求同组消费者以下行为必须一致：投递顺序，消费重试策略，保证消息在组内的正常负载和消费。

## 订阅关系 Subscription

**模型定义**

订阅关系是消费者拉取消息的规则，由消费者组动态注册到服务端，并在后续的消息传输中按配置的过滤规则进行消息匹配和消费进度维护。RocketMQ 会持久化订阅关系。

订阅关系可定义以下传输行为：

- 过滤规则：筛选主题的哪些消息可以被消费者组消费，默认允许全部消息。
- 消费状态：消费者组在指定主题的消费进度。

**内部属性**

过滤类型，有 Tag 和 SQL 两种可选；过滤表达式。

**订阅一致**

同组消费者的消费逻辑应该一致，比如过滤配置。

# 生产者实践

虽然本笔记主要参考 RocketMQ 5.x 版本文档，但是使用 4.9.4 版本进行实验。

## 客户端依赖

引入客户端依赖，生产者和消费者都使用这个依赖，应与 RocketMQ 服务端版本一致。

```
<dependency>
  <groupId>org.apache.rocketmq</groupId>
  <artifactId>rocketmq-client</artifactId>
  <version>4.9.4</version>
</dependency>
```

## 客户端实现

DefaultMQProducer 类表示生产者，最常用，它能发送绝大部分类型的消息，除了事务消息。

```
DefaultMQProducer producer = new DefaultMQProducer();
producer.setProducerGroup("producerGroup"); // 4.x版本需要设置生产者分组
producer.setNamesrvAddr("127.0.0.1:9876"); // NameSrv接入点信息，多个地址使用分号分隔
producer.start(); // 启动，然后就能使用该对象发送消息
```

这里仅有最基本的配置，还可以在启动前配置其它发送行为，比如请求超时时间、发送失败策略。

```
producer.setDefaultTopicQueueNums(1); // 新建主题的队列数量，默认4
producer.setSendMsgTimeout(1000); // 请求超时时间，单位毫秒
```

## 消息 Message

Message 类表示消息，它必须关联某个主题，数据需要转为二进制格式。

```
String msg = ...;
Message message = new Message("topic", msg.getBytes());
```

Tag 属性，用于过滤。

```
message.setTags("tag");
```

Keys 属性，服务端会为其建立哈希索引，多个 key 使用空格分隔，可用容器类批量设置属性。

```
message.setKeys("key1 key2");
```

Properties 属性，携带拓展信息。

```
message.putUserProperty("key","value");
```

## 普通消息

普通消息没有任何特性，最为简单常用。RocketMQ 生产者支持三种发送消息的方式：同步、异步和单向，前两种方式可靠，因为服务端会返回结果告知是否发送成功。

### 同步发送

生产者发出消息后，将一直阻塞，直到收到返回结果，然后才能继续发送下一条消息。

只有一个 Message 参数的 send 方法，使用同步发送，返回类型 SendResult 表示服务端响应结果。

```
SendResult sendResult = producer.send(message);
```

发送失败会抛出异常，必须捕获它，并设计兜底方案，忽略异常可能导致业务错误。

### 异步发送

生产者发出消息后，不需等待，可以继续发送下一条消息，需要注册回调函数用于接收和处理响应结果。

```
producer.send(message, new SendCallback() { // 第二个参数是异步回调接口
	
	@Override
	public void onSuccess(SendResult sendResult) {...} // 发送成功时执行

    @Override
    public void onException(Throwable throwable) {...} // 发送失败时执行
});
```

### 单向发送

生产者直接发送消息，不等待也不接收响应结果，即不管是否发送成功，效率最高，但不可靠。

```
producer.sendOneway(message);
```

### 批量发送

前面几种方式都是发送单条消息，每次发送都进行一次网络通信，消息多时这种方式很不划算。RocketMQ 支持批量传输，即单次通信发送多条消息，这些消息必须属于同一个主题，且不支持延迟消息和事务消息。

```
List<Message> messages = new ArrayList<>();
messages.add(new Message(...));
messages.add(new Message(...));
messages.add(new Message(...));
producer.send(messages); // 同步发送批量消息
```

## 顺序消息

顺序消息是生产顺序和消费顺序一致的消息，消息队列天然具有顺序性，服务端按接收顺序把消息存到队列。

### 生产顺序

消息的顺序性分为两部分：生产顺序和消费顺序，这里讨论生产顺序，首先需要满足以下条件：

* 单生产者：如果有多个生产者同时向一个主题发送消息，无法判断这些消息的先后顺序。
* 串行发送：生产者客户端支持多线程，如果有多个线程同时发送消息，无法判断这些消息的先后顺序。

满足以上条件，可以保证，在一个主题的任何队列，所有消息都是按照发送顺序存储。

### 队列选择

主题包含多个队列，普通 send 方法轮询地把消息发送到各个队列，实现存储均衡。我们可以使用选择器，自定义目标队列的选择策略。

接口  MessageQueueSelector  表示选择器，只有一个方法 select。第一个参数表示主题的所有队列，第二个参数是要发送的消息，第三个参数是任意对象，返回对象就是消息将要发送的目标队列。因此，我们可以根据消息本身和额外提供的对象，制定队列的选择策略。

```
MessageQueueSelector selector = new MessageQueueSelector() {
    @Override
    public MessageQueue select(
            List<MessageQueue> list, Message message, Object arg) {
        ...
    }
};
```

使用带有选择器参数的 send 方法发送消息

```
SendResult sendResult = producer.send(message, selector, message);
```

### 全局有序

如果主题只有一个队列，那么该主题全局有序。而且，也没有必要使用选择器，反正只有一个队列，只需要保证消息的生产顺序。

### 版本差异

顺序消息和普通消息好像没有什么区别，它完全是依靠队列特性来保证顺序。这样的实现比较简单，但会限制所有消息遵循同一种排序规则。

在  5.x  版本，消息  Message 新增属性 MessageGroup 消息组，同组消息会存到同一个队列，并在组内按照顺序存储，消费者支持按组消费。每个主题可以存储多组消息，不同分组能使用不同的排序规则。

## 延迟消息

服务端接收到延迟消息之后，需要等待一段时间，这个消息才能被投递消费。

Broker 收到延迟消息，首先把它放到名为 SCHEDULE_TOPIC_XXX 的主题，服务端设有定时任务，每秒执行一次检查，如果发现某个延迟消息到期，就把它转移到目标主题，然后这个消息就能被正常消费。

RocketMQ 使用 18 个等级表示消息的延迟程度，如下所示。

| 延迟等级 | 延迟时间 | 延迟等级 | 延迟时间 |
| -------- | -------- | -------- | -------- |
| 1        | 1 s      | 10       | 6 min    |
| 2        | 5 s      | 11       | 7 min    |
| 3        | 10 s     | 12       | 8 min    |
| 4        | 30 s     | 13       | 9 min    |
| 5        | 1 min    | 14       | 10 min   |
| 6        | 2 min    | 15       | 20 min   |
| 7        | 3 min    | 16       | 30 min   |
| 8        | 4 min    | 17       | 1 h      |
| 9        | 5 min    | 18       | 2 h      |

延迟等级是 Message 的属性，至于它的发送和消费，与普通消息完全一样。

```
message.setDelayTimeLevel(1); // 设置延迟等级
SendResult sendResult = producer.send(message);
```

> 在 5.x 版本，支持使用秒级时间戳来指定消息的生效时间，即定时消息，更加灵活。

## 消息发送重试

RocketMQ 生产者端具有消息发送重试机制，消息发送失败时，生产者将重新尝试发送。

单向消息不支持发送重试，毕竟生产者都不知道发送失败与否，顺序消息也不支持发送重试。

**最大重试次数**

消费者不能无限重试，默认最多两次，当超过最大重试次数，抛出异常，客户端需要进行兜底处理。

DefaultMQProducer 支持修改最大重试次数。

**同步发送失败**

普通消息默认轮询发送到不同队列，如果发送失败，重试时会尽量选择其它 Broker，如果只有一个 Broker，则会尽量选择其它队列。

```
producer.setRetryTimesWhenSendFailed(3); // 同步发送失败重试次数
```

**异步发送失败**

异步发送消息失败，生产者不会尝试其它 Broker，它将一直对一个 Broker 重试。

```
producer.setRetryTimesWhenSendAsyncFailed(3); // 异步发送失败重试次数
```

**消息刷盘失败**

消息刷盘失败，或 slave 不可用，生产者默认不尝试其它 Broker，可修改配置属性开启对其它 Broker 重试。

```
retryAnotherBrokerWhenNotStoreOK = true
```

**发送重试导致消息重复**

发送重试能保证消息的成功发送、不丢失，但也可能导致消息重复。如果消息已经被服务端接收，但返回结果没有被生产者收到，生产者会误以为发送失败，进行重试，造成消息重复。

消息重复是消息队列最基本、最常见的问题，必须制定兜底方案。

# 消费者实践

RocketMQ 有多种类型的消费者：PushConsumer、PullConsumer 和 SimpleConsumer（5.x 新增）。

## Push 消费

服务端主动推送消息给消费者，这种类型消费者能及时消费消息，但如果没有做好流量控制，容易造成客户端消息堆积甚至崩溃。

### 客户端实现

DefaultMQPushConsumer 类是 PushConsumer 实现，以下是它的基本配置。

```
consumer = new DefaultMQPushConsumer();
consumer.setConsumerGroup("consumerGroup"); // 消费者组名
consumer.setNamesrvAddr("127.0.0.1:9876"); // NameSrv接入点信息，多个地址使用分号分隔
consumer.subscribe("topic", "*"); // 订阅主题，过滤表达式
consumer.start(); // 启动，消费者持续运行消费消息
```

DefaultMQPushConsumer 需要配置监视器，负责接收消息并进行处理，两种监视器可选：

* MessageListenerConcurrently，并发消费
* MessageListenerOrderly，顺序消费

### 并发消费

注册 MessageListenerConcurrently 监视器，消费者可能会使用多个线程同时消费一个队列，这个接口只有一个方法 consumeMessage，第一个参数是消息集合，第二个参数是消费环境，返回 ConsumeConcurrentlyStatus 枚举类型表示消费状态，其中，CONSUME_SUCCESS 表示消费成功，RECONSUME_LATER 表示消费失败。

```
consumer.registerMessageListener(new MessageListenerConcurrently() {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> list, 
    		ConsumeConcurrentlyContext consumeConcurrentlyContext) {
        ...
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});
```

消费者实际会使用多少个线程来消费消息？这不固定，但可以指定线程数量的范围。

```
consumer.setConsumeThreadMin(1); // 最小线程数
consumer.setConsumeThreadMax(5); // 最大线程数
```

### 顺序消费

注册 MessageListenerConcurrently 监视器，消费者将会按照消息的存储顺序进行消费，这个接口只有一个方法 consumeMessage，参数与并发消费监视器相同，返回 ConsumeOrderlyStatus 枚举类型，SUCCESS 表示消费成功，SUSPEND_CURRENT_QUEUE_A_MOMENT 表示消费失败。

```
consumer.registerMessageListener(new MessageListenerOrderly() {
    @Override
    public ConsumeOrderlyStatus consumeMessage(List<MessageExt> list, 
    		ConsumeOrderlyContext consumeOrderlyContext) {
        ...
        return ConsumeOrderlyStatus.SUCCESS;
    }
});
```

### 消费模式

DefaultMQPushConsumer 支持两种消费模式：

* 集群模式：主题的每个消息，只能被同组的一个消费者消费一次。
* 传播模式：主题的所有消息，都会被所有消费者消费一次。

集群模式，主题的消息以队列为粒度分配给消费者组的消费者，修改消费者数量，可以影响消费性能。

```
consumer.setMessageModel(MessageModel.CLUSTERING); // 集群模式
consumer.setMessageModel(MessageModel.BROADCASTING); // 传播模式
```

### 负载均衡

#### 负载均衡策略

前面提到，集群模式会把消息按粒度分配给组内的消费者，可选以下分配策略：

* 平均分配：如果不能完全平分，多余队列会按顺序分给消费者；
* 环形平均：把所有消费者看作环形链表，挨个分配队列；
* 机房优先：选出与消费者同机房的队列，然后平均或环形平均分配，没有则平均或环形平均所有队列；
* 哈希：把消费者和队列存入哈希环，按顺时针方向，把队列分配给距离最近的消费者，虽然不均匀，但能减少因消费者数量变化而 Rebalance 时需要变更的关系。

DefaultMQPushConsumer 支持设置负载均衡策略，默认平均分配，如果消费者比队列多，就会有消费者空闲。

```
DefaultMQPushConsumer::setAllocateMessageQueueStrategy(AllocateMessageQueueStrategy);
```

AllocateMessageQueueStrategy 是负载均衡策略接口，它有一个方法 allocate，返回队列集合，这就是要分配给消费者的队列，可以通过实现这个接口自定义负载均衡策略。

> 在 5.x 版本，新增消息粒度的负载均衡功能，同一个队列的消息可以被多个消费者分摊消费。

#### 再平衡 Rebalance

集群模式，如果订阅主题的队列数量或消费者组的成员数量发生变化，那么之前负载均衡的结果就不再适用，需要进行 Rebalance，重新为消费者分配队列。

**作用**

显然，Rebalance 机制能及时地调整分配关系，充分发挥所有消费者的消费性能。

**危害**

* 消费暂停

  Rebalance 过程会暂停所有队列的消费

* 重复消费

  成功消费的消息还未来得及提交，消费者暂停进行 Rebalance，这个消息会被重复消费。

* 消费突刺

  如果重复消费过多，或者暂停时间过长导致消息积压，Rebalance 完后将面对大量消息待处理。

### 批量拉取

其实不论 PushConsumer 或 PullConsumer，每次拉取消息都是批量进行，只是默认每次只拉取一条消息。

DefaultMQPushConsumer 可以修改每次推送消息的数量

```
consumer.setPullBatchSize(10);
```

批量推送的消息数量最大是 32，可以修改这个限制。

```
consumer.setConsumeMessageBatchMaxSize(35);
```

## Pull 消费

客户端主动从服务端拉取消息，比 Push 自主性高，但会无法使用 RocketMQ 提供的一些服务，性能也较低。

Pull 模式有两种实现：Pull Consumer 和 Lite Pull Consumer。

### Pull Consumer

Pull Consumer 使用起来比较麻烦，需要手动指定目标队列的主题、所在 Broker，以及 Queue ID。

DefaultMQPullConsumer 类是 Pull Consumer 实现，以下是它的基本配置。

```
DefaultMQPullConsumer consumer = new DefaultMQPullConsumer();
consumer.setConsumerGroup("consumerGroup"); // 消费者组名
consumer.setNamesrvAddr("127.0.0.1:9876"); // NameSrv接入点信息
consumer.start(); // 启动，然后就能拉取消息消费
```

MessageQueue  类表示队列，DefaultMQPullConsumer 拉取消息之前需要使用 MessageQueue 对象指定目标队列。使用 pull 方法拉取消息，有 4 个参数：目标队列、过滤表达式、初始消费位点、最大拉取消息数量，返回类型 PullResult 表示拉取结果，它的 pullStatus 属性表示拉取状态，消费完成之后还需手动更新消费位点。

```
MessageQueue queue = new MessageQueue(); // 目标队列
queue.setQueueId(0); // Queue ID
queue.setTopic("topic"); // 队列所属主题名字
queue.setBrokerName("centos-7-01"); // 队列所在Broker名字
PullResult pullResult = consumer.pull(queue, "*", 0, 32); // 拉取目标队列消息
if (pullResult.getPullStatus().equals(PullStatus.FOUND)) { // 如果拉取成功
    List<MessageExt> messageExtList = pullResult.getMsgFoundList(); // 获取消息列表
    ...
    consumer.updateConsumeOffset(queue,pullResult. getNextBeginOffset()); // 更新消费位点
}
```

### Lite Pull Consumer

DefaultLitePullConsumer 类是 Lite Pull Consumer 实现，以下是它的基本配置，这种方式使用比较简便。

```
DefaultLitePullConsumer litePullConsumer = new DefaultLitePullConsumer();
litePullConsumer.setConsumerGroup("groupName");
litePullConsumer.setNamesrvAddr("127.0.0.1:9876");
litePullConsumer.start();
```

DefaultLitePullConsumer 有 Subscribe 和 Assign 两种使用方式。

**订阅 Subscribe**

执行 subscribe 方法设置订阅关系，第一个参数是订阅主题的名字，第二个参数是过滤表达式。

```
litePullConsumer.subscribe("topic", "*");
litePullConsumer.start();
```

RocketMQ 会根据负载均衡为消费者分配队列，使用 poll 方法拉取消息，默认自动提交消费位点。

```
List<MessageExt> messageExts = litePullConsumer.poll();
```

DefaultLitePullConsumer 默认每次只拉取一条消息，可以修改每次拉取消息的数量。

```
litePullConsumer.setPullBatchSize(10);
```

**分配 Assign**

Assign 方式与 Pull Consumer 类型类似，操作比较复杂，首先关闭自动提交消费位点。

```
litePullConsumer.setAutoCommit(false);
```

这种方式在消费完后同样需要手动提交消费位点

```
Collection<MessageQueue> messageQueues // 获取订阅主题所有队列
		= litePullConsumer.fetchMessageQueues("topic");

List<MessageQueue> list = new ArrayList<>(messageQueues); // 目标队列集合
List<MessageQueue> assignList = new ArrayList<>();
for (int i = 0; i < list.size() / 2; i++) {
	assignList.add(list.get(i));
}

litePullConsumer.assign(assignList); // 为消费者分配队列
litePullConsumer.seek(assignList.get(0), 10); // 设置初始消费位点

 while (true) {
 	List<MessageExt> messageExts = litePullConsumer.poll(); // 拉取消息
 	litePullConsumer.commitSync(); // 提交消费位点
 }
```

## 订阅消息过滤

RocketMQ 消息支持条件过滤，限制订阅主题的某些消息不能被消费，默认消费所有消息。

RocketMQ 支持两种过滤方式：Tag 过滤、 SQL 过滤。

### Tag 过滤

生产者端为消息设置 Tag 标签，每条消息只能设置一个 Tag 属性。

```
Message message = new ...
message.setTag("tag");
```

限制只有 Tag 属性与指定正则表达式匹配的消息才能被消费

```
consumer.subscribe("topic", "*"); // 所有消息
consumer.subscribe("topic", "tagA"); // tagA
consumer.subscribe("topic", "tagA||tagB"); // tagA或tagB
```

### SQL 过滤

这种方式使用 SQL 语句，针对消息 Properties 属性进行筛选，它的语义更灵活，筛选范围更大。其实，Tag 本质也是 Properties 属性，但它的键是保留字 "TAGS"。

生产者端为消息设置 Properties 属性，每条消息可以设置多个 Properties 属性。

```
message.putUserProperty("key", "value");
```

限制只有 Properties 满足 SQL 语句要求的消息才能被消费

```
consumer.subscribe("topic", 
	MessageSelector.bySql("TAGS in ('TagA', 'TagB')) and (k between 0 and 3)"));
```

SQL 语法不必多讲，但是，有些关键字在这有不同的含义，比如 =、!=、between、in 等运算符，比较的是值，而 is null、is not null 等运算符，比较的是属性的键。

## 消费进度管理

### 消息位点

前面提到，主题的队列根据到达服务端的顺序存储消息，每条消息在队列上都有一个唯一 Long 类型标记，这就是消息位点 MessageQueueOffset。

队列最早那条消息的位点，称为最小消息位点 MinOffse，最新消息的位点，称为最大消息位点 MaxOffset。消息位点可从 0  到 Long.MAX 无限增加，所以逻辑上队列能无限存储。但是，磁盘空间有限，RocketMQ 会滚动删除队列中最早的消息，因此，队列的 MinOffse 和 MaxOffset 会一直递增变化。

通过"主题+队列+位点"，可以精确定位消息队列的任何一条消息。

### 消费位点

RocketMQ 基于发布订阅模型，消息即使被某个消费者成功消费，也不会立即被删除，这条消息还有可能被其它消费者（组）消费。RocketMQ 队列会记录各个消费者组的消费进度，这就是消费位点 ConsumerOffset。

集群模式，消费位点由客户端提交给服务端保存，客户端重启依然能继续之前的进度消费，如果服务端保存的消费位点过期或被删除，就把消费位点前移至最小消费位点。广播模式，消费位点由客户端自己保存。

### 重置消费位点

**初始消费位点**

消费者组初次启动消费者进行消费时，它的消费位点称为初始消费位点，表示从哪开始消费，RocketMQ 规定初始消费位点为最小消息位点。

**重置消费位点**

如果初始消费位点或当前消费位点不满足需求，可以重置消费位点来调整消费进度。

DefaultMQPushConsumer 的 setConsumeFromWhere 方法用于重置消费位点，它的参数有以下可选值：

* ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET：最小消息位点

* ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET：最大消息位点

* ConsumeFromWhere.CONSUME_FROM_TIMESTAMP：指定时间之后的消息开始，Properties 设置时间。

  ```
  consumer.setConsumeTimestamp(“20210701080000”) # yyyyMMddHHmmss
  ```

这个方法存在较多限制。首先，必须是首次订阅，如果主题保存着该消费者组的消费进度，将直接从历史消费位点继续。然后，主题应该存在较长时间，如果最早的消息都没有过期，服务端会认为这是一个新上线的业务，强制从最小消息位点开始消费。

## 消息消费重试

### 消费重试

消息如果消费失败，或处理失败，或在 PushConsumer 排队超时，将触发消费重试。此时，服务端根据策略重新投递消息，如果达到最大重试次数仍未消费成功，消息会被投递到死信队列。

消费重试只适用于 Push 类型消费者，且只在集群模式有效，如果传播模式发生消费失败，直接跳过这条消息。

**重试次数**

DefaultMQPushConsumer 可以修改最大消费重试次数，默认 16 次。

```
consumer.setMaxReconsumeTimes(10);
```

**普通消息**

普通消息消费失败，重试的时间间隔逐渐递增：10 s、30 s、1 min、2 min、3 min、...

**顺序消息**

顺序消息消费失败，为了保证消费顺序，消费者会反复重试这条消息，直至消费成功或者超过最大次数，重试期间消费者阻塞，不会消费其它消息，重试的时间间隔默认 1000 毫秒。

DefaultMQPushConsumer 可以修改顺序消息的重试时间间隔，区间 [10,30000]，单位毫秒。

```
consumer.setSuspendCurrentQueueTimeMillis(5000);
```

### 死信队列

如果超过最大重试次数仍未消费成功，消息会被投递到该消费者组对应的死信队列，这条消息也变为死信消息。

每个消费者组都有一个对应的死信主题，名为 %DLQ%ConsumerGroupName，死信主题只有一个死信队列。

消费者组默认订阅死信主题，但是死信队列直到有死信消息时才会创建。死信消息的有效期与普通消息相同，只是它不能被消费，可以使用 RocketMQ Admin 或 RocketMQ Dashboard 查看死信消息。

# NameServer

## 基本介绍

Namesrv 在 RocketMQ 系统中扮演协调者的角色，它的主要功能是临时保存和管理 Topic 路由信息。Topic 包含多个队列，这些队列可能分布在多个 Broker 节点，Topic 路由信息就是指各个队列的位置

NamaSrv 没有状态，集群中各个 Namesrv 节点相互独立，互不影响。这样的话，搭建一个 Namesrv 集群就变得非常简单：首先启动所有 NamaSrv 服务，然后在所有 Broker 节点注册这些 NamaSrv 地址。

Namesrv 运行开销非常小，它主要的活动是维护与客户端的连接，以及动态更新 Topic 路由信息。

## 路由注册

Broker 具有定时心跳机制，服务启动就开始心跳，每次心跳 Broker 都会遍历配置的所有 Namesrv 节点，向它们发送自己的 Topic 路由信息以及其它信息。默认心跳间隔 30 秒。

Broker 第一次心跳时向 NameSrv 进行路由注册，以后的心跳则是为了维护这些信息。

## 路由剔除

Broker 路由注册之后，如果 Namesrv 长时间没有收到它的心跳，或收到它的关闭信息，NameSrv 将会删除这个节点的所有信息，把它从集群剔除出去。

客户端通过与 NameSrv 的长连接，动态获取最新 Topic 路由信息，如果某个 Broker 被剔除或添加，客户端都能及时知道。因此，Broker 的注册、剔除对客户端来说都是透明。

## 客户端连接

Broker 节点数量可能在运行时动态变化，但这不会影响客户端连接，因为客户端是从 Namesrv 获取 Broker 路由信息，且会不断从 Namesrv 获取更新，而 Namesrv 又会通过心跳机制保证路由有效。

因此，RocketMQ 客户端配置的服务地址是指 Namesrv 集群，而非具体某个 Broker。

假如客户端配有多个 Namesrv 节点，请求路由信息时，客户端会生成一个随机数，它把把该数与 Namesrv 数量求模，结果就是此次要访问的 Namesrv 的序号，然后尝试与其连接，如果失败，则轮询其它节点。

## 启动和关闭

RocketMQ 安装目录下的 *bin* 子目录中，*mqnamesrv* 脚本与 NameSrv 服务相关，使用格式如下：

```
mqnamesrv [-c <configFilePath>] [-h] [-p] [key=value key=value [,key=value...]]
```

-c 指定配置文件路径；-h 获取帮助信息；-p 查看所有可配置项。

RocketMQ 支持在配置文件或命令行设置配置属性，格式 "key=value"，命令行属性优先级更高。

使用 *mqshutdown* 脚本启动服务，指定端口 9876，这是默认端口。

```
nohup sh ./bin/mqnamesrv listenPort=9876 &
```

查看日志，检查启动是否成功。

```
tail -f ~/logs/rocketmqlogs/namesrv.log
```

使用 *mqshutdown* 脚本关闭服务

```
sh ./bin/mqshutdown namesrv
```

# Broker

Broker 是 RocketMQ 系统中真正提供消息队列服务的模块，它的功能主要是存储消息和处理各种请求。

## 存储机制

### 存储目录

RocketMQ 具有持久化功能，安装启动之后会生成许多文件，这些文件默认放在用户目录 *store* 子目录，目录结构如下所示。

| 文件/目录    | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| commitlog    | 消息文件目录，文件名由所存消息的最大位点高位补零构成，默认文件大小为 1GB |
| consumequeue | 消费队列文件目录，文件名格式：*./consumequeue/Topic name/queue id/具体文件*。消费队列本质是 commitlog 索引，提供给消费者用于拉取消息、更新位点。 |
| index        | 哈希索引文件目录，索引根据消息 key 属性创建，文件名是时间戳  |
| config       | 配置文件目录，存储 Broker 所有 Topic、订阅关系、消息进度，用于数据恢复 |
| abort        | 标志文件，出现表示上次异常关闭，Broker 会进行某些操作，比如重建 index 文件 |
| checkpoint   | 状态文件，记录 Broker 上次正常关闭时的状态，比如最近一次刷盘时间 |
| lock         | 资源文件，Broker 运行期间使用的全局锁                        |

以上数据目录的存放位置，可以通过以下配置属性修改。

| 属性                  | 默认                   | 说明                  |
| --------------------- | ---------------------- | --------------------- |
| storePathRootDir      | *~/store*              | store 目录位置        |
| storePathCommitLog    | *~/store/commitlog*    | commitlog 目录位置    |
| storePathConsumeQueue | *~/store/consumequeue* | consumequeue 目录位置 |
| storePathIndex        | *~/store/index*        | index 目录位置        |
| storeCheckpoint       | *~/store/checkpoint*   | checkpoint 文件位置   |
| abortFile             | *~/store/abort*        | abort 文件位置        |

### 消息文件

Broker 把 RocketMQ 消息存到 commitlog 目录，目录包含若干个文件，这些文件通过其保存的消息的物理位点首位相连。默认 commitlog  文件大小为 1GB，这个目录初始只有一个文件，当一个文件写满，Broker 创建一个新文件继上一个文件 offset 写入消息。

配置属性 mapedFileSizeCommitLog 指定 commitlog 文件大小

```
mapedFileSizeCommitLog = 1073741824
```

### 优化技术

RocketMQ 写文件非常快，这里说一下它使用到的优化技术。

**缓存页 Page Cache**

现代操作系统内核都被设计为以 Page 为粒度读写文件，每次从文件读或向文件写最少都是整页，每个 Page 默认大小 4KB。当 RocketMQ 请求一段文件内容，系统会额外加载一些页到缓冲，这就是预读，注意，这是操作系统提供的功能，下次的请求如果命中缓存就会直接返回，而不用 IO 磁盘。

> MySQL 的 InnoDB 存储引擎也是以页为单位读写数据，但那是在程序层面实现。

当然，Page Cache 也有缺点，比如脏页回写、内存回收等造成的读写延迟，RocketMQ 为此做了许多优化，比如内存预分配、文件预热等。

**虚拟内存 Virtual Memory**

所谓虚拟内存，其实就是磁盘。操作系统从磁盘划分一部分空间，作为交换区，当内存不足时，操作系统就把一些暂时不用的内存数据转移到交换区，从而空出一些空间用于程序运行。

这时，操作系统可分配内存=物理内存+虚拟内存，虚拟内存使系统能够运行占用内存空间更大的程序。

**零拷贝技术**

Java 原始读取文件的流程：磁盘 -> 内核态内存 -> 用户态内存 -> Java 虚拟机，文件内容被操作系统读取后，还需要经历两次拷贝，才能被 Java 进程使用。

为了提高读写效率，RocketMQ 读写文件过程大量使用零拷贝技术，java.nio.MappedByteBuffer 类实现了这种技术，它能与内核态内存直接交互，省去两次拷贝过程。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/rmq-86d6fe6f.png)

### 高效读写

RocketMQ 读写消息，其实就是读写 commitlog 文件，前面提到 RocketMQ 大量使用零拷贝技术，它会按照下图结构组织和映射消息数据。

CommitLog 类表示所有 commitlog 文件，MappedFileQueue 包含所有映射文件，文件按过期时间排序，新消息总是写到最后一个文件。MappedFile 表示 commitlog 文件，它会使用 MappedByteBuffer 类的零拷贝技术进行内存映射，同时拥有内存的写入速度和磁盘一样可靠的持久化方式。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/rmq-42384640.png)

写消息时，需加全局锁，因此任何时刻最多只有一个 commitlog 文件被写入。然后，RocketMQ 用 Append 方式写入最新 MappedFile。对于读消息，大部分时候消费者只关注最新消息，而这些消息早已被预读，即便读取历史消息，也仅在第一次读时操作磁盘，后续就能利用磁盘预读，几乎做到不直接读磁盘。

## 消息清理

磁盘空间有限，Broker 不会一直保存所有消息，况且消息被消费之后就不再需要，RocketMQ 通过设置过期时间清理消息数据，它会每日定时检查 commitlog 文件，删除过期文件，同时删除关联数据，比如 index 文件。

配置属性 fileReservedTime 指定 commitlog 文件的保留时间，默认 48 小时。

```
fileReservedTime = 48
```

配置属性 deleteWhen 指定删除时间点，默认凌晨 4 点，每日此时将检查并删除过期 commitlog 文件。

```
deleteWhen = 04
```

## 刷盘机制

Broker 收到消息同时会将其存入 Page Cache（零拷贝技术），然后需要进行持久化，两种刷盘方式可选：

* 同步，Broker 调用同步方法，把包含新消息的 Page Cache 写入磁盘，结束之后才返回客户端；

* 异步，消息一旦被 Broker 接收就返回客户端，刷盘交给异步服务完成。

配置属性 flushDiskType 指定刷盘方式，有可选值：SYNC_FLUSH 同步、ASYNC_FLUSH 默认异步。

```
flushDiskType = ASYNC_FLUSH 
```

异步刷盘存在一种特殊情况，如果开启 DM 读写分离，那么消息首先会被写入 DM，然后返回客户端，异步转存服务负责把消息写到 Page Cache，至于刷盘，依然由异步刷盘服务完成。

异步刷盘有许多可配置属性，用于调控运行方式，下表列出其中一些。

| 属性                             | 默认  | 说明                               |
| -------------------------------- | ----- | ---------------------------------- |
| flushCommitLogTimed              | true  | true 定时刷盘，false 实时刷盘      |
| flushIntervalCommitLog           | 500   | 定时刷盘时间间隔，单位毫秒         |
| flushPhysicQueueThoroughInterval | 60000 | 两次实时刷盘最大时间间隔，单位毫秒 |
| flushCommitLogLeastPages         | 4     | 每次刷盘的页数                     |

## 读写分离

读写分离，可以有效分担请求，减小压力，增加吞吐量，RocketMQ 两个地方支持读写分离机制。

### 主从读写分离

对于主从复制集群，通常都由 master 处理所有读写请求，如果开启主从读写分离，当 master 负载过高，客户端就向 slave 读取消息。

slave 节点通过配置属性 slaveReadEnable 设置允许读消息

```
slaveReadEnable = true
```

启用读写分离之后，消费者开始依然是从 master 拉取消息，且每次拉取都会计算 diff，这是此次拉取消息的最大位点与 Broker 存储的消息的最大位点的差，如果这个值很大，说明 master 还有很多消息没有消费，就认为比较繁忙，master 返回结果时将提示客户端下次尝试从 slave 拉取消息。

配置属性 accessMessageInMemoryMaxRatio 指定最大可用内存，它是最大物理内存占比，默认 40。

```
accessMessageInMemoryMaxRatio = 40
```

### DM 读写分离

上一节提到：开启异步刷盘，Broker 收到消息时会把数据写到 Page Cache，然后直接返回，异步服务会把数据刷盘。但是，如果启用 DM 读写分离，DM 是 Direct Memory 直接内存，Broker 收到消息时不会再把数据写到 Page Cache，而是放在 DM 中并返回，有个异步转存服务负责把 DM 数据移到 Page Cache，最后，还是由异步刷盘服务进行持久化。

在这个过程中，消息写到 DM 就返回客户端发消息成功，当消息被异步服务转存到 Page Cache 后，客户端才能消费这个消息。所以，消息的读和写分开，这也算是读写分离。这样做的目的，我认为，因为 DM 是 Java 直接从操作系统申请的内存，写入 DM 比写入 Page Cache 快的多，所以这能提高写消息的速度，增加吞吐量。

> 不论何时，客户端总是从 Page Cache 读取消息。

配置属性 transientStroePoolEnable 开启 DM 读写分离，默认关闭。

```
transientStroePoolEnable = true
```

## 主从复制

**基本介绍**

RocketMQ 支持主从复制，这能提供高可用和数据安全，自然 Broker 有 master 和 slave 两种角色。

主节点负责处理各种请求，以及存储数据，从节点同步数据到本地，它的作用体现在两个方面：

* 读写分离，减轻主节点读的压力，提升服务性能；
* 高可用性，如果主节点失效，从节点依然可以提供消费服务。

**同步方式**

从节点需要从主节点处同步数据至本地，同步的数据可分为两类：

* 配置数据，包括主题信息、消费位点等，主节点启动时会开启定时任务，每隔 60s 进行一次同步；
* 消息数据，主要就是 commitlog 文件的内容，这也是我们关注的重点。

对于消息数据，RocketMQ 支持两种主从复制方式：

* 同步复制，消息被主节点写入 Page Cache 后，需要同步传输到从节点，然后才返回客户端；
* 异步复制，消息被主节点写入成功后，直接返回客户端，有一个异步服务负责同步数据到从节点。

配置属性 brokerRole 指定 Broker 主从角色，以及主从复制的方式，可选值如下：

* SYNC_MASTER：主节点，同步复制；
* ASYNC_MASTER：主节点，异步复制；
* SLAVE：从节点

**主从搭建**

主节点和从节点不直接连接，它们依靠 NameSrv 组合，因此主从节点至少配有一个相同 NameSrv 服务。

```
namesrvAddr = 127.0.0.101:9876;127.0.0.102:9876
```

NameSrv 通过配置属性 brokerName 区分 Broker 节点，主从集群应该给人感觉是一个节点，所以主从节点应该配置相同的名字。

```
brokerName = broker-a
```

主从复制内部，使用 brokerId 区分节点，并且，主节点 Id 必须是零，从节点大于零。

```
brokerId = 0
```

最后，通过 brokerRole 配置属性，为各个节点分配角色，以及设置同步复制方式。

```
brokerRole = SYNC_MASTER
```

## 集群部署

RocketMQ 支持集群模式，它把 Topic 队列平均分布在集群节点，如果某个节点失效，将进行 Rebalance，重新分配队列位置，失效节点的消息变得不可用，除非它配有 slave 节点继续提供消费服务。

搭建集群非常容易，首先，所有节点都应该连接同一个 NameSrv 服务。

```
namesrvAddr = 127.0.0.101:9876;127.0.0.102:9876
```

配置属性 brokerClusterName 标识节点属于哪一个集群，同一个集群的节点这个属性的值应该相同。

```
brokerClusterName = DefaultCluster
```

现在，启动 NameSrv 服务，然后启动所有 Broker 节点，集群搭建成功。

## 其它配置

| 属性                        | 默认           | 说明                                           |
| --------------------------- | -------------- | ---------------------------------------------- |
| brokerClusterName           | DefaultCluster | Broker 集群名字                                |
| defaultTopicQueueNums       | 4              | 新建主题队列数量                               |
| autoCreateTopicEnable       | true           | 是否允许自动创建主题                           |
| autoCreateSubscriptionGroup | true           | 是否允许自动创建订阅关系                       |
| brokerIP                    |                | 主机 IP 地址，服务能自动识别，但多网卡时不确定 |
| listenPort                  | 10911          | Broker 服务端口                                |
| haListenPort                | 10912          | 主节点监听从节点请求的端口                     |
| maxMessageSize              | 65536          | 消息的最大大小，单位 KB                        |
| mapedFileSizeConsumeQueue   | 300000         | 队列最大存储的消息条数                         |

## 启动和关闭

RocketMQ 安装目录下的 *bin* 子目录中，*mqbroker* 脚本与 Broker 服务相关，使用格式如下：

```
mqbroker [-c <arg>] [-h] [-m] [-n <arg>] [-p]
```

参数 -c 指定配置文件路径；-h 获取帮助信息；-p 查看所有可配置项；-m 查看重要的可配置项；-n 指定 NameSrv 服务的地址，多个地址使用分号分隔，例如 *192.168.0.1:9876;192.168.0.2:9876*。

同样，*mqbroker* 脚本也支持在配置文件或命令行设置配置属性，命令行方式优先级更高。

启动服务，指定向本机 NameSrv 服务注册路由。

```
nohup sh ./bin/mqbroker -n 127.0.0.1:9876 &
```

查看日志，检查启动是否成功。

```
tail -f ~/logs/rocketmqlogs/broker.log
```

关闭服务，这里使用 *mqshutdown* 脚本。

```
sh ./bin/mqshutdown broker
```

# JVM 配置

NameSrv 和 Broker 都运行在 Java 虚拟机，安装目录下的 *bin* 子目录中，可在 *runserver.sh* 和 *runbroker.sh* 脚本分别配置这两个服务的 JVM 启动参数。

对于个人主机，内存空间很小，必须对 JVM 各种内存使用进行限制，否则服务大概率无法成功启动。

```
# runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

```
# runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

# Dashboard

RocketMQ 官方提供可视化管理平台 Dashboard，通过它可以很方便地查看 RocketMQ 运行情况。

下载地址：https://github.com/apache/rocketmq-dashboard

# Docker 搭建

拉取镜像

```
docker pull rocketmqinc/rocketmq:4.3.2
dokker pull styletang/rocketmq-console-ng
```

`broker.conf` 配置

```
#集群名字
brokerClusterName = DefaultCluster
#broker名字
brokerName = broker-a
#0表示Master
brokerId = 0
#清除消息时间，凌晨4点
deleteWhen = 04
#消息保留时长，48小时
fileReservedTime = 48
#异步复制
brokerRole = ASYNC_MASTER
#异步刷盘
flushDiskType = ASYNC_FLUSH
#磁盘使用率达到95%之后,生产者写入消息会报错
disk full

brokerIP1 = brokerIP
namesrvAddr = nameSrvIP:Port
```

创建 NameServer 容器

```
docker run -d --name namesrv -p 9876:9876 \
-v /data/docker/rocketmq/namesrv/logs:/root/logs \
-v /data/docker/rocketmq/namesrv/store:/root/store \
-e "MAX_POSSIBLE_HEAP=10000000" \
-e "autoCreateTopicEnable=true" \
rocketmqinc/rocketmq:4.3.2 sh mqnamesrv
```

创建 Broker 容器，这里很容易因为内存不足失败。

```
docker run -d --name broker-a --link namesrv:nameIP \
-p 10911:10911 -p 10909:10909 \
-v /data/docker/rocketmq/broker/logs:/root/logs \
-v /data/docker/rocketmq/broker/store:/root/store \
-v /data/docker/rocketmq/broker/conf/broker.conf:/opt/rocketmq-4.3.2/conf/broker.conf \
-e "NAMESRV_ADDR=nameIP:9876" -e "MAX_POSSIBLE_HEAP=40000000" \
rocketmqinc/rocketmq:4.3.2 sh mqbroker -c /opt/rocketmq-4.3.2/conf/broker.conf
```

创建控制台容器

```
docker run --name mq-console --link namesrv:nameIP \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=nameIP:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9001:8080 \
-d styletang/rocketmq-console-ng
```
