
Java8 本身也提供了一些函数式编程的思想接口，将整个代码的结构和思想向着函数式编程的范式去演进。

### Consumer

Consumer 接口接收一个参数。对参数做一系列操作，不存在返回值。

在流式处理中，其意义就是类似 foreach 可以直接处理

```java
Consumer<String> printLength = s -> System.out.println(s.length());
Arrays.asList("Hello", "World").forEach(printLength);
```

如果希望直接调用，可以直接使用 accept 逻辑完成 ， 流操作的另一个逻辑就是 andThen

```java
Arrays.asList("Hello", "Consumer").forEach(
                printLength.andThen(s -> System.out.println(s.isEmpty()))
        );
```

---

### Supplier

对比于 Consumer 接口， Supplier 接口是一个供给型的接口

Supplier 接口可以当做一个功能提供的角色， 还可以完成延迟初始化或者类似随机数生成器

```java
Supplier<Integer> randomInt = () -> ThreadLocalRandom.current().nextInt();
System.out.println(randomInt.get());
```

Supplier 不存在消费任何参数的性质，所以起没有andThen操作，只有一个 get逻辑

---

### Predicate

Predicate 接口是一个谓词性接口，接收一个参数，返回一个布尔值。

可以将其使用在条件判断的位置上完成。

```java
				// predicate
        Predicate<Integer> greaterThanTen = num -> num > 10;
        Stream.of(1,5,10,15)
                .filter(greaterThanTen)
                .collect(Collectors.toList());
```

可以看到 filter 的参数都是 Predicate 函数式接口

---

### Function

上述接口一般都是存在固定场景，最后加一个通用的Function接口 ， 即完成函数的通用功能 - 转换； 将输入的数据转换为输出的数据。

```java
				// function
        Function<Integer, Integer> square = n -> n * n;
        Stream.of(1, 2, 3, 4, 5).map(square).forEach(System.out::println);
```

可以看到 map 里面的参数都是 Function 类型的接口数据

---

### BIFunction

下面介绍一个比较少见的，但是必要情况可以使用一下

BIFunction 表示接收两个参数，返回一个结果的情况。

```java
BiFunction<Integer, Integer, Integer> add = Integer::sum;
System.out.println(add.apply(1, 1));
```

---

### Runnable 

如果希望延时计算，没有任何参数和返回任何的参数，可以使用这个

比如不希望重复写 try-catch 逻辑
```java
private void safeExecute(Runnable closeFunc) {  
    try {  
        closeFunc.run();  
    } catch (Exception | Error e) {  
        FineLoggerFactory.getLogger().error("execute fail", e);  
    }  
}
```


细节可以继续查看: `package java.util.function;`


- [ ] 复习 Java函数式编程 (@2023-12-14)