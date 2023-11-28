
Redis 中基于之前的数据类型的存储结构，来构建几个经典的对象。

redis object 主要是由 type 、 encoding 、ptr 三个属性组成。

type 主要分为: redis_string redis_list redis_hash redis_set redis_zset [Redis 命令行基础操作](https://www.notion.so/Redis-05283e50de3e4d94a5df5f230772c42d?pvs=21)

```bash
set msg "hello world"
type msg # string

rpush numbers 1 3 5
type numbers  # list

hmset profile name Tom age 25
type profile # hash

sadd fruits apple banana
type fruits # set

zadd price 8.5 apple
type price # zset
```

**对于 redis ，一个对象使用的底层数据结构可能不止一种 ， 主要为了针对不同的数据量和数据结构来使用对应的存储结构，完成节省内存的目的。**

比如 redis_list 可能使用 redis_encoding_ziplist 和 redis_encoding_linkedlist 实现

```bash
object encoding msg  # embstr
```

redis 存储的对象宏观上需要一个 redis object 对象来记录类型和指向的指针位置，然后对于存储还继续需要一个具体的存储结构来表示

所以一个对象声明需要两次申请内存结构。

### 字符串对象

**字符串对象的编码有 int 、raw(sds)、embstr**

如果字符串保存到是整数类型，可以使用long表示的，那么字符串的编码为 int

```bash
set number 10086
```

如果长度小于 39字节，直接使用 embstr 格式存储，好处是不需要申请 redis object 对象内存，只需要申请一次 embstr 内存，且是连续格式的内存机构

（如果是 浮点数类型的存储，也是使用的 embstr 格式存储）

如果存储是字符类型，长度大于 39 字节，使用raw 格式存储，即SDS

如果修改了存储的内容，使得其长度或者类型发生了变化，那么自然就需要编码格式的转换

```bash
127.0.0.1:6379> set number 10086
OK
127.0.0.1:6379> get number
"10086"
127.0.0.1:6379>
127.0.0.1:6379>
127.0.0.1:6379> type number
string
127.0.0.1:6379> object encoding number
"int"
127.0.0.1:6379> append number " is a good number"
(integer) 22
127.0.0.1:6379> get number
"10086 is a good number"
127.0.0.1:6379> object encoding number
"raw"
127.0.0.1:6379>
```

### 列表对象

**redis 的列表对象的编码可以是 ziplist 和 linkedlist**

什么时候使用 ziplist:

- 列表对象存储的字符串长度都小于 64 字节
- 列表对象存储的元素数量小于 512 个

### 哈希对象

**redis 的hash 对象的存储编码可以是 ziplist 和 hashtable**

什么时候使用 ziplist:

- 哈希对象所有键值对的字符长度小于 64字节
- 哈希对象保存的键值对数量小于 512个

```bash
127.0.0.1:6379> hset person name anner
(integer) 1
127.0.0.1:6379> object encoding person
"ziplist"
127.0.0.1:6379>
```

对于 ziplist ， 一个键值对和k,v 存储在相邻的两个 entry上。

### 集合对象

**集合对象的编码可以是 intset 和 hashtable**

什么时候使用 intset:

- 所有元素都是整数值
- 元素数量不超过 512 个

当使用 hashtable 作为底层的存储结构的时候，使用的存储结构的键作为底层的存储结构，值设置为 null

### 有序集合对象

**有序集合对象的编码可以是 ziplist 和 skiplist**

当使用 ziplist 的时候， 元素内容和分支存储在相邻的两个 entry 上。

如果使用 skiplist 作为存储结构，其底层会使用 字典和跳跃表 同时完成有序集合的存储：

兼顾 O(1) 查找成员 ， 和 批量获取有序内容。 使用 hashtable 存储元素值和分值的映射，使用 skiplist 有序存储集合元素。

- 如果仅仅使用 hashtable : 可以按照值获取到对应的分值， zscore 命令。 但是无法批量有序获取成员
- 如果仅仅使用skiplist : 无法 O(1) 获取一个成员的分值

即字典中对于多个元素的排列使用跳跃表存储，而不是之前传统的链表。

什么时候使用 skiplist:

- 元素数量 小于 128 个
- 所有元素长度都小于 64 字节

此外，对于对象还存在内存回收、对象共享的操作。