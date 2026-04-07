---
title: 虚谷数据库查询语法参考
description: 虚谷数据库（XuguDB）SQL 查询语法完整参考，涵盖 SELECT、FROM、WHERE、GROUP BY、ORDER BY、WITH(CTE)、连接查询、子查询、集合查询、层次查询、分析函数、结果集限定、执行计划等功能
tags: [xugudb, sql, query, select, join, subquery, analytic, connect-by, explain]
---

# 虚谷数据库查询语法参考

## SELECT 基础

### 语法格式

```
SelectStmt ::= with_clauses? select_no_parens | select_with_parens

select_no_parens ::= simple_select
                   | select_clause sort_clause? opt_for_update_clause? opt_select_limit?

select_clause ::= simple_select | select_with_parens
```

### 参数说明

- `with_clauses`：可选公共表表达式（CTE），参见 WITH 子句
- `select_no_parens`：无外部括号的查询语句
- `select_with_parens`：带外部括号的查询语句，可用于嵌套查询
- `sort_clause`：可选排序子句，参见 ORDER BY
- `opt_for_update_clause`：可选的更新锁定子句
- `opt_select_limit`：可选的结果限制子句，参见 LIMIT

### Hint 提示

虚谷数据库支持通过 `/*+ HINT_TEXT */` 语法对查询进行优化提示：

```sql
/*+ FULL(table_name) */              -- 指定全表扫描
/*+ INDEX(table_name index_name) */  -- 指定使用某索引
/*+ NOINDEX(table_name index_name) */-- 指定不使用某索引
/*+ INDEX_JOIN(t1, t2) */            -- 指定索引连接
/*+ USE_HASH(t1, t2) */              -- 指定哈希连接
```

---

## FROM 子句

### 语法格式

```
opt_from_clause ::= 'FROM' from_list

from_list ::= table_ref ( ',' table_ref )*

table_ref ::= base_table_ref
            | select_with_parens alias_clause? opt_pivot_unpivot_clause
            | '(' relation_expr ')' alias_clause?
            | '(' joined_table ')' alias_clause
            | joined_table
            | 'TABLE' '(' table_name ')'
            | 'TABLE' select_with_parens
            | 'XMLTABLE' '(' xmltable_exprs ')' alias_clause?

base_table_ref ::= relation_expr alias_clause? opt_pivot_unpivot_clause

relation_expr ::= table_name ( opt_parti_clip_clause? | '@' dblink_name )
```

### 别名子句

```
alias_clause ::= 'AS'? ColumnName ( '(' name_list ')' )?
```

- `ColumnName`：指定表别名
- `name_list`：可选的列别名列表，从首列开始按序设置

### 数据来源

虚谷数据库 FROM 子句支持以下数据来源：

- 堆表（普通表）
- 子查询
- 表连接结果集
- XMLTABLE
- DBLink 远程表（`table_name@dblink_name`）
- PIVOT/UNPIVOT 结果集

### PIVOT/UNPIVOT 子句

PIVOT 将行旋转为列，UNPIVOT 将列旋转为行。

```sql
-- PIVOT 行转列示例
SELECT * FROM (
  SELECT region, product, quarter, sales_amount FROM tab_sales_orders
) PIVOT (
  SUM(sales_amount) FOR quarter IN ('Q1', 'Q2')
) AS pvt
ORDER BY region, product;

-- UNPIVOT 列转行示例
SELECT region, product, quarter, sales_amount
FROM tab_pivoted_sales
UNPIVOT (
  sales_amount FOR quarter IN (Q1, Q2)
) AS unpvt
ORDER BY region, product, quarter;
```

> **注意**：PIVOT XML 功能暂未支持；不支持使用多参数的 `GROUP_CONCAT()` 聚合函数，推荐使用 `LISTAGG()`；聚合函数的参数只支持按名传参。

---

## WHERE 子句

### 语法格式

```
opt_where_clause ::= 'WHERE' bool_expr
```

### 谓词说明

