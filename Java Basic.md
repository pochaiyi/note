# final

## 修饰类：类不能被继承

此时，`final` 类中的所有方法都隐式为 `final`，因为它不存在子类，也就不可能被重写。

`final` 类的属性不会被限制，它可以作任何修改，除非也被 `final` 修饰。

**如何对 final 类进行拓展呢？**

很明显，只能使用组合的方式，但这其实相当于创建一个新类。

## 修饰方法：方法不能被重写

`private` 方法隐式为 `final`，子类虽然拥有父类的私有属性和方法，但不能访问，所以也无法重写它。

如果子类定义有一个方法，其签名和返回类型都与父类的一个 `private` 方法相同，这是合法的。子类并没有重写父类的方法，而是定义一个自己特有的方法。

`final` 方法是可以重载的，这很显然。

## 修饰属性：常量

JVM 不会为 `final` 字段赋默认值，使用没有显式初始化的 `final` 变量会抛出异常。

常量只能赋值一次，静态常量可以在定义语句、静态代码块赋值，普通常量可以在定义语句、代码块、构造器中赋值。

`final` 修饰的引用变量，赋值后不能修改引用其他对象，但可以修改其引用对象的状态。

**所有的 final 修饰的变量都是编译期常量吗?**

不是，比如下例代码，常量 `k` 就是运行时常量：

```
final int k = new Random().nextInt();
```

## 修饰参数：局部常量

Java 还允许使用 `final` 修饰方法参数，这意味着不能在方法中修改参数变量的值或使它引用其他对象。

这个特性主要用来向匿名内部类传递数据。

## final 字段的重排序规则

对于 `final` 字段，编译器和处理器要遵守两个重排序规则：

* `final` 字段写：在构造器内对一个 `final` 字段的写入，与随后在构造器外把这个被构造对象的引用赋给一个变量，这两个操作之间不能重排序。

* `final` 字段读：初次读一个包含 `final` 字段的对象的引用，与随后初次读这个 `final` 字段，这两个操作之间不能重排序。

针对 `final` 引用类型，也有一条重排序规则：

* `final` 引用类型：在构造器内对一个 `final` 引用类型的字段的写入，与随后在构造器外把这个被构造对象的引用赋给一个变量，这两个操作之间不能重排序。

**这些规则与内存可见性有什么关系呢？**

`final` 字段写规则保证在构造器中对 `final` 字段的赋值操作完成之前，其对外是不可获取的。等到其它线程能够通过 `this` 引用读 `final` 字段时，它已经完成初始化。前提是 `this` 引用没有逃逸，否则其它线程就可能读到未完成初始化的 `final` 字段。

# ==，equals，hashCode

## ==，equals

对于基本数据类型，`==` 比较的就是值，而对于引用类型，比较的则是它们的引用对象的内存地址。

> Java 只有值传递，所以对于 `==` 来说，不管是基本数据类型还是引用类型，本质比较的都是值。只不过引用变量保存的值是对象的内存地址。

`equals` 是 `Object` 类的方法，用于判段两个对象是否相等。默认实现为：

```
public boolean equals(Object obj) {
    return (this == obj);
}
```

大部分时候我们会重写这个方法，用于比较对象的内部状态，而非对象的内存地址。

## hashCode

`hashCode` 返回对象的哈希码，它的默认实现将对象的内存地址转换为一个整数返回。

`hashCode` 应该与 `equals` 保持一致，即 `equals` 返回 `true` 的两个比较对象，它们的 `hashCode` 也必须相等。但反之则不是必须的，因为可能由于哈希函数的原因，导致不同的对象哈希码相同。

在 Java 应用程序运行期间，如果 `equals` 所使用的信息没有被修改，那么 `hashCode` 返回值也不应该改变。

# 异常 Exception

# 泛型 Generic

泛型是 Java 5 引入的新特性，它使程序可读性更高，也更安全，可以避免类型强制转换的错误。

## 泛型类

泛型类就是有一个或多个泛型参数的类：

