
按照配置文件启动
```shell
redis-server /home/anner/redis-4.0.9/redis.conf
```
### conf 文件

```conf
daemonize yes 
port 6379 
requirepass redispassword
```


### 客户端连接
```shell
auth redispassword

./redis-cli -h 127.0.0.1 -p 6379 -a redispassword
```

