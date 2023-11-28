### 前文

[[../../操作系统/操作系统并发线程映射模型|操作系统并发线程映射模型]]

[[../并发/Java线程模型|Java线程模型]]

---

## Java的异步模式

### 线程池+Future异步调用

为了解决串行调用的低性能问题，我们会考虑使用并行异步调用的方式，最简单的方式便是使用线程池 +Future 去并行调用。

![[attachments/46e23eae65b994e4d667090f5ae4d7f4_MD5.jpeg]]

```java
ExecutorService executorService = Executors.newCachedThreadPool();  
Future<CommodityInfo> future1 = executorService.submit(() -> {  
    System.out.println("future1 is running...");  
    Thread.sleep(3000);  
    System.out.println("future1 is done");  
    return new CommodityInfo(10);  
});  
  
Future<CommodityInfo> future2 = executorService.submit(() -> {  
    System.out.println("future2 is running");  
    Thread.sleep(5000);  
    System.out.println("future2 is done");  
    return new CommodityInfo(20);  
});  
  
// 等待前面的结果返回  
Future<CommodityInfo> future3 = executorService.submit(() -> {  
    CommodityInfo info1 = future1.get();  
    CommodityInfo info2 = future2.get();  
    return cal(info1, info2);  
});  
  
System.out.println(future3.get().res);
```

上述例子中 ， future1 和 future2 是并行的， future3 是阻塞的，即需要等待 future1 和 future2 计算完毕

这种方式虽然解决了大部分场景下的串行调用低性能问题，但是也存在着严重的弊端，由于存在 Future 的前后依赖关系，当使用场景存在大量的前后依赖时，会使得**线程资源和 CPU 大量浪费在阻塞等待上**，导致资源利用率低



### 线程池+CompletableFuture异步调用

CompletableFuture 是由 Java8 引入的，在 Java8 之前一般通过 Future 实现异步。Future 用于表示异步计算的结果，如果存在流程之间的依赖关系，那么只能通过阻塞或者轮询的方式获取结果，同时原生的 Future 不支持设置回调方法，Java8 之前若要设置回调可以使用 Guava 的 ListenableFuture，回调的引入又会导致回调地狱，代码基本不具备可读性。

而 CompletableFuture 是对 Future 的扩展，原生支持通过设置回调的方式处理计算结果，同时也支持组合编排操作，一定程度解决了回调地狱的问题。

但是 CompletableFuture  仅仅是对复杂编排上做了改进，对一些本质问题还是没有解决：

- **`线程资源浪费瓶颈始终在 IO 等待上`**，导致 CPU 资源利用率较低。目前大部分服务是 IO 密集型服务，一次请求的处理耗时大部分都消耗在等待下游 RPC，数据库查询的 IO 等待中，此时线程仍然只能阻塞等待结果返回，导致 CPU 的利用率很低。
    
- **线程数量存在限制**， **为了增加并发度，我们会给线程池配置更大的线程数**，但是线程的数量是有限制的，Java 的线程模型是 1:1 映射平台线程的，导致 Java 线程创建的成本很高，不能无限增加。同时随着 CPU 调度线程数的增加，会导致更严重的资源争用，宝贵的 CPU 资源被损耗在上下文切换上。

## Virtual Thread

**`操作系统线程（OS Thread）`**：由操作系统管理，是操作系统调度的基本单位。
**`平台线程（Platform Thread）`**：Java.Lang.Thread 类的每个实例，都是一个平台线程，是 Java 对操作系统线程的包装，与操作系统是 1:1 映射。
**`虚拟线程（Virtual Thread）`**：一种轻量级，由 JVM 管理的线程。对应的实例java.lang.VirtualThread 这个类。
**载体线程（Carrier Thread）**：指真正负责执行虚拟线程中任务的平台线程。一个虚拟线程装载到一个平台线程之后，那么这个平台线程就被称为虚拟线程的载体线程。

**虚拟线程(Virtual Thread)它不与特定的操作系统线程相绑定**。它在平台线程上运行 Java 代码，但在代码的整个生命周期内不独占平台线程。这意味着许多虚拟线程可以在同一个平台线程上运行他们的 Java 代码，共享同一个平台线程。同时虚拟线程的成本很低，虚拟线程的数量可以比平台线程的数量大得多

![[attachments/3181b5c2a5d14b3dcebc854b6db30381_MD5.jpeg]]

