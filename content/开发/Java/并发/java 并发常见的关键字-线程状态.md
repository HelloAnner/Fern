![[attachments/Pasted image 20231029180158.png]]

### wait/notify/notifyAll方法的使用

wait()方法的作用是将当前运行的线程挂起（即让其进入阻塞状态），直到notify或notifyAll方法来唤醒线程.

wait方法的使用必须在同步的范围内，否则就会抛出IllegalMonitorStateException异常，wait方法的作用就是阻塞当前线程等待notify/notifyAll方法的唤醒，或等待超时后自动唤醒

wait方式是通过对象的monitor对象来实现的，所以只要在同一对象上去调用notify/notifyAll方法，就可以唤醒对应对象monitor上等待的线程了。notify和notifyAll的区别在于前者只能唤醒monitor上的一个线程，对其他线程没有影响，而notifyAll则唤醒所有的线程

```java
package com.anner.base;

public class WaitDemo {
    public static void main(String[] args) throws InterruptedException {
        final WaitDemo waitDemo = new WaitDemo();
        for (int i = 0; i < 5; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    waitDemo.testWait();
                }
            }).start();
        }

        synchronized (waitDemo) {
            waitDemo.notify();
        }

        Thread.sleep(3000);

        System.out.println("-----------------------------");

        synchronized (waitDemo) {
            waitDemo.notifyAll();
        }

    }

    public synchronized void testWait() {
        System.out.println(Thread.currentThread().getName() + " start------");
        try {
            wait(0);
        } catch (InterruptedException exception) {
            exception.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " end------");
    }
}
```

调用notify方法时只有线程Thread-0被唤醒，但是调用notifyAll时，所有的线程都被唤醒了。

最后，有两点点需要注意：

（1）调用wait方法后，线程是会释放对monitor对象的所有权的。

（2）一个通过wait方法阻塞的线程，必须同时满足以下两个条件才能被真正执行：

- 线程需要被唤醒（超时唤醒或调用notify/notifyll）
- 线程唤醒后需要竞争到锁（monitor）

任意一个object以及其子类对象都有两个队列

同步队列：所有尝试获取该对象Monitor失败的线程，都加入同步队列排队获取锁

等待队列：已经拿到锁的线程在等待其他资源时，主动释放锁，置入该对象等待队列中，等待被唤醒，当调用notify（）会在等待队列中任意唤醒一个线程，将其置入同步队列的尾部，排队获取锁

### sleep/yield/join方法解析

sleep方法的作用是让当前线程暂停指定的时间（毫秒），sleep方法是最简单的方法，在上述的例子中也用到过，比较容易理解。唯一需要注意的是其与wait方法的区别。最简单的区别是，wait方法依赖于同步，而sleep方法可以直接调用。

而更深层次的区别在于sleep方法只是暂时让出CPU的执行权，并不释放锁。而wait方法则需要释放锁

```java
package com.anner.base;

public class SleepDemo {
    public static void main(String[] args) {
        final SleepDemo demo = new SleepDemo();

        for (int i = 0; i < 3; i++) {
            new Thread(new Runnable() {
                @Override
                public void run() {
                    demo.sleepMethod();
                }
            }).start();
        }

        try {
            Thread.sleep(10000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("----------------------");

        for (int j = 0; j < 3; j++) {

            new Thread(new Runnable() {
                @Override
                public void run() {
                    demo.waitMethod();
                }
            }).start();
        }

    }

    public synchronized void sleepMethod() {
        System.out.println("sleep start -------");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("sleep end ------");
    }

    public synchronized void waitMethod() {
        System.out.println("wait start -------");
        try {
            wait(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("wait end ------");
    }
}
```

通过sleep方法实现的暂停，程序是顺序进入同步块的，只有当上一个线程执行完成的时候，下一个线程才能进入同步方法，sleep暂停期间一直持有monitor对象锁，其他线程是不能进入的。而wait方法则不同，当调用wait方法后，当前线程会释放持有的monitor对象锁，因此，其他线程还可以进入到同步方法，线程被唤醒后，需要竞争锁，获取到锁之后再继续执行

yield方法的作用是暂停当前线程，不会释放锁，以便其他线程有机会执行，不过不能指定暂停的时间，并且也不能保证当前线程马上停止。yield方法只是将Running状态转变为Runnable状态

1. 不要混淆cpu和锁，线程交出cpu并不等于一定要交出锁，这个yield只是让出cpu，让其他线程可以使用cpu，但是如果其他线程wait在该线程hold住的锁上的话，那些线程是不会被执行的，其实就是即使运行也还是继续wait。
2. 所有就绪的线程都可以竞争，高优先级的只是概率大些，但未必一定会先执行。而且刚刚用yield让出cpu的线程也有可能被再次调度到。

```java
package com.anner.base;

public class YieldDemo {
    public static void main(String[] args) {
        YieldTest test = new YieldTest();
        Thread t1 = new Thread(test, "t1");
        Thread t2 = new Thread(test, "t2");

        t1.start();
        t2.start();
    }
}

class YieldTest implements Runnable {
    @Override
    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " : " + i);
            Thread.yield();
        }
    }
}
```

通过yield方法来实现两个线程的交替执行。不过请注意：这种交替并不一定能得到保证，源码中也对这个问题进行说明：

```java
/**
     * A hint to the scheduler that the current thread is willing to yield
     * its current use of a processor. The scheduler is free to ignore this
     * hint.
     *
     * <p> Yield is a heuristic attempt to improve relative progression
     * between threads that would otherwise over-utilise a CPU. Its use
     * should be combined with detailed profiling and benchmarking to
     * ensure that it actually has the desired effect.
     *
     * <p> It is rarely appropriate to use this method. It may be useful
     * for debugging or testing purposes, where it may help to reproduce
     * bugs due to race conditions. It may also be useful when designing
     * concurrency control constructs such as the ones in the
     * {@link java.util.concurrent.locks} package.
*/
```

这段话主要说明了三个问题：

- 调度器可能会忽略该方法。
- 使用的时候要仔细分析和测试，确保能达到预期的效果。
- 很少有场景要用到该方法，主要使用的地方是调试和测试。

---

join方法的作用是父线程等待子线程执行完成后再执行，换句话说就是将异步执行的线程合并为同步的线程

join方法就是通过wait方法来将线程的阻塞，如果join的线程还在执行，则将当前线程阻塞起来，直到join的线程执行完成，当前线程才能执行。不过有一点需要注意，这里的join只调用了wait方法，却没有对应的notify方法，原因是**Thread的start方法中做了相应的处理，所以当join的线程执行完成以后，会自动唤醒主线程继续往下执行**

```java
package com.anner.base;

public class JoinDemo {

    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) {
            Thread test = new Thread(new JoinTest());
            test.start();
            try {
                test.join(); // 调用join方法
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("Finished~~~");
    }

}

class JoinTest implements Runnable {
    @Override
    public void run() {
        try {
            System.out.println(Thread.currentThread().getName() + " start-----");
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " end------");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```