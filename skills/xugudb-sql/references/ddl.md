---
title: 虚谷数据库 DDL 参考
description: 虚谷数据库数据定义语言（DDL）完整参考，涵盖库、模式、表、索引、视图、序列值的创建、修改与删除操作
tags: [xugudb, ddl, create, alter, drop, table, index, view, sequence, schema, database, partition]
---

# 虚谷数据库 DDL 参考

## 一、库管理（DATABASE）

### 1.1 创建数据库

#### 语法格式

```sql
CREATE DATABASE [IF NOT EXISTS] database_name
  [CHARACTER SET (charset_name | 'charset_name')]
  [TIME ZONE 'GMT+HH:MM']
  [ENCRYPT BY 'encryptor_name'];
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `IF NOT EXISTS` | 同名数据库已存在时忽略错误，不影响已有库 |
| `database_name` | 数据库名称 |
| `CHARACTER SET` | 字符集名称，系统库固定为 UTF8，用户库缺省为 GBK（可通过 `def_charset` 参数修改） |
| `TIME ZONE` | 时区，格式 `'GMT+HH:MM'` 或 `'GMT-HH:MM'`，默认 `GMT+08:00`，取值范围 `[-12:59, +14:59]` |
| `ENCRYPT BY` | 数据库加密机名称 |

> **注意：** 数据库的创建、修改与删除必须在系统库（SYSTEM）中进行，需使用 SYSDBA 用户或具有 SYSDBA 权限的用户操作。可通过 `SYS_DATABASES` 表查看所有库信息。

#### 示例

```sql
-- 创建指定字符集和时区的数据库
CREATE DATABASE IF NOT EXISTS mydb CHARACTER SET UTF8 TIME ZONE 'GMT+08:00';
```

---

## 二、模式管理（SCHEMA）

### 2.1 创建模式

#### 语法格式

```sql
CREATE SCHEMA schema_name [AUTHORIZATION user_name];
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `schema_name` | 新建模式名，不能与已存在用户或角色同名 |
| `AUTHORIZATION user_name` | 将模式属主授予指定用户，不指定时默认属主为当前用户 |

> **提示：** 创建用户时会自动创建与用户同名的模式。

#### 示例

```sql
-- 创建模式并指定属主
CREATE SCHEMA sch_test AUTHORIZATION GUEST;
```

### 2.2 切换模式

```sql
SET [CURRENT] SCHEMA (schema_name | DEFAULT);
```

- `DEFAULT`：切换回当前会话登录用户的默认同名模式。

### 2.3 修改模式

#### 语法格式

```sql
ALTER SCHEMA schema_name (RENAME TO new_name | OWNER TO user_name);
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `RENAME TO new_name` | 重命名模式（仅用户创建的非同名模式可修改） |
| `OWNER TO user_name` | 更改模式属主 |

#### 示例

```sql
ALTER SCHEMA sch_test RENAME TO sch_test_new;
ALTER SCHEMA sch_test_new OWNER TO usr_test;
```

### 2.4 删除模式

#### 语法格式

```sql
DROP SCHEMA schema_name [RESTRICT | CASCADE [CONSTRAINTS]];
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `RESTRICT` | 默认行为，若模式下有被依赖的对象则报错 |
| `CASCADE` | 强制删除模式，受依赖关系影响的对象将置为无效状态 |

> **提示：** 与用户同名的默认模式不允许单独删除，只能与用户一同删除。

#### 示例

```sql
DROP SCHEMA sch_test_new;
DROP SCHEMA sch_test_new CASCADE;
```

---

## 三、表管理（TABLE）

### 3.1 创建表（CREATE TABLE）

#### 语法格式

```sql
CREATE [LOCAL | GLOBAL] [TEMPORARY | TEMP] TABLE [IF NOT EXISTS] table_name (
  table_elements
)
[ON COMMIT (DELETE | PRESERVE) ROWS]
[ORGANIZATION (HEAP | EXTERNAL (...))]
[PARTITION BY ...]
[SUBPARTITION BY ...]
[constraint_props]
[store_props]
[PCTFREE n]
[COMMENT 'comment_string']
[ENCRYPT BY encryptor_name];
```

#### 列定义语法

