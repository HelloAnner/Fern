


### Executors框架

**为什么使用 Executor 框架？**

1. 每次执行任务创建线程 new Thread() 比较消耗性能，创建一个线程是比较耗时、耗资源的。
2. 调用 new Thread() 创建的线程缺乏管理，被称为野线程，而且可以无限制的创建，线程之间的相互竞争会导致过多占用系统资源而导致系统瘫痪，还有线程之间的频繁交替也会消耗很多系统资源。
3. 接使用 new Thread() 启动的线程不利于扩展，比如定时执行、定期执行、定时定期执行、线程中断等都不便实现。


### 线程池分类

newFixedThreadPool(int nThreads) : 创建一个固定长度的线程池

- 每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程规模将不再变化。
- 当线程发生未预期的错误而结束时，线程池会补充一个新的线程。
- 等待队列类型：`LinkedBlockingQueue`
- 这种线程池使用一个无界的阻塞队列来保存等待执行的任务。当线程池中的线程数达到**最大数量**时，新的任务会被放入队列中等待执行。

任务堆积。如果提交的任务过多，超过线程池处理能力，会导致任务堆积在等待队列中，可能导致内存溢出或系统负载过高。

队列部分 ： [[../集合/Java 队列|Java 队列]] 

---


newCachedThreadPool() 方法，创建一个可缓存的线程池

- 如果线程池的规模超过了处理需求，将自动回收空闲线程。
- 当需求增加时，则可以自动添加新线程。线程池的规模不存在任何限制。
- 等待队列类型：`SynchronousQueue`
- 这种线程池使用一个**没有容量的同步队列**，每个提交的任务都会尝试立即交给一个线程执行。如果没有空闲线程，将会创建一个新的线程。当线程空闲时间超过指定的**keep-alive时间**时，线程会被终止并从线程池中移除。


无限制的线程创建。当任务提交速度过快，超过线程池处理能力时，会不断创建新线程，可能导致线程数过多，消耗过多的系统资源。

---

newSingleThreadExecutor() 方法，创建一个单线程的线程池

- 它创建单个工作线程来执行任务，如果这个线程异常结束，会创建一个新的来替代它。
- 它的特点是，能确保依照任务在队列中的顺序来串行执行。
- 等待队列类型：`LinkedBlockingQueue`
- 这种线程池使用一个无界的阻塞队列来保存等待执行的任务。只有一个工作线程来处理任务，保证任务按照顺序串行执行。


存在的问题：
任务阻塞。由于只有一个工作线程，如果一个任务执行时间过长或发生死锁，会导致后续任务无法执行，造成整个系统的阻塞


---

newScheduledThreadPool(int corePoolSize) 方法，创建了一个固定长度的线程池，而且以延迟或定时的方式来执行任务，类似 Timer

- 等待队列类型：`DelayedWorkQueue`
- 这种线程池使用一个延迟队列来保存等待执行的任务，任务会在指定的延迟时间后执行。该线程池适用于需要按照固定时间间隔执行任务的场景。


任务调度不准确。如果任务执行时间超过预期，或者任务执行时间不稳定，可能会导致任务之间的调度不准确，影响系统的稳定性和性能

任务堆积。如果定时任务提交速度过快，超过线程池处理能力，会导致任务堆积，可能导致内存溢出或系统负载过高



### 执行顺序

![[attachments/9f272f87efdc4b4907a8f5c7b50c5078_MD5.jpeg]]

刚创建时，里面没有线程调用 execute() 方法，添加任务时：

- 如果正在运行的线程数量小于核心参数 corePoolSize ，继续创建线程运行这个任务
- 否则，如果正在运行的线程数量大于或等于 corePoolSize ，将任务加入到阻塞队列中。
- 否则，如果队列已满，同时正在运行的线程数量小于核心参数 maximumPoolSize ，继续创建线程运行这个任务。
- 否则，如果队列已满，同时正在运行的线程数量大于或等于 maximumPoolSize ，根据设置的拒绝策略处理。


完成一个任务，继续取下一个任务处理。
没有任务继续处理，线程被中断或者线程池被关闭时，线程退出执行，如果线程池被关闭，线程结束。
否则，判断线程池正在运行的线程数量是否大于核心线程数，如果是，线程结束，否则线程阻塞。因此线程池任务全部执行完成后，继续留存的线程池大小为 corePoolSize 。

### 拒绝策略

1. AbortPolicy（默认策略）：  
    当线程池无法接受新任务时，默认的拒绝策略是 `AbortPolicy`。它会抛出 `RejectedExecutionException` 异常，拒绝执行新任务。
    
2. CallerRunsPolicy：  
    ``CallerRunsPolicy` 是一种简单的拒绝策略，它将新任务返回给调用者来执行。如果线程池无法接受新任务，任务会在提交者的线程中执行。这种策略可以减缓任务提交速度，但也可能导致调用者执行任务的线程阻塞。
    
3. DiscardPolicy：  
    ``DiscardPolicy` 是一种简单的拒绝策略，它会默默地丢弃无法处理的新任务，不给予任何提示或记录。当线程池无法接受新任务时，新任务将被静默丢弃。
    
4. DiscardOldestPolicy：  
    ``DiscardOldestPolicy` 拒绝策略会丢弃等待时间最长的任务，并尝试执行新任务。它可以用来优先执行新提交的任务，而不是等待时间较长的任务。


### 自定义线程池参数

```java
// ThreadPoolExecutor.java

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

