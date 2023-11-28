
背景：

反射调用比较慢

反射调用API可以绕过安全检查。 什么是安全检查： 比如Java严格的类型系统检查等特性，丢失后，只能在运行时才会执行的对应的检查

byte buddy 创建自己的类: 最终是make() 完成一个unload 的对象，还是比较符合语义的

```java
DynamicType.Unloaded<?> dynamicType = new ByteBuddy()
                // 允许自定义创建自己的类的名称
                .with(new NamingStrategy.AbstractBase() {
                    @Override
                    protected String name(TypeDescription typeDescription) {
                        return "Hello."+ typeDescription.getSimpleName();
                    }
                })
                .name("example.type")
                .make();
```

byte buddy 围绕一个不可变的对象构建自己的实例， 即可以表现为链式写法，如果希望对最后实现的 make() 方法单独返回对象，是会被直接丢弃的

byte buddy 中存在两类逻辑完成类的重写: redefine 和 rebase ； redefine 直接覆盖原来的实现 , rebase 会将原来的实现修改方法名称，类似 old.name$oringal

byte buddy 默认生成的对象类型是 DynamicType.Unloaded ，即 仅仅在内存中生成了这个二进制流，不会load到虚拟机中，所以我们可以选择将二进制流的基础操作，如写到文件或者写到jar里面。

仅仅是将class的二进制流保存在内存中是没有任何作用的，如果需要加载到虚拟机内部，还需要指定对应的classloader完成加载。

如果新建一个 classloader 来加载这个内存流， jvm 不会认为其和其他class相同，因为归属于不同的classloader ，所以即使是同名的class都是认为其是不同的类型。所以自然无法访问任何jvm中的其他类型；

同理，在加载类的时候，jvm会使用同一个classloader完成关联的class加载，如果关联的class是其他的classloader创建的，那么加载这个关联关系就会失败。

—>

那么如何解决这个问题？ JVM第一次运行的时候其实是懒加载解析引用的类。

---

byte buddy 最终还是选择自己创建一个类加载器，比如创建一个子类加载优先的加载，比如 child_first 策略；

warpper策略即基于目前的classloader ，封装一个新的；

inject 即反射注入动态类型

下面对 unload 的类型指定加载器和模式，开始load 完成到 class 对象的转换

```java
Class<?> clazz = new ByteBuddy()
                // 允许自定义创建自己的类的名称
                .with(new NamingStrategy.AbstractBase() {
                    @Override
                    protected String name(TypeDescription typeDescription) {
                        return "Hello."+ typeDescription.getSimpleName();
                    }
                })
                .subclass(Object.class)
                .make()
                .load(ClassNotFoundException.class.getClassLoader(), ClassLoadingStrategy.Default.WRAPPER)
                .getLoaded();
```

---

上述均为如何重新创建一个新的 class 和加载到 虚拟机中 ， 重新使用一个新的 classloader 来加载一个新的 class ， 我理解理论是不存在任何的问题的。

对于已经加载进去的class ， 可以使用 redefine 完成其功能的覆盖

```java
static class Foo {
        String m() {
            return  "foo";
        }
    }

    static class Bar {
        String m(){
            return "bar";
        }
    }

    public static void main(String[] args) {
        ByteBuddyAgent.install();
        Foo foo = new Foo();
        new ByteBuddy()
								// 新的方法实现
                .redefine(Bar.class)
								// 重新加载的目标位置
                .name(Foo.class.getName())
                .make()
                .load(Foo.class.getClassLoader(),ClassReloadingStrategy.fromInstalledAgent());

        System.out.println(foo.m()); // bar
    }
```

上述完成将已经加载的 foo 对象重新使用 Bar 的类型去重新写

Java 的热加载需要使用 agent完成，上述可以看到已经使用 byte buddy 提供的内容完成了一次加载

同时，Java的热加载需要保证新的 Bar 里面的方法等完全和对应的 Foo 保持一致

---

下面这个例子使用 load 的 class 对象 使用反射的方法完成到一个对象实例的创建，然后调用 object 默认的 tostring 逻辑

```java
String s =  new ByteBuddy()
               .subclass(Object.class)
               .name("example.type")
               .make()
               .load(DebugPlugin.class.getClassLoader())
               .getLoaded()
               .newInstance()
               .toString();

        System.out.println(s); // example.type@25bbe1b6
```

下面这个例子将父类的方法重新拦截和对返回值做了一个重新定义

```java
String s = new ByteBuddy()
                .subclass(Object.class)
                .name("example.type")
                .method(ElementMatchers.named("toString")).intercept(FixedValue.value("Hello"))
                .make()
                .load(DebugPlugin.class.getClassLoader())
                .getLoaded()
                .newInstance()
                .toString();

        System.out.println(s); // hello
```

如果存在多个同名的逻辑，可以添加 过滤完成匹配

```java
String s = new ByteBuddy()
                .subclass(Object.class)
                .name("example.type")
                .method(ElementMatchers.named("toString")
                        .and(ElementMatchers.returns(String.class))
                        .and(ElementMatchers.takesArguments(0))).
                intercept(FixedValue.value("Hello"))
                .make()
                .load(DebugPlugin.class.getClassLoader())
                .getLoaded()
                .newInstance()
                .toString();

        System.out.println(s); // hello
```

上述是基于自己构建的class对象来修改父类的逻辑，同时使用反射完成到具体的实例的逻辑。

如果我们不希望是继承来修改对应的类，即不想自己修改自己，想直接使用另一个class完成对当前的class的一个方法的逻辑的重写（这个场景还是比较常见的）

```java
static class Foo {
        public String m() {
            return "foo";
        }
    }

    static class Bar {
        public String m() {
            return "bar";
        }
    }

    public static void main(String[] args) throws InstantiationException, IllegalAccessException {
        ByteBuddyAgent.install();
        String s = new ByteBuddy()
                .subclass(Foo.class)
                .method(ElementMatchers.named("m")
                        .and(ElementMatchers.returns(String.class))
                        .and(ElementMatchers.takesArguments(0))).intercept(MethodDelegation.to(Bar.class))
                .make()
                .load(DebugPlugin.class.getClassLoader())
                .getLoaded()
                .newInstance()
                .m();

        System.out.println(s); // hello
    }
```

## 参考