```sql
column_name data_type [IDENTITY(init_value, step_value) | AUTO_INCREMENT]
  [DEFAULT expr [ON NULL [FOR INSERT ONLY | FOR INSERT AND UPDATE]]]
  [NOT NULL]
  [PRIMARY KEY]
  [UNIQUE]
  [CHECK (condition)]
  [REFERENCES table(column)]
  [COMMENT 'comment_string']
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `IF NOT EXISTS` | 表已存在时不执行操作，仅发出警告 |
| `TEMPORARY / TEMP` | 创建局部临时表（结构与数据仅在当前会话存在） |
| `GLOBAL TEMPORARY` | 创建全局临时表（结构持久化，数据会话隔离，需启用 `support_global_tab`） |
| `ON COMMIT DELETE ROWS` | 事务提交时删除临时表中所有行 |
| `ON COMMIT PRESERVE ROWS` | 事务提交时保留临时表中的行（默认） |
| `IDENTITY(init, step)` | 自增序列，init 为初始值，step 为步长 |
| `AUTO_INCREMENT` | 等同于 `IDENTITY(1,1)` |
| `ENCRYPT BY` | 指定数据存储加密的加密机名称 |
| `PCTFREE n` | 存储参数，预留空闲空间百分比 |
| `COMMENT` | 表或列的备注信息 |

**临时表限制：**
- 不支持分区
- 不支持外键约束
- PL/SQL 中临时表不支持任何约束

#### 示例

```sql
-- 创建带自增列、约束、分区的完整表
CREATE TABLE IF NOT EXISTS sch_hr.tab_employees (
  employee_id   INT IDENTITY(1,1) PRIMARY KEY,
  employee_name VARCHAR(50) NOT NULL
    CONSTRAINT ck_name CHECK (LENGTH(employee_name) >= 2),
  email         VARCHAR(100) DEFAULT 'someone@example.com',
  hire_date     DATE NOT NULL DEFAULT SYSDATE,
  salary        NUMERIC(10,2) CHECK (salary > 0),
  department_id NUMERIC(4,0)
    CONSTRAINT fk_dept REFERENCES sch_hr.tab_departments(department_id),
  CONSTRAINT uk_email UNIQUE (email),
  CONSTRAINT ck_salary CHECK (salary <= 100000)
)
PARTITION BY RANGE (salary) PARTITIONS (
  part1 VALUES LESS THAN (5000),
  part2 VALUES LESS THAN (10000),
  part3 VALUES LESS THAN (MAXVALUES)
)
SUBPARTITION BY HASH (employee_id) SUBPARTITIONS 2
PCTFREE 10
COMMENT '雇员信息表';

-- 创建局部临时表
CREATE TEMP TABLE IF NOT EXISTS temp_data (
  id INT IDENTITY(1,1) PRIMARY KEY,
  name VARCHAR(50) NOT NULL
) ON COMMIT PRESERVE ROWS;

-- 创建全局临时表（需先 SET support_global_tab ON）
CREATE GLOBAL TEMP TABLE IF NOT EXISTS global_data (
  id INT IDENTITY(1,1) PRIMARY KEY,
  name VARCHAR(50)
);

-- 自增序列：初始值 5，步长 2
CREATE TABLE tab_serial (id INT IDENTITY(5,2));

-- 自增序列：初始值 0，步长 -1（递减）
CREATE TABLE tab_serial_dec (
  id INT IDENTITY(0,-1) PRIMARY KEY,
  name VARCHAR(50)
);

-- 外部文件表
CREATE TABLE tab_ext (c1 VARCHAR, c2 VARCHAR)
ORGANIZATION EXTERNAL (
  TYPE TXT
  DEFAULT DIR '/data/files'
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY '\n'
    FIELDS TERMINATED BY ','
  )
  LOCATION '/data/files/data.txt'
);
```

### 3.2 修改表（ALTER TABLE）

#### 语法格式

```sql
ALTER TABLE table_name
  -- 列操作
  ADD [COLUMN] (column_definitions)
  | ALTER|MODIFY COLUMN (column_modifications)
  | DROP COLUMN column_name [, column_name ...]
  -- 分区操作
  | ADD PARTITION parti_name VALUES [LESS THAN] (parti_values)
  | DROP PARTITION parti_name [REBUILD GLOBAL INDEX]
  | TRUNCATE PARTITION parti_name [REBUILD GLOBAL INDEX]
  | SET PARTITION parti_name (ONLINE | OFFLINE)
  -- 约束操作
  | (ENABLE | DISABLE) CONSTRAINT cons_name [, ...] [KEEP|DROP INDEX] [CASCADE CONSTRAINTS]
  | DROP CONSTRAINT cons_name [, ...] [KEEP|DROP INDEX] [CASCADE CONSTRAINTS]
  -- DML 权限控制
  | (ENABLE | DISABLE) (SELECT|INSERT|UPDATE|DELETE|...) [, ...]
  -- 其他操作
  | RENAME TO new_name
  | OWNER TO username
  | REBUILD HEAP
  | REOPEN
  | SET (ONLINE | OFFLINE)
  | SET CACHE (BY (column_list) | OFF)
  [NOWAIT | WAIT [wait_ms]];
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `ADD [COLUMN]` | 添加新列 |
| `ALTER/MODIFY COLUMN` | 修改列属性（类型、默认值、NULL 约束等） |
| `DROP COLUMN` | 删除列 |
| `ADD PARTITION` | 添加范围/列表分区 |
| `DROP PARTITION` | 删除分区，可选 `REBUILD GLOBAL INDEX` |
| `TRUNCATE PARTITION` | 截断分区数据，可选 `REBUILD GLOBAL INDEX` |
| `REBUILD HEAP` | 重整表，回收碎片存储空间 |
| `REOPEN` | 重开表，更新内存中表的元信息 |
| `OWNER TO` | 更改表属主 |
| `SET CACHE BY (cols)` | 启用表数据缓存 |
| `NOWAIT` | 无法立即获锁则报错 |
| `WAIT wait_ms` | 最多等待指定毫秒（范围 `[0, 2147483647]`，超过 300000 强制使用 300000） |

