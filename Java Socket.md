# 网络编程

**系统调用和编程接口**

应用程序可以通过系统调用接口，使用系统内置的服务。所谓系统调用，与程序设计的函数调用相似，只不过提供服务的一端是操作系统。由于在使用系统调用时，需要编写程序，系统调用接口也称为应用编程接口。

现代操作系统都内置 TCP/IP 协议软件，因为 TCP/IP 协议是应用在各种系统，所以并未规定 TCP/IP 软件的系统调用接口的实现细节，具体由设计者自由实现。目前只有几种可用的接口，其中最著名的就是 Socket Interface 套接字，它把网络连接的两端，抽象为两个数据的收发器。现在，套接字接口已被系统内置。

Java 原生支持网络编程，使用 JDK 就能在网络收发数据，原理就是在底层通过系统调用接口使用 TCP/IP 软件。

**通信机制：同步，异步**

同步，函数调用在得到结果之前不会返回，也就是说，调用者需要等待结果。而异步的调用直接返回，但不返回结果，函数得出结果后，通过状态、通知、回调等方式告知调用者。

**调用状态：阻塞，非阻塞**

程序等待调用结果时的状态。阻塞，调用者的线程在等待时会被挂起，直到函数返回。非阻塞，调用者在不能立即得到结果之前，会一直占用 CPU 运行。

# 套接字 Socket

`java.net.Socket` 表示套接字，基于 TCP 协议，可以主动向服务端发起连接请求，或与另一个 `Socket` 对象进行通信。注意，Socket 接口，并不只限于 TCP/IP 协议。

## 建立 TCP 连接

**普通构造方式**

带参的构造器都会立即发出连接请求，连接成功返回 `Socket` 对象，否则抛出异常，有以下几种异常情况，它们都是 `IOException` 的子类：

* `BindException`：绑定端口失败，可能被占用。
* `UnknownHostException`：服务端的域名或 IP 识别失败。
* `ConnectedException`：服务端没有进程监听端口，或拒绝连接。
* `SocketTimeoutException`：连接等待超时。

>**java.net.Socket**
>
>* `Socket(String host, int port[, InetAddress localAddr, int localPort])`
>
>* `Socket(InetAddress address, int port[, InetAddress localAddr, int localPort])`
>
> 构造器。*host*、*port* 和 *address* 定位服务端，本地的 IP 和端口由系统分配，如果有多个 IP 地址，或想指定绑定端口，可以使用 *localAddr*、*localPort* 参数。
>

**自定构造方式**

无参的空构造器，什么信息都没有，需要显式调用方法向服务端请求连接。

>**java.net.Socket**
>
>* `Socket()`：构造器。
>* `connect(SocketAddress endpoint)`：请求连接，*endpoint* 定位服务端。

`Socket` 发出连接请求，需要等待一段时间，因为网络有延迟，请求也要排队，默认永久等待。可以使用重载的 `connect()` 方法，设置限时等待，超时抛出异常。

> **java.net.Socket**
>
> * `connect(SocketAddress endpoint, int timeout)`：*timeout* 是超时时间，单位毫秒。

## 配置 TCP 连接

使用空构造器，除限时等待，更多时候是为定制 TCP 连接的细节，这些本来都由 TCP/IP 软件自动配置，但也可以通过 Socket 选项修改默认值。

**发送缓冲，TCP_NODELAY**

`Socket` 写出的数据不会立即被发送到网络，而是先放在本地的发送缓冲区，等到区满后再打包发出，这样可以减少传输次数，提高传输效率。

> **java.net.Socket**
>
> * `setTcpNoDelay(boolean on)`：关闭发送缓冲。

**本地端口复用，SO_RESUSEADDR**

`Socket` 调用 `close()` 后，如果网络还有发送到这个 `Socket` 的数据，底层 Socket 不会立即释放端口，而是延迟一段时间，以接收剩余数据。收到数据后不做任何处理，这样做只为保证后续占用这个端口的进程不会收到无关的数据。

延迟释放端口期间，端口处于被占用中，如果想让其它进程能够绑定这个端口，可以开启本地端口复用，这样新的 `Socket` 就能绑定还未被真正释放的端口，当然，它会收到那些与自己无关的数据。

注意，新、旧 `Socket` 都需开启本地端口复用，否则会绑定失败。

> **java.net.Socket**
>
> * `setReuseAddress(boolean on)`：开启本地端口复用。

