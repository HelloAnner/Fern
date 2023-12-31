
### 版本差异

`com.mysql.jdbc.Driver` 是 mysql-connector-java 5中的，   
`com.mysql.cj.jdbc.Driver` 是 mysql-connector-java 6中的

|mysql-connector-java|MySQL|JDK|补充|
|---|---|---|---|
|8.0.x|5.6、5.7、8.0 4.2|JDK 8.0或更高版本|全面上市。推荐版本。|
|5.1.x|5.6、5.7、8.0 3.0、4.0、4.1、4.2|JDK 5.0和JDK 8.0或更高版本|一般可用性|

如果mysql-connector-java用的6.0以上的 ， 但是driver用的还是`com.mysql.jdbc.Driver`，就会报错：
```
Loading class 'com.mysql.jdbc.Driver'. This is deprecated. The new 
driver class is 'com.mysql.cj.jdbc.Driver'. 
The driver is automatically registered via the SPI 
and manual loading of the driver class is generally unnecessary.
```

### SSL 差异

MySQL 5.5.45+, 5.6.26+ and 5.7.6+版本默认要求建立SSL连接,如果你不需要使用SSL连接，你需要通过设置`useSSL=false`来显式禁用SSL连接。
```
WARN: Establishing SSL connection without server’s identity verification is not recommended. 
According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection 
must be established by default if explicit option isn’t set. 
For compliance with existing applications not using SSL the verifyServerCertificate property is set to ‘false’. 
You need either to explicitly disable SSL by setting useSSL=false, 
or set useSSL=true and provide truststore for server certificate verification.
```

8.0x是不需要建立ssl连接的

### 时区差异

JDBC连接Mysql6 `com.mysql.cj.jdbc.Driver`， 需要指定时区serverTimezone

```
java.sql.SQLException: The server time zone value '???ú±ê×??±??' is unrecognized or represents more than one time zone.
```

需要增加
```
useSSL\=false&serverTimezone\=Asia/Shanghai&zeroDateTimeBehavior\=CONVERT_TO_NULL
```