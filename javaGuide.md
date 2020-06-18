## 一、java基础

Java 本身是一种面向对象的语言，最显著的特性有两个方面，一是所谓的“书写一次，到处运行”（Write once, run anywhere），能够非常容易地获得跨平台能力；另外就是垃圾收集（GC, Garbage Collection），Java 通过垃圾收集器（Garbage Collector）回收分配内存，大部分情况下，程序员不需要自己操心内存的分配和回收。

我们日常会接触到 JRE（Java Runtime Environment）或者 JDK（Java Development Kit）。 JRE，也就是 Java 运行环境，包含了 JVM 和 Java 类库，以及一些模块等。而 JDK 可以看作是 JRE 的一个超集，提供了更多工具，比如编译器、各种诊断工具等。

对于“Java 是解释执行”这句话，这个说法不太准确。我们开发的 Java 的源代码，首先通过 Javac 编译成为字节码（bytecode），然后，在运行时，通过 Java 虚拟机（JVM）内嵌的解释器将字节码转换成为最终的机器码。但是常见的 JVM，比如我们大多数情况使用的 Oracle JDK 提供的 Hotspot JVM，都提供了 JIT（Just-In-Time）编译器，也就是通常所说的动态编译器，JIT 能够在运行时将热点代码编译成机器码，这种情况下部分热点代码就属于编译执行，而不是解释执行了。

### 1.1 jvm

// TODO 


## 二、集合

Java 的集合框架，Collection 接口是所有集合的根，然后扩展开提供了三大类集合，分别是：

* List，也就是我们前面介绍最多的有序集合，它提供了方便的访问、插入、删除等操作。
* Set，Set 是不允许重复元素的，这是和 List 最明显的区别，也就是不存在两个对象 equals 返回 true。用在保证元素唯一性场合。
* Queue/Deque，则是 Java 提供的标准队列结构的实现，除了集合的基本功能，它还支持类似先入先出（FIFO， First-in-First-Out）或者后入先出（LIFO，Last-In-First-Out）等特定行为。这里不包括 BlockingQueue，因为通常是并发编程场合，所以被放置在并发包里。

广义 Java 集合框架中还包含Map，HashMap 使用评率很高。

![](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200615165653.png)

如上图所示每种集合的通用逻辑，都被抽象到相应的抽象类之中，比如 AbstractList 就集中了各种 List 操作的通用部分。这些集合不是完全孤立的，比如，LinkedList 本身，既是 List，也是 Deque 哦。

### 2.1  List

List顶层抽象接口是AbstractList，实现AbstractList接口的有 Vector、ArrayList、LinkedList 

这三者都是实现集合框架中的 List，也就是所谓的有序集合，因此具体功能也比较近似，比如都提供按照位置进行定位、添加或者删除的操作，都提供迭代器以遍历其内容等。但因为具体的设计区别，在行为、性能、线程安全等方面，表现又有很大不同。

* Vector 是 Java 早期提供的**线程安全的动态数组**，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。Vector 内部是使用对象数组来保存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据，然后插入数据。
* ArrayList 是应用更加广泛的**动态数组**实现，它本身不是线程安全的，所以性能要好很多。与 Vector 近似，ArrayList 也是可以根据需要调整容量，不过两者的调整逻辑有所区别，Vector 在扩容时会提高 1 倍，而 ArrayList 则是增加 50%。
* LinkedList 顾名思义是 Java 提供的**双向链表**，所以它不需要像上面两种那样调整容量，它也不是线程安全的。

Vector 和 ArrayList 作为动态数组，其内部元素以数组形式顺序存储的，所以非常适合随机访问的场合。除了尾部插入和删除元素，往往性能会相对较差，比如我们在中间位置插入一个元素，需要移动后续所有元素。而 LinkedList 进行节点插入、删除却要高效得多，但是随机访问性能则要比动态数组慢。

所以，在应用开发中，如果事先可以估计到，应用操作是偏向于插入、删除，还是随机访问较多，就可以针对性的进行选择。这也是面试最常见的一个考察角度，给定一个场景，选择适合的数据结构。

### 2.2 Set

