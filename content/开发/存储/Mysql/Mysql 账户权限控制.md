![[Pasted image 20231030144027.png|500]]

- **数据权限**分为：库、表和字段三种级别
- **管理权限**主要是管理员要使用到的权限，包括：数据库创建，临时表创建、主从部署、进程管理等
- **程序权限**主要是触发器、存储过程、函数等权限

DDL（Data Definition Language）：create，alter，drop，truncate  - 表结构
DML（Data Manipulation Language）：insert，update，delete - 表数据

### 常用的权限

- ALTER : 修改表结构
- CREATE : 创建数据库、表、索引
- CREATE USER : 创建用户
- CREATE VIEW ： 创建视图
- DELETE：删除表数据
- INDEX： 在已经存在的表创建索引
- INSERT： 插入数据
- SELECT： 查询数据
- UPDATE：更新数据库的权限
- FILE： load file  或者 into file 操作的权限

### Database Level

相比于系统权限 ， 少了一些系统级别的权限 ，如 

CREATE USER,FILE,PROCESS,RELOAD,REPLICATION CLIENT,REPLICATION SLAVE, SHOW DATABASES, SHUTDOWN

其他的都是一样的

```sql
grant all on test.* to test3,test4@'localhost' identified by 'test123';
```

### Table Level

```sql
grant all on test.test1 to wolf@'%' identified by 'wolf@123';
```


### 查询用户的权限

```sql
show grants for kelly@'%';
```


### 创建一个用户 & 分配权限

```sql
CREATE USER 'bnner'@'%' IDENTIFIED BY '1111';

GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'bnner'@'%';

FLUSH PRIVILEGES;
```

可以看到新建一个表会直接失败
```sql
MySQL bnner@10.211.55.4:tmp> CREATE TABLE test_table (
                          ->   id INT PRIMARY KEY,
                          ->   name VARCHAR(50)
                          -> );
(1142, "CREATE command denied to user 'bnner'@'10.211.55.2' for table 'test_table'")
MySQL bnner@10.211.55.4:tmp>
```


- [ ] 复习 Mysql 账户权限控制  (@2023-12-27)