- corePoolSize 参数，核心线程数大小，当线程数 < corePoolSize ，会创建线程执行任务。
    
- maximumPoolSize 参数，最大线程数， 当线程数 >= corePoolSize 的时候，会把任务放入 workQueue 队列中。
    
- keepAliveTime 参数，**保持存活时间，当线程数大于 corePoolSize 的空闲线程能保持的最大时间。**
    
- unit 参数，时间单位。
    
- workQueue 参数，保存任务的阻塞队列

- handler 参数，超过阻塞队列的大小时，使用的拒绝策略。
    
- threadFactory 参数，创建线程的工厂。


### 关闭线程池

**shutdown()**

调用 shutdown() 方法之后线程池并不是立刻就被关闭，因为这时线程池中可能还有很多任务正在被执行，或是任务队列中有大量正在等待被执行的任务，调用 shutdown() 方法后线程池会在执行完正在执行的任务和队列中等待的任务后才彻底关闭。

但这并不代表 shutdown() 操作是没有任何效果的，调用 shutdown() 方法后如果还有新的任务被提交，线程池则会根据拒绝策略直接拒绝后续新提交的任务,抛出RejectedExecutionException异常

**isShutdown()**

第二个方法叫作 isShutdown()，它可以返回 true 或者 false 来判断线程池是否已经开始了关闭工作，也就是是否执行了 shutdown 或者 shutdownNow 方法

如果调用 isShutdown() 方法的返回的结果为 true 并不代表线程池此时已经彻底关闭了，这仅仅代表线程池开始了关闭的流程，也就是说，此时可能线程池中依然有线程在执行任务，队列里也可能有等待被执行的任务

**isTerminated()**

检测线程池是否真正“终结”了，这不仅代表线程池已关闭，同时代表线程池中的所有任务都已经都执行完毕了

**awaitTermination()**

调用 awaitTermination 方法后当前线程会尝试等待一段指定的时间，如果在等待时间内，线程池已关闭并且内部的任务都执行完毕了，也就是说线程池真正“终结”了，那么方法就返回 true，否则超时返回 fasle

直到发生以下三种情况之一：

1. 等待期间（包括进入等待状态之前）线程池已关闭并且所有已提交的任务（包括正在执行的和队列中等待的）都执行完毕，相当于线程池已经“终结”了，方法便会返回 true；
2. 等待超时时间到后，第一种线程池“终结”的情况始终未发生，方法返回 false；
3. 等待期间线程被中断，方法会抛出 InterruptedException 异常。

**shutdownNow()**

在执行 shutdownNow 方法之后，首先会给所有线程池中的线程发送 interrupt 中断信号，尝试中断这些任务的执行，然后会将任务队列中正在等待的所有任务转移到一个 List 中并返回，我们可以根据返回的任务 List 来进行一些补救的操作，例如记录在案并在后期重试

```java
public List<Runnable> shutdownNow() {
    List<Runnable> tasks;
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        checkShutdownAccess();
        advanceRunState(STOP);
        interruptWorkers();
        tasks = drainQueue();
    } finally {
        mainLock.unlock();
    }
    tryTerminate();
    return tasks;
}

```

这里需要注意的是，由于 Java 中不推荐强行停止线程的机制的限制，即便我们调用了 shutdownNow 方法，如果被中断的线程对于中断信号不理不睬，那么依然有可能导致任务不会停止


一个比较经典的优雅关闭
```java
public void shutdownGracefully() {
   shutdownThreadPool(streamThreadPool, "main-pool");
}

/**
 * 优雅关闭线程池
 * @param threadPool
 * @param alias
 */
private void shutdownThreadPool(ExecutorService threadPool, String alias) {
   log.info("Start to shutdown the thead pool: {}", alias);

   threadPool.shutdown(); // 使新任务无法提交.
   try {
      // 等待未完成任务结束
      if (!threadPool.awaitTermination(60, TimeUnit.SECONDS)) {
         threadPool.shutdownNow(); // 取消当前执行的任务
         log.warn("Interrupt the worker, which may cause some task inconsistent. Please check the biz logs.");

         // 等待任务取消的响应
         if (!threadPool.awaitTermination(60, TimeUnit.SECONDS))
            log.error("Thread pool can't be shutdown even with interrupting worker threads, which may cause some task inconsistent. Please check the biz logs.");
      }
   } catch (InterruptedException ie) {
      // 重新取消当前线程进行中断
      threadPool.shutdownNow();
      log.error("The current server thread is interrupted when it is trying to stop the worker threads. This may leave an inconcistent state. Please check the biz logs.");

      // 保留中断状态
      Thread.currentThread().interrupt();
   }

   log.info("Finally shutdown the thead pool: {}", alias);
}
```

### 参考

[三分钟弄懂线程池执行过程 - 掘金](https://juejin.cn/post/6866054685044768782)
[有什么知名的开源apm(Application Performance Management)工具吗？ - 知乎](https://www.zhihu.com/question/27994350)
[Fetching Title#n1ef](https://www.cnblogs.com/frankyou/p/10135212.html)
[线程组ThreadGroup - duanxz - 博客园](https://www.cnblogs.com/duanxz/p/3368787.html)