| 谓词 | 说明 |
|------|------|
| `=, >, <, <>, !=, >=, <=` | 比较谓词，不可用于 NULL 比较 |
| `[NOT] LIKE ... [ESCAPE ...]` | 模糊匹配，ESCAPE 用于转义通配符 `%` 和 `_` |
| `BETWEEN ... AND ...` | 范围匹配，等同于 `>=` 和 `<=` 联合使用 |
| `IS [NOT] NULL` | 判断字段是否为 NULL，唯一可用于 NULL 判断的谓词 |
| `[NOT] IN` | 字段值与列表中的常量值进行匹配 |
| `[NOT] EXISTS` | 判断子查询是否存在数据 |

### 示例

```sql
-- 比较谓词
SELECT * FROM tab_pre WHERE id > 3;

-- LIKE 与 ESCAPE
SELECT * FROM tab_escape WHERE cc LIKE '\%%' ESCAPE '\';

-- EXISTS
SELECT * FROM tab_pre WHERE EXISTS (SELECT * FROM tab_pre WHERE id = 1);
```

---

## GROUP BY 子句

### 语法格式

```
opt_group_clause ::= 'GROUP' 'BY' group_item ( ',' group_item )*

group_item ::= b_expr
             | 'ROLLUP' '(' group_item ( ',' group_item )* ')'
             | 'CUBE' '(' group_item ( ',' group_item )* ')'
             | 'GROUPING' 'SETS' '(' group_item ( ',' group_item )* ')'
             | '(' ( b_expr ( ',' b_expr )* )? ')'
```

### 扩展功能

| 扩展 | 说明 | 示例分组 |
|------|------|----------|
| `ROLLUP(a,b)` | n+1 次分组，从右至左递减字段 | `(a,b)`, `(a)`, `()` |
| `CUBE(a,b)` | 按字段排列组合分组 | `(a,b)`, `(a)`, `(b)`, `()` |
| `GROUPING SETS(a,b)` | 仅返回每个字段的分组小计 | `(a)`, `(b)` |

### HAVING 子句

```
opt_having_clause ::= 'HAVING' bool_expr
```

HAVING 作用于分组之后，可结合聚合函数使用；WHERE 作用于分组之前，不可使用聚合函数。

### 示例

```sql
-- ROLLUP
SELECT department, gender, COUNT(*) rollup_cou
FROM tab_grp2
GROUP BY ROLLUP(department, gender);

-- CUBE
SELECT department, gender, COUNT(*) cube_cou
FROM tab_grp2
GROUP BY CUBE(department, gender);

-- GROUPING SETS
SELECT department, gender, COUNT(*) group_set_cou
FROM tab_grp2
GROUP BY GROUPING SETS(department, gender);

-- HAVING 过滤
SELECT department, COUNT(*) AS emp_count
FROM tab_having
GROUP BY department
HAVING COUNT(*) > 2;
```

---

## ORDER BY 子句

### 语法格式

```
sort_clause ::= 'ORDER' 'SIBLINGS'? 'BY' sortby ( ',' sortby )*

sortby ::= sort_ident ( 'ASC' | 'DESC' )? ( 'NULLS' ( 'FIRST' | 'LAST' ) )?

sort_ident ::= column_name | column_alias_name | target_index_number
```

### 参数说明

- `ASC`：升序（默认）
- `DESC`：降序
- `NULLS FIRST`：NULL 值排在结果集最前面
- `NULLS LAST`：NULL 值排在结果集最后面
- `SIBLINGS`：对同一父节点下的兄弟节点排序，与层次查询（CONNECT BY）配合使用

### 示例

```sql
-- 默认升序
SELECT * FROM tab_odr ORDER BY id;

-- 降序
SELECT * FROM tab_odr ORDER BY id DESC;
```

---

## WITH 子句（CTE）

### 语法格式

```
with_clauses ::= 'WITH' ( with_name 'AS' select_with_parens | ProcDef ';' )
                 ( ',' with_name 'AS' select_with_parens )*

with_name ::= ColId ( '(' name_list ')' )?
```

### 功能说明

- CTE（公共表表达式）是存在于查询中的临时表，生命周期随当前语句
- 多个 CTE 只使用一个 WITH，中间用逗号分隔
- 支持 `WITH FUNCTION` 和 `WITH PROCEDURE` 定义临时函数和临时存储过程
- WITH ProcDef 定义的存储过程/函数在对象名解析时拥有更高优先级

