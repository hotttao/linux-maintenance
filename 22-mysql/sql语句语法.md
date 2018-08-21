## 3. DDL
### 3.1 数据库管理：
```
# 1. 创建：
CREATE  {DATABASE | SCHEMA}  [IF NOT EXISTS]  db_name;
    - [DEFAULT]  CHARACTER SET [=] charset_name
    - [DEFAULT]  COLLATE [=] collation_name              

# 2. 修改：
ALTER {DATABASE | SCHEMA}  [db_name]
    - [DEFAULT]  CHARACTER SET [=] charset_name
    - [DEFAULT]  COLLATE [=] collation_name

# 3. 删除：
DROP {DATABASE | SCHEMA} [IF EXISTS] db_name

# 4. 查看：
SHOW DATABASES LIKE  ’‘;
```

### 3.2 表管理：
`help create table`
1. 创建：
    - CREATE TABLE  [IF NOT EXISTS]  tbl_name  (create_defination)  [table_options]
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
    - ALTER [ONLINE | OFFLINE] [IGNORE] TABLE tbl_name  [alter_specification [, alter_specification] ...]
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
3. 删除： DROP  TABLE  [IF EXISTS]  tbl_name [, tbl_name] ...


**表创建的其他方式**
1. 复制表结构；
2. 复制表数据；

### 3.3 索引管理：
- 特性:
    - 索引是特殊的数据结构；
    - 定义在查找时作为查找条件的字段上
    - 索引有索引名称；
- 创建：
    - `CREATE  [UNIQUE|FULLTEXT|SPATIAL] INDEX  index_name  [BTREE|HASH]  ON tbl_name (col1, col2,,...)`
- 删除：`DROP  INDEX index_name ON tbl_name`

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