```
public class A<T> {
    ...
}
```

泛型参数使用尖括号 `< >` 括起来，放在类名后面。可以有多个泛型参数，比如 `<T,U>`。

类定义的泛型参数，可以在整个类中用于指定属性、方法参数、方法返回值、局部变量等的类型，可以说能用在任何地方。

实例化时，可以用具体的类型替换泛型参数：

```
A<String> = A<String>A();
```

可以省略构造器中的泛型参数，Java 能根据变量声明进行类型推断：

```
A<String> = new<>A();
```

## 泛型方法

方法也可以定义泛型参数：

```
class A {
	public static <T> T getMiddle(T... a) {
		return a[a.length / 2];
	}
}
```

泛型参数使用尖括号 `< >` 括起来，放在访问修饰符后面，返回类型前面。它也可以有多个泛型参数。方法定义的泛型参数，只能用于指定当前方法的参数、返回值、局部变量的类型。

调用泛型方法时，可以使用尖括号 `< >` 指定具体的类型，放在方法名前面：

```
String middle = A.<String>getMiddle("John", "Q", "Tony");
```

Java 可以根据接收返回结果的变量，以及实参进行类型推断，省略泛型参数：

```
String middle = A.<String>getMiddle("John", "Q", "Tony");
```

## 类型参数的限定

有时我们对泛型有一些要求，比如：某种类型的子类型、实现某个接口。这可以通过类型参数的限定来实现。

下例的泛型变量声明，表示在实际运行中，泛型的实际类型必须实现 `Comparable` 接口。

```
<T extends Comparable>
```

`extends` 关键词后的限定类型可以是类，也可以是接口。类型参数可以有多个限定，使用 `&` 连接：

```
<T extends Comparable & Serializable>
```

如果有多个限定，最多只能有一个限定是类，且类限定必须是限定列表中的第一个。

## 泛型的继承规则

通常，无论类 `S` 与类 `T` 有什么关系，`A<S>` 与 `A<T>` 都没有任何关系。比如，`A<Employee>` 并不是 `A<Manager>` 的子类型。

任何参数类型都总是可以转换为原始类型。例如，`A<String>` 可以转换为 `A` 类型。这在与遗留代码交互时很重要，因为之前没有泛型。但是，原始类型 `A` 没有类型限制，而它引用的 `A<String>` 对象具有类型限制，这很容易发生类型错误。

泛型类可以拓展或实现其它的泛型类。就这一点而言，它们与普通类没有区别。例如，`ArrayList<T>` 实现 `List<T>` 接口。`ArrayList<Manager>` 是 `List<Manager>` 的子类，但 `ArrayList<Manager>` 与 `List<Employee>` 没有什么关系。

## 类型擦除

JVM 并没有泛型类型的对象，所有对象都是普通类。无论何时定义一个泛型类型，都会自动提供一个相应的原始类型。这个原始类型的名字就是去掉泛型参数后的泛型类型名。泛型变量会被擦除，并替换为其限定类型，没有限定类型则替换为 `Object`。如果有多个限定类型，则使用第一个限定。

## 通配符类型

严格的泛型系统使用起来不太方便，为此设计通配符类型。在通配符类型中，泛型参数表示一群类型，他能匹配所有满足条件的类型。

### 上界限定通配符 ? extends

下例的泛型定义匹配所有泛型参数是 A 的子类型的泛型：

```
<? extends A>
```

上界限定通配符只能用于方法的返回类型，不能用于参数类型。所以使用上界限定通配符的集合，只能读，不能存。

### 下界限定通配符 ? spuer

下例的泛型定义匹配所有泛型参数是 A 的超类型的泛型：

```
? super A
```

与上界限定通配符相反，下界限定通配符只能用于方法的参数类型，不能用于返回类型。而且，传入的参数类型只能是 A 或 A 的子类型。所以，使用下界限定通配符的集合，可以写，但读的返回类型都是 `Object`。

### 无限定通配符 ？