> **注意**：WITH ProcDef 暂不支持多个存储过程/函数。

### 示例

```sql
-- 单个 CTE
WITH with1 AS (SELECT 'abc' FROM dual)
SELECT * FROM with1;

-- 多个 CTE
WITH with1 AS (SELECT 'one' FROM dual),
     with2 AS (SELECT 'two' FROM dual)
SELECT * FROM with1 UNION SELECT * FROM with2;

-- WITH FUNCTION（临时函数）
WITH FUNCTION with_function(p_id IN NUMBER) RETURN NUMBER IS
BEGIN
  RETURN p_id;
END;
SELECT with_function(id) FROM tab_with_func WHERE rownum = 1;

-- WITH PROCEDURE（临时存储过程）
WITH PROCEDURE with_procedure(p_id IN NUMBER) IS
BEGIN
  DBMS_OUTPUT.put_line('p_id=' || p_id);
END;
SELECT id FROM tab_with_func WHERE rownum = 1;
```

---

## 连接查询（JOIN）

### 语法格式

```
joined_table ::= '(' joined_table ')'
               | table_ref 'CROSS' 'JOIN' table_ref
               | table_ref 'NATURAL' join_type? 'JOIN' table_ref
               | table_ref 'UNIONJOIN' table_ref
               | table_ref join_type? 'JOIN' table_ref 'USING' '(' name_list ')'
               | table_ref join_type? 'JOIN' table_ref 'ON' bool_expr

join_type ::= ( 'FULL' | 'LEFT' | 'RIGHT' ) 'OUTER'? | 'INNER'
```

### 连接类型

| 类型 | 关键字 | 说明 |
|------|--------|------|
| 内连接 | `[INNER] JOIN ... ON` | 返回满足条件的所有行 |
| 等值连接 | `JOIN ... USING(col)` | 使用同名列连接，结果中只出现一列 |
| 自然连接 | `NATURAL JOIN` | 自动按同名字段等值连接，无同名字段则返回笛卡尔积 |
| 左外连接 | `LEFT [OUTER] JOIN` | 返回左表所有行，右表无匹配则填 NULL |
| 右外连接 | `RIGHT [OUTER] JOIN` | 返回右表所有行，左表无匹配则填 NULL |
| 全外连接 | `FULL [OUTER] JOIN` | 返回左右表所有行，无匹配的一方填 NULL |
| 交叉连接 | `CROSS JOIN` | 笛卡尔积，结果行数 = 左表行数 x 右表行数 |
| 联合连接 | `UNIONJOIN` | 无条件联合两表，对一方每行另一方字段全为 NULL |

### 示例

```sql
-- 等值连接
SELECT * FROM tab_pre t1 JOIN tab_pre1 t2 ON t1.id = t2.id;

-- USING 格式等值连接
SELECT * FROM tab_pre t1 JOIN tab_pre1 t2 USING(id);

-- 自然连接
SELECT * FROM tab_pre t1 NATURAL JOIN tab_pre1 t2;

-- 不等值连接
SELECT * FROM tab_pre t1 INNER JOIN tab_pre1 t2 ON t1.id > t2.id;

-- 左外连接
SELECT * FROM tab_out t1 LEFT JOIN tab_out1 t2 ON t1.id = t2.id;

-- 右外连接
SELECT * FROM tab_out t1 RIGHT JOIN tab_out1 t2 ON t1.id = t2.id;

-- 全外连接
SELECT * FROM tab_out t1 FULL JOIN tab_out1 t2 ON t1.id = t2.id;

-- 交叉连接
SELECT * FROM tab_out t1 CROSS JOIN tab_out1 t2;
```

---

## 子查询

### 子查询类型

| 类型 | 结果形式 | 说明 |
|------|----------|------|
| 标量子查询 | 单行单列 | 可作为返回字段或 WHERE 条件 |
| 列子查询 | 单列多行 | 配合 IN、ANY、SOME、ALL 使用 |
| 行子查询 | 单行多列 | 配合元组比较，如 `(a, b) = (subquery)` |
| 表子查询 | 多行多列 | 常用于 FROM 子句中，相当于临时表 |

### 关键字说明

