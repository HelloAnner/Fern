
DDL 语句中的 varchar 后面的长度代表的是 字符长度
```sql
create table te (id int primary key auto_increment , name varchar(5) collate utf8mb4_bin);
```
上面的SQL代表name字段可以存储5个字符 ， 5个英文的可以存储，5个中文字符也可以存储。
```
+------------+--------------+------------------------+
| name       | length(name) | character_length(name) |
+------------+--------------+------------------------+
| 2          | 1            | 1                      |
| abcde      | 5            | 5                      |
| abc开      | 6            | 4                      |
| abc开始    | 9            | 5                      |
| 一个好的开 | 15           | 5                      |
+------------+--------------+------------------------+
```

![[attachments/ebbb57c9d65e84cf7a3cc941c47f9d12_MD5.jpeg]]

![[attachments/535237acdef914968dc6ad8eeb6eebfe_MD5.jpeg]]