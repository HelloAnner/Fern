代理模式是设置一个中间代理来控制访问原目标对象，以达到增强原对象的功能和简化访问方式

### 静态代理

优点：可以在不修改目标对象的前提下扩展目标对象的功能。

缺点：

1. 冗余。由于代理对象要实现与目标对象一致的接口，会产生过多的代理类
2. 不易维护。一旦接口增加方法，目标对象与代理对象都要进行修改

```java
package com.proxy;

public interface IUserDao {
    public void save();
}
```

```java
package com.proxy;

public class UserDao implements IUserDao{

    @Override
    public void save() {
        System.out.println("保存数据");
    }
}
```

- 静态代理对象：UserDapProxy **_需要实现IUserDao接口！_**
```java
package com.proxy;

public class UserDaoProxy implements IUserDao{

    private IUserDao target;
    public UserDaoProxy(IUserDao target) {
        this.target = target;
    }
    
    @Override
    public void save() {
        System.out.println("开启事务");//扩展了额外功能
        target.save();
        System.out.println("提交事务");
    }
}

```

### 动态代理

动态代理利用了[JDK API](https://link.segmentfault.com/?enc=GWK3raqqyEh2u0033%2BsQ5g%3D%3D.Zz13TgOi%2Bl0yljXideyk4t03IKTKxFYKZOSvsJq%2FliF4tAzvdjbjaq%2BcplGRi%2Fie)，动态地在内存中构建代理对象，从而实现对目标对象的代理功能。动态代理又被称为JDK代理或接口代理。

静态代理与动态代理的区别主要在：

- 静态代理在编译时就已经实现，编译完成后代理类是一个实际的class文件
- 动态代理是在运行时动态生成的，即编译完成后没有实际的class文件，而是在运行时动态生成类字节码，并加载到JVM中

动态代理对象不需要实现接口，但是要求目标对象必须实现接口，否则不能使用动态代理。

```java
package com.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ProxyFactory {

    private Object target;// 维护一个目标对象

    public ProxyFactory(Object target) {
        this.target = target;
    }

    // 为目标对象生成代理对象
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                new InvocationHandler() {

                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开启事务");

                        // 执行目标对象方法
                        Object returnValue = method.invoke(target, args);

                        System.out.println("提交事务");
                        return null;
                    }
                });
    }
}
```

- ? 为什么只能代理接口？
	在 Java 中，动态代理是通过生成代理类的字节码来实现的。当使用 `Proxy` 类的 `newProxyInstance` 方法创建代理对象时，它会动态地生成一个代理类，并将该代理类加载到内存中。这个代理类实现了被代理接口，并将方法调用转发给 `InvocationHandler` 的 `invoke` 方法。
	生成代理类的字节码需要遵循 Java 字节码规范，而在字节码中，只有接口可以被直接实现。具体的类在字节码层面上是被继承的 - 这也是 CGLIB 的原理
	动态代理的目标是为了实现面向接口的代理，它主要用于实现面向接口的编程风格，支持接口的解耦和灵活性。通过代理接口，可以在运行时动态地将代理逻辑应用于不同的实现类

### cglib代理

[cglib](https://link.segmentfault.com/?enc=82GbdEZSDPypzHfY9OOlnA%3D%3D.e5JdvS6%2BNdE%2BD6V5MG6WxzuVYn1eFxw2FWx5%2BcshM9c%3D) (Code Generation Library )是一个第三方代码生成类库，运行时在内存中动态生成一个子类对象从而实现对目标对象功能的扩展。

- CGLIB是一个强大的高性能的代码生成包，它可以在运行期扩展Java类与实现Java接口。  
    它广泛的被许多AOP的框架使用，例如Spring AOP和dynaop，为他们提供方法的interception（拦截）。
- CGLIB包的底层是通过使用一个小而快的字节码处理框架ASM，来转换字节码并生成新的类。  
    不鼓励直接使用ASM，因为它需要你对JVM内部结构包括class文件的格式和指令集都很熟悉。

```java
package com.cglib;

import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class ProxyFactory implements MethodInterceptor{

    private Object target;//维护一个目标对象
    public ProxyFactory(Object target) {
        this.target = target;
    }
    
    //为目标对象生成代理对象
    public Object getProxyInstance() {
        //工具类
        Enhancer en = new Enhancer();
        //设置父类
        en.setSuperclass(target.getClass());
        //设置回调函数
        en.setCallback(this);
        //创建子类对象代理
        return en.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("开启事务");
        // 执行目标对象的方法
        Object returnValue = method.invoke(target, args);
        System.out.println("关闭事务");
        return null;
    }
}
```

动态代理生成的类为 lass com.sun.proxy.\$Proxy4
cglib代理生成的类为class com.cglib.UserDao\$\$EnhancerByCGLIB\$\$552188b6

cglib会继承目标对象，需要重写方法，所以目标对象不能为final类


- ? 为什么 cglib 可以直接使用类 ， 符合字节码规范么
	在字节码规范中，除了接口可以被直接实现外，类还可以被继承。CGLIB通过创建被代理类的子类来实现代理，这个子类继承了被代理类并重写了被代理方法，从而在运行时拦截和处理方法调用。
	虽然CGLIB生成的代理类并不符合传统的字节码规范中的"接口代理"，但它利用了字节码规范中的"子类代理"手段，使得可以直接代理具体的类。

- [ ] 复习 Java代理和实践 (@2023-12-10)