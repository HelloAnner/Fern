## 搜索类 - sc

```bash
sc demo.*

#[d]	输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的ClassLoader等详细信息。 如果一个类被多个ClassLoader所加载，则会出现多次
sc -d demo.MathGame

# 输出当前类的成员变量信息（需要配合参数-d一起使用）
sc -d -f demo.MathGame
```

## 搜索方法 - sm

```bash
sm demo.MathGame

# 展示每个方法的详细信息
sm -d demo.MathGame
```

## 反编译 - jad

```bash
jad java.lang.String

# 反编译结果里会带有ClassLoader信息，通过--source-only选项，可以只打印源代码
jad --source-only demo.MathGame

# 反编译时指定ClassLoader
jad org.apache.log4j.Logger -c 69dcaba4
```

## 编译 - mc

```bash
mc /home/ligen/Test.java

# -d命令指定输出目录
mc /home/ligen/Test.java -d /home/ligen
```

## 重新加载 - ****redefine****

- 不允许新增加field/method，只能基于已有的方法和成员变量
- 正在跑的函数，没有退出不能生效，比如下面新增加的`System.out.println`，只有`main()`函数里的不会生效

```bash
jad --source-only demo.MathGame >> /home/ligen/MathGame.java
# 修改java文件
mc /home/ligen/MathGame.java -d /home/ligen/
redefine /home/ligen/demo/MathGame.class
```

```
thread --all > /Users/anner/Downloads/1.txt
thread --state TIMED_WAITING -n 3
```

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

## watch 查看参数

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

```
stack org.apache.commons.lang.StringUtils isBlank
stack *StringUtils isBlank
stack *StringUtils isBlank params[0].length==1
stack *StringUtils isBlank '#cost>100'
stack -E org\\.apache\\.commons\\.lang\\.StringUtils isBlank
```

### 火焰图

[https://www.yuque.com/anner-dhcrb/wlzvwi/pwy51xf4lr4q9cqg](https://www.yuque.com/anner-dhcrb/wlzvwi/pwy51xf4lr4q9cqg)

[https://arthas.aliyun.com/doc/profiler.html#参数说明](https://arthas.aliyun.com/doc/profiler.html#%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E)

profiler 命令支持生成应用热点的火焰图。本质上是通过不断的采样，然后把收集到的采样结果生成火焰图

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

profiler getSamples

profiler resume

profiler stop --format html
```

file参数支持一些变量：

- 时间戳： --file /tmp/test-%t.jfr
- 进程 ID： --file /tmp/test-%p.jfr

生成的结果可以用支持 jfr 格式的工具来查看。比如：

- JDK Mission Control ： [https://github.com/openjdk/jmc](https://github.com/openjdk/jmc)
- JProfiler ： [https://github.com/alibaba/arthas/issues/1416](https://github.com/alibaba/arthas/issues/1416)