Set顶层抽象接口是AbstractSet，实现AbstractSet接口的有，TreeSet，HashSet，LinkedHashSet。Set 是不允许重复元素的，这是和 List 最明显的区别，也就是不存在两个对象 equals 返回 true。我们在日常开发中有很多需要保证元素唯一性的场合。TreeSet 代码里实际默认是利用 TreeMap 实现的，Java 类库创建了一个 Dummy 对象“PRESENT”作为 value，然后所有插入的元素其实是以键的形式放入了 TreeMap 里面；同理，HashSet 其实也是以 HashMap 为基础实现的，原来他们只是 Map 类的马甲！

* TreeSet 支持自然顺序访问，但是添加、删除、包含等操作要相对低效（log(n) 时间）。
* HashSet 则是利用哈希算法，理想情况下，如果哈希散列正常，可以提供常数时间的添加、删除、包含等操作，但是它不保证有序。
* LinkedHashSet，内部构建了一个记录插入顺序的双向链表，因此提供了按照插入顺序遍历的能力，与此同时，也保证了常数时间的添加、删除、包含等操作，这些操作性能略低于 HashSet，因为需要维护链表的开销。

在遍历元素时，HashSet 性能受自身容量影响，所以初始化时，除非有必要，不然不要将其背后的 HashMap 容量设置过大。而对于 LinkedHashSet，由于其内部链表提供的方便，遍历性能只和元素多少有关系。

### 2.3 Queue/Deque
Queue/Deque，则是 Java 提供的标准队列结构的实现，除了集合的基本功能，它还支持类似先入先出（FIFO， First-in-First-Out）或者后入先出（LIFO，Last-In-First-Out）等特定行为。这里不包括 BlockingQueue，因为通常是并发编程场合，所以被放置在并发包里。

### 2.4 Map
Map 是广义 Java 集合框架中的另外一部分，Map顶层抽象接口为AbstractMap，实现AbstractMap接口的有，EnumMap、HashMap、TreeMap。Map是以**键值对**的形式存储和操作数据的容器类型。Hashtable例外，它继承了Dictionary类。

![](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200616105731.png)

* Hashtable 是早期 Java 类库提供的一个哈希表实现，本身是同步的，不支持 null 键和值，由于同步导致的性能开销，所以已经很少被推荐使用。作为类似 Vector、Stack 的早期集合相关类型，它是扩展了 Dictionary 类的，类结构上与 HashMap 之类明显不同。
* HashMap 是应用更加广泛的哈希表实现，行为上大致上与 HashTable 一致，主要区别在于 HashMap 不是同步的，支持 null 键和值等。通常情况下，HashMap 进行 put 或者 get 操作，可以达到常数时间的性能，所以**它是绝大部分利用键值对存取场景的首选**，比如，实现一个用户 ID 和用户信息对应的运行时存储结构。
* TreeMap 则是基于红黑树的一种提供顺序访问的 Map，和 HashMap 不同，它的 get、put、remove 之类操作都是 O（log(n)）的时间复杂度，具体顺序可以由指定的 Comparator 来决定，或者根据键的自然顺序来判断。

#### 2.4.1 HashMap
HashMap 的性能表现非常依赖于哈希码的有效性，hashCode 和 equals 有以下基本约定：
* equals 相等，hashCode 一定要相等。
* 重写了 hashCode 也要重写 equals。
* hashCode 需要保持一致性，状态改变返回的哈希值仍然要一致。
* equals 的对称、反射、传递等特性。

HashMap采用的链表法的方式，链表是单向链表。其他解决hash冲突的方法还有开放地址法，当冲突发生时，使用某种探查技术在散列表中形成一个探查(测)序列。沿此序列逐个单元地查找，直到找到给定的地址。按照形成探查序列的方法不同，可将开放定址法区分为线性探查法、二次探查法、双重散列法等。开放定址法公式为：`H i ( key ) = ( H ( key )+ d i ) mod m ( i = 1,2,…… ， k ( k ≤ m – 1)) `根据di的变化不通可以分为下面三种散列方式：
1. d i ＝ 1 ， 2 ， 3 ， …… 线性探测再散列；
2. d i ＝ 1^2 ，－ 1^2 ， 2^2 ，－ 2^2 ， k^2， -k^2…… 二次探测再散列；
3. d i ＝ 伪随机序列 伪随机再散列； 

