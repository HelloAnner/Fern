

并发编程领域抽象成三个核心问题：分工，同步和互斥。

**分工**：把一个功能分给不同的线程共同实现。核心是编程模式。

**同步**：当某个条件不满足时，线程需要等待，当某个条件满足时，线程需要被唤醒执行。管程是解决并发问题的万能钥匙。

**互斥**：保证线程安全，同一时刻，只允许一个线程访问共享变量。核心技术是锁。

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663986664903-72e64cb0-c08d-4f94-a063-60977f278b8c.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663986664903-72e64cb0-c08d-4f94-a063-60977f278b8c.png)

## 并发编程Bug的源头 - 可见性、原子性和有序性问题

为了平衡CPU、内存和I/O设备三者的速度差异，计算机体系机构、操作系统、编译程序都做了优化，主要为：

1. CPU增加了缓存，以均衡与内存的速度差异； - 可见性问题
2. 操作系统增加了进程、线程，已分时复用CPU，进而均衡CPU与I/O设备的速度差异；- 原子性问题
3. 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用； - 有序性问题

### 缓存导致的可见性问题

一个线程对共享变量的修改，另外一个线程能够立刻看到，我们称为**可见性**。

单核时代，不同线程操作同一个CPU里面的缓存，不存在可见性问题。

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987264067-09886bf4-aab7-4691-9bb1-fcca044fc440.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987264067-09886bf4-aab7-4691-9bb1-fcca044fc440.png)

多核时代，每颗CPU都有自己的缓存，不同的线程操作不同CPU的缓存时，对彼此之间就不具备可见性了

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987279237-c741838a-2dd6-4b93-88f3-70edbacb9e17.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987279237-c741838a-2dd6-4b93-88f3-70edbacb9e17.png)

### 线程切换带来的原子性问题

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987348748-fc24e38b-281e-420c-9a3b-a61e1279d65c.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987348748-fc24e38b-281e-420c-9a3b-a61e1279d65c.png)

任务切换的时机大多数是在时间片结束的时候。我们现在使用的高级程序语言一条语句往往对应多条CPU指令，例如语句：count += 1，至少需要三条CPU指令。

- 指令1：首先，需要把变量 count 从内存加载到CPU的寄存器；
- 指令2：之后，在寄存器中执行 +1 操作；
- 指令3：最后，将结果写入内存（缓存机制导致可能写入的是CPU缓存而不是内存）。

操作系统做任务切换，可以发生在任何一条CPU指令执行完。这就可能得到意想不到的结果

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987410123-98bfb71d-20c4-4543-86e5-db239c0537d1.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987410123-98bfb71d-20c4-4543-86e5-db239c0537d1.png)

### 编译优化带来的有序性问题

**有序性**指的是程序按照代码的先后顺序执行。编译器为了优化性能，有时候会改变程序中语句的先后顺序。

双重检查创建单例对象：

```java
public class Singleton {
    static Singleton instance;
    static Singleton getInstance(){
        if (instance == null) {
            synchronized(Singleton.class) {
                if (instance == null)
                    instance = new Singleton();
            }
        }
        return instance;
    }
}
```

instance = new Singleton() 语句经过编译优化重排序后的CPU执行过程可能是：

1. 分配一块内存M；
2. 将M的地址赋值给instance实例；
3. 最后再内存M上初始化Singleton对象。

当线程A执行完指令2时，线程切换B线程调用getInstance方法，获得未初始化的Singleton对象，如果此时访问对象成员变量，那么就可能触发空指针异常。

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987614376-ab2eb3a6-aa2d-4c12-88fd-f2d999741f66.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663987614376-ab2eb3a6-aa2d-4c12-88fd-f2d999741f66.png)

## Java内存模型：看Java如何解决可见性和有序性问题

此处为语雀内容卡片，点击链接查看：[https://www.yuque.com/anner-dhcrb/pafw88/1661904379980](https://www.yuque.com/anner-dhcrb/pafw88/1661904379980)

## 互斥锁 ：解决原子性问题

**同一时刻只有一个线程执行**，我们称之为**互斥**。如果能够保证对共享变量的修改是互斥的，那么，无论是单核CPU还是多核CPU，就都能保证原子性了

### synchronized

锁是一种通用的技术方案，Java语言提供的synchronized关键字，就是锁的一种实现。synchronized 关键字既可以用来修饰方法，也可以用来修饰代码块

```java
class X {
  // 修饰非静态方法
  synchronized void foo() {
	// 临界区
  }
  // 修饰静态方法
  synchronized static void bar() {
	// 临界区
  }
  // 修饰代码块
  Object obj = new Object()；
  void baz() {
	synchronized(obj) {
  		// 临界区
	}
  }
}
```

- 当synchronized 修饰静态方法的时候，锁定的是当前类的class 对象， 在上面的例子中就是 class X；
- 当synchronized 修饰非静态方法的时候，锁定的是当前实例对象this

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663989080790-62b07b85-a027-4bc6-8069-003311d83181.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663989080790-62b07b85-a027-4bc6-8069-003311d83181.png)

