# Redis

# 初识 Redis

## Redis 是 NoSQL

NoSQL，即 Not Only SQL，泛指非关系型数据库。传统数据库依赖业务逻辑，使用表存储数据，具有严格的约束和定义。Redis 是一种 NoSQL，使用简单的 key-value 结构保存数据，不遵循 SQL 标准，不支持 ACID。而且，它的所有数据都运行在内存，没有磁盘 IO。因此，Redis 的速度远远快于关系型数据库。

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

现在开始安装 Redis，这里使用官网的压缩包。

```
tar -zxvf <压缩文件> [-C 解压目录]
```

在解压目录，执行编译。

```
make
```

如果编译发送错误，执行以下命令进行恢复。

```
make distclean
```

在解压目录，执行安装。make test 是新版命令。destdir 指定安装目录，默认为 */usr/local/bin*。

```
make install|test [destdir=安装目录]
```

## Redis 的启动和连接

Redis 安装目录通常位于 */usr/local/bin/*，其中存放许多可执行文件，启动或连接 Redis 都是使用这些程序。

### 启动服务器

redis-server 程序用于启动 Redis 服务器，有两种启动方式。

**前台启动**

直接执行 redis-server 程序，当前命令窗口将进入 Redis 界面，退出界面会关闭 Redis 服务。

**后台运行**

需要使用配置文件，首先修改配置属性，设置后台启动。

```
daemonize yes
```

执行 redis-server 程序，添加配置文件路径参数。

```
redis-server <配置文件路径>
```

此时，命令窗口没有变化，但 Redis 服务器已在后台运行，可以使用 ps 查看进程。

```
ps -aux | grep redis
```

### 客户端连接

redis-cli 是 Redis 提供的用于连接服务器的客户端程序，它的使用格式如下。-h 指定 Redis 主机地址，-p 指定 Redis 服务端口，-a 指定 Redis 服务的密码。

```
redis-cli [-h host] -p <port> [-a password]
```

redis-cli 还能请求关闭 Redis 服务，有两种方式：连接服务器，执行 shutdown 命令；或者，在 shell 窗口执行以下命令。

```
redis-cli -p <port> shutdown
```

## 配置属性和配置文件

### 基本概念

类似 MySQL，Redis 也暴露有许多属性用于调控其运行方式，而且也支持配置文件。通常，Redis 压缩包本身带有一个标准配置文件，名为 redis.conf。

在客户端程序，可以使用 config 命令设置、查看配置属性。

```
config get <属性名匹配模式>
```

```
config set <属性名> <属性值>
```

配置文件可以永久性的设置 Redis 运行方式，它的格式如下：

```
属性名 属性值
```

配置文件支持引用其它文件的内容，如果一个属性多次设置，总是最后一个生效，且本地配置的优先级最高。

```
# include <其它配置文件路径>
```

如果 Redis 使用配置文件启动，可以使用 config 命令把运行时修改的配置属性持久化到配置文件。

```
config rewrite
```

### 常用属性

| 属性             | 默认       | 描述                                                         |
| ---------------- | ---------- | ------------------------------------------------------------ |
| port             | 6379       | Redis 服务暴露的端口号。                                     |
| bind             | 127.0.0.1  | 限制 Redis 只接受指定主机的客户端连接。                      |
| protected-mode   | yes        | 限制 Redis 只接受本机的客户端连接。                          |
| requirepass      | ""         | 连接密码，空字符串表示没有。                                 |
| maxclients       | 10000      | 最大连接数，达到上限后将拒绝新的连接。                       |
| timeout          | 0          | 空闲连接存活时间，0 表示永远存在。                           |
| tcp-keepalive    | 300        | 心跳检测时间间隔，单位秒，0 表示不检测。                     |
| dir              | ./         | 各种数据的保存目录。                                         |
| logfile          | ""         | 日志文件名，空字符串表示不输出日志。                         |
| loglevel         | notice     | 日志记录级别，可选 debug，verbose，notice，warning           |
| maxmemory        | 0          | 最大内存限制，0 表示不限制。当 Redis 使用的内存达到最大，将根据设定的策略移除某些数据，以留出空间。 |
| maxmemory-policy | noeviction | 内存达到限制，移除数据的策略，有多个可选：<br />volatile-lru，最近最少使用，对设置生存时间的 key 有效；<br />allkeys-lru：最近最少使用，对所有 key 有效；<br />volatile-random：随机移除，对设置生存时间的 key 有效；<br />allkeys-random：随机移除，对所有 key 有效；<br />volatile-ttl：移除将要过期的 key；<br />noeviction：不移除，对申请内存的操作返回错误信息。 |

## 数据库

Redis 本质是一个 DBMS 软件，它虽然没有表结构，但管理有多个数据库。数据库就是键值对的集合，各个库相互独立。Redis 默认有 16 个数据库，客户端初始连接第 0 个库。

**通过配置修改数据库的数量**

配置属性 databases 用于设定数据库数量，执行 config get databases 能查看该属性。

使用 config set 命令，或修改 Redis 配置文件，以自定义数据库的数量。

**查看当前数据库**

执行 client list 命令，返回结果的 db 属性表示当前连接的数据库的序号。

**更换数据库**

number 是目标数据库的序号。

```
select number
```

**清空当前数据库**

```
flushdb
```

**清空所有数据库**

```
flushall
```

## 键

### 基本操作

**设置**

键必须与值同时设置，否则没有意义，值的设置在基本数据类型章节介绍。

**删除**

删除若干个键，这些键对应的值也会被一并删除。

```
del key [key ...]
```

**判断存在**

如果指定的键存在，返回 1，否则返回 0。

```
exists key
```

**查看当前库键的数量**

```
dbsize
```

**查看键对应的值的类型**

```
type k1
```

**重命名键**

```
rename|renamenx key newkey
```

如果 newkey 已经存在，rename 会覆盖旧值，而 renamenx 返回 0，不执行修改。如果 key 不存在，报错。

### 键的查找

#### keys

keys 命令使用正则表达式查找名字匹配的键。

```
keys pattern
```

> ? 匹配单个字符，* 匹配零或若干个字符。

#### scan

```
scan cursor [MATCH pattern] [COUNT count]
```

cursor 指定从第几个游标位置开始查找。MATCH 指定键名的匹配模式，默认查找所有。COUNT 指定遍历的游标数量，也就是会检查多少个键。

scan 的返回结果分为两组，第一组只有一个元素，表示此次查找结束位置的下一个位置，可作为下次 scan 的起始位置，返回 0 表示搜索完所有键；第二组的元素是此次查找匹配到的键。

> 游标用于记录数据的迭代位置，我理解为数据的索引。

> **keys 和 scan 的区别**
>
> keys 命令以阻塞方式运行，当待查找键的数量较多时比较耗时。而 Redis 是单线程，这段时间就不能执行其它命令。
>
> scan 以非阻塞的方式运行，而且，它还能使用 COUNT 限制搜索的键的数量，所以它的运行效率比较高。

### 生存时间

键一旦设置就会一直存在，如果键设有生存时间，它在超时后会被删除。

**设置生存时间**

expire 和 pexpire 命令设置键的生存时间，前者单位秒，后者毫秒。它们会覆盖已设的生存时间。

```
expire key seconds
```

```
pexpire key milliseconds
```

**查看生存时间**

ttl 和 pttl 命令查看指定键的剩余生存时间，前者单位秒，后者毫秒。如果键不存在，返回 -2；如果键没有设置生存时间，即永远存在，返回 -1。

```
ttl|pttl key
```

**删除键的生存时间**

键的生存时间被删除，该键就会永远存在。

```
persist key
```

# 基本数据类型

Redis 的数据都是 key-value 形式，数据类型指的是 value。键的类型默认是 String，用户不能指定。本节介绍的操作命令都是运行在 redis-cli 客户端程序。

## 字符串 String

### 相关概念和原理

字符串 String 是 Redis 最基本的数据类型，最多能存储 512 MB 的内容。

String 具有二进制安全功能，也就是说字符串会被原模原样的保存，Redis 不会对其进行任何处理。这样，String 就可以用于存储任何数据，比如图片。

String 的数据结构是简单动态字符串，类似于 ArrayList，采用预分配冗余空间的方式减少内存的频繁分配。

### 设置和读取操作

> 字符串字面量可以使用双引号 "" 或单引号 '' 括起，或者不带任何符号。

**set**

```
set key value [EX seconds | PX milliseconds] [NX|XX] [KEEPTTL]
```

键值对默认永远存在，EX 和 PX 选项用于自定义生存时间，前者单位秒，后者单位毫秒。

如果 key 原本就存在，默认覆盖旧值。NX 限制仅当 key 不存在时进行设置，XX 限制仅当 key 存在时进行设置。

KEEPTTL 表示保留生存时间，下次再设置这个 key 时，此次设置的生存时间依然生效。

**get**

设置字符串以后，使用 get 命令读取指定 key 的字符串。

```
get key
```

**setnx**

setnx 命令与 `set ... nx` 等价，限制仅当 key 不存在时进行设置

```
setnx key value
```

**getset**

getset 命令在设置值的同时返回这个 key 的旧值。

```
getset key value
```

**批量设置和读取字符串**

mest 和 mget 命令分别能同时设置和读取多个字符串。注意，mset 和 mget 不支持 NX、XX、PX、EX 等参数，添加这些参数可能不会报错，但也不生效。

```
mset key value [key value ...]
```

```
mget key [key ...]
```

**数字值的增减**

如果字符串存储的是一个数字，可以使用 incr 和 decr 命令对其增减 1

```
incr|decr key
```

或使用 incrby 和 decrby 命令对其增减指定量

```
incrby|decrby key increment
```

### 其它不常用操作

> 在 Redis 中，大部分索引、下标，都是从 0 开始。

**读子串**

读取字符串指定范围的子串。

```
getrange key start end
```

**覆盖**

使用 value 在字符串 offset 偏移量后进行覆盖。

```
setrange key offset value
```

**获取字符串长度**

```
strlen key
```

**在字符串尾部追加内容**

```
append key value
```

## 哈希 Hash

### 相关概念和原理

哈希 Hash 是一个键值对集合，它的键只能是字符串，类似于 `Map<String, Object>`。

Hash 有两种数据结构：压缩列表 ZipList、哈希表 HashTable。键值对长度短且个数少时，使用 ZipList，否则使用 HashTable。

### 设置和读取操作

**hset**

```
hset key field value [field value ...]
```

filed 和 value 表示哈希的一个键值对。从这可以看出，哈希很适合存储对象数据。

**hget**

通过 key 和 field，读取哈希的一个值。

```
hget key field
```

**hsetnx**

设置哈希的一个键值对，仅当哈希没有这个 field 时进行设置。

```
hsetnx key field value
```

**批量读取 Hash 的键和值**

读取哈希的所有键。

```
hkeys key
```

读取哈希的所有值。

```
hvals key
```

以 key-value 的方式读取哈希的所有键值对。

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

列表 List 的元素都是字符串，按照插入顺序排列，元素允许重复。它可以操作两端的元素且性能较好，对中间元素的操作性能较低，类似于双向链表 `LinkedList`，。

列表最多存储 `2^32-1` 个字符串。

List 的数据结构是快速链表 QuickList。数据较少时使用一块连续的内存空间，即压缩列表 ZipList。数据较多时，使用多个 ZipList 并组成一个双向链表，即满足快速插入，又能节省大量的指针空间。

### 设置和读取操作

**lpush 和 rpush**

lpush 和 rpush 命令分别向 List 的左端、右端插入若干个元素。

```
lpush|rpush key element [element ...]
```

**lindex**

从 List 的左端开始，读取指定索引位置的元素。

```
lindex key index
```

**弹出元素**

lpop 和 rpop 命令分别弹出 List 左端、右端元素。

```
lpop|rpop key
```

**修改现有元素**

把指定位置的元素替换为 element，如果这个位置没有旧值，报错。

```
lset key index element
```

**读区间**

读取区间 [start, stop] 内的元素。

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

集合 Set 的元素都是字符串，无序，自动去重，支持快速判断某个元素是否存在。

Set 的数据结构是哈希表，所以添加、删除、查找元素的时间复杂度都是 *O(1)*。它类似于 `HashSet<String>`，没有什么需要特别说明。

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

以第一个 key 指定的 Set 为准，统计差集，即前者有而后者没有的元素集合。

```
sdiff key [key ...]
```

## 有序集  Zset

### 相关概念和原理

有序集 Zset 是去重的字符串集合。它的每个元素都关联一个权重 score，元素根据权重按升序排列。权重是 Double 类型的值。

Zset 使用两种数据结构：

* 哈希表：关联 value 和 score，保证 value 的唯一性，可通过 value 找到对应的 score。
* 跳跃表：它的作用是排序，还支持根据 score 范围访问元素。

### 设置和读取操作

**zadd**

```
zadd key [NX|XX] [CH] [INCR] score member [score member ...]
```

NX 限制仅当 key 不存在时进行设置，XX 限制仅当 key 存在时进行设置。

CH（TODO：unknow）

INCR 默认 0。如果 member 已经存在，则把它的权重增加 INCR。

**zrange**

读取权重在指定区间内的元素，按升序返回。WITHSCORES 选项表示同时显示元素的权重。

```
zrange key start stop [WITHSCORES]
```

**zrevrange**

读取权重在指定区间的元素，按降序返回。

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

Redis 提供 sort 命令以对列表、集合、有序集等类型的元素进行排序。注意，排序不影响原始数据。

```
sort key [BY pattern] [LIMIT offset count] [GET pattern] [ASC|DESC] [ALPHA] [STORE dest]
```

key 指定需要进行排序的值对应的键。

sort 默认只能对数字进行排序，如果待排元素有字母，需要标注 ALPHA。

sort 默认按升序排序，ASC 和 DESC 分别指定按升序或降序进行排序。

LIMIT 限制排序后返回的元素数量，offset 指定跳过的元素数量，count 指定返回的元素数量。

STORE dest 可以把排序结果进行保存，dest 指定保存的键名。排序结果默认是临时的。

> sort 是根据元素值进行排序，即使 Zset 也是如此，与权重 score 无关。

sort 默认按元素值进行排序，BY pattern 可以指定排序规则。pattern 是一个正则表达式，Redis 把元素值与它进行匹配，然后使用通配符匹配的内容进行排序。比如，by * 使用完整的元素值进行排序。

GET pattern 较复杂，它首先以集合的元素为 key，读取对应的 value。然后，把这些value 与 pattern 进行匹配，最后使用通配符匹配的部分进行排序并返回。

# 事务

在数据库层面，事务就是一组命令。关系型数据库的事务遵守 ACID 特性，但 Redis 并非如此。

## 命令队列

Redis 使用队列组织事务的所有命令。首先，执行 multi 命令以开启一个事务。

之后输入的每一行命令不会立即执行，而是被插入到队尾。当执行 exec 命令时，Redis 会按插入顺序执行队列的命令。

```
multi
...
exec
```

如果在添加命令时想要放弃事务，可以执行 discard 命令，此时将清空命令队列并退出事务。

```
multi
...
discard
```

## ACID 特性

Redis 事务具有隔离性，因为 Redis 是单线程，任何时刻只能运行一条命令。而且，命令队列在 exec 后不会被中断。所以，甚至都不存在两个事务交替运行的情况。

Redis 事务不能保证原子性，这与事务的错误处理策略有关：

* 如果输入队列的命令有语法错误，Redis 会检查并报错，但不终止事务。执行 exec 后，Redis 会报错，事务的任何命令都不会被执行。这符合原子性。
* 如果输入的命令有运行时错误，比如给不存在的 key 改名，执行 exec 后，事务照常执行，对于运行错误的命令，报错然后继续执行下一个命令。这不符合原子性，没有回滚操作。

既然不能保证原子性，那么一致性自然也无法保证。

至于持久性，Redis 具有持久化功能，它有两种实现方式：

* 把 appendsync 配置属性修改为 always 时，AOF 可以保证持久性，此时每一条命令，都会立即被持久化。
* RDB 需要一定条件才能触发持久化，比如 1 分钟内至少写入 100 个键，这就导致数据可能来不及保存。

## 监视键 watch

如果某个事务在运行时要操作某个键，应该考虑到这个键是否会在 exec 之前被修改，比如被删除。此时，可以在 multi 前，使用 watch 命令监视这些键，如果事务在 exec 时发现这个键已被修改，这个事务将不能正确执行。

```
watch key [key ...]
multi
...
exec
```

> 如果 watch 与事务在同一个连接，并且监视的 key 在事务内被修改，那这个事务正常执行。

exec 或 discard 会自动取消对所有 key 的监视，也可以执行 unwatch 命令显式取消所有 key 的监视。

```
unwatch
```

# 持久化

Redis 支持把内存中的数据保存到磁盘，它提供两种持久化方式，分别是 AOF、RDB。

## AOF

### 持久化机制

AOF，Append Only File 以日志的方式记录所有 "写操作" 命令，并在服务器重启时重新执行这些命令以恢复数据。这个 "写操作" 也包括 del、flushdb 等命令。

当 Redis 启用 AOF 时，每当执行写操作，这个命令就会被记录到 AOF 缓冲区，缓冲区根据配置策略定期同步追加到 AOF 文件。

随着执行过的写操作越来越多，日志文件逐渐变大，达到一定程度时这个文件会被重写，即在不影响持久化结果的前提下压缩文件。可以执行以下命令手动触发 AOF 文件的重写：

```
bgrewriteaof
```

重写通过读取当前缓存的数据实现，与原 AOF 文件无关。以下是重写流程：

* 首先检查当前是否有 bgrewriteaof 或 bgsave 正在运行，如果有，则等待它结束。
* Redis fork 一个子进程，它负责 AOF 文件的重写，这个子进程会遍历内存中的数据并写到临时文件。
* 子进程重写期间，Redis 会把新的写操作记录到 AOF 缓冲区，并按策略选择是否追加到原 AOF 文件。选择追加，这能保证即使重写失败，现有 AOF 文件依然完整，但会严重影响 Redis 性能，因为 Redis 在追加时可能会阻塞。选择不追加，在宕机时会丢失此期间的数据。
* 子进程重写完成，父进程将把 AOF 缓冲区的数据追加到临时文件。最后，使用临时文件替换 AOF 文件。

Redis 服务器重启时，会加载磁盘上的这个 AOF 日志文件，按序执行命令以实现数据恢复。

### 相关配置属性

**开启 AOF**

Redis 服务器默认不开启 AOF 持久化功能，可以修改配置属性 appendonly 开启它。

```
appendonly yes
```

**日志文件的名称和路径**

| 属性           | 说明                                |
| -------------- | ----------------------------------- |
| appendfilename | 日志文件名称，默认 appendonly.aof。 |
| dir            | 日志文件目录                        |

**持久化策略**

通过 appendfsync 配置属性，设置持久化的策略，即什么时候把 AOF 缓存同步到 AOF 文件。它有 3 个选项，如下表所示。

| 选项     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| always   | 每次写操作都会触发 AOF 持久化，这样可能丢失的数据最少，但严重影响性能。 |
| everysec | 每一秒都执行一次 AOF 持久化。                                |
| no       | 交由操作系统决定何时进行持久化，这种方式性能最高，但可能每次持久化之间的间隔过长，可能会丢失大量数据。 |

**AOF 文件加载策略**

通过 auto-load-truncated 配属属性，设置 AOF 文件的加载策略，即如果 AOF 文件损坏，Redis 服务器启动时是否还会加载它。

```
auto-load-truncated yes
```

**AOF 文件重写策略**

有两个与重写策略相关的配置属性：auto-aof-rewrite-percentage 和 auto-aof-rewrite-min-size，它们是 and 的关系，必须同时满足，才能触发重写。

表示当前 AOF 文件至少比上次重写时的文件大多少个百分比才能触发重写，100 就是指比上次大一倍才能触发。

```
auto-aof-rewrite-percentage 100
```

表示当前 AOF 文件至少达到指定大小才能触发重写。

```
auto-aof-rewrite-min-size 64MB
```

此外，还有一个配置属性 no-appendfsync-on-rewrite，它指示当 Redis 正在重写，是否不把此刻执行的新的写操作同步到 AOF 文件。如果是 yes，可以提升重写 AOF 文件时的性能，但如果此时宕机，会丢失这部分数据。

```
no-appendfsync-on-rewrite no
```

### 修复 AOF 文件

如果 AOF 文件被损坏，可以使用 Redis 的执行程序 redis-check-aof 进行修复，它位于 Redis 安装目录。

```
redis-check-aof --fix <AOF文件路径>
```

## RDB

### 持久化机制

RDB，Redis DataBase 把当前 Redis 中的数据以快照的方式写到磁盘文件，重启时就把快照文件读入内存，以实现数据恢复。RDB 快照文件是经过压缩的二进制文件。

### 相关配置属性

**持久化策略**

通过 save 配置属性，指定在多少秒内，发生多少次更新，就触发一次 RDB 持久化。可以配置多个 save，它们的关系是 or，任意一个条件满足都能触发持久化。

```
save <seconds> <changes>
```

**关闭 RDB**

Redis 默认开启 RDB，它没有专门提供一个配置属性负责开启、关闭 RDB。但是，可以把所有 save 配置删除，或添加一个空配置，以达到关闭 RDB 的效果。

```
save ""
```

**快照文件的名称和路径**

| 属性       | 说明                          |
| ---------- | ----------------------------- |
| dbfilename | 日志文件名称，默认 dump.rdb。 |
| dir        | 日志文件目录                  |

**其它配置属性**

| 属性                        | 默认 | 说明                                                     |
| --------------------------- | ---- | -------------------------------------------------------- |
| rdbcompression              | yes  | 持久化时是否压缩文件。                                   |
| rdbchecksum                 | yes  | 使用 RDB 快照恢复数据时，是否对文件进行校验。            |
| stop-writes-on-bgsave-error | yes  | 执行 bgsave 持久化命令时如果发生错误，Redis 终止写操作。 |

### 手动持久化命令

除触发 save 条件外，Redis 还支持使用命令手动触发 RDB 持久化。

**save**

执行 save 命令，Redis 将把当前内存里的数据写入快照文件，在写入的过程中会暂停执行其它命令。

**bgsave**

执行 bgsave 命令，Redis 会 fork 一个子进程，它拥有与父进程相同的数据，负责把内存数据写入快照文件。子进程首先是把数据写到一个临时文件，然后再用它替换快照文件。在此期间，Redis 可以执行其它命令。

> fork 可理解为拷贝，返回的子进程拥有与父进程一模一样的内容。

因为 bgsave 是后台运行，它只返回 "Background saving started" 信息。如果想检查 bgsave 是否成功，可以查看最近一次保存快照的时间，如果它与 bgsave 的执行时间对应，则说明 bgsave 执行成功。

```
lastsave
```

## 对比与选择

相比于 RBD，AOF 更容易触发持久化，它的数据更完整，而且 AOF 以追加的方式写入数据，性能更好。但是，RDB 文件是经过压缩的二进制文件，比 AOF 文件小，使用它恢复数据更快。

在某些场景，Redis 服务器在启动时无法使用 AOF 文件恢复数据，这种错误是灾难性的。

AOF 与 RBD 并不互斥，可以同时使用它们，Redis 会优先选择完整性更高的 AOF 恢复数据。当然，也可以不使用持久化功能，Redis 只作为缓存。

# 集群

单台 Redis 服务器通常不能满足要求。Redis 支持主从复制、哨兵、cluster 三种集群模式，它们能提高 Redis 的性能、吞吐量和可用性。

## 主从复制

### 集群机制

#### 主从复制

在基于主从复制模式的集群，服务器可分为主节点 master 和从节点 slave。向 master 写入的数据，会被同步复制到对应的 slave。这样，即使 master 出现问题，也可以使用 salve 的数据进行恢复，这能提高系统的可用性。

注意，数据只能从 master 同步到 slave，不能反向复制。slave 在连接 master 时会清空自身原来的数据，所以 slave 缓存的数据是 master 的子集。

info replication 命令返回当前节点主从复制的相关信息，比如当前 master 有多少个 slave 节点，以及这些节点的详细信息，再比如当前 slave 连接的 master 节点的详细信息。

#### 复制延迟

slave 连接 master 时，首先向 master 请求持久化文件以完成同步。此后，master 执行任何更新操作，都会随后发送给 slave，这能保证数据一致，但有一定的延迟。如果系统特别繁忙，延迟问题会很严重，增加 slave 节点同样也会加重延迟。

#### 迭代连接

Redis 支持一个节点，既作为 master，又充当 slave，也就是中间节点。中间节点可以分担 master 的同步压力，起到去中心化的作用。如果中间节点失效，两端的节点不会尝试连接，它们没有任何关系。

#### 读写分离

主从复制模式默认使用读写分离机制，也就是说，master 节点支持读写，而 slave 只能读。如果尝试向 slave 写数据，节点会报错。读写分离可以分割读写压力，这能提升系统的运行效率。

在 salve 执行 info replication 命令，返回的 slave_read_only 属性表示当前节点是否只读。可以通过修改配置属性，使 slave 支持写，但 slave 的数据不会同步到 master 节点。

```
slave_read_only no
```

#### 心跳机制

如果主从节点间正在进行数据同步，slave 会以一定的频率向 master 发送 REPLCONF 命令，以确保连接通畅，这就是心跳机制。

在 master 执行 info replication 命令，返回结果中描述 slave 节点的行内，lag 属性就是心跳的时间，单位秒。如果 master 发现 slave 失效，就应该停止同步。有两个配置属性与此相关，它们是 or 的关系，只要有一个满足，master 就会停止正在进行的同步。

* min-slaves-to-write 表示至少有多少个 slave 才能执行主从复制。
* min-slaves-max-lag 表示心跳延迟 lag 大于多少秒就不执行主从复制。

```
min-slaves-to-write 3
min-slaves-max-lag 5
```

### 搭建集群

Redis 默认是主节点，可以执行 slaveof 命令把当前节点作为另一个 Redis 服务器的从节点。

```
slaveof <IP> <Port>
```

还可以执行以下命令，把从节点转化为主节点。

```
slave no one
```

也可以通过配置属性定义从节点。

```
slaveof <ip> <port>
```

如果主节点设有密码，从节点连接时也需要相应的信息，使用 masterauth 配置属性进行设置。

```
masterauth <password>
```

最后，执行 info replication 命令查看主从节点是否连接成功。

## 哨兵模式

### 集群机制

主从复制模式实现数据备份和读写分离，这能提高系统的可用性和运行效率。但是，如果主节点发生故障，需要手动在从节点进行数据恢复，并重新配置主从关系。在这段时间内，无法写入数据，这会造成数据丢失。哨兵模式可以解决这个问题，它经常与主从复制联合使用。

所谓的哨兵 sentinel，其实是一个 Redis 节点。但它不与数据打交道，只负责监控指定的 master 节点，当它发现这个主节点失效，就执行 "故障自动恢复"，即从对应的从节点中选一个转化为主节点并重新分配主从关系。

哨兵完成故障恢复后，如果原来的主节点重新启动，它会被哨兵转换为新 master 的从节点。集群搭建

### 搭建集群

#### 修改配置文件

哨兵是 Redis 节点，自然也可以加载配置文件。使用 sentinel monitor 配置监视对象，MasterName 参数是哨兵为监视对象起的别名，num 参数在后文解释。

```
sentinel monitor <MasterName> <IP> <Port> <num>
```

如果主节点设有密码，哨兵连接时也需要相应的信息，使用 sentinel auth-pass 配置属性进行设置。

```
sentinel auth-pass <MasterName> <password>
```

故障恢复的时间是有限的，可通过 sentinel failover-timeout 配置属性设置，它表示如果经过 timeout 秒还未完成恢复，就认为此次恢复失败。

```
sentinel failover-timeout <MasterName> <seconds>
```

#### 启动哨兵节点

哨兵与普通节点不同，它使用与 redis-server 程序相同目录的 redis-sentinel 程序进行启动。

```
redis-sentinel <配置文件路径>
```

现在，可以使用 redis-cli 程序登录哨兵节点。执行 info sentinel 命令，它返回与哨兵相关的信息，比如监视节点的状态。

```
info sentine
```

#### 多个哨兵节点

可以使用多个哨兵监视一个主节点，前面 sentinel monitor 配置属性的最后一个参数 num，它表示当多少个哨兵认可时才能判定 master 失效。

哨兵如果在一定时间内没有收到 master 的正确回应，就认为它失效，有与这个时间相关的配置属性，单位毫秒。

```
sentinel down-after-milliseconds <MasterName> <timeout>
```

#### 选择新主节点

如果 master 有多个从节点，哨兵将根据以下规则选择新的 master 节点：

* 优选选择最高优先级，从节点的优先级可通过配置属性 replica-priority 设置，值越小越优先。

* 优先选择最大偏移量，偏移量 offset 表示 slave 从 master 同步的数据的字节数，越大数据越完整。

  ```
  replica-priority <value>
  ```

* 优选选择最小 runid，这是 Redis 启动时随机生成的 40 位数字。

## cluster 模式

### 集群机制

在基于主从复制 + 哨兵的集群，所有节点都缓存所有数据，这非常臃肿；而且虽然使用读写分离，但所有的写操作都集中在 master 节点，这依然会带来很大的压力。

对于 cluster 集群，它默认有 16384 个哈希槽 Hash Slot，槽是一个虚拟概念。设置 key-value 时，先用 CRC16 算法对 key 进行计算，再用 16384 对计算结果取模，最终结果就是这个 key 对应的槽的编号。我们可以把槽分配给多个 Redis 节点，根据前面的算法，把各个 key 存在其对应的槽所在的节点。

此外，cluster 还支持主从复制和故障自动恢复，而且从节点也属于 cluster 集群，它不需要额外设置专门的哨兵节点。更厉害的是，cluster 支持动态扩容，我们可以在运行期间向集群添加节点，并把旧节点的槽分配给新节点。

总结，cluster 集群支持主从复制、自动恢复、分布存储、动态扩容，它更适合项目应用。

### 搭建集群

#### 启动节点

Redis 默认不使用 cluster 模式，需要使用配置属性显式开启。

```
cluster-enabled yes
```

每个 Redis 在首次以 cluster 节点身份启动时，会生成一个配置文件，其中保存着这个节点和集群其它节点的信息以及关联方式，这个配置文件的名称需要使用配置属性显式设置。

```
cluster-config-file <ConfName>
```

现在，就可以使用 redis-server 程序启动所有的 cluster 节点，包括 master 和 slave。

cluster info 命令返回当前节点在 cluster 集群的相关信息，比如连接状态。此时在任意节点执行这个命令，可以发现它们都处于 fail 状态。因为这些节点还未进行集群配置，仍处于独立状态。

cluster node 命令返回当前节点所在 cluster 集群的所有节点信息，返回的每一行描述一个节点，包括 IP、端口号、主从关系等信息。对于状态 OK 的节点，行的第一个长字符串是 node-id，这是节点在 cluster 的唯一标识；对于 master 节点，行的尾部会描述其槽区间；对于 slave 节点，行中间会显示其 master 的 node-id。

#### 连接节点

首先要打破隔绝，使各个节点发现并连接彼此，这一步使用 redis-cli 程序的 cluster meet 功能，格式如下。

```
redis-cli -h <host> -p <Port> cluster meet <IP> <Port>
```

这个程序会登录 -h -p 指定的 Redis 服务，并使它与 IP Port 指定的节点连接，组成 cluster 集群。如果节点 A 分别与 B 和 C 进行 cluster meet，那么 B 和 C 自然处于同一个 cluster 集群，它们会自动连接。

此时再执行 cluster info 查看集群信息，发现各节点仍处于 fail 状态，因为还没有为它们分配槽位。

#### 分配槽位

前面提到，槽是一个逻辑概念，每个 key 可以通过计算唯一映射到一个槽。如果把槽位分配给某个节点，那么写入集群的映射到这个槽位的 key 就会存放在这个 cluster 节点。

由于 cluster 支持主从复制，且 slave 节点也属于集群，所以并非所有 cluster 节点都会分配槽位。至少，slave 节点不分配。没有槽位的 master 节点不会存储集群数据，所以它的状态为 fail。

槽位的分配使用 redis-cli 程序的 cluster addslots 功能，格式如下。

```
redis-cli -h <host> -p <Port> cluster addslots <id>
```

这个程序一次只为一个节点分配一个槽，id 是槽的编号。cluster 集群可有 16384 个槽，可以使用以下 bash 命令批量执行程序，它把 [start, end] 区间的槽分配给指定节点。

```
redis-cli -h <host> -p <Port> cluster addslots $(seq start end)
```

> cluster 集群有 16384 个槽，这些槽的编号从 0 到 16383。

此时执行 cluster info 查看集群信息，cluster_slots_assigned 属性表示分配的槽数量，cluster_slots_ok 属性表示可以写入的槽数量。且所有节点的状态已经变为 OK。

#### 配置主从

最后，为 master 配置从节点。这一步需要登录从节点，执行以下 Redis 命令，它使当前节点在 cluster 集群中作为指定节点的从节点。

```
cluster replicate <主节点的node-id>
```

> node-id 通过 cluster nodes 命令查看。

至此，cluster 集群搭建完成。

#### 读写透明

现在可以向 cluster 集群读写数据，但要为 redis-cli 的启动命令添加 -c 参数。因为 cluster 集群的数据分布在多个节点，普通客户端连接无法向集群的其它节点读写数据，-c 参数指示当前节点把命令转发给集群正确的节点并返回结果。

这样，无论登录哪个 cluster 节点，即使是 slave，都能向整个集群读写数据。

```
redis-cli -c -h <host> -p <Port>
```

#### 再分配槽

在 cluster 集群运行期间，可以对已分配的槽位进行再分配，这使用 redis-cli 程序的 cluster reshard 功能，格式如下。IP 和 Port 指定由哪个节点执行再分配，可以选择任意一个空闲节点；cluster-from 指定源，cluster-to 指定目标，cluster-slots 指定为目标分配的槽数量。

```
redis-cli --cluster reshard <IP>:<Port> 
	--cluster-from <node-id> [,<node-id>,<node-id>...] 
	--cluster-to <node-id>
	--cluster-slots <num>
```

然后，在客户端执行 cluster info 命令查看分配结果。

使用 reshard 进行动态扩容时，要预先按照前面的步骤加入新节点，然后再执行 reshard 程序。

注意，再分配时，在迁移槽以及对应的数据这段时间，这些数据都不可用，可能会出现缓存失效的现象。

### 其它配置属性

**cluster-require-full-coverage**

表示有节点失效时，cluster 集群是否不提供写服务。如果 yes，有节点失效时，集群只能读数据。

**cluster-node-timeout**

节点的最长不可用时间，单位毫秒。当某个 cluster 节点失效时间超过指定值，那么其从节点就进行故障恢复。

**cluster-migration-barrier**

主节点的最小从节点数。在 cluster 集群，如果某个 master 的 slave 数少于这个值，cluster 会自动从其它有富余的 master 调剂从节点过来。合理设置这个值，可以提升集群的可靠性。

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

使用 Jedis 获取 Redis 的连接有两种方式：直接连接和连接池，它们的区别和优劣已是老生常谈，不再赘述。

下例使用 Jedis 类直接连接 Redis。

```
public static void test_connect_redis() {
    Jedis jedis = new Jedis(IP, Port);
    // response PONG if connect successfully
    Assertions.assertEquals("PONG", jedis.ping());
    jedis.close();
}
```

下例使用 JedisPool 连接池与 Redis 通信。

```
public void test_connect_pool() {
    JedisPool jedisPool = new JedisPool(IP, Port);
    Jedis jedis = jedisPool.getResource();
    // response PONG if connect successfully
    Assertions.assertEquals("PONG", jedis.ping());
    jedisPool.close();
}
```

连接池 JedisPool 有几个核心参数，如下表所示。

| 属性               | 说明                                                       |
| ------------------ | ---------------------------------------------------------- |
| maxTotal           | 最大连接数                                                 |
| minIdel            | 最少空闲连接数                                             |
| maxIdle            | 最大空闲连接数                                             |
| blockWhenExhausted | 连接数最大时，是否阻塞新申请连接的线程，不阻塞则抛出异常。 |
| maxWait            | 阻塞时间，超时抛出异常。                                   |

## 执行各种命令

Jedis 拥有各种执行 Redis 命令的方法，它们的名字类似，下例示范各种操作。

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

Jedis 每次调用方法执行 Redis 命令都要与其进行一次通信，如果有大量操作，而每个操作都单独发送请求，这会严重降低通信效率。使用管道，可以批量发送请求和获取结果。

```
public void test_pipeline() {
    Pipeline pipelined = jedis.pipelined();
    pipelined.set("key", "pipeline");
    // send all commands until execute sync method
    pipelined.sync();
}
```

# SpringBoot 整合 Redis

Redis 作为重要的缓存中间件，SpringBoot 自然对其提供集成，下面是官方提供的 starter 包依赖。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

引入依赖并配置连接后，就可以在任何需要操作 Redis 的地方，注入 RedisTemplate 组件。RedisTemplate 类似于 Jedis，用于发送请求和接收结果，但它两的 API 完全不同，使用方式也有差异。

```
@Autowired
private RedisTemplate redisTemplate;

public void readAndWrite() {
	redisTemplate.opsForValue().set("key", "readAndWrite");
	Assertions.assertEquals("readAndWrite", redisTemplate.opsForValue().get("key"));
}
```

这里详细说明 SpringBoot 整合 Redis 时的配置。对于单服务器，直接配置 IP、Port 等属性即可。

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

整合 cluster 集群，只需配置 spring.redis.cluster.nodes 属性，它指定所有 master 节点。

```
spring:
  reids:
    cluster:
      nodes: IP:Port[,IP:Port,...]
```

当然，spring.redis 还有许多可配置的属性，这里深究。

# Redis 实战和应用问题

## 基本应用

在高并发场景，如果读写数据的请求都集中到 MySQL 等关系型数据库，可能会瞬间压垮它。因为关系型数据库通常把数据持久化到磁盘，读写数据可能导致大量磁盘 IO；如果有 order by、limit 等语句，数据库还要进行额外处理，这很耗费 CPU 资源；数据库支持的最大连接数有限，请求不一定都能成功执行。

此时，引入 Redis，把频繁访问的数据存入缓存，每次请求数据时，先尝试从 Redis 获取，失败后再访问数据库。初始时，Redis 为空，当某个数据初次被访问时，程序在 Redis 查不到，就向数据库请求，获得结果后先回设到缓存，然后才返回给用户。在此之后，相同数据就可以直接从缓存读取，而不用麻烦数据库。

由于 Redis 的速度非常快，它可以有效缓解数据库的压力，提升程序的响应速度。

## 缓存穿透

前面提到，初次访问的数据，先从数据库获取，然后回设缓存。但如果这个数据根本就不存在呢？按照前面的逻辑不会回设缓存，那么之后所有访问这个数据的请求还是会打到数据库，缓存层相当于没有，这就是缓存穿透。

缓存穿透的原因是不存在的数据没有被缓存，那么我们就把它们缓存起来。访问数据库时，如果返回 null，即这个数据不存在，就对其缓存 null、0 等空值 。

布隆过滤器。（TODO）

## 内存溢出

Redis 的所有数据都运行在内存，但内存是有限的，如果不为数据设置过期时间，很快就会使用完，造成 OOM 异常。

过期时间要根据应用场景而定，而且应该是一个范围内的随机值。如果过期时间固定，那么同时回设的缓存会同时失效，如果此时收到大量访问这些数据的请求，它们都会打到数据库，造成缓存雪崩，压垮数据库。

## 缓存击穿

如果某个数据在失效后，突然收到大量访问它的请求，那这些请求都会打到数据库，造成缓存击穿，压垮数据库。这种情况很常见，比如新热点数据就会突然被高并发访问。

缓存击穿的原因是大量访问 "同一个数据" 的请求在缓存失效时同时发起，它的数据真实存在，而缓存穿透的数据不存在，缓存雪崩是访问多个数据。

可以使用互斥锁解决这个问题，具体依靠 setnx 命令实现。当这些并发请求发现缓存失效，先不访问数据库，而是使用 setnx 命令设置值，相当于上锁，设置成功的请求获得锁，允许访问数据库，它获得数据后回设缓存并使用 del 命令释放锁。设置失败说明已有一个线程尝试访问数据库，那就等待一段时间，然后回到第一步查看缓存。

## 缓存雪崩

缓存雪崩类似于缓存击穿，都是由缓存失效引起，但它是批量数据同时过期，造成访问这些数据的大量并发请求打到数据库，从而压垮它。

设置随机过期时间，使缓存分散失效，可以避免雪崩；设置多级缓存，比如 Nginx+Redis；如果不要求高并发处理能力，可以考虑使用锁、队列保证不会有大量请求同时访问数据库。

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
* 此时，执行 C，查缓存没有结果，再查数据库，回设缓存并返回；
* B 继续执行，写数据库，覆盖旧值。

此时，数据库保存着 B 的新值，而缓存的却是 C 回设的旧值。

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

此时，数据库保存着新值，而缓存的却是旧值。但它发生的概率很小，因为需要同时满足缓存失效，和读操作比写操作更耗时两个条件。

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
mkdir -p /etc/docker/redis/data
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
