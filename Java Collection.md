# Java 集合框架

Java 类库提供各种常用数据结构的实现，与数组不同，这些容器只能存放引用类型元素，且大多支持动态扩容。

Java 集合有 `List`，`Queue`，`Set`，`Map` 四种基础接口，其中 `List`，`Queue`，`Set` 是 `Collection` 的子接口，而 `Map` 与 `Collection` 同级。Java 集合的接口与部分实现的关系：

```
|-- Collection：集合
	|-- List：列表
		|-- ArrayList：动态数组的索引序列
		|-- LinkedList：双向链表的有序序列
	|-- Queue：队列
		|-- Deque：双端队列
			|-- LinkedList：双向链表的双端队列
			|-- ArrayDeue：循环数组的双端队列
			|-- PriorityQueue：堆（优先队列）
	|-- Set：集
		|-- HashSet：哈希表
			|-- LinkedHashSet：元素有序的集
		|-- SortedSet：排序接口
			|-- TreeSet：有序集，使用树数据结构

|-- Map：映射
	|-- HashMap：哈希表
		|-- LinkedHashMap：元素有序的映射
	|-- SortedMap：排序接口
		|-- TreeMap：键有序的映射，使用树数据结构
```

# Collection 接口

`Collection` 是集合的基础接口，声明操作集合的通用方法。

> **java.util.Collection**
>
> * `boolean add(E e)`：添加 *e*。
> * `boolean remove(Object o)`：删除 *o*。
> * `clear()`：清空集合。
> * `int size()`：元素数量。
> * `boolean isEmpty()`：是否为空。
> * `boolean contains(Object o)`：是否包含 *o*。
> * `Object[] toArray()`：返回包含所有元素的数组。
>
> * `T[] toArray(T[] a)`：返回包含所有元素的数组，*a* 指定数组类型。
>* `Iterator iterator()`：返回遍历所有元素的迭代器。
> 
> 批量处理元素：
>
> * `boolean addAll(Collection c)`：添加 *c* 的所有元素。
>* `boolean removeAll(Collection c)`：删除 *c* 的所有元素。
> * `boolean containsAll(Collection c)`：是否包含 *c* 的所有元素。
> * `boolean retainAll(Collection c)`：保留与 *c* 的共同元素。

# Iterator 和 Iterable 

## 迭代器 Iterator

`Collection` 的 `iterator()` 方法返回一个迭代器，可用它遍历集合的所有元素。

> **java.util.Iterator**
>
> * `E next()`：返回下一个元素。
> * `void remove()`：删除上次 `next()` 的元素。
> * `boolean hasNext()`：是否还有下一个元素。
> * `forEachRemaining(Consumer action)`：使用 Lambda 处理后面所有元素。

迭代器遍历元素的顺序取决于具体实现。对于有序集合，比如 `ArrayList`，按索引顺序遍历，对于无序集合，比如 `HashSet`，看起来随机。总之，遍历顺序和元素的添加顺序并不一致。

使用迭代器期间，调用集合的方法修改元素（迭代器的 `remove()` 方法除外），迭代器在下次调用 `next()` 方法时会抛出 `ConcurrentModificationException` 异常。这里使用快速失败机制，集合一旦检测到修改，就直接抛出异常，避免发生潜在的并发问题。

## Iterable 与 for-each

`Collection` 继承 `Iterable` 接口，可用 *for-each* 结构遍历 `Collection` 的元素。因为 *for-each* 在字节码层面是使用 `Iterator` 进行实现，所以若在遍历内修改集合，会抛出异常。

# Comparable 和 Comparator

`java.lang.Comparable ` 接口被需要进行比较的类实现。

`java.util.Comparator` 接口实现为第三方的比较器。

> **java.lang.Comparable**
>
> * `int compareTo(T o)`：比较 *this* 与参数，相等返回 0，*this* 大返回正数，否则返回负数。

> **java.util.Comparator**
>
> * `int compare(T o1, T o2)`：比较 *o1* 与 *o2*，相等返回 0，*o1* 大返回正数，否则返回负数。

JDK 的许多类，比如各种包装类，都实现 `Comparable` 接口，通常是根据数字值或 Unicode 编码值进行比较。