**修改列支持的操作：**
- `SET DEFAULT expr`：设置默认值
- `SET DEFAULT expr ON NULL`：当值为 NULL 时填充默认值
- `DROP DEFAULT`：删除默认值
- `SET NOT NULL / SET NULL`：设置或取消 NOT NULL 约束

> **警告：** 修改表可能触发自动重整表（如新增带默认值的列、CHAR 精度变更等），对大数据量表耗时极长。碎片率超过 50% 时建议执行 `REBUILD HEAP`。

#### 示例

```sql
-- 添加列并修改现有列
ALTER TABLE tab_test ADD COLUMN c3 DATETIME
  ALTER COLUMN c2 SET NOT NULL;

-- 设置列默认值并删除列
ALTER TABLE tab_test ALTER COLUMN c1 DEFAULT 1024
  DROP COLUMN c2;

-- 添加范围分区
ALTER TABLE tab_data ADD PARTITION part4 VALUES LESS THAN ('2030-01-01 00:00:00');

-- 删除分区并重建全局索引
ALTER TABLE tab_data DROP PARTITION part3 REBUILD GLOBAL INDEX;

-- 添加/删除约束
ALTER TABLE tab_test ADD CONSTRAINT uk UNIQUE (c1);
ALTER TABLE tab_test DROP CONSTRAINT uk;

-- 禁用表的插入操作
ALTER TABLE tab_test DISABLE INSERT;

-- 重命名表
ALTER TABLE tab_old RENAME TO tab_new;

-- 更改属主
ALTER TABLE tab_test OWNER TO guest;

-- 重整表
ALTER TABLE tab_test REBUILD HEAP;

-- 重开表（带超时）
ALTER TABLE tab_test REOPEN WAIT 5000;
```

### 3.3 删除表（DROP TABLE）

#### 语法格式

```sql
DROP TABLE [IF EXISTS] table_name [RESTRICT | CASCADE [CONSTRAINTS]] [PURGE];
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `IF EXISTS` | 表不存在时仅发出警告（E5021），不报错 |
| `RESTRICT` | 默认行为，若存在依赖项则报错 |
| `CASCADE [CONSTRAINTS]` | 级联删除/禁用依赖项（依赖表不会被删除，仅删除被依赖表） |
| `PURGE` | 是否放入回收站 |

#### 示例

```sql
-- 预检测删除
DROP TABLE IF EXISTS tab_test;

-- 级联删除（依赖的存储函数将置为无效）
DROP TABLE tab_city CASCADE;
```

### 3.4 截断表（TRUNCATE TABLE）

#### 语法格式

```sql
TRUNCATE [TABLE] table_name [NOWAIT | WAIT [wait_ms]];
```

通过清空表上首存储并切断后续存储链，实现数据快速清空。不同于 DELETE，TRUNCATE 速度更快且不逐行记录日志。

#### 示例

```sql
TRUNCATE TABLE tab_data;
TRUNCATE tab_data NOWAIT;
```

### 3.5 分区表

虚谷数据库支持两级三种分区方式，适用于单表超过 1 亿行的大数据量场景。

#### 分区类型

| 分区类型 | 说明 |
|----------|------|
| **范围分区**（RANGE） | 按数据范围分割，适合连续值（如时间、金额） |
| **列表分区**（LIST） | 按离散值分割，适合有限取值（如地区、状态） |
| **哈希分区**（HASH） | 按哈希值平均分布数据 |

#### 一级分区语法

```sql
-- 范围分区
PARTITION BY RANGE (column [, column ...])
  [INTERVAL n (YEAR|MONTH|DAY|HOUR)]     -- 自动扩展分区
  PARTITIONS (
    part1 VALUES LESS THAN (value),
    part2 VALUES LESS THAN (MAXVALUES)
  )