**读数据阻塞超时，SO_TIMEOUT**

`Socket` 通过 BIO 传输数据，若输入流没有数据，读会阻塞，可以设置读数据的阻塞超时时间。并且，因为阻塞超时而抛出过异常的 `Socket` 对象仍然可用。

> **java.net.Socket**
>
> * `setSoTimeout(int timeout)`：开启读数据限时阻塞，单位毫秒，默认 0 表示永久阻塞。

**发送数据阻塞，Socket SO_LINGER**

`Socket` 调用 `close()` 后，底层的 Socket 不会立即关闭，而是延迟一段时间，直到把发送缓冲区的剩余数据都发送出去。可以设置 `close()` 调用限时阻塞，阻塞期间底层的 Socket 不断发送数据，当发送完剩余数据，或阻塞超时，方法返回，底层 Socket 立即关闭。这能保证，`close()` 返回后底层的 Socket 也关闭。

> **java.net.Socket**
>
> * `setSoLinger(boolean on, int linger)`：开启 `close()` 限时阻塞，时间单位毫秒。

**发送缓冲大小，SO_RCVBUF**

发送缓冲区的大小的默认值由操作系统决定。

> **java.net.Socket**
>
> * `setReceiveBufferSize(int size)`：设置发送缓冲区的大小，单位字节。

**接收缓冲大小，SO_SNDBUF**

接收缓冲区的大小的默认值由操作系统决定。

> **java.net.Socket**
>
> * `setSendBufferSize(int size)`：设置接收缓冲区的大小，单位字节。

**自动关闭空闲，Socket SO_KEEPLIVE**

对空闲超过 2 小时的连接，发送一个数据包给远程端点，如果没有响应，则持续尝试 11 分钟，如果在 12 分钟内未收到响应，就关闭 `Socket` 对象，断开连接。不同平台，TCP 的尝试时间可能不同。

> **java.net.Socket**
>
> * `setKeepAlive(boolean on)`：开启自动关闭空闲连接。

**紧急信息，OOBINLINE**

紧急信息是 TCP 的 1 字节的特殊数据，默认关闭，大多 TCP 实现都没有对紧急信息的处理，这个功能很鸡肋。

开启以允许发送 1 字节的 TCP 紧急数据，默认关闭。

> **java.net.Socket**
>
> * `setOOBInline(boolean on)`：开启紧急信息。
>* `sendUrgentData (int data)`：发送紧急信息。

## 进行网络通信

套接字是网络数据的收发器，`Socket` 通过 IO 流在 API 层传输网络数据。

> **java.net.Socket**
>
> 
>* `InputStream getInputStream()`：获取 TCP 连接的输入流。
> 
>* `OutputStream getOutputStream()`：获取 TCP 连接的输出流。

## 关闭网络连接

网络通信结束后，应该立即关闭 `Socket` 对象，以释放占用的系统资源。

> **java.net.Socket**
>
> * `close()`：关闭套接字。
>
> * `boolean isClosed()`
> * `boolean isBound()`
> * `boolean isConnected()`

判断套接字是否处于连接状态：

```
socket.isConnected() && !socket.isClosed();
```

半关闭状态，指套接字的 IO 流被关闭，但还没有释放占用的资源，比如端口号。

> **java.net.Socket**
>
> * `shutdownInput()`
>
> * `shutdownOutput()`
>

# 服务端 ServerSocket

`java.net.ServerSocket` 表示服务端，基于 TCP 协议，可以监听本地端口，接收 `Socket` 的连接请求，并返回一个 `Socket` 对象与客户端 `Socket` 进行通信。

## 建立服务端点

除空构造器，其它构造方法都会绑定端口，并立即进行监控，随时接收连接请求。可设绑定的端口号为 0，表示让系统随机分配。绑定 IP 由系统随机分配，如果有多个 IP 地址，可能需要专门指定。连接请求由系统管理，这些请求被放在一个队列，队满后系统会拒绝新的连接请求，队列长度通常是 50，如果自定义的长度小于 0，或大于最大限制，则仍使用默认长度。

对于 `Socket` 而言，发出的连接请求只要成功加入连接请求队列，就意味着连接建立成功。

> **java.net.ServerSocket**
>
> * `ServerSocket([int port,[ int backlog[, InetAddress bindAddr]]])`
>
>   构造器，*port* 是绑定端口，*backlog* 是连接请求队列的长度，*bindAddr* 是绑定地址。
>

