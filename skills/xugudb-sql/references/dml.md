---
title: 虚谷数据库 DML 语句参考
description: 虚谷数据库数据操作语言（DML）完整参考，涵盖 DELETE、INSERT、UPDATE、MERGE INTO、EXECUTE、PREPARE、SET、SHOW 语句的语法、参数及示例
tags: [xugudb, dml, insert, delete, update, merge, execute, prepare, set, show, sql]
---

# 虚谷数据库 DML 语句参考

本文档涵盖虚谷数据库支持的所有 DML（数据操作语言）语句，包括：DELETE、INSERT、UPDATE、MERGE INTO、EXECUTE、PREPARE、SET、SHOW。

---

## DELETE

### 概述

DELETE 用于从数据库表中删除记录。操作范围灵活，既可删除全部记录，也可仅删除满足特定条件的部分数据。作为可回滚的 DML 操作，执行结果纳入事务管理——执行 COMMIT 前删除操作不会持久化，执行 ROLLBACK 可恢复被删除记录。

### 语法格式

```sql
DeleteStmt ::= 'DELETE' del_targ_tab opt_from_clause? delete_where_clause? opt_returning? opt_bulk? opt_into_list?

del_targ_tab        ::= 'FROM'? base_table_ref
delete_where_clause ::= opt_where_clause | 'WHERE' 'CURRENT' 'OF' name
opt_where_clause    ::= 'WHERE' bool_expr
opt_returning       ::= 'RETURNING' target_list
opt_bulk            ::= 'BULK' 'COLLECT'
opt_into_list       ::= 'INTO' ident_list
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `del_targ_tab` | 目标表名（`base_table_ref`），可加模式名前缀 `schema_name.table_name`。支持带或不带 `FROM` 关键字 |
| `delete_where_clause` | 省略时删除整张表数据 |
| `opt_where_clause` | 根据 `bool_expr` 布尔表达式过滤条件删除满足的行 |
| `WHERE CURRENT OF` | 用于 PL/SQL 中通过游标删除当前行 |
| `opt_returning` | DELETE 后返回被删除的列值 |
| `target_list` | 需要返回的字段列表 |
| `opt_bulk` | 批量收集返回结果，常与 `RETURNING...INTO` 结合，用于一次性获取多行 |
| `opt_into_list` | 将 `DELETE...RETURNING` 的返回值赋值给 PL/SQL 中的变量 |

### SQL 示例

#### 删除全表数据

```sql
-- 带 FROM 关键字
DELETE FROM employees;

-- 不带 FROM 关键字
DELETE employees;
```

#### 多表条件删除（opt_from_clause）

```sql
DELETE FROM em_information AS t1 FROM employees AS t2
WHERE t1.id = t2.emp_id;
```

#### WHERE 条件删除

```sql
DELETE FROM employees WHERE dept_id = 30;
```

#### RETURNING INTO 删除单行并返回

```sql
DECLARE
  v_name   employees.name%TYPE;
  v_salary employees.salary%TYPE;
BEGIN
  DELETE FROM employees
  WHERE emp_id = 2
  RETURNING name, salary INTO v_name, v_salary;
  DBMS_OUTPUT.PUT_LINE('Deleted: ' || v_name || ', salary: ' || v_salary);
END;
/
-- 输出: Deleted: Bob, salary: 4000
```

#### RETURNING BULK COLLECT INTO 删除多行并批量收集

```sql
DECLARE
  TYPE name_list IS TABLE OF employees.name%TYPE;
  TYPE salary_list IS TABLE OF employees.salary%TYPE;
  v_names    name_list;
  v_salaries salary_list;
BEGIN
  DELETE FROM employees
  WHERE salary < 6000
  RETURNING name, salary BULK COLLECT INTO v_names, v_salaries;

  FOR i IN 1 .. v_names.COUNT LOOP
    DBMS_OUTPUT.PUT_LINE('Deleted: ' || v_names(i) || ', salary: ' || v_salaries(i));
  END LOOP;
END;
/
```

#### 使用 WHERE CURRENT OF（游标方式删除）

```sql
DECLARE
  CURSOR emp_cur IS
    SELECT emp_id FROM employees WHERE dept_id = 10 FOR UPDATE;
  v_emp_id employees.emp_id%TYPE;
BEGIN
  OPEN emp_cur;
  LOOP
    FETCH emp_cur INTO v_emp_id;
    EXIT WHEN emp_cur%NOTFOUND;
    DELETE FROM employees WHERE CURRENT OF emp_cur;
    DBMS_OUTPUT.PUT_LINE('Deleted emp_id: ' || v_emp_id);
  END LOOP;
  CLOSE emp_cur;