-- 列表分区
PARTITION BY LIST (column [, column ...])
  PARTITIONS (
    part1 VALUES (value1),
    part2 VALUES (OTHERVALUES)
  )

-- 哈希分区
PARTITION BY HASH (column [, column ...])
  PARTITIONS n                            -- 指定分区数
```

#### 二级分区语法

```sql
SUBPARTITION BY (HASH|LIST|RANGE) (column [, column ...])
  SUBPARTITIONS (...)
```

> **注意：**
> - 二级分区对所有一级分区统一生效，不能对不同一级分区设置不同二级分区规则。
> - 一级分区为哈希分区时，不能创建二级分区。
> - 当 `MAXVALUES` 或 `OTHERVALUES` 分区存在时，不允许再创建新分区。
> - 自动扩展分区仅支持一级分区，且分区键必须为时间类型。

#### 按分区访问数据

```sql
-- 访问一级分区
SELECT * FROM table_name PARTITION (part1, part2);

-- 访问二级分区
SELECT * FROM table_name SUBPARTITION (subpart1, subpart2);
```

#### 示例

```sql
-- 范围分区表
CREATE TABLE tab_range (
  id INTEGER IDENTITY(1,2),
  name VARCHAR,
  city VARCHAR
) PARTITION BY RANGE (id) PARTITIONS (
  part1 VALUES LESS THAN (1),
  part2 VALUES LESS THAN (1000),
  part3 VALUES LESS THAN (MAXVALUES)
);

-- 列表分区表
CREATE TABLE tab_list (
  id INT, name VARCHAR(20), city CHAR(20)
) PARTITION BY LIST (city) PARTITIONS (
  ('四川'), ('云南'), ('贵州'), (OTHERVALUES)
);

-- 哈希分区表
CREATE TABLE tab_hash (
  id INTEGER IDENTITY(1,2),
  name VARCHAR
) PARTITION BY HASH (id) PARTITIONS 5;

-- 自动扩展分区（按 5 天间隔自动创建新分区）
CREATE TABLE tab_auto_parti (
  id INT,
  create_time DATETIME
) PARTITION BY RANGE (create_time)
  INTERVAL 5 DAY
  PARTITIONS (('1970-01-01 00:00:00'));

-- 两级分区：一级列表 + 二级范围
CREATE TABLE tab_two_level (
  id INT, name VARCHAR, city VARCHAR, addr VARCHAR
) PARTITION BY LIST (city) PARTITIONS (
  part1 VALUES ('成都'),
  part2 VALUES ('北京'),
  part3 VALUES (OTHERVALUES)
) SUBPARTITION BY RANGE (id) SUBPARTITIONS (
  subpart1 VALUES LESS THAN (1),
  subpart2 VALUES LESS THAN (10),
  subpart3 VALUES LESS THAN (MAXVALUES)
);

-- 查询指定分区的数据
SELECT * FROM tab_list PARTITION (part1) ORDER BY id;
```

### 3.6 锁表（LOCK TABLE）

#### 语法格式

```sql
LOCK [TABLE] table_name [, table_name ...]
  [IN (SHARE MODE | EXCLUSIVE MODE | ROW SHARE MODE | ROW EXCLUSIVE MODE)]
  [NOWAIT | WAIT [wait_ms]];
```

#### 锁模式

| 锁模式 | 说明 |
|--------|------|
| `SHARE MODE` | 共享锁（S），允许多事务同时读取，不允许写 |
| `EXCLUSIVE MODE` | 排他锁（X），阻止其他事务的任何操作 |
| `ROW SHARE MODE` | 意向共享锁（IS），表示将在子资源上加 S 锁 |
| `ROW EXCLUSIVE MODE` | 意向排他锁（IX），表示将在子资源上加 X 锁 |
| 不指定 | 默认为排他锁（X） |

#### 锁兼容性矩阵

| 已有锁 \ 请求锁 | S | X | IS | IX |
|:---:|:---:|:---:|:---:|:---:|
| **S** | 兼容 | 冲突 | 兼容 | 冲突 |
| **X** | 冲突 | 冲突 | 冲突 | 冲突 |
| **IS** | 兼容 | 冲突 | 兼容 | 兼容 |
| **IX** | 冲突 | 冲突 | 兼容 | 兼容 |

#### 示例

```sql
-- 排他锁定表后更新数据
BEGIN;
LOCK TABLE accounts IN EXCLUSIVE MODE;
UPDATE accounts SET balance = balance - 200 WHERE name = 'Alice';
COMMIT;