通过源码我们来一起看看 HashMap 内部的结构，它可以看作是数组（Node<K,V>[] table）和链表结合组成的复合结构，
``` java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {

	static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
	static final int MAXIMUM_CAPACITY = 1 << 30;  // HashMap最大长度，二进制下1向左移动30位，也就是2的30次方
	static final float DEFAULT_LOAD_FACTOR = 0.75f;  // 默认负载因子
	static final int TREEIFY_THRESHOLD = 8;  // 链表树化默认阈值
	static final int UNTREEIFY_THRESHOLD = 6;  // 树还原为连表默认阈值
	
    static class Node<K,V> implements Map.Entry<K,V> {
            final int hash;
            final K key;
            V value;
            Node<K,V> next;

            Node(int hash, K key, V value, Node<K,V> next) {
                this.hash = hash;
                this.key = key;
                this.value = value;
                this.next = next;
            }
    }
    /* ---------------- Fields -------------- */
    transient Node<K,V>[] table;
    int threshold; // 所能容纳的key-value对极限 
    final float loadFactor;  // 负载因子
    /* ---------------- Cloning and serialization -------------- */
    private void writeObject(java.io.ObjectOutputStream s)
		// less ...
    }
    private void readObject(java.io.ObjectInputStream s)
        // less ...
    }
}
```
数组被分为一个个桶（bucket），通过哈希值决定了键值对在这个数组的寻址；哈希值相同的键值对，则以链表形式存储，最后放入的对象会被放置在连表头节点，是因为HashMap的发明者认为，**后插入的Entry被查找的可能性更大**，你可以参考上面的HashMap源码。这里需要注意的是，如果链表大小超过阈值（TREEIFY_THRESHOLD, 8），链表就会被改造为树形结构，当连表长度小于或者等于阈值（UNTREEIFY_THRESHOLD，6），树形结构就会变成连表结构。那么，为什么 HashMap 要树化呢？本质上这是个安全问题。因为在元素放置过程中，如果一个对象哈希冲突，都被放置到同一个桶里，则会形成一个链表，我们知道链表查询是线性的，会严重影响存取的性能。

![](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200616151815.png)

观察HashMap 的源码我们发现存对象的数组`Node<K,V>[] table;`被关键字`transient`修饰，所以当HashMap对象被序列化的时候table字段是没有被序列化的。着有点不符合逻辑，真正存放了数据的数组没有呗序列化，其实在序列化和反序列化的时候，java虚拟机会利用反射的方式调用HashMap中的私有方法`writeObject()`和`readObject()`，来读取和写入到`Node<K,V>[] table;`这样只序列化实际存储元素的数组。引用这种方式序列化是由于不同的虚拟机对于相同 hashCode 产生的 Code 值可能是不一样的，所以反序列化的话的过程也需要在当时的环境中重新构造出完整的对象。

HashTable是使用了 lazy-load（懒加载）的方式实现，它的初始化和扩容都是在调用`put()`方法时完成，我们看下源码：
``` java
/* ---------------- Constructor -------------- */
public HashMap(int initialCapacity, float loadFactor){  
    // less ... 
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

/* ---------------- Method -------------- */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbent,
               boolean evit) {
    Node<K,V>[] tab; Node<K,V> p; int , i;
    if ((tab = table) == null || (n = tab.length) = 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == ull)
        tab[i] = newNode(hash, key, value, nll);
    else {
        // less ...
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for first 
           treeifyBin(tab, hash);
        // less ... 
     }
}
```
从`putVal()`方法最初的几行，我们就可以发现几个有意思的地方：
* 如果表格是 null，resize 方法会负责初始化它，这从 tab = resize() 可以看出。
* resize 方法兼顾两个职责，创建初始存储表格，或者在容量不满足需求的时候，进行扩容（resize）。
* 放置新的键值对的过程中，如果长度超过threshold （`threshold = length * Load factor`），就会发生扩容，新的容量是原有容量的2倍，默认的负载因子0.75是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子Load factor的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子loadFactor的值，这个值可以大于1。

仔细观察哈希值的源头，我们会发现，它并不是 key 本身的 hashCode，而是来自于 HashMap 内部的另外一个 hash 方法。注意，为什么这里需要将高位数据移位到低位进行异或运算呢？**这是因为有些数据计算出的哈希值差异主要在高位，而 HashMap 里的哈希寻址是忽略容量以上的高位的，那么这种处理就可以有效避免类似情况下的哈希碰撞。**
``` java
/* ---------------- Method 1 -------------- */
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 高位参与运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
/* ---------------- Method 2 -------------- */
static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但是实现原理一样的
     return h & (length-1);  //第三步 取模运算
}
```
这里的Hash算法本质上就是三步：取key的hashCode值、高位参与运算、取模运算。最终得到新放入元素的下标。

