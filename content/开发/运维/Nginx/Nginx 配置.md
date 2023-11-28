### 后台运行

```conf
daemon on;
```

```shell
# 检查  nginx 配置
nginx -t
# 重新加载
nginx -s reload
```

### `upstream` 模块

`upstream` 模块用于定义后端服务器的集群。`upstream` 模块可以接受一些参数，以配置集群的行为和负载均衡策略 (**默认是轮询策略**)

以下是一些常用的 `upstream` 参数：

1. `ip_hash`：使用客户端 IP 地址进行负载均衡，确保同一个客户端的请求始终被发送到同一台后端服务器。如果这个节点挂了，那么就顺延到下一个机器
    
2. `least_conn`：根据当前连接数选择连接数最少的后端服务器。
    
3. `weight`：设置后端服务器的权重，用于分配负载。值越高，被选中的概率越大。
    
4. `backup`：将后端服务器定义为备份服务器。只有在其他所有主服务器不可用时才会将请求转发到备份服务器。
    
5. `max_fails` 和 `fail_timeout`：用于定义故障转移。`max_fails` 定义了在多少次连续失败后将后端服务器标记为不可用，`fail_timeout` 定义了标记为不可用后的暂停时间。
    
6. `keepalive`：启用或禁用与后端服务器的 keepalive 连接。

```conf
upstream FR.com {  
        #max_fails=n fail_timeout=m 表示m时间内超时n次失败尝试，将会在接下来m时间内标记此节点不可用（m时间过后无论此节点是否启动都会被重新标记为可用状态）  
        #其中fails为“失败尝试”，失败尝试由下面server配置中，proxy_next_upstream指令定义  
        server 127.0.0.1:8060 max_fails=15 fail_timeout=300s;  
        server 127.0.0.1:8061 max_fails=15 fail_timeout=300s;  
        keepalive 300;  
        #其他server参数说明：  
        #down 标记服务器挂掉  
        #backup 备份服务器，当主服务器（例如上面的95和96）不可用时才加入服务器；  
        #weight=number 权重，默认为1  
        #内置负载均衡策略有ip hash、轮询、加权轮询（设置server的weight值）  
        ip_hash;  
}
```

```conf
http {
  upstream my_cluster {
    server backend1.example.com weight=2;
    server backend2.example.com weight=3;
    server backend3.example.com weight=1;
  }
}
```

```conf
http {
  upstream my_cluster {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com backup;
  }
}
```