-- 共享锁定，不等待
LOCK TABLE tab_data IN SHARE MODE NOWAIT;

-- 限时等待
LOCK TABLE tab_data IN EXCLUSIVE MODE WAIT 5000;
```

---

## 四、索引管理（INDEX）

### 4.1 创建索引（CREATE INDEX）

#### 语法格式

```sql
CREATE [UNIQUE] INDEX [IF NOT EXISTS] index_name
  ON [schema_name.]table_name (index_keys)
  [INDEXTYPE IS (DEFAULT | BTREE | BITMAP)]
  [LOCAL | GLOBAL [partition_clause]]
  [ONLINE | OFFLINE]
  [NOPARALLEL | PARALLEL (n [, m])]
  [NOWAIT | WAIT [wait_ms]];
```

#### 索引键定义

```sql
index_keys ::= (col_name | func_expr) [ASC | DESC]
               [, (col_name | func_expr) [ASC | DESC] ...]
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `UNIQUE` | 唯一索引，等效于添加唯一值约束 |
| `IF NOT EXISTS` | 同名索引已存在时仅打印警告 |
| `INDEXTYPE IS BTREE` | B 树索引（默认），适用于高基数列，事务型场景 |
| `INDEXTYPE IS BITMAP` | 位图索引，适用于低基数列，分析型场景 |
| `LOCAL` | 局部分区索引，与表分区对齐 |
| `GLOBAL` | 全局分区索引，按自己的规则划分分区 |
| `ONLINE` | 在线创建索引，不阻塞表上读写操作 |
| `PARALLEL (n, m)` | 并行创建，n 为索引构建并行度（0-1024），m 为数据扫描并行度（0-128） |

#### 索引设计原则

- 在 WHERE 子句常用列和 JOIN 连接列上创建索引
- 不在重复度过高的列上创建索引
- 联合索引遵循**最左前缀匹配原则**
- 函数索引支持对表达式结果建索引，优化如 `WHERE lower(email) = 'xxx'` 的查询

#### 示例

```sql
-- 唯一索引
CREATE UNIQUE INDEX idx_unique ON tab_test (id);

-- 函数索引
CREATE INDEX idx_func ON tab_test (len(name));

-- 联合索引（混合排序方向）
CREATE INDEX idx_comp ON tab_test (id ASC, name DESC, birth);

-- 位图索引
CREATE INDEX idx_bm ON tab_test (status) INDEXTYPE IS BITMAP;

-- 局部分区索引
CREATE INDEX idx_local ON tab_parti (name) LOCAL;

-- 全局分区索引（按列表分区）
CREATE INDEX idx_global ON tab_test (name, age)
  GLOBAL PARTITION BY LIST (city_name) PARTITIONS (
    p_cd VALUES ('成都'),
    p_cq VALUES ('重庆'),
    p_other VALUES (OTHERVALUES)
  );

-- 在线并行创建索引
CREATE INDEX idx_fast ON tab_large (col1) ONLINE PARALLEL (4, 2);
```

### 4.2 修改索引（ALTER INDEX）

#### 语法格式

```sql
ALTER INDEX [schema_name.]table_name.index_name
  (RENAME TO new_index_name | ADD PARTITION parti_name VALUES [LESS THAN] (parti_values))
  [NOWAIT | WAIT [wait_ms]];
```

> **注意：** 添加分区仅支持全局分区索引，不支持局部分区索引。新分区类型须与索引分区类型一致，值范围不能与已有分区重叠。

#### 示例

```sql
-- 重命名索引
ALTER INDEX tab_test.idx_1 RENAME TO idx_2;

-- 添加索引分区
ALTER INDEX tab_test.idx_gparti_1 ADD PARTITION p_wuhan VALUES ('武汉');
ALTER INDEX tab_test.idx_gparti_2 ADD PARTITION p_100 VALUES LESS THAN (100);
```

### 4.3 删除索引（DROP INDEX）

