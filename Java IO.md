# Java 输入/输出流（IO）

在 Java，可以从中读入一个字节序列的**对象**称作输入流，可以向其写入一个字节序列的**对象**称作输出流。这些字节的来源地和目的地，可以是文件、网络、内存等。

`InputStream` 和 `OutputStream` 是字节输入/输出流的基础抽象，但字节数据不便于阅读，为此增加同级的基础抽象 `Reader` 和 `Writer`，它们的读写操作都基于两个字节的 `char`，即 Unicode 码元。

> 字符流在底层仍然是使用字节流，它只不过把字节序列转换为字符而已。

Java 拥有一个庞大的流家族，其数量超过 60 个，下面是一些常见的 IO 类：

```
|-- InputStream：字节输入流
	|-- ByteArrayInputStream：从数组读
	|-- FileInputStream：从文件读
	|-- PipedInputStream：从管道流读，连接PipedOutputStream
	|-- SequenceInputStream：按序合并多个字节输入流
	|-- ObjectInputStream：对象输入流
	|-- FilterInputStream：装饰器
		|-- BufferedInputStream：提供缓冲
		|-- DataInputStream：读基本数据类型，与DataOutputStream搭配使用
		|-- PushbackInputStream：能把已读的字节压回缓冲

|-- OutputStream：字节输出流
	|-- ByteArrayOutputStream：向数组写
	|-- FileOutputStream：向文件写
	|-- PipedOutputStream：向管道流写，连接PipedInputStream
	|-- ObjectOutputStream：对象输出流
	|-- FilterOutputStream：装饰器
		|-- BufferedOutputStream：提供缓冲
		|-- DataOutputStream：写基本数据类型，与DataInputStream搭配使用
		|-- PrintStream：格式化输出

|-- Reader：字符输入流
	|-- CharArrayReader：从数组读
	|-- BufferedReader：提供缓冲
		|-- LineNumberReader：可跟踪输入流行号
	|-- StringReader：从字符串读
	|-- PipedReader：从管道流读，连接PipedWriter
	|-- InputStreamReader：把InputStream转换为Reader，可指定字符编码
		|-- FileReader：从文件读
	|-- FilterReader：装饰器
		|-- PushBackReader：能把已读的字符压回缓冲
		
|-- Writer：字符输出流
	|-- CharArrayWriter：向数组写
	|-- BufferedWriter：提供缓冲
	|-- StringWriter：向StringBuffer写
	|-- PipedWriter：向管道流写，连接PipedReader
	|-- OutputStreamWriter：把OutputStream转换为Writer，可指定字符编码
		|-- FileWriter：向文件写
	|-- FilterWriter：装饰器
    |-- PrintWriter：格式化输出
```

此外，还有 4 个相关的接口：

* `Closeable`：它是 `AutoCloseable` 的子接口，所以能用在 *try-with-resource* 语句，不过它的 `close()` 方法只能抛出 `IOException` 异常。`InputStream`，`OutputStream`，`Reader` 和 `Writer` 都实现这个接口。
* `Flushable`：声明 `flush()` 方法，刷新缓冲。`OutputStream` 和 `Writer` 实现这个接口。
* `Readable`：声明 `reade(CharBuffer)` 方法，`CharBuffer` 是字符缓冲。`Reader` 实现这个接口。
* `Appendable`：声明 `append(char)` 和 `append(CharSequence)` 两个方法，`CharSequence ` 接口描述字符序列的属性。`Writer` 实现这个接口。                         

# 字节 InputStream 和 OutputStream

`InputStream ` 有一个读方法 `read()`，`OutputStream` 有一个写方法 `write()`，它们都可能阻塞线程，直到数据可读或被写出，这也是为什么把它们称作阻塞式 IO（Blocked IO，BIO）。

操作结束后，应调用 `close()` 关闭流，以释放占用的系统资源，输出流关闭的同时还会刷新输出缓冲。

> 为什么要有缓冲？
>
> 举例网络，每次传输，都要做一堆准备工作，只传输一个字节，效率太低，也很浪费。若是把一堆字节打包发送，可以减少传输次数，从而提高效率，节约资源。缓冲就是数据打包传输前的临时安置点。

> **java.io.InputStream **
>
> * `int read()`：读一个字节并返回，遇到结尾返回 -1。
>
>   `int read(byte b[])`：读多个字节，存入数组，返回读的字节数，遇到结尾返回 -1。
>
>   `int read(byte b[], int off, int len)`：读 *len* 个字节，存入数组 *off* 偏移后的位置。
>
> * `long skip(long n)`：跳过 *n* 个字节。
>
> * `int available()`：返回可读字节数。
>
> * `close()`：关闭流。
>
> * `boolean markSupported()`：是否支持标记。
>
>   `mark(int readlimit)`：标记指定位置，若小于已读的字节数则忽略。
>
>   `reset()`：重置到最后一个标记的位置。
>

