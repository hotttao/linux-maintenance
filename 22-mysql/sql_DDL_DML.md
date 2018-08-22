## 3. SQL DDL 与 DML
SQL 是关系型数据库专用的结构化查询语言，用来管理和查询关系型数据库中的数据。本节我们就来学习基础的 SQL 语言。

## 1. SQL 的分类
```
MariaDB [(none)]> help contents
You asked for help about help category: "Contents"
For more information, type 'help <item>', where <item> is one of the following
categories:
   Account Management
   Administration
   Compound Statements
   Data Definition
   Data Manipulation
   Data Types
   Functions
   Functions and Modifiers for Use with GROUP BY
   Geographic Features
   Help Metadata
   Language Structure
   Plugins
   Procedures
   Table Maintenance
   Transactions
   User-Defined Functions
   Utility
```

在 mysql 客户端内使用 `help contents` 可以查看到 SQL 语句的所有类别，最常用的是如下三类:
1. `DDL`：`Data Defined Language`
    - 作用: 数据定义语言，主要用于管理数据库组件，例如表、索引、视图、用户、存储过程
    - 命令: `CREATE`、`ALTER`、`DROP`
2. `DML`：`Data Manapulating Language`
    - 作用: 数据操纵语言，主要用管理表中的数据，实现数据的增、删、改、查；
    - 命令: `INSERT`， `DELETE`， `UPDATE`， `SELECT`
3. `DCL`:
  - 作用: 权限管理命令
  - 命令: `GRANT`，`REVOKE`

## 2 MariaDB 中的数据类型
MariaDB 在存储数据之前，我们首先需要创建表，创建表的核心就是定义字段和取定表使用的字符集，而定义字段，关键的一步即为确定其数据类型。

数据类型用于确定数据存储格式、能参与运算种类、可表示的有效的数据范围。字符集就是码表，在字符和二进制数字之间建立映射关系，对于非英语系的国家字符集的设置至关重要。mysql 默认的字符集是 `latin1`，UTF8 在 mysql 中是 `utf8mb4` 而不是 utf8

```
show character set   # 查看 mysql 支持的字符集
show collation       # 查看字符集支持的排序方法
```

### 2.1 数据类型
MariaDB 常见的数据类型如下所示:
1. 字符型：
    - 定长字符型：
        - `CHAR(#)`：不区分字符大小写
        - `BINARY(#)`：区分字符大小写
    - 变长字符型：
        - `VARCHAR(#)`：不区分字符大小写
        - `VARBINARY(#)`：区分字符大小写
    - 对象存储：
        - `TEXT`：不区分字符大小写
        - `BLOB`：区分字符大小写
    - 内置类型：
        - `SET`
        - `ENUM`
2. 数值型：
    - 精确数值型：
        - `INT（TINYINT，SMALLINT，MEDIUMINT，INT，BIGINT)`
        - `DECIMAL`: 十进制数
    - 近似数值型：
        - `FLOAT`
        - `DOBULE`
3. 日期时间型：
    - 日期型：`DATE`
    - 时间型：`TIME`
    - 日期时间型：`DATETIME`
    - 时间戳：`TIMESTAMP`
    - 年份：`YEAR(2)`, YEAR(4)`

### 2.3 数据类型的修饰符
MariaDB 的数据类型还有修饰符的概念，用于限定字段属性，常见的修饰符如下所示:
1. 所有类型修饰符：
    - `NOT NULL`：非空；
    - `DEFAULT  value`：默认值；
    - `primary key`
    - `unique key`
2. 整型修饰符:
    - `UNSIGNED`：无符号
    - `AUTO_INCREMENT`: 自增

## 3. DDL
### 3.1 数据库管理
```
# 1. 创建：
CREATE  {DATABASE | SCHEMA}  [IF NOT EXISTS]  db_name;
        [DEFAULT]  CHARACTER SET [=] charset_name
        [DEFAULT]  COLLATE [=] collation_name              

# 2. 修改：
ALTER {DATABASE | SCHEMA}  [db_name]
      [DEFAULT]  CHARACTER SET [=] charset_name
      [DEFAULT]  COLLATE [=] collation_name

# 3. 删除：
DROP {DATABASE | SCHEMA} [IF EXISTS] db_name