## 配置服务端点

空构造器创建的 `ServerSocket` 对象，可以在运行之前，进行相关配置，以修改默认的运行方式。

**客户连接超时时间，SO_TIMEOUT**

`ServerSocket::accept()` 从连接请求队列取出一个请求，如果队列为空，默认会永久阻塞线程，直到队列有请求可被取出。可以通过选项修改 `accept()` 为限时阻塞，超时抛出异常。

> **java.net.ServerSocket**
>
> * `setSoTimeout(int timeout)`：单位毫秒。

**本地端口复用，SO_RESUSEADDR**

这个选项与 `Socket` 的 SO_RESUSEADDR 作用完全相同。

> **java.net.ServerSocket**
>
> * `setReuseAddress(boolean on)`

**接收缓冲大小，SO_SNDBUF**

相当于为所有 `accept()` 返回的 `Socket` 对象设置接收缓冲大小。通常，在 `ServerSocket` 绑定端口前后设置接收缓冲都会生效，但缓冲如果大于 64K，就必须在绑定端口前设置。

> **java.net.ServerSocket**
>
> * `setSendBufferSize(int size)`：设置接收缓冲区的大小，单位字节。

## 接收连接请求

`accept()` 从连接请求队列取出一个请求，然后返回一个与这个请求通信的 `Socket` 对象。如果队列为空，这个方法默认会永久阻塞，直到队列有请求可被取出。

> **java.net.ServerSocket**
>
> * `Socket accept()`：接收连接请求，返回与之通信的 `Socket` 对象。
>
> * `bind(SocketAddress endpoint[, int backlog])`
>
>   *endpoint* 绑定资源，比如本地 IP 和端口，*backlog* 是请求队列长度。

## 关闭服务端点

停止服务后，应该立即关闭 `ServerSocket` 对象，以释放占用的系统资源，并断开所有 `Socket` 连接。

> **java.net.ServerSocket**
>
> * `close()`
> * `boolean isBound()`
> * `boolean isClosed()`

# 非阻塞 NIO

BIO 使用数据流的思想，流是连接数据源/汇和 Java 程序的通道，用户可以向流写数据，或从流读数据，以此与网络、文件进行数据传输。

因为读写 BIO 可能阻塞，且 `ServerSocket::accept()` 也是阻塞调用，所以如果 `ServerSocket` 想要同时与多个客户端 `Socket` 进行通信，将不得不使用多线程。维护多个线程比较耗费资源，即便使用成熟的线程池。

非阻塞 IO 使用 `Channel` 连接数据源/汇和 Java 程序，`Channel` 可读可写。程序使用 `Buffer` 与 `Channel` 交换数据，而不直接操作通道。通道支持非阻塞，当不可读不可写时，方法直接返回，不会阻塞。另外，`Channel` 还能与选择器 `Selector` 合作，实现 IO 多路复用模型。

## 缓冲 Buffer

`Buffer` 是 JVM 内存的一块空间，可以存放数据，某些 BIO 带有缓冲功能，其实就是在内部使用 `Buffer` 临时保存数据。使用缓冲，可以减少物理 IO 的次数，还能不断复用申请的内存。

`Buffer` 有 3 个重要属性，指向 3 种位置：

* `capacity`：缓冲容量，不可改变。
* `position`：读写位置，非负整数，不大于 `limit`，用户根据 `position` 读写数据。
* `limit`：读写上限，写模式时为 `capacity` 值，转换读模式后，又更新为 `position` 值。

`Buffer` 初始是写模式，每写入一个单元数据，*position++*，直到 position 等于 limit，不能继续写入。再转换为读模式，把 limit 设为 position，表示内容边界，把 position 设为 0，表示内容开始位置，每读取一个单元数据，*position++*，直到 position 等于 limit。

调用 `clear()` 清空缓冲，其实只是把 limit 和 position 重置为初始值。所以，使用这 3 个属性，可以省去不少修改真实数据的操作，提高读写的效率。

此外，还有一个 mark 属性，它标记某个位置，初始为 -1，调用 `reset()` 可把 position 置为 mark。

> 注意，任何时候都能读和写，所谓的转换读模式，只是修改属性的值，并不存在想象中的模式限制。