`?` 匹配任意类型。它返回 `Object` 类型，但不能用于方法的参数类型。

# 枚举 Enumeration

枚举类是指实例数目有限的类。

## 传统枚举

枚举类的传统设计模式为：

* 将构造器定义为 `private`；
* 提供一些 `public static final` 常量，每个常量引用一个枚举类的实例；
* 如果需要，提供静态工厂方法，允许根据特定参数获得与之匹配的实例。

此外，需要注意。如果这个枚举类型是可序列化的，那么必须提供 `readResolve()` 方法。这能避免在反序列化时创建一个新的实例。该建议同样适用于单例类。

下面是一个传统的枚举类 `Gender`：

```
public class Gender implements Serializable {
    public static final Gender FEMALE = new Gender('F', "Female");
    public static final Gender MALE = new Gender('M', "Male");

    private final Character sex;
    private final transient String desc;

    public Character getSex() { return sex; }

    public String getDesc() { return desc; }

    private Gender(Character sex, String desc) {
        this.sex = sex;
        this.desc = desc;
    }

	//反序列化
    private Object readResolve() {
        if (this.sex == 'F') {
            return Gender.FEMALE;
        } else if (this.sex == 'M') {
            return Gender.MALE;
        } else {
            return null;
        }
    }
}
```

## Enum 枚举

根据传统枚举类的设计方式，枚举类有以下特点：实例有限且都是静态常量，提供获取这些实例的工厂方法。

为提高代码的可重用性，JDK5 提供抽象的 `java.lang.Enum` 枚举类，它的声明如下：

```
public abstract class Enum<E extends Enum<E>>
```

`Enum` 是抽象类，`<E extends Enum<E>>` 泛型标记表示 `Enum` 类拥有的静态常量也是 `Enum` 类型或其子类型。现在，用户自定义枚举类只需继承 `Enum` 即可，比如 `Gender` 枚举类：

```
public class Gender extends Enum<Gender> {
    public static final Gender FEMALE;
    public static final Gender MALE;
        
    ...
}
```

为进一步简化编程，JDK5 提供专门的用于声明枚举类型的关键字 `enum`，上面的代码等价于：

```
public enum Gender{FEMALE, MALE}
```

`Enum` 类具有许多非抽象方法：

> **java.lang.Enum**
>
> * `final int compareTo(E o)`：比较当前枚举常量与指定对象的顺序。
> * `final Class<E> getDeclaringClass()`：返回表示当前枚举常量的枚举类型的 `Class` 对象。
> * `final String name()`：返回当前枚举常量的名称。比如 `Gender.FEMALE` 的 `name()` 方法返回 `"FEMALE"`。枚举名称默认是常量名。
> * `final int ordinal()`：返回当前枚举常量的序数，即它在枚举声明中的位置，初始序数为 0.
> * `String toString()`：返回枚举常量的名称。
> * `static <T extends Enum<T>> T valueOf(Class<T> enumType, String name)`：根据指定的枚举类型和名称，返回响应的枚举常量实例。
> * `static Enum[] values()`：以数组形式返回该枚举类型的所有枚举常量实例。

**枚举类型的构造方法**

`Enum` 类有属性 `String name` 和构造方法 `protected Enum(String name, int ordinal)`。`enum Gender{FEMALE, MALE}` 语句会调用这个构造方法，将变量名 `FEMALE` `MALE` 赋值给 `name` 属性，并设置相应的序数值。

在自定义的枚举类中，也可以定义自己的构造方法和属性。这个构造方法必须是 `private` 类型的。然后，枚举类在构造实例时会调用你自定义的构造器，而非 `Enum(String name, int ordinal)`。

下例，`Gender` 类有一个 `desc` 属性，在构造器中为 `desc` 赋值：

```
class GenderTest {
    enum Gender {
        MALE("男"),
        FEMALE("女");

        private final String desc;

        private Gender(String desc) { this.desc = desc; }

        public String getDesc() { return this.desc; }
    }

    public static void main(String[] args) {
        System.out.println(Gender.MALE.getDesc());
    }
}
```