> **java.io.OutputStream**
>
> * `write(int b)`：写一个字节。
>
>   `write(byte b[][, int off, int len])`：写出数组 *b* 的所有/指定范围的字节。
>
> * `flush()`：刷新缓冲。
>
> * `close()`：刷新缓冲并关闭流。

# 字符 Reader 和 Writer

任何信息在计算机都存为 0 和 1 的 bit 序列，字节流读入这些序列，并把它们解释为 btye 字节。如果使用字符流，并指定正确字符编码，那么 Java 程序就能把 bit 序列解释为可读的字符。

`Reader` 和 `Writer` 是字符流的抽象，它们也有 `read()` 和 `write()` 方法，不过是以 `char` 为基本单位。

> **java.io.Reader**
>
> * `int read()`：读一个字符并返回，遇到结尾返回 -1。
>
>
> * `int read(char cbuf[][, int off, int len])`
>
> * `long transferTo(Writer out)`

> **java.io.Writer**
>
> * `write(int c)`
>
>
> * `write(char cbuf[][, int off, int len])`
> * `write(String str[, int off, int len])`
> * `Writer append(char c)`
> * `Writer append(CharSequence csq[, int start, int end])`

`append()` 和 `write()` 的不同：

* `append()` 可以写出 `null`，值是 "null"。
* `append()` 返回 `writer` 对象，多个 `append()` 调用可以连接。
* `append()` 的参数是 `char`，而 `write()` 是 `int`。

# 组合输入/输出装饰器

Java IO 大量使用装饰器设计模式，可以把 IO 流分为具体流和装饰器。具体流基于数据源或数据汇创建，只有基本和常用的读写功能。装饰器不能单独使用，必须在内部封装一个流，以此为这个流提供额外的功能。

使用 `BufferedInputStream` 为 `FileInputStream` 提供缓冲，提高读文件的效率。

```
BufferedInputStream in = new BufferedInputStream(new FileInputStream("/text.txt"));
```

装饰器的数据源和数据汇由被封装的流决定，下面把数组作为源。

```
BufferedInputStream in = 
		new BufferedInputStream(new ByteArrayInputStream(new byte[]{1, 2, 3}));
```

装饰器可以相互组合，叠加功能。

```
DataInputStream in = 
		new DataInputStream(
				new BufferedInputStream(new FileInputStream("/text.txt")));
```

不同场景，对 IO 的需求可能不同，若为每种可能的功能组合专门设计一个类，IO 类的数量将几何增长。使用装饰器设计模式，把 IO 不同的功能拆开，使之能随意组合，灵活实现。这节省了大量代码，也方便管理和使用。

理解装饰器之后，我觉得 Java IO 便没有什么难点，剩下的就只是熟悉各种 API 而已。

# 文件输入与输出

## 字节流

`FileInputStream` 和 `FileOutputStream` 提供一个以磁盘文件作为源或汇的输入/输出字节流。

> **java.io.FileInputStream**
>
> * `FileInputStream(String name)`
>
>   `FileInputStream(File file)`
>
>   `FileInputStream(FileDescriptor fdObj)`
>
>   构造器，*name* 是文件的完整路径或相对路径。

> **java.io.FileOutputStream**
>
> * `FileOutputStream(String name[, boolean append])`
>
>   `FileOutputStream(File file[, boolean append])`
>
>   `FileOutputStream(FileDescriptor fdObj)`
>
>   构造器，*append* 设置追加，默认不追加会删除已有的同名文件。

## 字符流

`FileReader` 和 `FileWriter` 提供一个以文件作为源或汇的输入/输出字符流。查看源码，可以发现这两个流其实是使用 `FileInputStream`，`FileOutputStream` 构造的 `InputStreamReader`，`OutputStreamWriter`。

> **java.io.FileReader**
>
> * `FileReader(String fileName[, Charset charset])`
>
>   `FileReader(File file[, Charset charset])`
>
>   `FileReader(FileDescriptor fd)`
>
>   构造器。

> **java.io.FileWriter**
>
> * `FileWriter(String fileName[, boolean append])`
>
>   `FileWriter(File file[, boolean append])`
>
>   `FileWriter(FileDescriptor fd)`
>
>   `FileWriter(String fileName, Charset charset[, boolean append])`
>
>   `FileWriter(File file, Charset charset[, boolean append])`
>
>   构造器。