> **java.nio.Buffer**
>
> * `Buffer flip()`：转换读模式，*limit=position*，*position=0*。
> * `Buffer clear()`：清空缓冲区，*limit=capacity*，*position=0*。
> * `Buffer rewind()`：重置，*position=0*，*mark=*-1。
> * `int remaining()`：查看剩余容量，*limit-position*。

如果读数据中途想要追加数据，但还有一些数据未读，可以调用 `compact()`，该方法把 *[position,limit]* 的未读数据移至最前面，并设 position 为 *limit-position*，设 limit 为 capacity。这样，就可以照常写入数据，下次读时就会从上次未读的数据开始。`Buffer` 没有声明 `compact()`，但大多缓冲都有该方法。

`ByteBuffer` 是 `Buffer` 的重要实现，以字节为读写单元。除 `boolean` 类型，其它基本类型都有以其为读写单元的 `Buffer` 实现，

> **java.nio.ByteBuffer**
>
> * `static allocate(int capacity)`：工厂方法，*capacity* 是缓冲容量。
>
> * `static allocateDirect(int capacity)`：工厂方法，使用直接缓冲，开销大但读写快。
>
> * `byte get()`：读 *position* 位置的一个字节。
>
> * `ByteBuffer put(byte b)`：在 *position* 位置写一个字节。
>
> * `byte get(int index)`：读 *index* 位置的一个字节。
>
> * `ByteBuffer put(int index, byte b)`：在 *index* 位置写一个字节。

## 通道 Channel

`Channel` 是连接数据源/汇和缓冲区 `Buffer` 的通道，另外，`Channel` 也可以与 `Channel` 连接。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-d9b490fk.png)

### Channel

`Channel` 接口声明两个方法，通道初始处于打开状态，只要关闭，就无法再次打开。

> **java.nio.channels.Channel**
>
> * `close()`
> * `boolean isOpen()`

`ReadableByteChannel` 和 `WritableByteChannel` 是 `Channel` 的拓展，增加读、写数据的方法。

> **java.nio.channels.ReadableByteChannel**
>
> * `int read(ByteBuffer dst)`：从数据源读数据，并放入 `ByteBuffer`，返回读的字节数。
>

> **java.nio.channels.WritableByteChannel**
>
> * `int write(ByteBuffer src)`：把 `ByteBuffer` 的内容写到数据汇，返回写的字节数。

`ScatteringByteChannel` 和 `GatheringByteChannel` 是进一步的拓展，增加分散读、集中写多个 `ByteBuffer` 的方法。

> **java.nio.channels.ScatteringByteChannel**
>
> * `long read(ByteBuffer[] dsts[, int offset, int length])`
>
>   从数据源读数据，并依次放入数组的每个 `ByteBuffer`，返回读的字节数。

> **java.nio.channels.GatheringByteChannel**
>
> * `long write(ByteBuffer[] srcs[, int offset, int length])`
>
>   把数组的 `ByteBuffer` 的内容依次写到数据汇，返回写的字节数。

`FileChannel` 是常见的通道类型，支持以文件作为数据源/汇，使用工厂方法 `open()` 创建实例，或者，使用文件传输流的 `getChannel()` 方法获取对应的通道。

### SelectableChannel

`SelectableChannel` 实现 `Channel` 接口，抽象类声明 `Selector` 相关的方法，并支持阻塞/非阻塞通信。

> **java.nio.channels.SelectableChannel**
>
> * `configureBlocking(boolean block)`：设置阻塞/非阻塞模式，默认阻塞。
>
> * `SelectionKey register(Selector sel, int ops[, Object att])`
>
>   向 `Selector` 注册事件，返回 `SelectionKey` 对象，*att* 是关联对象，默认为 `null`。

### ServerSocketChannel

`ServerSocketChannel` 继承 `SelectableChannel`，表示非阻塞服务端通道，封装 `ServerSocket` 实例，支持非阻塞地接收连接请求。

阻塞模式的行为与 `ServerSocket` 相同。非阻塞模式，如果请求连接队列为空，`accept()` 立即返回 `null`。`accept()` 返回的 `SocketChannel` 对象默认是阻塞模式。

> **java.nio.channels.ServerSocketChannel**
>
> * `static open()`：工厂方法。
> * `bind(SocketAddress local[, int backlog])`：绑定资源。
> * `SocketChannel accept()`：接收连接，并返回与之通信的 `SocketChannel` 对象。
> * `int validOps()`：返回支持事件：*OP_ACCEPT*。
> * `ServerSocket socket()`：返回关联 `ServerSocekt`。

