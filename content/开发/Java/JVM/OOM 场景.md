
### OOM原因

**内存泄露**：**申请使用完的内存没有释放**，导致虚拟机不能再次使用该内存，此时这段内存就泄露了，因为申请者不用了，而又不能被虚拟机分配给别人用。

**内存溢出**：**申请的内存超出了JVM能提供的内存大小**，此时称之为溢出

按照JVM规范，除了程序计数器不会抛出OOM外，其他各个内存区域都可能会抛出OOM


### OOM 分类
最常见的OOM情况有以下三种：

- java.lang.OutOfMemoryError: Java heap space ------>java堆内存溢出，此种情况最常见，一般由于内存泄露或者堆的大小设置不当引起。对于内存泄露，需要通过内存监控软件查找程序中的泄露代码，而堆大小可以通过虚拟机参数-Xms,-Xmx等修改。

- java.lang.OutOfMemoryError: PermGen space ------>java永久代溢出，即方法区溢出了，一般出现于大量Class或者jsp页面，或者采用cglib等反射机制的情况，因为上述情况会产生大量的Class信息存储于方法区。此种情况可以通过更改方法区的大小来解决，使用类似-XX:PermSize=64m -XX:MaxPermSize=256m的形式修改。另外，过多的常量尤其是字符串也会导致方法区溢出。

- java.lang.StackOverflowError ------> 不会抛OOM error，但也是比较常见的Java内存溢出。JAVA虚拟机栈溢出，一般是由于程序中存在死循环或者深度递归调用造成的，栈大小设置太小也会出现此种溢出。可以通过虚拟机参数-Xss来设置栈的大小。

### OOM 解决思路

##### 针对内存溢出
更改申请对象的数量，比如分页

##### 针对内存泄露
及时释放资源，比如连接池

### OOM分析--heapdump

要dump堆的内存镜像，可以采用如下两种方式：

- 设置JVM参数-XX:+HeapDumpOnOutOfMemoryError，设定当发生OOM时自动dump出堆信息。不过该方法需要JDK5以上版本。
- 使用JDK自带的jmap命令。 其中pid可以通过jps获取。
```shell
jmap -dump:format=b,file=heap.bin pid
```



dump堆内存信息后，需要对dump出的文件进行分析，从而找到OOM的原因。常用的工具有：
- mat: eclipse memory analyzer, 基于eclipse RCP的内存分析工具。详细信息参见：[http://www.eclipse.org/mat/，推荐使用。](http://www.eclipse.org/mat/%EF%BC%8C%E6%8E%A8%E8%8D%90%E4%BD%BF%E7%94%A8%E3%80%82)
- jhat：JDK自带的java heap analyze tool，可以将堆中的对象以html的形式显示出来，包括对象的数量，大小等等，并支持对象查询语言OQL，分析相关的应用后，可以通过http://localhost:7000来访问分析结果。不推荐使用，因为在实际的排查过程中，一般是先在生产环境 dump出文件来，然后拉到自己的开发机器上分析，所以，不如采用高级的分析工具比如前面的mat来的高效。




 - [ ] 复习 OOM 场景 (@2023-12-16)