# 4. 查看：
SHOW DATABASES LIKE  ’‘;
```

### 3.2 表管理：
`help create table`
1. 创建：
    - `CREATE TABLE  [IF NOT EXISTS]  tbl_name  (create_defination)  [table_options]`
    - create_defination:
        - 字段：col_name  data_type
        - 键：
            - PRIMARY KEY (col1, col2, ...)
            - UNIQUE KEY  (col1, col2,...)
            - FOREIGN KEY (column)
        - 索引：KEY|INDEX  [index_name]  (col1, col2,...)      
    - table_options：
        - ENGINE [=] engine_name
2. 修改：
    - `ALTER [ONLINE | OFFLINE] [IGNORE] TABLE tbl_name  [alter_specification [, alter_specification] ...]`
    - alter_specification:
        - 字段：
            - 添加：ADD  [COLUMN]  col_name  data_type  [FIRST | AFTER col_name ]
            - 删除：DROP  [COLUMN] col_name
            - 修改：
                - CHANGE [COLUMN] old_col_name new_col_name column_definition  [FIRST|AFTER col_name]  -- 更改字段名称
                - MODIFY [COLUMN] col_name column_definition  [FIRST | AFTER col_name]  -- 更改字段属性定义
                - ALTER [COLUMN] col_name [SETDEFAULT literal | DROP DEFAULT] -- 更改字段默认值
        - 键：
            - 添加：ADD  {PRIMARY|UNIQUE|FOREIGN}  KEY (col1, col2,...)
            - 删除：
                - 主键：DROP PRIMARY KEY
                - 外键：DROP FOREIGN KEY fk_symbol
        - 索引：
            - 添加：ADD {INDEX|KEY} [index_name]  (col1, col2,...)
            - 删除：DROP {INDEX|KEY}  index_name  
        - 表选项：ENGINE [=] engine_name  
3. 删除： `DROP  TABLE  [IF EXISTS]  tbl_name [, tbl_name] ...`


**表创建的其他方式**
```
# 1. 复制表结构；
CREATE table  like table_name;

# 2. 复制表数据；
CREATE table  select_sql;
```

### 3.3 索引管理：
索引是特殊的数据结构,定义在查找时作为查找条件的字段上，实现快速查找

#### 创建
```
CREATE  [UNIQUE|FULLTEXT|SPATIAL] INDEX  index_name  
        [BTREE|HASH]  ON tbl_name (col1, col2,,...)`
```

#### 删除
`DROP  INDEX index_name ON tbl_name`

## 4. DML：
INSERT， DELETE， UPDATE， SELECT

### 4.1 INSERT
- `INSERT  [INTO]  tbl_name  [(col1,...)]  {VALUES|VALUE}  (val1, ...),(...),...`
- 注意：
    - 字符型：引号；
    - 数值型：不能用引号；

### 4.2 SELECT：
1. `SELECT  *  FROM  tbl_name;`
2. `SELECT  col1, col2, ...  FROM  tbl_name;`
    - 显示时，字段可以显示为别名: col_name  AS  col_alias
3. `SELECT  col1, ...  FROM tbl_name  WHERE clause; `
    - WHERE clause：用于指明挑选条件；
    - col_name 操作符 value：
        - age > 30;
        - `>, <, >=, <=, ==, !=`
        - 组合条件：and  or  not
        - BETWEEN ...  AND ...
        - LIKE 'PATTERN'
            - `%`：任意长度的任意字符；
            - `_`：任意单个字符；
        - **RLIKE**  'PATTERN'
            - 正则表达式对字符串做模式匹配；
        - IS NULL
        - IS NOT NULL
4. `SELECT col1, ... FROM tbl_name  [WHERE clause]  ORDER BY  col_name, col_name2, ...  [ASC|DESC];`
    - ASC: 升序；
    - DESC： 降序；

### 4.3 DELETE：
- `DELETE  FROM  tbl_name  [WHERE where_condition]  [ORDER BY ...]  [LIMIT row_count]`
- `DELETE  FROM  tbl_name  WHERE where_condition`
- `DELETE  FROM  tbl_name  [ORDER BY ...]  [LIMIT row_count]`

### 4.4 UPDATE：
- `UPDATE [LOW_PRIORITY] [IGNORE] table_reference  SET col_name1=value1 [, col_name2=value2] ... [WHERE where_condition]  [ORDER BY ...] [LIMIT row_count]`
