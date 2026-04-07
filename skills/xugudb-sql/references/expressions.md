---
title: 虚谷数据库表达式
description: 常量表达式、标识符表达式、算术表达式、函数调用表达式、类型转换表达式、条件表达式、逻辑表达式、子查询表达式语法及示例，与 Oracle/MySQL/PostgreSQL 对比
tags: xugudb, expression, cast, case-when, subquery, like, between, in, is-null, arithmetic, constant, identifier, type-cast
---

# 虚谷数据库表达式

表达式是 SQL 中用于计算值的基本构造。虚谷数据库支持多种表达式类型，可用于 SELECT 列表、WHERE 条件、ORDER BY 排序以及各种 SQL 子句中。表达式可以互相嵌套组合，构成复杂的计算逻辑。

---

## 常量表达式

常量表达式是直接书写在 SQL 中的固定值，不依赖任何表或列。

### 数值常量

整数和浮点数可直接书写：

```sql
SELECT 42;
SELECT 3.14;
SELECT -100;
SELECT 1.23E+10;
```

### 字符串常量

字符串使用**单引号**括起：

```sql
SELECT '虚谷数据库';
SELECT 'Hello, XuguDB';
```

字符串中包含单引号时，使用两个连续单引号转义：

```sql
SELECT 'it''s a test';
```

> **MySQL 兼容模式**：虚谷数据库在 MySQL 兼容模式下支持双引号作为字符串定界符。在标准模式下，双引号用于标识符引用（类似 PostgreSQL）。

### 布尔常量

```sql
SELECT TRUE;
SELECT FALSE;
```

### 日期时间常量

通过 `::` 类型转换语法将字符串转换为日期时间类型：

```sql
SELECT '2024-01-15'::DATE;
SELECT '2024-01-15 10:30:00'::DATETIME;
SELECT '10:30:00'::TIME;
SELECT '2024-01-15 10:30:00'::TIMESTAMP;
```

也可以使用 DATE、TIME、TIMESTAMP 关键字前缀：

```sql
SELECT DATE '2024-01-15';
SELECT TIMESTAMP '2024-01-15 10:30:00';
```

### NULL 常量

```sql
SELECT NULL;
```

---

## 标识符表达式

标识符用于引用数据库对象（列、表、模式等）。虚谷数据库支持多级限定名称。

### 语法格式

```
column_name
table_name.column_name
schema_name.table_name.column_name
```

### SQL 示例

```sql
-- 直接引用列名
SELECT name FROM employees;

-- 表名限定列名（多表查询时消除歧义）
SELECT e.name, d.dept_name
FROM employees e, departments d
WHERE e.dept_id = d.id;

-- 模式名限定
SELECT SYSDBA.employees.name FROM SYSDBA.employees;
```

### 别名

可以使用 `AS` 关键字（可省略）为列或表定义别名：

```sql
SELECT name AS employee_name, salary * 12 AS annual_salary
FROM employees e;
```

### 标识符引用

包含特殊字符或与关键字冲突的标识符使用双引号括起：

```sql
SELECT "select", "column-name" FROM "my table";
```

---

## 算术表达式

算术表达式使用数学运算符对数值进行计算。

### 语法格式

```
expr + expr    -- 加法
expr - expr    -- 减法
expr * expr    -- 乘法
expr / expr    -- 除法
- expr         -- 取负
( expr )       -- 括号改变优先级
```

运算符优先级（从高到低）：括号 `()` > 取负 `-` > 乘除 `* /` > 加减 `+ -`。

### SQL 示例

```sql
-- 基本算术
SELECT 10 + 20;
SELECT salary * 1.1 FROM employees;

-- 组合运算
SELECT (price * quantity) - discount AS total
FROM orders;

-- 字符串连接使用 || 运算符
SELECT first_name || ' ' || last_name AS full_name
FROM employees;
```

---

## 函数调用表达式

函数调用表达式是通过函数名加参数列表的形式调用系统函数或用户自定义函数。

### 语法格式

```
function_name(arg1, arg2, ...)
function_name()                   -- 无参数函数
```

### SQL 示例

