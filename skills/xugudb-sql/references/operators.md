---
title: 虚谷数据库运算符参考
description: 虚谷数据库（XuguDB）所有运算符的完整参考，包括赋值、连接、JSON、位、几何、日期时间、比较、算术和逻辑运算符。
tags: [xugudb, operators, sql, 运算符, 参考]
---

# 虚谷数据库运算符参考

虚谷数据库支持九大类运算符，部分运算符在不同数据类型上下文中具有多重语义（如 `=` 可作为赋值、比较和几何运算符，`+` 可作为算术、日期时间和几何运算符）。

> **提示：** 可在数据库中执行 `SELECT * FROM sys_operators WHERE NAME = '<运算符>';` 查看完整的操作数据类型及返回类型。

---

## 目录

1. [赋值运算符](#1-赋值运算符)
2. [连接运算符](#2-连接运算符)
3. [JSON 运算符](#3-json-运算符)
4. [算术运算符](#4-算术运算符)
5. [比较运算符](#5-比较运算符)
6. [逻辑运算符](#6-逻辑运算符)
7. [位运算符](#7-位运算符)
8. [日期时间运算符](#8-日期时间运算符)
9. [几何运算符](#9-几何运算符)
10. [与 Oracle/MySQL/PostgreSQL 运算符对比](#10-与-oraclemysqlpostgresql-运算符对比)

---

## 1. 赋值运算符

### `=`（赋值）

**功能描述：** 将右侧的值存储到左侧的变量或列中。`=` 不仅可作为赋值运算符，亦可作为比较运算符和几何运算符。

**语法格式：**

```sql
L_OPERAND_TYP = R_OPERAND_TYP
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| L_OPERAND_TYP | 任意有效变量类型或者表中存在的列 |
| R_OPERAND_TYP | 所需存入变量或更新数据库字段 |

**输出结果：** 无输出参数。

**示例：**

```sql
-- 使用 UPDATE 赋值修改表中存在的列 age
SQL> CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50), age INT, email VARCHAR(100));
SQL> INSERT INTO users (id, name, age, email) VALUES (1, '张三', 25, 'zhangsan@example.com');
SQL> UPDATE users SET age = 26 WHERE name = '张三';
SQL> SELECT * FROM users;
+----+------+-----+----------------------+
| ID | NAME | AGE | EMAIL                |
+----+------+-----+----------------------+
|  1 | 张三 |  26 | zhangsan@example.com |
+----+------+-----+----------------------+
```

---

## 2. 连接运算符

### `||`（字符串连接）

**功能描述：** 字符串连接运算符，用于将两个或多个字符串值拼接成一个字符串。

**语法格式：**

```sql
L_OPERAND_TYP || R_OPERAND_TYP
```

**输入参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| L_OPERAND_TYP | VARCHAR | VARCHAR 类型，或能隐式转换为 VARCHAR 类型的其他类型的值或表达式 |
| R_OPERAND_TYP | VARCHAR | VARCHAR 类型，或能隐式转换为 VARCHAR 类型的其他类型的值或表达式。输入参数为 NULL 视为空字符 |

**输出结果：** VARCHAR 类型。

**示例：**

```sql
-- 连接 2019 和 ABC 字符串
SQL> SELECT 2019 || 'ABC' FROM DUAL;
+---------+
| EXPR1   |
+---------+
| 2019ABC |
+---------+

-- 连接 NULL 和 ABC 字符串（NULL 视为空字符）
SQL> SELECT NULL || 'ABC' FROM DUAL;
+---------+
| EXPR1   |
+---------+
| ABC     |
+---------+

-- 连接多个字符串，返回类型为 VARCHAR
SQL> SELECT 'xxx' || 'db' || ' and ' || 'xxx' FROM DUAL;
+---------------+
| EXPR1         |
+---------------+
| xxxdb and xxx |
+---------------+
```

---

## 3. JSON 运算符

### `->`（返回 JSON 数据）

**功能描述：** 返回对应路径数据，结果保留 JSON 格式。

**语法格式：**

```sql
L_OPERAND_TYP -> R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| JSON        | CHAR        | JSON    |

**输出结果：** CHAR 类型。若任意输入参数为 NULL，则输出结果为 NULL。

**示例：**

```sql
-- 查询 JSON 数组 [1,2,"abc"] 中的第 3 个元素（索引从 0 开始），输出为 JSON 格式
SQL> SELECT '[1,2,"abc"]' -> '$[2]';
+-------+
| EXPR1 |
+-------+
| "abc" |
+-------+
```

### `->>`（返回路径数据文本）

**功能描述：** 返回对应路径数据并取消对 JSON 类型的引用（去掉引号）。

**语法格式：**

```sql
L_OPERAND_TYP ->> R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| JSON        | CHAR        | CHAR    |

**输出结果：** CHAR 类型。若任意输入参数为 NULL，则输出结果为 NULL。

**示例：**

```sql
-- 查询 JSON 数组 [1,2,"abc"] 中的第 3 个元素（索引从 0 开始），将其转换为 VARCHAR
SQL> SELECT '[1,2,"abc"]' ->> '$[2]';
+-------+
| EXPR1 |
+-------+
| abc   |
+-------+
```

---

## 4. 算术运算符

### `+`（加法）

**功能描述：** 对数字类型的值进行加法计算。当没有左操作数时，可作为一元正号运算符。`+` 亦可作为日期时间运算符和几何运算符。

**语法格式：**

```sql
L_OPERAND_TYP + R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| INTEGER     | INTEGER     | INTEGER |
| DATETIME    | DATETIME    | NUMERIC |
| FLOAT       | FLOAT       | DOUBLE  |

**输出结果：** 类型根据输入参数类型决定。若任意输入参数为 NULL，则输出结果为 NULL。

**示例：**

```sql
-- 整数加法，返回类型为 INTEGER
SQL> SELECT 5 + 2;
+-------+
| EXPR1 |
+-------+
|     7 |
+-------+

-- 浮点数运算，返回类型为 DOUBLE
SQL> SELECT 1.5::FLOAT + 1.1::FLOAT;
+---------------+
| EXPR1         |
+---------------+
| 2.600000e+00  |
+---------------+

-- 一元正号运算符
SQL> SELECT +5;
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+
```

### `-`（减法）

**功能描述：** 对数字类型的值进行减法计算。当没有左操作数时，可作为一元负号运算符。`-` 亦可作为几何运算符。

**语法格式：**

```sql
L_OPERAND_TYP - R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| INTEGER     | INTEGER     | INTEGER |
| DATETIME    | DATETIME    | NUMERIC |
| FLOAT       | FLOAT       | DOUBLE  |

**示例：**

```sql
-- 整数减法，返回类型为 INTEGER
SQL> SELECT 10 - 2;
+-------+
| EXPR1 |
+-------+
|     8 |
+-------+

-- 浮点数运算，返回类型为 DOUBLE
SQL> SELECT 1.2::FLOAT - 1.1::FLOAT;
+---------------+
| EXPR1(DOUBLE) |
+---------------+
| 1.000000e-01  |
+---------------+

-- 一元负号运算符
SQL> SELECT -5;
+-------+
| EXPR1 |
+-------+
|    -5 |
+-------+
```

### `*`（乘法）

**功能描述：** 对数字类型的值进行乘法计算。

**语法格式：**

```sql
L_OPERAND_TYP * R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| NUMERIC     | NUMERIC     | NUMERIC |
| BIGINT      | INTEGER     | NUMERIC |
| DOUBLE      | FLOAT       | DOUBLE  |

**示例：**

```sql
-- 整数乘法，返回类型为 BIGINT
SQL> SELECT 5 * 2;
+-------+
| EXPR1 |
+-------+
|    10 |
+-------+

-- 浮点数运算，返回类型为 DOUBLE
SQL> SELECT 1.5::FLOAT * 1.1::FLOAT;
+---------------+
| EXPR1         |
+---------------+
| 1.650000e+00  |
+---------------+
```

### `/`（除法）

**功能描述：** 对数字类型的值进行除法计算。除数不能为 0。

**语法格式：**

```sql
L_OPERAND_TYP / R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| NUMERIC     | NUMERIC     | NUMERIC |
| DOUBLE      | BIGINT      | BIGINT  |
| DOUBLE      | FLOAT       | DOUBLE  |

**示例：**

```sql
-- 除法运算，返回类型为 DOUBLE
SQL> SELECT 5 / 2;
+--------------+
| EXPR1        |
+--------------+
| 2.500000e+00 |
+--------------+

-- 浮点数运算，返回类型为 DOUBLE
SQL> SELECT 1.5::FLOAT / 1.1::FLOAT;
+---------------+
| EXPR1         |
+---------------+
| 1.363636e+00  |
+---------------+

-- 除数为 0 报错
SQL> SELECT 10 / 0;
Error: [E19005 L1 C9] 除数为0
```

---

## 5. 比较运算符

### `=`（等于）

**功能描述：** 比较两个值的相等关系，判断左操作数是否等于右操作数。`=` 亦可作为几何运算符和赋值运算符。

**语法格式：**

```sql
L_OPERAND_TYP = R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| CHAR        | CHAR        | BOOLEAN |
| TINYINT     | TINYINT     | BOOLEAN |
| FLOAT       | FLOAT       | BOOLEAN |
| BIGINT      | BIGINT      | BOOLEAN |

**输出结果：** BOOLEAN 类型。

**示例：**

```sql
-- 隐式转换为数字比较
SQL> SELECT 10 = '3.14';
+-------+
| EXPR1 |
+-------+
| F     |
+-------+

-- 日期比较
SQL> SELECT DATE('2023-01-15') = DATE('2023-01-01');
+-------+
| EXPR1 |
+-------+
| F     |
+-------+
```

### `<>`（不等于）

**功能描述：** 比较两个值的不相等关系，判断左操作数是否不等于右操作数。不等于同样支持 `!=` 写法。

**语法格式：**

```sql
L_OPERAND_TYP <> R_OPERAND_TYP
-- 或
L_OPERAND_TYP != R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| CHAR        | CHAR        | BOOLEAN |
| TINYINT     | TINYINT     | BOOLEAN |
| FLOAT       | FLOAT       | BOOLEAN |
| BIGINT      | BIGINT      | BOOLEAN |

**示例：**

```sql
-- 隐式转换为数字比较
SQL> SELECT 10 <> '3.14';
+-------+
| EXPR1 |
+-------+
| T     |
+-------+

-- 日期比较
SQL> SELECT DATE('2023-01-15') <> DATE('2023-01-01');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

### `<`（小于）

**功能描述：** 比较两个值的大小关系，判断左操作数是否小于右操作数。

**语法格式：**

```sql
L_OPERAND_TYP < R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| CHAR        | CHAR        | BOOLEAN |
| TINYINT     | TINYINT     | BOOLEAN |
| FLOAT       | FLOAT       | BOOLEAN |
| BIGINT      | BIGINT      | BOOLEAN |

**示例：**

```sql
SQL> SELECT 10 < '3.14';
+-------+
| EXPR1 |
+-------+
| F     |
+-------+
```

### `<=`（小于或等于）

**功能描述：** 比较两个值的小于或等于关系。

**语法格式：**

```sql
L_OPERAND_TYP <= R_OPERAND_TYP
```

**输入输出类型：** 同 `<` 运算符，返回 BOOLEAN 类型。

**示例：**

```sql
SQL> SELECT 10 <= '3.14';
+-------+
| EXPR1 |
+-------+
| F     |
+-------+
```

### `>`（大于）

**功能描述：** 比较两个值的大小关系，判断左操作数是否大于右操作数。

**语法格式：**

```sql
L_OPERAND_TYP > R_OPERAND_TYP
```

**输入输出类型：** 同 `<` 运算符，返回 BOOLEAN 类型。

**示例：**

```sql
SQL> SELECT 10 > '3.14';
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

### `>=`（大于或等于）

**功能描述：** 比较两个值的大于或等于关系。

**语法格式：**

```sql
L_OPERAND_TYP >= R_OPERAND_TYP
```

**输入输出类型：** 同 `<` 运算符，返回 BOOLEAN 类型。

**示例：**

```sql
SQL> SELECT 10 >= '3.14';
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

---

## 6. 逻辑运算符

### `AND`（逻辑与）

**功能描述：** 连接两个或多个条件，要求所有条件必须同时为真，结果才为真。

**语法格式：**

```sql
L_OPERAND_TYP AND R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| BOOLEAN     | BOOLEAN     | BOOLEAN |

**输出结果：** BOOLEAN 类型。输入参数不能为 NULL。

**示例：**

```sql
SQL> SELECT TRUE AND FALSE;
+---------+
| EXPR1   |
+---------+
| F       |
+---------+
```

### `OR`（逻辑或）

**功能描述：** 用于连接多个条件，只要至少有一个条件为真，整个表达式即为真。

**语法格式：**

```sql
L_OPERAND_TYP OR R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| BOOLEAN     | BOOLEAN     | BOOLEAN |

**输出结果：** BOOLEAN 类型。输入参数不能为 NULL。

**示例：**

```sql
SQL> SELECT TRUE OR FALSE;
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

### `NOT`（逻辑非）

**功能描述：** 取反布尔表达式的值。

**语法格式：**

```sql
NOT R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| 无          | BOOLEAN     | BOOLEAN |

**输出结果：** BOOLEAN 类型。输入参数不能为 NULL。

**示例：**

```sql
SQL> SELECT NOT FALSE;
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

---

## 7. 位运算符

### `&`（按位与）

**功能描述：** 将两个表示为二进制操作数进行按位与计算。

**语法格式：**

```sql
L_OPERAND_TYP & R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| VARBIT      | VARBIT      | VARBIT  |
| BIGINT      | BIGINT      | BIGINT  |

**输出结果：** 类型根据输入参数类型决定。若任意输入参数为 NULL，则输出结果为 NULL。

**示例：**

```sql
-- 对二进制 VARBIT 类型字符串进行按位与运算
SQL> SELECT b'101' & b'1011';
+---------+
| EXPR1   |
+---------+
| b'0001' |
+---------+

-- 对 BIGINT 类型数据进行按位与运算
SQL> SELECT 9223372036854775800 & 9223372036854775705;
+---------------------+
| EXPR1               |
+---------------------+
| 9223372036854775704 |
+---------------------+
```

### `|`（按位或）

**功能描述：** 将两个表示为二进制操作数进行按位或计算。

> **提示：** 若两个操作数为 VARBIT 类型且位数不等时，对较短的操作数左侧补零，再执行计算。

**语法格式：**

```sql
L_OPERAND_TYP | R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| VARBIT      | VARBIT      | VARBIT  |
| BIGINT      | BIGINT      | BIGINT  |

**示例：**

```sql
-- 对二进制 VARBIT 类型字符串进行按位或运算
SQL> SELECT b'101' | b'1011';
+---------+
| EXPR1   |
+---------+
| b'1111' |
+---------+

-- 浮点数显式转换后按位或运算，返回类型为 BIGINT
SQL> SELECT 1.5::FLOAT | 1.1::FLOAT;
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+
```

### `^`（按位异或）

**功能描述：** 将两个表示为二进制操作数进行按位异或计算。

**语法格式：**

```sql
L_OPERAND_TYP ^ R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| VARBIT      | VARBIT      | VARBIT  |
| BIGINT      | BIGINT      | BIGINT  |

**示例：**

```sql
-- 对二进制 VARBIT 类型字符串进行按位异或运算
SQL> SELECT b'101' ^ b'1011';
+---------+
| EXPR1   |
+---------+
| b'1110' |
+---------+

-- 对 BIGINT 类型数据进行按位异或运算
SQL> SELECT 9223372036854775800 ^ 9223372036854775705;
+-------+
| EXPR1 |
+-------+
|    97 |
+-------+
```

### `~`（按位非）

**功能描述：** 将二进制数的所有位执行取反操作。

**语法格式：**

```sql
~ R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| 无          | BIGINT      | BIGINT  |
| 无          | VARBIT      | VARBIT  |

**示例：**

```sql
-- 对二进制 VARBIT 类型字符串进行按位非运算
SQL> SELECT ~ b'101';
+--------+
| EXPR1  |
+--------+
| b'010' |
+--------+

-- 对 BIGINT 类型数据进行按位非运算
SQL> SELECT ~ 1;
+-------+
| EXPR1 |
+-------+
|    -2 |
+-------+
```

### `<<`（位左移）

**功能描述：** 将二进制数的所有位向左移动指定的位数。`<<` 亦可作为几何运算符（位左侧判断）。

> **提示：** 当移位数为负数时，将往反方向移位。

**语法格式：**

```sql
L_OPERAND_TYP << R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| VARBIT      | INTEGER     | VARBIT  |
| BIGINT      | BIGINT      | BIGINT  |

**示例：**

```sql
-- 对二进制 VARBIT 类型字符串向左移动 2 位
SQL> SELECT b'1011' << 2;
+---------+
| EXPR1   |
+---------+
| b'1100' |
+---------+

-- 对 BIGINT 类型数据向左移动 2 位
SQL> SELECT 1 << 2;
+-------+
| EXPR1 |
+-------+
|     4 |
+-------+
```

### `>>`（位右移）

**功能描述：** 将二进制数的所有位向右移动指定的位数。`>>` 亦可作为几何运算符（位右侧判断）。

> **提示：** 当移位数为负数时，将往反方向移位。

**语法格式：**

```sql
L_OPERAND_TYP >> R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| VARBIT      | INTEGER     | VARBIT  |
| BIGINT      | BIGINT      | BIGINT  |

**示例：**

```sql
-- 对二进制 VARBIT 类型字符串向右移动 2 位
SQL> SELECT b'1011' >> 2;
+---------+
| EXPR1   |
+---------+
| b'0010' |
+---------+

-- 对 BIGINT 类型数据向右移动 2 位
SQL> SELECT 1 >> 2;
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+
```

---

## 8. 日期时间运算符

### `+`（日期加法）

**功能描述：** 对日期类型进行加法计算操作。

**语法格式：**

```sql
L_OPERAND_TYP + R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| DATE        | TIME        | DATETIME |
| DATE        | INTERVAL DAY | DATE |
| DATE        | INTERVAL YEAR TO MONTH | DATE |
| DATE        | INTERVAL HOUR | DATETIME |
| DATE        | INTERVAL DAY TO SECOND | DATETIME |
| DATE        | INTERVAL DAY TO MINUTE | DATETIME |
| DATE        | DOUBLE      | DATETIME |
| DATETIME    | INTERVAL DAY TO SECOND | DATETIME |
| DATETIME    | INTERVAL YEAR TO MONTH | DATETIME |
| DATETIME    | INTERVAL HOUR | DATETIME |
| DATETIME    | INTERVAL HOUR TO SECOND | DATETIME |
| DATETIME    | DOUBLE      | DATETIME |
| DATETIME    | INTERVAL DAY TO MINUTE | DATETIME |
| DATETIME    | INTERVAL DAY | DATETIME |

**示例：**

```sql
-- 计算当前时间再过 7 天后的时间
SQL> SELECT CURRENT_DATE + INTERVAL '7' DAY;
+-------------+
| EXPR1       |
+-------------+
| 2025-07-19  |
+-------------+

-- 计算当前时间再过 2 天 3 小时 30 分钟后的时间
SQL> SELECT CURRENT_TIMESTAMP + INTERVAL '2 03:30:00' DAY TO SECOND;
+--------------------------+
| EXPR1                    |
+--------------------------+
| 2025-07-14 19:32:29.148  |
+--------------------------+

-- 计算当前时间再过 30 天后的时间（DOUBLE 类型加法）
SQL> SELECT CURRENT_TIMESTAMP + 30;
+--------------------------+
| EXPR1                    |
+--------------------------+
| 2025-08-11 16:18:44.628  |
+--------------------------+
```

### `-`（日期减法）

**功能描述：** 对日期类型进行减法计算操作。

**语法格式：**

```sql
L_OPERAND_TYP - R_OPERAND_TYP
```

**输入输出类型（常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| DATE        | TIME        | DATETIME |
| DATE        | INTERVAL DAY | DATE |
| DATE        | INTERVAL YEAR TO MONTH | DATE |
| DATE        | INTERVAL HOUR | DATETIME |
| DATE        | INTERVAL DAY TO SECOND | DATETIME |
| DATE        | INTERVAL DAY TO MINUTE | DATETIME |
| DATE        | DOUBLE      | DATETIME |
| DATETIME    | INTERVAL DAY TO SECOND | DATETIME |
| DATETIME    | INTERVAL YEAR TO MONTH | DATETIME |
| DATETIME    | INTERVAL HOUR | DATETIME |
| DATETIME    | INTERVAL HOUR TO SECOND | DATETIME |
| DATETIME    | DOUBLE      | DATETIME |
| DATETIME    | INTERVAL DAY TO MINUTE | DATETIME |
| DATETIME    | INTERVAL DAY | DATETIME |

**示例：**

```sql
-- 计算当前时间往前 7 天
SQL> SELECT CURRENT_DATE - INTERVAL '7' DAY;
+-------------+
| EXPR1       |
+-------------+
| 2025-07-05  |
+-------------+

-- 计算当前时间往前 2 天 3 小时 30 分钟
SQL> SELECT CURRENT_TIMESTAMP - INTERVAL '2 03:30:00' DAY TO SECOND;
+--------------------------+
| EXPR1                    |
+--------------------------+
| 2025-07-10 12:48:15.533  |
+--------------------------+

-- 计算当前时间往前 30 天
SQL> SELECT CURRENT_TIMESTAMP - 30;
+--------------------------+
| EXPR1                    |
+--------------------------+
| 2025-06-12 16:18:27.533  |
+--------------------------+
```

### 日期时间比较运算符

日期时间类型支持以下比较运算符：`=`、`<>`（也支持 `!=`）、`<`、`<=`、`>`、`>=`。

**支持的日期时间类型：**

| 数据类型 |
|---------|
| DATE |
| DATETIME |
| DATETIME WITH TIME ZONE |
| TIME |
| TIME WITH TIME ZONE |
| INTERVAL DAY |
| INTERVAL YEAR TO MONTH |

**示例：**

```sql
-- 日期等于比较
SQL> SELECT DATE('2023-01-15') = DATE('2023-01-01');
+-------+
| EXPR1 |
+-------+
| F     |
+-------+

-- 日期大于比较
SQL> SELECT DATE('2023-01-15') > DATE('2023-01-01');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+

-- 时间间隔值比较
SQL> SELECT INTERVAL '1 12:00:00' DAY TO SECOND <> INTERVAL '1 06:00:00' DAY TO SECOND;
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

---

## 9. 几何运算符

虚谷数据库提供了丰富的几何运算符，用于对几何对象（POINT、BOX、CIRCLE、LINE、LSEG、PATH、POLYGON）进行操作。

### 9.1 几何平移与变换

#### `+`（连接/平移）

**功能描述：** 对几何对象执行坐标平移，将第二个参数坐标值叠加到第一个对象的坐标属性上。适用于 POINT、BOX、CIRCLE。将两条打开的路径连接为一条连续路径（若任意一条路径为闭合状态则返回 NULL），适用于 PATH。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | POINT   |
| BOX         | POINT       | BOX     |
| PATH        | POINT       | PATH    |
| CIRCLE      | POINT       | CIRCLE  |
| PATH        | PATH        | PATH    |

**示例：**

```sql
-- POINT 点执行几何平移
SQL> SELECT POINT('(2.0,0)') + POINT('(0,2.0)');
+-------+
| EXPR1 |
+-------+
| (2,2) |
+-------+

-- BOX 矩形执行几何平移
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) + POINT('(0,2.0)');
+-------------+
| EXPR1       |
+-------------+
| (1,3),(0,2) |
+-------------+

-- PATH 连接两个打开的路径
SQL> SELECT PATH('[(0,0),(1,1)]') + PATH('[(0,0),(-1,-1)]');
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| [(0,0),(1,1),(0,0),(-1,-1)] |
+-----------------------------+
```

#### `-`（平移）

**功能描述：** 对几何对象执行坐标平移，将第二个参数坐标值从第一个对象的坐标属性移除指定向量。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | POINT   |
| BOX         | POINT       | BOX     |
| PATH        | POINT       | PATH    |
| CIRCLE      | POINT       | CIRCLE  |

**示例：**

```sql
-- POINT 平移操作
SQL> SELECT POINT('(2.0,0)') - POINT('(0,2.0)');
+--------+
| EXPR1  |
+--------+
| (2,-2) |
+--------+

-- BOX 矩形执行几何平移
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) - POINT('(0,2.0)');
+---------------+
| EXPR1         |
+---------------+
| (1,-1),(0,-2) |
+---------------+
```

#### `*`（缩放旋转）

**功能描述：** 将几何对象的每个坐标点与给定的 POINT 参数进行复数乘法运算，实现缩放与旋转的组合变换。

> **提示：** 将点视为由实部和虚部组成的复数，再进行乘法运算。若将第二个 POINT 解释为向量，则该操作等同于将对象按向量的长度进行缩放，并以其与 x 轴的夹角为基准绕原点进行逆时针旋转。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | POINT   |
| BOX         | POINT       | BOX     |
| PATH        | POINT       | PATH    |
| CIRCLE      | POINT       | CIRCLE  |

**示例：**

```sql
-- POINT 复数乘法运算
SQL> SELECT POINT('(2.0,0)') * POINT('(0,2.0)');
+-------+
| EXPR1 |
+-------+
| (0,4) |
+-------+

-- PATH 几何旋转 45 度
SQL> SELECT PATH('((0,0),(1,0),(1,1))') * point(cosd(45), sind(45));
+---------------------------------------------------------------------+
| EXPR1                                                               |
+---------------------------------------------------------------------+
| ((0,0),(0.7071067811865475,0.7071067811865475),(0,1.414213562373095))|
+---------------------------------------------------------------------+
```

#### `/`（缩放旋转-除法）

**功能描述：** 将几何对象的每个坐标点与给定的 POINT 参数进行复数除法运算，实现缩放与旋转的组合变换。

> **提示：** 将点视为复数进行除法运算。若将第二个 POINT 解释为向量，则等同于将对象按向量的长度进行缩放，并以其与 x 轴的夹角为基准绕原点进行顺时针旋转。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | POINT   |
| BOX         | POINT       | BOX     |
| PATH        | POINT       | PATH    |
| CIRCLE      | POINT       | CIRCLE  |

**示例：**

```sql
-- POINT 复数除法运算
SQL> SELECT POINT('(2.0,0)') / POINT('(0,2.0)');
+--------+
| EXPR1  |
+--------+
| (0,-1) |
+--------+
```

### 9.2 几何度量

#### `@-@`（长度）

**功能描述：** 计算线段或路径长度。适用于 LSEG、PATH。

**语法格式：**

```sql
@-@ R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| 无          | LSEG        | DOUBLE  |
| 无          | PATH        | DOUBLE  |

**示例：**

```sql
-- 两点之间的线段长度
SQL> SELECT @-@ LSEG('[(-1,0),(1,0)]');
+--------------+
| EXPR1        |
+--------------+
| 2.000000e+00 |
+--------------+

-- 多点之间的总路径长度
SQL> SELECT @-@ PATH('[(0,0),(1,0),(1,1)]');
+--------------+
| EXPR1        |
+--------------+
| 2.000000e+00 |
+--------------+
```

#### `@@`（中心点）

**功能描述：** 计算几何的中心点。适用于 BOX、LSEG、POLYGON、CIRCLE。

**语法格式：**

```sql
@@ R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| 无          | LSEG        | POINT   |
| 无          | BOX         | POINT   |
| 无          | POLYGON     | POINT   |
| 无          | CIRCLE      | POINT   |

**示例：**

```sql
-- 矩形的中心点
SQL> SELECT @@ BOX('(2,2),(0,0)');
+-------+
| EXPR1 |
+-------+
| (1,1) |
+-------+

-- 圆形的中心点
SQL> SELECT @@ CIRCLE('<(3,4),2>');
+-------+
| EXPR1 |
+-------+
| (3,4) |
+-------+
```

#### `#`（交集/顶点数）

**功能描述：** 在几何数据类型中有三种功能：
- 返回几何对象包含的顶点总数，适用于 PATH、POLYGON
- 计算两条线段的交点（若无交点则返回 NULL），适用于 LSEG、LINE
- 计算两个 BOX 的交集区域（若无交集则返回 NULL），适用于 BOX

**语法格式：**

```sql
[L_OPERAND_TYP] # R_OPERAND_TYP
```

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| 无          | PATH        | INTEGER |
| 无          | POLYGON     | INTEGER |
| LSEG        | LSEG        | POINT   |
| LINE        | LINE        | POINT   |
| BOX         | BOX         | BOX     |

**示例：**

```sql
-- PATH 返回顶点的数量
SQL> SELECT # PATH('((1,0),(0,1),(-1,0))');
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- LSEG 计算交点
SQL> SELECT LSEG('[(0,0),(1,1)]') # LSEG('[(1,0),(0,1)]');
+-----------+
| EXPR1     |
+-----------+
| (0.5,0.5) |
+-----------+

-- BOX 计算两个方框的交集
SQL> SELECT BOX('(2,2),(-1,-1)') # BOX('(1,1),(-2,-2)');
+---------------+
| EXPR1         |
+---------------+
| (1,1),(-1,-1) |
+---------------+
```

#### `##`（最近点）

**功能描述：** 计算并返回第二个对象内距离第一个对象最近的点。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | BOX         | POINT   |
| LSEG        | BOX         | POINT   |
| POINT       | LINE        | POINT   |
| LINE        | LSEG        | POINT   |
| POINT       | LSEG        | POINT   |
| LSEG        | LSEG        | POINT   |

**示例：**

```sql
-- 返回边框到点的最近的点
SQL> SELECT POINT('(0,0)') ## BOX('(2,2),(4,4)');
+-------+
| EXPR1 |
+-------+
| (2,2) |
+-------+

-- 返回线段到点的最近的点
SQL> SELECT POINT('(0,0)') ## LSEG('[(2,0),(0,2)]');
+-------+
| EXPR1 |
+-------+
| (1,1) |
+-------+
```

#### `<->`（距离）

**功能描述：** 计算几何对象之间的距离。支持多种几何类型组合。

**输入输出类型（部分常用）：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | DOUBLE  |
| POINT       | LSEG        | DOUBLE  |
| POINT       | BOX         | DOUBLE  |
| POINT       | LINE        | DOUBLE  |
| POINT       | CIRCLE      | DOUBLE  |
| POINT       | POLYGON     | DOUBLE  |
| CIRCLE      | CIRCLE      | DOUBLE  |
| BOX         | BOX         | DOUBLE  |
| LINE        | LINE        | DOUBLE  |
| LSEG        | LSEG        | DOUBLE  |
| POLYGON     | POLYGON     | DOUBLE  |
| PATH        | PATH        | DOUBLE  |

**示例：**

```sql
-- 计算圆到圆的距离
SQL> SELECT CIRCLE('<(0,0),1>') <-> CIRCLE('<(5,0),1>');
+--------------+
| EXPR1        |
+--------------+
| 3.000000e+00 |
+--------------+

-- 计算点到线段的距离
SQL> SELECT POINT('(0,0)') <-> LSEG('[(2,0),(0,2)]');
+--------------+
| EXPR1        |
+--------------+
| 1.414214e+00 |
+--------------+
```

### 9.3 几何比较

#### `=`（几何相等）

**功能描述：** 判断第一个对象的面积是否等于第二个对象的面积（BOX），或判断两个对象是否相等（LINE）。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| BOX         | BOX         | BOOLEAN |
| LINE        | LINE        | BOOLEAN |

**示例：**

```sql
-- BOX 两矩形面积相等
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) = BOX(POINT('(4,5)'),POINT('(5,4)'));
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

#### `~=`（一般相等）

**功能描述：** 判断两个对象是否一般相等。适用于 POINT、BOX、POLYGON、CIRCLE。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | BOOLEAN |
| CIRCLE      | CIRCLE      | BOOLEAN |
| POLYGON     | POLYGON     | BOOLEAN |
| BOX         | BOX         | BOOLEAN |

**示例：**

```sql
-- 两个多边形一般相等（点的顺序不同）
SQL> SELECT POLYGON('((0,0),(1,1))') ~= POLYGON('((1,1),(0,0))');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

#### `<>`（几何不等于）

**功能描述：** 判断两个对象是否不等于。适用于 POINT、LSEG、CIRCLE。当操作数为 CIRCLE 时，判断两个圆面积是否不等。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POINT       | POINT       | BOOLEAN |
| CIRCLE      | CIRCLE      | BOOLEAN |
| LSEG        | LSEG        | BOOLEAN |

#### 几何面积大小比较：`<`、`<=`、`>`、`>=`

适用于 BOX 和 CIRCLE 类型，比较两个对象的面积大小。

**示例：**

```sql
-- 第一个方框面积小于第二个方框
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) < BOX(POINT('(4,5)'),POINT('(6,8)'));
+-------+
| EXPR1 |
+-------+
| T     |
+-------+

-- 第一个圆面积大于等于第二个圆
SQL> SELECT CIRCLE('<(0,0),1>') >= CIRCLE('<(5,0),1>');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

### 9.4 几何位置关系

#### `@>`（包含）

**功能描述：** 检查第一个对象是否包含第二个对象。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| BOX         | BOX         | BOOLEAN |
| BOX         | POINT       | BOOLEAN |
| POLYGON     | POLYGON     | BOOLEAN |
| POLYGON     | POINT       | BOOLEAN |
| PATH        | POINT       | BOOLEAN |
| CIRCLE      | CIRCLE      | BOOLEAN |
| CIRCLE      | POINT       | BOOLEAN |

**示例：**

```sql
-- CIRCLE 圆包含点关系
SQL> SELECT CIRCLE('<(0,0),2>') @> POINT('(1,1)');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

#### `<@`（被包含）

**功能描述：** 检查第一个对象是否被包含于第二个对象。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| POLYGON     | POLYGON     | BOOLEAN |
| LSEG        | LINE        | BOOLEAN |
| POINT       | LSEG        | BOOLEAN |
| POINT       | LINE        | BOOLEAN |
| POINT       | BOX         | BOOLEAN |
| POINT       | POLYGON     | BOOLEAN |
| LSEG        | BOX         | BOOLEAN |
| POINT       | CIRCLE      | BOOLEAN |
| CIRCLE      | CIRCLE      | BOOLEAN |
| BOX         | BOX         | BOOLEAN |
| POINT       | PATH        | BOOLEAN |

**示例：**

```sql
-- CIRCLE 圆被包含
SQL> SELECT CIRCLE('<(1,1),2>') <@ CIRCLE('<(0,0),5>');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

#### `&&`（重叠）

**功能描述：** 检查两个图形是否有任意点重叠。适用于 BOX、POLYGON、CIRCLE。

**示例：**

```sql
-- BOX 矩形框是否重叠
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) && BOX(POINT('(0,1)'),POINT('(-1,0)'));
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

#### `?#`（相交判断）

**功能描述：** 判断两个对象在平面中是否相交。

**输入输出类型：**

| 左操作数类型 | 右操作数类型 | 返回类型 |
|-------------|-------------|---------|
| LSEG        | LINE        | BOOLEAN |
| LINE        | BOX         | BOOLEAN |
| BOX         | BOX         | BOOLEAN |
| LSEG        | LSEG        | BOOLEAN |
| LINE        | LINE        | BOOLEAN |
| PATH        | PATH        | BOOLEAN |
| LSEG        | BOX         | BOOLEAN |

**示例：**

```sql
-- BOX 两个方框是否相交
SQL> SELECT BOX('((2,2),(1,1))') ?# BOX('((1,1),(0,0))');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

#### `?-|`（垂直）

**功能描述：** 判断两条线是否相互垂直。适用于 LSEG 和 LINE。

**示例：**

```sql
SQL> SELECT LSEG('[(0,0),(0,1)]') ?-| LSEG('[(0,0),(1,0)]');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

### 9.5 几何方向位置判断

| 运算符 | 功能 | 适用类型 |
|--------|------|---------|
| `<<`   | 完全位于左侧 | POINT, BOX, POLYGON, CIRCLE |
| `>>`   | 完全位于右侧 | POINT, BOX, POLYGON, CIRCLE |
| `<<\|` | 完全位于下方 | BOX, POLYGON, CIRCLE, POINT |
| `\|>>` | 完全位于上方 | BOX, POLYGON, CIRCLE, POINT |
| `<^`   | 位于下方（允许边缘相切） | POINT, BOX |
| `>^`   | 位于上方（允许边缘相切） | POINT, BOX |
| `&<`   | 位于左侧或重叠（横坐标最大值小于等于） | BOX, POLYGON, CIRCLE |
| `&>`   | 位于右侧或重叠（横坐标最小值大于等于） | BOX, POLYGON, CIRCLE |
| `&<\|` | 不超过上方 | BOX, POLYGON, CIRCLE |
| `\|&>` | 不超过下方 | BOX, POLYGON, CIRCLE |

所有方向位置判断运算符的返回类型均为 BOOLEAN。

**示例：**

```sql
-- POINT 点是否完全在左侧
SQL> SELECT POINT('(0,2.0)') << POINT('(2.0,0)');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+

-- BOX 矩形是否完全在左侧
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) << BOX(POINT('(4,5)'),POINT('(6,7)'));
+-------+
| EXPR1 |
+-------+
| T     |
+-------+

-- POINT 第一个点是否完全位于第二个点的下方
SQL> SELECT POINT('(2.0,0)') <<| POINT('(0,2.0)');
+-------+
| EXPR1 |
+-------+
| T     |
+-------+

-- BOX 第一个矩形不超过第二个矩形的上方
SQL> SELECT BOX(POINT('(0,1)'),POINT('(1,0)')) &<| BOX(POINT('(4,5)'),POINT('(6,7)'));
+-------+
| EXPR1 |
+-------+
| T     |
+-------+
```

---

## 10. 与 Oracle/MySQL/PostgreSQL 运算符对比

### 10.1 连接运算符

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 字符串连接 | `\|\|` | `\|\|` | `CONCAT()` 函数，或设置 `sql_mode` 后支持 `\|\|` | `\|\|` |
| NULL 处理 | NULL 视为空字符 | NULL 视为空字符 | `CONCAT()` 中 NULL 导致结果为 NULL | NULL 导致结果为 NULL |
| 隐式类型转换 | 支持（非 VARCHAR 类型自动转换） | 支持 | `CONCAT()` 支持 | 支持 |

> **关键差异：** 虚谷数据库与 Oracle 在 `||` 连接运算符行为上最为接近，均将 NULL 视为空字符。MySQL 默认不支持 `||` 作为连接运算符（默认为逻辑 OR），PostgreSQL 中 `||` 连接 NULL 会导致结果为 NULL。

### 10.2 JSON 运算符

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `->` 提取 JSON 元素 | 支持，返回 JSON | 不支持（使用 `JSON_QUERY`） | 支持，返回 JSON | 支持，返回 JSON |
| `->>` 提取文本值 | 支持，返回 CHAR | 不支持（使用 `JSON_VALUE`） | 支持，返回文本 | 支持，返回 TEXT |
| 路径语法 | `'$[index]'` / `'$.key'` | 非运算符方式 | `'$[index]'` / `'$.key'` | 使用整数索引或键名字符串 |

> **关键差异：** 虚谷数据库的 JSON 运算符语法与 MySQL 类似，均使用 `->` 和 `->>` 配合 JSONPath 表达式。Oracle 不支持这两个运算符，需使用 `JSON_QUERY` / `JSON_VALUE` 函数。PostgreSQL 也支持 `->` 和 `->>` 但路径语法略有不同（直接使用键名或整数索引，不使用 `$` 前缀）。

### 10.3 位运算符

| 运算符 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|--------|-----------|--------|-------|------------|
| 按位与 `&` | 支持（VARBIT, BIGINT） | 不支持（使用 `BITAND` 函数） | 支持 | 支持 |
| 按位或 `\|` | 支持（VARBIT, BIGINT） | 不支持 | 支持 | 支持 |
| 按位异或 `^` | 支持（VARBIT, BIGINT） | 不支持 | 支持 | `#`（PostgreSQL 用 `#` 表示异或） |
| 按位非 `~` | 支持 | 不支持 | `~` | `~` |
| 左移 `<<` | 支持 | 不支持 | `<<` | `<<` |
| 右移 `>>` | 支持 | 不支持 | `>>` | `>>` |
| VARBIT 类型支持 | 支持 | 不支持 | 不支持 | 支持（`bit` / `bit varying`） |

> **关键差异：** 虚谷数据库的位运算符与 MySQL 语法一致，但额外支持 VARBIT 类型（类似 PostgreSQL 的 `bit varying`）。Oracle 不支持位运算符，需使用 `BITAND` 函数。PostgreSQL 的按位异或使用 `#` 而非 `^`。虚谷数据库的位运算支持隐式类型转换（如 TINYINT、SMALLINT、FLOAT 等可隐式转换为 BIGINT 进行运算）。

### 10.4 几何运算符

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 原生几何运算符 | 丰富支持（20+ 个运算符） | 不支持（需 Oracle Spatial） | 不支持（需空间扩展） | 丰富支持 |
| 几何类型 | POINT, BOX, CIRCLE, LINE, LSEG, PATH, POLYGON | 需 SDO_GEOMETRY | 需 GEOMETRY | POINT, BOX, CIRCLE, LINE, LSEG, PATH, POLYGON |
| 距离运算符 `<->` | 支持 | 不支持 | 不支持 | 支持 |
| 包含运算符 `@>` / `<@` | 支持 | 不支持 | 不支持 | 支持 |
| 复数乘除（缩放旋转）`*` / `/` | 支持 | 不支持 | 不支持 | 支持 |

> **关键差异：** 虚谷数据库的几何运算符体系与 PostgreSQL 高度一致，提供了丰富的原生几何运算符。Oracle 和 MySQL 需要借助空间扩展模块才能实现类似功能，且不使用运算符语法。

### 10.5 日期时间运算符

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 日期 + 数字 | 支持（`CURRENT_TIMESTAMP + 30` 表示加 30 天） | 支持（`SYSDATE + 30`） | 需使用 `DATE_ADD` 函数 | 支持（需显式 `INTERVAL`） |
| 日期 + INTERVAL | 支持 | 支持 | 支持 | 支持 |
| 日期 + TIME | 支持，返回 DATETIME | 不直接支持 | 不直接支持 | 支持 |

> **关键差异：** 虚谷数据库支持 `DATE + 数字` 的简写语法（加天数），这与 Oracle 行为一致。MySQL 不支持此语法，需使用 `DATE_ADD()` 函数。虚谷数据库还支持 `DATE + TIME` 直接生成 DATETIME，这是一个便利特性。

### 10.6 比较运算符与逻辑运算符

虚谷数据库的比较运算符（`=`、`<>`/`!=`、`<`、`<=`、`>`、`>=`）和逻辑运算符（`AND`、`OR`、`NOT`）与 Oracle、MySQL、PostgreSQL 基本一致，属于 SQL 标准运算符，各数据库行为相同。

### 10.7 赋值运算符

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 赋值语法 | `=`（UPDATE SET 中） | `=`（UPDATE SET 中） | `=`（UPDATE SET 中）；`:=`（变量赋值） | `=`（UPDATE SET 中）；`:=`（PL/pgSQL） |

> **关键差异：** 各数据库在 UPDATE SET 语句中均使用 `=` 进行赋值，行为一致。MySQL 在用户变量赋值中还支持 `:=` 运算符。