# 线性结构 List

## List

`List` 接口表示线性数据结构，它的元素有序、可重复，允许为 `null`。

> **Java.util.List**
>
> * `add(int index, E element)`：在 *index* 位置插入 *e*。
> * `addAll(int index, Collection c)`：在 *index* 位置插入 *c* 的所有元素。
> * `remove(int index)`：删除 *index* 位置的元素。
> * `E get(int index)`：返回 *index* 位置的元素。
> * `E set(int index, E element)`：用 *e* 替换 *index* 位置的元素。
> * `int indexOf(Object o)`：返回 *o* 首次出现的位置，没有则返回 -1。
> * `int lastIndexOf(Object o)`：返回 *o* 最后出现的位置，没有则返回 -1。
> * `ListIterator listIterator()`：返回列表迭代器。
> * `ListIterator listIterator(int index)`：返回列表迭代器，并预先移到 *index* 位置。

## 列表迭代器 ListIterator

`LinkedList ` 按插入顺序把元素组织为链表，它访问任何元素，都得沿着链表挨个检索。迭代器在有序集按照添加顺序遍历元素，所以 `LinkedList ` 非常适合使用迭代器进行遍历。考虑到双向链表的特性，Java 提供一个增强的迭代器：列表迭代器 `ListIterator`，它是 `Iterator` 的子接口。

> **java.util.ListIterator**
>
> * `void add(E e)`
> * `void set(E e)`
> * `E previous()`
> * `int nextIndex()`
> * `remove()`：删除上次 `next()` 或 `previous()` 的元素。
>

## 随机访问 RandomAccess

`List` 接口定义有以索引位置来访问元素的方法，为避免对链表进行随机访问，从而影响性能，Java 1.4 引入标记接口 `RandomAccess`，它表示实现类支持使用索引位置进行随机访问。

```
if (c instanceof RandomAccess) {
    use random access algorithm
} else {
    use sequential access algorithm
}
```

## ArrayList

`ArrayList` 使用 `Object` 动态数组存储元素，所以它支持随机访问。元素从下标 0 开始按插入顺序排列，如果删除某个元素，后续元素要移位补齐，所以它非常不适合增删元素。

```
transient Object[] elementData;
```

对于 `ArrayList`，`get(int)` 的参数就是数组的下标，随机访问直接返回元素。

**构造器**

可向构造器传入一个 `int`，以指定底层数组的初始长度，若不指定，或参数为 0 值，则初始是一个空数组。

```
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);
    }
}

public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

**数组扩容**

添加元素时若容量不足，`ArrayList` 将自动扩容。基本逻辑是：每次扩容最少一半，若指定大小超过最小扩容，则扩展指定大小，然后检查是否超过最大容量，若超过，则设置为最大容量。最后使用 `Arrays` 工具类创建新数组并复制元素。

```
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1); // 扩容一半
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

**清空容器**

直接赋值底层数组 `null`。

```
public void clear() {
    modCount++;

    // clear to let GC do its work
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```

## LinkedList

`LinkedList` 把元素组织为双向链表，虽然链表插入、删除元素不需要移位后面的元素，但它每次都需要从头结点遍历到目标位置，所以它的增删操作不一定比 `ArrayList` 高效。

`LinkedList ` 实现 `Deque` 接口，这使它拥有许多额外的方法，可用作栈、队列和双端队列。

> **java.util.LinkedList**
>
> * `LinkedList([Collection c])`
>
>   构造器，初始添加 *c* 的所有元素。
>
> * `addFirst(E e)`
>
>   `addLast(E e)`
>
>   向表头/尾添加 *e*。
>
> * `removeFirst()`
>
> * `removeLast()`
>
>   删除表头/尾元素。
>
> * `E getFirst()`
>
> * `E getLast()`
>
>   返回头结点或尾结点的元素。

**结点 Node**

`LinkedList` 使用内部类 `Node` 表示结点，属性 first 和 last 是表头和表尾。

```
transient Node<E> first;

transient Node<E> last;

private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

**构造器**

```
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

**表头，表尾**

