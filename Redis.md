# Redis

# 初识 Redis

## Redis 是 NoSQL

NoSQL（Not Only SQL）指非关系型数据库。传统数据库依赖业务逻辑，使用关系表存储数据，遵守严格的约束和定义。Redis  是一种 NoSQL，使用 key-value 结构存储数据，不遵循 SQL 标准，不支持 ACID。而且它的数据都运行在内存，没有磁盘 IO，因此读写速度远远快于关系型数据库。

NoSQL 数据库适用于 "数据量小但对性能有一定要求" 的场景。

## Centos 安装 Redis

Redis 基于 c 开发，首先安装相应的语言环境。

```
yum install -y centos-release-scl scl-utils-build
yum install -y devtoolset-8-toolchain
scl enable devtoolset-8 bash
```

检查 c 环境是否安装成功。

```
gcc --version
```

开始安装 Redis，这里使用官网压缩包。

```
tar -zxvf <压缩文件> [-C 解压目录]
```

在解压目录，执行编译。

```
make
```

如果编译发生错误，执行以下命令进行恢复。

```
make distclean
```

在解压目录，执行安装。make test 是新命令，destdir 指定安装目录，默认 */usr/local/bin*。

```
make install|test [destdir=安装目录]
```

## Redis 的启动和连接

Redis 安装目录通常位于 */usr/local/bin/*，其中有许多可执行文件，启动或连接 Redis 都是通过它们完成。

### 启动服务器

redis-server 脚本用于启动 Redis 服务器，有两种启动方式。

**前台启动**

直接执行 redis-server 程序，当前命令窗口将进入 Redis 界面，退出界面会关闭 Redis 服务。

**后台运行**

需要修改配置文件的配置属性，设置后台启动。

```
daemonize yes
```

执行 redis-server 程序，添加配置文件路径参数。

```
redis-server <配置文件路径>
```

现在，虽然命令窗口没有变化，但 Redis 服务已在后台运行，以下使用 ps 命令查看进程。

```
ps -aux | grep redis
```

### 客户端连接

redis-cli 是 Redis 提供的用于连接服务器的客户端程序，它的使用格式如下。-h 指定 Redis 主机地址，-p 指定 Redis 服务端口，-a 指定 Redis 服务密码。

```
redis-cli [-h host] -p <port> [-a password]
```

通过 redis-cli 请求关闭 Redis 服务，有两种方式：连接 Redis 服务，执行 shutdown 命令；或在 shell 窗口执行以下命令。

```
redis-cli -p <port> shutdown
```

## 配置属性和配置文件

### 基本概念

类似 MySQL，Redis 提供许多可修改属性用于调控运行方式，支持配置文件。通常，Redis 压缩包自带一个标准配置文件，名为 redis.conf。

在客户端程序，可用 config 命令设置、查看配置属性，Redis 有些属性支持动态修改。

```
config get <属性名匹配模式>
```

```
config set <属性名> <属性值>
```

通过配置文件可以持久性地设置 Redis 运行方式，格式如下：

```
属性名 属性值
```

配置文件支持引用其它配置文件的内容，如果属性被多次设置，总是最有一个生效，且本地配置的优先级最高。

```
# include <其它配置文件路径>
```

除了手动修改文件，Redis 可以使用 config 命令将运行时的配置属性持久化到配置文件。

```
config rewrite
```

### 常用属性

| 属性             | 默认       | 描述                                                         |
| ---------------- | ---------- | ------------------------------------------------------------ |
| port             | 6379       | Redis 暴露出去的服务端口                                     |
| bind             | 127.0.0.1  | 限制 Redis 只接受指定主机的客户端连接                        |
| protected-mode   | yes        | 限制 Redis 只接受本机的客户端连接                            |
| requirepass      | ""         | 密码，空字符串表示没有                                       |
| maxclients       | 10000      | 最大连接数，达到上限后将拒绝新的连接                         |
| timeout          | 0          | 空闲连接存活时间，0 表示永远存在                             |
| tcp-keepalive    | 300        | 心跳检测时间间隔，单位秒，0 表示不检测                       |
| dir              | ./         | 各种数据的存放目录                                           |
| logfile          | ""         | 日志文件名，空字符串表示不输出日志                           |
| loglevel         | notice     | 日志记录级别，可选 debug，verbose，notice，warning           |
| maxmemory        | 0          | 最大内存限制，0 表示不限制。当 Redis 使用的内存达到上限，将会根据指定的策略移除一些数据，以留出空间。 |
| maxmemory-policy | noeviction | 使用的内存达到上限之后，移除数据的策略，以下可选：<br />volatile-lru：最近最少使用，对有过期时间的 key 生效；<br />allkeys-lru：最近最少使用，对所有 key 生效；<br />volatile-random：随机移除，对有过期时间的 key 生效；<br />allkeys-random：随机移除，对所有 key 生效；<br />volatile-ttl：移除将要过期的 key；<br />noeviction：什么都不做，对申请内存的操作返回错误信息。 |

## 数据库

Redis 没有什么表结构，所有数据都是键值对，它管理有多个数据库，数据库可以理解为是键值对的集合，各个库相互独立。默认有 16 个数据库，客户端初始会连接第 0 个库。

**修改数据库数量**

配置属性 databases 用于指定数据库的数量，可以执行 config get databases 查看该属性的值。

使用 config set 命令，或者修改 Redis 配置文件，自定义数据库的数量。

**常用数据库命令**

查看当前数据库，返回结果中的 bd 属性表示当前数据库的序号

```
client list
```

更换数据库，number 表示目标数据库序号

```
select number
```

清空当前数据库

```
flushdb
```

清空所有数据库

```
flushall
```

## 键

前面提到，Redis 数据都是键值对，通常都是针对键来操作数据，比如增删改查。

### 基本操作

**设值**

键与值必须同时设置，只有键或只有值没有意义，值得设置在基本数据类型章节介绍。

**删除**

通过指定键来删除若干个键值对

```
del key [key ...]
```

**判断存在**

如果指定的键存在，返回 1，否则返回 0。

```
exists key
```

**重命名键**

如果 newkey 已经存在，rename 会覆盖旧值，而 renamenx 返回 0，不执行修改。如果 key 不存在，报错。

```
rename|renamenx oldkey newkey
```

**查看当前库键的数量**

```
dbsize
```

**查看指定键的值的类型**

```
type k1
```

### 键的查找

**keys 命令**

查找键名与正则表达式匹配的键，? 匹配单个字符，* 匹配零或若干个字符。

```
keys pattern
```

**scan 命令**

基于游标的迭代器。cursor 指定从第几个游标位置开始查找，0 表示初始位置；MATCH 指定键名匹配模式，默认查找所有键；COUNT 指定此次遍历的游标数量，即会检查多少个键。

返回结果分为两组：第一组只有一个数字，表示此次查找结束位置的下一个位置，可以作为下一次迭代查找的起始游标位置，返回 0 表示遍历完所有键；第二组有若干个元素，是此次查找匹配的键。

```
scan cursor [MATCH pattern] [COUNT count]
```

> 这里的游标，可以理解为所有满足匹配模式的键中，某个键的序号，基于 0。

> **keys 和 scan 的区别**
>
> keys 以阻塞方式运行，并且是全盘扫描，键较多时非常耗时，而 Redis 是单线程，所以在此期间 Redis 将无法做其它任何事情。scan 以非阻塞方式运行，而且，它能使用 COUNT 限制此次扫描的键的数量，所以单次运行效率比较高，适合用于遍历大量键，常用于 Lua 脚本。

### 生存时间

Redis 支持对键值对设置生存时间，超时之后会被自动删除，默认永不过期。

**设置生存时间**

使用 expire 或 pexpire 命令设置生存时间，前者单位秒，后者单位毫秒，这会覆盖已有生存时间。

```
expire key seconds
```

```
pexpire key milliseconds
```

**查看生存时间**

使用 ttl 或 pttl 命令查看剩余生存时间，前者单位秒，后者单位毫秒，键不存在返回 -2，键永不过期返回 -1。

```
ttl|pttl key
```

**取消生存时间**

删除生存时间之后，键值对永不过期。

```
persist key
```

# 基本数据类型

对于 Redis 键值对，键的类型默认是 String，用户不能指定，这里学习值的类型。

## 字符串 String

### 相关概念和原理

字符串 String 是 Redis 最常用的数据类型，最多可以存储 512 MB 内容。

String 具有二进制安全特点，也就是说字符串会被原模原样的保存，Redis 不会修改一个比特，所以，String 可以用于存储二进制数据，比如图片、视频。

String 的数据结构是简单动态字符串，类似于 ArrayList，采用预分配冗余空间的方式减少内存的频繁分配。

### 设置和读取操作

> 字符串可以使用双引号 "" 或单引号 '' 括起，甚至不带任何符号。

**set**

设置键值对

```
set key value [EX seconds | PX milliseconds] [NX|XX] [KEEPTTL]
```

EX 和 PX 选项用于指定生存时间，前者单位秒，后者单位毫秒，默认永不过期。

如果 key 原本就有，默认覆盖旧值，NX 指定仅当 key 不存在时设值，XX 指定仅当 key 存在时设值。

KEEPTTL 指定保留生存时间，下次再对这个 key 设值，默认会使用此次设置的生存时间。

**get**

读取指定键的字符串值

```
get key
```

**setnx**

等价 `set ... nx`，仅当 key 不存在时设值。

```
setnx key value
```

**getset**

设置键值对，同时返回旧值，如果有的话。

```
getset key value
```

**批量设置和读取字符串**

同时设置或者读取多个字符串，这里设值不支持 NX、XX、PX、EX 这些选项。

```
mset key value [key value ...]
```

```
mget key [key ...]
```

**数字值的增减**

如果字符串字面量是一个数字，可以使用 incr 和 decr 将其加 1 或减 1。

```
incr|decr key
```

或者，使用 incrby 和 decrby 增减指定数值。

```
incrby|decrby key increment
```

### 其它不常用操作

> 在 Redis 中，索引、下标通常都从 0 开始。

**读子串**

读取字符串指定范围的子串

```
getrange key start end
```

**覆盖**

使用 value 在原字符串 offset 偏移量后进行覆盖

```
setrange key offset value
```

**获取字符串长度**

```
strlen key
```

**字符串尾部追加内容**

```
append key value
```

## 哈希 Hash

### 相关概念和原理

哈希 Hash 是一个键值对集合，它的键只能是字符串，类似于 `Map<String, Object>`。

哈希的数据结构有两种：压缩列表 ZipList、哈希表 HashTable，键值对短且个数少时使用前者，否则选后者。

### 设置和读取操作

**hset**

filed 和 value 成对出现，表示一个键值对，从这可以看出，哈希很适合存储对象数据。

```
hset key field value [field value ...]
```

**hget**

读取指定 key 指定 field 的哈希表的值

```
hget key field
```

**hsetnx**

设置哈希的一个键值对，仅当哈希没有这个 field 时。

```
hsetnx key field value
```

**批量读取 Hash 的键和值**

读取哈希的所有键

```
hkeys key
```

读取哈希的所有值

```
hvals key
```

以 key-value 形式读取哈希的所有键值对

```
hagetall key
```

**判断哈希是否有指定的 field**

如果 filed 存在，返回 1，否则返回 0。

```
hexists key field
```

**删除 Hash 的一个 field**

```
hdel key field
```

## 列表 List

### 相关概念和原理

列表 List 是一个字符串集合，元素允许重复且按照插入顺序排列。列表支持操作两端的元素且性能较好，对中间元素的操作性能较低，类似双向链表 `LinkedList`。列表最多存储 `2^32-1` 个字符串元素。

List 的数据结构是快速链表 QuickList，数据量较小时使用一块连续的内存空间，即压缩列表 ZipList；数据较多时使用多个 ZipList 并组成一个双向链表，即能快速插入，又能节省大量指针空间。

### 设置和读取操作

**lpush 和 rpush**

插入若干个元素到 List 的左端或右端

```
lpush|rpush key element [element ...]
```

**lindex**

从 List 左端开始，读取指定索引位置的元素。

```
lindex key index
```

**弹出元素**

弹出 List 左端、右端元素

```
lpop|rpop key
```

**修改现有元素**

替换指定位置的元素为 element，如果这个位置没有旧值则报错。

```
lset key index element
```

**读区间**

读取区间 [start, stop]  内的元素

```
lrang key start stop
```

**删除指定元素**

当 count > 0 时，从左端开始，删除 count 个 element 元素；当 count < 0 时，从右端开始，删除  -count 个  element 元素。

```
lrem key count element
```

**获取列表长度**

```
llen key
```

## 集合 Set

### 相关概念和原理

集合 Set 是一个字符串集合，元素无序，自动去重，支持快速判断某个元素是否存在。

Set 的数据结构是哈希表，所以添加、删除、查找元素的时间复杂度都是 *O(1)*，类似于 `HashSet<String>`，没有什么需要特别说明。

### 设置和读取操作

**添加多个元素**

```
sadd key member [member ...]
```

**读取所有元素**

```
smembers key
```

**判断元素是否存在**

```
sismember key member
```

**删除指定的多个元素**

```
srem key member [member ...]
```

### 统计操作

**交集**

```
sinter key [key ...]
```

**并集**

```
suion key [key ...]
```

**差集**

以第一个 key 的 Set 为准，统计差集，即前者有而后者没有的元素集合。

```
sdiff key [key ...]
```

## 有序集  Zset

### 相关概念和原理

有序集 Zset 类似 Set，是去重的字符串集合，它的每个元素都关联一个权重 score，元素按照权重升序排列，权重是 Double 类型的值。

Zset 的数据结构有两种：压缩列表、跳跃表。

### 设置和读取操作

**zadd**

```
zadd key [NX|XX] [CH] [INCR] score member [score member ...]
```

NX 指定仅当 key 不存在时设值，XX 指定仅当 key 存在时设值。

CH 返回变更元素的数量，包括新增元素和 score 值变化的元素，默认只返回新增元素数量。

INCR 默认为 0，如果 member 已经存在，则把它的权重增加 INCR，等价 zincrby 命令。

**zrange**

读取 score 位于指定区间的元素，结果按升序排列，WITHSCORES 表示显示元素权重。

```
zrange key start stop [WITHSCORES]
```

**zrevrange**

读取 score 位于指定区间的元素，结果按降序排列。

```
zrevrange key start stop [WITHSCORES]
```

**读取某个元素权重**

```
zscore key member
```

**增加某个元素权重**

```
zincrby key increment member
```

**查看某个元素排名**

```
zrank key member
```

**查看某个元素倒序排名**

```
zrevrank key member
```

**删除若干个元素**

```
zrem key member [member ...]
```

**删除排名在指定区间内的元素**

```
zremrangebyrank key start stop
```

**删除权重在指定区间内的元素。**

```
zremrangebyscore key min max
```

**统计权重在指定区间内的元素个数**

```
zcount key min max
```

## 集合类型的排序

Redis 提供 sort 命令用于对列表、集合、有序集等类型的元素进行排序，这不会影响原始集合。

```
sort key [BY pattern] [LIMIT offset count] [GET pattern] [ASC|DESC] [ALPHA] [STORE dest]
```

key 指定需要进行排序的集合；

ALPHA 表示待排元素包含字母，默认只能排序数字；

ASC 和 DESC 指定升序或降序，默认升序排序；

LIMIT 获取排序结果指定范围的内容，offset 跳过多少个元素，count 返回多少个元素；

STORE dest 表示保存排序结果，键名 dest，默认不保存结果；

BY pattern 指定比较规则，pattern 是一个正则表达式，Redis 会用元素值匹配的部分进行排序，比如，by * 表示使用完整元素值排序；

> 命令 sort 默认按元素值排序，即使 Zset 也是如此，与 score 无关。

GET pattern 首先会以集合的元素为键，读取对应的值，再用这一堆值与 pattern 匹配部分排序，最后返回结果。

# 事务

事务是一堆命令的集合，关系型数据库的事务遵守 ACID 特性，但 Redis 并不如此。

## 命令队列

Redis 把事务的命令组织为一个队列。首先，执行 multi 命令开启一个事务，创建命令队列。然后，按序输入事务命令，这些命令不会立即执行，而是挨个插到队尾。最后，执行 exec 命令，Redis 将按序执行队列所有命令。

```
multi
...
exec
```

如果在添加命令时想要放弃事务，可以执行 discard 命令，Redis 将会清空命令队列并退出事务。

```
multi
...
discard
```

## ACID 特性

Redis 事务具有**隔离性**，Redis 是单线程，任何时刻只能执行一条命令，而且，命令队列执行 exec 的过程中不会被中断，所以，Redis 甚至都不存在两个事务交替执行的情况。

Redis 事务不能保证**原子性**，这与事务的错误处理方式有关：

* 如果输入队列的命令有语法错误，Redis 会检查并报错，但不终止事务，执行 exec 时，Redis 会报错，队列所有命令都不会被执行，符合原子性。
* 如果输入队列的命令有运行时错误，比如给不存在的 key 改名，执行 exec 时，事务照常执行，遇到运行错误命令，报错然后继续执行，这不符合原子性，没有回滚。

既然不能保证原子性，那么**一致性**当然也无法保证。

至于**持久性**，Redis 具有持久化功能，有两种实现方式：

* 把 appendsync 配置属性设为 always，AOF 可以保证持久性，每执行一条命令就会立即被持久化；
* RDB 需要一定条件才能触发持久化，比如 1 分钟内至少写入 100 个键，这就导致数据可能来不及保存。

## 监视键 watch

假如某个事务在运行时会操作某个键，需要考虑这个键是否会在 exec 前被修改，比如被删除。这时，可以在执行 multi 前，使用 watch 命令监视这个键，执行 exec 时将会检查键是否有变化，如果有 exec 将取消执行。

```
watch key [key ...]
multi
...
exec
```

执行 exec 或 discard 自动取消对任何 key 的监视，或者使用 unwatch 命令显式取消所有 key 的监视。

```
unwatch
```

# 持久化

Redis 支持把内存中的数据保存到磁盘，它提供两种持久化方式，分别是 AOF、RDB。

## AOF

### 持久化机制

AOF（Append Only File）是以日志形式记录所有 "写操作" 命令，Redis 服务重启时会读取 AOF 文件并按序执行这些命令从而恢复数据。所谓 "写操作"，包括 set、del、flushdb 等所有修改数据的命令。

当 Redis 启用 AOF 时，每次执行写操作，对应命令就会被写到 AOF 缓冲区，缓冲区根据策略定期把内容同步追加到 AOF 文件。

### 相关配置属性

**启用 AOF**

Redis 默认不启用 AOF 持久化功能，可以通过配置属性 appendonly 开启。

```
appendonly yes
```

**日志文件配置属性**

| 属性           | 说明                                |
| -------------- | ----------------------------------- |
| appendfilename | 日志文件名称，默认 appendonly.aof。 |
| dir            | 日志文件存放目录                    |

**持久化策略**

通过 appendfsync 配置属性指定持久化策略，即什么时候同步 AOF 缓冲区内容，可选如下：

| 选项     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| always   | 每个写操作都会触发 AOF 持久化，可能丢失的数据最少，但严重影响性能。 |
| everysec | 每一秒执行一次 AOF 持久化                                    |
| no       | 让操作系统决定何时持久化，性能最高，两次持久化的间隔可能太长，可能会丢失大量数据。 |

**AOF 文件加载策略**

通过 auto-load-truncated 配属属性指定如果 AOF 文件损坏，Redis 启动时是否还会加载它。

```
auto-load-truncated yes
```

### 重写 AOF 文件

**基本认识**

随着执行过的写操作越来越多，AOF 文件不断扩大，当达到一定程度时文件会被重写，即在不影响数据的前提下压缩文件。可以执行 bgrewriteaof 命令手动触发 AOF 文件的重写。

AOF 文件的重写不会影响原文件，以下是重写过程：

* 首先检查是否有 bgrewriteaof 或 bgsave 正在运行，如果有则等待其结束；
* 服务 fork 一个子进程，它会遍历 Redis 内存数据并写到临时文件；
* 子进程重写期间，是否持久化 AOF 缓冲区的内容到原 AOF 文件？可以选择同步，这能保证即使重写失败日志文件依然完整，但会严重影响 Redis 性能，因为同步过程阻塞。如果不同步，宕机将丢失在此期间的更新。
* 子进程重写完成，父进程把 AOF 缓冲区的数据追加到临时文件，然后临时文件替换 AOF 文件。

**重写策略**

两个与文件重写相关的配置属性：auto-aof-rewrite-percentage 和 auto-aof-rewrite-min-size，仅当这两个条件同时被满足时才能触发 AOF 文件重写。

第一个配置属性表示当前 AOF 文件至少比上次重写时大多少百分比才支持重写，100 就表示至少大一倍。

```
auto-aof-rewrite-percentage 100
```

第二个配置属性表示当前 AOF 文件至少达到多大才支持重写，这个属性用于限制第一次重写。

```
auto-aof-rewrite-min-size 64MB
```

另外，还有一个配置属性 no-appendfsync-on-rewrite，指示 Redis 正在重写时，是否不把期间执行的新的写操作同步到 AOF 文件。yes 是不重写，这能提高重写性能，但可能丢失数据。

```
no-appendfsync-on-rewrite no
```

### 修复 AOF 文件

如果 AOF 文件被损坏，可以使用 Redis 脚本 redis-check-aof 进行修复，这个文件在安装目录。

```
redis-check-aof --fix <AOF文件路径>
```

## RDB

### 持久化机制

RDB（Redis DataBase）是把当前内存中的数据以快照形式存为磁盘文件，Redis 服务重启时会把快照文件加载到内存从而恢复数据。RDB 快照文件是经过压缩的二进制文件。

### 相关配置属性

**持久化策略**

通过 save 配置属性，指定在多少秒内发生了多少次更新就会触发一次 RDB 持久化，可以配置多个 save，它们是或的关系，任何一个被满足都会触发 RDB 持久化。

```
save <seconds> <changes>
```

**关闭 RDB**

Redis 默认启用 RDB，Redis 没有专门提供一个属性负责开启、关闭 RDB 功能，但可以删除所有 save 配置，或添加一个空配置，以达到关闭 RDB 效果。

```
save ""
```

**快照文件的名称和路径**

| 属性       | 说明                          |
| ---------- | ----------------------------- |
| dbfilename | 快照文件名称，默认 dump.rdb。 |
| dir        | 快照文件存放目录              |

**其它配置属性**

| 属性                        | 默认 | 说明                                           |
| --------------------------- | ---- | ---------------------------------------------- |
| rdbcompression              | yes  | 持久化时是否压缩文件                           |
| rdbchecksum                 | yes  | 使用 RDB 快照恢复数据时，是否对文件进行校验    |
| stop-writes-on-bgsave-error | yes  | 执行 bgsave 命令时如果发生错误，是否终止写操作 |

### 手动持久化命令

除了触发 save 条件，Redis 还支持使用命令手动触发 RDB 持久化。

**save**

执行 save 命令，Redis 将把当前内存中的数据写入快照文件，期间暂停执行其它任何命令。

**bgsave**

执行 bgsave 命令，Redis 会 fork 一个子进程，其拥有与父进程完全一样的数据，负责把内存数据写到快照。子进程首先会把数据写到一个临时文件，写完之后再用临时文件替换原文件，期间主线程可以执行其它命令。

> fork 可以理解为拷贝，返回的子进程拥有与父进程一模一样的内容。

因为 bgsave 是后台运行，只会返回 "Background saving started" 信息，想要检查 bgsave 是否成功，可以查看最近一次保存快照的时间，如果与 bgsave 执行时间对应，则说明 bgsave 执行成功。

```
lastsave
```

## 对比与选择

与 RDB 相比，AOF 更容易触发同步，所以它的数据通常更完整，而且 AOF 是追加写入文件，性能更好。快照是经过压缩的二进制文件，比 AOF 文件小的多，使用它恢复数据更快。

某些场景，Redis 服务启动时无法使用 AOF 文件恢复数据，这种错误是一种灾难。

AOF 与 RDB 可以同时运行，Redis 重启时优先选择 AOF 恢复数据，因为它的完整性更高。当然，还可以完全不使用持久化功能，Redis 只作为缓存。

# 集群

单台 Redis 服务器通常不能满足需求，Redis 支持主从复制、哨兵、cluster 三种集群模式，搭建集群，可以提高缓存服务的性能、吞吐量和可用性。

## 主从复制

### 集群机制

#### 主从复制

对于基于主从复制的集群，服务器可以分为主节点 master 和从节点 slave。向 master 写入的数据，会被同步到对应的 slave 节点。这样即使 master 发生问题，也能使用 slave 恢复数据，提高服务的可用性。

数据只会从 master 往 slave 复制，而不会反向同步，slave 连接 master 时会清空自身所有数据，所以 slave 数据只是 master 子集。

执行 info replication 命令查看当前节点主从复制的相关信息，比如当前 master 有多少个 slave 节点，以及这些节点的详细信息，再比如当前 slave 连接的 master 节点的详细信息。

#### 复制延迟

slave 连接 master 后，首先会向 master 请求持久化文件同步数据，此后，master 执行一个更新操作，都会随即发送给 slave，以此保证主从数据一致，但有一定延迟。系统特别繁忙时，延迟问题会很严重，增加 slave 节点同样会加重延迟。

#### 迭代连接

Redis 支持一个节点既作为 master，又充当 slave，这就是中间节点。中间节点可以分担 master 同步压力，起到去中心化的作用，如果中间节点失效，两端的节点不会尝试连接，它们没有任何关系。

#### 读写分离

主从复制集群默认采用读写分离机制，也就是说，master 节点支持读写，而 slave 只能读，如果尝试向 slave 写数据，节点会报错。读写分离可以分担 master 压力，提高服务效率。

在 slave 执行 info replication 命令，返回信息中的 slave_read_only 属性表示当前节点是否只读。可以通过修改配置属性使 slave 可写，但是，salve 的数据可不会同步到 master 节点。

```
slave_read_only no
```

#### 心跳机制

如果主从节点间正在进行数据同步，slave 会以一定频率向 master 发送 REPLCONF 命令，确保连接正常，这就是心跳机制。

在 master 执行 info replication 命令，返回信息中描述 slave 节点的行内，lag 属性表示心跳时间，单位秒，如果 master 发现 slave 失效，就该停止同步。有两个相关配置属性，它们是或的关系，只要有一个满足，master 就会停止正在进行的同步。

* min-slaves-to-write：表示至少有多少个 slave 才能进行主从复制；
* min-slaves-max-lag：表示心跳延迟大于多少秒就不执行主从复制。

```
min-slaves-to-write 3
min-slaves-max-lag 5
```

### 搭建集群

Redis 服务默认都是主节点，执行 slaveof 命令把当前节点作为另一个 Redis 服务的从节点。

```
slaveof <IP> <Port>
```

通过配置属性定义主节点信息，服务启动时默认会作为从节点连接主节点。

```
slaveof <ip> <port>
```

如果主节点设有密码，从节点可以使用 masterauth 配置属性设置。

```
masterauth <password>
```

执行以下命令把从节点转化为主节点

```
slave no one
```

最后，执行 info replication 命令查看主从节点是否连接成功。

## 哨兵模式

### 集群机制

主从复制模式实现了数据备份和读写分离，这能提高服务的可用性和运行效率，但是，如果主节点发生故障，需要手动在从节点进行数据恢复，再重新配置主从关系，在此期间，无法写入数据。哨兵模式可以解决这个问题，它经常与主从复制联合使用。

所谓哨兵 sentine，其实是一个 Redis 节点，但它不与数据打交道，只是负责监控指定的 master 节点，当它发现这个主节点失效，就会执行 "故障自动恢复"，选一个从节点转化为主节点并重新分配主从关系。

哨兵完成故障恢复后，如果原来的主节点重新启动，它会被哨兵转换为新 master 的从节点。

### 搭建集群

#### 修改配置属性

通过配置属性 sentinel monitor 定义监视对象，MasterName 是哨兵为监视对象起的别名，num 后文解释。

```
sentinel monitor <MasterName> <IP> <Port> <num>
```

如果主节点设有密码，哨兵连接时需要有相应信息，使用 sentinel auth-pass 配置属性设置。

```
sentinel auth-pass <MasterName> <password>
```

通过 sentinel failover-timeout 配置属性指定故障恢复超时时间，单位秒。

```
sentinel failover-timeout <MasterName> <seconds>
```

#### 启动哨兵节点

使用 redis-sentinel 程序启动哨兵服务

```
redis-sentinel <配置文件路径>
```

使用 redis-cli 程序连接哨兵，执行 info sentinel 命令返回与哨兵相关的信息。

```
info sentine
```

#### 多个哨兵节点

可以配置多个哨兵监视同一个主节点，前面编写 sentinel monitor 属性有个 num 参数，表示至少需要多少个哨兵同时确认才能认为 master 节点失效。

哨兵如果在一定时间内没有收到 master 正确回应，就认为它失效，以下配置属性定义等待时间，单位毫秒。

```
sentinel down-after-milliseconds <MasterName> <timeout>
```

#### 选择新主节点

如果 master 有多个 slave 节点，哨兵会根据以下规则来选择新的 master 节点：

* 优先选择最高优先级，slave 的优先级可以通过配置属性 replica-priority 设置，值越小越优先。

  ```
  replica-priority <value>
  ```

* 优先选择最大偏移量，偏移量 offset 是 slave 从 master 同步的数据的字节数，越大数据越完整。
* 优选选择最小 runid，这是 Redis 启动时随机生成的 40 位数字。

## cluster 模式

### 集群机制

对于主从复制 + 哨兵模式，每个节点都会缓存所有数据，比较臃肿，而且虽然使用了读写分离，但所有写操作都集中在 master 节点，这给 master 带来很大性能压力。

对于 cluster 集群，每个 key-value 只会保存到一个 master，怎么选择保存节点呢？cluster 有一个槽的概念，默认有 16384 个哈希槽 Hash Slot，这些槽会平均分配给各个节点。添加 key-value 时，先用 CRC16 算法对 key 计算，再把计算结果对 16384 取模，最后得到 key-value 对应的槽的编号，key-value 就存到对应槽位所属的节点。

另外，cluster 还支持主从复制和故障自动恢复，而且 slave 自动被 cluster 管理，所以不需专门设置哨兵节点。更厉害的是，cluster 支持动态扩容，可以在运行期间向集群添加节点，把旧节点的槽重新分配给新节点。

总结，cluster 集群支持主从复制、自动恢复、分布存储、动态扩容，非常强大，这应该是主要的应用方式。

### 搭建集群

#### 启动节点

Redis 默认关闭 cluster 模式，需要通过配置属性显式开启。

```
cluster-enabled yes
```

首次以 cluster 节点身份启动时，Redis 会生成一个配置文件，其中记录着这个节点和集群其它节点的信息以及关联方式，这个配置文件的名称需要通过配置属性显式设置。

```
cluster-config-file <ConfName>
```

现在，就可以使用 redis-server 程序启动所有的 cluster 节点，包括 master 和 slave。

执行 cluster info 命令查看节点在 cluster 集群的相关信息，比如连接状态。现在任意节点执行该命令，可以发现它们都处于 fail 状态，因为节点还未进行集群配置，仍处于独立状态。

执行 cluster node 命令查看节点所在 cluster 集群的所有节点信息，返回的每一行描述一个节点，包括 IP、端口号、主从关系等。对于状态 OK 的节点，行的第一个长字符串是 node-id，这是节点在 cluster 的唯一标识；对于 master 节点，行的尾部会描述其槽的区间；对于 slave 节点，行中间会显示其 master 的 node-id。

#### 连接节点

首先要打破隔绝，使各个节点发现并连接彼此，这一步使用 redis-cli 程序的 cluster meet 功能。

```
redis-cli -h <host> -p <Port> cluster meet <IP> <Port>
```

这个程序会登录 -h -p 指定的 Redis 服务，使它与 IP Port 指定的节点连接，组成 cluster 集群。如果节点 A 分别与 B 和 C 进行 cluster meet，那么 B 和 C 自然处于同一个 cluster 集群，它们会自动连接。

执行 cluster info 命令查看集群信息，发现各个节点仍处于 fail 状态，因为还没有为它们分配槽位。

#### 分配槽位

前面提到，cluster 根据槽来选择 master 节点存储键值对，没有槽的 master 不会存储任何数据，所以它的状态只能是 fail。注意，并非所有 cluster 节点都有槽位，slave 肯定没有，master 不主动分配也不会有。

使用 redis-cli 程序的 cluster addslots 功能来分配槽位

```
redis-cli -h <host> -p <Port> cluster addslots <id>
```

这个程序一次只为一个节点分配一个槽，id 是槽的编号，cluster 集群可有 16384 个槽呀！可以使用以下 bash 命令批量执行程序，它把 [start, end] 区间的槽分配给指定节点。注意，槽的编号基于 0。

```
redis-cli -h <host> -p <Port> cluster addslots $(seq start end)
```

执行 cluster info 命令查看集群信息，cluster_slots_assigned 属性表示已分配的槽的数量，cluster_slots_ok 属性表示可以写入的槽的数量，所有节点的状态已经变为 OK。

#### 配置主从

最后，为 master 配置 slave 节点，这一步需要登陆 salve 节点，执行以下命令，它使当前节点在 cluster 集群中作为指定节点的从节点。

```
cluster replicate <主节点node-id>
```

至此，成功搭建 cluster 集群。

#### 读写透明

现在可向 cluster 集群读写数据，redis-cli 程序启动时需要添加 -c 参数，因为集群数据分布在多个节点，而客户端只能连接一个节点，正常状态无法向其它节点读写数据，参数 -c 指示节点自动转发命令给正确节点并返回结果。

现在，无论登录哪个 cluster 节点，即使是 slave，都可以向整个集群读写数据。

```
redis-cli -c -h <host> -p <Port>
```

#### 再分配槽

在 cluster 集群运行期间，可以对已分配的槽进行再分配，通过 redis-cli 程序的 cluster reshard 功能完成，格式如下。IP 和 Port 指定由哪个节点执行再分配，可选择任意一个空闲节点；cluster-from 指定源，cluster-to 指定目标，cluster-slots 指定为目标分配的槽的数量。

```
redis-cli --cluster reshard <IP>:<Port> 
	--cluster-from <node-id> [,<node-id>,<node-id>...] 
	--cluster-to <node-id>
	--cluster-slots <num>
```

使用 reshard 进行动态扩容时，需要先按照前面步骤添加新节点到集群，然后再分配槽位。

再分配时，迁移槽位和数据期间，这些数据都不可用，可能会短暂出现缓存失效的情况。

### 其它配置属性

**cluster-require-full-coverage**

当有节点失效时，cluster 集群是否不提供写服务。如果 yes，有节点失效的话就只能读数据

**cluster-node-timeout**

节点的最长不可用时间，单位毫秒。当某个 cluster 节点失效时间超过指定值，将使用它的从节点进行故障恢复。

**cluster-migration-barrier**

主节点的最小从节点数。如果某个 master 的 slave 数少于指定值，cluster 将会自动从其它有富于的 master 调剂从节点过来。合理设置这个值，可以提升集群的可用性。

# 线程模型

# Java 标准客户端 Jedis

Jedis 是 Redis 官方推荐的 Java 客户端开发包，Java 程序可以使用 Jedis 连接 Redis 并执行各种操作。

## 引入 Maven 依赖

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>4.2.3</version>
</dependency>
```

## 获取 Redis 连接

使用 Jedis 获取 Redis 的连接有两种方式：直接连接和连接池，它们的区别和优劣已是老生常谈，不必多说。

下例使用 Jedis 类直接连接 Redis

```
public static void test_connect_redis() {
    Jedis jedis = new Jedis(IP, Port);
    // response PONG if connect successfully
    Assertions.assertEquals("PONG", jedis.ping());
    jedis.close();
}
```

下例使用 JedisPool 连接池与 Redis 通信

```
public void test_connect_pool() {
    JedisPool jedisPool = new JedisPool(IP, Port);
    Jedis jedis = jedisPool.getResource();
    // response PONG if connect successfully
    Assertions.assertEquals("PONG", jedis.ping());
    jedisPool.close();
}
```

连接池 JedisPool 有几个核心参数，下表列出。

| 属性               | 说明                                                       |
| ------------------ | ---------------------------------------------------------- |
| maxTotal           | 最大连接数                                                 |
| minIdel            | 最少空闲连接数                                             |
| maxIdle            | 最大空闲连接数                                             |
| blockWhenExhausted | 连接数最大时，是否阻塞新申请连接的线程，不阻塞则抛出异常。 |
| maxWait            | 阻塞时间，超时抛出异常。                                   |

## 执行各种命令

Jedis 拥有各种执行 Redis 命令的方法，它们的名字类似，下面示范各种操作。

```
public void test_readAndWrite_string() {
    jedis.set("key", "readAndWrite");
    Assertions.assertEquals("readAndWrite", jedis.get("key"));
}

public void test_del_key() {
    jedis.set("key", "del_key");
    Assertions.assertNotNull(jedis.get("key"));
    jedis.del("key");
    Assertions.assertNull(jedis.get("key"));
}

public void test_exist_key() {
    jedis.set("key", "exist_key");
    Assertions.assertTrue(jedis.exists("key"));
    jedis.del("key");
    Assertions.assertFalse(jedis.exists("key"));
}

public void test_multi() {
    jedis.del("key");
    Assertions.assertNull(jedis.get("key"));

    // can't use jedis in transtation
    Transaction multi = jedis.multi();
    multi.set("key", "multi");
    multi.discard();
    Assertions.assertNull(jedis.get("key"));

    multi = jedis.multi();
    multi.set("key", "multi");
    multi.exec();
    Assertions.assertNotNull(jedis.get("key"));
}
```

## 管道批量执行

前面 Jedis 没调用一次方法执行 Redis 命令都要与 Redis 进行一次通信，如果有大量操作，这样做会带来大量通信消耗，且效率很低。Jedis 支持批量发送请求和获取结果，即一次通信执行多个操作。

```
public void test_pipeline() {
    Pipeline pipelined = jedis.pipelined();
    pipelined.set("key", "pipeline");
    // send all commands until execute sync method
    pipelined.sync();
}
```

# SpringBoot 整合 Redis

Redis 作为重要的缓存中间件，SpringBoot 自然对其提供集成，以下是 Spring 提供的 starter 包。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

引入依赖并配置属性后，就可以在任何需要操作 Redis 的地方，注入 RedisTemplate 组件。RedisTemplate 类似于 Jedis，用于发送请求和接收结果，但它们的 API 完全不同，使用方式也有差异。

```
@Autowired
private RedisTemplate redisTemplate;

public void readAndWrite() {
	redisTemplate.opsForValue().set("key", "readAndWrite");
	Assertions.assertEquals("readAndWrite", redisTemplate.opsForValue().get("key"));
}
```

这里详细说明 SpringBoot 整合 Redis 时的配置，对于单服务器，直接配置 IP、Port 等属性即可。

```
spring:
  redis:
    host: 192.168.200.101
    port: 6379
```

整合哨兵集群，使用 spring.redis.sentinel.nodes 配置所有哨兵，spring.redis.sentinel.master 指定监视的主节点，它的值是哨兵为主节点起的别名，可在哨兵配置文件的 sentinel monitor 属性处查看。

```
spring:
  reids:
    sentinel:
      master: master's alias
      nodes: IP:Port[,IP:Port,...]
```

整合 cluster 集群，只需配置 spring.redis.cluster.nodes 属性，指定所有 master 节点。

```
spring:
  reids:
    cluster:
      nodes: IP:Port[,IP:Port,...]
```

当然，spring.redis 还有许多可配置的属性，暂不深究。

# Redis 实战和应用问题

## 基本应用

在高并发场景，如果读写数据的请求都集中到 MySQL 等关系型数据库，可能会瞬间压垮它。因为关系型数据库通常会把数据持久化到磁盘，巨量请求可能导致大量磁盘 IO；如果有 order by、limit 等语句，数据库还要进行额外处理，这很耗费 CPU 资源；数据库支持的最大连接数有限，请求不一定都能成功执行。

此时，引入 Redis，把频繁访问的数据存入缓存，每次请求数据时，先尝试从 Redis 获取，没有得到结果时再访问数据库。初始时，Redis 为空，当某个数据初次被访问时，程序在 Redis 找不到，就向数据库请求，获得结果后先回设到缓存，然后才返回给用户。在此之后，相同数据就可以直接从缓存读取，而不用麻烦数据库。

由于 Redis 的速度非常快，它可以有效缓解数据库的压力，提升程序的响应速度。

## 缓存穿透

前面提到，初次访问的数据，先从数据库获取，然后回设缓存。但如果这个数据根本就不存在呢？按照前面的逻辑不会回设缓存，那么之后所有访问这个数据的请求还是会打到数据库，缓存层相当于没有，这就是缓存穿透。

缓存穿透的原因是不存在的数据没有被缓存，那么我们就把它们缓存起来。访问数据库时，如果返回 null，即这个数据不存在，就对其缓存 null、0 等空值 。

布隆过滤器。（TODO）

## 内存溢出

Redis 的所有数据都运行在内存，但内存是有限的，如果不为数据设置过期时间，很快就会使用完，造成 OOM 异常。

过期时间要根据应用场景而定，而且应该是一个范围内的随机值。如果过期时间固定，那么同时回设的缓存会同时失效，如果此时收到大量访问这些数据的请求，它们都会打到数据库，造成缓存雪崩，压垮数据库。

## 缓存雪崩

缓存雪崩类似于缓存击穿，都是由缓存失效引起，但它是批量数据同时过期，造成访问这些数据的大量并发请求打到数据库，从而压垮它。

设置随机过期时间，使缓存分散失效，可以避免雪崩；设置多级缓存，比如 Nginx+Redis；如果不要求高并发处理能力，可以考虑使用锁、队列保证不会有大量请求同时访问数据库。

## 缓存击穿

如果某个数据在失效后，突然收到大量访问它的请求，那这些请求都会打到数据库，造成缓存击穿，压垮数据库。这种情况很常见，比如新热点数据就会突然被高并发访问。

缓存击穿的原因是大量访问 "同一个数据" 的请求在缓存失效时同时发起，它的数据真实存在，而缓存穿透的数据不存在，缓存雪崩是访问多个数据。

可以使用互斥锁解决这个问题，具体依靠 setnx 命令实现。当这些并发请求发现缓存失效，先不访问数据库，而是使用 setnx 命令设置值，相当于上锁，设置成功的请求获得锁，允许访问数据库，它获得数据后回设缓存并使用 del 命令释放锁。设置失败说明已有一个线程尝试访问数据库，那就等待一段时间，然后回到第一步查看缓存。

## 缓存一致性

缓存数据必须与数据库相同才有意义，如果数据库发生更新，就必须保证缓存同步更新，或被删除。

注意，因为写数据库和写缓存都是远程操作，为防止大事务造成死锁问题，代码通常不把它们放在同一个事务。

### 先写缓存，再写数据库

这种方案最简单，问题也最严重。如果更新缓存后，写数据库失败，那缓存的是假数据。

### 先写数据库，再写缓存

如果更新数据库后，写缓存失败，那缓存的是旧数据。

这种方案在并发场景也有问题，假设有两个写数据请求 A 和 B：

* A 写数据库成功，然后因为网络卡顿一会；
* B 写数据库成功，更新缓存成功；
* A 继续更新缓存，覆盖 B 的缓存。

此时，数据库保存着 B 的内容，而缓存的却是 A 的内容。

### 先删缓存，再写数据库

**并发场景**

这种方案在并发场景有问题，假设有读数据请求 A 和写数据请求 B：

* 先执行 B，删除缓存，然后因为网络卡顿一会；
* 此时，执行 A，查缓存没有结果，再查数据库，回设缓存并返回；
* B 继续执行，写数据库，覆盖旧值。

此时，数据库保存着 B 的新值，而缓存的却是 A 回设的旧值。

**缓存双删**

上面问题的原因是删除缓存和写数据库之间，有其它请求把旧数据回设到缓存。

考虑在写数据库完成后，再执行一次删除缓存，这个操作不要马上执行，应该等待一段时间，以保证所有回设旧值到缓存的操作都执行完毕。

### 先写数据库，再删缓存

这个方案较好，在并发场景，假设有读数据请求 A 和写数据请求 B：

* B 先写数据库，然后因为网络卡顿一会；
* A 查询数据，从缓存获取旧值；
* B 继续执行，删除缓存。

这种情况，虽然 A 读取到的是旧值，但缓存和数据库保持一致。

但它并不完美，在并发场景也有出现问题的可能，假设有读数据请求 A 和写数据请求 B：

* 缓存数据过期；
* A 查缓存没有结果，继续查数据库，获得数据，然后因为网络卡顿一会；
* B 写数据库，并删除缓存；
* A 继续执行，回设前面查到的旧值到缓存。

此时，数据库保存着新值，而缓存的却是旧值。但它发生的概率很小，因为需要同时满足缓存失效和读操作比写操作更耗时两个条件。

由此可见，很难 100% 保证缓存一致，但当前方案出现问题的概率最小。

### 删除缓存失败重试

先写数据库再删缓存，和缓存双删，都有一个风险：删除缓存失败。可以增加重试机制保证成功删除缓存，它有多种实现方案。

**消息队列**

删除失败时，生产一个消息到队列，订阅线程读到消息，根据内容删除缓存。

**定时任务**

删除失败时，把相关信息存入数据库，开启一个定时任务线程，每隔一段时间就查表，根据内容删除缓存。

**订阅 binlog**

开启一个线程订阅 MySQL 的 binlog 日志，如果发现数据更新，根据内容删除缓存。这种方案，业务逻辑在写数据库后可以直接返回。

# Docker 搭建

Redis 容器通常都没有 `redis.conf`  文件，需要从其它地方获取并做文件挂载。

拉取镜像

```
docker pull redis:6.2.6
```

创建挂载目录

```
mkdir -p /data/docker/redis/data
```

将 `redis.conf` 文件放在母机的 `/data/docker/redis/conf` 目录。

修改配置属性，`daemonize` 属性必须设置为 `no`，因为容器必须有一个前台进程才能留存。

创建容器

```
docker run -d --name redis -p 6379:6379 \
-v /data/docker/redis/redis.conf:/redis/redis.conf \
-v /data/docker/redis/data:/data \
redis:6.2.6 \
redis-server /redis/redis.conf --appendonly yes

#####说明
redis-server /etc/redis/redis.conf 表示启动容器后使用指定的配置文件启动Redis
--appendonly yes 开启持久化
如果母机没有创建redis.conf文件，那么docker会将redis.conf创建为空目录
```

启动 Redis 客户端

```
docker exec -it redis redis-cli
```

查看持久化文件

```
ls /data/docker/redis/data
```
