## [classloader](https://arthas.aliyun.com/doc/classloader.html)

`classloader` 命令将 JVM 中所有的 classloader 的信息统计出来，并可以展示继承树，urls

```
# 按类型查看信息
classloader

# 查看继承树
classloader -t

# 查看加载的包
classloader -c [hash]
```

![[attachments/e29a96791ee83304c3c4b5c15e46da72_MD5.jpeg]]
![[attachments/d570d212c66abfcf7874eb6b379ad735_MD5.jpeg]]
![[attachments/79d4d59e857220db504295020728ef8d_MD5.jpeg]]
## 搜索类 - sc

```bash
sc demo.*

#[d]	输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的ClassLoader等详细信息。 如果一个类被多个ClassLoader所加载，则会出现多次
sc -d demo.MathGame

# 输出当前类的成员变量信息（需要配合参数-d一起使用）
sc -d -f demo.MathGame
```
![[attachments/78e740ce7a806234ec33c40a54475033_MD5.jpeg]]

## 搜索方法 - [sm](https://arthas.aliyun.com/doc/sm.html#%E4%BD%BF%E7%94%A8%E5%8F%82%E8%80%83)

```bash
sm demo.MathGame

# 展示每个方法的详细信息
sm -d demo.MathGame
```
![[attachments/Pasted image 20240118185925.png]]
![[attachments/c9fdaa31070610a2c013f464ba90b62c_MD5.jpeg]]
## 反编译 - [jad](https://arthas.aliyun.com/doc/jad.html#%E5%8F%8D%E7%BC%96%E8%AF%91%E6%97%B6%E6%8C%87%E5%AE%9A-classloader)

```bash
jad java.lang.String

# 反编译结果里会带有ClassLoader信息，通过--source-only选项，可以只打印源代码
jad --source-only demo.MathGame

# 指定某一个方法+ 不显示行号
jad --source-only  com.fr.decision.authority.controller.SoftDataControllerImpl findById --lineNumber false

# 反编译时指定ClassLoader
jad org.apache.log4j.Logger -c 69dcaba4
```
![[attachments/cd3fb8703b49e385bca76a0693c6ee58_MD5.jpeg]]
## 编译 - mc

```bash
mc /home/ligen/Test.java

# -d命令指定输出目录
mc /home/ligen/Test.java -d /home/ligen
```

## 重新加载 - **redefine**

- 不允许新增加field/method，只能基于已有的方法和成员变量
- 正在跑的函数，没有退出不能生效，比如下面新增加的`System.out.println`，只有`main()`函数里的不会生效

```bash
jad --source-only demo.MathGame >> /home/ligen/MathGame.java
# 修改java文件
mc /home/ligen/MathGame.java -d /home/ligen/
redefine /home/ligen/demo/MathGame.class
```


## [thread](https://arthas.aliyun.com/doc/thread.html)

查看当前线程信息，查看线程的堆栈

这里的 cpu 使用率与 linux 命令`top -H -p <pid>` 的线程`%CPU`类似，一段采样间隔时间内，当前 JVM 里各个线程的增量 cpu 时间与采样间隔时间的比例。


```
thread --all > /Users/anner/Downloads/1.txt
# 查看指定状态 ， 耗时前三个的线程
thread --state TIMED_WAITING -n 3

# 查找最耗时的线程 block
thread -b

# 指定采样时间
thread -n 3 -i 1000
```
![[attachments/88ca80aa903d40d0afea1c9454c4aa0f_MD5.jpeg]]

## [trace](https://arthas.aliyun.com/doc/trace.html)

```
trace org.apache.commons.lang.StringUtils isBlank
trace *StringUtils isBlank
trace *StringUtils isBlank params[0].length==1
trace *StringUtils isBlank '#cost>100'
trace -E org\\\\.apache\\\\.commons\\\\.lang\\\\.StringUtils isBlank
trace -E com.test.ClassA|org.test.ClassB method1|method2|method3
trace demo.MathGame run -n 5
trace demo.MathGame run --skipJDKMethod false
trace javax.servlet.Filter * --exclude-class-pattern com.demo.TestFilter
trace OuterClass$InnerClass *
```


## [watch](https://arthas.aliyun.com/doc/watch.html) 查看参数

-b 函数调用前

-s 函数返回后

-e 函数异常

-f 函数正常结束或者异常结束

-x 参数的查看堆栈 ，一个bean 可能需要多个层数查看

```
watch com.fr.decision.webservice.v10.login.LoginService login "{params,target,returnObj,throwExp}"

# 比较常用的场景
watch  com.fr.decision.webservice.v10.entry.ReportEntryService getReportTemplateTree "{params[0],returnObj}"

watch -b com.fr.decision.webservice.v10.entry.ReportEntryService getReportTemplateTree params -x 2

watch -f com.fr.decision.webservice.v10.entry.ReportEntryService getReportTemplateTree returnObj
```

## [stack](https://arthas.aliyun.com/doc/stack.html)

输出当前方法被调用的调用路径

```
stack org.apache.commons.lang.StringUtils isBlank

stack *StringUtils isBlank
stack *StringUtils isBlank params[0].length==1
stack *StringUtils isBlank '#cost>100'
stack -E org\\.apache\\.commons\\.lang\\.StringUtils isBlank
```
![[attachments/c283dfb45c4880116721dae84ee4a6be_MD5.jpeg]]
## [profiler](https://arthas.aliyun.com/doc/profiler.html#%E4%BD%BF%E7%94%A8execute%E6%9D%A5%E6%89%A7%E8%A1%8C%E5%A4%8D%E6%9D%82%E7%9A%84%E5%91%BD%E4%BB%A4) 火焰图


profiler 命令支持生成应用热点的火焰图。本质上是通过不断的采样，然后把收集到的采样结果生成火焰图

>y 轴表示调用栈，每一层都是一个函数。调用栈越深，火焰就越高，顶部就是正在执行的函数，下方都是它的父函数。
>
>x 轴表示抽样数，如果一个函数在 x 轴占据的宽度越宽，就表示它被抽到的次数多，即执行的时间长。注意，x 轴不代表时间，而是所有的调用栈合并后，按字母顺序排列的。
>
>**火焰图就是看顶层的哪个函数占据的宽度最大。只要有"平顶"（plateaus），就表示该函数可能存在性能问题。**

颜色没有特殊含义，因为火焰图表示的是 CPU 的繁忙程度，所以一般选择暖色调。



profiler 区分不同的event 默认是采集cpu

在不同的平台，不同的 OS 下面，支持的 events 各有不同。比如在 macos 下面：

```
profiler list

Basic events:
  cpu
  alloc
  lock
  wall
  itimer
Java method calls:
  ClassName.methodName
```

```
profiler start

# 指定运行的时间
profiler start --duration 300
# jfr 只支持在 start时配置。如果是在stop时指定不生效
profiler start --file /tmp/test.jfr


profiler status

# 获取已采集的 sample 的数量
profiler getSamples

# start是新开始采样，resume会保留上次`stop`时的数据。
profiler resume

profiler stop --format html
```

file参数支持一些变量：

- 时间戳： --file /tmp/test-%t.jfr
- 进程 ID： --file /tmp/test-%p.jfr

生成的结果可以用支持 jfr 格式的工具来查看。比如：

- JDK Mission Control ： [https://github.com/openjdk/jmc](https://github.com/openjdk/jmc)
- JProfiler ： [https://github.com/alibaba/arthas/issues/1416](https://github.com/alibaba/arthas/issues/1416)

![[attachments/47a681967c33bde40a884c2daa588c0b_MD5.jpeg]]