### VT 的创建
```java
// 直接创建并启动  
Thread.startVirtualThread(() -> {  
    System.out.println("hello virtual thread");  
});  
  
// 选择线程类型，和是否默认启动  
Thread notStartVT = Thread.ofVirtual().unstarted(() -> {  
    System.out.println("hello virtual thread");  
});  
notStartVT.start();  
  
// 通过虚拟线程的 ThreadFactory 创建虚拟线程  
ThreadFactory tf = Thread.ofVirtual().factory();  
Thread vt = tf.newThread(() -> {  
  
});  
vt.start();  
  
// Executors.newVirtualThreadPerTaskExecutor()  
ExecutorService executorService = Executors.newVirtualThreadPerTaskExecutor();  
executorService.submit(() -> {  
    return true;  
});
```

### VT 实现原理

虚拟线程是由 Java 虚拟机调度，而不是操作系统。虚拟线程占用空间小，同时使用轻量级的任务队列来调度虚拟线程，避免了线程间基于内核的上下文切换开销，因此可以极大量地创建和使用。

**简单来看，虚拟线程实现如下：**`virtual thread =continuation+scheduler+runnable`

虚拟线程会把任务（java.lang.Runnable实例）包装到一个 **`Continuation`** 实例中:
- 当任务需要阻塞挂起的时候，会调用 Continuation 的 **`yield`** 操作进行阻塞，虚拟线程会从平台线程卸载。    - 阻塞会自动调用，如 IO、Lock、Thread、concurrent 模块的阻塞。
- 当任务解除阻塞继续执行的时候，调用 **`Continuation.run`** 会从阻塞点继续执行。

**`Scheduler`** 也就是执行器，由它将任务提交到具体的载体线程池中执行。
- 它是 java.util.concurrent.Executor 的子类。
- 虚拟线程框架提供了一个默认的 FIFO 的 ForkJoinPool 用于执行虚拟线程任务。默认的pool的大小为CPU的核心数，不超过 256.

**`Runnable`** 则是真正的任务包装器，由 Scheduler 负责提交到载体线程池中执行

---


JVM 把虚拟线程分配给平台线程的操作称为 **`mount（挂载）`**，取消分配平台线程的操作称为 **`unmount（卸载`**`）`
- **`mount 操作`**：虚拟线程挂载到平台线程，虚拟线程中包装的 Continuation 堆栈帧数据会被拷贝到平台线程的线程栈，这是一个从堆复制到栈的过程。即线程内的资源转移到了堆存储。
- **`unmount 操作`**：虚拟线程从平台线程卸载，此时虚拟线程的任务还没有执行完成，所以虚拟线程中包装的 Continuation 栈数据帧会会留在堆内存中。

![[attachments/9a8d7f211692490a8b12dca3cd600d50_MD5.jpeg]]

上图中体现了多个 VT ，随着时间线中，会从载体线程依次让出。

---



### 虚拟线程内存占用评估

**单个平台线程的资源占用**：  
- 根据 JVM 规范，预留 1 MB 线程栈空间。
- 平台线程实例，会占据 2000+ byte 数据。
    

**单个虚拟线程的资源占用**：

- Continuation 栈会占用数百 byte 到数百 KB 内存空间，是作为堆栈块对象存储在 Java 堆中。
- 虚拟线程实例会占据 200 - 240 byte 数据

```java
public class VTMem {
    private static final int COUNT = 4000;

    /**
     * -XX:NativeMemoryTracking=detail
     *
     * @param args args
     */
    public static void main(String[] args) throws Exception {
        for (int i = 0; i < COUNT; i++) {
            new Thread(() -> {
                try {
                    Thread.sleep(Long.MAX_VALUE);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
        Thread.sleep(Long.MAX_VALUE);
    }
}

```

```
Total: reserved=18287015KB, committed=8916055KB
       malloc: 45919KB #112311
       mmap:   reserved=18241096KB, committed=8870136KB

-                 Java Heap (reserved=8388608KB, committed=528384KB)
                            (mmap: reserved=8388608KB, committed=528384KB)

-                     Class (reserved=1049690KB, committed=1242KB)
                            (classes #638)
                            (  instance classes #532, array classes #106)
                            (malloc=1114KB #9300)
                            (mmap: reserved=1048576KB, committed=128KB)
                            (  Metadata:   )
                            (    reserved=65536KB, committed=256KB)
                            (    used=156KB)
                            (    waste=100KB =39.24%)
                            (  Class space:)
                            (    reserved=1048576KB, committed=128KB)
                            (    used=8KB)
                            (    waste=120KB =93.80%)

-                    Thread (reserved=8289912KB, committed=8289912KB)
                            (thread #4018)
                            (stack: reserved=8277080KB, committed=8277080KB)
                            (malloc=8125KB #24362)
                            (arena=4707KB #8034)

-                      Code (reserved=247744KB, committed=7632KB)
                            (malloc=48KB #1032)
                            (mmap: reserved=247696KB, committed=7584KB)

-                        GC (reserved=216585KB, committed=63081KB)
                            (malloc=19433KB #4603)
                            (mmap: reserved=197152KB, committed=43648KB)

-                 GCCardSet (reserved=29KB, committed=29KB)
                            (malloc=29KB #390)

-                  Compiler (reserved=167KB, committed=167KB)
                            (malloc=4KB #27)
                            (arena=164KB #4)

-                  Internal (reserved=7232KB, committed=7232KB)
                            (malloc=7200KB #29752)
                            (mmap: reserved=32KB, committed=32KB)

-                    Symbol (reserved=1131KB, committed=1131KB)
                            (malloc=771KB #51)
                            (arena=360KB #1)
```

