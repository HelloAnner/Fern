
ThreadLocal对象可以提供线程局部变量，每个线程Thread拥有一份自己的**副本变量**，多个线程互不干扰

![https://cdn.nlark.com/yuque/0/2022/png/22813151/1663997824598-263a7c32-1397-4f51-8cdd-f35fcdc2e9ef.png](https://cdn.nlark.com/yuque/0/2022/png/22813151/1663997824598-263a7c32-1397-4f51-8cdd-f35fcdc2e9ef.png)

Thread类有一个类型为ThreadLocal.ThreadLocalMap的实例变量threadLocals，也就是说每个线程有一个自己的ThreadLocalMap

ThreadLocalMap有自己的独立实现，可以简单地将它的key视作ThreadLocal，value为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个**弱引用**）

每个线程在往ThreadLocal里放值的时候，都会往自己的ThreadLocalMap里存，读也是以ThreadLocal作为引用，在自己的map里找对应的key，从而实现了**线程隔离**

ThreadLocalMap有点类似HashMap的结构，只是HashMap是由**数组+链表**实现的，而ThreadLocalMap中并没有**链表**结构 , 直接就是一个 Entry 数组 ， 直接hash key 获取的index

### 源码解析

ThreadLocalMap 里面存储的 Entry 的 key 是 ThreadLocal自己的弱引用， Value 就是 我们设置的值
```java
static class ThreadLocalMap {  
   static class Entry extends WeakReference<ThreadLocal<?>> {  
        /** The value associated with this ThreadLocal. */  
        Object value;  
  
        Entry(ThreadLocal<?> k, Object v) {  
            super(k);  
            value = v;  
        }  
    }
```

set一个元素的过程 ， 获取index就是hash key 获取到的 ， 存储在 entry 里面
```java
private void set(ThreadLocal<?> key, Object value) {  
  
    // We don't use a fast path as with get() because it is at  
    // least as common to use set() to create new entries as    // it is to replace existing ones, in which case, a fast    // path would fail more often than not.  
    Entry[] tab = table;  
    int len = tab.length;  
    int i = key.threadLocalHashCode & (len-1);  
  
    for (Entry e = tab[i];  
         e != null;  
         e = tab[i = nextIndex(i, len)]) {  
        ThreadLocal<?> k = e.get();  
  
        if (k == key) {  
            e.value = value;  
            return;  
        }  
  
        if (k == null) {  
            replaceStaleEntry(key, value, i);  
            return;  
        }  
    }  
  
    tab[i] = new Entry(key, value);  
    int sz = ++size;  
    if (!cleanSomeSlots(i, sz) && sz >= threshold)  
        rehash();  
}
```

get 为按照自己的引用获取 Entry 对象
```java
public T get() {  
    Thread t = Thread.currentThread();  
    ThreadLocalMap map = getMap(t);  
    if (map != null) {  
        ThreadLocalMap.Entry e = map.getEntry(this);  
        if (e != null) {  
            @SuppressWarnings("unchecked")  
            T result = (T)e.value;  
            return result;  
        }  
    }  
    return setInitialValue();  
}
```


### 内存泄露问题 - value 导致的

**突然我们ThreadLocal是null了，也就是要被垃圾回收器回收了，但是此时我们的ThreadLocalMap（thread 的内部属性）生命周期和Thread的一样，它不会回收，这时候就出现了一个现象。那就是ThreadLocalMap的key没了，但是value还在，所以 value 就永远无法回收,这就造成了内存泄漏**

解决办法：使用完`ThreadLocal`后，执行`remove`操作，避免出现内存溢出情况。

如果不`remove` 当前线程对应的`VALUE` ,就会一直存在这个值。

使用了线程池，可以达到“线程复用”的效果。但是归还线程之前记得清除`ThreadLocalMap`，要不然再取出该线程的时候，`ThreadLocal`变量还会存在。这就不仅仅是内存泄露的问题了，整个业务逻辑都可能会出错。

看看 remove 的过程
```java
private void remove(ThreadLocal<?> key) {  
    Entry[] tab = table;  
    int len = tab.length;  
    int i = key.threadLocalHashCode & (len-1);  
    for (Entry e = tab[i];  
         e != null;  
         e = tab[i = nextIndex(i, len)]) {  
        if (e.get() == key) {  
            e.clear();  
            expungeStaleEntry(i);  
            return;  
        }  
    }  
}
```

### 为什么 Key 使用 弱引用 - key导致的

如果使用强引用，当`ThreadLocal` 对象的引用（强引用）被回收了，`ThreadLocalMap`本身依然还持有`ThreadLocal`的强引用，如果没有手动删除这个key ,则`ThreadLocal`不会被回收，所以只要当前线程不消亡，`ThreadLocalMap`引用的那些对象就不会被回收， 可以认为这导致`Entry`内存泄漏。

[[JAVA引用#弱引用]]

### 使用 DEMO
```java
    private List<String> messages = new ArrayList<>();

    public final static ThreadLocal<ThreadLocalDemo> holder = ThreadLocal.withInitial(ThreadLocalDemo::new);

    public static void add(String message){
        holder.get().messages.add(message);
    }

    public static List<String> clear(){
        List<String> ms = holder.get().messages;
        ms.clear();
        return ms;
    }
```


```java
public static void main(String[] args) {  
    ThreadLocal<String> local = new ThreadLocal<>();  
    IntStream.range(0, 10).forEach(i -> new Thread(() -> {  
        local.set(Thread.currentThread().getName() + ":" + i);  
        System.out.println("Thread " + Thread.currentThread().getName() + " : " + local.get());  
    }).start());  
}
```

- [ ] 复习 ThreadLocal (@2023-12-16)