```
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}

public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

**根据索引位置访问元素**

若索引位置在前半部分，则从前往后遍历，否则从后往前。

```
public E get(int index) {
    checkElementIndex(index); // return index >= 0 && index < size;
    return node(index).item;
}

Node<E> node(int index) {
	if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

**删除元素**

删除指定元素，使用 `equals()` 方法判断是否相等，因为 `List` 能存 `null`，所以还要进行特判。

```
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

**清空容器**

遍历所有结点，为其所有属性赋 `null`，使其能被垃圾回收。

```
public void clear() {
    for (Node<E> x = first; x != null; x = next) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

# 队列 Queue，Deque

## Queue，Deque

`Queue` 继承 `Collection`，表示队列，`Deque` 是 `Queue` 的子接口，表示双端队列。

**Queue**

|          | 队满/队空抛出异常 | 返回特殊值，队满返回 `false`，队空返回 `null`。 |
| -------- | ----------------- | ----------------------------------------------- |
| 插入队尾 | `add(e)`          | `offer(e)`                                      |
| 删除队头 | `remove()`        | `poll()`                                        |
| 查看队头 | `element()`       | `peek()`                                        |

**Deque**

|          | 队满/队空抛出异常 | 返回特殊值，队满返回 `false`，队空返回 `null`。 |
| -------- | ----------------- | ----------------------------------------------- |
| 插入队头 | `addFirst(E e)`   | `offerFirst(E e)`                               |
| 插入队尾 | `addLast(E e)`    | `offerLast(E e)`                                |
| 删除队头 | `removeFirst()`   | `pollFirst()`                                   |
| 删除队尾 | `removeLast()`    | `pollLast()`                                    |
| 查看队头 | `getFirst()`      | `peekFirst()`                                   |
| 查看队尾 | `getLast()`       | `peekLast()`                                    |

除双端队列，`Deque` 还能模拟栈。

|          | 栈满/栈空抛出异常 |
| -------- | ----------------- |
| 栈顶压入 | `void push(E e)`  |
| 弹出栈顶 | `E pop()`         |
| 查看栈顶 | `E peek()`        |

## 数组队列 ArrayDeque

`ArrayDeque` 在底层使用数组存储元素，且实现为循环数组，以保证能在两端增删元素，因此数组的任何位置都可能是起点或终点。`ArrayDeque` 不允许存 `null`。

虽然 `ArrayDeque` 插入元素时可能要扩容，但普通插入操作的复杂度是 *O(1)*，而 `LinkedList` 虽然不需扩容，但每次插入都要创建新结点，申请堆空间，平均性能较慢。所以，使用队列时首选 `ArrayDeque`。

> **java.util.ArrayDeque**
>
> * `ArrayDeque([int numElements])`：构造器，*numElements* 是初始容量，默认 16，队列会拓容。
>

**构造器**

`ArrayDeque` 的初始容量默认 16，可在构造器自定义，若指定容量小于 8，则初始容量为 8，若大于等于 8，则自动向上转换为 2 的整数次幂。

```
transient Object[] elements;

private static final int MIN_INITIAL_CAPACITY = 8;

public ArrayDeque() {
    elements = new Object[16];
}

public ArrayDeque(int numElements) {
    allocateElements(numElements);
}

private void allocateElements(int numElements) {
    elements = new Object[calculateSize(numElements)];
}

private static int calculateSize(int numElements) {
	
    int initialCapacity = MIN_INITIAL_CAPACITY; // 最小容量为8
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        // 向上转换为2的整数次幂
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;

        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
    }
    return initialCapacity;
}
```

**队头队尾**

属性 head 和 tail 分别表示队头和队尾后一个空位，head 不一定为 0，也不一定小于 tail。

```
transient int head;

transient int tail;
```

**插入元素**

队头前面插入元素，`(head-1) & (elements.length-1)` 保证下标始终在 0 到 *elements.length-1* 之间。

数组长度是 2 的整数次幂，长度减 1 即在二进制低位全部取 1，和它 "与" 运算相当于取模。如果 *head-1=-1*，那就变成求相对于 `elements.length` 的补码。

```
public void addFirst(E e) {
    if (e == null)
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}
```

**数组扩容**

插入元素后，若 `head == tail`，需要扩容。扩容的逻辑是：创建一个原来两倍大的数组，然后把原数组的内容复制过去，分两次进行，第一次复制 head 右边的元素，第二次复制 head 左边的元素。

```
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0)
        throw new IllegalStateException("Sorry, deque too big");
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0;
    tail = n;
}
```

## 优先队列 PriorityQueue

`PriorityQueue` 表示堆（优先队列），它在底层使用动态数组实现，无论何时调用 `remove()` 方法，都返回当前的根元素。`PriorityQueue` 不允许存 `null`。

> 数组实现推的原理，可参考数据结构与算法笔记：堆的实现。

堆内的元素需要实现 `Comparable` 接口，或者在构造时提供 `Comparator` 比较器。默认小根堆，可通过提供自定义的比较器，改为大根堆。

堆只保证根元素最小，其它的元素在数组无序存放。

`PriorityQueue` 不支持对修改的元素重排序，若真的需要，应该把元素从容器删除，修改后再重新添加。

> **java.util.PriorityQueue**
>
> * `PriorityQueue([[int initialCapacity][, Comparator comparator]])`：构造器。
>

**添加元素**

数组末尾添加元素，然后不断向上进行 UP 调整。

```
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}
```

**删除元素**

记录根元素，然后把最后一个元素移到根进行覆盖，接着不断向下进行 Down 调整。

```
private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}
```

# 集 Set

## Set

`Set` 接口表示集的数据结构，它的元素不重复，每次添加都要检查是否已有相等元素，允许存 `null`。

## HashSet

`HashSet` 基于哈希表实现，能快速检索某个元素是否存在，因为它只遍历某个桶，而不是所有元素。

使用迭代器遍历 `HashSet`，实质是遍历桶数组，元素看起来无序。

> **java.util.HashSet**
>
> * `HashSet([[int initialCapacity][, float loadFactor]])`：构造器，*initialCapacity* 是初始容量，*loadFactor* 是负载因子。

## TreeSet

`TreeSet` 是有序集，默认按升序排列，迭代器遍历 `TreeSet` 总是有序。`TreeSet` 把元素组织为红黑树，所以它插入元素的效率很低，因为每次都要计算新元素的位置，并调整其它元素的位置，但平衡二叉树检索元素要比遍历数组或链表快的多，复杂度是 *log2(n)*。

元素需要实现 `Comparable` 接口，或者在构造时提供 `Comparator` 比较器，且比较结果应该与 `equals()` 一致。

`TreeSet` 不支持对修改的元素重排序，若真的需要，应该把元素从容器删除，修改后再重新添加。

> **java.util.TreeSet**
>
> * `TreeSet([Comparator comparator])`：构造器，可选提供比较器。

**SortedSet，NavigableSet**

`TreeSet` 实现 `NavigableSet` 接口，它是 `SortedSet` 的子接口。

> **java.util.SortedSet**
>
> * `Comparator comparator()`：返回比较器，没有则返回 `null`。
>
> * `E first()`：最小元素。
>
> * `E last()`：最大元素。

> **java.util.NavigableSet**
>
> * `E higher(E e)`：返回大于 *e* 的最小元素。
>
> * `E lower(E e)`：返回小于 *e* 的最大元素。
>
> * `E ceiling(E e)`：返回大于等于 *e* 的最小元素。
>
> * `E floor(E e)`：返回小于等于 *e* 的最大元素。
>
> * `E pollFirst()`：删除并返回最大元素。
>
> * `E pollLast()`：删除并返回最小元素。
>
> * `Iterator iterator()`：返回升序遍历的迭代器。
>
> * `Iterator descendingIterator()`：返回降序遍历的迭代器。

# 映射 Map

## Map

`Map` 接口表示映射数据结构，它是与 `Collection` 同级的顶级接口。`Map` 的元素都是 key-value 键值对，支持根据键快速查找对应的值。映射的键不能重复。

> `HashMap`、`TreeMap` 和 `HashSet`、`TreeSet` 密切相关，它们的实现原理基本相同。

> **java.util.Map**
>
> * `V get(Object key)`：返回 *key* 的值。
> * `V getOrDefault(Object key, V defaultValue)`：返回 *key* 的值，没有则返回 *defaultValue*。
> * `V put(K key, V value)`：添加元素，覆盖并返回旧值，没有则返回 `null`。
> * `clear()`：清空容器。
> * `int size()`：元素数量。
> * `boolean isEmpty()`：是否为空。
> * `putAll(Map m)`：添加集合 *m* 的所有元素。
> * `boolean containsKey(Object key)`：是否包含指定的键。
> * `boolean containsValue(Object value)`：是否包含指定的值。
> * `forEach(BiConsumer action)`：使用 Lambda 处理所有元素。
>

**键值对 Entry**

`Map` 的内部接口 `Entry` 表示 key-value 键值对，`Map` 的实现都有使用这个接口的实现来表示元素。

> **java.util.Map.Entry**
>
> * `K getKey()`：元素的键。
> * `V getValue()`：元素的值。
> * `V setValue(V value)`：设置元素的值，并返回旧值。
>

## HashMap

`HashMap` 基于哈希表实现，不过它只对键进行哈希，`HashMap` 允许键和值是 `null`。

> **java.util.HashMap**
>
> * `HashMap([[int initialCapacity[, float loadFactor]]])`：构造器，*initialCapacity* 是初始容量，*loadFactor* 是负载因子。

获取 `HashMap` 的 `Collection` 视图。

> **java.util.HashMap**
>
> * `Set keySet()`
>
> * `Collection values()`
>
> * `Set<Map.Entry> entrySet()`

## TreeMap

与 `TreeSet` 相同，`TreeMap` 把元素组织为红黑树，它的元素有序，默认按升序排列。插入、删除、更新等操作由于需要维护搜索树，所以很慢，但查找效率很高，复杂度是 *log2(n)*。

键需要实现 `Comparable` 接口，或者在构造时提供 `Comparator` 比较器，且比较结果应该与 `equals()` 一致。

`TreeMap` 不支持对修改的键重排序，若真的需要，应该把元素从容器删除，修改后再重新添加。

> **java.util.TreeMap**
>
> * `TreeMap([Comparator comparator])`：构造器。

**SortedMap，NavigableMap**

`TreeMap` 实现 `NavigableMap` 接口，它是 `SortedMap` 的子接口。

> **java.util.SortedMap**
>
> * `Comparator comparator()`：返回比较器，没有则返回 `null`。
> * `K firstKey()`：最小元素。
> * `K lastKey()`：最大元素。

# HashMap 原理

## 哈希表的应用

`HashMap` 把元素组织为哈希表，具体是数组桶。插入元素，首先对键的 `hashCode()` 返回扰动处理，得到键的哈希值，然后用哈希值对数组长度取模，结果就是这个键应该存放的桶在数组的下标。最后访问桶，存入元素。

**数组的桶 Bucket**

所谓的桶，就是发生哈希冲突的元素的集合。在 JDK 1.7，桶组织为链表，到 JDK 1.8，桶初始为链表，当元素达到 8 个时，链表转换为红黑树。元素较多时，平衡二叉树的检索效率自然远高于链表。

数组的长度，或桶的数量，叫做 Capacity 容量。`HashMap` 的容量默认是 16，桶越多，元素越分散。桶的数量必须是 2 的整数次幂，这是为使用位运算进行取模。可在构造时设定初始容量，`HashMap` 会自动把值向上转为 2 的整数次幂。

**负载因子 LoadFactor**

`HashMap` 支持动态扩容，它有一个阈值 LoadFactor，根据 `capacity * loadFactor` 计算，capacity 是当前容量，loadFactor 就是负载因子，它的默认为 0.75。当元素数量超过阈值，`HashMap` 就调用 `resize()` 方法进行扩容。每次扩容，增加一倍容量，且所有元素都要重新分配。

**扰动函数 hash(Object)**

`HashMap` 对 key 的 `hashCode()` 返回调用 `hash(Object)` 函数，进行扰动处理，得到的才是哈希值。扰动函数的作用是减少哈希冲突，使元素在哈希表更分散，防止实现较差的 `hashCode()` 使 key 太密集。

```
// JDK 1.8
static final int hash(Object key) {
 int h;
 return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

**取模式子 `h & (length-1)`**

若 length 是 2 的整数次幂，*h & (length-1)* 等价于 *h % (length-1)*，前者的效率更高，因为数据在计算机本身保存为二进制，位运算省去十进制转换的步骤。

**`hashCode()` 和 `equals()`**

插入元素，`HashMap` 需检查是否已有 key 相等的元素，判断方式是 *equals() || ==*。所以，对于要作为 key 的类型，需重写其 `hashCode()` 和 `equals()` 方法，使它们保持一致。

也就是说，`equals()` 返回 `true` 的两个 key，它们的 `hashCode()` 结果应该相等，否则，可能发生 key 相等的两个元素分别存于不同的桶。

返回来则不强求，`hashCode()` 相等的两个 key 的值可以不相等，但如果反过来也能做到，那么每个桶最多只有一个元素，哈希表的检索效率将大大提高，不过大小也会急剧增加。

## HashMap 1.8

到 JDK 1.8，`HashMap` 使用数组+链表+红黑树的结构存储元素，桶初始为链表，直到桶的元素达到 8 个，才把链表转换为红黑树，从而把桶的查找复杂度从 *O(n)* 降为 *O(logn)*。

**为什么桶不直接使用红黑树？**

链表的查找复杂度是 *O(n)*，若元素较少，遍历桶的所有元素也很快。

使用红黑树并不是没有代价，`TreeNode` 要比 `Node` 占用更多的空间，它的维护也很复杂。

**元素的实现 `Node`、`TreeNode`**

在 JDK 1.7，`HashMap` 的内部类 `Entry` 实现 `Map.Entry`，表示元素。

在 JDK 1.8，`HashMap` 的内部类 `Node` 实现 `Map.Entry`，`TreeNode` 是 `Node` 的拓展。当桶组织为链表，使用前者表示结点，若桶转换为红黑树，则使用后者表示结点。所以，可根据桶的第一个结点的类型，判断其组织为链表或红黑树。

**添加元素 `putVal()`**

* 首先，检查哈希表是否为空，如果为空调用 `resize()` 进行初始化。
* 计算下标，如果桶为空，直接插入新结点。
* 如果头结点是 `TreeNode` 类型，调用红黑树的插值方法。
* 遍历链表，检查是否 key 相等，相等则更新值并返回旧值。
* 到达链表尾部，插入新结点。如果链表长度达到 8，扩容或转换为红黑树。
* 最后，检查结点数是否超过阈值，尝试扩容。

```
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) // 首次Put时初始化哈希表
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) // 计算桶下标，若桶为空，直接添加结点
        tab[i] = newNode(hash, key, value, null);
    else { // 桶不为空
        Node<K,V> e; K k;
        // 检查头结点与新结点是否key相等
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) // 桶是红黑树，调用对应的插值方法
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else { // 桶是链表，遍历结点
            for (int binCount = 0; ; ++binCount) { // binCount记录遍历第几个结点
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null); // 尾插法，创建新结点
                    // 当结点数达到8个，尝试转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash); // 容量小于64扩容，否则转换红黑树
                    break;
                }
                if (e.hash == hash && // 检查当前结点与新结点是否key相等
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // 存在key相等的旧结点，更新值并返回旧值
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e); // 空方法，LinkedHashMap用它更新连接关系
            return oldValue;
        }
    }
    ++modCount;
    // 如果哈希表的结点数超过阈值，则进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