```sql
-- 字符串函数
SELECT UPPER('hello'), LENGTH('虚谷');

-- 数学函数
SELECT ABS(-10), ROUND(3.1415, 2);

-- 日期函数
SELECT NOW(), CURRENT_DATE;

-- 聚合函数
SELECT COUNT(*), SUM(salary), AVG(salary)
FROM employees;

-- 嵌套函数调用
SELECT UPPER(SUBSTR(name, 1, 3)) FROM employees;

-- 用户自定义函数
SELECT my_schema.my_function(col1, col2) FROM my_table;
```

---

## 类型转换表达式

虚谷数据库支持两种类型转换语法：标准 SQL 的 `CAST` 函数和 PostgreSQL 风格的 `::` 运算符。

### 语法格式

```
-- CAST 语法（SQL 标准）
CAST(expr AS data_type)

-- :: 语法（PostgreSQL 风格）
expr::data_type
```

### SQL 示例

```sql
-- CAST 语法
SELECT CAST(123 AS VARCHAR);
SELECT CAST('2024-01-15' AS DATE);
SELECT CAST(salary AS NUMERIC(10,2)) FROM employees;

-- :: 语法
SELECT 123::VARCHAR;
SELECT '2024-01-15'::DATE;
SELECT salary::NUMERIC(10,2) FROM employees;

-- 在表达式中使用类型转换
SELECT '100'::INTEGER + 200;
SELECT CAST(AVG(salary) AS NUMERIC(10,2)) FROM employees;
```

### 常用类型转换

| 源类型 | 目标类型 | 示例 |
|--------|----------|------|
| 字符串 | 整数 | `'123'::INTEGER` |
| 字符串 | 日期 | `'2024-01-15'::DATE` |
| 字符串 | 时间戳 | `'2024-01-15 10:30:00'::TIMESTAMP` |
| 数值 | 字符串 | `123::VARCHAR` |
| 浮点数 | 定点数 | `3.14::NUMERIC(5,2)` |
| 整数 | 浮点数 | `100::FLOAT` |

### 与其他数据库对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| CAST 语法 | 支持 | 支持 | 支持 | 支持 |
| :: 语法 | 支持 | 不支持 | 不支持 | 支持 |
| 隐式转换 | 部分支持 | 较宽松 | 较宽松 | 较严格 |

> **迁移提示**：从 Oracle/MySQL 迁移到虚谷时，`CAST` 语法可直接使用。虚谷额外支持 `::` 语法，与 PostgreSQL 兼容，在书写上更简洁。

---

## 条件表达式 (CASE WHEN)

CASE 表达式用于在 SQL 中实现条件分支逻辑，类似编程语言中的 if-else。虚谷数据库支持两种形式。

### 搜索式 CASE

根据多个布尔条件依次判断，返回第一个条件为真的结果值。

#### 语法格式

```
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ELSE default_result]
END
```

#### SQL 示例

```sql
SELECT name,
    CASE
        WHEN salary >= 10000 THEN '高'
        WHEN salary >= 5000  THEN '中'
        ELSE '低'
    END AS salary_level
FROM employees;
```

### 简单式 CASE

将一个表达式与多个候选值比较，返回匹配值对应的结果。

#### 语法格式

```
CASE expr
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    [ELSE default_result]
END
```

#### SQL 示例

```sql
SELECT name,
    CASE dept_id
        WHEN 1 THEN '研发部'
        WHEN 2 THEN '市场部'
        WHEN 3 THEN '财务部'
        ELSE '其他'
    END AS dept_name
FROM employees;
```

### CASE 在不同子句中的使用

```sql
-- 在 WHERE 子句中
SELECT * FROM orders
WHERE CASE WHEN type = 'VIP' THEN amount > 100 ELSE amount > 500 END;

-- 在 ORDER BY 子句中
SELECT * FROM employees
ORDER BY CASE WHEN dept_id = 1 THEN 0 ELSE 1 END, name;

-- 在 UPDATE 中
UPDATE employees
SET salary = CASE
    WHEN performance = 'A' THEN salary * 1.2
    WHEN performance = 'B' THEN salary * 1.1
    ELSE salary
END;
```