在HashMap中，哈希桶数组table的长度length大小必须为2的n次方(一定是合数)，这是一种非常规的设计，主要是为了在取模和扩容时做优化，同时为了减少冲突，HashMap定位哈希桶索引位置时，也加入了高位参与运算的过程。

HashMap的定位做的非常巧妙，它通过h` & (table.length -1)`来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当length总是2的n次方时，`h & (length-1)`运算等价于对length取模，也就是`h % length`，但是&比%具有更高的效率。在JDK1.8的实现中，优化了高位运算的算法，通过`hashCode()`的高16位异或低16位实现的：`(h = k.hashCode()) ^ (h >>> 16)`，主要是从速度、功效、质量来考虑的，这么做可以在当数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

![hash](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200617143427.png)

而且当HashMap扩容的时候，我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，左侧表示扩容前的key1和key2两种key确定索引位置的示例，右侧表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。

![](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200617145017.png)

因此元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200617145619.png)

因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2016/3cc9813a.png)

#### 2.4.2 LinkedHashMap
LinkedHashMap 和 TreeMap 都可以保证某种顺序，但二者还是非常不同的，LinkedHashMap 通常提供的是遍历顺序符合插入顺序，它的实现是通过为条目（键值对）维护一个双向链表。
``` java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>{

	static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
 }
```
通过源代码我们可以看到LinkedHashMap.Entry是继承了HashMap.Node，增加了 before, after，组成双向连表。LinkedHashMap的有序可以设置成插入顺序和访问顺序，默认是按照插入顺序组成双向连表。通过特定构造函数，将accessOrder设置成true，可以创建反映访问顺序的实例，所谓的 put、get、compute 等，都算作“访问”。

#### 2.4.3 treeMap
LinkedHashMap保证数据可以保持插入顺序，或者访问顺序，而如果我们希望Map可以保持key的大小顺序的时候，我们就需要利用TreeMap了。TreeMap的实现是的红黑树，使用红黑树的好处是能够使得树具有不错的平衡性，这样操作的速度就可以达到log(n)的水平。
红黑树是一种自平衡二叉查找树。它的统计性能要好于平衡二叉树

### 2.5 线程安全的集合
Vector、Hashtable 、等是线程安全的，除此之外我们完全可以利用类似方法来实现基本的线程安全集合：
``` java
static <T> List<T> synchronizedList(List<T> list)
List list = Collections.synchronizedList(new ArrayList());
```
它的实现，基本就是将每个基本方法，比如 get、set、add 之类，都通过 synchronizd 添加基本的同步支持，非常简单粗暴，但也非常实用。注意这些方法创建的线程安全集合，都符合迭代时 fail-fast[1]行为，当发生意外的并发修改时，尽早抛出 `ConcurrentModificationException` 异常，以避免不可预计的行为。

或者我在`java.util.concurrent`里面的线程安全容器，TODO

### 2.6 集合排序
* 对于原始数据类型，目前使用的是所谓双轴快速排序（Dual-Pivot QuickSort），是一种改进的快速排序算法，早期版本是相对传统的快速排序，该算法是不稳定的。
* 而对于对象数据类型，目前则是使用TimSort，思想上也是一种归并和二分插入排序（binarySort）结合的优化排序算法。TimSort 并不是 Java 的独创，简单说它的思路是查找数据集中已经排好序的分区（这里叫 run），然后合并这些分区来达到排序的目的。

Java 8 引入了并行排序算法parallelSort，这是为了充分利用现代多核处理器的计算能力，底层实现基于 fork-join 框架，处理的数据集比较小的时候，差距不明显，甚至还表现差一点；但是，当数据集增长到数万或百万以上时，提高就非常大了，具体还是取决于处理器和系统环境。只有当数组长度大于 `MIN_ARRAY_SORT_GRAN` 时才进行多线程并行排序 (源码中 MIN_ARRAY_SORT_GRAN 为 1<<13，即 8192)，简单的说实现方法就是将数组拆分成多个子数组，多线程进行排序，然后归并

### 2.7 集合新特性

