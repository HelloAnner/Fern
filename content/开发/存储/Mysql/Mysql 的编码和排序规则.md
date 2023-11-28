
mysql 里面的 utf8 == utf8mb3 , 无法存储一些特殊符号

mysql8 之后 , 修改为 utf8mb4

对于一个汉字 , 在mysql 8中

- utf8 编码 , 一个汉字三个字节
- utf8mb4 是四个字节
- gbk 两个字节
- gb2312 两个字节

utf8mb4_0900_ai_ci ：

- `utf8mb4`: 表示字符集 , 支持辅助字符集
- 0900: unicode版本号
- ai: 排序规则 ,按照字母排序 ; general 表示通用的 , 一般是不区分大小写的
- ci : case insensitivate 不区分大小写 - cs 区分大小写

对于 注解 national 标识的 nvarchar 字段 , mysql 不区分 , 都是使用 varchar 去存储

### 排序规则 - 大小写敏感

查看一个表的编码规则 - 查看是不是大小写敏感的最好是直接查看表的，不要直接看数据库的

```sql
show full columns from fine_authority;

+---------------------+--------------+-----------------+------+-----+---------+-------+---------------------------------+---------+
| Field               | Type         | Collation       | Null | Key | Default | Extra | Privileges                      | Comment |
+---------------------+--------------+-----------------+------+-----+---------+-------+---------------------------------+---------+
| id                  | varchar(255) | utf8_general_ci | NO   | PRI | <null>  |       | select,insert,update,references |         |
| authority           | int(11)      | <null>          | NO   |     | <null>  |       | select,insert,update,references |         |
| authorityEntityId   | varchar(255) | utf8_general_ci | NO   |     | <null>  |       | select,insert,update,references |         |
| authorityEntityType | int(11)      | <null>          | NO   |     | <null>  |       | select,insert,update,references |         |
| authorityType       | int(11)      | <null>          | NO   |     | <null>  |       | select,insert,update,references |         |
| roleId              | varchar(255) | utf8_general_ci | NO   | MUL | <null>  |       | select,insert,update,references |         |
| roleType            | int(11)      | <null>          | NO   |     | <null>  |       | select,insert,update,references |         |
| tenantId            | varchar(255) | utf8_general_ci | NO   |     | default |       | select,insert,update,references |         |
+---------------------+--------------+-----------------+------+-----+---------+-------+---------------------------------+---------+

```

修改表的编码规则

### 全角和半角识别问题

全角（Full-width）和半角（Half-width）指的是字符所占的宽度
英文环境中，通常使用的是半角字符，而全角字符更常用于东亚语言（如中文、日文、韩文）的排版和输入。在计算机系统和软件中，处理英文文本时，通常默认使用半角字符。


utf8mb4_bin 支持识别 () 和 （）, 可以插入数据，保证唯一键
utf8mb4_0900_ai_ci 就会唯一键冲突
但是都是 utf8mb4 的字符集，只有编码规则不同

Unicode编码是一种标准的字符编码，用于表示全球范围内的字符
在Unicode编码中，半角字符和全角字符都有对应的编码。在某些排序规则中，比如`utf8mb4_bin`，半角字符和全角字符会被视为不同的字符
其他排序规则如`utf8mb4_unicode_ci`和`utf8mb4_general_ci`等则会将相应的半角和全角字符视为相同的字符。

区分的编码：
utf8mb4_bin
utf8mb4_zh_0900_as_cs

不区分的编码排序：
utf8mb4_unicode_ci
utf8mb4_general_ci
utf8mb4_unicode_ci

https://dev.mysql.com/doc/refman/8.0/en/charset-mysql.html


- [ ] 复习 mysql编码规则

