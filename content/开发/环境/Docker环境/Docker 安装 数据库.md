### mysql

```shell
docker pull mysql/mysql-server:latest
docker run --name=mysql -d mysql/mysql-server:latest

docker logs mysql // 查看密码

ALTER USER 'root'@'localhost' IDENTIFIED BY '[newpassword]';

// 允许局域网连接 (1130, "192.168.65.1' is not allowed to connect to this MySQL server")


```
[[../../存储/Mysql/Mysql 账户权限控制|Mysql 账户权限控制]]

### sql server

```java
docker pull microsoft/mssql-server-linux:2017-latest

docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=qazWSX123' -p 1401:1433 --name sql-server -d microsoft/mssql-server-linux:2017-latest

docker exec -it sql-server "bash"

/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P ’qazWSX123’

CREATE DATABASE TestDB
SELECT Name from sys.Databases
GO

/opt/mssql-tools/bin/sqlcmd -s 127.0.0.1 -o 1401 -u SA -p "qazWSX123" "select name, database_id from sys.databases"
```

[mssql-jdbc-8.4.1.jre8.jar.zip](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/049b04b3-24a4-466c-82c9-033c2579b285/mssql-jdbc-8.4.1.jre8.jar.zip)

### oracle

```java
docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
docker run -dp 9090:8080 -p 1521:1521 registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
// ip：localhost 服务名：helowin 初始用户名密码：system/helowin
```

### db2

```java
docker pull ibmoms/db2express-c
docker run -it  --name db2 -p 50000:50000 -e DB2INST1_PASSWORD=db2root-pwd -e LICENSE=accept ibmoms/db2express-c:latest bash

-p 50000:50000 允许远程的客户端可以从50000 端口连接到数据库实例.
通过指定 -e DB2INST1_PASSWORD=db2root-pwd 参数, 你可以为缺省的Db2实例用户db2inst1设置密码.注意：这里“DB2INST1”是用户名，而“b2root-pwd”是密码。
通过指定-e LICENSE=accept参数, 表示你接受了使用Db2软件的许可证协议.

```

### PostgreSQL

```java
docker pull postgres
docker run --name mypostgres -d -p 5432:5432 -e POSTGRES_PASSWORD=123456 postgres
docker exec -it mypostgres psql -U postgres -d postgres
```

[postgresql-42.1.4.jar.zip](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a97dc6e-c03a-409e-b03d-0f3e9281239e/postgresql-42.1.4.jar.zip)