END;
/
```

### 注意事项

- 用户需具备目标表的 DELETE 权限。
- 若使用外键约束，必须确保被删除记录不会违反引用完整性。
- DELETE 可与事务控制语句（`BEGIN`、`COMMIT`、`ROLLBACK`）配合使用。
- 无 WHERE 条件会删除全表记录，但不会释放表空间结构，慎重使用。
- DELETE 属于逐行处理，删除大量数据可能导致性能下降或产生大量日志，可考虑使用 `TRUNCATE`。
- `TRUNCATE TABLE` 是 DDL 操作，效率高于 DELETE，自动 COMMIT，无法 ROLLBACK，适用于不需要回滚的大批量清表场景。
- 索引列上的删除可能导致索引碎片增加，需考虑定期重建索引（`REINDEX`）。
- `WHERE CURRENT OF` 只能用于游标循环中，游标必须声明为 `FOR UPDATE`。
- `WHERE CURRENT OF` 只能用于 UPDATE 或 DELETE，且不能和普通 WHERE 混合使用。

---

## INSERT

### 概述

INSERT 是 SQL 中最基本的数据操作语句之一，用于向数据库表或视图中添加新记录。虚谷数据库支持以下 INSERT 类型：

- **标准 INSERT 语句**：支持单行、多行插入、将查询结果直接插入目标表
- **多表插入（MULTI INSERT）**：单语句插入多表多行数据
- **替换插入（REPLACE）**：主键唯一值冲突时自动替换原记录
- **忽略冲突插入（IGNORE）**：主键唯一值冲突时忽略报错

### 使用前提

1. 表或视图必须已存在。
2. 指定的列必须存在于目标表中；省略列名列表时，VALUES 子句必须为所有列提供值。
3. 插入的值必须与对应列的数据类型兼容。
4. 插入数据必须满足表的约束条件（NOT NULL、PRIMARY KEY、UNIQUE 等）。
5. 对于自增列（IDENTITY），通常不应在 INSERT 中指定值。
6. 用户必须对目标表或视图有 INSERT 权限；使用查询插入还需 SELECT 权限。

### 语法格式

```sql
InsertStmt      ::= SingleTabInsert | MultiTabInsert | ReplaceInsert | IgnoreInsert

SingleTabInsert ::= 'INSERT' 'INTO' relation_expr insert_rest returning_clause
MultiTabInsert  ::= (conditional_insert | ('ALL' insert_into_clause+)) select_no_parens
ReplaceInsert   ::= 'REPLACE' 'INTO' relation_expr insert_rest
IgnoreInsert    ::= 'INSERT' 'IGNORE' 'INTO' relation_expr insert_rest

relation_expr   ::= (schema_name '.')? (table_name (opt_parti_clip_clause | '@' link_name)? | view_name ('@' link_name)?)
insert_rest     ::= 'VALUES' (insert_values | record_val)
                   | 'DEFAULT' 'VALUES'
                   | SelectStmt
                   | '(' columnList ')' ('VALUES' (insert_values | record_val) | SelectStmt)
insert_values   ::= '(' values_list ')' ( ','? '(' values_list ')' )*
values_list     ::= (expr | 'DEFAULT') (',' (expr | 'DEFAULT'))*