### 与其他数据库对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 搜索式 CASE | 支持 | 支持 | 支持 | 支持 |
| 简单式 CASE | 支持 | 支持 | 支持 | 支持 |
| DECODE 函数 | 支持 | 支持 | 不支持 | 不支持 |
| IF 函数 | MySQL 兼容模式支持 | 不支持 | 支持 | 不支持 |

> **迁移提示**：CASE WHEN 语法在各数据库中基本一致，迁移时通常无需修改。Oracle 的 `DECODE` 函数在虚谷中也受支持。MySQL 的 `IF(condition, true_val, false_val)` 函数在虚谷 MySQL 兼容模式下可用。

---

## 逻辑表达式

逻辑表达式用于构造 WHERE 和 HAVING 子句中的过滤条件，返回布尔值。

### 比较运算符

```
expr = expr       -- 等于
expr <> expr      -- 不等于（也支持 !=）
expr > expr       -- 大于
expr < expr       -- 小于
expr >= expr      -- 大于等于
expr <= expr      -- 小于等于
```

```sql
SELECT * FROM employees WHERE salary > 5000;
SELECT * FROM employees WHERE dept_id <> 3;
SELECT * FROM employees WHERE name = '张三';
```

### BETWEEN 范围判断

#### 语法格式

```
expr [NOT] BETWEEN low AND high
```

等价于 `expr >= low AND expr <= high`。

```sql
SELECT * FROM employees WHERE salary BETWEEN 5000 AND 10000;
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01'::DATE AND '2024-12-31'::DATE;
```

### IN 集合判断

#### 语法格式

```
expr [NOT] IN (value1, value2, ...)
expr [NOT] IN (subquery)
```

```sql
SELECT * FROM employees WHERE dept_id IN (1, 2, 3);
SELECT * FROM employees WHERE dept_id NOT IN (SELECT id FROM inactive_depts);
```

### IS NULL 空值判断

#### 语法格式

```
expr IS [NOT] NULL
```

```sql
SELECT * FROM employees WHERE manager_id IS NULL;
SELECT * FROM employees WHERE email IS NOT NULL;
```

> **注意**：不能使用 `= NULL` 或 `<> NULL` 来判断空值，必须使用 `IS NULL` / `IS NOT NULL`。

### LIKE 模糊匹配

#### 语法格式

```
expr [NOT] LIKE pattern [ESCAPE escape_char]
```

- `%`：匹配零个或多个任意字符
- `_`：匹配单个任意字符

```sql
-- 以"张"开头的名字
SELECT * FROM employees WHERE name LIKE '张%';

-- 名字第二个字为"明"
SELECT * FROM employees WHERE name LIKE '_明%';

-- 包含"数据"的描述
SELECT * FROM employees WHERE description LIKE '%数据%';

-- ESCAPE 转义（查找包含 % 的内容）
SELECT * FROM products WHERE name LIKE '%10\%%' ESCAPE '\';
```

### 逻辑运算符

```
condition AND condition    -- 逻辑与
condition OR condition     -- 逻辑或
NOT condition              -- 逻辑非
```

优先级：`NOT` > `AND` > `OR`。使用括号可以改变运算顺序。

```sql
SELECT * FROM employees
WHERE dept_id = 1 AND salary > 5000;

SELECT * FROM employees
WHERE dept_id = 1 OR dept_id = 2;

SELECT * FROM employees
WHERE NOT (salary < 3000);

-- 组合使用（括号明确优先级）
SELECT * FROM employees
WHERE (dept_id = 1 OR dept_id = 2) AND salary > 5000;
```

### IS TRUE / IS FALSE 布尔判断

#### 语法格式

```
expr IS [NOT] TRUE
expr IS [NOT] FALSE
expr IS [NOT] UNKNOWN
```

```sql
SELECT * FROM tasks WHERE is_completed IS TRUE;
SELECT * FROM tasks WHERE is_completed IS NOT FALSE;
```

---

## 子查询表达式

子查询表达式是嵌套在其他 SQL 语句中的 SELECT 查询。虚谷数据库支持多种子查询形式。