### 锁和受保护资源的关系

受保护资源和锁之间的关联关系是 N:1 的关系。一个锁可以锁多个资源，但是多个锁不能锁一个资源

```java
class SafeCalc {
  static long value = 0L;
  synchronized long get() {
	return value;
  }
  synchronized static void addOne() {
	value += 1;
  }
}
```

上面的例子中，get() 方法加的是 this 锁， addOne() 方法加的是 SafeCalc.class 锁，它们都保护 value 资源。但是由于是不同的锁，因此两个临界区就没有互斥关系，就导致了addOne() 方法对value的操作对临界区 get() 没有可见性保证，从而导致并发问题

### 如何用一把锁保护多个资源？

一个锁来保护多个资源的关键是：锁能覆盖所有受保护资源

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663991690869-5a2e62a7-1312-4598-87b0-cefb54730bb7.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663991690869-5a2e62a7-1312-4598-87b0-cefb54730bb7.png)

当要保护多个资源时，关键要分析多个资源之间的关系。如果资源之间没有关系，那么就每个资源都有一把锁。如果资源之间有关联关系，就要选择一个粒度更大的锁，这个锁应该能够覆盖所有相关的资源。

原子性的本质是多个资源间有一致性的要求，操作的中间状态对外不可见。解决原子性问题，是要保证中间状态对外不可见

## 死锁问题

当下面这四个条件都发生时才会出现死锁：

1. 互斥，共享资源 X 和 Y 只能被一个线程占用；
2. 占有且等待，线程 T1 已经取得共享资源 X，在等待共享资源 Y 的时候，不释放共享资源 X；
3. 不可抢占，其他线程不能强行抢占线程 T1 占有的资源；
4. 循环等待，线程 T1 等待线程 T2占有的资源，线程T2 等待线程 T1 占有的资源，就是循环等待。

只要破坏其中一个条件，就可以避免死锁的发生

### 破坏占用且等待条件

要破坏这个条件，可以一次性申请所有资源。

### 破坏不可抢占条件

synchronized如果申请不到资源就会进入阻塞状态，同时线程已经占有的资源也不会释放。但是在 java.util.concurrent包下面提供的Lock是可以轻松解决这个问题。

### 破坏循环等待条件

破坏这个条件，需要对资源进行排序，然后按序申请资源

[死锁](https://www.notion.so/8ce223f3cf6f45d79fa953316d195fc7?pvs=21)

## MESA 模型

管程模型有：Hasen 模型、Hoare 模型和 MESA 模型。Java 管程实现用的是 MESA 模型

并发领域的两大核心问题：互斥，即同一时刻只允许一个线程访问共享资源；另一个是同步，即线程之间如何通信、协作。

管程解决互斥问题的思路，就是将共享变量及其对共享变量的操作统一封装起来。同一时刻的操作只允许一个线程进入管程

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663992281615-ec801d66-338e-4012-91d5-e445e6a505db.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663992281615-ec801d66-338e-4012-91d5-e445e6a505db.png)

当一个线程 T1 从入口等待队列中出列，从入口处进入管程中。只有满足某一条件时才能执行操作，这个条件就是条件变量。条件不满足时，就会进入这个条件变量的等待队列等待（通过调用 wait() 方法实现），此时管程是允许其他线程进入的。其他线程 T2 进入了管程，执行了某些操作，导致 T1 线程的条件满足，T2 要通知 T1 条件已经满足（通过调用 notify()/notifyAll() 方法实现）。T1 得到通知后，会从条件变量等待队列中出来，然后进入入口等待队列，等待从新进入管程

### wait() 的正确姿势

MESA 管程特有的一个编程范式，就是需要在一个 while 循环里面调用 wait()

```java
while(条件不满足) {
  wait();
}
```

管程要求同一时刻只允许一个线程执行，当线程 T2 的操作使线程 T1 等待的条件满足时，究竟是哪个线程执行呢。不同的模型有不同的执行策略：

1. **Hasen 模型里面，要求 notify() 放在代码的最后，这样 T2 通知完 T1 后，T2 就结束了，然后 T1 再执行，这样就能保证同一时刻只有一个线程执行。**
2. **Hoare 模型里面，T2 通知完 T1 后，T2 阻塞，T1 马上执行；等 T1 执行完，再唤醒 T2 ，也能保证同一时刻只有一个线程执行。但是 T2 多了一次阻塞唤醒操作。**
3. **MESA 模型里面，T2 通知完 T1 后，T2 还是会继续执行，T1 并不立即执行，仅仅是从条件变量的等待队列进入入口等待队列里面。这样做不好的地方是，当 T1 再次执行时，可能曾经满足的条件，现在已经不满足了。所以需要以循环的方式检验条件变量。**

### notify() 何时可以使用

一般情况下尽量使用 notifyAll() 方法，只有满足下面三个条件时，才使用 notify():

1. 所有等待线程有相同的等待条件；
2. 所有等待线程被唤醒后，执行相同的操作；
3. 只需要唤醒一个线程。

## 任务取消

### 标志位