returning_clause ::= 'RETURNING' target_list ('BULK' 'COLLECT')? ('INTO' IDENT (',' IDENT)*)
```

### 标准 INSERT 语句

#### 参数说明

| 参数 | 说明 |
|------|------|
| `relation_expr` | 插入目标，可以是表或视图，支持 `schema_name.table_name` 格式 |
| `@link_name` | 指定远程数据库连接对象名称（通过 `CREATE DATABASE LINK` 建立） |
| `opt_parti_clip_clause` | 指定分区 `PARTITION(name)` 或子分区 `SUBPARTITION(name)` |
| `VALUES` | 直接指定要插入的值 |
| `DEFAULT VALUES` | 插入所有列的默认值；无默认值的列插入 NULL |
| `DEFAULT` | 指定某列使用默认值 |
| `columnList` | 显式指定要插入的列 |
| `SelectStmt` | 将查询结果插入表中 |
| `record_val` | 使用 `%ROWTYPE` 记录变量的值作为数据插入 |
| `RETURNING ... INTO` | 返回被插入的列值到变量 |
| `BULK COLLECT` | 批量收集插入的多行数据到 PL/SQL 集合中 |

#### SQL 示例

**单行插入：**

```sql
INSERT INTO employees VALUES (1, 'Alice', 8000, 10);
```

**多行插入（逗号分隔）：**

```sql
INSERT INTO tb_vals VALUES(3),(4);
```

**多行插入（无逗号分隔，虚谷特有）：**

```sql
INSERT INTO tb_vals VALUES(1)(2);
```

**指定列名插入：**

```sql
INSERT INTO tb_cols(id) VALUES(1);
```

**跨模式插入：**

```sql
INSERT INTO sch_test.tb_test VALUES(1);
```

**向指定分区插入：**

```sql
INSERT INTO tb_part PARTITION(p1)(id) VALUES(1);
```

**使用 DEFAULT VALUES：**

```sql
-- 有默认值的列使用默认值，无默认值的列插入 NULL
INSERT INTO tb_defs DEFAULT VALUES;
```

**指定列使用 DEFAULT：**

```sql
INSERT INTO tb_def VALUES(DEFAULT, DEFAULT);
```

**插入查询结果：**

```sql
INSERT INTO tb_sels SELECT 10 FROM dual;
```

**使用不同类型值插入（常量、函数、表达式）：**

```sql
INSERT INTO tb_val VALUES (1001, '张三')(1002, SYSDATE)(1003, 2*1.1);
```

**插入 RECORD 类型：**

```sql
DECLARE
  TYPE tb_res IS TABLE OF tb_rec%ROWTYPE;
  rs tb_res;
BEGIN
  SELECT * BULK COLLECT INTO rs FROM tb_rec;
  FOR i IN rs.FIRST .. rs.LAST LOOP
    INSERT INTO tb_rec1 VALUES rs(i);
  END LOOP;
END;
/
```

**RETURNING BULK COLLECT INTO：**

```sql
DECLARE
  TYPE type_table_varchar IS TABLE OF VARCHAR;
  var_chr type_table_varchar;
BEGIN
  INSERT INTO tb_ret VALUES(1)(2)(3)
  RETURNING id BULK COLLECT INTO var_chr;
  FOR j IN 1..var_chr.COUNT() LOOP
    SEND_MSG(var_chr(j));
  END LOOP;
END;
/
```

### 多表插入（MULTI INSERT）

#### 无条件多表插入

使用 `INSERT ALL` 将查询结果无条件插入所有指定表：

```sql
INSERT ALL
  INTO tb_all1(id)
  INTO tb_all2(id)
SELECT level FROM dual CONNECT BY level <= 200;
```

> 若省略 VALUES 子句，直接使用 SELECT 查询结果的列值作为插入数据。若包含 VALUES 子句且提供常量值，SELECT 查询将被忽略。

#### 条件多表插入（INSERT FIRST / INSERT ALL）

**INSERT FIRST**：对每行数据按顺序检查 WHEN 条件，遇到第一个满足的条件执行对应插入后跳过后续条件。每行最多插入到一个表中。

```sql
INSERT FIRST
  WHEN col1 > 10 THEN INTO ins_tab2 VALUES(col1, col2, col3, col4)
  WHEN col1 > 3  THEN INTO ins_tab3 VALUES(col1, col2, col3, col4)
SELECT col1, col2, col3, col4 FROM ins_tab1;
```

**INSERT ALL（条件）**：对每行数据检查每个 WHEN 条件，满足的都会执行插入。一行数据可能被插入到多个表中。

```sql
INSERT ALL
  WHEN col1 > 10 THEN INTO ins_tab2 VALUES(col1, col2, col3, col4)
  WHEN col1 > 3  THEN INTO ins_tab3 VALUES(col1, col2, col3, col4)
SELECT col1, col2, col3, col4 FROM ins_tab1;
```

> 不指定 FIRST 或 ALL 时默认为 ALL。存在 ELSE 子句时，当没有任何 WHEN 条件满足时执行 ELSE 对应的插入。

### 替换插入（REPLACE）

当新数据违反唯一约束（主键或唯一值冲突）时，数据库自动删除已存在的冲突行，然后插入新数据；无冲突时等同于普通 INSERT。

```sql
REPLACE INTO tb_ins VALUES(1, 'abc')(1, 'one');
-- 结果：仅保留 (1, 'one')
```

> REPLACE 操作如果有冲突会在日志中记载错误信息。

### 忽略冲突插入（INSERT IGNORE）

插入数据违反唯一约束时忽略错误，不插入该行数据，不影响其他行的插入。

```sql
INSERT IGNORE INTO tb_ins VALUES(3, 'abc')(3, 'three')(4, 'abc');
-- 第二组冲突被跳过，第一、三组成功插入
```

> 目前仅支持跳过主键唯一值相关错误，不支持跳过数据类型不匹配等错误。IGNORE 操作如果有冲突会在日志中记载错误信息。

### 注意事项

- 多行插入支持两种格式：逗号分隔 `VALUES(1),(2)` 和无逗号分隔 `VALUES(1)(2)`。
- `DEFAULT VALUES` 要求表中每列都有默认值或允许 NULL，否则插入失败。
- 查询结果列数和类型必须与目标表匹配。
- 在 PL/SQL 中，RETURNING 返回的结果必须匹配对应的字段列表。

---

## UPDATE

### 概述

UPDATE 用于修改表或视图中现有数据的值，支持：修改单条或多条记录、基于条件选择性更新、批量更新、多列同时更新、跨表关联更新。

### 语法格式

```sql
UpdateStmt ::= 'UPDATE' base_table_refs 'SET' (update_filter_clause | update_select_clause)
               opt_returning? opt_bulk? opt_into_list?

