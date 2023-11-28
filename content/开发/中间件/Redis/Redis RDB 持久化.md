
Redis 作为内存数据库 ， 提供了基础的 RDB 持久化的机制。

RDB文件是一个压缩的二进制文件保存在磁盘之中。

```java
SAVE # 阻塞式创建 RDB文件

BGSAVE # 创建子进程创建 RDB 文件
```

Redis 启动的时候会自动的加载生成的RDB文件，所以没有显式的load rdb 文件的命令。

当Redis启动的时候，如果开启了AOF持久化功能，会默认载入AOF文件，否则继续载入 RDB 文件。

当开启 BGSAVE 之后， 会开启一个子进程开始持久化，这个过程中，系统不会允许第二个 BGSAVE SAVE 命令的执行，甚至 BGREWRITEAOF 也不被允许， 因为存在多个子进程开始读写IO 和 多个进程开始竞争不是一个很好的选择。

### 自动间隔保存

```bash
save 300 10 # 300秒内修改10次就会开启 bgsave 后台保存
```

redis 的默认save条件

```bash
save 900 1
save 300 10
save 60 10000
```

每一个save配置选项 ，都会保存在 saveParam 数组中

```bash
struct redisServer {

struct saveParam *saveparams;

long long dirty; # 距离上一次save后的数据修改次数

time_t lastsave; # 上一次执行保存的时间

}
```

那么当修改次数达到配置的上线的时候，如何进行触发？

最直观的我理解应该是 在 dirty 修改的时候去判断保存配置，可能是成本太高。

redis 采取的是一个 serverCron 函数来定期维护系统的状态，包括 saveParam 状态

### RDB 文件结构

rdb文件是一个二进制组织结构的文件

- 文件标识: RDB 文件开头是 REDIS 部分，采用 5字节保存 ，标识是一个 RDB 文件
- db_version : 记录一下文件记录的数据库版本 ， 长度为 4 字节 ，表示一个字符串表示的整数 比如 ”0006“
- database : 零个或者任意个数据库的数据的内容，如果为空表示0字节 ， 每一个数据库由下面几个部分组成:
    - selectdb ： 一字节 ，表示接下来读取的是数据库号码
    - db_number : 保存一个数据库号码
    - key_value_pairs : 保存键值对
- EOF: 1 字节的常量，表示正文内容的结束，表示文件已经载入结束了
- check_sum : 8字节长度的无符号整数，保存一个校验和 ， 计算一下前面的四个部分的内容。

下面细说一下 key_value_pairs :

在内存中存储的结构中，redis数据基于一个字典存储多个key 和value 的结构；同时使用一个过期字典保存键和过期时间。

但是在 rdb 文件之中，过期时间和真实的键值都同时保存在 key_value_pairs 之中 ， 由下面三个方面组成:

- TYPE : 1字节表示类型 ， 表示数据结构类型和存储结构
    
    ```bash
    REDIS_RDB_TYPE_STRING
    REDIS_RDB_TYPE_LIST
    REDIS_RDB_TYPE_SET
    REDIS_RDB_TYPE_ZSET
    REDIS_RDB_TYPE_HASH
    REDIS_RDB_TYPE_LIST_ZIPLIST
    REDIS_RDB_TYPE_SET_INTSET
    REDIS_RDB_TYPE_ZSET_ZIPLIST
    REDIS_RDB_TYPE_HASH_ZIPLIST
    
    EXPIRETIME_MS
    ```
    
- KEY
    
- VALUE