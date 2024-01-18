```properties
#Hibernate configure properties
#Fri Dec 10 18:56:33 CST 2021
# 数据库连接密码
hibernate.connection.password=
# 连接池中连接的最大等待时间，当连接池中没有可用连接时，请求连接的线程将等待该时间后抛出异常，默认为 -1，表示无限等待
hibernate.maxWait=500000
# 连接池中连接最短空闲时间，单位为毫秒
hibernate.minEvictableIdleTimeMillis=1800000

# 当数据库表结构发生变化时，Hibernate 的自动更新策略，这里的值为 "update"，表示会自动更新
hibernate.hbm2ddl.auto=update


# 连接池初始化大小连接池的初始化连接数，默认为 0
hibernate.initialSize=0
hibernate.default_schema=
# 连接池中连接的验证 SQL，用于检测连接是否有效，默认为 null
hibernate.validationQuery=
# 连接池中连接是否空闲时测试可用性
hibernate.testWhileIdle=false
hibernate.connection.username=ppd_sinofinebi_adm
# 事务隔离级别 ， 此处为串行化
hibernate.connection.isolation=4
hibernate.connection.driver_class=com.mysql.jdbc.Driver
# 连接池中空闲连接的检测周期，默认为 -1，表示不检测
hibernate.timeBetweenEvictionRunsMillis=-1
hibernate.connection.provider_class=com.fr.config.FineDBDruidConnectionProvider
# 连接池中连接借用时是否测试可用性
hibernate.testOnBorrow=true
# 连接池中连接归还时是否测试可用性
hibernate.testOnReturn=false
# Hibernate 方言，用于生成 SQL 语句
hibernate.dialect=com.fr.third.org.hibernate.dialect.MySQL8Dialect
# 连接池中空闲连接检测线程每次检测的连接数量
hibernate.numTestsPerEvictionRun=3
hibernate.connection.url=jdbc\:mysql\://pc-bp167p75ck3i4ul8r.rwlb.rds.aliyuncs.com\:3306/ppd_sino_finebi?useUnicode\=true&characterEncoding\=UTF-8
# 连接池中最大连接数，当连接数达到该值时，连接池将阻塞请求新的连接，默认为 8
hibernate.maxActive=50
# 连接池中最小空闲连接数，低于该值时，连接池将创建新的连接以达到该值，默认为 0
hibernate.minIdle=0
```

### hibernate.hbm2ddl.auto

`hibernate.hbm2ddl.auto" value="update` won't modify existing table column definitions. Doing some testing I found that:

`hibernate.hbm2ddl.auto" value="update` will add a db column that doesn't already exist.

`hibernate.hbm2ddl.auto" value="update` will not delete a db column that is removed/no longer in your entity.

`hibernate.hbm2ddl.auto" value="update` will not modify a db column that has already been created.

`hibernate.hbm2ddl.auto=update` 可能会更新数据库模式以保持与实体类定义的一致性，包括创建**新表**、添加新列、修改列的数据类型和约束等。它**并不会删除已有的列或修改已经创建的列的长度**



其他选项：

- _validate_: validate the schema, makes no changes to the database.
- _create-only_: database creation will be generated.
- _drop_: database dropping will be generated.
- _update_: update the schema.
- _create_: creates the schema, destroying previous data.
- _create-drop_: drop the schema when the SessionFactory is closed explicitly, typically when the application is stopped.
- _none_: does nothing with the schema, makes no changes to the database



**Hibernate Druid 层面是不能控制类似数据库的读写超时时间的参数**