base_table_refs       ::= base_table_ref (',' base_table_ref)*
base_table_ref        ::= relation_expr alias_clause?
update_filter_clause  ::= update_target_list opt_from_clause? update_where_clause?
update_select_clause  ::= '(' update_target_list ')' '=' select_with_parens opt_where_clause?
update_target_list    ::= update_target_el (',' update_target_el)*
update_target_el      ::= ColumnName ('=' (bool_expr | b_expr | 'DEFAULT'))?
                         | ident_arr '=' b_expr
opt_from_clause       ::= 'FROM' form_list
update_where_clause   ::= opt_where_clause | 'WHERE' 'CURRENT' 'OF' cursor_name
opt_returning         ::= 'RETURNING' target_list
opt_bulk              ::= 'BULK' 'COLLECT'
opt_into_list         ::= 'INTO' ident_list
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `base_table_refs` | 更新的表或视图名列表，支持多表更新，可加模式名前缀 |
| `alias_clause` | 表的别名（可选） |
| `opt_parti_clip_clause` | 指定分区（`PARTITION`）或子分区（`SUBPARTITION`） |
| `@link_name` | 指定远程数据库对象 |
| `update_target_el` | 更新目标元素：列名 = 值 / 布尔表达式 / `DEFAULT` |
| `ident_arr` | 数组类型的更新方式 |
| `opt_from_clause` | FROM 子句，允许其他表的列出现在 WHERE 子句和更新表达式中 |
| `update_where_clause` | 更新满足条件的行 |
| `WHERE CURRENT OF` | 在 PL/SQL 中通过游标更新当前行 |
| `update_select_clause` | 利用 SELECT 子句的查询结果更新表 |
| `opt_returning` | 把更新后的结果输出至指定变量 |
| `opt_bulk` | 允许一次性加载查询结果集到变量中 |

### 约束条件

- **主键约束**：不能更新为 NULL 或已存在的值。
- **唯一约束**：新值不能与其他行重复。
- **外键约束**：新值必须在被引用表的主键列或唯一约束列中存在。
- **检查约束**：更新值必须满足 CHECK 条件。
- **非空约束**：不能将 NOT NULL 列更新为 NULL。

### SQL 示例

#### 更新为常值

```sql
UPDATE tab_update1 SET c2 = 'new c2', c3 = DEFAULT WHERE c1 = 1;
```

#### 使用子查询更新

```sql
UPDATE tab_update1
SET (c2, c3) = (SELECT MAX(c2), MIN(c3) + 10 FROM tab_update1)
WHERE c1 = 2;
```

#### 更新并 RETURNING BULK COLLECT INTO

```sql
DECLARE
  TYPE type_table_1 IS TABLE OF tab_update1%ROWTYPE;
  var_t1 type_table_1;
  TYPE type_table_int IS TABLE OF tab_update1.c2%TYPE;
  var_int type_table_int;
  CURSOR cur IS SELECT * FROM tab_update1;
BEGIN
  OPEN cur;
  FETCH cur BULK COLLECT INTO var_t1;
  UPDATE tab_update1 SET c2 = var_t1(1).c2, c3 = c3 + 100
  RETURNING c1 BULK COLLECT INTO var_int;
  CLOSE cur;
  FOR i IN 1..var_int.COUNT() LOOP
    SEND_MSG(var_int(i));
  END LOOP;
END;
/
```

### 注意事项

