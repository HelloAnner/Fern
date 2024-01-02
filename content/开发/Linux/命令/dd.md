用指定大小的块拷贝一个文件，并在拷贝的同时进行指定的转换。可以用于测试磁盘命令、数据备份或恢复等

```
if=file 　　　　　　　　输入文件名，缺省为标准输入。 从file读取，如if=/dev/zero,该设备无穷尽地提供0,（不产生读磁盘IO）
of=file 　　　　　　　　输出文件名，缺省为标准输出。 向file写出，可以写文件，可以写裸设备。如of=/dev/null，"黑洞"，它等价于一个只写文件. 所有写入它的内容都会永远丢失. （不产生写磁盘IO）
ibs=bytes 　　　　　　　一次读入 bytes 个字节(即一个块大小为 bytes 个字节)。
obs=bytes 　　　　　　　一次写 bytes 个字节(即一个块大小为 bytes 个字节)。
bs=bytes 　　　　　　　同时设置读写块的大小为 bytes ，可代替 ibs 和 obs。如bs=8k 每次读或写的大小，即一个块的大小为8K。
cbs=bytes 　　　　　　 一次转换 bytes 个字节，即转换缓冲区大小。
skip=blocks 　　　　　从输入文件开头跳过 blocks 个块后再开始复制。
seek=blocks      　　从输出文件开头跳过 blocks 个块后再开始复制。(通常只有当输出文件是磁盘或磁带时才有效)。
count=blocks 　　　　仅拷贝 blocks 个块，块大小等于 ibs 指定的字节数。
iflag=FLAGS　　　　　 指定读的方式FLAGS，参见“FLAGS参数说明”
oflag=FLAGS　　　　　 指定写的方式FLAGS，参见“FLAGS参数说明”

conv=conversion：用指定的参数转换文件。
ascii：转换ebcdic为ascii
ebcdic：转换ascii为ebcdic
ibm：转换ascii为alternate ebcdic
block：把每一行转换为长度为cbs，不足部分用空格填充
unblock：使每一行的长度都为cbs，不足部分用空格填充
lcase：把大写字符转换为小写字符
ucase：把小写字符转换为大写字符
swab：交换输入的每对字节
noerror：出错时不停止
notrunc：不截短输出文件
sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。

flags参数
```



/dev/null，它是空设备，也称为位桶（bit bucket）、回收站、无底洞，可以向它输出任何数据。任何写入它的输出都会被抛弃。如果不想让消息以标准输出显示或写入文件，那么可以将消息重定向到位桶。

/dev/zero，是一个输入设备，可用它来初始化文件。该设备无穷尽地提供0，可以使用任何需要的数目——设备提供的要多的多。他可以用于向设备或文件写入字符串0。


测试磁盘写能力
```shell
time dd if=/dev/zero of=test.log bs=64k count=4k oflag=direct
```

/dev//zero是一个伪设备，它只产生空字符流，对它不会产生IO，所以，IO都会集中在of文件中，of文件只用于写，所以这个命令相当于测试磁盘的写能力。命令结尾添加oflag=direct将跳过内存缓存，添加oflag=sync将跳过hdd缓存



测试同时读写能力
```shell
time dd if=/dev/sdb of=/testrw.dbf bs=64k
```
一个是物理分区，一个是实际的文件，对它们的读写都会产生IO（对/dev/sdb是读，对/testrw.dbf是写），假设它们都在一个磁盘中，这个命令就相当于测试磁盘的同时读写能力


- [ ] 复习 dd (@2024-01-23)