注意，此时定义的枚举常量 `FEMALE` 和 `MALE` 在结尾处必须标注 `;`，常量间以 `,` 分隔。

**EnumSet 和 EnumMap**

Java 还为 `Enum` 提供两个适配器：

* `java.util.EnumSet`：把 `Enum` 枚举类型转换为集合类型
* `java.util.EnumMap`：把 `Enum` 枚举类型转换为映射类型。枚举常量以 `Key` 的形式存放在 `Map` 中。

> **java.util.EnumSet**
>
> * `static <E extends Enum<E>> EnumSet<E> allOf(Class<E> elementType)`：把指定枚举类型的所有常量实例存放到一个 `EnumSet` 集合，并返回这个集合。

> **java.util.EnumMap**
>
> * `EnumMap(Class<K> keyType)`：构造器，`keyType` 参数指定存放的枚举类型。
> * `public V put(K key, V value)`：存放映射，`key` 是枚举常量实例。

# 字符串 String

简单理解，Java 字符串就是 Unicode 字符序列。

Java 字符串不是基本类型，而是 JDK 的标准库类 `java.lang.String`。每个用双引号括起来的字符串都是 `String` 类的一个实例。

`String` 类有非常多的方法，可怕的是，这些方法都很用且使用频率很高。

> **java.lang.String**
>
> * `int length()`：返回字符串代码单元数量。
>
> * `boolean isEmpty()`：是否为空串，即长度为 0。
>
> * `boolean isBlank()`：是否只含有空格。
>
> * `boolean equals(Object anObject)`
>
> * `boolean equalsIgnoreCase(String anotherString)`
>
>   判断当前字符串是否与指定对象相等，后者忽略字符串大小写。
>
>
> * `compareTo(String anotherString)`
>
> * `int compareToIgnoreCase(String str)`
>
>   判断当前字符串是否与指定对象相等，后者忽略字符串大小写。
>
>
> * `String replace(CharSequence old, CharSequence new)`
>
> * `String replace(char old, char new)`
>
>   返回一个新字符串，用 new 字符串替换 old 子串。
>
>
> * `String substring(int beginIndex)`
>
> * `String substring(int beginIndex, int endIndex)`
>
>   返回一个新字符串，子串 `[beginIndex, endIndex)`。
>
>
> * `String toLowerCase()`
>
> * `String toUpperCase()`
>
>   返回一个新字符串，将原来的所有字母改为小写或大写。
>
>
> * `String trim()`
>
> * `String strip()`
>
>   返回一个新字符串，删除原来头部和尾部小于等于 U+0020 的字符或空格（`strip`）。
>
>
> * `static String join(CharSequence delimiter, CharSequence... elements)`：返回一个新字符串，用指定的定界符连接所有参数。
>
> * `String repeat(int count)`：返回一个新字符串，将当前的字符串重复 count 次。
>
> * `boolean startsWith(String prefix)`
>
> * `boolean endsWith(String suffix)`
>
>   判段当前字符串是否以指定的子串开始或结尾。
>
>
> * `int indexOf(String str)`
>
> * `int indexOf(String str, int fromIndex)`
>
> * `int indexOf(int ch)`
>
> * `indexOf(int ch, int fromIndex)`
>
>   返回与字符串 `str` 或码点 `ch` 匹配的第一个子串的开始位置，从索引 0 或 fromIndex 开始。如果不存在匹配项，则返回 -1。
>
>
> * `int lastIndexOf(String str)`
>
> * `int lastIndexOf(String str, int fromIndex)`
>
> * `int lastIndexOf(int ch)`
>
> * `int lastIndexOf(int ch, int fromIndex)`
>
>   返回与字符串 `str` 或码点 `ch` 匹配的最后一个子串的开始位置，从索引 0 或 fromIndex 开始。如果不存在匹配项，则返回 -1。
>
>
> * `char charAt(int index)`：返回指定位置的代码单元。
>
> * `int codePointAt(int index)`：返回从给指定置开始的码点。
>
> * `int offsetByCodePoints(int index, int Offset)`：返回从 index 开始，Offset 个码点后的码点位置。
> * `int codePointCount(int beginIndex, int endIndex)`：返回 `[beginIndex, endIndex-1]` 区间的码点数量。 
>
> * `IntStream codePoints()`：返回字符串的码点流，可以调用 `toArray()` 将它们转换为数组。