### SocketChannel

`SocketChannel` 继承 `SelectableChannel`，表示非阻塞套接字通道，封装 `Socket` 实例，支持非阻塞地发出连接请求和读写数据。

阻塞模式的行为与 `Socket` 相同。非阻塞模式，`connect()` 发起连接，如果不能立即完成，返回 `false`。稍后必须调用 `finishConnect()` 确认连接结果，这个方法的作用是二次确认连接结果，它在非阻塞模式必须，因为网络连接往往不能立即建立，连接成功返回 `true`，仍在连接返回 `false`，连接失败则抛出异常。

除连接外，`SocketChannel` 还支持非阻塞读写数据。非阻塞模式，读操作遇到通道没有数据，或数据不够多，只读可读的数据，然后立即返回。写操作也是如此，尽可能多地向通道写数据，无法写时立即返回。

> **java.nio.channels.SocketChannel**
>
> * `static open()`：工厂方法。
>
> * `connect(SocketAddress remote)`：请求连接。
>
>   `boolean isConnectionPending()`：仅当发出连接，但还未 `finishConnect()`，返回 `true`。
>
>   `finishConnect()`：完成连接过程。
>
> * `int validOps()`：返回支持事件：*OP_READ* + *OP_WRITE* + *OP_CONNECT*。
>
> * `Socket socket()`：返回关联 `Socket`。
>
> * `int read(ByteBuffer dst)`
>
>   `long read(ByteBuffer[] dsts)`
>
>   从数据源读数据，并放入 `ByteBuffer`，返回读的字节数。
>
> * `int write(ByteBuffer src)`
>
>   `long write(ByteBuffer[] srcs)`
>
>   把 `ByteBuffer` 的数据写到数据汇，返回写的字节数。

## 选择器 Selector

虽然 `SelectableChannel` 支持非阻塞，但它不知道何时能执行操作，这就需要不停地调用方法进行尝试，非常耗费资源。`Selector` 是通道的监视器，它会不断地轮询检查已经注册的通道是否准备就绪。这样，非阻塞模型就变成 IO 多路复用模型。

`register()` 返回一个 `SelectionKey` 对象，它是跟踪注册通道的句柄，选择器有 3 个相关的集合：

* `all-keys`：包括所有注册通道的 `SelectionKey`，调用 `keys()` 返回。
* `selected-keys`：包括所有被发现发生事件的通道的 `SelectionKey`，调用 `selectedKeys()` 返回。
* `cancelled-keys`：包括所有取消的通道的 `SelectionKey`，这个集合无法访问。

初始这 3 个集合都为空。注册通道，选择器会创建对应的 `SelectionKey` 对象并把它加入 all-keys 集合。关闭通道，或取消 `SelectionKey`，这个 `SelectionKey` 会被加入 selected-keys 集合，每次执行 `select()` 都会把 cancelled-keys 的元素从 3 个集合清除。

此外，`select()` 还会把发生事件的通道的 `SelectionKey` 加入 selected-keys 集合。用户查看这个集合的元素，从而知道哪些通道能执行哪些操作，结束后必须手动把这些 `SelectionKey` 从 selected-keys 删除，这可以使用 `remove()` 或迭代器完成。

> **java.nio.channels.Selector**
>
> * `static open()`：工厂方法。
>
>   `boolean isOpen()`：是否打开，默认打开。
>
>   `close()`：关闭选择器，所有 `SelectionKey` 会被取消，关联 `Channel` 也会被取消。
>
> * `Set<SelectionKey> keys()`：返回 all-keys 集合。
>
>   `Set<SelectionKey> selectedKeys()`：返回 selected-keys 集合。
>
> * `int select()`：返回准备就绪的通道数量，没有则阻塞，直到事件发生，或被唤醒，或线程中断。
>
>   `int select(long timeout)`：限时阻塞，超时正常返回，单位毫秒。
>
> * `wakeup()`：唤醒因 `select()` 而阻塞的线程。

## 事件句柄 SelectionKey

`SelectionKey` 是用于追踪注册到 `Selector` 的通道的句柄，它在以下情况失效：

* 调用 `cancel()`
* 关联的 `Selector` 关闭
* 关联的 `SelectableChannel` 关闭

