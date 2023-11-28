
### 作用

1. 存储中间结果集：当需要进行复杂的查询操作时，可以将中间结果存储在临时表中，以便后续的查询和处理。
2. 缓存计算结果：如果某个查询结果是频繁使用的，可以将其存储在临时表中，以避免每次都重新计算。
3. 临时存储大量数据：当需要处理大量数据时，可以使用临时表暂时存储数据，以减轻内存压力或优化查询性能。

### 操作


临时表可以手动删除：
DROP TEMPORARY TABLE IF EXISTS temp_tb;

**临时表只在当前连接可见，当关闭连接时，MySQL会自动删除表并释放所有空间。因此在不同的连接中可以创建同名的临时表，并且操作属于本连接的临时表**。

创建临时表的语法与创建表语法类似，不同之处是**增加关键字TEMPORARY**，如：

```sql
CREATE TEMPORARY TABLE tmp_table (

NAME VARCHAR (10) NOT NULL,

time date NOT NULL

);

select * from tmp_table;

-- 或者

CREATE TEMPORARY TABLE temp_products AS
SELECT * FROM products WHERE price > 100;

UPDATE temp_products SET price = price * 0.9 WHERE id = 1;

DROP TEMPORARY TABLE temp_products;
```

- [ ] 复习 Mysql 临时表 (@2023-12-20)