- `IN`：查询字段值是否在子查询结果集中
- `ANY` / `SOME`：子查询结果中任意一个值满足比较条件即可
- `ALL`：子查询结果中所有值都必须满足条件
- `EXISTS`：判断子查询是否返回数据（true/false）

### 限制

- WHERE 中子查询返回多行须使用 `EXISTS`、`IN`、`ANY`、`ALL` 修饰
- 比较谓词、`BETWEEN`、`LIKE` 引入的子查询必须返回单行单列
- 多个子查询中列名相同时，必须对列引用加表名或别名

### 示例

```sql
-- 标量子查询（作为返回字段）
SELECT id, name, (SELECT MAX(score) FROM tab_sub_query) AS max_score
FROM tab_sub_query;

-- 列子查询 IN
SELECT id, name, score FROM tab_sub_query
WHERE id IN (SELECT id FROM tab_sub_query WHERE score > 40);

-- 列子查询 ANY
SELECT id, name, score FROM tab_sub_query
WHERE id > ANY(SELECT id FROM tab_sub_query WHERE score < 50);

-- 列子查询 ALL
SELECT id, name, score FROM tab_sub_query
WHERE id > ALL(SELECT id FROM tab_sub_query WHERE score < 50);

-- 行子查询
SELECT id, name, score FROM tab_sub_query
WHERE (id, name) = (SELECT id, name FROM tab_sub_query WHERE score = 50);

-- 表子查询（FROM 子句中）
SELECT name, rnk FROM (
  SELECT id, name, RANK() OVER(ORDER BY score) AS rnk FROM tab_sub_query
);

-- EXISTS
SELECT * FROM tab_sub_query
WHERE EXISTS(SELECT id FROM tab_sub_query);
```

---

## 集合查询（UNION / INTERSECT / EXCEPT）

### 集合操作

| 操作 | 关键字 | 说明 |
|------|--------|------|
| 并集（去重） | `UNION` | 合并两个查询结果并去重 |
| 并集（不去重） | `UNION ALL` | 合并两个查询结果，保留重复行 |
| 交集 | `INTERSECT` | 取两个查询结果的交集 |
| 差集 | `MINUS` / `EXCEPT` | 取两个查询结果的差集 |

> **要求**：集合关键字两边的查询必须含有相同字段数，且对应字段的数据类型必须相同或可隐式转换。返回结果的字段名以左边查询为准。

### 示例

```sql
-- 交集
SELECT id, deptno, sal FROM tab_2
INTERSECT
SELECT id, col1, col2 FROM tab_1;

-- 并集去重
SELECT id, deptno, sal FROM tab_2
UNION
SELECT id, col1, col2 FROM tab_1;

-- 并集不去重
SELECT id, deptno, sal FROM tab_2
UNION ALL
SELECT id, col1, col2 FROM tab_1;

-- 差集（MINUS 兼容 Oracle，EXCEPT 兼容 MySQL）
SELECT id, deptno, sal FROM tab_2
MINUS
SELECT id, col1, col2 FROM tab_1;

SELECT id, deptno, sal FROM tab_2
EXCEPT
SELECT id, col1, col2 FROM tab_1;
```

---

## 层次查询（CONNECT BY）

### 语法格式

```
opt_connect_by ::= connect_by?

connect_by ::= 'CONNECT' 'BY' 'NOCYCLE'? bool_expr 'START' 'WITH' bool_expr
             | 'START' 'WITH' bool_expr 'CONNECT' 'BY' 'NOCYCLE'? bool_expr
             opt_keep_clause

opt_keep_clause ::= ( 'KEEP' bool_expr )?
```

### 参数说明

- `CONNECT BY bool_expr`：层次结构的连接条件，使用 `PRIOR` 运算符引用父行
- `START WITH bool_expr`：层次结构的根节点（起点）
- `NOCYCLE`：防止循环引用导致无限递归
- `KEEP bool_expr`：在层次聚合操作中保留满足条件的特定行

### 相关伪列

| 伪列 | 说明 |
|------|------|
| `LEVEL` | 当前节点在树中的层级，从 1 开始 |
| `CONNECT_BY_ISLEAF` | 是否为叶子节点（1=是，0=否） |
| `CONNECT_BY_ISCYCLE` | 是否会产生环（1=是，0=否） |