- 用户需对目标表或视图具有 UPDATE 权限；使用查询结果更新还需 SELECT 权限。
- UPDATE 语句不支持 `ORDER BY` 子句和 `LIMIT` 子句。
- `WHERE CURRENT OF` 只能用于游标循环中，游标必须声明为 `FOR UPDATE`。
- `WHERE CURRENT OF` 不能和普通 WHERE 混合使用。
- 在 PL/SQL 中，RETURNING 返回的结果必须匹配对应的字段列表。

---

## MERGE INTO

### 概述

MERGE INTO 用于实现合并操作，结合了 INSERT 和 UPDATE 功能，允许在单一操作中根据条件判断数据是否存在，从而决定执行更新还是插入新数据。在处理大量数据时能显著提高效率，减少多次查询的开销。

### 语法格式

```sql
MergeIntoStmt ::= 'MERGE' 'INTO' base_table_ref
                  'USING' table_ref
                  'ON' '(' bool_expr ')'
                  ( merge_update_clause merge_insert_clause?
                  | merge_insert_clause merge_update_clause? )

merge_update_clause ::= 'WHEN' 'MATCHED' 'THEN' 'UPDATE' 'SET' update_target_list opt_where_clause?
merge_insert_clause ::= 'WHEN' 'NOT' 'MATCHED' 'THEN' 'INSERT' insert_merge opt_where_clause?
insert_merge        ::= 'DEFAULT' 'VALUES'
                       | (('(' columnList2 ')')? 'VALUES' insert_values)
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `INTO base_table_ref` | 指定要更新/插入的目标表，可加模式名 |
| `USING table_ref` | 获取源表或视图数据，可以是表名、子查询、连接表等 |
| `ON (bool_expr)` | 指定匹配条件 |
| `WHEN MATCHED THEN UPDATE` | ON 条件为真时执行更新 |
| `WHEN NOT MATCHED THEN INSERT` | ON 条件为 false 时插入新行 |
| `opt_where_clause` | 可选的 WHERE 从句，用于过滤或限制数据 |
| `insert_merge` | 插入子句，指定要插入的数据 |

### SQL 示例

#### 仅 INSERT（不匹配时插入）

```sql
MERGE INTO tab_merge1 a
USING (SELECT b.aid, b.name, b.year FROM tab_merge2 b) c
ON (a.id = c.aid)
WHEN NOT MATCHED THEN
  INSERT(a.id, a.name, a.year) VALUES(c.aid, c.name, c.year);
```

#### 仅 UPDATE（匹配时更新）

```sql
MERGE INTO tab_merge3 a
USING (SELECT b.aid, b.name, b.year FROM tab_merge4 b) c
ON (tab_merge3.id = c.aid)
WHEN MATCHED THEN
  UPDATE SET tab_merge3.year = c.year;
```

#### 匹配时更新，不匹配时新增

```sql
MERGE INTO tab_merge5 a
USING (SELECT b.aid, b.name, b.year, b.city FROM tab_merge6 b) c
ON (tab_merge5.id = c.aid)
WHEN MATCHED THEN
  UPDATE SET a.name = c.name
WHEN NOT MATCHED THEN
  INSERT(a.id, a.name, a.year) VALUES(c.aid, c.name, c.year);
```

### 注意事项

- 更新语句中更新的列不能为 ON 语句中的列。
- 如果省略列名列表，目标表的列数必须与 VALUES 中的字段数匹配。
- MERGE INTO 支持同时包含 UPDATE 和 INSERT 子句，顺序可互换。

---

## EXECUTE

### 概述

EXECUTE 命令用于调用预定义数据库程序单元，功能包括：

- 调用存储过程/存储函数（无参或有参）
- 调用包内存储过程/存储函数
- 执行动态 SQL（`EXECUTE IMMEDIATE`）
- 简化执行语句，无需封装在 `BEGIN ... END;` 块中

> 在虚谷数据库中，`CALL` 与 `EXECUTE` 两者等价。`EXEC` 是 `EXECUTE` 的简写形式。

### 语法格式

```sql
ExecuteStmt ::= ('EXECUTE' | 'EXEC') func_name ('(' func_params ')')?
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `func_name` | 要执行的存储过程/存储函数名字 |
| `func_params` | 存储过程/存储函数的实参列表，多个参数用英文逗号分隔 |

### SQL 示例

#### 调用无参存储过程/函数

```sql
EXECUTE say_hello;
EXEC say_hello;
```

#### 调用有参存储过程

```sql
EXECUTE log_action(1, 'user_login');
```

#### 调用包内存储过程/函数

```sql
EXECUTE math_pkg.add_num(10, 20);
EXECUTE math_pkg.add_num_2(10, 20);
```

#### 使用 EXECUTE IMMEDIATE 执行动态 SQL

