

hashCode()方法和equals()方法的作用其实一样，在Java里都是用来对比两个对象是否相等一致

**equals()既然已经能实现对比的功能了，为什么还要hashCode()呢？**

因为重写的equals（）里一般比较的比较全面比较复杂，这样效率就比较低，而利用hashCode()进行对比，则只要生成一个hash值进行比较就可以了，效率很高。

**hashCode()既然效率这么高为什么还要equals()呢？**

因为hashCode()并不是完全可靠，有时候不同的对象他们生成的hashcode也会一样（生成hash值得公式可能存在的问题），所以hashCode()只能说是大部分时候可靠，并不是绝对可靠，所以我们可以得出：

- equals()相等的两个对象他们的hashCode()肯定相等，也就是用equals()对比是绝对可靠的。
- hashCode()相等的两个对象他们的equals()不一定相等，也就是hashCode()不是绝对可靠的。

**为什么equals()相等，hashCode就一定要相等，而hashCode相等，却不要求equals相等?**

如果equals相等，但是hashcode不相等，在散列表中的结果会错乱

equals相等，即默认逻辑上这个对象就是一个，但是hashcode不同，散列表hash获取索引的时候会到不同的entry，然后继续查找，会导致散列表错乱

```java
public V put(K key, V value) {
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key.hashCode());
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

之所以**hashCode相等，却可以equal不等**，就比如ObjectA和ObjectB他们都有属性name，那么hashCode都以name计算，所以hashCode一样，但是两个对象属于不同类型，所以equals为false

###  为什么需要hashCode?

- 通过hashCode可以很快的查到小内存块。
- 通过hashCode比较比equals方法快，当get时先比较hashCode，如果hashCode不同，直接返回false

### 为什么重写equals 一定需要重写hashcode ？

hashCode()和equals()一样都是基本类Object里的方法，而和equals()一样，**Object里hashCode()计算和内存地址无关**， [[../JVM/JVM HashCode 和 内存地址的关系|JVM HashCode 和 内存地址的关系]] 
如果是这样的话，那么我们相同的一个类，new两个对象，由于他们在内存里的地址不同，则他们的hashCode（）不同，导致equals相等，但是hashcode不相等的情况出现。

所以这显然不是我们想要的，所以我们必须重写我们类的hashCode()方法，即一个类，在hashCode()里面返回唯一的一个hash值