`SelectionKey` 有 4 个静态常量，表示 4 种事件，可读、可写、可连接，以及可接收连接：

* `public static final int OP_READ = 1 << 0;`
* `public static final int OP_WRITE = 1 << 2;`
* `public static final int OP_CONNECT = 1 << 3;`
* `public static final int OP_ACCEPT = 1 << 4;`

`ServerSocketChannel` 支持 *OP_ACCEPT* 事件，`SocketChannel` 支持剩余的 3 种事件。

> **java.nio.channels.SelectionKey**
>
> * `Selector selector()`：返回关联 `Selector`。
>
> * `SelectableChannel channel()`：返回关联 `SelectableChannel`。
>
> * `boolean isValid()`：是否有效。
>
>   `cancel()`：取消注册，会被 `Selector` 加到 cancelled-keys 集合。
>
> * `int interestOps()`：返回注册的事件。
>
>   `interestOps(int ops)`：重设注册的事件。
>
> * `int readyOps()`：返回就绪的事件。
>
> * `boolean isReadable()`
>
>   `boolean isWritable()`
>
>   `boolean isConnectable()`
>
>   `boolean isAcceptable()`
>
>   是否可读、可写、可连接、可接收连接。
>
> * `Object attach(Object ob)`：绑定关联对象。
>
>   `Object attachment()`：获取关联对象。

## 字符编码 Charset

`Charset` 表示字符编码，`StandardCharsets` 声明许多静态常量，都是常用的字符编码实例。

> **java.nio.charset.Charset**
>
> * `static Charset forName(String charsetName)`：根据字符集名字获取实例。
> * `static Charset defaultCharset()`：查看平台的默认编码。
> * `final ByteBuffer encode(CharBuffer cb)`：把 *cb* 的内容编码为 `ByteBuffer`。
> * `final CharBuffer decode(ByteBuffer bb)`：把 *bb* 的内容解码为 `CharBuffer`。

# 异步 AIO

## 相关的类

**AsynchronousChannel**

`AsynchronousChannel` 是 `Channel` 的子接口，表示异步通道，只有一个 `close()` 方法。

**AsynchronousServerSocketChannel**

`AsynchronousServerSocketChannel` 实现 `AsynchronousChannel` 接口，表示异步服务端通道。

> **AsynchronousServerSocketChannel**
>
> * `static open([AsynchronousChannelGroup group])`：工厂方法。
>
> * `bind(SocketAddress local[, int backlog])`：绑定资源，比如本地 IP 和端口。
>
> * `Future<AsynchronousSocketChannel> accept()`
>
> * `void accept(A attachment, CompletionHandler handler)`
>
>   异步接收连接，前者使用 `Future` 阻塞获取结果，后者使用 `handler` 异步处理结果。

**AsynchronousSocketChannel**

`AsynchronousSocketChannel` 实现 `AsynchronousByteChannel` 接口，后者是 `AsynchronousChannel` 的子接口，表示异步套接字通道，以字节为单元传输数据。

> **AsynchronousSocketChannel**
>
> * `static open([AsynchronousChannelGroup group])`：工厂方法。
>
> * `Future<Void> connect(SocketAddress remote)`
>
> * `connect(SocketAddress remote, A attachment, CompletionHandler handler)`
>
>   异步发起连接，前者使用 `Future` 阻塞获取结果，后者使用 `handler` 异步处理结果。
>
> * `void read(ByteBuffer dst, A attachment, CompletionHandler handler)`
>
> * `void write(ByteBuffer src, A attachment, CompletionHandler handler)`
>
>   异步读、写数据，完成后使用 `handler` 异步处理结果。

**AsynchronousChannelGroup**

表示多个异步通道使用的共享资源，主要是线程池。如果没有指定，异步通道默认使用系统提供的分组。

> **AsynchronousChannelGroup**
>
> * `static withFixedThreadPool(int nThreads, ThreadFactory threadFactory)`
>
> * `static withCachedThreadPool(ExecutorService executor, int initialSize)`
>
> * `static withThreadPool(ExecutorService executor)`
>
>   工厂方法，自定义线程池。

**CompletionHandler<V,A>**

表示任务完成后的回调，用于处理返回结果。两个泛型，第一个是返回类型，第二个是绑定对象类型，绑定对象用于辅助任务完成，可有可无。

