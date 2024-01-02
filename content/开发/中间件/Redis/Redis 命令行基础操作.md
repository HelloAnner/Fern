## redis_string

- 设置键值对：`SET key value`
- 获取键值对：`GET key`
- 批量设置键值对：`MSET key1 value1 key2 value2 ...`
- 批量获取键值对：`MGET key1 key2 ...`
- 自增：`INCR key`
- 自减：`DECR key`
- 追加字符串到已有字符串的末尾：`APPEND key value`

## redis_list

- 在列表左侧添加一个或多个元素：`LPUSH key element1 element2 ...`
- 在列表右侧添加一个或多个元素：`RPUSH key element1 element2 ...`
- 弹出并返回列表左侧的第一个元素：`LPOP key`
- 弹出并返回列表右侧的第一个元素：`RPOP key`
- 获取列表指定范围内的元素：`LRANGE key start stop`
- 获取列表中元素的个数：`LLEN key`

## redis_hash

- 设置哈希表中的一个字段的值：`HSET key field value`
- 获取哈希表中指定字段的值：`HGET key field`
- 获取哈希表中所有字段和值：`HGETALL key`
- 获取哈希表中所有字段名：`HKEYS key`
- 获取哈希表中所有值：`HVALS key`
- 获取哈希表中字段数量：`HLEN key`

## redis_set

- 向集合添加一个或多个成员：`SADD key member1 member2 ...`
- 从集合中移除一个或多个成员：`SREM key member1 member2 ...`
- 获取集合中的所有成员：`SMEMBERS key`
- 获取集合中成员的数量：`SCARD key`
- 判断一个成员是否在集合中：`SISMEMBER key member`

## redis_zset

- 向有序集合添加一个元素：`ZADD key score member`
- 获取有序集合中指定范围内的元素：`ZRANGE key start stop [WITHSCORES]`
- 获取有序集合中指定成员的排名：`ZRANK key member`
- 获取有序集合中成员数量：`ZCARD key`
- 获取有序集合中指定成员的分数：`ZSCORE key member`

- [ ]  复习 Redis 基础命令行 (@2024-01-16)