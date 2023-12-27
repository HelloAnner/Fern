
## varchar

varchar[(n)]：长度为 <mark style="background: #FFF3A3A6;">n 个字节</mark>的可变长度且非 Unicode的字符数据。n 必须是一个介于   1 和 8,000之间的数值。存储大小为输入数据的字节的实际长度，而不是 n 个字节。所输入的数据字符长度可以为零。
<mark style="background: #FF5582A6;">对于 sql server ， n 的含义就是字节了</mark>




注意： ANSI主要是以单字节来存储数据，一般适合英文。而我们常用的汉字需要用两个字节来存储，所以就要使用unicode的数据类型，不然读取出来的数据可能会乱码。

## nvarchar

nvarchar(n) ：包含 <mark style="background: #FFF3A3A6;">n个字符</mark>的可变长度 Unicode 字符数据。n 的值必须介于 1  与  4,000 之间。字节的存储大小是所输入字符个数的两倍。所输入的数据字符长度可以为零。

```java
    @Column(name = COLUMN_VALUE, length = 3000)
    @Nationalized
    private String value;
```

```sql
`i18nValue` varchar(3000) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci DEFAULT NULL
```

上面是国际化存储的建表语句，对于 Mysql ， 虽然表结构的定义是 nvarchar 类型，但是其对于这个这个字段会修改一下定义和指定为国际化类型的编码。


## 区别

从存储方式上，nvarchar是按字符存储的，而 varchar是按字节存储的

从存储量上考虑， varchar比较节省空间，因为存储大小为字节的实际长度，而 nvarchar是双字节存储

在使用上，如果存储内容都是英文字符而没有汉字等其他语言符号，建议使用varchar；含有汉字的使用nvarchar，因为nvarchar是使用Unicode编码，即统一的字符编码标准，会减少乱码的出现几率

如果你做的项目可能涉及不同语言之间的转换，建议用nvarchar

## 参考

[浅谈SQL Server、MySQL中char，varchar，nchar，nvarchar区别-阿里云开发者社区](https://developer.aliyun.com/article/646218)
[Sql Server中的nvarchar(n)、varchar(n) 和Mysql中的char(n)、varchar(n) - 神雕爱大侠 - 博客园](https://www.cnblogs.com/sdadx/p/7908415.html)

