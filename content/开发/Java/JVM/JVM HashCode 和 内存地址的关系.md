hashCode 是 java.lang.Object.hashCode() 或者 java.lang.System.identityHashCode(obj) 会返回的值。他是一个对象的身份标识。官方称呼为：标识哈希码（ identity hash code）

#### hashcode 的特点

- 一个对象在其生命期中 identity hash code 必定保持不变
- 如果 a == b，那么他们的 System.identityHashCode() 必须相等
- 如果 System.identityHashCode() 相等的话，并不能保证 a == b（毕竟这只是一个散列值，是允许冲突的）

#### 为什么hashCode()性能高

因为 hashCode()的结果算出来后缓存起来，下次调用直接用不需要重新计算，提高性能


#### identityHashCode
看官方提供的 API , 对 System.identityHashCode()的解释为 :  
public static int **identityHashCode** ([Object] x)
>返回给定对象的哈希码，该代码与默认的方法 hashCode() 返回的代码一样，无论给定对象的类是否重写 hashCode()。null 引用的哈希码为 0。

```java
 @Test
    public void testHashCode() {
        TestExample example = new TestExample();
        int exampleCode = System.identityHashCode(example);
        int exampleCode2 = System.identityHashCode(example);
        int exampleHashcode = example.hashCode();
        System.out.println("example identityHashCode:" + exampleCode);
        System.out.println("example2 identityHashCode:" + exampleCode2);
        System.out.println("example Hashcode:" + exampleHashcode);
        String str = "dd";
        String str2 = "dd";
        int strCode = System.identityHashCode(str);
        int strHashCode = str.hashCode();
        int str2HashCode = str.hashCode();
        System.out.println("str identityHashCode:" + strCode);
        System.out.println("str hashcode:" + strHashCode);
        System.out.println("str2 hashcode:" + str2HashCode);
    }
输出：
example identityHashCode:1144748369
example2 identityHashCode:1144748369
example Hashcode:1144748369
str identityHashCode:340870931
str hashcode:3200
str2 hashcode:3200 

```

#### hashCode 如何计算的
```
public native int hashCode(); 
```

即该方法是一个本地方法，Java 将调用本地方法库对此方法的实现。由于 Object 类中有 JNI 方法调用，按照 JNI 的规则，应当生成 JNI 的头文件，在此目录下执行 **javah -jni java.lang.Object** 指令，将生成一个 **java_lang_Object.h** 头文件

```c
/*
 * Class:     java_lang_Object
 * Method:    hashCode
 * Signature: ()I
 */
JNIEXPORT jint JNICALL Java_java_lang_Object_hashCode
  (JNIEnv *, jobject); 

```

```c
/*-
 *      Implementation of class Object
 *
 *      former threadruntime.c, Sun Sep 22 12:09:39 1991
 */
#include <stdio.h>
#include <signal.h>
#include <limits.h>
#include "jni.h"
#include "jni_util.h"
#include "jvm.h"
//使用了上面生成的java_lang_Object.h文件
#include "java_lang_Object.h"
static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM_IHashCode},//hashcode的方法指针JVM_IHashCode
    {"wait",        "(J)V",                   (void *)&JVM_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM_Clone},
}; 

```

生成 hash 最终由 get_next_hash 函数生成，该函数提供了基于某个 hashCode 变量值的六种方法。怎么生成最终值取决于 hashCode 这个变量值

##### 6 种 hashCode 算法
0 - 使用 Park-Miller 伪随机数生成器（跟地址无关）  
1 - 使用地址与一个随机数做异或（地址是输入因素的一部分）  
2 - 总是返回常量 1 作为所有对象的 identity hash code（跟地址无关）  
3 - 使用全局的递增数列（跟地址无关）  
4 - 使用对象地址的“当前”地址来作为它的 identity hash code（就是当前地址）  
5 - 使用线程局部状态来实现 Marsaglia’s xor-shift 随机数生成（跟地址无关）

##### VM 到底用的是哪种方法？
在 openjdk\hotspot\src\share\vm\runtime\globals.hpp 定义  
JDK 8 和 JDK 9 默认值：

> product(intx, hashCode, 5,“(Unstable) select hashCode generation algorithm”) ;

JDK 8 以前默认值：

> product(intx, hashCode, 0,“(Unstable) select hashCode generation algorithm”) ;  
> 不同的 JDK，生成方式不一样。

注意：  
虽然方式不一样，但有个共同点：*java 生成的 hashCode 和对象内存地址没什么关系*。

HotSpot 提供了一个 VM 参数来让用户选择 identity hash code 的生成方式：  
`-XX:hashCode`

#### 什么时候计算出来的
在 VM 里，Java 对象会在首次真正使用到它的 identity hash code（例如通过 Object.hashCode() / System.identityHashCode()）时调用 VM 里的函数来计算出值，然后会保存在对象里，后面对同一对象查询其 identity hash code 时总是会返回最初记录的值。

因此，不是对象创建时。

对此以 Hotspot 为例，最直接的实现方式就是在对象的 header 区域中划分出来一部分（32位机器上是占用25位，64位机器上占用31）用来存储 hashcode 值。但这种方式会添加额外信息到对象中，而在大多数情况下 hashCode 方法并不会被调用，这就造成空间浪费。

那么JVM是如何进行优化的呢？当hashCode方法未被调用时，object header中用来存储hashcode的位置为0，只有当hashCode方法（本质上是System#identityHashCode）首次被调用时，才会计算对应的hashcode值，并存储到object header中。当再次被调用时，则直接获取计算好hashcode即可。

**上述实现方式就保证了即使 GC 发生，对象地址发生了变化，也不影响 hashcode 的值。比如在 GC 发生前调用了 hashCode 方法，hashcode 值已经被存储，即使地址变了也没关系；在 GC 发生后调用 hashCode 方法更是如此**

```java
// 创建对象并打印JVM中对象的信息
Object person = new Object();
System.out.println(ClassLayout.parseInstance(person).toPrintable());
// 调用hashCode方法，如果重写了hashCode方法则调用System#identityHashCode方法
System.out.println(person.hashCode());
// System.out.println(System.identityHashCode(person));
// 再次打印对象JVM中的信息
System.out.println(ClassLayout.parseInstance(person).toPrintable());

```
执行上述程序，控制台打印如下：
```
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total
1898220577
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 21 8c 24 (00000001 00100001 10001100 00100100) (613163265)
      4     4        (object header)                           71 00 00 00 (01110001 00000000 00000000 00000000) (113)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

```

在调用hashCode方法前后，我们可以看到OFFSET为0的一行存储的值（Value），从原来的1变为613163265，也就是说将hashcode的值进行了存储。如果未调用对应方法，则不会进行存储 。

