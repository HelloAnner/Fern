
- 修饰一个普通方法时，作用域是当前调用对象，即只要还没出方法的作用域，其他试图获取该对象的锁线程都将被阻塞
- 类的静态方法属于类，而不属于类的某个特定实例，所以对类的静态方法修饰直接作用于类本身，相当于synchronized(ClassA.class)，即直接锁定整个类

```java
public class Singleton {
    private volatile static Singleton instance; //声明成 volatile
    private Singleton (){}
    public static Singleton getSingleton() {
        if (instance == null) {                         
            synchronized (Singleton.class) {
                if (instance == null) {       
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

sync中的Singleton.class是什么意思？为什么不是this？

sync(this)以及非static的sync方法，只能防止多个线程同时执行同一个对象的同步代码段。我们回归原来的问题，sync到底锁的是对象还是代码段？答案是：sync锁住的是括号里面的对象，而不是代码段。对于非static的sync方法，锁住的就是对象本身，也就是this

### synchronized底层语义原理

在JVM中，对象在内存中的布局分为三块区域：对象头、实例数据和对齐填充

- 实例变量：存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐。
- 填充数据：由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了字节对齐

对于顶部，则是Java头对象，它实现synchronized的锁对象的基础

jvm中采用2个字来存储对象头(如果对象是数组则会分配3个字，多出来的1个字记录的是数组长度)，其主要结构是由Mark Word 和 Class Metadata Address 组成

![[Pasted image 20231102182311.png]]

Mark Word在默认情况下存储着对象的HashCode、分代年龄、锁标记位等以下是32位JVM的Mark Word默认存储结构

![[Pasted image 20231102182323.png]]

---

### Monitor 模式

一般的 monitor 实现模式是编程语言在语法上提供语法糖，而如何实现 monitor 机制，则属于编译器的工作，Java 就是这么干的。

monitor 的重要特点是，同一个时刻，只有一个 进程/线程 能进入 monitor 中定义的临界区，这使得 monitor 能够达到互斥的效果。但仅仅有互斥的作用是不够的，无法进入 monitor 临界区的 进程/线程，它们应该被阻塞，并且在必要的时候会被唤醒。

显然，monitor 作为一个同步工具，也应该提供这样的管理 进程/线程 状态的机制。想想我们为什么觉得 semaphore 和 mutex 在编程上容易出错，因为我们需要去亲自操作变量以及对 进程/线程 进行阻塞和唤醒。monitor 这个机制之所以被称为“更高级的原语”，那么它就不可避免地需要对外屏蔽掉这些机制，并且在内部实现这些机制，使得使用 monitor 的人看到的是一个简洁易用的接口。

**monitor 机制需要几个元素来配合，分别是：**

1. **临界区**
2. **monitor object 及锁**
3. **条件变量以及定义在 monitor 对象上的 wait，signal 操作。**

使用 monitor 机制的目的主要是为了互斥进入临界区，为了做到能够阻塞无法进入临界区的 进程/线程，还需要一个 monitor object 来协助，这个 monitor object 内部会有相应的数据结构，例如列表，来保存被阻塞的线程；同时由于 monitor 机制本质上是基于 mutex 这种基本原语的，所以 monitor object 还必须维护一个基于 mutex 的锁。此外，为了在适当的时候能够阻塞和唤醒 进程/线程，还需要引入一个条件变量，这个条件变量用来决定什么时候是“适当的时候”，这个条件可以来自程序代码的逻辑，也可以是在 monitor object 的内部，总而言之，程序员对条件变量的定义有很大的自主性。不过，由于 monitor object 内部采用了数据结构来保存被阻塞的队列，因此它也必须对外提供两个 API 来让线程进入阻塞状态以及之后被唤醒，分别是 wait 和 notify。

### Java 语言对 monitor 的支持

被 synchronized 关键字修饰的方法、代码块，就是 monitor 机制的临界区

synchronized 关键字在使用的时候，往往需要指定一个对象与之关联，例如 synchronized(this)，或者 synchronized(ANOTHER_LOCK)，synchronized 如果修饰的是实例方法，那么其关联的对象实际上是 this，如果修饰的是类方法，那么其关联的对象是 this.class。总之，synchronzied 需要关联一个对象，而这个对象就是 monitor object

monitor 的机制中，monitor object 充当着维护 mutex以及定义 wait/signal API 来管理线程的阻塞和唤醒的角色

Java 对象存储在内存中，分别分为三个部分，即对象头、实例数据和对齐填充，而在其对象头中，保存了锁标识；同时，java.lang.Object 类定义了 wait()，notify()，notifyAll() 方法，这些方法的具体实现，依赖于一个叫 ObjectMonitor 模式的实现，这是 JVM 内部基于 C++ 实现的一套机制，基本原理如下所示：

![[Pasted image 20231029175949.png|400]]

当一个线程需要获取 Object 的锁时，会被放入 EntrySet 中进行等待，如果该线程获取到了锁，成为当前锁的 owner。如果根据程序逻辑，一个已经获得了锁的线程缺少某些外部条件，而无法继续进行下去（例如生产者发现队列已满或者消费者发现队列为空），那么该线程可以通过调用 wait 方法将锁释放，进入 wait set 中阻塞进行等待，其它线程在这个时候有机会获得锁，去干其它的事情，从而使得之前不成立的外部条件成立，这样先前被阻塞的线程就可以重新进入 EntrySet 去竞争锁。这个外部条件在 monitor 机制中称为条件变量。

### 参考

[https://segmentfault.com/a/1190000016417017](https://segmentfault.com/a/1190000016417017)