#### 语法格式

```sql
DROP INDEX [IF EXISTS] [schema_name.]table_name.index_name;
```

#### 示例

```sql
DROP INDEX tab_test.idx_1;
DROP INDEX IF EXISTS tab_test.idx_1;
```

### 4.4 重建索引（REINDEX）

#### 语法格式

```sql
REINDEX [schema_name.]table_name.(index_name | *)
  [PARTITION parti_name]
  [ONLINE | OFFLINE]
  [NOPARALLEL | PARALLEL (...)]
  [NOWAIT | WAIT [wait_ms]];
```

#### 示例

```sql
-- 重建单个索引
REINDEX tab_test.idx_1;

-- 重建表上所有索引
REINDEX tab_test.*;

-- 重建索引的一个分区
REINDEX tab_parti.idx_local PARTITION partition_chengdu;
```

> **注意：** 二级局部分区索引不支持重建一级分区；不支持重建全局分区索引的单个分区。

---

## 五、视图管理（VIEW）

### 5.1 创建视图（CREATE VIEW）

#### 语法格式

```sql
CREATE [OR REPLACE] [NOFORCE | FORCE] VIEW [IF NOT EXISTS]
  [schema_name.]view_name [(col1, col2, ...)]
  AS select_statement
  [WITH (READ ONLY | CHECK OPTION)]
  [COMMENT 'comment_string'];
```

#### 参数说明

| 参数 | 说明 |
|------|------|
| `OR REPLACE` | 视图已存在则替换 |
| `FORCE` | 强制创建（即使基表不存在） |
| `IF NOT EXISTS` | 同名视图已存在时忽略错误 |
| `WITH READ ONLY` | 只读视图，不能执行 INSERT/UPDATE/DELETE |
| `WITH CHECK OPTION` | 允许 INSERT/UPDATE，但数据必须满足视图的 WHERE 条件 |

> **注意：**
> - 不支持使用本地临时表创建视图，但支持全局临时表。
> - 通过 `*` 选择所有列后，向基表添加新列，视图中不会包含新列。
> - JOIN TABLE 中若两个表存在同名字段，创建视图将失败。

#### 示例

```sql
-- 创建基本视图
CREATE VIEW student_view (id, name, sex, department)
  AS SELECT st_no, st_name, st_sex, st_department FROM student_tab;

-- 创建带检查选项的多表视图
CREATE VIEW order_view AS
  SELECT u.id, u.name, o.product, l.status
  FROM user_tab u
  JOIN order_tab o ON u.id = o.user_id
  JOIN logistics_tab l ON o.order_id = l.order_id
  WHERE u.name = 'Alice'
  WITH CHECK OPTION
  COMMENT 'Alice的订单信息';

-- 替换已有视图
CREATE OR REPLACE VIEW order_view AS
  SELECT * FROM user_tab JOIN order_tab ON user_tab.id = order_tab.user_id;
```

### 5.2 删除视图（DROP VIEW）

#### 语法格式

```sql
DROP VIEW [IF EXISTS] [schema_name.]view_name [RESTRICT | CASCADE [CONSTRAINTS]];
```

| 参数 | 说明 |
|------|------|
| `RESTRICT` | 默认行为，若视图被其他对象依赖则报错 |
| `CASCADE` | 级联删除所有依赖该视图的对象 |

#### 示例

```sql
DROP VIEW IF EXISTS my_view;
DROP VIEW my_view CASCADE;
```

### 5.3 重编译视图

当基表结构发生变化导致视图失效时，需重编译恢复可用性。

```sql
ALTER VIEW view_name RECOMPILE;
```

#### 示例

```sql
-- 基表被删除后重建，重编译视图
ALTER VIEW dep_view RECOMPILE;

-- 查询视图有效性
SELECT VALID FROM SYS_VIEWS WHERE view_name = 'dep_view';
```

---

## 六、序列值管理（SEQUENCE）

### 6.1 创建序列值（CREATE SEQUENCE）

#### 语法格式

```sql
CREATE SEQUENCE [IF NOT EXISTS] [schema_name.]seq_name
  [INCREMENT BY step_value]
  [START WITH init_value]
  [MINVALUE min_value | NOMINVALUE]
  [MAXVALUE max_value | NOMAXVALUE]
  [CACHE cache_count | NOCACHE]
  [CYCLE | NO CYCLE]
  [COMMENT 'comment_string'];
```

#### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `INCREMENT BY` | 1 | 步长，负值表示递减 |
| `START WITH` | 递增时为最小值，递减时为最大值 | 序列初始值 |
| `MINVALUE` | 递增：1，递减：-9223372036854775808 | 最小值 |
| `MAXVALUE` | 递增：9223372036854775807，递减：-1 | 最大值 |
| `CACHE` | 1（无缓存） | 预分配到内存中的序列号数量，提升获取速度 |
| `CYCLE` | NO CYCLE | 达到最大/最小值后是否循环 |

> **提示：**
> - 高并发场景建议设置较大的 CACHE 值以减少磁盘交互。
> - 集群模式下若需各工作节点序列值连续同步，需将 CACHE 设为 1。
> - 系统重启后未使用的缓存序列值将丢失。

#### 示例

```sql
-- 创建标准递增序列
CREATE SEQUENCE seq_1 MINVALUE 1 MAXVALUE 1000 START WITH 100 INCREMENT BY 1;

-- 创建带缓存和循环的序列
CREATE SEQUENCE seq_cached MINVALUE 1 MAXVALUE 10000
  START WITH 1 INCREMENT BY 1 CACHE 10 CYCLE;

-- IF NOT EXISTS
CREATE SEQUENCE IF NOT EXISTS seq_safe MAXVALUE 1000;
```

### 6.2 修改序列值（ALTER SEQUENCE）

```sql
ALTER SEQUENCE [schema_name.]seq_name
  [INCREMENT BY step_value]
  [MINVALUE min_value | NOMINVALUE]
  [MAXVALUE max_value | NOMAXVALUE]
  [CACHE cache_count | NOCACHE]
  [CYCLE | NO CYCLE];
```

#### 示例

```sql
ALTER SEQUENCE seq_test MINVALUE 5 MAXVALUE 500 INCREMENT BY 5 CACHE 20 CYCLE;
```

### 6.3 使用序列值

```sql
-- 获取下一个值
SELECT seq_name.NEXTVAL FROM DUAL;

-- 获取当前值（必须先调用 NEXTVAL）
SELECT CURRVAL('seq_name') FROM DUAL;
```

> **注意：** `CURRVAL` 的生命周期仅存在于当前会话中，各会话之间独立。在调用 NEXTVAL 之前使用 CURRVAL 会报错。

### 6.4 删除序列值（DROP SEQUENCE）

```sql
DROP SEQUENCE [IF EXISTS] [schema_name.]seq_name [RESTRICT | CASCADE];
```

| 参数 | 说明 |
|------|------|
| `RESTRICT` | 默认行为，存在依赖对象时报错 |
| `CASCADE` | 强制删除（但序列值作为字段默认值时不允许删除） |

#### 示例

```sql
-- 序列被表字段引用时，需先解除引用
ALTER TABLE tab_1 ALTER COLUMN col_1 DEFAULT 1;
DROP SEQUENCE seq_test;
```

---

## 七、与 Oracle / MySQL / PostgreSQL DDL 语法对比

### 7.1 CREATE TABLE 核心差异

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 条件创建 | `IF NOT EXISTS` | 不支持（需 PL/SQL 判断） | `IF NOT EXISTS` | `IF NOT EXISTS` |
| 自增列 | `IDENTITY(init,step)` / `AUTO_INCREMENT` | `GENERATED AS IDENTITY`（12c+） | `AUTO_INCREMENT` | `GENERATED AS IDENTITY` / `SERIAL` |
| 表加密 | `ENCRYPT BY encryptor_name` | TDE（透明加密） | 表级 `ENCRYPTION='Y'`（InnoDB） | 不支持表级（需 pgcrypto） |
| 临时表 | `LOCAL/GLOBAL TEMPORARY` | `GLOBAL TEMPORARY`（仅全局） | `TEMPORARY`（仅局部） | `TEMPORARY` / `UNLOGGED` |
| 表注释 | `COMMENT 'text'`（建表时直接指定） | `COMMENT ON TABLE`（单独语句） | `COMMENT 'text'`（建表时） | `COMMENT ON TABLE`（单独语句） |
| 列注释 | `COMMENT 'text'`（列定义后） | `COMMENT ON COLUMN` | `COMMENT 'text'`（列定义后） | `COMMENT ON COLUMN` |
| 外部表 | `ORGANIZATION EXTERNAL (...)` | `ORGANIZATION EXTERNAL (...)` | 不支持（需引擎如 FEDERATED） | `CREATE FOREIGN TABLE`（FDW） |