#### 2.7.1 Lambda 和 Stream
在 Java 8 之中，Java 平台支持了 Lambda 和 Stream（在程序界称之为流），相应的 Java 集合框架也进行了大范围的增强，以支持类似为集合创建相应 stream或者 parallelStream 的方法实现，我们可以非常方便的实现函数式代码。
阅读 Java 源代码，你会发现，这些 API 的设计和实现比较独特，它们并不是实现在抽象类里面，而是以默认方法的形式实现在 Collection 这样的接口里！（方法被default关键字修饰）这是 Java 8 在语言层面的新特性，允许接口实现默认方法，理论上来说，我们原来实现在类似 Collections 这种工具类中的方法，大多可以转换到相应的接口上。

流在处理数据进行一些迭代操作的时候确认很方便，但是在执行一些耗时或是占用资源很高的任务时候，串行化的流无法带来速度/性能上的提升，并不能满足我们的需要，通常我们会使用多线程来并行或是分片分解执行任务，而在Stream中也提供了这样的并行方法，那就是使用parallelStream()方法或者是使用stream().parallel()来转化为并行流，而并行流是使用了Fork/Join 框架


#### 2.7.2 List.of()、Set.of()
在 Java 9 中，Java 标准类库提供了一系列的静态工厂方法，比如，List.of()、Set.of()，大大简化了构建小的容器实例的代码量。根据业界实践经验，我们发现相当一部分集合实例都是容量非常有限的，而且在生命周期中并不会进行修改。利用新的容器静态工厂方法，一句代码就够了，并且保证了不可变性。

``` java
List<String> simpleList = List.of("Hello","world");
```
更进一步，通过各种 of 静态工厂方法创建的实例，还应用了一些我们所谓的最佳实践，比如，它是不可变的，符合我们对线程安全的需求；它因为不需要考虑扩容，所以空间上更加紧凑等。


## 三、 多线程

### 3.1 多线程演进
* 1.0-1.4 中的 java.lang.Thread
* 5.0 中的 java.util.concurrent
* 6.0 中的 Phasers 等
* 7.0 中的 Fork/Join 框架
* 8.0 中的 Lambda

## 四、算法

### 4.1 数据结构
#### 4.1.1 数组
#### 4.1.2 连表
#### 4.1.3 树
树是一种非线性的数据结构，相对于线性的数据结构(链表、数组)而言，树的平均运行时间更短(往往与树相关的排序时间复杂度都不会高)，但是在编程的世界中，我们一般把树**“倒”**过来看，这样容易我们分析。一般的树是有很多很多个分支的，分支下又有很多很多个分支，如果在程序中研究这个会非常麻烦。因为本来树就是非线性的，而我们计算机的内存是线性存储的，太过复杂的话我们无法设计出来。因此，我们一般研究常用的二叉树，也就是每个节点有两个分支的树结构。

树结构包含二叉树、二叉查找树、平衡二叉树、平衡查找树之AVL树、平衡二叉树之红黑树、B树、B+树、B*树、Trie树、等。
##### 4.1.3.1 二叉树
* 二叉树：二叉树的每个结点至多只有二棵子树(不存在度大于2的结点)，二叉树的子树有左右之分，次序不能颠倒。二叉树的第i层至多有2的i-1次方个结点；深度为k的二叉树至多有2的k-1次方个结点。
* 满二叉树：除最后一层无任何子节点外，每一层上的所有结点都有两个子结点。也可以这样理解，除叶子结点外的所有结点均有两个子结点。节点数达到最大值，所有叶子结点必须在同一层上。
* 完全二叉树：若设二叉树的深度为h，除第 h 层外，其它各层 (1～(h-1)层) 的结点数都达到最大个数，第h层所有的结点都连续集中在最左边，这就是完全二叉树。

##### 4.1.3.2 二叉查找树
二叉查找树，又称为是二叉排序树（Binary Sort Tree）或二叉搜索树。二叉查找树具有以下性质：
1. 若左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2. 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值；
3. 左、右子树也分别为二叉排序树；
4. 没有值相等的节点。
5. 二叉查找树的性质：对二叉查找树进行中序遍历，即可得到有序的数列。
##### 4.1.3.2 平衡二叉树
二叉查找树的时间复杂度：它和二分查找一样，插入和查找的时间复杂度均为O(logn)，在某些极端的情况下（如在插入的序列是有序的时），二叉搜索树将退化成近似链或链，此时，其操作的时间复杂度将退化成线性的，时间复杂度将退化为O(n)。原因在于插入和删除元素的时候，树没有保持平衡。

