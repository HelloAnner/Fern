 [-aHS][-c <core文件上限>][-d <数据节区大小>][-f <文件大 小>][-m <内存大小>][-n <文件数目>][-p <缓冲区大小>][-s <堆栈大小>][-t <CPU时间>][-u <程序数目>][-v <虚拟内存大小>]

- H 设置硬件资源限制,是管理员所设下的限制.
- S 设置软件资源限制,是管理员所设下的限制.
- a 显示当前所有的资源限制.

- u 进程数目:用户最多可启动的进程数目.
- c size:设置core文件的最大值.单位:blocks
- d size:设置程序数据段的最大值.单位:kbytes
- f size:设置shell创建文件的最大值.单位:blocks
- l size:设置在内存中锁定进程的最大值.单位:kbytes
- m size:设置可以使用的常驻内存的最大值.单位:kbytes
- n size:设置内核可以同时打开的文件描述符的最大值.单位:n  - [Linux 文件描述符](https://www.notion.so/Linux-8536202890bd445cb64de599715d4c09?pvs=21)
- p size:设置管道缓冲区的最大值.单位:kbytes
- s size:设置堆栈的最大值.单位:kbytes
- t size:设置CPU使用时间的最大上限.单位:seconds
- v size:设置虚拟内存的最大值.单位:kbytes

```bash
ulimit -u 10000 # 默认1024

ulimit -n 4096 # 默认 1024

ulimit -d unlimited

ulimit -t unlimited
```

上述修改暂时地，适用于通过 ulimit 命令登录 shell 会话期间

永久地，通过将一个相应的 ulimit 语句添加到由登录 shell 读取的文件中， 即特定于 shell 的用户资源文件

### 方式一 : 修改 /etc/security/limits.conf

```bash

        * soft noproc 11000

        * hard noproc 11000 # 最大进程数

        * soft nofile 4100 # 最大文件打开数

        * hard nofile 4100
```

注意部分系统是nproc

* 代表针对所有用户

### 方式二: 修改 /etc/profile

```bash
ulimit -u 10000

ulimit -n 4096

ulimit -d unlimited 

ulimit -m unlimited 

ulimit -s unlimited 

ulimit -t unlimited 

ulimit -v unlimited
```

通过 ulimit -n或者ulimit -a 查看系统的最大文件打开数已经生效了。但此时查看进程的最大文件打开数没有变，原因是这个值是在进程启动的时候设定的，要生效必须重启！

遇到limits修改不生效的时候，请查一下进程是否只是子进程，如果是，那就要把父进程也一并重启才可以

在linux下，每个进程的limit信息保存在/proc/PID/limits文件中(linux OS kenerl > 2.6.24)。低于2.6.24版本的kenerl需要手动统计 /proc/PID/fd下面有多个少个文件

## 参考

[Linux中的文件描述符与打开文件之间的关系 - Smah - 博客园](https://www.cnblogs.com/still-smile/p/11645960.html)


- [ ]  复习 ulimit (@2024-01-30)