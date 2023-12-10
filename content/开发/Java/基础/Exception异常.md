
Error类和Exception类的父类都是throwable类，他们的区别是：

- Error类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢等。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这样的错误，建议让程序终止。
- Exception类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常

Exception类又分为运行时异常（Runtime Exception）和受检查的异常(Checked Exception )

- 运行时异常;ArithmaticException,IllegalArgumentException，编译能通过，但是一运行就终止了，程序不会处理运行时异常，出现这类异常，程序会终止
- 受检查的异常，要么用try。。。catch捕获，要么用throws字句声明抛出，交给它的父类处理，否则编译不会通过
![[Pasted image 20231029180308.png]]

**建议使用非检查异常让代码更加简洁，而且更容易保持接口的稳定性。**

## **字节码层面分析异常处理**
![[Pasted image 20231029180318.png]]

图片中的字节码是在 JDK 1.6 （class 文件的版本号为50，表示java编译器的版本为jdk 1.6）及之前的编译器生成的，因为有 jsr 和 ret 指令可以使用。然而在 idea 中通过 **jclasslib 插件** 查看 try-catch-finally 的字节码文件并没有 jsr/ret 指令，通过查阅资料，有如下说明：

**jsr / ret 机制最初用于实现finally块，但是他们认为节省代码大小并不值得额外的复杂性，因此逐渐被淘汰了。Sun JDK 1.6之后的javac就不生成jsr/ret指令了，那finally块要如何实现？**

**javac采用的办法是把finally块的内容复制到原本每个jsr指令所在的地方，这样就不需要jsr/ret了，代价则是字节码大小会膨胀，但是降低了字节码的复杂性，因为减少了两个字节码指令（jsr/ret）。**

## 异常处理不规范案例

### 捕获

- 捕获异常的时候不区分异常类型
- 捕获异常不完全，比如该捕获的异常类型没有捕获到

```java
try{
    ……
} catch (Exception e){ // 不应对所有类型的异常统一捕获，应该抽象出业务异常和系统异常，分别捕获
    ……
}
```

### **传递**

- 异常信息丢失
- 异常信息转译错误，比如在抛出异常的时候将业务异常包装成了系统异常
- 吃掉异常
- 不必要的异常包装
- 检查异常传递过程中不适用非检查检异常包装，造成代码被throws污染

```java
try{
    ……
} catch (BIZException e){ 
    throw new BIZException(e); // 重复包装同样类型的异常信息 
} catch (Biz1Exception e){ 
    throw new BIZException(e.getMessage()); // 没有抛出异常栈信息，正确的做法是throw new BIZException(e); 
} catch (Biz2Exception e){
    throw new Exception(e); // 不能使用低抽象级别的异常去包装高抽象级别的异常，这样在传递过程中丢失了异常类型信息
} catch (Biz3Exception e){
    throw new Exception(……); // 异常转译错误，将业务异常直接转译成了系统异常
} catch (Biz4Exception e){
    …… // 不抛出也不记Log，直接吃掉异常
} catch (Exception e){
    throw e;
}
```

### **处理**

- 重复处理
- 处理方式不统一
- 处理位置分散

```java
try{
    try{
        try{
            ……
        } catch (Biz1Exception e){
            log.error(e);  // 重复的LOG记录
            throw new e;
        }
        
        try{
            ……
        } catch (Biz2Exception e){
            ……  // 同样是业务异常，既在内层处理，又在外层处理
        }
    } catch (BizException e){
        log.error(e); // 重复的LOG记录
        throw e;
    }
} catch (Exception e){
    // 通吃所有类型的异常
    log.error(e.getMessage(),e);
}
```

## 异常处理规范案例

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/80fe5df2-b86f-45e3-948b-fe6bf779fb2d/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fcb51c3d-88f2-449c-a8e4-9af4e768ee78/Untitled.png)

1、使用 try-with-resource 关闭资源。

2、抛出具体的异常而不是 Exception，并在注释中使用 @throw 进行说明。

3、捕获异常后使用描述性语言记录错误信息，如果是调用外部服务最好是包括入参和出参。

```
 logger.error("说明信息，异常信息：{}", e.getMessage(), e)
```

4、优先捕获具体异常。

5、不要捕获 Throwable 异常，除非特殊情况。

6、不要忽略异常，异常捕获一定需要处理。

7、不要同时记录和抛出异常，因为异常会打印多次，正确的处理方式要么抛出异常要么记录异常，如果抛出异常，不要原封不动的抛出，可以自定义异常抛出。

8、自定义异常不要丢弃原有异常，应该将原始异常传入自定义异常中。

```
throw MyException("my exception", e);
```

9、**自定义异常尽量不要使用检查异常**。

10、尽可能晚的捕获异常，如非必要，建议所有的异常都不要在下层捕获，而应该由最上层捕获并统一处理这些异常。。

11、为了避免重复输出异常日志，建议所有的异常日志都统一交由最上层输出。就算下层捕获到了某个异常，如非特殊情况，也不要将异常信息输出，应该交给最上层统一输出日志。

## 项目中的异常处理实践

应用程序中定义的异常应该分为两类：

- 业务异常：用户能够看懂并且能够处理的异常，比如用户没有登录，提示用户登录即可。
- 系统异常：用户看不懂需要程序员处理的异常，比如网络连接超时，需要程序员排查相关问题。

![[Pasted image 20231029180343.png]]

## 参考


- [ ]  复习 Java 异常 (@2023-12-07)