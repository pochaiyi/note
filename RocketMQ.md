# RocketMQ

# 消息队列概念

消息队列（Message Queue）是提供一套完整的消息生产、传递、消费等功能的软件。消息也就是数据，生产者向 MQ 发送消息，消费者从 MQ 读取消息。

MQ 最主要的作用是作为服务间的一种通信方式，相比于 RPC，它有什么不同呢？

## 程序解耦

服务间使用 MQ 进行通信，上游服务把数据封装成消息发送到 MQ，然后返回。下游服务从 MQ 拉取消息并进行处理。如此，上下游服务间没有直接联系，却执行完所有流程。

解耦使得分布式业务更易编写和维护，增加业务，只需新增服务订阅消息，删除业务则直接关闭子服务即可。

## 异步执行

基于前面内容。生产者把消息发送到 MQ 后就直接返回，后续的流程并不需要立即完成。至于什么时候完成？这取决于下游服务何时拉取消息。

因此，MQ 使得业务的上下游能够异步执行，这有效地减少请求的返回时间，增加系统吞吐量。相比之下，RPC 是同步通信，它要求远程服务立即执行。

## 削峰填谷

基于前面内容。在高并发场景，服务可能来不及处理，这将导致大量请求阻塞，甚至压垮服务器。可以先把请求封装成消息发送到 MQ，然后返回。使用异步线程或子服务，尽可能快的拉取消息进行处理。如此，可以延缓请求的处理，降低系统的瞬时压力。

# 消息传输模型

什么是传输模型？可理解为消息的组织、使用方式。主流的传输模型有：队列模型，发布订阅模型。

## 队列模型

队列模型，也称点对点模型，消息组织为队列结构，尾进头出。消息一旦出队，就被删除。所以，这种模型的消息只能被消费一次。因为消费即删除，所以队列模型没有为消费者赋予独立的身份。

## 发布订阅模型

发布订阅模型使用主题作为消息容器，消费者有明确的身份，通常是订阅组，不同的订阅组相互独立。主题能持久化消息，并基于订阅组身份记录消费进度，所以，单个主题能被多个订阅组消费，每个订阅组都能读取主题的全部消息，必要时还能重置消费进度，以此读取历史消息。

# 领域模型概念

RocketMQ 基于发布订阅模型设计，它的消息的生命周期分为生产、存储、消费三部分。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/rmq-0dfe6afa.png)

## 主题 Topic

**模型定义**

主题是 RocketMQ 消息的顶级容器，通过 TopicName 在集群内唯一标识和区分。主题的主要作用有两个：

* 消息分类：把不同业务的消息划分到不同的主题，以此实现消息的存储隔离和订阅隔离。

* 批量管理：通过主题，对一类消息进行统一管理，比如身份识别、权限管理。


**自动创建**

创建和管理主题需要占用系统资源，RocketMQ 默认支持自动创建主题，在生产环境应该关闭。

**版本差异**

每个主题应该只存储一种类型的消息。在 5.x 版本，服务端会校验接收的消息的类型；但 4.x 版本不支持消息类型检验，需要客户端自己保证类型一致。

## 队列 MessageQueue

**模型定义**

队列是 RocketMQ 消息的实际容器，通过 QueueId 在主题内唯一标识和区分。可以这么理解：队列是主题的具体实现，类似的，Kafka 使用分区实现主题。

队列的主要作用有两个：

* 顺序存储：在单个队列，消息按照写入的顺序进行存储，使用消息位点 offset 标记管理。
* 流式操作：基于位点，可从队列的任意位置读取任意数量的消息。

**读写权限**

队列可配置读写权限：6 表示读写，默认；4 表示只读；2 表示只写；0 表示不可读写。

通过 Dashboard 创建主题，底部的 perm 选项用于设置所有队列的读写权限。

**使用建议**

在 4.x 版本的集群消费模式，消息以队列为粒度分配给消费者。所以，增加队列数量可以拓展主题的消费能力。队列数量可在定义或修改主题时设置，应遵循少用够用原则。如果队列过多，不但增加服务端的管理压力，还会增加客户端压力，因为客户端是轮询地访问队列，容易产生无效请求，增加负荷。

## 消息 Message

**模型定义**

消息是 RocketMQ 最小的数据传输单元。每条消息都必定属于某个主题。消息不可变，其一旦被服务端接收，就不会改变。RocketMQ 会持久化消息。

**内部属性**

消息不单只有通信数据，还包含许多其它可设置的属性。

| 属性              | 必要 | 说明                                                         |
| ----------------- | ---- | ------------------------------------------------------------ |
| Topic             | 必须 | 主题名称。标识这条消息属于哪个主题。                         |
| Body              | 必须 | 负载。二进制内容，用于存放业务数据。                         |
| Tag               | 可选 | 标签。主题之下的二级分类，用于订阅筛选。                     |
| Key               | 可选 | 服务器基于Key建立哈希索引，方便系统快速查询，也能用于订阅筛选。 |
| Properties        | 可选 | 属性。key-value 格式，用于存放扩展信息。Tag 和 Key 本质也是属性。 |
| DelayTimeLevel    | 可选 | 延迟级别。用于设置消息的延迟时间。                           |
| DeliveryTimestamp | 可选 | 毫秒级时间戳。5.x 版本新增，补充延迟级别，是消息的生效时间。 |

## 生产者 Producer

**模型定义**

生产者是构建并发送消息到服务端的运行实体，通常集成在业务系统。单个生产者能向多个主题发送消息，单个主题也能接收多个生产者的消息。

生产者客户端可定义以下传输行为：

* 发送方式：同步，或异步。
* 批量发送：消息条数，和消息大小。
* 事务行为：事务消息需要生产者配合进行事务检查等行为，以保证最终一致性。

**内部属性**

| 属性         | 必要 | 说明                                                     |
| ------------ | ---- | -------------------------------------------------------- |
| 客户端 ID    | 必须 | 生产者客户端的标识，由客户端 SDK 自动生成，不支持修改。  |
| 接入点信息   | 必须 | 服务端的连接地址。                                       |
| 身份认证信息 | 可选 | 身份验证的凭证信息，仅在服务端开启身份识别和认证时需要。 |
| 请求超时时间 | 可选 | 网络请求的超时时间。                                     |
| 发送重试策略 | 可选 | 消息发送失败时的重试策略。                               |

**生产者组**

生产者基于生产者组进行身份认证。从 5.x 版本开始，生产者匿名，不再使用生产者组。

## 消费者组 ConsumerGroup

