### 现象
目前我们的外置库可以迁移到 GP ，但是会出现类似：

次管登录慢： 原因是计算次管管理的节点信息，查权限表慢 - 因为权限表的六个联合唯一键创建失败了

超管登录，数据量大的时候，每一个位置都很慢，请求基本都是10s+ ： 还是因为所有的表唯一键创建失败了


```sql
CREATE TABLE "public"."trm_concept" (
"pid" int8 NOT NULL,
"code" varchar(100)  NOT NULL,
"codesystem_pid" int8,
"display" varchar(400) ,
"index_status" int8,
CONSTRAINT "trm_concept_pkey" PRIMARY KEY ("pid"),
CONSTRAINT "idx_concept_cs_code" UNIQUE ("codesystem_pid", "code")
);
```

```sql
ERROR:  Greenplum Database does not allow having both PRIMARY KEY and UNIQUE constraints
```

### 分析

在大部分传统数据库中，索引能够极大地改善数据访问时间。不过，在一个Greenplum之类的分布式数据库中，索引应该被更保守地使用

Greenplum数据库会执行非常快的顺序扫描，索引则使用一种随机搜索的模式在磁盘上定位记录。Greenplum的数据分布在Segment上，因此每个Segment会扫描全体数据的一小部分来得到结果。通过表分区，要扫描的数据量可能会更少。因为商业智能（BI）查询负载通常会返回非常大的数据集，使用索引并不是很有效。

索引更有可能为OLTP负载改进性能，在那种场景中查询会返回一个单一记录或者数据的一个小的子集。在被压缩过的追加优化表上，索引也可以提高返回一个目标行集合的查询的性能，因为优化器在适当的时候可以使用一种索引访问方法而不是全表扫描。对于压缩过的数据，使用一种索引访问方法意味着只有必要的行会被解压。

Greenplum数据库会自动为带有主键的表创建PRIMARY KEY约束。要在一个被分区的表上创建索引，就在用户创建的分区表上创建一个索引。该索引会被传播到Greenplum数据库所创建的所有子表上。不支持在Greenplum数据库为分区表创建的子表上创建索引。

注意一个UNIQUE CONSTRAINT（例如PRIMARY KEY CONSTRAINT）会隐式地创建一个UNIQUE INDEX，它必须包括分布键中所有的列以及任何分区键。UNIQUE CONSTRAINT会在整个表上被强制要求，包括所有的表分区（如果有）


In Greenplum Database, unique indexes are allowed only if the columns of the index key are the same as (or a superset of) the Greenplum distribution key. On partitioned tables, a unique index is only supported within an individual partition - not across all partitions.

For a table to have a primary key, it must be hash distributed (not randomly distributed), and the primary key The column(s) that are unique must contain all the columns of the Greenplum distribution key.

[在Greenplum数据库中使用索引 | Greenplum数据库文档](https://gp-docs-cn.github.io/docs/admin_guide/ddl/ddl-index.html)

### 备份到GP数据库的转换小工具

[gp数据库备份唯一键冲突](https://kms.fineres.com/pages/viewpage.action?pageId=946475806)

### 相关说明

[[Page not found](https://docs.vmware.com/en/VMware-Greenplum/index.html](https://docs.vmware.com/en/VMware-Greenplum/index.html))

[postgresql - How should I deal with my UNIQUE constraints during my data migration from Postgres9.4 to Greenplum - Stack Overflow](https://stackoverflow.com/questions/40987460/how-should-i-deal-with-my-unique-constraints-during-my-data-migration-from-postg)


---

https://kms.fineres.com/pages/viewpage.action?pageId=972295532