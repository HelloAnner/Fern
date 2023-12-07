
### 运行时数据区

- 程序计数器：当前线程的字节码的行号选择器，每一个线程私有的
- Java 虚拟机栈：每一个方法执行，都会push进去一个栈帧，存储局部变量表、操作数栈；
- 局部变量表：在栈帧中存放了编译期可知的数据类型和堆中的数据引用 - 存储在局部变量槽中，即运行前已经完全确定了，运行期间不需要任何修改；比如方法参数信息
- 本地方法栈：虚拟机栈为字节码服务，本地方法栈为 Native 方法服务
- Java堆：存放对象实例的位置；物理上可以不连续、但是逻辑上需要时连续的
- 方法区：存储类型信息、常量、静态变量、JIT编译后的数据
    - Java8之前一般是永久代，因为设计问题，永久代就被纳入堆管理中，并且是存在默认大小的，所以很容易直接内存溢出；但是新的设计中，只要不到系统线程内存限制就不会存在问题
    - 运行时常量池： class文件解析后，会生成一个常量池表
- 直接内存：不受到虚拟机内存的限制，可以直接分配；比如 NIO可以直接使用 native 直接分配直接内存

### Java 对象创建过程

new → 检查常量池中的类的符号引用 → 为对象分配内存 （分配一个全部连续的内存空间 或者 按照空闲列表分配非连续的空间） → 设置对象头: 元数据、哈希码

### 对象的内存布局

- 对象头 ： hashcode 、锁状态、GC分代年龄
- 实例数据 ： 存储的真正的有效信息，包括从父类继承的数据
- 对齐填充：对象的起始地址必须是 8 字节的整数倍



### 对象的访问定位

JVM 规范中只规定了引用类型是指向对象的引用，并没有限制具体的实现
因此，不同虚拟机的实现方式可能不同。通常有两种实现形式：句柄访问和直接指针访问。

#### 句柄访问
栈中存储的 reference → （毕竟一个 reference 无法定义对象是如何定位，如果可以直接定位了，那么必须存储对象的大小等信息才可以） ，这里走到的是一个堆中的特殊区域 - 句柄池 ，这样的好处就是隔离了一层，堆中的对象移动之后，不用修改 reference 信息
当 GC 操作移动对象时只用维护句柄池中存储的信息即可，特别是多个变量都引用同一个句柄池中的句柄时，可以减少更新变量存储的引用，同时确保变量的地址不变。

缺点就是多了一次中转，访问效率会有影响。

#### 直接指针访问

直接指针访问省去了中间的句柄池，对象引用中保持的直接是对象地址。

这种方式很明显节省了一次指针定位的开销，访问速度快。但是当GC发生对象移动时，变量中保持的引用地址也需要维护，如果多个变量指向一个地址，需要更新多次。 **Hot Spot虚拟机** 便是基于这种方式实现的。

#### 验证

```xml
<dependency>
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol-core</artifactId>
    <version>0.10</version>
</dependency>
```

```java
public class TestHashCode {

    public static void main(String[] args) {
        Object obj = new Object();
        long address = VM.current().addressOf(obj);
        long hashCode = obj.hashCode();
        System.out.println("before GC : The memory address is " + address);
        System.out.println("before GC : The hash code is " + hashCode);

        new Object();
        new Object();
        new Object();

        System.gc();

        long afterAddress = VM.current().addressOf(obj);
        long afterHashCode = obj.hashCode();
        System.out.println("after GC : The memory address is " + afterAddress);
        System.out.println("after GC : The hash code is " + afterHashCode);
        System.out.println("---------------------");

        System.out.println("memory address = " + (address == afterAddress));
        System.out.println("hash code = " + (hashCode == afterHashCode));
    }
}

```

上述代码执环境为Hotspot虚拟机，执行时如果未出现GC，则可将JVM参数设置的小一点，比如可以设置为16M：-Xms16m -Xmx16m -XX:+PrintGCDetails。

```
before GC : The memory address is 31856020608
before GC : The hash code is 2065530879
after GC : The memory address is 28991167544
after GC : The hash code is 2065530879
---------------------
memory address = false
hash code = true
```

上面的控制台信息可以看出，GC 前后对象的地址的确变了，但 hashCode 却并未发生变化。同时也可以看出 hashcode 的值与内存地址的值是完全不一样的 [[JVM HashCode 和 内存地址的关系]]