**根据键查询值 `getNode()`**

```
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

**扩容 `resize()`**

* 首先，计算新表的容量和阈值。
* 创建新表，若旧表为空，则这里初始化哈希表，直接返回。
* 遍历旧表
  * 如果桶只有一个结点，直接复制到新表。
  * 如果是红黑树，调用专门的方法。
  * 如果是链表，把它拆分到新表的两个桶，并保持原来的排序。

```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

static final float DEFAULT_LOAD_FACTOR = 0.75f;

final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    
    // 计算容量
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) { // 若容量已达到最大，则调整阈值直接返回
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && // 容量、阈值为原来的两倍
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // 对应HashMap(int initialCapacity)第一次Put元素
        newCap = oldThr;
    else { // 对应HashMap()第一次Put元素
        newCap = DEFAULT_INITIAL_CAPACITY; // 16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 0.75
    }
    
    // 计算阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }  
    threshold = newThr; // set threshold
    
    // 开始扩容
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) { // 旧表不为空，重新分配元素
        for (int j = 0; j < oldCap; ++j) { // 遍历旧表
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null) // 桶只有一个结点，直接复制到新表
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode) // 调用树结点的方法
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // 处理链表，把元素分别复制到新表的两个桶
                    ...
                }
            }
        }
    }
    return newTab;
}
```

## HashMap 1.7

在 JDK 1.7，`HashMap` 使用数组+链表的结构存储元素，每次查找元素都要遍历桶的链表，时间复杂度是 *O(n)*。当链表的结点较多时，这会影响查询效率。并且，它还使用头插法处理新结点，这种方式在并发环境扩容时很容易造成循环链表。

`HashMap` 在设计之初就没有考虑并发环境，尽管使用尾插法能避免链表循环，但在并发环境它仍存在许多其它的问题，比如数据丢失。所以，在并发环境，应使用专门的容器，比如 `ConcurrentHashMap `。

## HashSet 依赖 HashMap

`HashSet` 在内部封装 `HashMap`，它把元素存为 `HashMap` 的 key，以此实现唯一性。

```
private transient HashMap<E,Object> map;

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