>**CompletionHandler**
>
>* `void completed(V result, A attachment)`：完成回调。
>* `void failed(Throwable exc, A attachment)`：失败回调。

## 异步回调

异步通道支持异步地接收请求、发送请求、读/写数据，并在完成后进行回调，有两种回调方式。

**阻塞等待**

对于返回 `Future` 的异步方法，执行 `Future::get()` 获取结果，当前线程会阻塞，直到异步任务结束，结果可获取。这种方式，必须显式获取返回结果，然后进行处理，也就是说，回调逻辑在主线程内。

**异步回调**

把 `CompletionHandler` 对象传入异步调用，无论成功或异常，异步任务结束后都会把结果传给 Handler 的方法进行处理。这种方式比较典型，回调逻辑封装在专门的对象，推荐使用，但它的编写比上面一种更难。

# 系统 IO 模型

最开始提到，应用通过系统调用使用操作系统的 TCP/IP 软件进行网络通信。数据首先发送到网卡，然后被系统接收存放在内核，最后从内核复制到应用进程的缓冲，写数据过程相反。

根据系统调用的阻塞/非阻塞、同步/异步，UNIX 类系统的网络传输支持 5 种 IO 模型。

**阻塞 IO 模型**

应用使用 recvfrom 调用，请求从套接字读取数据，线程会阻塞，直到有数据可读，即进程缓冲有数据。如果没有数据可读，可能是因为数据还未从内核复制到进程缓冲，或者网卡还未收到数据。

这个 recvfrom 就是系统调用，它是一个 C 语言函数，毕竟操作系统也是由编程语言开发。

**非阻塞 IO 模型**

设置 Socket 为非阻塞，再次进行 recvfrom 调用，有数据直接返回，没数据也返回一个错误码，不会阻塞。这种方式在有数据可读之前，需要不停地轮询，比较浪费 CPU 资源。

**多路复用 IO 模型**

参考 Java 的 NIO + Selector 形式，那就是典型的多路复用，好处是节省资源，免去多线程/多进程的创建和维护。

多路复用是一种思想，操作系统提供多种系统调用实现：

* select

  这是一个阻塞调用，传入 FD 集合以及关心的状态，支持可读、可写、异常。每次调用，都要把 FD 集合复制到内核，不断轮询，直到有 FD 就绪，返回就绪的 FD 数量和状态。应用再根据返回，遍历 FD 集合，找到就绪的文件描述符。

  所谓 FD 集合，是一个长为 FD_SETSIZE 的 BitMap 类型，每一位表示一个 FD。所以，FD 号码的值以及单个进程可打开的 FD 数量，都受 FD_SETSIZE 限制，默认 1024，修改需要重新编译内核，可能降低网络效率。

* poll

  与 select 相似，改用链表而非 BitMap 存放 FD 集合，所以 FD 没有数量限制。

* epoll

  与 select 不同，epoll 使用一个 FD 管理需要监视的 FD，使用 epoll_create 创建 epoll 文件描述符，FD 没有数量限制。后续使用 epoll_ctl 向管理 FD 注册 FD，然后调用 epoll_wait，应用阻塞，内核检查所有 FD，把就绪的 FD 加入给定的队列，返回并唤醒应用，应用则使用预定的回调函数对 FD 进行操作。

  从这里看，epoll 比 select 强大许多，没有 FD 数量限制，注册时只需复制一次 FD 到内核，还可以直接操作就绪的 FD 而不用遍历所有。即便如此，也不是说 epoll 绝对高效，毕竟 epoll 需要很多回调。

**信号驱动 IO 模型**

开启 Socket 信号驱动，使用 sigaction 调用安装一个信号处理函数，当有数据可读，内核会产生一个 SIGIO 信号告知进程，进程再回调处理函数，可在回调内执行 recvfrom 读取数据。

与非阻塞相比，这种 IO 方式不需要不断地尝试。但是，如果有大量 IO 操作，存放信号的队列可能溢出，导致通知失败。对于 UDP，每个信号都表示一个数据报到达，而对于 TCP，产生信号的情况实在太多，处理时对每种情况都进行判断，会耗费大量资源。所以，这种 IO 方式较少使用。

**异步 IO 模型**

应用使用 aio_read 调用，告知内核在有数据复制到缓冲时执行给定的回调函数，这个调用立即返回。信号驱动是由内核通知何时可以进行 IO 操作，而异步 IO 是由内核通知何时 IO 操作完成。
