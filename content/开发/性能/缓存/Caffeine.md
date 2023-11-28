
## 底层结构

[Design zh CN](https://github.com/ben-manes/caffeine/wiki/Design-zh-CN)

## 加载策略

### 手动加载

```java
// 手动同步缓存
        Cache<String, String> cache = Caffeine.newBuilder()
                .expireAfterAccess(5, TimeUnit.MINUTES)
                .maximumSize(1000)
                .build();
        cache.put("name", "caffeine");
        // 单独提供同步的缓存计算机制，如果计算失败了，还是返回null
        assertEquals("age-value", cache.get("age", k -> k + "-value"));
        assertEquals("caffeine", cache.getIfPresent("name"));
        assertNull(cache.getIfPresent("not exist"));
```

### 自动加载

```java
// 自动加载
        LoadingCache<Integer, Integer> loadingCache = Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .build(key -> key + 1); // 提供一个key 到 value 的计算机制，方便自动加载 , 这里仅仅覆写了 load 方法
        // 如果 get 的缓存不存在，会同步按照策略计算好之后返回
        assertEquals(new Integer(2), loadingCache.get(1));
        // getAll 会全部一一计算
        Map<Integer, Integer> allMap = loadingCache.getAll(Arrays.asList(1, 2, 3, 4));
        assertEquals(new Integer(2), allMap.get(1));
```

如果 getAll 的时候 ， 批量查询更有效率 ， 可以继续覆写 loadAll

```java
loadingCache = Caffeine.newBuilder()
                .expireAfterWrite(1, TimeUnit.SECONDS)
                .build(new CacheLoader<Integer, Integer>() {
                    @Override
                    public @Nullable Integer load(@NonNull Integer key) throws Exception { // load + 1
                        return key + 1;
                    }

                    @Override
                    public @NonNull Map<@NonNull Integer, @NonNull Integer> loadAll(@NonNull Iterable<? extends @NonNull Integer> keys) throws Exception { // loadAll + 2
                        final Map<Integer, Integer> loadAllRes = new HashMap<>();
                        keys.forEach(k -> loadAllRes.put(k, k + 10));
                        return loadAllRes;
                    }
                });
        allMap = loadingCache.getAll(Arrays.asList(1, 2, 3, 4));
        assertEquals(new Integer(11), allMap.get(1));
```

### 手动异步加载

```java
// 手动异步加载
        AsyncCache<Integer, Integer> asyncCache = Caffeine.newBuilder().buildAsync();
        CompletableFuture<Integer> completableFuture = asyncCache.get(1, k -> k + 1); // 返回的是 CompletableFuture
        assertEquals(new Integer(2), completableFuture.get());
        // 阻塞等待缓存更新
        assertEquals(new Integer(2), asyncCache.synchronous().getIfPresent(1));
```

AsyncCache 是在 executor 中生成 CompletableFuture

提供支持阻塞直到更新完毕

默认使用的线程池是 ForkJoinPool.commonPool

### 自动异步加载

```java
// 自动异步加载
        AsyncLoadingCache<Integer, Integer> asyncLoadingCache = Caffeine.newBuilder().buildAsync(k -> k + 1);
        assertEquals(new Integer(2), asyncLoadingCache.get(1).get());
```

即提前定义了自动加载的策略

## 淘汰策略

### 基于容量淘汰

```java
// 基于容量策略的淘汰
        Cache<Integer, Integer> cache = Caffeine.newBuilder()
                .maximumSize(10) // 也可以对一个对象获取其中一个属性作为评价指标
                .build();

        for (int i = 0; i < 100; i++) {
            cache.put(i, i);
        }

        // 标志过期，但是没有清除
        assertEquals(100, cache.estimatedSize());
        for (int i = 0; i < cache.estimatedSize(); i++) {
            Integer n = cache.getIfPresent(i);
            if (n != null) {
                System.out.println(n); // 输出的内容多余10个 , 即只清理了部分
            }
        }
```

### 基于时间淘汰

```java
// 基于固定的过期时间驱逐策略
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfterAccess(5, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build(key -> createExpensiveGraph(key));

// 基于不同的过期驱逐策略
LoadingCache<Key, Graph> graphs = Caffeine.newBuilder()
    .expireAfter(new Expiry<Key, Graph>() {
      public long expireAfterCreate(Key key, Graph graph, long currentTime) {
        // Use wall clock time, rather than nanotime, if from an external resource
        long seconds = graph.creationDate().plusHours(5)
            .minus(System.currentTimeMillis(), MILLIS)
            .toEpochSecond();
        return TimeUnit.SECONDS.toNanos(seconds);
      }
      public long expireAfterUpdate(Key key, Graph graph, 
          long currentTime, long currentDuration) {
        return currentDuration;
      }
      public long expireAfterRead(Key key, Graph graph,
          long currentTime, long currentDuration) {
        return currentDuration;
      }
    })
    .build(key -> createExpensiveGraph(key));
```

- `expireAfterAccess(long, TimeUnit):` 一个元素在上一次读写操作后一段时间之后，在指定的时间后没有被再次访问将会被认定为过期项。在当被缓存的元素时被绑定在一个session上时，当session因为不活跃而使元素过期的情况下，这是理想的选择。
- `expireAfterWrite(long, TimeUnit):` 一个元素将会在其创建或者最近一次被更新之后的一段时间后被认定为过期项。在对被缓存的元素的时效性存在要求的场景下，这是理想的选择。
- `expireAfter(Expiry):` 一个元素将会在指定的时间后被认定为过期项。当被缓存的元素过期时间收到外部资源影响的时候，这是理想的选择。

### 基于引用淘汰

首先需要了解，如果


`Caffeine.weakKeys()` 在保存key的时候将会进行弱引用。这允许在GC的过程中，当key没有被任何强引用指向的时候去将缓存元素回收。由于GC只依赖于引用相等性。这导致在这个情况下，缓存将会通过引用相等(==)而不是对象相等 `equals()`去进行key之间的比较。
这里可能就会存在无法命中string 的key的情况
https://www.zhyea.com/2019/05/06/caffeinecache-weakkeys-flaw.html


`Caffeine.weakValues()`在保存value的时候将会使用弱引用。这允许在GC的过程中，当value没有被任何强引用指向的时候去将缓存元素回收。由于GC只依赖于引用相等性。这导致在这个情况下，缓存将会通过引用相等(==)而不是对象相等 `equals()`去进行value之间的比较。

`Caffeine.softValues()`在保存value的时候将会使用软引用。为了相应内存的需要，在GC过程中被软引用的对象将会被通过LRU算法回收。由于使用软引用可能会影响整体性能，我们还是建议通过使用基于缓存容量的驱逐策略代替软引用的使用。同样的，使用 `softValues()` 将会通过引用相等(==)而不是对象相等 `equals()`去进行value之间的比较。

##### Cache<String,WeakReference\<?>> 和 weakValue 的区别




`AsyncCache`不支持软引用和弱引用。

```java
// 基于引用策略
        cache = Caffeine.newBuilder()
                .weakKeys() // 弱引用 ，不紧张也会被清理
                .weakValues()
                .build();

        cache = Caffeine.newBuilder()
                .softValues() // 软引用 ， 内存紧张，发生GC ，就会被清理
                .build();
```

也可以支持动态修改

```java
Cache<Integer, Integer> cache = Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(10, TimeUnit.SECONDS)
                .refreshAfterWrite(10, TimeUnit.SECONDS)
                .build();

        // 重新设置最大容量
        cache.policy().eviction().ifPresent(integerIntegerEviction -> {
            integerIntegerEviction.setMaximum(2 * integerIntegerEviction.getMaximum());
        });

        // 重新设置时间
        cache.policy().expireAfterAccess().ifPresent(integerIntegerExpiration -> {
            integerIntegerExpiration.setExpiresAfter(3, TimeUnit.SECONDS);
        });
```

## 刷新

刷新指的是 可以调用 refresh 方法 ，重新刷新值 ， 在刷新的过程中，如果获取值，返回的依旧是旧值。

对于 refreshAfterWrite 逻辑 ，如果到时间了，将其标识为 允许刷新 的状态 ，但是不会主动刷新； 直到下一次查询的时候，如果发现是 允许刷新 的状态，才会认为是过期的 ，那么就去刷新。

当 refreshAfterWrite 和 expireAfterWrite 一起使用的时候 ， 就会发现当到了需要 refresh 的时候，因为不会刷新，所以标识过期了 ，这样也不会和 expire 时间冲突

如果刷新的时候，希望有老的值的参与，可以覆写 cacheLoader 的 reload 方法 ，不然还是继续重新计算key 对应的 value 逻辑

```java
LoadingCache<Integer, Integer> loadingCache = Caffeine.newBuilder()
                .refreshAfterWrite(10, TimeUnit.SECONDS)
                .expireAfterWrite(30, TimeUnit.SECONDS)
                .build(new CacheLoader<Integer, Integer>() {
                    @Override
                    public @Nullable Integer load(@NonNull Integer key) throws Exception {
                        return key + 1; // load 的时候 +1
                    }

                    @Override
                    public @Nullable Integer reload(@NonNull Integer key, @NonNull Integer oldValue) throws Exception {
                        return oldValue + 10; // 刷新的时候 加10
                    }
                });
        assertEquals(new Integer(2), loadingCache.get(1));
        loadingCache.refresh(1);
        // 这里不会load 了
        assertEquals(new Integer(2 + 10), loadingCache.get(1));
```

## 缓存状态

```java
Cache<Integer, Integer> cache = Caffeine.newBuilder()
                .maximumSize(10_000)
                .recordStats()
                .build();

        for (int i = 0; i < 100_000; i++) {
            cache.put(i, i + 1);
            cache.get(i - 10_000, k -> k - 10_1000 + 1);
        }

        // pull 模式拉报告
        cache.stats();
        /**
         * CacheStats{hitCount=7300,
         * missCount=92700,
         * loadSuccessCount=92700,
         * loadFailureCount=0,
         * totalLoadTime=3113953,
         * evictionCount=182588,
         * evictionWeight=182588}
         */
```

### estimatedSize
`estimatedSize()` 方法用于估计缓存中的条目数。它提供了一个近似值，而不是精确的准确值

`estimatedSize()` 方法通过遍历所有的哈希桶，并计算每个哈希桶中链表的长度，然后将这些长度相加来估计缓存中的条目数。这个估计值并不是实时计算的，而是在某些特定的操作（如读取或修改操作）触发时进行更新。

## 性能

[Memory overhead zh CN](https://github.com/ben-manes/caffeine/wiki/Memory-overhead-zh-CN)

## 参考