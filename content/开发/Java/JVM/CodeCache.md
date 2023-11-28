
JVM 生成的 native code 存放的内存空间称之为 Code Cache；

JIT 编译、JNI 等都会编译代码到 native code (方法区)，其中 JIT 生成的 native code 占用了 Code Cache 的绝大部分空间。

> 关于 JIT ： Java程序最初是仅仅通过解释器解释执行的，即对字节码逐条解释执行，这种方式的执行速度相对会比较慢，尤其当某个方法或代码块运行的特别频繁时，这种方式的执行效率就显得很低。于是后来在JVM中引入了JIT编译器（即时编译器），当虚拟机发现某个方法或代码块运行特别频繁时，达到某个阈值，就会把这些代码认定为Hot Spot Code（热点代码），为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各层次的优化，完成这项任务的正是JIT编译器。

在JVM中存在三种代码编译执行模式,分别是解释执行、纯编译执行与分层编译执行，这几种方式有各自的特点。

1、解释执行该模式下表示全部代码均是解释执行，不做任何JIT编译，这种模式会降低运行速度，通常低10倍或者更多，一般不会采用。

2、纯编译执行这个参数可以使JVM运行在纯编译的模式，所有的方法在第一次调用的时候就会编成机器代码，但是现实的话，设置了这个参数之后系统启动负载的确没有上升，但是随之而来的问题，启动的时间会大幅度增加。

3、分层编译执行HotSpot 内置两种编译器，分别是Client启动时的c1编译器和Server启动时的c2编译器：**在默认情况下，JDK8是会开启分层编译执行，这里会涉及到两个线程C1 Compiler Threads、C2 Compiler Threads**

这里CPU占用高的可能原因：

1、有大量的热点方法需要编译

2、code cache满了，导致大量的编译指令异常

### 概念

JVM里有一块比较特殊的内存叫做CodeCache，Java代码在执行时一旦被编译器编译为机器码，下一次执行的时候就会直接执行编译后的代码，也就是说，**编译后的代码被缓存了起来。缓存编译后的机器码的内存区域就是CodeCache**。

这块内存主要存储JVM动态生成的代码，动态生成的代码有挺多，最主要的是JIT编译后的代码，Java之所以执行快，是因为随着程序的运行，大部分热点代码会被编译成优化过的机器码来执行，除了JIT编译的代码之外，动态生成的代码，本地方法代码（JNI）也会存在CodeCache中。所以如果这块内存不够就会影响程序的执行效率。

如果CodeCache区域被占满，编译器被停用，字节码将不再会被编译成机器码，应用程序将继续运行，但运行速度会降低一个数量级，严重影响应用服务的运行。

**如果 CodeCache 这部分内存空间耗尽的话，jvm 就无法继续生成新的 native code，就会导致性能大幅下降，现象表现为各线程执行都阻塞在一些 CPU 密集的方法，例如计算 hash、运行正则表达式、实例化新对象等等。**

### 问题暴露

正常情况下 java 的默认 CodeCache 配置是能够满足需求的，此处不会产生性能瓶颈，但是经过调查发现，**在 arm64 的 cpu 平台上，jvm 默认指定的 CodeCache 值会过小**。

以 oracle 的 jdk1.8 为例，正常 x86_64 平台上默认的 CodeCache 大小为 250m 左右：

而 arm64 平台上默认为 50m 左右（影响因素可能包括不限于 cpu 类型）：

这样就导致 arm64 服务器上运行报表系统更容易出现 CodeCache 耗尽导致性能下降的问题，同时由于 jit 的动态编译、清理机制，CodeCache 耗尽需要运行一段时间后才会出现，所以表现出来的现象为**正常运行好几天甚至个把月后出现整体性的访问性能下降，但是重启后恢复**。

截至目前我们已经发现至少 3 个客户存在过这样的现象，并且客户使用的都是 arm64 的服务器。

同时 arm64 平台的【代码密度】（即编译出的汇编指令大小）更小，因此编译出来的 native code 需要占用更大的空间（网上的资料说明 arm64 的汇编指令大小比 x86_64 大 30% 左右），进一步加剧了 CodeCache 不够用的问题。