### 示例

```sql
-- 创建测试表
CREATE TABLE tab_employees(
  emp_id INT,
  emp_name VARCHAR2(50),
  manager_id INT
);

INSERT INTO tab_employees VALUES
  (1, 'CEO', NULL)(2, 'CTO', 1)(3, 'CFO', 1)
  (4, 'Dev1', 2)(5, 'Dev2', 2)(6, 'Accountant', 3);

-- 从根节点向下查询（自顶向下）
SELECT LEVEL AS level,
       CONNECT_BY_ISLEAF AS isleaf,
       CONNECT_BY_ISCYCLE AS iscycle,
       emp_id, emp_name, manager_id
FROM tab_employees
START WITH manager_id IS NULL
CONNECT BY PRIOR emp_id = manager_id;

-- 从子节点向上查询（自底向上）
SELECT LEVEL AS level,
       CONNECT_BY_ISLEAF AS isleaf,
       CONNECT_BY_ISCYCLE AS iscycle,
       emp_id, emp_name, manager_id
FROM tab_employees
START WITH emp_id = 6
CONNECT BY PRIOR manager_id = emp_id;
```

---

## 分析函数（窗口函数）

### 语法格式

```
over_clause ::= analytic_function '(' arguments ')' 'OVER' '(' analytic_clause ')'

analytic_clause ::= opt_win_parti? opt_win_order? opt_win_range?

opt_win_parti ::= 'PARTITION' 'BY' expr ( ',' expr )*

opt_win_order ::= 'ORDER' 'BY' expr ( 'ASC' | 'DESC' )? ( 'NULLS' ( 'FIRST' | 'LAST' ) )?

opt_win_range ::= ( 'ROWS' | 'RANGE' ) ( range_with_between | range_without_between )
```

### 子句说明

#### PARTITION BY（分组子句）

将数据按指定字段分组，每个分组独立计算。省略此子句则将所有行作为单个组处理。

```sql
SUM(salary) OVER (PARTITION BY department_id)
```

#### ORDER BY（排序子句）

定义分区内的排序规则。使用注意事项：

- `DENSE_RANK`、`NTILE`、`RANK`：多行值相同时返回相同结果
- `ROW_NUMBER`：为每行分配不同值，排序不唯一时结果不确定
- 在分析函数中使用时，ORDER BY 必须接受表达式，位置和列别名无效

#### 窗口帧（ROWS / RANGE）

| 关键字 | 说明 |
|--------|------|
| `ROWS` | 基于物理行号的偏移 |
| `RANGE` | 基于排序字段值的逻辑偏移 |
| `UNBOUNDED PRECEDING` | 分区第一行 |
| `UNBOUNDED FOLLOWING` | 分区最后一行 |
| `CURRENT ROW` | 当前行 |
| `value_expr PRECEDING` | 当前行前 N 行/值 |
| `value_expr FOLLOWING` | 当前行后 N 行/值 |

> 默认窗口范围为 `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`。

#### ROWS 与 RANGE 的常用窗口帧

```sql
-- 整个分区
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

-- 从分区开始到当前行
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- 当前行前后各 N 行
ROWS BETWEEN N PRECEDING AND N FOLLOWING

-- 从当前行到分区末尾
ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING
```

### 示例

```sql
-- 分区内行计数
SELECT name, age,
  COUNT(age) OVER(
    PARTITION BY age ORDER BY age
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS cnt
FROM tab_over_test
ORDER BY age;

-- RANK 排名
SELECT *,
  RANK() OVER(PARTITION BY deptno ORDER BY sal) rk
FROM tab_sal_info
ORDER BY deptno, rk;
```

---

## 结果集限定（LIMIT / TOP / DISTINCT）

### LIMIT

#### 语法格式

```
opt_select_limit ::= select_limit?

select_limit ::= 'LIMIT' ( select_offset_value ',' select_limit_value
               | select_limit_value ( 'OFFSET' select_offset_value )? )
```

#### 参数说明

- `select_limit_value`：指定返回最大行数，可为数字、`ALL`（返回全部）或参数占位符
- `select_offset_value`：指定跳过的行数（偏移量）
- 当前版本 LIMIT 最大支持到 **2147483647**

