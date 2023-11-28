查看建表语句，其对应的是  tinyblob 类型

`tinyblob` 是 MySQL 数据库中的一种数据类型，用于存储最多 255 字节的二进制数据。它是 `BLOB`（Binary Large Object）数据类型的一种变体

```text
Caused by: com.fr.third.org.hibernate.type.SerializationException: could not deserialize
	at com.fr.third.org.hibernate.internal.util.SerializationHelper.doDeserialize(SerializationHelper.java:243)
	at com.fr.third.org.hibernate.internal.util.SerializationHelper.deserialize(SerializationHelper.java:287)
	at com.fr.third.org.hibernate.type.descriptor.java.SerializableTypeDescriptor.fromBytes(SerializableTypeDescriptor.java:138)
	at com.fr.third.org.hibernate.type.descriptor.java.SerializableTypeDescriptor.wrap(SerializableTypeDescriptor.java:113)
	at com.fr.third.org.hibernate.type.descriptor.java.SerializableTypeDescriptor.wrap(SerializableTypeDescriptor.java:27)
	at com.fr.third.org.hibernate.type.descriptor.sql.VarbinaryTypeDescriptor$2.doExtract(VarbinaryTypeDescriptor.java:60)
	at com.fr.third.org.hibernate.type.descriptor.sql.BasicExtractor.extract(BasicExtractor.java:47)
	at com.fr.third.org.hibernate.type.AbstractStandardBasicType.nullSafeGet(AbstractStandardBasicType.java:235)
	at com.fr.third.org.hibernate.type.AbstractStandardBasicType.nullSafeGet(AbstractStandardBasicType.java:231)
	at com.fr.third.org.hibernate.type.AbstractStandardBasicType.nullSafeGet(AbstractStandardBasicType.java:222)
	at com.fr.third.org.hibernate.type.AbstractStandardBasicType.hydrate(AbstractStandardBasicType.java:296)
	at com.fr.third.org.hibernate.persister.entity.AbstractEntityPersister.hydrate(AbstractEntityPersister.java:2838)
	at com.fr.third.org.hibernate.loader.Loader.loadFromResultSet(Loader.java:1735)
	at com.fr.third.org.hibernate.loader.Loader.instanceNotYetLoaded(Loader.java:1661)
	at com.fr.third.org.hibernate.loader.Loader.getRow(Loader.java:1550)
	at com.fr.third.org.hibernate.loader.Loader.getRowFromResultSet(Loader.java:733)
	at com.fr.third.org.hibernate.loader.Loader.processResultSet(Loader.java:978)
	at com.fr.third.org.hibernate.loader.Loader.doQuery(Loader.java:936)
	at com.fr.third.org.hibernate.loader.Loader.doQueryAndInitializeNonLazyCollections(Loader.java:342)
	at com.fr.third.org.hibernate.loader.Loader.doList(Loader.java:2622)
	at com.fr.third.org.hibernate.loader.Loader.doList(Loader.java:2605)
	at com.fr.third.org.hibernate.loader.Loader.listIgnoreQueryCache(Loader.java:2434)
	at com.fr.third.org.hibernate.loader.Loader.list(Loader.java:2429)
	at com.fr.third.org.hibernate.loader.criteria.CriteriaLoader.list(CriteriaLoader.java:109)
	at com.fr.third.org.hibernate.internal.SessionImpl.list(SessionImpl.java:1786)
	at com.fr.third.org.hibernate.internal.CriteriaImpl.list(CriteriaImpl.java:363)
	... 6 common frames omitted
Caused by: java.io.StreamCorruptedException: invalid stream header: C2ACED9C
	at java.io.ObjectInputStream.readStreamHeader(ObjectInputStream.java:808)
	at java.io.ObjectInputStream.<init>(ObjectInputStream.java:301)
	at com.fr.third.org.hibernate.internal.util.SerializationHelper$CustomObjectInputStream.<init>(SerializationHelper.java:309)
	at com.fr.third.org.hibernate.internal.util.SerializationHelper$CustomObjectInputStream.<init>(SerializationHelper.java:299)
	at com.fr.third.org.hibernate.internal.util.SerializationHelper.doDeserialize(SerializationHelper.java:218)
	... 31 common frames omitted
```

为什么 tinyblob 数据无法反序列化

如果是2个16进制字符转为8字节 ， 那么为什么无法转义呢