## 其它方法

如果数据源与 *.class* 文件相同目录，可用 `Class::getResourceAsStream()` 获取文件的输入流，参数 *text.txt* 表示文件路径，它以 *.class* 文件的目录作为根。

```
InputStream in = Test.class.getResourceAsStream("text.txt");
```

反斜杠 *\\* 在 Java 字符串有转义意思，因此 Windows 风格的路径以 *\\\\* 作为分隔符，不过 Windows 多数的系统调用也把 */* 解释为路径分隔符。

# 字符文本输入与输出

虽然字节流的效率更高，但字符流更适合人类阅读。

## 编码方式和转换字节流

Java 在内部使用 UTF-16 编码数据，但数据源、数据汇可能使用其它的编码方式，为此，输入字符流需要知道源的字符集，输出字符流需要指定汇接受的字符集。如不指定，则使用平台的默认编码方式。

> 静态方法 `Charset.defaultCharset()` 返回平台的默认编码。

`InputStreamReader` 和 `OutputStreamWriter` 可以使用指定的编码，把字节流转换为字符流。

> **java.io.InputStreamReader**
>
> * `InputStreamReader(InputStream in)`
>
>   `InputStreamReader(InputStream in, String charsetName)`
>
>   `InputStreamReader(InputStream in, Charset cs)`
>
>   `InputStreamReader(InputStream in, CharsetDecoder dec)`
>
>   构造器，第二个参数指定数据源的编码方式，默认使用平台默认编码。
>
> * `String getEncoding()`

> **java.io.OutputStreamWriter**
>
> * `OutputStreamWriter(OutputStream out)`
>
>   `OutputStreamWriter(OutputStream out, String charsetName)`
>
>   `OutputStreamWriter(OutputStream out, Charset cs)`
>
>   `OutputStreamWriter(OutputStream out, CharsetEncoder enc)`
>
>   构造器，第二个参数指定数据汇的编码方式，默认使用平台默认编码。
>
> * `String getEncoding()`

## 文本输出 PrintWriter

打印流 `PrintWriter` 能以文本格式输出字符串和数字，还可以直接向文件输出，它的 API 与 `System.out` 非常相似。打印流自带缓冲功能，不需要附魔，且有可选的自动刷新缓冲功能。

> **java.io.PrintWriter**
>
> * `PrintWriter (Writer out[, boolean autoFlush])`
>
>   `PrintWriter(OutputStream out[, boolean autoFlush[, Charset charset]])`
>
>   `PrintWriter(File file[, Charset charset])`
>
>   构造器，*autoFlush* 开启自动刷新，调用 `println()` 会刷新缓冲。
>
> * `print(boolean b)`，`println(boolean x)`，`println(String x)`，`printf(String format, Object ... args)`，`format(String format, Object ... args)`

## 文本读入 BufferedReader

`BufferedReader` 用于处理文本输入，其使用方式如下所示：

```
InputStream inputStream = ...;
try (var in = new BufferedReader(new InputStreamReader(inputStream, "UTF-8"))) {
    String line;
    while ((line = in.readLine()) != null) {
        do something with line
    }
}
```

> **java.io.BufferedReader**
>
> * `BufferedReader(Reader in[, int size])`：构造器，*size* 是缓冲大小，单位字符。
>* `String readLine()`：按行读文本，遇到结尾返回 `null`。

# 读写基本类型数据

接口 `DataInput` 和 `DataOutput` 定义许多形如 `read***()`、`write***()` 的方法，以二进制格式读写数据。

`DataInputStream` 读数据的顺序，必须与 `DataOutputStream` 写数据的顺序一致。

> **java.io.DataInputStream**
>
> * `DataInputStream(InputStream in)`：构造器。
> * `readBoolean()`，`readByte()`，`readUnsignedByte()`，`readShort()`，`readUnsignedShort()`，`readChar()`，`readInt()`，`readLong()`，`readFloat()`，`readDouble()`，`readUTF()` ...

> **java.io.DataOutputStream**
>
> * `DataOutputStream(OutputStream out)`：构造器。
> * `writeBoolean(boolean v)`，`writeByte(int v)`，`writeShort(int v)`，`writeChar(int v)`，`writeInt(int v)`，`writeLong(long v)`，`writeFloat(float v)`，`writeBytes(String s)`，`writeChars(String s)`，`writeUTF(String str)` ...

`writeUTF()` 输出的字符串，只能以 `readUTF()` 读取，因为它们使用 Java 修订过的 UTF-8 编码字符。

# 字节打印流

字节打印流 `PrintStream` 与 `PrintWriter` 功能相同，接口相似。不同的是，`PrintStream` 只能使用平台默认的编码方式。

设置自动刷新，`PrintStream` 会在以下情况刷新缓冲：

* 打印字节数组。
* 打印换行符，即 `print("\n")` 或 `println()`。

> **java.io.PrintStream**
>
> * `PrintStream(OutputStream out[, boolean autoFlush[, Charset charset]])`：构造器。

# 标准 IO

系统类 `java.lang.System` 提供 3 个静态常量：

* 标准输入流 `System.in`：`InputStream` 类型，默认数据源是键盘。
* 标准输出流 `System.out`：`PrintStream` 类型，默认数据汇是控制台。
* 标准错误输出流 `System.err`：`PrintStream` 类型，默认数据汇是控制台。

标准 IO 由 JVM 启动时自动创建，它们始终处于打开状态，可用 `System` 的方法重定向标准 IO 的源或汇。

> **java.lang.System**
>
> * `static setIn(InputStream in)`
>
> * `static setOut(PrintStream out)`
>
> * `static setErr(PrintStream err)`
>
>   重定向。

# 随机读写文件

前面的文件流只能按序读写文件数据，而 `RandomAccessFile` 可在任何时刻改变当前文件的读写位置，被随机访问的文件不会被删除。`RandomAccessFile` 输出的数据会覆盖原内容。

> `RandomAccessFile` 可用于处理大文件，比如分片传输。

> **java.io.RandomAccessFile**
>
> * `RandomAccessFile(String name, String mode)`
>
>   `RandomAccessFile(File file, String mode)`
>
>   构造器，*name* 是文件路径，*mode* 是访问模式：*r* 只读，*rw* 读写。
>
> * `long getFilePointer()`：返回当前读写位置。
>
> * `void seek(long pos)`：移至距离文件开始 *pos* 偏移的位置，单位字节。
>
> * `long length()`：返回文件长度，单位字节。
>
> * `void close()`

# 对象的序列化

Java 支持把对象转换为字节流，或把字节流还原为对象，从而传输对象或把对象持久化。

## 序列化对象

使用 `ObjectOutputStream`、`ObjectInputStream` 输出/读入对象。对象的类、类的签名，以及超类的非静态和非 `transient` 的属性都会被持久化。被序列化的对象，必须实现 `Serializable` 标记接口。

> **java.io.ObjectOutputStream**
>
> `ObjectOutputStream(OutputStream out)`：构造器。
>
> `writeObject(Object obj)`：输出对象。

> **java.io.ObjectInputStream**
>
> `ObjectInputStream(InputStream in)`：构造器。
>
> `Object readObject()`：读取对象。

## 序列化过程

序列化输出对象，然后读回，将获得原对象的深拷贝。

写对象过程：

* 对遇到的每一个对象引用都关联一个序列号。
* 对每个对象，第一次遇见时，把它写到输出流。
* 若某个对象已被保存，则使用与之关联的序列号进行记录。

读对象过程：

* 对每个对象，第一次遇见时，进行构建，使用流的数据进行初始化，然后关联序列号和新对象。
* 遇到序列号记录时，用新构建对象的引用进行替代。

## 禁止序列化 transient

使用 `transient` 关键字修饰的属性，将在序列化对象时被忽略。

```
private transient String desc;
```

## 版本管理 serialVersionUID

序列化保存的对象，可能和当前类的定义不一致。只要类的定义发生过改变，它的 SHA 指纹也会改变，对象输入流拒绝加载指纹不同的对象数据。

为适配不同版本的对象数据和类定义，可以手动指定类的 SHA 指纹。

```
public static final long serialVersionUID = -6849794470754667710L;
```

反序列化时，对象输入流检查两边的属性，若发现名字相同但类型不同的属性，反序列化失败。对象数据包含的当前类没有的属性，直接忽略。对于对象数据缺少的属性，设为默认值。

使用 JDK 工具查看类的 SHA 指纹。

```
serialver java.lang.String
```

## 自定义序列化

在类中定义以下两个方法，可修改这个类的实例的序列化行为：

* `private void readObject(ObjectInputStream in)`
* `private void writeObject(ObjectOutputStream out)`

此后，调用 `readObject()`/`writeObject()` 方法，将不执行默认的行为，而是执行这两个方法。若在方法中还想执行默认行为，可在方法内调用 `defaultReadObject()`/`defaultWriteObject()` 方法。

```
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException
{
	// custom action
    in.defaultReadObject(); //default action
    // custom action
}

private void writeObject(ObjectOutputStream out) throws IOException 
{
	// custom action
    out.defaultWriteObject(); //default action
    // custom action
}
```

## 序列化单例类型

序列化单例对象时，应当加倍小心。假如现在遇到某个古老的枚举类型，定义如下：

```
public class Orientation implements Serializable
{
    public static final Orientation HORIZONTAL = new Orientation(1);
    public static final Orientation VERTICAL = new Orientation(2);
	
	private int value;

    private Orientation(int v) {value = v;}
}
```

输出枚举实例，然后读回，得到一个新的实例，这使枚举实例不是单例。可在类中定义 `readResolve()` 方法，它的返回值就是 `readObject()` 反序列化的结果。

```
protected Object readResolve() throws ObjectStreamException 
{
    if (value == 1) return Orientation.HORIZONTAL;
    if (value == 2) return Orientation.VERTICAL;
    throw new ObjectStreamException();
}
```

# 文件 Path，Files，File

JDK 7 引入 `Path` 接口和 `Files` 类，用于管理、操作文件，它们比 `File` 类强大的多。

## 路径 Path

`Path` 表示文件路径，文件不必真实存在。

> **java.nio.file.Path**
>
> * `Path resolve(Path|String other)`
>
>   若 *other* 是绝对路径，直接返回，否则，使用 *this* 连接 *other* 再返回。
>
> * `Path resolveSibling(Path|String other)`
>
>   若 *other* 是绝对路径，直接返回，否则，使用父路径连接 *other* 再返回。
>
> * `Path relativize(Path other)`：返回相对路径，*this* 连接返回等于 *other*。
>
> * `Path normalize()`：移除冗余部件。
>
> * `Path toAbsolutePath()`：返回绝对路径。
>
> * `Path getParent()`：返回父路径，没有则返回 `null`。
>
> * `Path getFileName()`：返回尾部件，没有则返回 `null`。
>
> * `Path getRoot()`：返回根部件，没有则返回 `null`。
>
> * `File toFile()`：返回对应的 `File` 对象。
>

> **java.nio.file.Paths**
>
> * `static Path get(String first, String... more)`：连接多个路径，创建 `Paht` 实例。
> * `static Path get(URI uri)`：把 `URI` 转为 `Path` 类型。 

## 文件 Files

`Files` 用于操作文件，毕竟创建、删除、读/写。

> **java.nio.file.Files**
>
> 获取 IO 流
>
> * `static InputStream newInputStream(Path path, OpenOption... options)`
> * `static OutputStream newOutputStream(Path path, Charset cs, OpenOption... options)`
>
> 读写文件
>
> * `static byte[] readAllBytes(Path path)`
>* `static Path write(Path path, byte[] bytes, OpenOption... options)`
> 
>创建文件
> 
> * `static Path createFile(Path path, FileAttribute<?>... attrs)`
> * `static Path createDirectory(Path dir, FileAttribute<?>... attrs)`
> * `static Path createDirectories(Path dir, FileAttribute<?>... attrs)`
>* `static Path createTempDirectory(String prefix, FileAttribute<?>... attrs)`
> 
>删除文件
> 
> * `static void delete(Path path)`：文件不存在则抛出异常。
> * `static boolean deleteIfExists(Path path)`：文件不存在则返回 `flase`。
> 
>复制/移动
> 
>* `static Path copy(Path source, Path target, CopyOption... options)`
> * `static Path move(Path source, Path target, CopyOption... options)`
> 
>遍历目录
> 
>* `static Stream<Path> list(Path dir)`：不进入子目录。
> * `static Stream<Path> walk(Path start, FileVisitOption... options)`：进入子目录。

## 文件 File

> **java.io.File**
>
> 构造器
>
> * `File(URI uri)`
>
>   `File(String pathname)`
>
>   `File(String parent, String child)`
>
> 文件信息
>
> * `String getParent()`
> * `boolean isFile()`
> * `boolean isDirectory()`
> * `String getName()`
> * `boolean exists()`
> * `boolean canRead()`
> * `boolean canWrite()`
>
> 文件操作
>
> * `boolean createNewFile()`
> * `boolean renameTo(File dest)`
> * `boolean delete()`
> * `boolean mkdir()`
> * `boolean mkdirs()`
>
> 访问目录
>
> * `String[] list()`：不进入子目录。
> * `File[] listFiles()`：进入子目录。
