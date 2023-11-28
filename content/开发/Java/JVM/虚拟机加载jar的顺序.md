
**1.先搜索JVM自带的jar或zip包。**

Bootstrap，搜索路径可以用`System.getProperty("sun.boot.class.path")`获得；

**2.搜索`JRE_HOME/lib/ext`下的jar包。**

Extension，搜索路径可以用`System.getProperty("java.ext.dirs")`获得；

**3.搜索用户自定义目录，顺序为：当前目录（.），CLASSPATH，-cp。**

搜索路径用`System.getProperty("java.class.path")`获得。

```
System.out.println(System.getProperty("sun.boot.class.path"));
System.out.println(System.getProperty("java.ext.dirs"));
System.out.println(System.getProperty("java.class.path"));
```