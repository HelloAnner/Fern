
### 线程不安全的HashMap

因为多线程环境下，[1.7使用HashMap进行put操作会引起死循环，导致CPU使用率接近100%](https://blog.csdn.net/m0_46405589/article/details/109206432) ，所以在并发环境下不能使用HashMap

> 假设两个线程A、B都在进行put操作，并且hash函数计算出的插入下标是相同的，当线程A执行完第六行代码后由于时间片耗尽导致被挂起，而线程B得到时间片后在该下标处插入了元素，完成了正常的插入，然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所有此时不会再进行判断，而是直接进行插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全

### 效率低下的HashTable

HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常的低，因为当一个线程正在访问HashTable的同步方法时，这时另外一个线程也来访问HashTable的同步方法，可能会进入阻塞或轮询状态。如线程A使用put进行添加元素，线程B不但不能使用put方法添加元素，并且不能使用get方法来获取元素，所以竞争越激烈效率越低，也就是说对于HashTable而言，synchronized是针对Hash整张表的，即每次锁住整张表让线程独占。相当于所有线程进行读写的时候都去竞争一把锁，导致效率非常低下。

### ConcurrentHashMap的锁分段技术

ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的储存，然后给每一段数据配一把锁，当一个线程占用锁访问一个段数据时，其他的段数据也是可以被其他线程访问的

**ConcurrentHashMap是可以做到读取数据不加锁，并且其内部的结构可以让其在进行写操作时能够将锁的粒度保持尽量的小，不用对整个ConcurrentHashMap加锁**

### ConcurrentHashMap的内部结构

ConcurrentHashMap为了提高本身的并发能力，在内部采用了一个叫做Segment的结构，一个Segment其实就是一个类似HashTable的结构，Segment内部维护了一个链表数组，我们用下面一幅图来看下ConcurrentHashMap的内部结构：

![https://cdn.nlark.com/yuque/0/2023/png/22813151/1672801761653-4021354e-c38c-4fd2-975a-01a734e5d2cb.png](https://cdn.nlark.com/yuque/0/2023/png/22813151/1672801761653-4021354e-c38c-4fd2-975a-01a734e5d2cb.png)

ConcurrentHashMap是由Segment数据结构和HashEntry数据结构组成:

- Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap中扮演锁的角色
- HashEntry则用于存储键值对数据。

一个ConcurrentHashMap中包含一个Segment数组，Segment结构和HashMap类似，是一种数组和链表结构，一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组中的数据进行修改时，必须先获得它对应的Segment锁

ConcurrentHashMap定位一个元素需要进行两次Hash操作，第一次Hash定位到Segment，第二次Hash定位到元素所在的链表的头部，因此，这一种结构带来的副作用是Hash的过程要比普通的HashMap要长，但是带来的好处是写操作的时候可以只对元素所在的Segment进行加锁即可，不会影响到其他的Segment，这样，在最理想的情况下，ConcurrentHashMap可以最高同时支持Segment数量大小的写操作，所以通过这种结构，ConcurrentHashMap的并发能力可以得到很大的提高

### Segment

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {
    // Segment中元素的数量
    transient volatile int count;
    // 对table的大小造成影响的操作的数量（比如put或者remove操作）
    transient int modCount;
    // 阈值，Segment里面元素的数量超过这个值依旧就会对Segment进行扩容
    transient int threshold;
    // 链表数组，数组中的每一个元素代表了一个链表的头部
    transient volatile HashEntry<K,V>[] table;
    // 负载因子，用于确定threshold
    final float loadFactor;
}
```

- count - 用来统计该段数据的个数，它是volatile变量，它用来协调修改和读取操作: 每次修改操作做了结构上的变化，如增加/删除节点（修改节点的值不算结构上的变化），都要写count值，每次读取操作开始都要读取count的值
- modCount - 统计段数据改变的次数，主要为了检测对多个段进行遍历过程中某个段是否发生改变
- threashold - 用来表示需要进行rehash的界限值
- table - 数组存储段中节点，每个数组是个hash链，用HashEntry表示。table也是volatile，这使得能够读取到最新的table值而不需要同步

### HashEntry

```java
static final class HashEntry<K,V> {
    final K key;
    final int hash;
    volatile V value;
    final HashEntry<K,V> next;
}
```

可以看到HashEntry的一个特点，除了value外，其他的几个变量都是final修饰的，这意味着不能从hash链表的尾部和中部进行添加或删除节点，因为这需要修改next引用值，所有的节点的修改只能从头部开始。

对于put操作，可以一律添加到hash链表的头部。但是对于remove操作，可能需要从中间删除一个节点，这时候就需要把将要删除的节点的前面的节点都复制一遍，最后一个节点指向要删除的节点的下一个节点。

为了确保读操作能看到最新的值，将value设置为volatile，这避免了加锁。

### ConcurrentHashMap的初始化

```java
public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    int sshift = 0;
    int ssize = 1;
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    segmentShift = 32 - sshift;
    segmentMask = ssize - 1;
    this.segments = Segment.newArray(ssize);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = 1;
    while (cap < c)
        cap <<= 1;
    for (int i = 0; i < this.segments.length; ++i)
        this.segments[i] = new Segment<K,V>(cap, loadFactor);
}
```

CurrentHashMap的初始化一共有三个参数：

- initialCapacity - 表示初始的容量
- loadFactor - 表示负载参数
- concurrentLevel - 代表ConcurrentHashMap内部Segment的数量，ConcurrentLevel一经指定，不可该变，后续如果ConcurrentHashMap的元素数量增加导致ConcurrentHashMap需要扩容，ConcurrentHashMap不会增加Segment的数量，只会增加Segment中链表数组的容量，这样扩容的好处是不需要对ConcurrentHashMap中所有的元素做rehash，只需要对Segment中的元素做一次rehash就可以

默认初始表的容量，必须是2的幂（至少为1），最大为16

### ConcurrentHashMap的get操作

```java
 public V get(Object key) {
     int hash = hash(key.hashCode());
     return segmentFor(hash).get(key, hash);
 }
```

第二行，对hash值进行了第二次hash，之所以要再次hash，是为了减少hash冲突，使元素能够均匀的分布在不同Segment上，从而提高容器的存取效率。

第三行，segmentFor这个函数用于确定操作应该在哪一个Segment中进行，几乎对ConcurrentHashMap的所有操作都需要用到找个函数。

```java
final Segment<K,V> segmentFor(int hash) {
     return segments[(hash >>> segmentShift) & segmentMask];
}
```

这个函数用了位操作来确定Segment，根据传入的hash值向右无符号右移segmentShift位，然后和segmentMask进行与操作，假设Segment的数量是2的n次方，根据元素的hash值的高n位就可以确定元素到底在哪一个Segment中

```java
V get(Object key, int hash) {
      if (count != 0) { // read-volatile // ①
          HashEntry<K,V> e = getFirst(hash);
          while (e != null) {
              if (e.hash == hash && key.equals(e.key)) {
                  V v = e.value;
                  if (v != null) // ② 注意这里
                      return v;
                  return readValueUnderLock(e); // recheck
             }
             e = e.next;
         }
     }
     return null;
 }
```

这个实现很微妙，没有锁同步的话，靠什么保证同步呢？

第一步，先判断一下 count != 0；count变量表示segment中存在entry的个数。如果为0就不用找了。假设这个时候恰好另一个线程put或者remove了这个segment中的一个entry，会不会导致两个线程看到的count值不一致呢？看一下count 变量的定义：

```java
transient volatile int count;
```

它使用了volatile来修改。在Java5之后，JMM实现了对volatile的保证：对volatile域的写入操作happens-before于每一个后续对同一个域的读写操作。所以，每次判断count变量的时候，即使恰好其他线程改变了segment也会体现出来

第二步，获取到要该key所在segment中的索引地址，如果该地址有相同的hash对象，顺着链表一直比较下去找到该entry。当找到entry的时候，先做了一次比较： if(v != null) 我们用红色注释的地方。这是为何呢？考虑一下，如果这个时候，另一个线程恰好新增/删除了entry，或者改变了entry的value，会如何？

前面说过HashEntry类的结构，除了 value，其它成员都是final修饰的，也就是说value可以被改变，其它都不可以改变，包括指向下一个HashEntry的next也不能被改变。

（1）**在get代码的①和②之间，另一个线程新增了一个entry。如果另一个线程新增的这个entry又恰好是我们要get的**，这事儿就比较微妙了。下图大致描述了put 一个新的entry的过程。

![https://cdn.nlark.com/yuque/0/2023/webp/22813151/1672803403748-f647609a-0824-44d8-8594-34e9bc43b037.webp](https://cdn.nlark.com/yuque/0/2023/webp/22813151/1672803403748-f647609a-0824-44d8-8594-34e9bc43b037.webp)

因为每个HashEntry中的next也是final的，没法对链表最后一个元素增加一个后续entry所以新增一个entry的实现方式只能通过头结点来插入了。

newEntry对象是通过 new HashEntry(K k , V v, HashEntry next) 来创建的。如果另一个线程刚好new 这个对象时，当前线程来get它。因为没有同步，就可能会出现当前线程得到的newEntry对象是一个没有完全构造好的对象引用。没有锁同步的话，new 一个对象对于多线程看到这个对象的状态是没有保障的，这里同样有可能一个线程new这个对象的时候还没有执行完构造函数就被另一个线程得到这个对象引用。所以才需要判断一下：if (v != null) 如果确实是一个不完整的对象，则使用锁的方式再次get一次。

**有没有可能会put进一个value为null的entry？ 不会的，已经做了检查，这种情况会抛出异常**，所以 ②处的判断完全是出于对多线程下访问一个new出来的对象的状态检测。

（2）在get代码的①和②之间，另一个线程修改了一个entry的value。value是用volitale修饰的，可以保证读取时获取到的是修改后的值。

（3）在get代码的①之后，另一个线程删除了一个entry。

假设我们的链表元素是：e1-> e2 -> e3 -> e4 我们要删除 e3这个entry，因为HashEntry中next的不可变，所以我们无法直接把e2的next指向e4，而是将要删除的节点之前的节点复制一份，形成新的链表。它的实现大致如下图所示：

![https://cdn.nlark.com/yuque/0/2023/webp/22813151/1672803403788-a4c7133a-139c-4e4e-b932-07d12c857a2e.webp](https://cdn.nlark.com/yuque/0/2023/webp/22813151/1672803403788-a4c7133a-139c-4e4e-b932-07d12c857a2e.webp)

如果我们get的也恰巧是e3，可能我们顺着链表刚找到e1，这时另一个线程就执行了删除e3的操作，而我们线程还会继续沿着旧的链表找到e3返回。这里没有办法实时保证了。

我们第①处就判断了count变量，它保障了在 ①处能看到其他线程修改后的。①之后到②之间，如果再次发生了其他线程再删除了entry节点，就没法保证看到最新的了。不过这也没什么关系，即使我们返回e3的时候，它被其他线程删除了，暴漏出去的e3也不会对我们新的链表造成影响。

这其实是一种乐观设计，设计者假设 ①之后到②之间 发生被其它线程增、删、改的操作可能性很小，所以不采用同步设计，而是采用了事后（其它线程这期间也来操作，并且可能发生非安全事件）弥补的方式。而因为其他线程的“改”和“删”对我们的数据都不会造成影响，所以只有对“新增”操作进行了安全检查，就是②处的非null检查，如果确认不安全事件发生，则采用加锁的方式再次get。这样做减少了使用互斥锁对并发性能的影响。可能有人怀疑remove操作中复制链表的方式是否代价太大，这里我没有深入比较，不过既然Java5中这么实现，我想new一个对象的代价应该已经没有早期认为的那么严重。

使用和性能对比:

[https://github.com/HelloAnner/demo_for_java/blob/master/src/main/java/com/anner/map/ConcurrentHashmapPerformance.java](https://github.com/HelloAnner/demo_for_java/blob/master/src/main/java/com/anner/map/ConcurrentHashmapPerformance.java)

[https://github.com/HelloAnner/demo_for_java/blob/master/src/main/java/com/anner/map/ConcurrentHashmapUseDemo.java](https://github.com/HelloAnner/demo_for_java/blob/master/src/main/java/com/anner/map/ConcurrentHashmapUseDemo.java)

### ConcurrentHashMap JDK1.8与JDK1.7的区别

JDK1.8 中 ConcurrentHashMap 类取消了 Segment 分段锁，采用 CAS + synchronized 来保证并发安全，数据结构跟 jdk1.8 中 HashMap 结构类似，都是**数组 + 链表(当链表长度大于8时，链表结构转为红黑二叉树**)结构。

ConcurrentHashMap 中 synchronized 只锁定当前链表或红黑二叉树的首节点，只要节点 hash 不冲突，就不会产生并发，相比 JDK1.7 的 ConcurrentHashMap 效率又提升了 N 倍！

# 参考

[https://juejin.cn/post/7070397572627562504](https://juejin.cn/post/7070397572627562504)