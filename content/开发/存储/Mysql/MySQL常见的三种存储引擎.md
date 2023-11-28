虽然MySQL里的存储引擎不只是MyISAM与InnoDB这两个，但常用的就是两个

**InnoDB支持事务，MyISAM不支持**，这一点是非常之重要。事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而MyISAM就不可以了。

MyISAM适合查询以及插入为主的应用。
InnoDB适合频繁修改以及涉及到安全性较高的应用

InnoDB支持外键，MyISAM不支持。

从MySQL5.5.5以后，InnoDB是默认引擎

· InnoDB不支持FULLTEXT类型的索引。

· InnoDB中不保存表的行数，如select count() from table时，InnoDB需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count()语句包含where条件时MyISAM也需要扫描整个表。

· 对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中可以和其他字段一起建立联合索引。

DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的 删除，效率非常慢。MyISAM则会重建表。

· InnoDB支持行锁（某些情况下还是锁整表，如 update table set a=1 where user like '%lee%'。


如果你的应用程序对查询性能要求较高，就要使用MyISAM了。MyISAM索引和数据是分开的，而且其索引是压缩的，可以更好地利用内存。所以它的**查询性能明显优于INNODB**。压缩后的索引也能节约一些磁盘空间。**MyISAM拥有全文索引的功能，这可以极大地优化LIKE查询的效率**。


**现在一般都是选用innodb了，主要是MyISAM的全表锁，读写串行问题，并发效率锁表，效率低**，MyISAM对于读写密集型应用一般是不会去选用的。

- [ ] 复习 MySQL常见的三种存储引擎 (@2023-11-23)