```java
class MyThread implements Runnable
{
    public volatile boolean isComplete = false;
    @Override
    public void run() {
       while(!isComplete)
       {
       }
    }
}
```

这种方式有个很大的弊端：如果while循环中存在阻塞代码，更不巧的是这段阻塞代码不知道什么时候结束甚至可能永远也不结束，那么任务永远也不可能去检查isComple标志位，导致线程永远不可能结束。

不建议使用。

### 中断状态 - interrupt

```java
public class Thread{
    public void interrupt(){}    //中断目标线程，实际上就是设置中断状态为true
    public boolean isInterrupted(){} //返回目标线程的中断状态
    public static boolean interrupted(){} //清除当前线程的中断状态，并返回它之前的值
}
```

Java的中断是一种非抢占式的中断，Thread.interrupt()发出中断信号中断目标线程，实际上也只是设置中断标志位为true

对中断正确的理解是：调用方并不真正中断正在运行的线程，而只是发出中断请求，由被中断线程在下一个合适的时刻中断自己。中断是取消线程最合理的方式。

### 阻塞库响应中断

阻塞库方法在入口处就会去检查中断状态，在阻塞的过程中也会即可响应中断，**一旦发现中断状态被设置，则：清除中断状态、抛出InterruptedException**。JVM不能保证阻塞方法检测到中断的速度，但实际情况中响应中断的速度还是非常快的。

会检查中断状态并响应的阻塞库包括：

1、Thread.sleep()

2、Object.wait()

3、BlockingQueue.put()/take()

4、Thread.join()

不可响应中断的阻塞包括：

1、 同步IO，如InputStream.read()和OutputStream.write()

2、 异步IO，如Selector.select()

3、 等待获取某个内置锁

### 处理中断

阻塞函数抛出InterruptedException的同时已经清除了中断状态，对于应用程序来说有两种策略可以用于处理这个InterruptedException：

策略1：继续抛出此InterruptedException

策略2：退出前通过Thread.currentThread.interrupt()恢复中断

策略3：编写中断处理代码，保证中断得到完全处理

```java
class MyThread implements Runnable
{
    @Override
    public void run() {
       try{
           while(!Thread.currentThread().isInterrupted()){
                Thread.sleep(20000l);
           }
       }catch(InterruptedExceptione){
        }
    }
}
```

### interrupt、interrupted和isInterrupted的区别

interrupt() 向当前调用者线程发出中断信号

isinterrupted() 查看当前中断信号是true还是false

interrupted() 是静态方法，查看当前中断信号是true还是false并且清除中断信号，顾名思义interrupted为已经处理中断信号。

```java
public class ThreadTest {
    public static void main(String[] args) throws Exception {
        Thread t = new Thread(new Worker());
        t.start();
        Thread.sleep(2000);
        t.interrupt();
        System.out.println("Main thread stopped.");
    }

    public static class Worker implements Runnable {
        public void run() {
            System.out.println("Worker started.");
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                System.out.println("Worker IsInterrupted: " + Thread.currentThread().isInterrupted());
                //Thread.currentThread().isInterrupted()   false
            }
            System.out.println("Worker stopped.");
        }
    }
}
```

Worker明明已经被中断，但是isInterrupted()方法竟然返回了false，为什么呢？因为**当抛出InterruptedException 异常的时候，中断状态已被清除**。

interrupt方法是用于中断线程的，调用该方法的线程的状态将被置为"中断"状态。注意：线程中断仅仅是设置线程的中断状态位，不会停止线程。需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出InterruptedException的方法，比如这里的sleep，以及Object.wait等方法）就是在监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。

如果我们需要isInterrupted这个方法返回true，那应该捕获异常后，再次进行中断：

```java
public class ThreadTest {
    public static void main(String[] args) throws Exception {
        Thread t2 = new Thread(new Worker2());
        t2.start();
        Thread.sleep(200);
        t2.interrupt();
        System.out.println("Main thread stopped.");
    }
    public static class Worker2 implements Runnable {
        public void run() {
            System.out.println("Worker started.");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread curr = Thread.currentThread();
                curr.interrupt();  //再次中断
                System.out.println("Worker IsInterrupted: " + curr.isInterrupted());   //true
                System.out.println("Worker IsInterrupted: " + curr.isInterrupted());   //true
                System.out.println("Static Call: " + Thread.interrupted());//clear status  //获取上次状态，并重置  true
                System.out.println("---------After Interrupt Status Cleared----------");
                System.out.println("Static Call: " + Thread.interrupted());   //false
                System.out.println("Worker IsInterrupted: " + curr.isInterrupted());  //false
                System.out.println("Worker IsInterrupted: " + curr.isInterrupted());  //false
            }
            System.out.println("Worker stopped.");
        }
    }
}
```

为什么要在抛出InterruptedException的时候清除掉中断状态呢？这里有个似乎比较合理的解释：一个中断应该只被处理一次（你catch了这个InterruptedException，说明你能处理这个异常，你不希望上层调用者看到这个中断）。

### Future.cancel

cancel()方法取消线程的方法是调用interrupt()方法尝试中断线程。