### 7.2 分区语法对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 范围分区 | `PARTITION BY RANGE ... PARTITIONS (... VALUES LESS THAN ...)` | `PARTITION BY RANGE ... (PARTITION ... VALUES LESS THAN ...)` | `PARTITION BY RANGE ... (PARTITION ... VALUES LESS THAN ...)` | `PARTITION BY RANGE ...`（需单独 `CREATE TABLE ... PARTITION OF`） |
| 列表分区 | `PARTITION BY LIST ... PARTITIONS (... VALUES ...)` | `PARTITION BY LIST ... (PARTITION ... VALUES ...)` | `PARTITION BY LIST ... (PARTITION ... VALUES IN ...)` | 需单独创建分区表 |
| 哈希分区 | `PARTITION BY HASH ... PARTITIONS n` | `PARTITION BY HASH ... PARTITIONS n` | `PARTITION BY HASH ... PARTITIONS n` | `PARTITION BY HASH ...` + 单独创建 |
| 二级分区 | `SUBPARTITION BY ...`（与主语句合并定义） | `SUBPARTITION BY ...` | 仅支持 `RANGE-RANGE` 等有限组合 | 需手动嵌套定义 |
| 自动扩展分区 | `INTERVAL n DAY/MONTH/...` | `INTERVAL` 语法类似 | 不支持 | 不支持（需触发器） |
| 默认分区 | `MAXVALUES` / `OTHERVALUES` | `MAXVALUE` / `DEFAULT` | `MAXVALUE` / 不支持列表默认 | `DEFAULT` |
| 按分区查询 | `SELECT ... FROM t PARTITION (p1)` | `SELECT ... FROM t PARTITION (p1)` | `SELECT ... FROM t PARTITION (p1)` | 直接查询分区子表 |

### 7.3 序列语法对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 独立序列对象 | `CREATE SEQUENCE` | `CREATE SEQUENCE` | 不支持（通过 `AUTO_INCREMENT`） | `CREATE SEQUENCE` |
| 获取下一个值 | `seq.NEXTVAL` | `seq.NEXTVAL` | 不适用 | `nextval('seq')` |
| 获取当前值 | `CURRVAL('seq')` | `seq.CURRVAL` | 不适用 | `currval('seq')` |
| 缓存 | `CACHE n` | `CACHE n` | 不适用 | `CACHE n` |
| 循环 | `CYCLE / NO CYCLE` | `CYCLE / NOCYCLE` | 不适用 | `CYCLE / NO CYCLE` |
| IF NOT EXISTS | 支持 | 不支持 | 不适用 | 支持（PG 9.5+） |

### 7.4 索引语法对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 位图索引 | `INDEXTYPE IS BITMAP` | `CREATE BITMAP INDEX` | 不支持 | 不支持（可用 BRIN 替代） |
| 函数索引 | `CREATE INDEX ... (func_expr)` | `CREATE INDEX ... (func_expr)` | 支持（8.0+） | `CREATE INDEX ... (func_expr)` |
| 在线创建 | `ONLINE` | `ONLINE` | `ALGORITHM=INPLACE`（InnoDB） | `CONCURRENTLY` |
| 并行创建 | `PARALLEL (n, m)` | `PARALLEL n` | 不支持 | `max_parallel_maintenance_workers` |
| 索引分区 | `LOCAL / GLOBAL PARTITION BY ...` | `LOCAL / GLOBAL PARTITION BY ...` | 不支持全局分区索引 | 自动继承表分区 |
| 重建索引 | `REINDEX table.index` | `ALTER INDEX ... REBUILD` | `ALTER TABLE ... FORCE` / `OPTIMIZE TABLE` | `REINDEX INDEX` |
| IF NOT EXISTS | 支持 | 不支持 | `IF NOT EXISTS`（部分版本） | `IF NOT EXISTS` |

### 7.5 视图语法对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `OR REPLACE` | 支持 | 支持 | `OR REPLACE` | `OR REPLACE` |
| `FORCE` 创建 | 支持 `FORCE / NOFORCE` | 支持 `FORCE / NOFORCE` | 不支持 | 不支持 |
| `WITH CHECK OPTION` | 支持 | 支持 | 支持 | 支持 |
| `WITH READ ONLY` | 支持 | 支持 | 不支持（需 GRANT 控制） | 不支持（需规则/触发器） |
| 重编译视图 | `ALTER VIEW ... RECOMPILE` | `ALTER VIEW ... COMPILE` | 不需要（自动） | 不需要（自动） |
| `IF NOT EXISTS` | 支持 | 不支持 | `IF NOT EXISTS`（部分版本） | 不支持 |