Thread **栈空间**占用在 8289 MB ，堆已经使用了 528MB

那么换成 VT 呢

```
 Java Heap (reserved=8388608KB, committed=528384KB)
                            (mmap: reserved=8388608KB, committed=528384KB)

-                     Class (reserved=1048670KB, committed=222KB)
                            (classes #733)
                            (  instance classes #616, array classes #117)
                            (malloc=94KB #1605)
                            (mmap: reserved=1048576KB, committed=128KB)
                            (  Metadata:   )
                            (    reserved=65536KB, committed=576KB)
                            (    used=459KB)
                            (    waste=117KB =20.39%)
                            (  Class space:)
                            (    reserved=1048576KB, committed=128KB)
                            (    used=40KB)
                            (    waste=88KB =68.64%)

-                    Thread (reserved=59826KB, committed=59826KB)
                            (thread #29)
                            (stack: reserved=59740KB, committed=59740KB)
                            (malloc=54KB #179)
                            (arena=32KB #56)
```

堆已经使用了 528MB ， Thread 堆空间 占用 59 MB
虚拟线程在大量创建的前提下也不会去占用过多的内存，且虚拟线程的堆栈是作为堆栈块对象存储在 Java 的堆中的，可以被 GC 回收，又降低了虚拟线程的占用


### VT局限和使用建议

VT 仅对非CPU密集场景使用，如果是CPU密集，会导致载体线程严重阻塞。

#### 无法让出载体线程的场景 PinnedThreads

虚拟线程存在 `native` 方法或者`外部方法` (**Foreign Function & Memory** **API****，jep 424 )** 调用不能进行 `yield` 操作，此时载体线程会被阻塞。
    
`当运行在 synchronized` 修饰的代码块或者方法时，不能进行 `yield` 操作，此时载体线程会被阻塞，推荐使用 ReentrantLock。


#### ThreadLocal 数量爆炸问题

**ThreadLocal 相关问题**，目前虚拟线程仍然是支持 ThreadLocal 的，但是由于虚拟线程的数量非常多，会导致 Threadlocal 中存的线程变量非常多，需要频繁 GC 去清理，对性能会有影响，官方建议尽量少使用 ThreadLocal，同时不要在虚拟线程的 ThreadLocal 中放大对象，目前官方是想通过 ScopedLocal 去替换掉 ThreadLocal，但是在 21 版本还没有正式发布，**这个可能是大规模使用虚拟线程的一大难题。**

#### 使用建议

**无需池化虚拟线程** 虚拟线程占用的资源很少，因此可以大量地创建而无须考虑池化，它不需要跟平台线程池一样，平台线程的创建成本比较昂贵，所以通常选择去池化，去做共享，**但是池化操作本身会引入额外开销**，对于虚拟线程池化反而是得不偿失，使用虚拟线程我们抛弃池化的思维，用时创建，用完就扔。

### VT 适合的场景

- 大量的 IO 阻塞等待任务，例如下游 RPC 调用，DB 查询等。
    
- 大批量的处理时间较短的计算任务。
    
- Thread-per-request (一请求一线程)风格的应用程序，例如主流的 Tomcat 线程模型或者基于类似线程模型实现的 SpringMVC 框架 ，这些应用只需要小小的改动就可以带来巨大的吞吐提升。

### 压测结论
![[attachments/c6559539041abbe2a268b41ee4d663ce_MD5.jpeg]]


## 参考

https://mp.weixin.qq.com/s/vdLXhZdWyxc6K-D3Aj03LA

https://www.bilibili.com/video/BV1Ae411Z7vN/?vd_source=5523c0780f6f8cbee236d42fbf45240f