
Redis 中允许存在多个数据库db

```python
struct redisServer {

redisDb *db; // 指向 db 数组的指针

int dbnum; // 服务器中数据库数量

}
```

如果需要切换数据库 ， 可以直接选择 select 2 等操作

其作用就是修改 redisClient 对象中指向 当前使用的数据库的指针

```python
struct redisClient {

redisDb *db;
}
```

### 数据库键空间

redis 中的键空间就是这个数据库中存储的所有的键，每一个键都是字符串对象；

键空间的值就是数据库空间的值，可以是之前涉及到的五种数据结构。

比较有趣的就是，键空间和值本身就是一个字典的结构；

然后字典的数组的entry指向的数据结构可以是不同的数据存储结构。

所以对键进行一些基础的更新、删除操作，本质还是操作字典的key

### 过期

```python
set key value
expire key 5 # 5s后过期
expireat key 129899089 # 指定一个具体的时间节点过期
ttl key # 查看过期时间 - 秒
pttl key # 查看过期时间 - 毫秒
```

键的过期时间自然也是需要保存的，称之为 ”过期字典“

键是一个指针，指向键空间的一个键对象，值是一个 long long 类型的整数，表示一个毫秒级别 unix 时间戳

```python
persist messgae # 移除过期时间的限制
```

那么如果一个键过期，其删除策略是什么呢？

- 定时删除： 当键到期的时候立即删除，所以cpu 时间不友好 ， 无法高效处理
- 惰性删除： 取出键的时候才判断是不是过期了，但是内存不友好，过期的键依旧会保存在内存中
- 定期删除： 定期删除 ， 即按照一定的时间和频率去删除过期键，相当于是一个折中的策略

那么在实际使用中， redis 使用的是 惰性删除和定期删除的策略

即当开始获取一个键的时候，会走一个 expireIfNeeded 的过滤逻辑，如果已经过期，会立即进行删除；

还存在一个 redis.c/serverCron 函数定期的删除过期的键值；函数运行的时候会取出一定数量的随机键值进行检查。

如果需要对Redis对键值进行保存的时候，需要自动过滤掉已经过期的key，比如生成 rdb 文件的时候会忽略，当开始载入rdb 文件的时候， 已经过期的键会被忽略。

当aof模式运行的时候，键过期会追加一个 del 命令

当复制模式下运行的时候， 主服务器会发送 del 命令到其他服务器去删除