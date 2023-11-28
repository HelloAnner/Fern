
在JVM中，一个类用其全名+类加载器作为唯一标识

### 线程上下文类加载器

Java提供了很多服务提供者接口（Service Provider Interface，SPI），常见的有JDBC、JNDI、JAXP等。这些SPI接口由Java核心库提供（通过BootstrapClassLoader加载），而它们的实现由第三方库提供（通过SystemClassLoader加载）

BootstrapClassLoader是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给SystemClassLoader，因为它是SystemClassLoader的祖先类

线程上下文类加载器就是解决这个问题，如果不做任何的设置，Java的线程的上下文类加载器默认就是SystemClassLoader。在SPI接口的代码中使用线程上下文类加载器，就可以成功的加载到SPI实现的类。

使用线程上下文加载器，要注意保证多个需要通信的线程间的类加载器应该是同一个，**防止因为不同的类加载器导致类型转换异常(ClassCastException)**

[](https://www.notion.so/fb03f32a088d4d7cb199dcf366c7b135?pvs=21)

### 解决方法

- 序列化和反序列化过渡一下

```java
ObjectMapper mapper = new ObjectMapper();
String sourceStr = mapper.writeValueAsString(syncSource);
LdapSyncSource ldapSyncSource = mapper.readValue(sourceStr, LdapSyncSource.class);
```

- 写一个工具类，保证同一个类加载使用到