**模型定义**

消费者组是订阅组的实现，它是多个消费行为一致的消费者的负载均衡分组。消费者组不是运行实体，而是一个逻辑资源。基于消费者组，可以实现消费性能的水平扩展以及高可用。

**内部属性**

| 属性         | 说明                                                   |
| ------------ | ------------------------------------------------------ |
| 消费者组名称 | 标识符。                                               |
| 投递方式     | 发送消息方式，并发投递，或顺序投递。                   |
| 消费重试策略 | 消息消费失败时的重试策略，仅对 PushConsumer 类型有效。 |
| 订阅关系     | 消费规则。                                             |

**自动创建**

RocketMQ 默认支持自动创建消费者组，与主题相同，为避免资源浪费，应在生产环境应该关闭

**消费一致**

组内所有消费者的消费行为应该相同。在 5.x 版本，消费者客户端从服务端请求相关信息，以保证组内消费行为一致。但 4.x 版本不支持，需要客户端自己保证行为一致。

## 消费者 Producer

**模型定义**

消费者是从服务端拉取消息进行处理的运行实体，通常集成在业务调用链的下游系统。

消费者客户端可定义以下传输行为：

- 消费者身份：关联某个消费者组。
- 消费者类型：PushConsumer、PullConsumer 和 SimpleConsumer（5.x 新增）。
- 本地运行配置：比如消费并发度、客户端线程等。

**内部属性**

| 属性         | 必要 | 说明                                                     |
| ------------ | ---- | -------------------------------------------------------- |
| 客户端 ID    | 必须 | 生产者客户端的标识，由客户端 SDK 自动生成，不支持修改。  |
| 消费者组名称 | 必须 | 指定当前消费者属于哪个消费者组。                         |
| 接入点信息   | 必须 | 服务端的连接地址。                                       |
| 身份认证信息 | 可选 | 身份验证的凭证信息，仅在服务端开启身份识别和认证时需要。 |
| 请求超时时间 | 可选 | 网络请求的超时时间。                                     |
| 消费监听器   | 可选 | 监听服务端推送的消息，并进行处理，仅 PushConsumer 需要。 |

**行为一致**

RocketMQ 要求以下行为必须相同：投递顺序，消费重试策略，以保证组内消息的正常负载和消费。

**版本差异**

组内所有消费者的消费行为应该相同。在 5.x 版本，消费者客户端从服务端请求相关信息，以保证组内消费行为一致。但 4.x 版本不支持，需要客户端自己保证行为一致。

## 订阅关系 Subscription

**模型定义**

订阅关系是消费者拉取消息的规则，由消费者组动态注册到服务端，并在后续的消息传输中按照配置的过滤规则进行消息匹配和消费进度维护。

订阅关系可定义以下传输行为：

- 过滤规则：筛选主题内的哪些消息可以被消费，默认拉取全部消息。
- 消费状态：消费者组消费到什么位置，服务端会持久化订阅关系。

**内部属性**

过滤类型，有 Tag 和 SQL 两种可选；过滤表达式。

**订阅一致**

组内所有消费者的消费逻辑应该一致，比如过滤配置。

# 生产者实践

虽然本笔记主要参考 RocketMQ 5.x 版本的文档，但这里使用 4.9.4 版本进行实践。

## 客户端依赖

无论生产者或消费者，客户端都使用相同的依赖。客户端 SDK 与 RocketMQ 必须版本一致。

```
<dependency>
  <groupId>org.apache.rocketmq</groupId>
  <artifactId>rocketmq-client</artifactId>
  <version>4.9.4</version>
</dependency>
```

## 客户端实现

DefaultMQProducer 类是生产者的抽象，最常用，它能发送绝大部分类型的消息，事务消息除外。

```
DefaultMQProducer producer = new DefaultMQProducer();
producer.setProducerGroup("producerGroup"); // 4.x版本的生产者需要设置分组
producer.setNamesrvAddr("127.0.0.1:9876"); // NameSrv接入点信息，多个地址使用分号分隔
producer.start(); // 启动，然后就能使用该对象发送消息
```

上面仅是最基本的配置，还可以在启动前配置其它的发送行为，比如请求超时时间、发送失败策略。

```
producer.setDefaultTopicQueueNums(1); // 新创建主题的队列数量，默认4
producer.setSendMsgTimeout(1000); // 请求超时时间，单位毫秒
```

## 消息 Message

Message 类是消息的抽象，它至少关联某个主题，并设置二进制负载。

```
String msg = ...;
Message message = new Message("topic", msg.getBytes());
```

还能设置一个 Tag 属性用于消息过滤。

```
message.setTags("tag");
```

设置 Keys 属性，服务端将为其建立 Hash 索引。多个 key 使用空格分隔，或使用容器类设置。

```
message.setKeys("key1 key2");
```

设置 Properties 属性，存放拓展信息。

```
message.putUserProperty("key","value");
```

## 普通消息

普通消息是最简单的类型，没有任何特性。RocketMQ 生产者支持三种发送消息的方式：同步、异步和单向。前两种方式可靠，因为无论发送成功与否，服务端都返回发送结果。

### 同步发送

生产者发出消息后，将一直阻塞，直到收到返回结果，然后才能继续发送下一条消息。

只有一个参数的 send 方法，使用同步发送。返回类型 SendResult 是服务端响应结果的抽象。

```
SendResult sendResult = producer.send(message);
```

发送失败会抛出异常，必须捕获它，并设计兜底方案，忽略异常可能导致业务错误。

### 异步发送

生产者发出消息后，不等服务端响应，直接发送下一条消息。需要注册回调接口，用于接收响应结果和处理。

```
producer.send(message, new SendCallback() { // 第二个参数是异步回调接口的实现
	
	@Override
	public void onSuccess(SendResult sendResult) {...} // 发送成功时执行

    @Override
    public void onException(Throwable throwable) {...} // 发送失败时执行
});
```

### 单向发送

生产者直接发送消息，不等待也不接收服务端返回的响应。此方式最快，但不可靠。

```
producer.sendOneway(message);
```

### 批量消息

前面几种方式，都是发送单条消息。如果每条消息的发送都要进行一次网络通信，非常不划算。RocketMQ 支持批量传输，单次网络通信发送多条消息。但消息的主题必须相同，且不支持延迟消息和事务消息。

```
List<Message> messages = new ArrayList<>();
messages.add(new Message(...));
messages.add(new Message(...));
messages.add(new Message(...));
producer.send(messages); // 同步发送批量消息
```

## 顺序消息

