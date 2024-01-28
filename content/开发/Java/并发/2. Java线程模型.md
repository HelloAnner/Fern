
[[操作系统并发线程映射模型|操作系统并发线程映射模型]]

### JDK8 线程模型

Java 当前默认的是 1：1 内核线程实现

所以基于线程池的逻辑，其线程的数量很少

![[e6d5c11c7d8e6d52e447d32c88f11c55_MD5.jpeg]]

Java也在尝试新的模型 ， 如一个特殊的有栈协程 - 纤程 Fiber

如上文 ， 协程需要自己管理自己的现场保护、调度 ， 操作系统的内核无法感知到 ， 所以这里的逻辑一般都是自己完成实现

Fiber 亦是如此：

- 执行过程 ： 维护现场、保护、恢复上下文
- 调度器：负责线程的调度

![[ff8e98bd0165156d68f7a134d7821a67_MD5.jpeg]]

### JDK21 - 虚拟线程（Virtual Threads）

老的调用线程的写法可以修改为:
```java
Thread.ofPlatform().name("thread-test").start(new SimpleThread());

private static class SimpleThread implements Runnable {  
    @Override  
    public void run() {  
        System.out.println("Hello from a thread!");  
    }  
}
```



#### 虚拟线程和协程 的区别

如上文， 协程是基于编程语言自己的，其上下文切换都是依赖语言自己实现的。

虚拟线程是基于 JVM 实现的 ， 所有的 JVM 语言都是可以使用的。

虚拟线程是基于线程的协程实现，调度交给JVM管理。

