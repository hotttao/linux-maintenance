# 22.4 MariaDB 权限管理
本节我们来介绍 MariaDB 中的权限管理

### 1. mysql 状态查询
前面我们学习了 DDL 与 DML，下面是 mysql 一些常用的状态查询语句，用于查看mysql 的各种状态和变量。

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


## 2. DCL 用户账号及权限管理：
### 2.1 用户账号
mysql的用户账号由两部分组成：`'USERNAME'@'HOST'`
- `USER`: 表示用户名称
- `HOST`: 用于限制此用户可通过哪些远程主机连接当前的mysql服务.

HOST的表示方式，支持使用通配符：
- `%`：匹配任意长度的任意字符；
- `172.16.%.%` == `172.16.0.0/16`
- `_`：匹配任意单个字符；

默认情况下 mysql 登陆时会对客户端的 IP 地址进行反解，这种反解一是浪费时间可能导致阻塞，二是如果反解成功而 mysql 在授权时只授权了 IP 地址而没有授权主机名，依旧无法登陆，所以在配置 mysql 时都要关闭名称反解功能。

```
vim  /etc/mysql/my.cnf  # 添加三个选项：
[mysqld]
datadir = /mydata/data
innodb_file_per_table = ON
skip_name_resolve = ON  # 配置禁止检查主机名
```

### 2.2 账号管理
1. 创建用户账号：`CREATE  USER  'username'@'host'  [IDENTIFIED BY  'password'];`
2. 删除用户账号：`DROP USER  ’user‘@’host' [, user@host] ...`

```
mysql
mysql> create user 'wpuser'@'%' identified by 'wppass';
mysql> select * from mysql.user;
```

### 2.3 授权
`GRANT  priv_type,...  ON  [object_type]  db_name.tbl_name  TO  'user'@'host'  [IDENTIFIED BY  'password'];`
- `priv_type`： 要授权的操作
    - `ALL`: 所有操作
- `db_name.tbl_name`： 授权的范围
    - `*.*`：所有库的所有表；
    - `db_name.*`：指定库的所有表；
    - `db_name.tbl_name`：指定库的特定表；
    - `db_name.routine_name`：指定库上的存储过程或存储函数
- `[object_type]`: 授权可操作额对象
    - TABLE，默认
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
