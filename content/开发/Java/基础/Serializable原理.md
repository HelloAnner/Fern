
Serializable接口没有方法和属性，只是一个**识别类可被序列化的标志**

```java
/**
 * Classes that require special handling during the serialization and
 * deserialization process must implement special methods with these exact
 * signatures:
 *
 * <PRE>
 * private void writeObject(java.io.ObjectOutputStream out)
 *     throws IOException
 * private void readObject(java.io.ObjectInputStream in)
 *     throws IOException, ClassNotFoundException;
 * private void readObjectNoData()
 *     throws ObjectStreamException;
 * </PRE>
 */
```

serialVersionUID 是 Java 为每个序列化类产生的版本标识，可用来保证在反序列时，发送方发送的和接受方接收的是可兼容的对象。**如果接收方接收的类的 serialVersionUID 与发送方发送的 serialVersionUID 不一致，进行反序列时会抛出 InvalidClassException**。序列化的类可显式声明 serialVersionUID 的值

不同的 jdk 编译很可能会生成不同的 serialVersionUID 默认值，进而导致在反序列化时抛出 InvalidClassExceptions 异常

**serialVersionUID 的修饰符最好是 private**，因为 serialVersionUID 不能被继承，所以建议使用 private 修饰 serialVersionUID

serialVersionUID 就是控制版本是否兼容的，若我们认为修改的类是向后兼容的，则不修改 serialVersionUID；反之，则提高 serialVersionUID的值

若不显式定义 serialVersionUID 的值，Java 会根据类细节自动生成 serialVersionUID 的值，如果对类的源代码作了修改，再重新编译，新生成的类文件的serialVersionUID的取值有可能也会发生变化。类的serialVersionUID的默认值完全依赖于Java编译器的实现，对于同一个类，用不同的Java编译器编译，也有可能会导致不同的serialVersionUID

源码中类型是long类型，所以需要自己指定一个Long类型