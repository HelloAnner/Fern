
Java的内存分配和内存回收，都不需要程序员负责，一个对象是否可以被回收，主要看是否有引用指向此对象，说的专业点，叫可达性分析。Java设计这四种引用的主要目的有两个：

1. 可以让程序员通过代码的方式来决定某个对象的生命周期。
2. 有利于垃圾回收。

### 强引用

强引用是最普遍的一种引用，绝大部分我们写的代码，都是强引用：

```java
Object o = new Object();
```

这种就是强引用了，只要某个对象有强引用与之关联，这个对象永远不会被回收，即使内存不足，JVM宁愿抛出OOM，也不会去回收。那么什么时候才可以被回收呢？当强引用和对象之间的关联被中断了，就可以被回收了。我们可以手动把关联给中断了，方法也特别简单：

```java
o = null;
```

我们可以手动调用GC，看看如果强引用和对象之间的关联被中断了，资源会不会被回收，为了更方便、更清楚的观察到回收的情况，我们需要新写一个类，然后重写finalize方法，下面我们来进行这个实验：

```java
public class Student {
  @Override
  protected void finalize() throws Throwable {
      System.out.println("Student 被回收了");
  }
}
public static void main(String[] args) {
  Student student = new Student();
  student = null;
  System.gc();
}

// Student 被回收了
```

### 软引用

```java
SoftReference studentSoftReference = new SoftReference(new Student());
```

软引用有什么特点呢：当内存不足，会触发JVM的GC，如果GC后，内存还是不足，就会把软引用的包裹的对象给干掉，也就是只有在内存不足，JVM才会回收该对象。

还是一样的，必须做实验，才能加深印象：

```java
SoftReference<byte[]> softReference = new SoftReference<byte[]>(new byte[1024*1024*10]);
System.out.println(softReference.get());
System.gc();
System.out.println(softReference.get());
 
byte[] bytes = new byte[1024 * 1024 * 10];
System.out.println(softReference.get());
```

定义了一个软引用对象，里面包裹了byte[]，byte[]占用了10M，然后又创建了10Mbyte[]。

运行程序，需要带上一个参数：-Xmx20M

```java
[B@11d7fff
[B@11d7fff
null
```

可以很清楚的看到手动完成GC后，软引用对象包裹的byte[]还活的好好的，但是当我们创建了一个10M的byte[]后，最大堆内存不够了，所以把软引用对象包裹的byte[]给干掉了，如果不干掉，就会抛出OOM。软引用到底有什么用呢？比较适合用作缓存，当内存足够，可以正常的拿到缓存，当内存不够，就会先干掉缓存，不至于马上抛出OOM。

### 弱引用

弱引用的使用和软引用类似，只是关键字变成了WeakReference：

```java
WeakReference<byte[]> weakReference = new WeakReference<byte[]>(new byte[1024*1024*10]);
System.out.println(weakReference.get());
```

弱引用的特点是不管内存是否足够，只要发生GC，都会被回收：

```java
WeakReference<byte[]> weakReference = new WeakReference<byte[]>(new byte[1]);
System.out.println(weakReference.get());
System.gc();
System.out.println(weakReference.get());

// [B@11d7fff
// null
```

**弱引用指向的对象，一旦没有被强引用或软引用指向，就会被垃圾回收器回收，即使内存不紧张也会回收。**

<mark style="background: #BBFABBA6;">软引用或者弱引用，如果被强引用关联了，那么内存不足，或者 GC 的时候都不会被删除</mark>

弱引用在很多地方都有用到，比如 [[../并发/8. ThreadLocal]] 、WeakHashMap

### 虚引用

虚引用又被称为幻影引用，我们来看看它的使用：

```java
ReferenceQueue queue = new ReferenceQueue();
PhantomReference<byte[]> reference = new PhantomReference<byte[]>(new byte[1], queue);
System.out.println(reference.get());

public T get() {
    return null;
}
```

无法通过虚引用来获取对一个对象的真实引用

那虚引用存在的意义是什么呢？这就要回到我们上面的代码了

创建虚引用对象，我们除了把包裹的对象传了进去，还传了一个ReferenceQueue，从名字就可以看出它是一个队列。

虚引用的特点之二就是 虚引用必须与ReferenceQueue一起使用，当GC准备回收一个对象，如果发现它还有虚引用，就会在回收之前，把这个虚引用加入到与之关联的ReferenceQueue中。

```java
ReferenceQueue queue = new ReferenceQueue();
List<byte[]> bytes = new ArrayList<>();
PhantomReference<Student> reference = new PhantomReference<Student>(new Student(),queue);
 
new Thread(() -> {
    for (int i = 0; i < 100;i++ ) {
        bytes.add(new byte[1024 * 1024]);
    }
}).start();
 
new Thread(() -> {
    while (true) {
        Reference poll = queue.poll();
        if (poll != null) {
            System.out.println("虚引用被回收了：" + poll);
        }
    }
}).start();
 
Scanner scanner = new Scanner(System.in);
scanner.hasNext();
}
```

```java
Student 被回收了
虚引用被回收了：java.lang.ref.PhantomReference@1ade6f1
```

我们简单的分析下代码：

第一个线程往集合里面塞数据，随着数据越来越多，肯定会发生GC。第二个线程死循环，从queue里面拿数据，如果拿出来的数据不是null，就打印出来。

从运行结果可以看到：当发生GC，虚引用就会被回收，并且会把回收的通知放到ReferenceQueue中。

虚引用有什么用呢？在NIO中，就运用了虚引用管理堆外内存。


- [ ] 复习 JAVA 引用 (@2023-12-14)