```sql
BEGIN
  EXECUTE IMMEDIATE 'CREATE TABLE temp_table (id NUMBER, name VARCHAR2(50))';
END;
/

BEGIN
  EXECUTE IMMEDIATE 'INSERT INTO temp_table VALUES (1, ''test'')';
END;
/
```

### 注意事项

- EXECUTE 不能直接用于存储函数返回值赋值，可通过 OUT 型参数间接返回结果。
- 参数类型和顺序需与被调用程序严格匹配，缺失或错误将导致运行时失败。

---

## PREPARE

### 概述

PREPARE 为服务端提供预处理语句，可用于优化性能。执行 PREPARE 时将解析、分析和重写指定的 SQL。当发出执行命令时将执行准备好的语句，避免重复的解析分析工作。

### 创建 PREPARE

#### 语法格式

```sql
PrepareStmt ::= 'PREPARE' pre_name 'AS' prepare_sql_stmt
```

| 参数 | 说明 |
|------|------|
| `pre_name` | PREPARE 语句的名称 |
| `prepare_sql_stmt` | 预编译的 SQL 语句，支持 DML 和 DDL |

#### 示例

```sql
-- 创建查询的 PREPARE
PREPARE pre_s AS SELECT * FROM tab_test;

-- 创建插入视图的 PREPARE
PREPARE pre_i AS INSERT INTO view_test1 VALUES(1, 'a');
```

### 执行 PREPARE

#### 语法格式

```sql
CallPreStmt  ::= '?' pre_name AsCursorStmt?
AsCursorStmt ::= 'AS' 'CURSOR' cursor_name ('RETURN' ICONST)?
```

| 参数 | 说明 |
|------|------|
| `pre_name` | PREPARE 语句的名称 |
| `AsCursorStmt` | 将 PREPARE 语句声明为游标（仅对 SELECT 的 PREPARE 有效） |
| `cursor_name` | 游标名称 |
| `RETURN ICONST` | 执行 PREPARE 语句返回的行数，剩余数据通过 `FETCH cursor_name` 获取 |

#### 示例

```sql
-- 直接执行
?pre_s;

-- 返回一条数据并声明为游标，后续通过 FETCH 获取剩余数据
?pre_s AS CURSOR cur_s RETURN 1;
FETCH cur_s;
```

### 释放 PREPARE

#### 语法格式

```sql
DeallocatePreStmt ::= 'DEALLOCATE' pre_name
```

#### 示例

```sql
DEALLOCATE pre_s;
```

### 注意事项

- PREPARE 语句仅在当前会话中生效，会话关闭后自动释放。
- 可通过参数 `max_prepare_num` 设置当前会话的最大 PREPARE 语句数。
- 可通过参数 `prepare_reuse` 关闭预处理重用。
- PREPARE 方式执行 DML 语句，持有的依赖对象锁在事务提交时释放。
- 若依赖对象被删除，执行 PREPARE 时将报错。

---

## SET

### 概述

SET 语句用于设置运行时参数，支持三类参数：

1. **系统级配置参数**：影响整个集群，写入 `xugu.ini` 文件，不可回滚。
2. **连接会话级参数**：特定于当前会话，生命周期由会话决定。
3. **用户自定义变量**：会话级别，客户端定义，其他客户端不可见，会话结束时自动释放。

### 设置系统参数

#### 语法格式

```sql
VariableSetStmt ::= 'SET' var_name ('TO' var_value | ('ON' | 'OFF')) opt_on_node?

var_value    ::= opt_boolean | SCONST | ('-')? (ICONST | FCONST)
opt_boolean  ::= 'TRUE' | 'FALSE' | 'ON' | 'OFF'
opt_on_node  ::= 'ON' ( ('LOCAL' | 'ALL') 'NODE' | 'NODE' NODE_ID )
```

| 参数 | 说明 |
|------|------|
| `var_name` | 系统参数名 |
| `var_value` | 参数值，支持布尔、字符、整型、浮点型 |
| `ON ALL NODE` | 对集群所有节点设置（默认行为） |
| `ON LOCAL NODE` | 对当前节点设置 |
| `ON NODE NODE_ID` | 对指定节点设置 |

#### 示例

```sql
-- 设置反斜杠转义
SET backslash_escapes TO ON;

-- 对指定节点设置参数
SET reg_command TO TRUE ON NODE 2;
```

### 设置会话参数

#### SET SESSION AUTHORIZATION

设置当前会话的用户标识符。

```sql
SET SESSION AUTHORIZATION usr_auth;
```

#### SET TRANSACTION

设置当前事务隔离级别或访问模式。