# 链接 HashMap、HashSet

## LinkedHashMap

`LinkedHashMap` 是 `HashMap` 的拓展，它的原理和 `HashMap` 基本相同，但会为每个结点增加两个属性，用于记录前后结点，从而在哈希表上又维护了一个双向链表。属性 head、tail 表示链表的头结点和尾结点。

默认按插入顺序维护链表，但也能在构造时设置按访问顺序维护。所谓访问顺序，每当一个元素被访问，就把它放到链表尾部，这使得头结点永远是最近最少访问（LRU）的那个元素。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/java-06b178b9.png" style="zoom:50%;" />

> **java.util.LinkedHashMap**
>
> * `LinkedHashMap([int initialCapacity[, float loadFactor[, boolean accessOrder]]])`
>
>   构造器，*initialCapacity* 初始容量，*loadFactor* 负载因子，*accessOrder* 设置按访问顺序维护为例，默认插入顺序。

## LinkedHashSet

同样，`LinkedHashSet` 就是按序维护一个双向链表的 `HashSet` 而已。

> **java.util.LinkedHashSet**
>
> * `LinkedHashSet([int initialCapacity[, float loadFactor[, boolean accessOrder]]])`
>
>   构造器。
>

# 工具类 Collections

> **java.util.Collections**
>
> `List` 相关
>
> * `copy(List dest, List src)`：把 *src* 元素复制到 *dest* 内。
> * `fill(List list, T obj)`：向 *list* 填充 *obj* 对象。
> * `sort(List list)`：排序，元素需要实现 ` Comparable` 接口。
> * `sort(List list, Comparator c)`：使用比较器排序。
> * `int binarySearch(List list, T key)`：二分查找与 *key* 相等的元素，需要先按升序排序。
> * `int binarySearch(List list, T key, Comparator c)`：比较器二分查找，需要先按升序排序。
> * `shuffle(List list)`：随机排序列表。
>
> 其它
>
> * `T max(Collection coll[, Comparator comp])`
>
> * `T min(Collection coll[, Comparator comp])`
>
>   返回集合的最大值、最小值，元素需要实现 `Comparable` 接口，或提供比较器。
>
> * `Set singleton(T o)`
>
> * `List singletonList(T o)`
>
> * `Map singletonMap(K key, V value)`
>
>   创建不可变的单例集合。
>
> * `Collection synchronizedCollection(Collection c)`
>
> * `Set synchronizedSet(Set s)`
>
> * `List synchronizedList(List list)`
>
> * `Map synchronizedMap(Map m)`
>
>   创建集合的同步视图，其实就是使用 `synchronized` 修饰所有方法。
>
> * `Collection unmodifiableCollection(Collection c)`
>
> * `Set unmodifiableSet(Set s)`
>
> * `List unmodifiableList(List list)`
>
> * `Map unmodifiableMap(Map m)`
>
>   创建集合的不可变视图，调用修改方法会抛出 `UnsupportedOperationException` 异常。

