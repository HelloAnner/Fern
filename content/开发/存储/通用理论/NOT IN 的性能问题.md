
## NOT IN 慢的原因

如果在 `NOT IN` 子句中包含的列表非常大，查询性能可能会下降。每个要排除的值都需要逐一与查询结果进行比较，这在大数据集上可能会导致性能问题。

**NOT IN 是不走索引的，需要一一比较**

## IN 操作

当使用 `IN` 子句时，数据库优化器会尝试将其转换为更高效的操作，如使用 `HASH JOIN` 或 `MERGE JOIN` 等技术来处理。这些技术可以通过创建临时哈希表或排序表来加快查询速度。优化器会尝试选择最佳的执行计划，以减少比较操作的次数，并利用索引等数据结构来加速查询。

此外，`IN` 子句通常比 `NOT IN` 子句更容易进行优化，因为在 `IN` 中，数据库可以使用索引来查找匹配的值。而在 `NOT IN` 中，数据库需要执行逐一比较，以查找在子查询结果中不存在的值。

 虽然IN 走了索引优化 ， 数据量大的时候，IN 还是可能会很慢

_**阿里开发规约，也有这样的建议：in 操作能避免则避免，若实在避免不了，需要仔细评估 in 后边的集合元素数量，控 制在 1000 个之内**_

## 优化

原始的慢SQL： （子查询存在几十万条数据，主表也是存在几十万条，耗时30min）
```sql
SELECT id FROM fine_user_role_middle WHERE userid NOT IN (SELECT id FROM fine_user u WHERE u.creationType = 2);
```

### 业务上思考降低 IN 里面的数据数量

```sql
SELECT id FROM fine_user_role_middle WHERE userid  IN (SELECT id FROM fine_user u WHERE u.creationType = 1 or creationType = 0);
```

### 使用 left join

```sql
select  a.id from fine_user_role_middle a left join fine_user b on a.userId <> b.id where b.creationType = 2;
```

