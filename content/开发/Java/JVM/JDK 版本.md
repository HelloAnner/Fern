
### 市场版本差异

openjdk 是完全开源免费的 ， 很多软件都是选择内置此类，如 Jetbrain

openJDK安装仅支持源码，其他商用的如 OracleJDK安装只有二进制安装包

oracle jdk 和 azul zulu jdk 都是基于 openjdk 构建的 ， 只是加了安全补丁和错误修复；

两者都是做了 TCK 测试 ，即保证完全兼容Java SE 的JDK

oracle 自己也说明从 java11 之后 ，Oracle jdk 和 openJDK 的功能基本一致 [https://blogs.oracle.com/java/post/oracle-jdk-releases-for-java-11-and-later](https://blogs.oracle.com/java/post/oracle-jdk-releases-for-java-11-and-later)

OpenJDK不支持 LTS 服务 ， OracleJdk 每三年推出一个LTS版本。 和 OracleJDK 对标的应该是 Zulu 类似的其他商业公司支持的JDK

OracleJDK 修改了协议，除了演示、开发、测试的场景，其他都是需要收费的

OracleJDK 也不捆绑 JavaFX ， 不支持32位构建了

关于 JavaFX ， zulu 单独发行了内置JavaFX的版本，以支持类似 image 绘制的场景

### OpenJDK 演进功能概览