顺序消息是生产顺序和消费顺序一致的消息。消息队列天然具有顺序性，先到达服务端的消息存储在队列前面。

### 生产顺序

消息的顺序性分为两部分：生产顺序和消费顺序。这里暂时只讨论生产顺序，首先需要满足以下条件：

* 单生产者：如果有多个生产者同时向一个主题发送消息，无法判断这些消息的先后顺序。
* 串行发送：生产者客户端支持多线程，如果有多个线程使用它发送消息，无法判断这些消息的先后顺序。

满足以上条件，可以保证，在一个主题的任何队列，所有消息都是按发送顺序存储。

### 队列选择

主题包含多个队列，普通的 send 方法会轮询地把消息发送到不同队列，实现存储均衡。我们可以使用选择器，自定义发送队列的选择策略。

接口 MessageQueueSelector 是选择器的抽象，它仅有一个方法 select。第一个参数表示主题的所有队列，第二个参数是待发送的消息，第三个参数是任意对象。返回对象就是消息将要发送到的目标队列。因此，我们可以根据消息本身和额外提供的对象，来制定队列的选择策略。

```
MessageQueueSelector selector = new MessageQueueSelector() {
    @Override
    public MessageQueue select(
            List<MessageQueue> list, Message message, Object arg) {
        ...
    }
};
```

顺序消息使用带有选择器参数的 send 重载进行发送。

```
SendResult sendResult = producer.send(message, selector, message);
```

### 全局有序

如果主题只有一个队列，那么该主题全局有序。而且，也没有必要使用选择器，反正只有一个队列，只需要保证消息的生产顺序。

### 版本差异

顺序消息和普通消息好像没有什么区别，完全是依靠队列的特性来保证顺序。这样的实现比较简单，但是限制所有消息遵守同一种排序规则。

在 5.x 版本，消息 Message 新增属性 MessageGroup 消息组，同组消息将存放在一个队列，并在组内按序存储。消费者支持按组消费。每个主题可以存储多组消息，不同分组能使用不同的排序规则。

## 延迟消息

延迟消息发送到服务端后，还需等待一定时间，才能被服务端投递。

Broker 收到延迟消息，首先把它存到名为 SCHEDULE_TOPIC_XXX 的主题。设有定时任务，每秒执行一次检查，如果发现某个延迟消息到期，则把它转移到原始主题，然后就能被投递。

RocketMQ 使用 18 个等级设置消息的延迟时间，如下表所示。

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

延迟等级是 Message 的属性，至于它的发送和消费，与普通消息相同。

```
message.setDelayTimeLevel(1); // 设置延迟等级
SendResult sendResult = producer.send(message);
```

> 在 5.x 版本，延迟/定时消息支持使用秒级时间戳，指定消息的生效时间。

## 消息发送重试

RocketMQ 生产者客户端支持消息发送重试机制。消息发送失败时，生产者将重新尝试发送。

单向消息不支持发送重试，毕竟生产者都不知道发送失败与否。顺序消息也不支持发送重试，原因未知。

**最大重试次数**

消费者不能无限重试，默认最多两次。当超过最大重试次数，抛出异常，由客户端自己保证消息不丢失。

DefaultMQProducer 支持修改最大重试次数。

```

```

**同步发送失败**

普通消息默认轮询地发送到不同队列。如果发送失败，重试时尽量选择其它 Broker，如果只有一个 Broker，则尽量选择其它队列。

```
producer.setRetryTimesWhenSendFailed(3); // 同步发送失败重试次数
```

**异步发送失败**

异步发送消息失败，生产者不会尝试其它 Broker，它将一直对一个 Broker 进行重试。

```
producer.setRetryTimesWhenSendAsyncFailed(3); // 异步发送失败重试次数
```

**消息刷盘失败**

消息刷盘失败，或 slave 不可用，生产者默认不尝试其它 Broker，可修改配置属性开启对其它 Broker 的重试。

```
retryAnotherBrokerWhenNotStoreOK = true
```

**发送重试导致消息重复**

发送重试能保证消息的成功发送、不丢失，但也可能导致消息重复。如果消息已经被服务端接收，但返回结果没有被生产者收到，生产者将误以为发送失败，进行重试，造成消息重复。

消息重复是消息队列最基本、最常见的问题，必须制定解决措施。

# 消费者实践

RocketMQ 支持多种类型的消费者：PushConsumer、PullConsumer 和 SimpleConsumer（5.x 新增）。

## Push 消费

服务端主动推送消息给消费者。这种类型消费者能及时消费消息，但如果没有做好流量控制，容易造成客户端消息堆积甚至崩溃。

### 客户端实现

DefaultMQPushConsumer 类是 PushConsumer 的抽象，以下是 DefaultMQPushConsumer 的基本配置。

```
consumer = new DefaultMQPushConsumer();
consumer.setConsumerGroup("consumerGroup"); // 消费者组名称
consumer.setNamesrvAddr("127.0.0.1:9876"); // NameSrv接入点信息，多个地址使用分号分隔
consumer.subscribe("topic", "*"); // 第一个参数是订阅Topic的名称，第二个参数是过滤表达式
consumer.start(); // 启动，然后该消费者将持续运行消费消息
```

DefaultMQPushConsumer 需要配置监视器，用于接收消息并进行处理。有两种监视器接口：

* MessageListenerConcurrently，并发消费。
* MessageListenerOrderly，顺序消费。

### 并发消费

消费者注册 MessageListenerConcurrently 监视器，它可能使用多个线程同时消费一个队列。该接口只有一个方法 consumeMessage，第一个参数表示接收的消息集合，第二个参数表示消费环境。返回值表示消费状态，是枚举类 ConsumeConcurrentlyStatus 的值，其中，CONSUME_SUCCESS 表示消费成功，RECONSUME_LATER 表示消费失败。

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

PushConsumer 使用多少个线程消费消息，这不固定，可以限制消费线程的数量。

```
consumer.setConsumeThreadMin(1); // 最小线程数
consumer.setConsumeThreadMax(5); // 最大线程数
```

### 顺序消费

多个线程同时消费一个队列，无法保证消费顺序。改为 MessageListenerConcurrently 监视器，它将按照顺序消费消息。该接口也只有一个方法 consumeMessage，参数和并发消费监视器的方法相同，但返回类型是枚举类 ConsumeOrderlyStatus，SUCCESS 表示消费成功，SUSPEND_CURRENT_QUEUE_A_MOMENT 表示消费失败。

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

PushConsumer 有两种消费模式：