## 字符串的拼接 +、+=

Java 允许使用 `+`、`+=` 运算符拼接字符串，这也是 Java 中唯二的两个重载运算符。

> 使用 `+`、`+=` 拼接字符串，实质是创建 `StringBuilder` 对象进行操作。
>
> 但需要注意，JVM 不会重用前面创建的 `StringBuilder` 对象，每次拼接，都会创建新对象，所以要避免在循环中使用 `+`、`+=` 拼接字符串。因为这会占用大量堆内存，还会增大 GC 压力。

当将一个字符串与一个非字符串拼接时，后者会转换为字符串（字面量或者是 `toString()` 的结果）。

## 字符串相对性

判断字符串的值是否相等，应该使用 `equals` 方法。不要使用 `==`，这个运算符比较的是两个变量指向的内存地址。

除 `equals` 方法，`String` 类还实现 `Comparable` 接口，所以还可以使用 `compareTO` 判断相等性。

## 空串和 null

空串 `""` 是一个字符串对象，只是它不包含任何字符。

而 `null` 表示不指向任何对象，对这个变量调用任何方法都会抛出空指针异常。

## 不可变对象和字符串常量池

**字符串不可变**

`String` 类在内部使用 `char` 数组存储字符串，它被 `private final` 修饰，且不提供任何修改方法。此外，`String` 类被 `final` 修饰，这避免子类破坏它的不可变性。

> JDK 9 开始，`Java` 使用 `byte` 数组存储字符串。

对字符串的截取子串、改变大小写等操作，获取的都是新的对象。

`String` 为什么设计为不可变？主要是保证安全，比如存入 `HashSet` 的两个字符串不能修改为相等。

**字符串常量池**

JVM 为提升性能和减少内存消耗，使用一块专门的区域 "字符串常量池" 来保存字符串对象的引用，字符串对象仍然在堆。字符串常量池的字符串可以被共享。

下例代码，首先查看字符串常量池是否有 "abc" 对象，如果有则返回，否则在堆创建字符串对象并将引用保存到字符串常量池。

```
String str = "abc";
```

下例代码，首先在堆创建 `str` 字符串对象，此时还未初始化，然后查看字符串常量池是否有 "abc" 对象，有则使用它初始化 `str`，否则在堆创建字符串对象 "abc" 并保存引用到字符串常量池，然后再使用它初始化 `str`。所以这句代码可能会创建两个字符串对象。无论字符串常量池原本是否有 "abc"，这句代码都会创建 `str` 对象。

```
String str = new String("abc");
```

**JVM 优化：常量折叠**

代码什么时候会共享字符串常量池的对象呢？

字符串常量经常会被共享，但提取子串、拼接等操作，获取的总是新的字符串对象。这是为什么呢？

对于编译期可以确定值的字符串，即字符串常量，JVM 会将其保存到字符串常量池。并且，由字符串常量拼接得到的字符串在编译阶段也会被存到字符串常量池。

比如，`String str = "str" + "ing";` 会被编译器优化为 `String str = "string";`。

字符串引用的值在编译期无法确定，编译器不能对其进行优化。前面也提到，`+`、`+=` 拼接字符串其实是在创建 `StringBuilder` 对象。而提取子串，就是纯粹的使用 `char` 数组的部分内容构建新字符串。

## StringBuffer、StringBuilder

`StringBuilder` 和 `StringBuffer` 都继承 `AbstractStringBuilder`。`AbstractStringBuilder` 与 `String` 相同，也使用 `char` 数组存储字符串，但没有使用 `private final` 修饰，且提供许多修改字符串的方法。