```sql
-- 语法
SET TRANSACTION (READ ONLY | READ WRITE)
SET TRANSACTION ISOLATION LEVEL (READ ONLY | READ COMMITTED | REPEATABLE READ | SERIALIZABLE)
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL ...
```

| 隔离级别 | 说明 |
|----------|------|
| `READ ONLY` | 只读事务 |
| `READ COMMITTED` | 读已提交 |
| `REPEATABLE READ` | 可重复读 |
| `SERIALIZABLE` | 序列化 |

#### SET AUTO COMMIT

```sql
SET AUTO COMMIT ON;   -- 开启自动提交
SET AUTO COMMIT OFF;  -- 关闭自动提交
```

#### SET SCHEMA

设置当前会话的模式。

```sql
SET SCHEMA schema_name;
SET SCHEMA DEFAULT;      -- 切换到当前用户的默认同名模式
SET CURRENT SCHEMA schema_name;
```

#### SET NAMES

设置当前会话字符集。

```sql
SET NAMES 'GBK';
SET NAMES 'UTF8';
```

#### SET NULL

设置空字符串是否作为 NULL 处理。

```sql
SET NULL NULL;    -- 空串不作为 NULL 处理
SET NULL '';      -- 空串作为 NULL 处理
```

### 设置用户自定义变量

#### 语法格式

```sql
VarListSetStmt ::= 'SET' variable_el (',' variable_el)*
variable_el    ::= '@' var_name ('=' | 'ASSIGN') b_expr
```

#### 示例

```sql
-- 设置多个自定义变量
SET @var_test1 = ('test1'), @var_test2 = ('test2');
SELECT @var_test1;

-- 赋值为表达式
SET @var_dt = SYSDATE();
SELECT @var_dt;
```

### 注意事项

- 系统参数变更写入 `xugu.ini` 文件，不能被回滚。
- 部分系统参数无对应会话级参数，更改后影响所有会话并立即生效。另一部分作为会话参数初始化值，仅影响后续新建会话。
- 会话参数修改只在当前会话有效，不影响其他连接。
- 自定义变量不支持右值为查询语句。
- 访问未定义的自定义变量将返回 NULL。

---

## SHOW

### 概述

SHOW 命令用于显示变量值，支持以下几类：

- 系统配置（`xugu.ini` 相关）
- 连接会话参数
- 内置全局变量
- 用户自定义会话变量

### 语法格式

```sql
VariableShowStmt ::= 'SHOW' (ColumnName | 'TRANSACTION' 'ISOLATION' 'LEVEL' | 'AUTO' 'COMMIT' | 'CURRENT' 'SCHEMA')
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `ColumnName` | 待查看的变量名 |
| `TRANSACTION ISOLATION LEVEL` | 当前会话的事务隔离级别 |
| `AUTO COMMIT` | 当前会话的事务是否自动提交 |
| `CURRENT SCHEMA` | 当前模式 |

### SQL 示例

```sql
SHOW TRANSACTION ISOLATION LEVEL;
-- 输出: SERIALIZABLE

SHOW AUTO COMMIT;
-- 输出: T

SHOW CURRENT SCHEMA;
-- 输出: SYSDBA

