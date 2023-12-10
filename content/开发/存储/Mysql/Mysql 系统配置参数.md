
### Mysql 配置说明


### max_allowed_packet

mysql服务器端和客户端在一次传送数据包的过程当中最大允许的数据包大小

经典报错

```cpp
Packet for query is too large (20682943>1048576). You can change this value on the server by setting the max_allowed_packet’ variable.
```

业务上还遇到了

```cpp
Transaction is timeout
```

```cpp
show variables like ‘%max_allowed_packet%’;
select @@max_allowed_packet;
```

```cpp
set global max_allowed_packet = 500 * 500 * 1024
```

通过命令行方式修改时，不能用M、G，只能这算成字节数设置。使用配置文件修改才允许设置M、G单位

命令行修改之后，需要退出当前回话(关闭当前mysql server链接)，然后重新登录才能查看修改后的值。通过命令行修改只能临时生效，如果下次数据库重启后对应的配置就会又复原了，因为重启的时候加载的是配置文件里面的配置项。

max_allowed_packet 最大值是1G(1073741824)，如果设置超过1G，查看最终生效结果也只有1G

### 连接数配置

#### 固定连接数配置

```sql
show variables like '%max_connections%';
```

```
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| max_connections        | 151   |
| mysqlx_max_connections | 100   |
+------------------------+-------+
```

- max_connections  传统的连接数量限制 , 默认是 151 个 ，最大 100000
- mysqlx_max_connections 限制使用 Mysql X Protocol 连接方式的数量; MySQL X Protocol 是一种基于 X DevAPI 的新协议，提供了更加灵活、强大的功能和性能。默认情况下，mysqlx_max_connections的值为 100

临时修改
```sql
set global max_connections = $DesiredValue;
```

配置文件修改
```c
[mysqld]
max_connections = 1000
```

#### 实时连接数量

```sql
SELECT
    SUBSTRING_INDEX(HOST, ':', 1) AS machine,
    USER AS username,
    IF(State = 'User', 'activate', 'inactivate') AS connection_status,
    COUNT(*) AS connections
FROM
    information_schema.processlist
GROUP BY
    machine,
    username,
    connection_status;
```

```
+-------------+-----------------+-------------------+-------------+
| machine     | username        | connection_status | connections |
+-------------+-----------------+-------------------+-------------+
| 10.211.55.2 | moss            | inactivate        | 7           |
| localhost   | event_scheduler | inactivate        | 1           |
+-------------+-----------------+-------------------+-------------+
```

information_schema.processlist 关于**当前活动连接的详细信息**，包括连接的 ID、用户、主机、状态


查看现在的所有连接信息
```sql
show full processlist;
```


```sql
show status like "%connect%";
```
- `Connections`：已建立的连接数。
- `Max_used_connections`：达到的最大连接数。
- `Aborted_connects`：连接尝试失败的次数。
- `Threads_connected`：当前连接的线程数。
- `Threads_created`：已创建的线程数。
- `Threads_running`：正在运行的线程数
查看当前状态的连接信息

### Thread

```sql
show status where `variable_name` like 'Threads_%';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Threads_cached    | 8     |
| Threads_connected | 1     |
| Threads_created   | 67    |
| Threads_running   | 2     |
+-------------------+-------+
```

Threads_running ：这个数值指的是激活的连接数，这个数值一般远低于connected数值. **准确的来说，Threads_running是代表当前并发数**

**Threads_connected 跟show processlist结果相同，表示当前连接数。**

Threads_created：创建过的线程数

如果发现Threads_created值过大的话，表明MySQL服务器一直在创建线程，这也是比较耗资源，可以适当增加配置文件中thread_cache_size值，查询服务器thread_cache_size的值：

```sql
show variables like 'thread_cache_size';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| thread_cache_size | 100   |
+-------------------+-------+
1 row in set (0.00 sec)
```

如果我们在MySQL服务器配置文件中设置了thread_cache_size，当客户端断开之后，服务器处理此客户的线程将会缓存起来以响应下一个客户而不是销毁(前提是缓存数未达上限)。