#### 示例

```sql
-- 返回前 2 行
SELECT * FROM tab_limit LIMIT 2;

-- 跳过前 2 行，返回 2 行（两种写法等价）
SELECT * FROM tab_limit LIMIT 2, 2;
SELECT * FROM tab_limit LIMIT 2 OFFSET 2;

-- 子查询中使用 LIMIT
SELECT * FROM tab_limit
WHERE id = (SELECT id FROM tab_limit LIMIT 10, 1);
```

### TOP

限制查询结果返回前 N 行。

```sql
SELECT TOP 2 * FROM tab_top ORDER BY id DESC;
```

### DISTINCT

去除 SELECT 查询中的重复记录。

```sql
SELECT DISTINCT id FROM tab_dis;
```

---

## 执行计划（EXPLAIN）

### 语法格式

```
explain_stmt ::= 'EXPLAIN' 'VERBOSE'? dml_stmt

dml_stmt ::= ( SelectStmt | UpdateStmt | InsertStmt | DeleteStmt | MergeIntoStmt )
             parallel_opt? opt_wait? dml_opt?

dml_opt ::= 'WITH' ( 'NO' | 'NOT' ) ( 'ROLLBACK' | 'UNDO' )
```

### 参数说明

- `VERBOSE`：输出更详细的执行计划
- `dml_stmt`：要查看执行计划的 DML 语句
- `parallel_opt`：执行并发数
- `opt_wait`：执行等待超时时间
- `dml_opt`：事务控制选项（不记录回滚日志）

### 执行计划算子

| 算子 | 说明 |
|------|------|
| `LimitScan` | 限制结果行数的扫描器 |
| `Sort` | 排序扫描器 |
| `HashGroup` | 哈希分组扫描器 |
| `IndexJoin` | 索引连接扫描器 |
| `BtIdxScan` | B 树索引扫描器 |
| `SeqScan` | 顺序扫描器 |

### 执行计划输出字段

| 字段 | 说明 |
|------|------|
| 最左侧序号 | 执行计划树的层级 |
| `[(步骤序号 左右支)]` | 圆括号内左值为执行步骤序号，右值 1=左支，2=右支，右值小的先执行 |
| `cost` | 当前算子的预估执行代价 |
| `result_num` | 预估返回行数 |
| `table` | 扫描的表名称 |
| `index` | 使用的索引名称 |
| `join_filter` | 连接过滤条件 |
| `scan_filter` | 扫描过滤条件 |

### 示例

```sql
EXPLAIN VERBOSE
SELECT /*+INDEX(a idx_hint)*/ COUNT(*)
FROM tab_hint_1 a INNER JOIN tab_hint_2 b ON a.id = b.id
WHERE a.id = 3
GROUP BY a.name
ORDER BY a.name
LIMIT 10;
```

输出示例：

```
1 LimitScan[(9 1) cost=0,result_num=1]
2   Sort[(7 1) cost=0,result_num=1]
3     HashGroup[(5 1) cost=0,result_num=1]
4       IndexJoin[(4 1) cost=210,result_num=2]
5         BtIdxScan[(2 1) cost=300,result_num=1](table=TAB_HINT_1)(index=IDX_HINT)
6         SeqScan[(1 2) cost=0,result_num=1](table=TAB_HINT_2)
-----------------------Tips--------------------------
4 join_filter: (TAB_HINT_1.ID)=(TAB_HINT_2.ID)
5 scan_filter: (TAB_HINT_1.ID)=(3)
6 scan_filter: (TAB_HINT_2.ID)=(3)
```

---

## 与 Oracle / MySQL / PostgreSQL 的查询语法对比

### 层次查询（CONNECT BY）

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| CONNECT BY | 支持 | 支持 | 不支持 | 不支持 |
| START WITH | 支持 | 支持 | 不支持 | 不支持 |
| NOCYCLE | 支持 | 支持 | 不支持 | 不支持 |
| LEVEL 伪列 | 支持 | 支持 | 不支持 | 不支持 |
| CONNECT_BY_ISLEAF | 支持 | 支持 | 不支持 | 不支持 |
| CONNECT_BY_ISCYCLE | 支持 | 支持 | 不支持 | 不支持 |
| KEEP 子句 | 支持 | 支持 | 不支持 | 不支持 |
| ORDER SIBLINGS BY | 支持 | 支持 | 不支持 | 不支持 |
| 递归 CTE（WITH RECURSIVE） | 参见 WITH 子句 | 支持（11g+） | 支持（8.0+） | 支持 |

