---
title: 虚谷数据库聚合函数与分析函数
description: 虚谷数据库（XuguDB）聚合函数（24个）和分析函数（8个）完整参考，包括 AVG、SUM、COUNT、GROUP_CONCAT、LISTAGG、STDDEV、RANK、ROW_NUMBER 等，与 Oracle/MySQL/PostgreSQL 对比
tags: [xugudb, aggregate, analytic, window-function, 聚合, 分析函数, 窗口函数]
---

# 虚谷数据库聚合函数与分析函数

虚谷数据库提供 24 个聚合函数和 8 个分析（窗口）函数，用于对数据集进行统计计算和排名分析。聚合函数将多行数据合并为单个结果值，分析函数在保留行级明细的同时计算聚合结果。

---

## 目录

1. [基本聚合函数](#1-基本聚合函数)
2. [位运算聚合函数](#2-位运算聚合函数)
3. [字符串聚合函数](#3-字符串聚合函数)
4. [JSON/XML 聚合函数](#4-jsonxml-聚合函数)
5. [统计聚合函数](#5-统计聚合函数)
6. [分析（窗口）函数](#6-分析窗口函数)
7. [与 Oracle/MySQL/PostgreSQL 对比](#7-与-oraclemysqlpostgresql-对比)

---

## 1. 基本聚合函数

用于对数值集进行求和、求平均、取最值、计数等基本统计运算。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `AVG([DISTINCT] expr)` | NUMERIC/DOUBLE | 返回表达式的平均值，忽略 NULL |
| `SUM([DISTINCT] expr)` | NUMERIC/DOUBLE | 返回表达式的总和，忽略 NULL |
| `MAX(expr)` | 与输入类型相同 | 返回表达式的最大值 |
| `MIN(expr)` | 与输入类型相同 | 返回表达式的最小值 |
| `MEDIAN(expr)` | NUMERIC/DOUBLE | 返回表达式的中位数（Oracle 兼容） |

> **注意**：`AVG` 和 `SUM` 支持 `DISTINCT` 关键字，仅对不重复值进行计算。所有聚合函数均忽略 NULL 值。

### SQL 示例

```sql
-- 基本聚合
SELECT
    COUNT(*) AS total_rows,
    AVG(salary) AS avg_salary,
    SUM(salary) AS total_salary,
    MAX(salary) AS max_salary,
    MIN(salary) AS min_salary
FROM employees;

-- 使用 DISTINCT
SELECT AVG(DISTINCT salary) AS avg_distinct_salary
FROM employees;

-- MEDIAN（Oracle 兼容）
SELECT MEDIAN(salary) AS median_salary
FROM employees;

-- 分组聚合
SELECT dept_id,
    COUNT(*) AS emp_count,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY dept_id
HAVING COUNT(*) > 5;

-- 在表达式中使用聚合函数
SELECT dept_id,
    SUM(CASE WHEN salary > 10000 THEN 1 ELSE 0 END) AS high_salary_count,
    ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY dept_id;
```

---

## 2. 位运算聚合函数

对数值列按位进行聚合运算。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `BIT_AND(expr)` | BIGINT | 返回所有值的按位与结果 |
| `BIT_OR(expr)` | BIGINT | 返回所有值的按位或结果 |
| `BIT_XOR(expr)` | BIGINT | 返回所有值的按位异或结果 |

### SQL 示例

```sql
-- 位运算聚合
SELECT
    BIT_AND(flags) AS all_flags_and,
    BIT_OR(flags) AS all_flags_or,
    BIT_XOR(flags) AS all_flags_xor
FROM permissions;

-- 实际场景：检查所有用户是否都具有某权限位
SELECT BIT_AND(permission_bits) & 4 AS all_have_write
FROM user_permissions
WHERE group_id = 1;
```

---

## 3. 字符串聚合函数

将多行字符串值合并为单个字符串。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `GROUP_CONCAT([DISTINCT] expr [ORDER BY ...] [SEPARATOR sep])` | VARCHAR | 将多行值连接为一个字符串（MySQL 兼容），默认分隔符为逗号 |
| `LISTAGG(expr [, delimiter]) WITHIN GROUP (ORDER BY ...)` | VARCHAR | 将多行值连接为有序字符串（Oracle 兼容） |
| `WM_CONCAT(expr)` | VARCHAR | 将多行值用逗号连接（Oracle 兼容，WMSYS.WM_CONCAT 简写） |

### SQL 示例

```sql
-- GROUP_CONCAT（MySQL 风格）
SELECT dept_id,
    GROUP_CONCAT(name ORDER BY name SEPARATOR ', ') AS employee_names
FROM employees
GROUP BY dept_id;
-- 结果示例: 1, '张三, 李四, 王五'

-- GROUP_CONCAT 去重
SELECT dept_id,
    GROUP_CONCAT(DISTINCT job_title ORDER BY job_title) AS job_titles
FROM employees
GROUP BY dept_id;

-- LISTAGG（Oracle 风格）
SELECT dept_id,
    LISTAGG(name, ', ') WITHIN GROUP (ORDER BY name) AS employee_names
FROM employees
GROUP BY dept_id;

-- WM_CONCAT（Oracle 兼容）
SELECT dept_id,
    WM_CONCAT(name) AS employee_names
FROM employees
GROUP BY dept_id;

-- 业务场景：生成 IN 列表
SELECT GROUP_CONCAT(DISTINCT dept_id) AS dept_list
FROM employees
WHERE salary > 10000;
-- 结果示例: 1,3,5
```

---

## 4. JSON/XML 聚合函数

将多行数据聚合为 JSON 或 XML 格式。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `JSON_ARRAYAGG(expr)` | JSON | 将多行值聚合为 JSON 数组 |
| `JSON_OBJECTAGG(key, value)` | JSON | 将多行键值对聚合为 JSON 对象 |
| `XMLAGG(xml_expr [ORDER BY ...])` | XML | 将多行 XML 片段聚合为单个 XML（Oracle 兼容） |

### SQL 示例

```sql
-- JSON_ARRAYAGG
SELECT dept_id,
    JSON_ARRAYAGG(name) AS names_json
FROM employees
GROUP BY dept_id;
-- 结果示例: ["张三","李四","王五"]

-- JSON_OBJECTAGG
SELECT JSON_OBJECTAGG(name, salary) AS salary_map
FROM employees
WHERE dept_id = 1;
-- 结果示例: {"张三":8000,"李四":9500,"王五":7200}

-- XMLAGG（Oracle 兼容）
SELECT dept_id,
    XMLAGG(XMLELEMENT("name", name) ORDER BY name) AS names_xml
FROM employees
GROUP BY dept_id;
-- 结果示例: <name>张三</name><name>李四</name><name>王五</name>
```

---

## 5. 统计聚合函数

用于计算标准差和方差等统计量。

### 5.1 标准差函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `STDDEV(expr)` | DOUBLE | 返回样本标准差（Oracle 兼容，等价于 STDDEV_SAMP） |
| `STDDEV_POP(expr)` | DOUBLE | 返回总体标准差 |
| `STDDEV_SAMP(expr)` | DOUBLE | 返回样本标准差 |
| `STDEV(expr)` | DOUBLE | 返回样本标准差（SQL Server 兼容，等价于 STDDEV_SAMP） |
| `STDEVP(expr)` | DOUBLE | 返回总体标准差（SQL Server 兼容，等价于 STDDEV_POP） |

### 5.2 方差函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `VARIANCE(expr)` | DOUBLE | 返回样本方差（Oracle 兼容，等价于 VAR_SAMP） |
| `VAR(expr)` | DOUBLE | 返回样本方差（SQL Server 兼容） |
| `VAR_POP(expr)` | DOUBLE | 返回总体方差 |
| `VAR_SAMP(expr)` | DOUBLE | 返回样本方差 |
| `VARP(expr)` | DOUBLE | 返回总体方差（SQL Server 兼容，等价于 VAR_POP） |

> **样本 vs 总体**：样本标准差/方差（SAMP）使用 N-1 作为分母，总体标准差/方差（POP）使用 N 作为分母。当数据量较大时两者差异很小。

### SQL 示例

```sql
-- 标准差
SELECT
    STDDEV(salary) AS sample_stddev,
    STDDEV_POP(salary) AS pop_stddev,
    STDDEV_SAMP(salary) AS sample_stddev2
FROM employees;

-- 方差
SELECT
    VARIANCE(salary) AS sample_variance,
    VAR_POP(salary) AS pop_variance,
    VAR_SAMP(salary) AS sample_variance2
FROM employees;

-- SQL Server 兼容写法
SELECT
    STDEV(salary) AS stdev_val,
    STDEVP(salary) AS stdevp_val,
    VAR(salary) AS var_val,
    VARP(salary) AS varp_val
FROM employees;

-- 分组统计
SELECT dept_id,
    COUNT(*) AS cnt,
    AVG(salary) AS avg_sal,
    STDDEV(salary) AS stddev_sal,
    VARIANCE(salary) AS var_sal
FROM employees
GROUP BY dept_id;

-- 业务场景：找出薪资偏离平均值超过 2 个标准差的员工
SELECT e.*
FROM employees e,
    (SELECT AVG(salary) AS avg_sal, STDDEV(salary) AS std_sal FROM employees) s
WHERE ABS(e.salary - s.avg_sal) > 2 * s.std_sal;
```

---

## 6. 分析（窗口）函数

分析函数在保留每行明细数据的同时，基于窗口（分区和排序）计算结果。使用 `OVER` 子句定义窗口。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `ROW_NUMBER() OVER(...)` | BIGINT | 为分区内每行分配唯一递增序号（不处理并列） |
| `RANK() OVER(...)` | BIGINT | 为分区内每行分配排名（并列时跳号，如 1,2,2,4） |
| `DENSE_RANK() OVER(...)` | BIGINT | 为分区内每行分配排名（并列时不跳号，如 1,2,2,3） |
| `NTILE(n) OVER(...)` | INTEGER | 将分区内的行均匀分为 n 组，返回组号 |
| `FIRST_VALUE(expr) OVER(...)` | 与输入类型相同 | 返回窗口内第一行的值 |
| `LAST_VALUE(expr) OVER(...)` | 与输入类型相同 | 返回窗口内最后一行的值 |
| `LAG(expr [, offset [, default]]) OVER(...)` | 与输入类型相同 | 返回当前行之前第 offset 行的值（默认 offset=1） |
| `LEAD(expr [, offset [, default]]) OVER(...)` | 与输入类型相同 | 返回当前行之后第 offset 行的值（默认 offset=1） |

### OVER 子句语法

```sql
function_name(...) OVER (
    [PARTITION BY partition_expr, ...]
    [ORDER BY order_expr [ASC|DESC], ...]
    [ROWS|RANGE BETWEEN frame_start AND frame_end]
)
```

- `PARTITION BY`：按指定列分区，类似 GROUP BY 但不折叠行
- `ORDER BY`：定义分区内的排序
- `ROWS/RANGE BETWEEN`：定义窗口帧范围

### SQL 示例

```sql
-- ROW_NUMBER：为每个部门的员工按薪资降序编号
SELECT
    dept_id,
    name,
    salary,
    ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
FROM employees;

-- RANK 与 DENSE_RANK 对比
SELECT
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank_val,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank_val
FROM employees;
-- 如果有两人并列第 2：
-- RANK:       1, 2, 2, 4, 5, ...
-- DENSE_RANK: 1, 2, 2, 3, 4, ...

-- NTILE：将员工按薪资分为 4 组（四分位数）
SELECT
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary) AS quartile
FROM employees;

-- LAG / LEAD：获取前后行数据
SELECT
    name,
    hire_date,
    salary,
    LAG(salary, 1, 0) OVER (ORDER BY hire_date) AS prev_salary,
    LEAD(salary, 1, 0) OVER (ORDER BY hire_date) AS next_salary
FROM employees;

-- FIRST_VALUE / LAST_VALUE
SELECT
    dept_id,
    name,
    salary,
    FIRST_VALUE(name) OVER (PARTITION BY dept_id ORDER BY salary DESC) AS highest_paid,
    LAST_VALUE(name) OVER (
        PARTITION BY dept_id ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_paid
FROM employees;

-- 业务场景：Top N（每个部门薪资前 3 名）
SELECT * FROM (
    SELECT
        dept_id,
        name,
        salary,
        ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
    FROM employees
) t
WHERE rn <= 3;

-- 业务场景：计算环比增长
SELECT
    month_date,
    revenue,
    LAG(revenue) OVER (ORDER BY month_date) AS prev_revenue,
    ROUND((revenue - LAG(revenue) OVER (ORDER BY month_date))
        / LAG(revenue) OVER (ORDER BY month_date) * 100, 2) AS growth_pct
FROM monthly_sales;

-- 业务场景：累计求和
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM orders;

-- 聚合函数作为窗口函数使用
SELECT
    dept_id,
    name,
    salary,
    AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY dept_id) AS diff_from_avg
FROM employees;
```

> **注意**：`LAST_VALUE` 默认窗口帧为 `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`，这意味着它实际返回的是当前行而非分区最后一行。若要获取分区最后一行的值，需显式指定 `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`。

---

## 7. 与 Oracle/MySQL/PostgreSQL 对比

### 7.1 基本聚合函数对比

| 函数 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `AVG` | 支持 | 支持 | 支持 | 支持 |
| `SUM` | 支持 | 支持 | 支持 | 支持 |
| `MAX` / `MIN` | 支持 | 支持 | 支持 | 支持 |
| `MEDIAN` | 支持 | 支持 | 不支持 | 不支持（需 `PERCENTILE_CONT`） |

### 7.2 位运算聚合对比

| 函数 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `BIT_AND` | 支持 | 不支持 | 支持 | `BIT_AND` |
| `BIT_OR` | 支持 | 不支持 | 支持 | `BIT_OR` |
| `BIT_XOR` | 支持 | 不支持 | 支持 | 不支持（需用表达式模拟） |

### 7.3 字符串聚合对比

| 函数 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `GROUP_CONCAT` | 支持 | 不支持 | 支持 | 不支持 |
| `LISTAGG` | 支持 | 支持（11g+） | 不支持 | 不支持 |
| `WM_CONCAT` | 支持 | 支持（已废弃） | 不支持 | 不支持 |
| `STRING_AGG` | -- | 不支持 | 不支持 | 支持 |

> **迁移提示**：Oracle 的 `LISTAGG` 和 MySQL 的 `GROUP_CONCAT` 在虚谷中均可直接使用。PostgreSQL 的 `STRING_AGG` 可用 `LISTAGG` 或 `GROUP_CONCAT` 替代。Oracle 已废弃的 `WM_CONCAT` 在虚谷中仍可使用，但建议迁移后改用 `LISTAGG`。

### 7.4 JSON/XML 聚合对比

| 函数 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `JSON_ARRAYAGG` | 支持 | 支持（12c+） | 支持（5.7+） | 支持（16+）/ `JSON_AGG`（9.3+） |
| `JSON_OBJECTAGG` | 支持 | 支持（12c+） | 支持（5.7+） | 支持（16+）/ `JSON_OBJECT_AGG`（9.3+） |
| `XMLAGG` | 支持 | 支持 | 不支持 | `XMLAGG` |

### 7.5 统计函数对比

| 函数 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `STDDEV` / `STDDEV_SAMP` | 支持 | 支持 | `STDDEV` / `STDDEV_SAMP`（8.0+） | 支持 |
| `STDDEV_POP` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `VARIANCE` / `VAR_SAMP` | 支持 | 支持 | `VARIANCE` / `VAR_SAMP`（8.0+） | 支持 |
| `VAR_POP` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `STDEV` / `STDEVP` | 支持（SQL Server 兼容） | 不支持 | 不支持 | 不支持 |
| `VAR` / `VARP` | 支持（SQL Server 兼容） | 不支持 | 不支持 | 不支持 |

### 7.6 分析（窗口）函数对比

| 函数 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `ROW_NUMBER` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `RANK` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `DENSE_RANK` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `NTILE` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `LAG` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `LEAD` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `FIRST_VALUE` | 支持 | 支持 | 支持（8.0+） | 支持 |
| `LAST_VALUE` | 支持 | 支持 | 支持（8.0+） | 支持 |

> **迁移提示**：分析函数语法在各主流数据库中高度一致（均遵循 SQL:2003 标准），虚谷数据库的分析函数可与 Oracle、MySQL 8.0+、PostgreSQL 直接互相迁移。需注意 MySQL 8.0 之前的版本不支持窗口函数，迁移旧版 MySQL 时需将窗口函数逻辑改写为子查询或临时变量方案。

### 7.7 虚谷数据库聚合函数独有优势

| 特性 | 说明 |
|------|------|
| 多语法兼容 | 同时支持 Oracle 的 `LISTAGG`/`MEDIAN`/`WM_CONCAT`、MySQL 的 `GROUP_CONCAT`、SQL Server 的 `STDEV`/`STDEVP`/`VAR`/`VARP` |
| 零改动迁移 | 从 Oracle/MySQL/PostgreSQL 迁移聚合函数和分析函数时，绝大多数语句无需修改 |
| MEDIAN 内置支持 | 直接提供 `MEDIAN` 聚合函数，无需像 PostgreSQL 那样使用 `PERCENTILE_CONT` 模拟 |