平衡二叉树定义：平衡二叉树（Balanced Binary Tree）又被称为AVL树（有别于AVL算法），且具有以下性质：它是一 棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。平衡二叉树的常用算法有红黑树、AVL树等。在平衡二叉搜索树中，我们可以看到，其高度一般都良好地维持在O(log2n)，大大降低了操作的时间复杂度。
##### 4.1.3.3 平衡查找树之AVL树
AVL树定义：AVL树是最先发明的**自平衡二叉查找树**。AVL树得名于它的发明者 G.M. Adelson-Velsky 和 E.M. Landis，他们在 1962 年的论文 "An algorithm for the organization of information" 中发表了它。在AVL中任何节点的两个儿子子树的高度最大差别为1，所以它也被称为高度平衡树，n个结点的AVL树最大深度约1.44log2n。查找、插入和删除在平均和最坏情况下都是O(logn)。增加和删除可能需要通过一次或多次树旋转来重新平衡这个树。这个方案很好的解决了二叉查找树退化成链表的问题，把插入，查找，删除的时间复杂度最好情况和最坏情况都维持在O(logN)。但是频繁旋转会使插入和删除牺牲掉O(logN)左右的时间，不过相对二叉查找树来说，时间上稳定了很多。

##### 4.1.3.4 平衡二叉树之红黑树
红黑树的定义：红黑树是一种自平衡二叉查找树，是在计算机科学中用到的一种数据结构，典型的用途是实现关联数组。它是在1972年由鲁道夫·贝尔发明的，称之为"对称二叉B树"，它现代的名字是在 Leo J. Guibas 和 Robert Sedgewick 于1978年写的一篇论文中获得的。它是复杂的，但它的操作有着良好的最坏情况运行时间，并且在实践中是高效的: 它可以在O(logn)时间内做查找，插入和删除，这里的n是树中元素的数目。

红黑树的性质：红黑树是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。在二叉查找树强制的一般要求以外，对于任何有效的红黑树我们增加了如下的额外要求:
* 节点是红色或黑色。
* 根是黑色。
* 所有叶子都是黑色（叶子是NIL节点）。
* 每个红色节点必须有两个黑色的子节点。(从每个叶子到根的所有路径上不能有两个连续的红色节点。)
* 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。

![An example of a red-black tree](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200618175807.png)

##### 4.1.3.5 B树
B树的定义：B树（B-tree）是一种树状数据结构，能够用来存储排序后的数据，它也是平衡树，但是它不是二叉树。这种数据结构能够让查找数据、循序存取、插入数据及删除的动作，都在对数时间内完成。B树，概括来说是一个一般化的二叉查找树，可以拥有多于2个子节点。与自平衡二叉查找树不同，B-树为系统最优化大块数据的读和写操作。B-tree算法减少定位记录时所经历的中间过程，从而加快存取速度。这种数据结构常被应用在数据库和文件系统的实作上。B树作为一种多路搜索树，它有以下性质：

* 定义任意非叶子结点最多只有M个儿子；且M>2；

* 根结点的儿子数为[2, M]；

* 除根结点以外的非叶子结点的儿子数为[M/2, M]；

* 每个结点存放至少M/2-1（取上整）和至多M-1个关键字；（至少2个关键字）

* 叶子结点的关键字个数=指向儿子的指针个数-1；

* 叶子结点的关键字：K[1], K[2], …, K[M-1]；且K[i] < K[i+1]；

* 叶子结点的指针：P[1], P[2], …, P[M]；其中P[1]指向关键字小于K[1]的子树，P[M]指向关键字大于K[M-1]的子树，其它P[i]指向关键字属于(K[i-1], K[i])的子树；

* 有叶子结点位于同一层；

  ![img](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200618182052.gif)


在B树中查找给定关键字的方法是，首先把根结点取来，在根结点所包含的关键字K1,…,Kn查找给定的关键字（可用顺序查找或二分查找法），若找到等于给定值的关键字，则查找成功；否则，一定可以确定要查找的关键字在Ki与Ki+1之间，Pi为指向子树根节点的指针，此时取指针Pi所指的结点继续查找，直至找到，或指针Pi为空时查找失败。
##### 4.1.3.6 B+树
B+树是B树的变体，也是一种多路搜索树， 其定义基本与B-树相同，除了以下特性：
* 非叶子结点的子树指针与关键字个数相同；
* 非叶子结点的子树指针P[i]，指向关键字值属于[K[i], K[i+1])的子树（B-树是开区间）；
* 为所有叶子结点增加一个链指针；
* 所有关键字都在叶子结点出现；