* 集群模式：主题的任意消息，只能被同组的一个消费者消费一次。
* 传播模式：同组的所有消费者，都会消费主题全部的消息。

集群模式，主题的消息在组内按队列分摊，此时改变消费者数量，可以提升或降低消费性能。

DefaultMQPushConsumer 支持设置消费模式。

```
consumer.setMessageModel(MessageModel.CLUSTERING); // 集群模式
consumer.setMessageModel(MessageModel.BROADCASTING); // 传播模式
```

### 负载均衡

#### 负载均衡策略

集群模式，需要把主题的消息分配给组内的消费者。RocketMQ 提供多种分配策略，具体如下。

| 分配策略   | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| 平均分配   | 平分队列，如果不能整除，把剩余队列按顺序分配。               |
| 环形平均   | 把所有消费者看作环形链表，挨个分配队列。                     |
| 机房优先   | 选出与消费者同机房的队列，进行平均或环形平均分配。如果没有同机房队列，则平均或环形平均分配所有队列。 |
| 一致性哈希 | 把消费者和队列存入哈希环，按顺时针方向，把队列分配给距离最近的消费者。这种方式分配不均匀，但能减少由于消费者数量变化而进行 Rebalance 时需要变更的关系。 |

DefaultMQPushConsumer 支持设置负载均衡策略，默认平均分配。

```
DefaultMQPushConsumer::setAllocateMessageQueueStrategy(AllocateMessageQueueStrategy);
```

AllocateMessageQueueStrategy 是负载均衡策略接口，它仅有一个方法 allocate，返回队列集合，这就是预分配给消费者的消息。可以通过实现它自定义负载均衡策略，也可以使用现有的实现进行配置。

从 AllocateMessageQueueStrategy 可以看出，负载均衡是以队列为粒度进行分配。在平均分配，如果消费者比队列多，就会有消费者没有被分配队列，处于空闲状态，造成资源浪费。

> 在 5.x 版本，新增消息粒度的负载均衡功能，同一个队列的消息可以被多个消费者分摊消费。

#### 再平衡 Rebalance

在集群模式，如果订阅主题的队列数量，或消费者组的成员数量发生变化，那么之前负载均衡的结果就不再适用，需要进行 Rebalance，重新分配消费者和队列的关系。

**作用**

显然，Rebalance 机制能及时地调整分配关系，充分发挥所有消费者的消费性能。

**危害**

* 消费暂停

  Rebalance 的过程需要暂停所有队列的消费。

* 重复消费

  成功消费的消息还未来得及提交位点，此时消费者宕机，进行 Rebalance，该消息将被重复消费。

* 消费突刺

  如果重复消费过多，或暂停时间过长使得大量消息积压，将导致 Rebalance 后突然要处理大量消息。

## Pull 消费

客户端主动从服务端拉取消息，相比于 Push 模式，自主性更高。但这也就不能使用 RocketMQ 提供的一些服务，性能可能比较低。

Pull 模式有两种实现方式：Pull Consumer 和 Lite Pull Consumer。

### Pull Consumer

Pull Consumer 是非常原始的 Pull 方式，它需要指定拉取队列的主题、所在 Broker，以及 Queue ID，听起来已经感觉非常不方便。该方式已过时。

DefaultMQPullConsumer 类是 Pull Consumer 的抽象，以下是 DefaultMQPullConsumer 的基本配置。

```
DefaultMQPullConsumer consumer = new DefaultMQPullConsumer();
consumer.setConsumerGroup("consumerGroup"); // 消费者组的名称
consumer.setNamesrvAddr("127.0.0.1:9876"); // NameSrv接入点信息，多个地址使用分号分隔
consumer.start(); // 启动，然后就可以使用该消费者拉取消息
```

使用 DefaultMQPullConsumer 拉取消息。需要实例化 MessageQueue 对象明确指定拉取的队列，为此要设置队列的主题名称、Queue ID、Broker 名称。然后执行 pull 方法拉取消息，第一个参数是目标队列，第二个参数是过滤表达式，第三个参数是消费位点初始值，第四个参数是最大拉取消息条数。返回类型 PullResult 表示拉取结果，它的 pullStatus 属性表示拉取状态，FOUND 值表示拉取到消息。如果拉取成功，对消息进行处理后，还要手动更新消费位点。

```
MessageQueue queue = new MessageQueue(); // 此次要拉取的队列
queue.setQueueId(0); // Queue ID
queue.setTopic("topic"); // 队列所属主题的名称
queue.setBrokerName("centos-7-01"); // 队列所在Broker的名称
PullResult pullResult = consumer.pull(queue, "*", 0, 32); // 执行拉取
if (pullResult.getPullStatus().equals(PullStatus.FOUND)) { // 判断拉取成功
    List<MessageExt> messageExtList = pullResult.getMsgFoundList(); // 拉取的消息列表
    ...
    consumer.updateConsumeOffset(queue,pullResult. getNextBeginOffset()); // 更新消费位点
}
```

### Lite Pull Consumer

相比于 Pull Consumer，Lite Pull Consumer 方便的多。

DefaultLitePullConsumer 类是 Lite Pull Consumer 的抽象，以下是 DefaultLitePullConsumer 的基本配置。

```
DefaultLitePullConsumer litePullConsumer = new DefaultLitePullConsumer();
litePullConsumer.setConsumerGroup("groupName");
litePullConsumer.setNamesrvAddr("127.0.0.1:9876");
litePullConsumer.start();
```

DefaultLitePullConsumer 有 Subscribe 和 Assign 两种使用方式。

**订阅 Subscribe**

执行 subscribe 方法设置订阅关系，第一个参数是订阅主题的名称，第二个参数是过滤表达式。

```
litePullConsumer.subscribe("topic", "*");
litePullConsumer.start();
```

在 Subscribe 模式，RocketMQ 将为其进行负载均衡分配，与 PushConsumer 相同。简单执行 poll 方法拉取消息。LitePullConsumer 默认自动提交消费位点。

```
List<MessageExt> messageExts = litePullConsumer.poll();
```

DefaultLitePullConsumer 支持修改每次拉取的消息数量，默认每次只拉取一条。

```
litePullConsumer.setPullBatchSize(10);
```

**分配 Assign**

Assign 方式与 Pull Consumer 类型相似，操作比较复杂。首先，关闭自动提交消费位点。

```
litePullConsumer.setAutoCommit(false);
```

手动指定拉取的队列，处理完消息后还要手动提交消费位点。