-- 查看会话用户
SHOW session_user;
```

### 注意事项

- SHOW 为只读操作，不修改任何参数。
- 可用于查看通过 SET 设置的各类参数值。

---

## 与 Oracle/MySQL/PostgreSQL 的 DML 语法对比

### INSERT 对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 单行插入 | `INSERT INTO t VALUES(1)` | 相同 | 相同 | 相同 |
| 多行插入（逗号分隔） | `VALUES(1),(2),(3)` | 不支持（需 INSERT ALL） | 支持 | 支持 |
| 多行插入（无逗号） | `VALUES(1)(2)(3)` | 不支持 | 不支持 | 不支持 |
| 多表插入 INSERT ALL | 支持（兼容 Oracle 语法） | 支持 | 不支持 | 不支持 |
| 条件多表插入 INSERT FIRST | 支持 | 支持 | 不支持 | 不支持 |
| REPLACE INTO | 支持 | 不支持 | 支持 | 不支持（用 `INSERT ... ON CONFLICT`） |
| INSERT IGNORE | 支持 | 不支持 | 支持 | 不支持（用 `INSERT ... ON CONFLICT DO NOTHING`） |
| DEFAULT VALUES | 支持 | 不支持（需显式指定） | 支持 `INSERT INTO t() VALUES()` | 支持 |
| RETURNING 子句 | 支持 | 支持（PL/SQL） | 不支持 | 支持 |
| BULK COLLECT | 支持 | 支持 | 不支持 | 不支持 |

**关键差异说明：**
- 虚谷数据库独有的无逗号多行插入语法 `VALUES(1)(2)(3)` 是其他数据库均不支持的特有语法。
- 虚谷同时支持 Oracle 风格的 `INSERT ALL/FIRST` 多表插入和 MySQL 风格的 `REPLACE INTO`、`INSERT IGNORE`，兼容性较广。

### MERGE INTO 对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| MERGE INTO 语法 | 支持 | 支持 | 不支持 | 支持（v15+） |
| WHEN MATCHED THEN UPDATE | 支持 | 支持 | -- | 支持 |
| WHEN NOT MATCHED THEN INSERT | 支持 | 支持 | -- | 支持 |
| 子句顺序 | UPDATE/INSERT 顺序可互换 | UPDATE 在前 | -- | 类似 Oracle |
| 替代方案 | -- | -- | `INSERT ... ON DUPLICATE KEY UPDATE` | `INSERT ... ON CONFLICT` |

**关键差异说明：**
- 虚谷数据库的 MERGE INTO 语法与 Oracle 高度兼容。
- MySQL 不支持 MERGE INTO，需使用 `INSERT ... ON DUPLICATE KEY UPDATE` 替代。
- PostgreSQL v15 起原生支持 MERGE，之前版本需用 `INSERT ... ON CONFLICT` 替代。

### DELETE 对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 基本 DELETE | 支持 | 支持 | 支持 | 支持 |
| DELETE 不带 FROM | `DELETE table_name` 支持 | 不支持 | 支持 | 不支持 |
| 多表条件删除（opt_from_clause） | 支持 | 不支持 | 支持（JOIN 语法） | 支持（USING 语法） |
| RETURNING 子句 | 支持 | 支持（PL/SQL） | 不支持 | 支持 |
| WHERE CURRENT OF | 支持 | 支持 | 不支持 | 支持 |
| BULK COLLECT | 支持 | 支持 | 不支持 | 不支持 |

### UPDATE 对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 基本 UPDATE | 支持 | 支持 | 支持 | 支持 |
| 多表 UPDATE | 支持 | 支持（子查询方式） | 支持（JOIN 语法） | 支持（FROM 语法） |
| SET (col1,col2) = (SELECT ...) | 支持 | 支持 | 不支持 | 支持 |
| RETURNING 子句 | 支持 | 支持（PL/SQL） | 不支持 | 支持 |
| WHERE CURRENT OF | 支持 | 支持 | 不支持 | 支持 |
| ORDER BY / LIMIT | 不支持 | 不支持 | 支持 | 不支持（直接） |

### EXECUTE / PREPARE 对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| EXECUTE 调用存储过程 | `EXECUTE proc_name` | `EXECUTE proc_name`（仅 SQL*Plus） | `CALL proc_name` | `CALL proc_name`（v11+） |
| EXEC 简写 | 支持 | 支持（SQL*Plus） | 不支持 | 不支持 |
| CALL 等价 | 支持（CALL = EXECUTE） | 不等价 | CALL 为标准方式 | CALL 为标准方式 |
| PREPARE 语法 | `PREPARE name AS sql` | 不支持（使用游标） | `PREPARE name FROM sql` | `PREPARE name AS sql` |
| 执行 PREPARE | `?pre_name` | -- | `EXECUTE name USING @var` | `EXECUTE name(params)` |
| 释放 PREPARE | `DEALLOCATE name` | -- | `DEALLOCATE PREPARE name` | `DEALLOCATE name` |

### SET / SHOW 对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 设置系统参数 | `SET var TO value` | `ALTER SYSTEM SET var = value` | `SET GLOBAL var = value` | `ALTER SYSTEM SET var = value` |
| 设置会话参数 | `SET var TO value` | `ALTER SESSION SET var = value` | `SET SESSION var = value` | `SET var = value` |
| 自定义变量 | `SET @var = value` | 不支持（用 PL/SQL 变量） | `SET @var = value` | 不支持（用 PL/pgSQL 变量） |
| SHOW 变量 | `SHOW var_name` | 不支持（用 `SHOW PARAMETER`） | `SHOW VARIABLES` | `SHOW var_name` |
| SET NAMES | 支持 | 不支持 | 支持 | 支持（`SET client_encoding`） |
| SET NULL | 支持（空串转 NULL） | 内置行为（空串 = NULL） | 不支持 | 不支持 |
| SET SCHEMA | 支持 | `ALTER SESSION SET CURRENT_SCHEMA` | `USE database` | `SET search_path` |