对于需要频繁修改的字符串，如果使用 `String` 类，则会创建大量的字符串对象，耗时又浪费空间。这时就可以选择使用 `StringBuilder` 和 `StringBuffer`。

> `StringBuilder` 在 Java 5 引入，它的前身 `StringBuffer` 是线程安全的，方法都使用 `synchronized` 同步，效率较低。
>
> 如果所有操作都在单线程中进行，建议使用 `StringBuilder`，这两个类的 API 完全相同。

> **java.lang.StringBuilder**
>
> * `StringBuilder()`
>
> * `StringBuilder(int capacity)`
>
> * `StringBuilder(CharSequence seq)`
>
>   构造器。`capacity` 设置初始容量，`seq` 指定初始字符串。
>
>
> * `int length()`：返回构建器代码单元个数。
>
> * `StringBuilder append(CharSequence s)`
>
> * `StringBuilder append(char c)`
>
>   各种 `append` 方法。
>
>
> * `void setCharAt(int i, char c)`：将第 i 个代码单元设置为 c。
>
> * `StringBuilder insert(int offset, char c)`
>
> * `StringBuilder insert(int Offset, CharSequence s)`
>
>   在 Offset 位置插入代码单元 c 或字符序列 s。
>
>
> * `StringBuilder delete(int start, int end)`：删除 `[start, end-1]` 区间的代码单元。
>
> * `String toString()`：返回 `char` 数组保存的字符串。

# 数组 Array

数组是相同类型的数据的集合，这些数据可以是基本类型，也可以是引用类型，但一个数组只能存放一种类型的数据。数组也是 Java 对象。

## 声明变量，创建和初始化数组

**数组变量的声明**

一维数组变量声明：

```
int scores[];
String[] names;
```

二维或多维数组变量声明：

```
int[][] a;
int[] a[];
int a[][];
```

> 声明数组变量时，不能指定数组长度。`int[10] scores` 这种格式是非法的。

**创建数组对象**

数组对象使用 `new` 进行创建：

```
int[] scores = new int[100];
```

必须指定数组的长度，且一旦创建，数组长度就不能再改变。

数组长度可以是 0，此时数组中一个元素也没有。

`new` 语句会为数组的每个元素赋默认值。

**初始化数组**

数组被创建后，可以根据索引（从 0 开始）显示访问它的每个元素：

```
int[] a = new int[5];
a[1] = 11;
```

或者，可以按如下方式创建并初始化数组：

```
int[] a = new int[]{1, 2, 3, 4, 5};
int[] b = {1, 2, 3, 4, 5}; // 这种格式只能出现在定义语句，不能用于赋值语句
```

**数组长度**

数组对象拥有一个属性 `public final length`，表示数组的长度。

## 多维不规则数组

Java 支持多维数组，并且支持不规则数组。

```
int[][] persons = new int[3][];
persons[0] = new int[1];
persons[1] = new int[2];
persons[2] = new int[3];
```

## 数组参数和可变长参数

假设 `max` 方法要从一组 `int` 类型数据中找到最大值，而这组数据的数目不固定。比较呆板的解决办法是实现多个重载方法，灵活一点，可以将方法参数定义为数组：

```
public int max(int[] datas){
	...
}
```

为简化编程，JDK5 增加一个新特性：用符号 `...` 来声明数目可变的方法参数。可变参数适用于参数数目不确定，而类型确定的情况。前面的代码可以简化为以下形式：

```
public int max(int ... datas){
	...
}
```

`...` 符号位于参数类型和参数名称之间，前后有无空格都可以。

可变参数必须作为最后一个参数。

JVM 在运行时会为可变参数创建一个数组，`datas` 在方法内部其实是一个数组对象。所以，可变参数既可以接收多个参数，也可以接收一个数组类型。以下都是合法的调用：

```
max(1, 2, 3, 4);
max(new int{1, 2, 3, 4});
```