```
Collection<MessageQueue> messageQueues // 获取订阅主题的所有队列
		= litePullConsumer.fetchMessageQueues("topic");

List<MessageQueue> list = new ArrayList<>(messageQueues);
List<MessageQueue> assignList = new ArrayList<>();
for (int i = 0; i < list.size() / 2; i++) {
	assignList.add(list.get(i));
}

litePullConsumer.assign(assignList); // 为消费者分配队列
litePullConsumer.seek(assignList.get(0), 10); // 设置已分配队列的消费位点初始值

 while (true) {
 	List<MessageExt> messageExts = litePullConsumer.poll(); // 拉取消息
 	litePullConsumer.commitSync(); // 提交消费位点
 }
```

## 批量拉取数量

不论 Push 或 Pull 类型的消费者，每次拉取都是批量进行，默认每次只拉取一条消息。

DefaultMQPushConsumer 类支持修改批量拉取的消息数量。

```
consumer.setPullBatchSize(10);
```

批量拉取的消息数量最大是 32，DefaultMQPushConsumer 支持修改这个限制。

```
consumer.setConsumeMessageBatchMaxSize(35);
```

## 订阅消息过滤

投递消息时，可使用过滤条件，限制某些消息不能被投递。如果未设置，默认投递所有消息。

RocketMQ 支持两种过滤方式：Tag 过滤和 SQL 过滤。任何使用 Tag 过滤的地方，都有 SQL 过滤的重载。

### Tag 过滤

生产者端为消息设置标签，每条消息只允许设置一个 Tag 属性。

```
Message message = new ...
message.setTag("tag");
```

使用过滤表达式，限制 Tag 与指定值相等的消息才能被投递。

```
consumer.subscribe("topic", "*"); // 所有消息
consumer.subscribe("topic", "tagA"); // Tag等于"tagA"
consumer.subscribe("topic", "tagA||tagB"); // Tag等于"tagA"或"tagB"
```

### SQL 过滤

SQL 过滤筛选的范围更大，语义更灵活，它筛选的依据是消息的 Properties 属性。注意，Tag 其实也是消息属性，只不过它的键是保留字。Tag 的键是 "TAGS"。

生产者端为消息设置属性，每条消息允许设置多个 Properties 属性。

```
message.putUserProperty("key", "value");
```

使用 SQL 语句，限制 Properties 符合要求的消息才能被投递。

```
consumer.subscribe("topic", 
	MessageSelector.bySql("TAGS in ('TagA', 'TagB')) and (k between 0 and 3)"));
```

SQL 的语法不必多将，但有些关键字在这有不同的含义。比如 =、!=、between、in 等运算符，比较的是 value，而 is null、is not null 等运算符，比较的是 key。

## 消费进度管理

### 消息位点

消息按到达服务端的顺序存储在指定主题的某个队列，每条消息在队列上都有一个唯一的 Long 类型坐标，即消息位点 MessageQueueOffset。

队列中最早的一条消息，即队头消息的位点，称为最小消息位点 MinOffse。最新的一条消息，即队尾消息的位点，称为最大消息位点 MaxOffset。

消息位点能从 0 到 Long.MAX 无限增加，所以逻辑上，队列能无限存储。但磁盘空间有限，RocketMQ 将滚动删除队列中最早的消息。因此，队列的 MinOffse 和 MaxOffset 会一直递增变化。

"主题+队列+位点" 可以精确定位任何一条消息。

### 消费位点

消息被某个消费者成功消费，不会像队列模型那样立即被删除。RocketMQ 在所有队列，都要记录各个消费者组在此的消费进度，即消费位点 ConsumerOffset。

在集群模式，消费位点由客户端提交给服务端保存。在广播模式，消费位点由客户端自己保存。所以，集群模式的消费者重新上线后，能继续之前的进度进行消费。如果服务端保存的消费位点过期或被删除，将把消费位点前移至最小消息位点，即从头消费。

### 重置消费位点

**初始消费位点**

消费者组初次启动消费者进行消费时，它的消费位点称为初始消费位点，表示从哪开始消费。RocketMQ 规定初始消费位点为当时队列的最小消息位点。

**重置消费位点**

如果初始消费位点或当前消费位点不满足需求，可以通过重置消费位点调整消费进度。

DefaultMQPushConsumer 类的 setConsumeFromWhere 方法能重置消费位点，它的参数是 3 个枚举值：

* ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET：最小消息位点。

* ConsumeFromWhere.CONSUME_FROM_LAST_OFFSET：最大消息位点。

* ConsumeFromWhere.CONSUME_FROM_TIMESTAMP：从指定时间后的消息开始，Properties 设置时间。

  ```
  consumer.setConsumeTimestamp(“20210701080000”) # yyyyMMddHHmmss
  ```

这个方法存在较多限制。首先，必须是首次订阅，如果主题保存有该消费者组的消费进度，将直接从历史消费位点继续。然后，主题应该存在较长时间，如果最早的消息都没有过期，服务端会认为这是一个新上线的业务，强制从第一个消息开始消费。

## 消息消费重试

### 消费重试

消息如果消费失败，或处理超时，或在 PushConsumer 排队超时，将触发消费重试。此时，服务端根据策略重新投递消息，若达到最大重试次数仍未消费成功，消息会被投递到死信队列。

消费重试只适用于 Push 类型的消费者，且只在集群模式有效，如果传播模式发生消费失败，直接跳过这条消息。

**最大重试次数**

DefaultMQPushConsumer 支持修改每条消息的重试次数，默认 16 次。

```
consumer.setMaxReconsumeTimes(10);
```

**普通消息**

普通消息消费失败，重试的时间间隔逐渐递增：10 s、30 s、1 min、2 min、3 min、...。

**顺序消息**

顺序消息消费失败，为保证消费顺序，消费者将反复重试这条消息，直至消费成功或达到最大重试次数。重试期间消费者阻塞，每次重试的时间间隔默认 1000 毫秒。

DefaultMQPushConsumer 支持修改顺序消息的重试时间间隔，区间 [10,30000]，单位毫秒。

```
consumer.setSuspendCurrentQueueTimeMillis(5000);
```

### 死信队列

如果达到最大重试次数，仍未消费成功，消息将被投递到该消费者组对应的死信队列，这条消息也变为死信消息。

每个消费者组都有一个对应的死信主题，名为 %DLQ%ConsumerGroupName，该主题只有一个死信队列。

消费者组默认订阅死信主题，但死信队列直到有死信消息时才创建。死信消息的有效期与普通消息相同，但它不能被消费。可以使用 RocketMQ Admin 或 RocketMQ Dashboard 查看死信消息。