# 遗留集合

## HashTable

`HashTable` 与 `HashMap` 的功能相同，接口也相似，但是 `HashTable` 的方法都用 `synchronized` 修饰。

## 属性映射 Propertys

`Propertys` 是特殊的映射，它有 3 个特性：

* 键和值都是字符串类型。
* 可以轻松地保存到文件，或从文件加载。
* 可在二级表为属性设置默认值。

> **java.util.Properties**
>
> * `Properties([Properties defaults])`：构造器，可选以 *defaults* 作为默认值。
>
> * `synchronized Object setProperty(String key, String value)`：设置属性，返回旧值。
>
> * `String getProperty(String key[, String defaultValue])`：返回键的值，可选指定默认值。
>
> * `synchronized void load(InputStream inStream)`
>
> * `synchronized void load(Reader reader)`
>
>   从输入流加载属性映射。
>
> * `void store(OutputStream out, String comments)`
>
> * `void store(Writer writer, String comments)`
>
>   把集合内容写到输出流，*comments* 是文件的第一行注释。

# JUC 集合

## 阻塞队列

`BlockingQueue` 表示阻塞队列，它是 `Queue` 的子接口。`BlockingDeque` 继承 `BlockingQueue`，表示双端阻塞队列。什么是阻塞队列？插入元素而队列已满，或取出元素而队列为空，线程会被阻塞，直到条件允许继续执行。