### 标量子查询

返回单行单列值，可用于 SELECT 列表或 WHERE 条件中。

```sql
-- 在 SELECT 列表中
SELECT name,
    (SELECT dept_name FROM departments WHERE id = e.dept_id) AS dept_name
FROM employees e;

-- 在 WHERE 条件中
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

### IN 子查询

判断值是否在子查询结果集中。

```sql
SELECT * FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE location = '北京');

SELECT * FROM employees
WHERE dept_id NOT IN (SELECT id FROM departments WHERE is_active = FALSE);
```

### EXISTS 子查询

判断子查询是否返回至少一行结果。

```sql
-- 查找有员工的部门
SELECT * FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.dept_id = d.id);

-- 查找没有订单的客户
SELECT * FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id);
```

### UNIQUE 子查询

判断子查询返回结果中是否有重复行。

```sql
SELECT * FROM departments d
WHERE UNIQUE (SELECT dept_id FROM employees WHERE dept_id = d.id);
```

### ANY / ALL 子查询

与比较运算符组合使用，对子查询结果集进行批量比较。

#### 语法格式

```
expr operator ANY (subquery)    -- 满足任意一个即为 TRUE
expr operator ALL (subquery)    -- 满足全部才为 TRUE
```

```sql
-- =ANY 等价于 IN
SELECT * FROM employees
WHERE salary = ANY (SELECT salary FROM managers);

-- >ALL：大于子查询中的所有值
SELECT * FROM employees
WHERE salary > ALL (SELECT salary FROM interns);

-- <>ALL 等价于 NOT IN
SELECT * FROM employees
WHERE dept_id <> ALL (SELECT id FROM inactive_depts);
```

### 表子查询

子查询作为临时表（派生表）使用。

```sql
SELECT t.dept_name, t.avg_salary
FROM (
    SELECT d.dept_name, AVG(e.salary) AS avg_salary
    FROM departments d
    JOIN employees e ON d.id = e.dept_id
    GROUP BY d.dept_name
) t
WHERE t.avg_salary > 8000;
```

### 与其他数据库对比

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 标量子查询 | 支持 | 支持 | 支持 | 支持 |
| IN 子查询 | 支持 | 支持 | 支持 | 支持 |
| EXISTS 子查询 | 支持 | 支持 | 支持 | 支持 |
| ANY/ALL 子查询 | 支持 | 支持 | 支持 | 支持 |
| UNIQUE 子查询 | 支持 | 不支持 | 不支持 | 不支持 |
| 关联子查询 | 支持 | 支持 | 支持 | 支持 |

> **迁移提示**：子查询语法在各数据库中高度一致，迁移时通常无需修改。虚谷额外支持 UNIQUE 子查询，这是 SQL 标准中的功能，但大多数数据库未实现。

---

## 表达式组合与优先级

多种表达式可以自由组合。以下是运算符优先级（从高到低）：

| 优先级 | 运算符/表达式 | 说明 |
|--------|--------------|------|
| 1 | `()` | 括号 |
| 2 | `::` | 类型转换 |
| 3 | `-`（一元） | 取负 |
| 4 | `* /` | 乘除 |
| 5 | `+ -` | 加减 |
| 6 | `\|\|` | 字符串连接 |
| 7 | `= <> > < >= <=` | 比较 |
| 8 | `IS, IS NULL, LIKE, BETWEEN, IN` | 特殊比较 |
| 9 | `NOT` | 逻辑非 |
| 10 | `AND` | 逻辑与 |
| 11 | `OR` | 逻辑或 |

### 组合示例

```sql
-- 算术 + 类型转换 + 条件 + 子查询的组合
SELECT
    name,
    CAST(salary * 1.1 AS NUMERIC(10,2)) AS new_salary,
    CASE
        WHEN salary > (SELECT AVG(salary) FROM employees) THEN '高于平均'
        ELSE '低于平均'
    END AS salary_rank
FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE location = '北京')
    AND hire_date BETWEEN '2020-01-01'::DATE AND '2024-12-31'::DATE
    AND name LIKE '张%'
ORDER BY new_salary DESC;
```
