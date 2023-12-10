![[Pasted image 20231029180229.png|600]]

Java线程存在六个线程状态

- 新建 New ： 创建后未启动
- 运行 Runnable ： 对应的操作系统线程状态的 running 和 ready ， 可能正在执行， 可能正在等待操作系统分配执行时间
- 无限期等待 Waiting ： 在等待唤醒动作的发生
- 限期等待 Timed Waiting
- 阻塞 Blocked ： 和等待状态的区别就是阻塞状态在等待获取一个排他锁
- 结束 Terminated

**只有runnable到running时才会占用cpu时间片，其他都会出让cpu时间片。线程的资源有不少，但应该包含CPU资源和锁资源这两类。sleep(long mills)：让出CPU资源，但是不会释放锁资源。wait()：让出CPU资源和锁资源。**

锁是用来线程同步的，sleep(long mills)虽然让出了CPU，但是不会让出锁，其他线程可以利用CPU时间片了，但如果其他线程要获取sleep(long mills)拥有的锁才能执行，则会因为无法获取锁而不能执行，继续等待。但是那些没有和sleep(long mills)竞争锁的线程，一旦得到CPU时间片即可运行了。

- **stop() 和 interrupt() 方法的区别**
    
    stop() 方法会立即杀死线程，如果线程持有 ReentrantLock 锁，是不会调用 ReentrantLock 的 unlock() 去释放锁的，因此其他线程也就没有机会获得 ReentrantLock 锁，这是很危险的。类似的还有 suspend() 方法和 resume() 方法。
    
    interrupt() 方法仅仅是通知线程中断，被通知的线程有机会执行一些后续的操作，同时也可以无视这个通知。收到通知的方式有两种：一种是异常，另一种是主动检测。
    
    **当线程 A 处于 WAITING、TIMED_WAITING 状态时，其他线程调用 A.interrupt() 方法，会使线程 A 转到 RUNNABLE 状态，同时线程 A 的代码会触发 InterruptedException 异常。**
    
    **当线程 A 处于 RUNNABLE 状态时，并且阻塞在 Java.nio.channels.InterruptibleChannel 上时，如果其他线程调用 A.interrupt() ，A 会触发 ClosedByInterruptException 异常；如果阻塞在 Selector 上，会立即返回。**
    
    **如果线程处于 RUNNABLE 状态，并且没有阻塞在某个 I/O 操作上，中断就要依赖线程主动检测中断状态了，通过调用 isInterrupted() 方法。**
    
- 为什么局部变量是线程安全的？
    
    ![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993102797-982b7ce7-d280-452a-b586-45a960da679a.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993102797-982b7ce7-d280-452a-b586-45a960da679a.png)
    
    每个方法在调用栈里都有自己的独立空间，称为栈帧。
    
    每个栈帧里都有对应方法需要的参数和返回地址。当调用方法时，会创建新的栈帧，并压入调用栈；当方法返回时，对应的栈帧就会被自动弹出。也就是说栈帧和方法是同生共死的。方法的调用是利用栈结构解决的
    
    ![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993146186-e404a399-6783-4411-a05c-8b53f37bfd97.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663993146186-e404a399-6783-4411-a05c-8b53f37bfd97.png)
    
    局部变量放到了调用栈里
    
- wait yield sleep join
    
    sleep **，释放cpu资源，不释放锁资源，**如果线程进入sleep的话，释放cpu资源，如果外层包有Synchronize，那么此锁并没有释放掉。
    
    wait，**释放cpu资源，也释放锁资源，** 一般用于锁机制中 肯定是要释放掉锁的，因为notify并不会立即调起此线程，因此cpu是不会为其分配时间片的，也就是说wait 线程进入等待池，cpu不分时间片给它，锁释放掉。
    
    **(wait用于锁机制，sleep不是，这就是为啥sleep不释放锁，wait释放锁的原因，sleep是线程的方法，跟锁没半毛钱关系，wait，notify,notifyall 都是Object对象的方法，是一起使用的，用于锁机制)**
    
    yield：**让出CPU调度**，Thread类的方法，类似sleep。只是**不能由用户指定暂停多长时间 ，**并且yield()方法**只能让同优先级的线程**有执行的机会。 yield()只是使当前线程重新回到可执行状态，所以执行yield()的线程有可能在进入**到可执行状态后**马上又被执行。调用yield方法只是一个建议，告诉线程调度器我的工作已经做的差不多了，可以让别的相同优先级的线程使用CPU了，没有任何机制保证采纳。
    
    join：一种特殊的wait，当前运行线程调用另一个线程的join方法，当前线程进入阻塞状态直到另一个线程运行结束等待该线程终止。 注意该方法也需要捕捉异常。