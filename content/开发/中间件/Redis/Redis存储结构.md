对redis来说，所有的key（键）都是字符串。我们在谈基础数据结构时，讨论的是存储值的数据类型，主要包括常见的5种数据类型，分别是：String、List、Set、Zset、Hash

![[Pasted image 20231118100500.png]]

Redis 存储的字符内容是基于 C语言格式的存储模式，基于此之上。做了一些改良。

即由 free字段 + len 字段 + buffer 存储一个基础的SDS字符串格式（简单动态字符串 ）。

于是获取长度的时间复杂度为 O(1) , 当缓冲区快要溢出的时候，可以提前分配空间。

对于linked list 的格式，使用的是双端无环链表存储的。

对于hash tables , 使用的是经典的hash 结构，即数组加链表的形式。对下标计算hash确定在数组的位置。

需要注意的是，对于redis ，是存在多个hashtable 存储的，需要扩容的时候，会渐进式的将内容转移到下一个 hashtable 数组的位置。

即维持一个 rehashidx ， 每一次对字典做操作的时候均为 rehash 此刻的值对应的key转移到新的hashtable 中去，一一转移。

对于有序集合 sorted sets ， 使用调表的存储结构作为底层结构。

---

因为redis 不限制集合的存储内容，所以为了机制的内存占用考虑，对一些情况做了一些优化，如下：

（可以看到，对于 intset 或者 ziplist 这些优化的数据结构，均记录了类似 编码、长度等字段，方便做一些获取全部元素大小的操作）

如果集合只有整数，数量不多的情况下，会使用整数集合（ intset）。

```java
sadd numbers 1 3 5 7 9
```

整数集合由：

- encoding : uint32 - redis 可以按照存储数字的格式来升级 encoding大小 ， 即重新分配数组，元素转换对应的类型。
- length : uint32
- contents[] int8

可以使用查看底层的存储结构

```java
object encoding ***
```

---

如果列表中存储的是小数字、短的字符串，可以使用 ziplist 存储：

宏观上

- zlbytes ： 记录 zip list 的大小
- zltail ： 记录 zip list 尾结点的位置
- zllen: 记录 zip list 的结点数量
- entry1 : 存储内容的实际结点
- entry2…..

对具体的entry：

- previous_entry_length ： 前一个节点的大小 ，存储空间占用 1字节或者 5 字节; 可以计算出前一个节点的位置；完成从表尾到表头的计算。
- encoding : 记录数据类型和数据的长度 。 比如 11 0000 00 为 int16类型的整数
- content : 节点记录的值 ， 字节数组或者存储的是整数

试想一下，如果存在多个连续的 250-253 字节的entry ， 当一个entry长度增长，那么后面的entry 中记录 previous_entry_length 都会增加，相互关联增长。

但是这个场景比较的低频。