# NameServer

## 基本介绍

Namesrv 在 RocketMQ 系统中扮演协调者的角色，它的主要功能是临时保存和管理 Topic 路由信息。Topic 包含多个队列，这些队列可能分布在不同的 Broker 节点，Topic 的路由信息就是它包含的队列的位置。

NamaSrv 是无状态的，集群中的多个 Namesrv 节点相互独立，没有任何联系。这使得 Namesrv 集群的搭建非常简单：启动所有 NamaSrv 服务，然后在所有 Broker 节点添加它们的地址。

Namesrv 工作的开销非常小，它主要的活动是维护与客户端的连接，以及动态更新 Topic 路由信息。

## 路由注册

Broker 设计有定时心跳机制，在服务启动时开始心跳。每次心跳，Broker 都会遍历配置的所有 Namesrv 节点，向它们发送自己的 Topic 路由信息以及其它信息。默认心跳间隔 30 秒。

首次心跳时，NameSrv 第一次存储该 Broker 的信息，这就是 Broker 的路由注册。之后的心跳，便是为了动态更新所有信息。

## 路由剔除

如果某个 Broker 注册后，Namesrv 长时间没有收到它的心跳，或收到它的关闭信息，NameSrv 将剔除该 Broker 节点，删除它的所有信息。

客户端程序通过与 NameSrv 的长连接，动态获取最新的路由信息。所以，当某个 Broker 被剔除，客户端也能及时知道。因此，Broker 的注册、剔除对客户端来说都是透明的。

## 客户端连接

类似于分布式服务的情况，Broker 节点在运行时可能新增、减少、变化，如果在客户端动态维护这些信息，非常不方便。这就凸显 NameSrv 的重要性，它相当于 Eureka 。客户端只需与 NameSrv 通信，就能在任何时候获取实时的、正确的路由信息。NameSrv 使得 Broker 的注册、剔除对客户端透明。客户端只需配置 NameSrv 集群的接入点信息。

如果有多个 Namesrv 节点，请求路由信息时，客户端将生成一个随机数，把该数与 Namesrv 数量求模，结果就是本次访问的 Namesrv 的下标。然后尝试与它连接，如果失败，则轮询其它 Namesrv 节点。

## 启动和关闭

RocketMQ 安装目录下的 *bin* 子目录中，*mqnamesrv* 脚本与 NameSrv 服务相关。它的使用格式如下。

```
mqnamesrv [-c <configFilePath>] [-h] [-p] [key=value key=value [,key=value...]]
```

-c 参数指定配置文件路径；-h 获取帮助信息；-p 查看所有可配置项。

RocketMQ 支持在配置文件或命令行，设置配置属性，格式 "key=value"，且后者优先级更高。

启动服务，指定服务端口为 9876，这也是默认端口。

```
nohup sh ./bin/mqnamesrv listenPort=9876 &
```

查看日志，检查启动是否成功。

```
tail -f ~/logs/rocketmqlogs/namesrv.log
```

关闭服务，执行 *mqshutdown* 脚本。

```
sh ./bin/mqshutdown namesrv
```

# Broker

## 基本介绍

Broker 是 RocketMQ 系统中真正提供消息队列服务的模块，主要功能是存储消息以及处理各种请求。

## 存储机制

### 存储目录

RocketMQ 具有持久化功能，安装启动后将自动生成若干文件，这些文件默认存到用户目录的 *store* 子目录。目录结构如下所示。

| 文件         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| commitlog    | 消息文件目录。文件名由所存消息的最大物理offset在高位补零构成。默认大小1GB。 |
| consumequeue | 消费队列文件目录。文件命名格式：*./consumequeue/Topic name/queue id/具体文件*。消费队列本质是commitlog的索引，提供给消费者做拉取消息、更新位点使用。 |
| index        | 哈希索引文件目录。根据消息的key创建，文件名是创建时间戳。    |
| config       | 配置文件目录，保存当前Broker的所有Topic、订阅关系和消费进度。这些数据定时从内存刷新到磁盘，以便宕机恢复。 |
| abort        | 标志文件。存在时表示上次关闭异常，Broker据此进行某些操作，比如重建index文件。 |
| checkpoint   | 状态文件。存储Broker上次正常关闭时的状态，比如最后一次正常刷盘时间。 |
| lock         | 资源文件。Broker运行期间使用的全局锁。                       |

数据文件的存储目录，可通过配置属性进行设置。

| 属性                  | 默认                 |                         |
| --------------------- | -------------------- | ----------------------- |
| storePathRootDir      | ~/store              | store 目录位置。        |
| storePathCommitLog    | ~/store/commitlog    | commitlog 目录位置。    |
| storePathConsumeQueue | ~/store/consumequeue | consumequeue 目录位置。 |
| storePathIndex        | ~/store/index        | index 目录位置。        |
| storeCheckpoint       | ~/store/checkpoint   | checkpoint 文件位置。   |
| abortFile             | ~/store/abort        | abort 文件位置。        |

### 消息文件

RocketMQ 把消息存到 commitlog 文件。commitlog 目录包含多个文件，其实只有一个文件，但为操作方便，把它切分为多个子文件，这些子文件通过其保存的消息的物理位点首尾相连。

commitlog 文件的大小默认是 1GB，可通过配置属性 mapedFileSizeCommitLog 修改，单位 KB。

```
mapedFileSizeCommitLog = 1073741824
```

当一个 commitlog 文件写满，就创建一个新文件，并继续上一个文件的 offset 写入。

### 优化技术

RocketMQ 写文件非常快，这里介绍它所使用到的优化技术。

**缓存页 Page Cache**

现代操作系统内核被设计为按照 Page 读取文件，每个 Page 的默认大小是 4KB。当 RocketMQ 请求一段文件内容，操作系统将以 Page 的粒度进行读取，对于那些被加载的请求内容以外的部分，这就是预读，下次的请求内容如果命中 Page Cache，就能直接返回，而不用 IO 磁盘。

> 这非常类似 MySQL 的加载机制，都是以页为粒度读写磁盘，只不过 MySQL 的页在程序实现。

当然，Page Cache 也有缺点，比如脏页回写、内存回收等造成的读写延迟。RocketMQ 为此做了许多优化，比如内存预分配、文件预热等。

**虚拟内存 Virtual Memory**

所谓的虚拟内存，其实就是磁盘。操作系统从磁盘划分一部分空间，作为交换区。当内存不足时，操作系统就把一些暂时不用的内存数据转存到交换区，从而空出一些空间用于程序运行。