### 问题定位 & 优化方案

Co**mpileBroker::compiler_thread_loop()**这个线程的方法是JVM中JIT提供的编译线程所执行的

关键日志

```prolog
Java HotSpot (TM) 64-Bit Server VM warning: CodeCache is full
Compiler has been disabled
•Hava HotSpot (TM) 64-Bit Server VM warnina:
Try increasing the code cache size using - XX:ReservedCodeCacheSize=
• CodeCache: size=245760Kb used=233448Kb max used=235088Kb free=12311Kb bounds [0x00007f f05000000, 0×00007f0c4000000, 0×00007f f0c4000000] total blobs=54817 methods=52674 adapters=2046 compilation: enabled
```

当已用大小达到或即将 90% 的最大值时，一般认为可能导致性能下降，优化方案就是指定一个更大的 CodeCache 空间，就像 jvm 内存一样。

本地工程设置-XX:+PrintCompilation -XX:ReservedCodeCacheSize=10m 参数，开启打印编译日志，同时设置代码缓存大小为10M

可以发现，在运行过程中，有很多跳过编译的指令，其原因是code cache满了

![https://cdn.nlark.com/yuque/0/2023/png/22813151/1673234523762-eb762530-a0c1-41bc-b840-04f674d00969.png](https://cdn.nlark.com/yuque/0/2023/png/22813151/1673234523762-eb762530-a0c1-41bc-b840-04f674d00969.png)

添加 jvm 参数：**-XX:ReservedCodeCacheSize=<期望的大小>**，一般来说指定到 250m 就够了（如：-XX:ReservedCodeCacheSize=250m）。

### codeCache 大小控制选项

|选项|默认值|描述|
|---|---|---|
|InitialCodeCacheSize|2555904|默认的CodeCache区域大小，单位为字节|
|ReservedCodeCacheSize|251658240|CodeCache区域的最大值，单位为字节|
|CodeCacheExpansionSize|65536|CodeCache每次扩展大小，单位为字节|

### codeCache 刷新选项

|选项|默认值|描述|
|---|---|---|
|ExitOnFullCodeCache|false|当CodeCache区域满了的时候是否退出JVM|
|UseCodeCacheFlushing|true|是否在关闭JIT编译前清除CodeCache|
|MinCodeCacheFlushingInterval|30|刷新CodeCache的最小时间间隔 ，单位为秒|
|CodeCacheMinimumFreeSpace|512000|当CodeCache区域的剩余空间小于参数指定的值时停止JIT编译。剩余的空间不会再用来存放方法的本地代码, 可以存放本地方法适配器代码。|

### 编译策略选项

|选项|默认值|描述|
|---|---|---|
|CompileThreshold|10000|指定方法在在被JIT编译前被调用的次数|
|OnStackReplacePercentage|140|该值为用于计算是否触发OSR（OnStackReplace）编译的阈值|

### JIT 编译限制选项

|选项|默认值|描述|
|---|---|---|
|MaxInlineLevel|9|在进行方法内联前，方法的最多嵌套调用次数|
|MaxInlineSize|35|被内联方法的字节码最大值|
|MinInliningThreshold|9|方法被内联的最小调用次数|
|InlineSynchronizedMethods|true|是否对同步方法进行内联|

### 诊断选项

|选项|默认值|描述|
|---|---|---|
|PrintFlagsFinal|false|是否打印所有的JVM参数|
|PrintCodeCache|false|是否在JVM退出前打印CodeCache的使用情况|
|PrintCodeCacheOnCompilation|false|是否在每个方法被JIT编译后打印CodeCache区域的使用情况|

```bash
jinfo -flag ReservedCodeCacheSize 62012 # -XX:ReservedCodeCacheSize=251658240
```

### 参考文档

[https://www.jianshu.com/p/b064274536ed](https://www.jianshu.com/p/b064274536ed)

[https://cloud.tencent.com/developer/article/1879628](https://cloud.tencent.com/developer/article/1879628)

[https://leokongwq.github.io/2016/10/11/jvm-codecache.html](https://leokongwq.github.io/2016/10/11/jvm-codecache.html)