如果没有传入任何参数，JVM 会创建一个长度为 *0* 的数组。因此，在方法内访问可变参数永远不会抛出空指针异常。

## 数组工具类 Arrays

`Arrays` 是专门用于数组的工具类，它提供许多有用的静态方法。

> **java.util.Arrays**
>
> * 各种 `equals(Type[] a, Type[] b)`：比较两个数组是否相等。只有两数组元素数目相同，且对应位置的元素也都相同时，两数组才相等。
> * 各种 `void fill(Type[] a, Type val)`：向数组填充数据。
> * 各种 `void sort(Type[] a)`：按升序对数组元素排序。如果数组元素是引用类型，则采用自然排序。
> * 各种 `void parallelSort(Type[] a)`：开启多个线程，以并发的方式对数组元素进行排序。
> * 各种 `T binarySearch(Type[] a, Type key)`：按照二分查找算法，查找数组中值与 key 相等的元素。调用该方法前，必须先对数组按照升序排列。
> * `<T> List<T> asList(T... a)`：数组转 `List`，返回类型是 `Arrays` 的内部类，它没有实现修改方法，如果尝试修改返回的 `List` 会抛出异常。
> * 各种 `toString(Type[] a)`：返回包含指定数组所有元素的字符串。
> * 各种 `static <T> T[] copyOf(T[] original, int length)`：返回数组指定范围的拷贝，如果 `length` 比源数组长，额外的元素会被赋默认值。所以，这个方法既能用于复制，也能用于扩容。
> * `<T> T[] copyOfRange(T[] original, int from, int to)`：也用于复制数组，左闭右开。

> **java.lang.System**
>
> * `void arraycopy(Object src,  int  srcPos, Object dest, int destPos, int length)`：将源数组指定范围的数据拷贝到目标数组。

# 大数

如果基本的整数和浮点数类型不能满足需求，比如精度不足，数值过大。可以使用 `java.math` 包中的两个类：`BigInteger` 和 `BigDecimal`。这两个类可以处理包含任意长度数字序列的数值，前者实现任意精度的整数运算，后者实现任意精度的浮点数运算。

> **java.math.BigInteger**
>
> * `BigInteger(int[] val)`
> * `BigInteger(long val)`
>
> * `igInteger(String val)`
>
>   构造器。
>
> * `static BigInteger valueOf(long val)`：使用 `long` 值构造大数的工厂方法。
>
> * `BigInteger add(BigInteger val)`
>
> * `BigInteger subtract(BigInteger val)`
>
> * `BigInteger multiply(BigInteger val)`
>
> * `BigInteger divide(BigInteger val)`
>
> * `BigInteger mod(BigInteger m)`
>
>   返回这个大数与另一个大数的和，差，积，商，求余。
>
> * `BigInteger sqrt()`：返回这个大数的平方根。
>
> * `int compareTo(BigInteger val)`：返回这个大数与另一个大数的比较值。

> **java.math.BigDecimal**
>
> * `BigDecimal(int val)`
>
> * `BigDecimal(long val)`
>
> * `BigDecimal(String val)`
>
>   构造器。
>
> * `static BigDecimal valueOf(long val)`
>
> * `static BigDecimal valueOf(long unscaledVal, int scale)`
>
>   使用 `long` 值构造大数的工厂方法，`scale` 指定精度。
>
> * `BigDecimal add(BigDecimal augend)`
>
> * `BigDecimal subtract(BigDecimal subtrahend)`
>
> * `BigDecimal multiply(BigDecimal multiplicand)`
>
> * `BigDecimal divide(BigDecimal divisor)`
>
> * `BigDecimal divide(BigDecimal divisor, int scale, RoundingMode roundingMode)`
>
>   返回这个大数与另一个大数的和，差，积，商。如果商是个无限循环小数，第一个 `divide` 会抛出异常。要得到一个舍入的结果，就需要使用第二个 `divide`。`RoundingMode.HALF_UP` 是四舍五入选项。
>
> * `int compareTo(BigDecimal val)`：返回这个大数与另一个大数的比较值。