> 虚谷数据库的 CONNECT BY 语法与 Oracle 兼容。MySQL 和 PostgreSQL 需使用递归 CTE 实现同等功能。

### 结果集限定

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `LIMIT n` | 支持 | 不支持 | 支持 | 支持 |
| `LIMIT n OFFSET m` | 支持 | 不支持 | 支持 | 支持 |
| `LIMIT m, n` | 支持 | 不支持 | 支持 | 不支持 |
| `TOP n` | 支持 | 不支持 | 不支持 | 不支持 |
| `ROWNUM` | 支持 | 支持 | 不支持 | 不支持 |
| `FETCH FIRST n ROWS ONLY` | 参见文档 | 支持（12c+） | 不支持 | 支持 |

> 虚谷数据库同时兼容 MySQL 风格的 `LIMIT` 语法和 Oracle 风格的 `ROWNUM`，还支持类似 SQL Server 的 `TOP` 语法。LIMIT 最大值为 2147483647。

### 集合操作

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `UNION` / `UNION ALL` | 支持 | 支持 | 支持 | 支持 |
| `INTERSECT` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `MINUS` | 支持 | 支持 | 不支持 | 不支持 |
| `EXCEPT` | 支持 | 不支持 | 支持（8.0+） | 支持 |

> 虚谷数据库同时支持 `MINUS`（兼容 Oracle）和 `EXCEPT`（兼容 MySQL/PostgreSQL），两者功能等价。

### 分析函数

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| OVER 子句 | 支持 | 支持 | 支持（8.0+） | 支持 |
| PARTITION BY | 支持 | 支持 | 支持（8.0+） | 支持 |
| ROWS 窗口帧 | 支持 | 支持 | 支持（8.0+） | 支持 |
| RANGE 窗口帧 | 支持 | 支持 | 支持（8.0+） | 支持 |
| RANK / DENSE_RANK | 支持 | 支持 | 支持（8.0+） | 支持 |
| ROW_NUMBER | 支持 | 支持 | 支持（8.0+） | 支持 |
| NTILE | 支持 | 支持 | 支持（8.0+） | 支持 |

> 虚谷数据库的分析函数语法与 Oracle 基本一致。MySQL 8.0 之前版本不支持窗口函数。

### PIVOT / UNPIVOT

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| PIVOT | 支持 | 支持（11g+） | 不支持 | 不支持（需 crosstab 扩展） |
| UNPIVOT | 支持 | 支持（11g+） | 不支持 | 不支持 |

> 虚谷数据库的 PIVOT/UNPIVOT 语法与 Oracle 兼容。MySQL 和 PostgreSQL 需使用 CASE 表达式或扩展模块实现。

### WITH 子句

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| WITH ... AS（CTE） | 支持 | 支持 | 支持（8.0+） | 支持 |
| WITH FUNCTION | 支持 | 支持（12c+） | 不支持 | 不支持 |
| WITH PROCEDURE | 支持 | 支持（12c+） | 不支持 | 不支持 |

> 虚谷数据库支持与 Oracle 12c+ 兼容的 WITH FUNCTION / WITH PROCEDURE 语法，可在查询内临时定义函数或存储过程。

### Hint 提示

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `/*+ HINT */` 语法 | 支持 | 支持 | 支持（部分） | 不支持（需 pg_hint_plan 扩展） |
| FULL 全表扫描提示 | 支持 | 支持 | 不支持 | 不支持 |
| INDEX 索引提示 | 支持 | 支持 | 支持（USE INDEX） | 不支持 |
| USE_HASH 哈希连接提示 | 支持 | 支持 | 不支持 | 不支持 |

> 虚谷数据库的 Hint 语法与 Oracle 风格一致，采用 `/*+ ... */` 注释形式。
