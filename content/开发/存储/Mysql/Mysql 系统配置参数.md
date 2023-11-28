
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