某些情况，可用阻塞队列代替同步锁来解决线程安全问题。

**BlockingQueue**

|              | 阻塞   | 限时阻塞                 |
| ------------ | ------ | ------------------------ |
| 队尾添加元素 | `put`  | `offer(E,long,TimeUnit)` |
| 删除队头元素 | `take` | `poll(long, TimeUnit)`   |

**BlockingDeque**

|          | 阻塞            | 限时阻塞                                       |
| -------- | --------------- | ---------------------------------------------- |
| 插入队头 | `putFirst(E e)` | `offerFirst(E e, long timeout, TimeUnit unit)` |
| 插入队尾 | `putLast(E e)`  | `offerLast(E e, long timeout, TimeUnit unit)`  |
| 取出队头 | `takeFirst()`   | `pollFirst(long timeout, TimeUnit unit)`       |
| 取出队尾 | `takeLast()`    | `pollLast(long timeout, TimeUnit unit)`        |

**BlockingQueue 实现**

* `LinkedBlockingQueue`：链表，FIFO，队长无限。
* `ArrayBlockingQueue`：数组，FIFO，队长可选，默认无限，可选公平策略以优先响应等待较久的线程。
* `PriorityBlockingQueue`：元素需实现 `Comparable`，或提供比较器，比较结果大的元素优先被取出。
* `DelayQueue`：元素需实现 `Delayed`，只有期满的元素才能被取出。

## ConcurrentHashMap

`ConcurrentHashMap` 是线程安全的 `HashMap` 拓展，专为并发环境设计。

至于 `HashTable`，由于同步锁 `synchronized` 的特性，任何时刻至多有一个线程能访问它，并发性能太低。

**JDK 1.7 分段**

在 JDK 1.7，`ConcurrentHashMap` 使用分段锁机制保证线程安全和并发性能。所谓分段 `Segment`，其实是一个继承 `ReentrantLock` 的 `HashMap`，`ConcurrentHashMap` 把哈希表拆分成若干个分段，不同分段可被不同的线程同时安全访问。所以，分段越多，并发性能越高。

**JDK 1.8 桶锁**

在 JDK 1.8，`ConcurrentHashMap` 舍弃分段，把同步的粒度下沉到桶，使用 `synchronized` 和 CAS 实现并发控制，每个桶锁定的对象是它的头结点。

与 JDK 1.7 相比，并发性能得到极大提升，且由于 JDK 6 对 `synchronized` 的优化，以及使用 CAS 操作，同步效率也更高。
