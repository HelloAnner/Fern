```shell
docker run -d --name redis -v /Users/anner/Docker/redis/redis.conf:/usr/local/etc/redis/redis.conf -p 6379:6379 redis redis-server /usr/local/etc/redis/redis.conf
```

配置自己启动
```
docker update --restart unless-stopped redis
```

关于配置的解释 [[../../中间件/Redis/Redis 安装启动|Redis 安装启动]]


