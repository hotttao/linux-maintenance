# 22.1 关系型数据库基础
mariadb(mysql) 是最常用的开源关系型数据库，本节我们就对关系型数据库和 mysql 做一个简单介绍。

## 1. 关系型数据库
### 1.1 数据模型
数据模型有，层次模型、网状模型、关系模型。关系模型是二维关系表现为表中的列和行。数据库管理系统称为 DBMS(DataBase Management System)，关系型数据库管理系统则称为 RDBMS(Relational  DataBase Management System)，常见的 RDMBS:
- Mysql/MariaDB/Percona-Server
- PostgreSQL
- Oracle

### 1.2 事务(Transaction)
事务含义是将多个操作为一个整体，要么全部都执行，要么全部都不执行；其遵循 ACID：
- A：Atomicity,原子性；
- C：Consistency,一致性；
- I：Isolation,隔离性；
- D：Durability,持久性；

## 2. RDMBS设计范式
设计关系数据库时，遵从不同的规范要求，设计出合理的关系型数据库，这些不同的规范要求被称为不同的范式，各种范式呈递次规范，越高的范式数据库冗余越小。目前关系数据库有六种范式：
- 第一范式（1NF）: **每一列都是原子，不可在分割的**
- 第二范式（2NF）: **每一行都可以使用其有限字段进行唯一标志(不存在重复的行)**
- 第三范式（3NF）、巴德斯科范式（BCNF）: **任何表都不应该有依赖于其他表的非主键字段**
- 第四范式(4NF）和
- 第五范式（5NF，又称完美范式）

满足最低要求的范式是第一范式（1NF）。在第一范式的基础上进一步满足更多规范要求的称为第二范式（2NF），其余范式以次类推。一般说来，数据库只需满足第三范式(3NF）就行了。


## 3. mysql 简介
自从 mysql 被 Oracle 收购之后，由于担心版权问题，mysql 的创始人就新建了另一开源分支 mariadb，在 Centos6 中默认安装的是 mysql，而在 Centos7 中默认安装的已经是  mariadb。mariadb 跟 mysql 底层的基础特性是类似的，但是高级特性有很大不同，彼此支持的高级功能也不相同。除了 mariadb，mysql还有很多二次发行版本，比如`Percona`，`AllSQL`(阿里的mysql 发行版)以及，`TIDB`

mysql 与 mariadb 的官网分别是：
- www.mysql.com
- MariaDB: www.mariadb.org

### 3.1 mariadb 特性
MariaDB的 支持插件式存储引擎，即存储管理器有多种实现版本，彼此间的功能和特性可能略有区别；用户可根据需要灵活选择。存储引擎也称为“表类型”。常见的存储引擎就是
1. `MyISAM`:不支持事务和表级锁，奔溃后不保证安全恢复；
2. `InnoDB`: 支持事务，行级锁，外键和热备份；

`MyISAM` 在 mariadb 中被扩展为 `Aria`，支持安全恢复, `InnoDB` 在 Mariadb 中的开源实现为 `XtraDB`。在 mysql 的客户端中输入 `show engines` 即可查看 mariadb 支持的所有存储引擎。

```
MariaDB [(none)]> show engines;
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                                    | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
| CSV                | YES     | CSV storage engine                                                         | NO           | NO   | NO         |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                                      | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables                  | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears)             | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                                      | NO           | NO   | NO         |
| InnoDB             | DEFAULT | Percona-XtraDB, Supports transactions, row-level locking, and foreign keys | YES          | YES  | YES        |
| ARCHIVE            | YES     | Archive storage engine                                                     | NO           | NO   | NO         |
| FEDERATED          | YES     | FederatedX pluggable storage engine                                        | YES          | NO   | YES        |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                                         | NO           | NO   | NO         |
| Aria               | YES     | Crash-safe tables with MyISAM heritage                                     | NO           | NO   | NO         |
+--------------------+---------+----------------------------------------------------------------------------+--------------+------+------------+
```

### 3.2 MariaDB程序的组成
mariadb 是 C/S 架构的服务，其命令分为服务器端和客户端两个部分
- C：Client
    - mysql：CLI交互式客户端程序；
    - mysqldump：备份工具；
    - mysqladmin：管理工具；
    - mysqlbinlog：
    - ...
- S：Server
    - mysqld
    - mysqld_safe：建议运行的服务端程序；
    - mysqld_multi：多实例；

msyql 服务器可监听在两种套接字上
1. IPV4/6 的 tcp 的 3306 端口上，支持远程通信
2. Unix Sock，监听在 socket 文件上，仅支持本地通信，套接子文件通常位于 `/var/lib/mysql/mysql.sock`或 `/tmp/mysql.sock` 由配置文件指定。

```
ll /var/lib/mysql/mysql.sock
srwxrwxrwx. 1 mysql mysql 0 8月  21 11:10 /var/lib/mysql/mysql.sock
```

### 3.3 mysql 客户端启动命令
`mysql [OPTIONS] [database]`
- 常用选项：
    - `-u, --user=name`：用户名，默认为root；
    - `-h, --host=name`：远程主机（即mysql服务器）地址，默认为localhost;
    - `-p, --password`：USERNAME所表示的用户的密码； 默认为空；
    - `-P, --port`: 指定 mysql 服务监听的端口，默认为 3306
    - `-D, --database`：连接到服务器端之后，设定其处指明的数据库为默认数据库；
    - `-e, --execute='SQL COMMAND;'`：连接至服务器并让其执行此命令后直接返回；
    - `-S, --socket`: 指定本地通信的套接字路经

mysql 客户端内可输入的命令分为两类:
1. 客户段命令: 只在客户端运行的命令，使用 `help` 可获取此类命令的帮助
2. 服务段命令: 通过 mysql 的协议送到服务段运行的命令，所以必须要有命令结束符,默认为 `;`；使用 `help contents` 获取服务器端命令使用帮助。

#### 查看本地命令
mysql> help
- `\u db_name`：设定哪个库为默认数据库
- `\q`：退出
- `\d CHAR`：设定新的语句结束符，默认为 `;`
- `\g`：语句结束标记，默认就相当于 `;` 作用
- `\G`：语句结束标记，结果竖排方式显式
- `\! COMMAND`: 在客户端内运行 shell 命令
- `\. PATH`: 在客户端内执行 sql 脚本(包含 sql 的文本)

```
$ mysql -uroot -p1234

MariaDB [(none)]> help              # help 查看 mysql 的所有命令

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.

For server side help, type 'help contents'

# 执行 shell 命令
MariaDB [(none)]> \! ls /var
account  cache	db     games   iso	 lib	lock  mail   nis  preserve  spool   tmp  yp
adm	 crash	empty  gopher  kerberos  local	log   named  opt  run	    target  www
```

#### 查看服务端命令
```
MariaDB [(none)]> help contents   # 查看 mysql 命令的组成部分
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
.........


MariaDB [(none)]> help 'Account Management'   # 查看特定命令组内的命令
topics:
   CREATE USER
   DROP USER
   GRANT
   RENAME USER
   REVOKE
   SET PASSWORD

MariaDB [(none)]> help 'CREATE USER'   # 查看特定命令使用帮助
Name: 'CREATE USER'
Description:
Syntax:
CREATE USER user_specification
    [, user_specification] ...

user_specification:
    user
    [
        IDENTIFIED BY [PASSWORD] 'password'
      | IDENTIFIED WITH auth_plugin [AS 'auth_string']
    ]
...............
```


### 3.4 mysql 数据库组件
mysql 数据库包括如下组件:
1. 数据库: database
2. 表: table:
    - 行: row
    - 列: column
3. 索引: index
4. 视图: view
5. 用户: user
6. 权限: privilege
7. 存储过程: procedure
8. 存储函数: function
9. 触发器: trigger
10. 事件调度器: event scheduler