B+树的性质：
* 所有关键字都出现在叶子结点的链表中（稠密索引），且链表中的关键字恰好是有序的；
* 不可能在非叶子结点命中；
* 非叶子结点相当于是叶子结点的索引（稀疏索引），叶子结点相当于是存储（关键字）数据的数据层；
* 更适合文件索引系统。

![img](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200618183522.gif)

##### 4.1.3.6 B*树
B*树是B+树的变体，在B+树的非根和非叶子结点再增加指向兄弟的指针，将结点的最低利用率从1/2提高到2/3。B*树定义了非叶子结点关键字个数至少为(2/3)*M，即块的最低使用率为2/3（代替B+树的1/2）；
B+树的分裂：当一个结点满时，分配一个新的结点，并将原结点中1/2的数据复制到新结点，最后在父结点中增加新结点的指针；B+树的分裂只影响原结点和父结点，而不会影响兄弟结点，所以它不需要指向兄弟的指针；
B*树的分裂：当一个结点满时，如果它的下一个兄弟结点未满，那么将一部分数据移到兄弟结点中，再在原结点插入关键字，最后修改父结点中兄弟结点的关键字（因为兄弟结点的关键字范围改变了）；如果兄弟也满了，则在原结点与兄弟结点之间增加新结点，并各复制1/3的数据到新结点，最后在父结点增加新结点的指针；
所以，B*树分配新结点的概率比B+树要低，空间使用率更高。


### 4.2 排序算法

#### 4.2.1 双轴快速排序（Dual-Pivot QuickSort）
DualPivotQuicksort，该排序算法是不稳定的，即：相等的两个元素在排序前后的相对位置可能会发生变化

#### 4.2.2 TimSort
思想上也是一种归并和二分插入排序（binarySort）结合的优化排序算法，简单说它的思路是查找数据集中已经排好序的分区（这里叫 run），然后合并这些分区来达到排序的目的。

#### 4.2.3 parallelSort
简单的说实现方法就是将数组拆分成多个子数组，多线程进行排序，然后归并


## 名词解释

### 1. ffail-fast
在系统设计中，快速失效系统一种可以立即报告任何可能表明故障的情况的系统。快速失效系统通常设计用于停止正常操作，而不是试图继续可能存在缺陷的过程。这种设计通常会在操作中的多个点检查系统的状态，因此可以及早检测到任何故障。快速失败模块的职责是检测错误，然后让系统的下一个最高级别处理错误。
``` java
for (String userName : userNames) {
    if (userName.equals("Hollis")) {
        userNames.remove(userName);
    }
}
```
上面代码中foreach 循环里进行元素的 remove/add 操作，运行程序会报`ConcurrentModificationException`这其实也是运用了fail-fast机制。我们剖析下源码发现在增强for循环中，集合遍历是通过iterator进行的，而在`iterator.next()`方法中我们发现下面的代码：
``` java
final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```
会在上面的代码中发现`modCount` 是ArrayList中的一个成员变量。它表示该集合实际被修改的次数，数组初始化时默认值为0。`expectedModCount` 是 ArrayList中的一个内部类，也是iterator中的成员变量，它表示这个迭代器预期该集合被修改的次数，值随着Itr被创建而初始化。那么，接着我们看下`userNames.remove(userName);`里面做了什么事情，为什么会导致`expectedModCount`和`modCount`不一样。通过翻阅代码，我们也可以发现，remove方法核心逻辑如下：
``` java
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}
```
可以看到，它只修改了modCount，并没有对expectedModCount做任何操作。

画一张图理解一下

![](https://gitee.com/suqianlei/Pic-Go-Repository/raw/master/img/20200618185354.png)

所以，在使用Java的集合类的时候，如果发生CMException，优先考虑fail-fast有关的情况，实际上这里并没有真的发生并发，只是Iterator使用了fail-fast的保护机制，只要他发现有某一次修改是未经过自己进行的，那么就会抛出异常。

### 2. java运算符
&（按位与）、&&（逻辑与）、|（按位或）、||（逻辑或）、^（异或运算符），与运算、或运算和异或运算，最终都会把运算符两端的值转换成二进制，然后再进行运算。

<<（左移运算符）、>>（右移运算符）、~（取反运算符）、>>>（无符号右移运算符），先把计算的对象转化为二进制，然后进行二进制位数的移动。