此时，操作系统可分配内存=物理内存+虚拟内存。所以，虚拟内存使得系统能够运行占用内存空间更大的程序。

**零拷贝技术**

Java 原始读取文件的流程：磁盘 -> 内核态内存 -> 用户态内存 -> Java 虚拟机。文件内容被操作系统读取后，还需要经历两次拷贝，才能被 Java 进程使用。

为提高读写效率，RocketMQ 在读写文件操作中大量使用零拷贝技术。java.nio.MappedByteBuffer 类实现零拷贝，它直接操作内核态内存，免去原来的两次拷贝过程。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/rmq-86d6fe6f.png)

### 高效读写

RocketMQ 读写消息，实质就是读写 commitlog 文件。前面提到 RocketMQ 大量使用零拷贝技术，基本的结构如下图所示。

CommitLog 类是完整 commitlog 文件的抽象。MappedFileQueue 包含所有映射文件，按过期时间排序，新消息总是写到最后一个文件。MappedFile 是 commitlog 文件的抽象，使用 MappedByteBuffer 类的零拷贝技术实现内存映射，同时拥有内存的写入速度和磁盘一样可靠的持久化方式。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/rmq-42384640.png)

写消息时，先加全局锁，因此任何时刻最多只有一个 commitlog 文件被写入。然后，使用 Append 方式写入最新的 MappedFile。对于读消息，大部分时候消费者只关注最新消息，而这些消息都在 Page Cache。即使要读取历史消息，也仅在第一次拉取消息时读磁盘，后续就可以利用磁盘预读，几乎做到不再直接读磁盘。

## 消息清理

由于磁盘的空间有限，Broker 无法永久保存所有数据，RocketMQ 通过设置过期时间清理数据文件。

由于内存和磁盘的空间有限，Broker 不能永久保存所有数据，RocketMQ 通过设置过期时间清理额外的数据文件，它每日都定时检查 commitlog 文件，并删除过期文件，同时删除关联的其它数据，比如 index 文件。

通过配置属性 fileReservedTime 设置 commitlog 文件的保留时间，默认 48 小时。

```
fileReservedTime = 48
```

通过配置属性 deleteWhen 设置删除时间点，默认凌晨 4 点。每日此时将检查并删除过期 commitlog 文件。

```
deleteWhen = 04
```

## 刷盘机制

消息被 Broker 接收并存到 Page Cache 后，需要进行持久化。RocketMQ 支持两种刷盘方式：

* 同步。Broker 同步调用方法，把包含新消息的 Page Cache 写入磁盘，结束后才返回客户端结果。

* 异步。消息一旦被成功写入 Page Cache，就返回客户端结果，由异步服务负责刷盘。

通过配置属性 flushDiskType 设置刷盘方式，默认异步刷盘，可选值如下。

| 可选值      | 说明 |
| ----------- | ---- |
| SYNC_FLUSH  | 同步 |
| ASYNC_FLUSH | 异步 |

异步刷盘存在一种特殊情况，如果此时开启 DM 读写分离，则消息首先被写入 DM，然后返回客户端结果。由异步转存服务负责把消息转移到 Page Cache，至于刷盘，同样由异步刷盘服务负责。

异步刷盘服务暴露有许多可配置属性，用于调控其运行方式，下表列出部分内容。

| 属性                             | 默认  | 说明                                   |
| -------------------------------- | ----- | -------------------------------------- |
| flushCommitLogTimed              | true  | true：定时刷盘；false：实时刷盘。      |
| flushIntervalCommitLog           | 500   | 定时刷盘时间间隔，单位毫秒。           |
| flushPhysicQueueThoroughInterval | 60000 | 两次实时刷盘的最大时间间隔，单位毫秒。 |
| flushCommitLogLeastPages         | 4     | 每次刷盘的页数                         |

## 读写分离

读写分离，可以有效减少瞬时压力，增加吞吐量。RocketMQ 有两处地方可使用读写分离机制。

### 主从读写分离

在主从复制集群，通常都是由 master 处理所有读写请求，如果开启主从读写分离，当 master 负载过高，客户端就从 slave 拉取消息。

从节点通过配置属性 slaveReadEnable 设置允许拉取消息。

```
slaveReadEnable = true
```

此后，消费者初始依然从 master 拉取消息，且每次都计算 diff，这是当前拉取的最大消息位点和 Broker 所有消息的最大物理位点的差值。如果 diff 大于消息的最大可用内存，说明 master 有较多消息还未消费，认为 master 此时内存繁忙。master 在返回结果时，将提示客户端下次尝试从 slave 拉取消息。

通过配置属性 accessMessageInMemoryMaxRatio 设置最大可用内存，它表示物理内存的占比，默认 40。

```
accessMessageInMemoryMaxRatio = 40
```

### DM 读写分离

在刷盘机制已经谈到，如果 Broker 使用异步刷盘，并开启 DM 读写分离。客户端发送的消息，首先是写到 DM，然后由异步转存服务将其转移到 Page Cache，最后由异步刷盘服务写到磁盘。

在此过程当中，消息写到 DM 就返回客户端写入成功，消息被异步服务转移到 Page Cache 时，客户端就能消费这个消息。所以，消息的读和写是分开的，这也算是读写分离。这样做的目的，我认为，因为 DM 是 Java 直接从操作系统申请的内存，写入 DM 比写入 Page Cache 快的多，所以，这能提高写请求的返回速度，增加吞吐量。

> 不论何时，客户端总是从 Page Cache 读取消息。

通过配置属性 transientStroePoolEnable 开启 DM 读写分离，默认关闭。

```
transientStroePoolEnable = true
```

## 主从复制

### 基本介绍

RocketMQ 支持主从复制，它的好处不必多说，高可用+数据安全。Broker 自然有 master 和 slave 两种角色。

主节点负责处理各种请求，并存储数据。从节点同步数据到本地，它的作用体现在两个方面：

* 读写分离。有效减轻主节点的压力，提升服务性能。
* 高可用。如果 master 宕机，slave 依然可以提供消费服务。

### 同步方式

从节点需要从主节点同步数据至本地。同步的数据有两种类别：

* 配置数据：包括主题信息、消费位点等，主节点启动时将开启定时任务，每隔 60s 进行一次同步。
* 消息数据：主要就是 commitlog 文件的内容，也是我们关注的重点。

对于消息数据，RocketMQ 支持两种主从复制方式：

