# 22.3 sql 语法基础

## 1. 数据定义
### 1.2 数据类型
1. 字符型：
    - 定长字符型：
        - CHAR(#)：不区分字符大小写
        - BINARY(#)：区分字符大小写
    - 变长字符型：
        - VARCHAR(#)：不区分字符大小写
        - VARBINARY(#)：区分字符大小写
    - 对象存储：
        - TEXT：不区分字符大小写
        - BLOB：区分字符大小写
    - 内置类型：
        - SET
        - ENUM                        
2. 数值型：
    - 精确数值型：
        - INT（TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT）
        - DECIMAL: 十进制数
    - 近似数值型：
        - FLOAT
        - DOBULE
3. 日期时间型：
    - 日期型：DATE
    - 时间型：TIME
    - 日期时间型：DATETIME
    - 时间戳：TIMESTAMP
    - 年份：YEAR(2), YEAR(4)

### 1.3 修饰符
1. 所有类型修饰符：
    - NOT NULL：非空；
    - DEFAULT  value：默认值；
    - primary key
    - unique key
2. 整型修饰符:
    - UNSIGNED：无符号
    - AUTO_INCREMENT: 自增


## 2. mysql 管理 - sql 语句
1. DDL：Data Defined Language
    - 数据定义语言，主要用于管理数据库组件，例如表、索引、视图、用户、存储过程
    - CREATE、ALTER、DROP
2. DML：Data Manapulating Language
    - 数据操纵语言，主要用管理表中的数据，实现数据的增、删、改、查；
    - INSERT， DELETE， UPDATE， SELECT
3. DCL:
    - 权限管理命令: GRANT，REVOKE
4. 获取命令帮助:
```
mysql> help keyword
mysql> help create table
mysql> help create database
mysql> help drop index
```

### 2.1 mysql 状态查询
|作用|sql语句|
|:---|:---|
|-可用配置查询-|----------|
|查看支持的所有字符集|`SHOW CHARACTER SET `|
|查看支持的所有排序规则|`SHOW  COLLATION`|
|查看数据库支持的所有存储引擎类型|`SHOW  ENGINES;`|
|-表状态信息-|------------|
|查看表状态|`SHOW  TABLES  STATUS  [LIKE  'tbl_name']\G`|
|查看表上的索引的信息|`SHOW INDEXES FROM tbl_name;`|
|查看表结构|`desc tbl_name`;|
|查看表创建命令|`show create table tbl_name;`|
|查看指定用户所获得的授权|`SHOW GRANTS FOR  'user'@'host'`|
|查看指定用户所获得的授权|`SHOW GRANTS FOR CURRENT_USER;`|


## 3. DCL 用户账号及权限管理：
### 3.1 用户账号：'username'@'host'
- host：此用户访问当前mysql服务器时，允许其通过哪些主机远程创建连接；
- 表示方式：IP，网络地址、主机名、通配符(%和_)；

```
# 配置禁止检查主机名：my.cnf
[mysqld]
skip_name_resolve = ON
```

### 3.2 账号管理
1. 创建用户账号：`CREATE  USER  'username'@'host'  [IDENTIFIED BY  'password'];`
2. 删除用户账号：`DROP USER  ’user‘@’host' [, user@host] ...
```
mysql
mysql> create user 'wpuser'@'%' identified by 'wppass';
mysql> select * from mysql.user;
```

### 3.3 授权：
`GRANT  priv_type,...  ON  [object_type]  db_name.tbl_name  TO  'user'@'host'  [IDENTIFIED BY  'password'];`
- `priv_type`： ALL  [PRIVILEGES]
- `db_name.tbl_name`：
    - *.*：所有库的所有表；
    - db_name.*：指定库的所有表；
    - db_name.tbl_name：指定库的特定表；
    - db_name.routine_name：指定库上的存储过程或存储函数
- `[object_type]`
    - TABLE
    - FUNCTION
    - PROCEDURE  
```
mysql
mysqsl> grant select,delete on testdb.* to 'test'@'%' identified by 'testpass'
mysql> revoke delete on testdb.* from 'test'@'%';
```

### 3.4 回收权限：
`REVOKE  priv_type, ...  ON  db_name.tbl_name  FROM  'user'@'host';`
- 注意：MariaDB服务进程启动时，会读取mysql库的所有授权表至内存中；
- GRANT或REVOKE命令等执行的权限操作会保存于表中，MariaDB此时一般会自动重读授权表，权限修改会立即生效；
- 其它方式实现的权限修改，要想生效，必须手动运行`FLUSH PRIVILEGES`命令方可；
