
[GitHub - google/guava: Google core libraries for Java](https://github.com/google/guava)

### JVM缓存

首先是 JVM 缓存，也可以认为是堆缓存。

其实就是创建一些全局变量，如 `Map、List` 之类的容器用于存放数据。

这样的优势是使用简单但是也有以下问题：

- 只能显式的写入，清除数据。
- 不能按照一定的规则淘汰数据，如 `LRU，LFU，FIFO` 等。
- 清除数据时的回调通知。
- 其他一些定制功能等

### Ehcache、Guava Cache

出现了一些专门用作 JVM 缓存的开源工具出现了，如 Guava Cache。

它具有上文 JVM 缓存不具有的功能，如自动清除数据、多种清除算法、清除回调等。

但也正因为有了这些功能，**这样的缓存必然会多出许多东西需要额外维护，自然也就增加了系统的消耗**。

### Guava Cache

Guava Cache是Google开源的Java重用工具集库Guava里的一款缓存工具，它的设计灵感来源于ConcurrentHashMap**，使用多个segments方式的细粒度锁，在保证线程安全的同时，支持高并发场景需求，同时支持多种类型的缓存清理策略，包括基于容量的清理、基于时间的清理、基于引用的清理等**

Guava Cache有两种缓存加载的方式：CacheLoader 和 Callable，这两种方式都是按照"获取缓存-如果没有-则计算"[get-if-absent-compute]的规则加载的。不同的是，CacheLoader是在创建Cache的时候，实现了一个统一的根据key获取value的方法，而Callable更加灵活，允许你在get的时候指定一个callable来获取value

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()  
            .maximumSize(1000)
            .build(
                new CacheLoader<Key, Graph>() {
                    public Graph load(Key key) throws AnyException {
                        return createExpensiveGraph(key);
                    }
                });

...
try {  
    return graphs.get(key);
} catch (ExecutionException e) {
    throw new OtherException(e.getCause());
}
```

```java
Cache<String, String> cache = CacheBuilder.newBuilder()  
            .maximumSize(1000)
            .build();

// 获取某个key时，在Cache.get中单独为其指定load方法
String resultVal = cache.get("hello", new Callable<String>() {  
                        public String call() {
                            String strProValue="hello world!";
                            return strProValue;
                        }
                    });
```

使用cache.put(key, value)方法可以直接向缓存中插入值，这会直接覆盖掉该key之前缓存的值。使用Cache.asMap()视图提供的任何方法也能相应的修改缓存。但是，asMap视图的任何方法都不能保证缓存项被原子地加载到缓存中

因为我们的数据是缓存在java堆内存上的，存储容量受到堆内存大小限制。当我们缓存的数据量很大时，会影响到GC，所以缓存必须要有回收策略。Guava Cache提供三种回收方式：

**基于容量回收**

通过CacheBuilder.maximumSize(long)设置缓存项的最大数目，当达到最大数目后，继续添加缓存项，Guava Cache会根据LRU策略回收缓存项来保证不超过最大数目

另外，可以通过CacheBuilder.weigher(Weigher)设置不同缓存项的权重，Guava Cache根据权重来回收缓存项

```java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()  
        .maximumWeight(100000)
        .weigher(new Weigher<Key, Graph>() {
            public int weigh(Key k, Graph g) {
                return g.vertices().size();
            }
        })
        .build(
            new CacheLoader<Key, Graph>() {
                public Graph load(Key key) { // no checked exception
                    return createExpensiveGraph(key);
                }
            });
```

**定时回收**

CacheBuilder提供两种定时回收的方法：

- `expireAfterAccess(long, TimeUnit)`：缓存项在给定时间范围内没有读/写访问，那么下次访问时，会被回收，然后同步`load()`（一个线程去load，其他线程等待）。
- `expireAfterWrite(long, TimeUnit)`：缓存项在给定时间范围内没有写访问，那么下次访问时，会被回收，然后同步`load()`（一个线程去load，其他线程等待）

Guava Cache不会专门维护一个线程来回收这些过期的缓存项，只有在读/写访问时，才去判断该缓存项是否过期，如果过期，则会回收

**基于引用回收**

通过使用弱引用的键、或弱引用的值、或软引用的值，Guava Cache可以把缓存设置为允许垃圾回收

- `CacheBuilder.weakKeys()`：使用弱引用存储键。当键没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（），使用弱引用键的缓存用而不是equals比较键。
- `CacheBuilder.weakValues()`：使用弱引用存储值。当值没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（），使用弱引用值的缓存用而不是equals比较值。
- `CacheBuilder.softValues()`：使用软引用存储值。软引用只有在响应内存需要时，才按照全局最近最少使用的顺序回收。考虑到使用软引用的性能影响，我们通常建议使用更有性能预测性的缓存大小限定（见上文，基于容量回收）。使用软引用值的缓存同样用==而不是equals比较值。

CacheBuilder如果没有指明，默认是强引用的

```java
keyStrength = builder.getKeyStrength();  
...
Strength getKeyStrength() {  
    return MoreObjects.firstNonNull(keyStrength, Strength.STRONG);
}
```

显式清除

除了系统回收外，也可以主动清除：

- 个别清除：`Cache.invalidate(key)`
- 批量清除：`Cache.invalidateAll(keys)`
- 清除所有缓存项：`Cache.invalidateAll()`

**刷新**

`CacheBuilder.refreshAfterWrite(long, TimeUnit)`刷新和回收不太一样，刷新表示为key加载新值，这个过程可以是异步的（加载新值的操作默认是同步调`load()`，异步需要重写`CacheLoader.reload()`）。在刷新操作进行时，缓存仍然可以向其他线程返回旧值，而不像回收操作，读缓存的线程必须等待新值加载完成

```java
//有些键不需要刷新，并且我们希望刷新是异步完成的
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()  
    .maximumSize(1000)
    .refreshAfterWrite(1, TimeUnit.MINUTES)
    .build(
        new CacheLoader<Key, Graph>() {
            public Graph load(Key key) { // no checked exception
                return getGraphFromDatabase(key);
            }

        public ListenableFuture<Key, Graph> reload(final Key key, Graph prevGraph) {
            if (neverNeedsRefresh(key)) {
                return Futures.immediateFuture(prevGraph);
            }else{
                // asynchronous!
                ListenableFutureTask<Key, Graph> task=
                                ListenableFutureTask.create(new Callable<Key, Graph>() {
                        public Graph call() {
                        return getGraphFromDatabase(key);
                    }
                });
                executor.execute(task);
                return task;
            }
        }
    });
```

因为刷新动作和回收一样，都是在检索的时候才会触发，所以当你的缓存配置了`CacheBuilder.refreshAfterWrite(long, TimeUnit)`时，如果部分缓存项很久没有被访问，那么再次被访问时，可能会获得过期很久的数据，这显然是不行的。而单独配置`expireAfterWrite(long, TimeUnit)`也是有问题的，如果热点数据突然过期，因为同步`load()`必然会影响读效率

通常我们都是`CacheBuilder.refreshAfterWrite(long, TimeUnit)`和`expireAfterWrite(long, TimeUnit)` 同时配置，并且刷新的时间间隔要比过期的时间间隔短！这样当较长时间没有被访问的缓存项突然被访问时，会触发过期回收而不是刷新，后面会分析这一块的源码，而热点数据只会触发刷新操作不会触发回收操作

**移除监听器**

通过`CacheBuilder.removalListener(RemovalListener)`，你可以声明一个监听器，以便缓存项被移除时做一些额外操作。缓存项被移除时，`RemovalListener`会获取移除通知`RemovalNotification`，其中包含移除原因`RemovalCause`、键和值