* 同步复制：消息被主节点写入 Page Cache 后，需要同步传输到从节点，才能返回客户端结果。
* 异步复制：消息被主节点写入成功后，直接返回客户端结果，由异步服务负责同步主从数据。

Broker 可通过配置属性 brokerRole 设置主从角色，以及主从复制的方式，可选值如下。

| 可选值       | 说明               |
| ------------ | ------------------ |
| SYNC_MASTER  | 主节点，同步复制。 |
| ASYNC_MASTER | 主节点，异步复制。 |
| SLAVE        | 从节点。           |

### 主从搭建

主节点和从节点不直接连接，它们依靠 NameSrv 进行组合。因此，主从节点应该配置相同的 NameSrv 地址。

```
namesrvAddr = 127.0.0.101:9876;127.0.0.102:9876
```

NameSrv 通过配置属性 brokerName 区分不同的 Broker 节点。主从集群应该给人是一个节点的感觉，因此，主从节点应该配置相同的名称。

```
brokerName = broker-a
```

主从集群内部，使用 brokerId 区分不同节点，并且，主节点的 Id 必须是零，从节点大于零。

```
brokerId = 0
```

最后，使用 brokerRole 配置属性，为各节点分配角色，以及设置同步复制方式。

```
brokerRole = SYNC_MASTER
```

## 集群部署

RocketMQ 支持集群模式，Topic 的队列将平均分布在这些节点。若某个节点失效，将进行 Rebalance，重新分配队列位置，失效节点的消息将不可用，除非它配置有 slave 继续提供消费服务。

集群的配置非常简单。首先，它们必须配置相同的 NameSrv 地址。

```
namesrvAddr = 127.0.0.101:9876;127.0.0.102:9876
```

配置属性 brokerClusterName 标识节点属于哪一个集群，集群内所有节点的这个属性的值应该相同。

```
brokerClusterName = DefaultCluster
```

至此，集群搭建完成。

## 其它配置

| 属性                        | 默认           | 说明                                                 |
| --------------------------- | -------------- | ---------------------------------------------------- |
| brokerClusterName           | DefaultCluster | Broker集群名称。                                     |
| defaultTopicQueueNums       | 4              | 默认新建主题的队列数量。                             |
| autoCreateTopicEnable       | true           | 是否允许自动创建主题。                               |
| autoCreateSubscriptionGroup | true           | 是否允许自动创建订阅关系。                           |
| brokerIP                    |                | 主机IP地址，可空，系统将自动识别，但多网卡时不准确。 |
| listenPort                  | 10911          | Broker服务的端口。                                   |
| haListenPort                | 10912          | 主节点监听从节点请求的端口。                         |
| maxMessageSize              | 65536          | 消息的最大大小，单位KB。                             |
| mapedFileSizeConsumeQueue   | 300000         | 队列最大存储的消息条数。                             |

## 启动和关闭

RocketMQ 安装目录下的 *bin* 子目录中，*mqbroker* 脚本与 Broker 服务相关。它的使用格式如下。

```
mqbroker [-c <arg>] [-h] [-m] [-n <arg>] [-p]
```

-c 参数指定配置文件路径；-h 获取帮助信息；-p 查看所有可配置项；-m 查看重要的可配置项；-n 参数指定 NameSrv 服务的地址，多个地址使用分号分隔，例如 *192.168.0.1:9876;192.168.0.2:9876*。

同样，mqbroker 脚本也支持在配置文件或命令行设置配置属性。

启动服务，指定向本机 NameSrv 注册路由信息。

```
nohup sh ./bin/mqbroker -n 127.0.0.1:9876 &
```

查看日志，检查启动是否成功。

```
tail -f ~/logs/rocketmqlogs/broker.log
```

关闭服务，这是使用 *mqshutdown* 脚本。

```
sh ./bin/mqshutdown broker
```

# JVM 配置

NameSrv 和 Broker 都运行在 Java 虚拟机，安装目录下的 *bin* 子目录中，*runserver.sh* 和 *runbroker.sh* 脚本分别能配置这两服务的 JVM 启动参数。

对于个人主机，内存空间较小，必须对 JVM 的各种内存使用进行限制，否则服务大概率无法成功启动。

```
# runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

```
# runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms256m -Xmx256m -Xmn128m"
```

# Dashboard

RocketMQ 官方提供可视化管理平台 Dashboard，通过它可以很方便的查看 RocketMQ 运行情况。

下载地址：https://github.com/apache/rocketmq-dashboard

# Docker 搭建

拉取镜像

```
docker pull rocketmqinc/rocketmq:4.3.2
dokker pull styletang/rocketmq-console-ng
```

`broker.conf` 配置

```
#集群名称
brokerClusterName = DefaultCluster
#broker名称
brokerName = broker-a
#0表示Master
brokerId = 0
#清除消息时间，默认凌晨4点
deleteWhen = 04
#消息保留时长，单位小时
fileReservedTime = 48
#同步规则：SYNC_MASTER、ASYNC_MASTER、SLAVE
brokerRole = ASYNC_MASTER
#刷盘策略：ASYNC_FLUSH、SYNC_FLUSH、SYNC_FLUSH
flushDiskType = ASYNC_FLUSH
# 磁盘使用达到95%之后,生产者再写入消息报错
disk full

brokerIP1 = brokerIP
namesrvAddr = nameSrvIP:Port
```

启动 NameServer

```
docker run -d --name namesrv -p 9876:9876 \
-v /data/docker/rocketmq/namesrv/logs:/root/logs \
-v /data/docker/rocketmq/namesrv/store:/root/store \
-e "MAX_POSSIBLE_HEAP=10000000" \
-e "autoCreateTopicEnable=true" \
rocketmqinc/rocketmq:4.3.2 sh mqnamesrv
```

启动 Broker，这里因为内存极容易启动失败

```
docker run -d --name broker-a --link namesrv:nameIP \
-p 10911:10911 -p 10909:10909 \
-v /data/docker/rocketmq/broker/logs:/root/logs \
-v /data/docker/rocketmq/broker/store:/root/store \
-v /data/docker/rocketmq/broker/conf/broker.conf:/opt/rocketmq-4.3.2/conf/broker.conf \
-e "NAMESRV_ADDR=nameIP:9876" -e "MAX_POSSIBLE_HEAP=40000000" \
rocketmqinc/rocketmq:4.3.2 sh mqbroker -c /opt/rocketmq-4.3.2/conf/broker.conf
```

启动控制台

```
docker run --name mq-console --link namesrv:nameIP \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=nameIP:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9001:8080 \
-d styletang/rocketmq-console-ng
```
