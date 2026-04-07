---
title: 虚谷数据库 SQL 基础
description: 关键字与保留字、标识符命名规则、字面量类型、字符集支持、类型转换机制，与 Oracle/MySQL/PostgreSQL 基础语法差异
tags: xugudb, sql-basics, keywords, identifiers, literals, charset, type-conversion, reserved-words
---

# 虚谷数据库 SQL 基础

---

## 关键字

### 概述

在数据库中，关键字是 SQL 中用于表示特定功能、有实际意义的词。为防止歧义，一般不推荐使用关键字作为对象名或集合名等。根据系统要求，关键字分为保留字和非保留字。

### 保留字

数据库内部强制不能使用的字。例如 `CREATE`，在数据库的 DDL 语句中用于定义对象创建的操作，如果使用这个关键字作为表对象名就会产生歧义，数据库一般会报错。

出于兼容性需求，某些保留字允许直接被用户使用；如果保留字确实需要被使用，可以通过添加双引号作为表名或列名等标识符。

```sql
-- 使用允许作为表名的保留字作为表名（部分保留字允许直接使用）
CREATE TABLE both(id int, name varchar);

-- 使用不允许作为表名的保留字作为表名（报错）
CREATE TABLE boverlaps(id int, name varchar);
-- Error: [E19132] 语法错误 [E19260 L1 C13] 期待符号: syntax error, unexpected BOVERLAPS

-- 使用双引号包裹保留字作为表名
CREATE TABLE "both"(id int, name varchar);
CREATE TABLE "boverlaps"(id int, name varchar);
```

> **提示**：如果有关键字需要在业务中使用，并且无法增加双引号包裹关键字，可以通过关键字过滤（KEYWORD_FILTER）功能实现关键字作为对象名使用。

### 非保留字

非保留字是作为将来数据库可能会被用到的功能预留。

```sql
-- 使用允许作为表名的非保留字作为表名
CREATE TABLE call(id int, name varchar);

-- 使用不允许作为表名的非保留字作为表名（报错）
CREATE TABLE cast(id int, name varchar);
-- Error: [E19132] 语法错误 [E19260 L1 C13] 期待符号: syntax error, unexpected CAST
```

### 关键字使用规则

每个关键字有不同的使用权限，分为以下维度：

| 维度 | 说明 |
|------|------|
| 作为表名 | 该关键字能否直接用作表名 |
| 作为列名 | 该关键字能否直接用作列名 |
| 作为列别名 | 该关键字能否直接用作列别名 |

常见保留字示例：

| 关键字 | 类别 | 作为表名 | 作为列名 | 作为列别名 |
|--------|------|----------|----------|-----------|
| BEGIN | 保留 | 不可 | 不可 | 不可 |
| COMMIT | 保留 | 不可 | 不可 | 不可 |
| CREATE | 保留 | 不可 | 不可 | 可 |
| SELECT | 保留 | 不可 | 不可 | 可 |
| TABLE | 保留 | 不可 | 不可 | 可 |
| WHERE | 保留 | 可 | 不可 | 可 |
| NULL | 保留 | 不可 | 不可 | 可 |
| INDEX | 保留 | 不可 | 不可 | 可 |
| DATE | 保留 | 不可 | 不可 | 可 |
| VARCHAR | 保留 | 不可 | 不可 | 可 |

常见非保留字示例：

| 关键字 | 类别 | 作为表名 | 作为列名 | 作为列别名 |
|--------|------|----------|----------|-----------|
| DATABASE | 非保留 | 可 | 可 | 可 |
| NAME | 非保留 | 可 | 不可 | 可 |
| COMMENT | 非保留 | 不可 | 不可 | 可 |
| END | 非保留 | 不可 | 不可 | 可 |
| CAST | 非保留 | 不可 | 不可 | 可 |

---

## 标识符

### 定义

标识符是用于命名数据库对象的符号。数据库对象从逻辑结构上分为模式对象和非模式对象。

**模式对象**（属于某个模式/用户）：
- 表（TABLE）、视图（VIEW）、索引（INDEX）、分区（PARTITION）
- 序列（SEQUENCE）、约束（CONSTRAINT）、同义词（SYNONYM）
- 触发器（TRIGGER）、数据库链接（DATABASE LINK）
- 自定义数据类型（UDT）、函数（FUNCTION）、存储过程（PROCEDURE）、包（PACKAGE）

**非模式对象**：
- 用户（USER）、角色（ROLE）

### 标识符类型

| 类型 | 说明 | 示例 |
|------|------|------|
| 引号标识符 | 使用双引号 `"` 或反引号 `` ` `` 包裹 | `"my_table"`、`` `my_table` `` |
| 非引号标识符 | 不包含任何外部标点符号 | `my_table` |

> 数据库名称和数据库链接名称始终不区分大小写。若以引号标识符形式指定此类名称，引号将被忽略。

### 命名规则

1. **长度限制**：标识符的长度为 1 到 127 字节。如果标识符包含由点分隔的多个部分，则每个属性最多可长达 127 字节。

   ```
   "schema"."table"."column"
   ```

2. **关键字限制**：未加引号的标识符不能是关键字。加引号的标识符可以是关键字，但不推荐。

3. **大小写规则**：未加引号的标识符不区分大小写，数据库将它们解释为大写（PostgreSQL 兼容模式除外，受兼容参数 `def_compatible_mode` 的影响）。加引号的标识符区分大小写。

4. **命名空间唯一性**：在命名空间内，不能有两个对象具有相同的名称。

   共享命名空间的模式对象（同一模式下不能重名）：
   - 包、私有同义词、序列、存储过程、独立存储函数、表、用户定义类型、视图

   拥有独立命名空间的模式对象（对象名称可以重复）：
   - 约束、触发器、索引、数据库链接

5. **大小写敏感控制**：`cata_case_sensitive` 参数控制系统字典中对象名的存储和匹配是否区分大小写。

### 标识符引用

引用对象或其组成部分的语法：

```
database_object_or_part ::= (schema '.')? object ('.' part)? ('@' dblink)?
```

- `schema`：对象所属模式名称，省略时默认引用当前用户模式中的对象
- `object`：对象的名称
- `part`：对象的一部分（如表的列）
- `dblink`：数据库链接标识符，用于访问远程数据库对象

```sql
-- 删除模式 hr 中的表 employees
DROP TABLE hr.employees;

-- 通过 CURRENT_SCHEMA 参数指定目标模式
SET CURRENT_SCHEMA TO hr;
DROP TABLE employees;

-- 引用远程数据库对象
SELECT * FROM hr.employees@remote_db;

-- 引用分区
SELECT * FROM schema.object PARTITION (partition_name);

-- 引用子分区
SELECT * FROM schema.object SUBPARTITION (subpartition_name);
```

---

## 字面量

### 概述

字面量（Literal）也称"常量值"，指固定的数据值。虚谷数据库支持以下字面量类型：文本字面量、数值字面量、日期时间字面量、区间字面量、位值字面量、十六进制字面量、空值、布尔字面量。

字符字面量用单引号括起来，以便将其与模式对象名称区分开来。例如 `'XUGU'`、`'DATABASE'`、`'101'` 都是字符字面量；`5001` 是数字字面量。

### 文本字面量 (Text Literals)

文本字面量是使用单引号 `' '` 引起来的字符串，本身具有 CHAR 数据类型的属性。

**语法**：

```
literal ::= (('Q' | 'q') "'" delimiter char* delimiter "'" | "'" char* "'")
```

**规则**：
- 要在字面量中表示一个单引号，请输入两个单引号 `''`，或使用转义字符
- 可使用 `Q` 或 `q` 替代单引号作为分隔符，`delimiter` 是 `[`, `{`, `<`, `\` 或 `(` 之一，结尾必须分别对应 `]`, `}`, `>`, `\` 或 `)`

```sql
SELECT '这是文本' FROM dual;
-- 结果: 这是文本

SELECT q'<abcd1>' FROM dual;
-- 结果: abcd1

SELECT Q'{defa3}' FROM dual;
-- 结果: defa3
```

### 数值字面量 (Numeric Literals)

**语法**：

```
number ::= ('+' | '-')? (digit+ '.' digit* | '.' digit) (('e' | 'E') ('+' | '-') digit+)? ('f' | 'F' | 'd' | 'D')?
```

- `+` 或 `-` 表示正值或负值，省略默认为正值
- 支持科学计数法（`e` / `E`）
- 后缀 `f`/`F` 表示 FLOAT，`d`/`D` 表示 DOUBLE

### 日期时间字面量 (Datetime Literals)

虚谷数据库支持四种日期时间数据类型：`DATE`、`TIMESTAMP`、`TIMESTAMP WITH TIME ZONE` 和 `TIMESTAMP WITH LOCAL TIME ZONE`。

```sql
-- DATE 字面量
DATE('2025-07-25')
TO_DATE('2025-07-25 15:35:30', 'YYYY-MM-DD HH24:MI:SS')

-- TIMESTAMP 字面量
TIMESTAMP('2025-07-25 12:36:52.421')

-- TIMESTAMP WITH TIME ZONE 字面量
TIMESTAMP('2025-07-25 12:36:52.421 +08:00')
```

> 当两个 `TIMESTAMP WITH TIME ZONE` 字面量表示 GMT 时区的同一时刻，即使时区字段不同，也被视为相同的字面量。

### 区间字面量 (Interval Literals)

区间字面量指定一个时间段，可以用年和月来指定，也可以用天、小时、分钟和秒来指定。

```sql
SELECT INTERVAL '100' YEAR(3) FROM dual;       -- 100
SELECT INTERVAL '3' MONTH FROM dual;            -- 3
SELECT INTERVAL '1-3' YEAR TO MONTH FROM dual;  -- 1-03
SELECT INTERVAL '7' DAY FROM dual;              -- 7
SELECT INTERVAL '3 04' DAY TO HOUR FROM dual;   -- 3 04
SELECT INTERVAL '2 12:30' DAY TO MINUTE FROM dual;           -- 2 12:30
SELECT INTERVAL '1 01:01:01' DAY TO SECOND FROM dual;        -- 1 01:01:01.000000
SELECT INTERVAL '1 01:01:01.123' DAY TO SECOND(3) FROM dual; -- 1 01:01:01.123000
SELECT INTERVAL '12:30' HOUR TO MINUTE FROM dual;  -- 12:30
SELECT INTERVAL '30' MINUTE FROM dual;             -- 30
SELECT INTERVAL '5.678' SECOND(3) FROM dual;       -- 5.678000
```

### 位值字面量 (Bit-Value Literals)

使用 `b'val'` 表示法书写，`val` 为 0 和 1 组成的二进制值。

```sql
SELECT b'1011', B'1101' FROM dual;
-- 结果: b'1011', b'1101'

-- 空位值求值为零长度二进制字符串
SELECT LENGTH(B''), LENGTH(b'') FROM dual;
-- 结果: 0, 0
```

### 十六进制字面量 (Hexadecimal Literals)

使用 `X'val'` 或 `H'val'` 表示法书写，`val` 包含十六进制数字（0-9、A-F、a-f）。奇数个数字的十六进制字面量会被视为有一个额外的前导 0（例如 `X'abc'` 等于 `X'0abc'`）。

```sql
SELECT HEX(X'01AF'), HEX(X'01af'), HEX(x'01AF'), HEX(x'01af') FROM dual;
-- 结果: 01AF, 01AF, 01AF, 01AF

-- 空十六进制值
SELECT LENGTH(x'') FROM dual;
-- 结果: 0
```

### 空值字面量 (NULL Values)

`NULL` 值表示"无数据"，可以用任何大小写形式书写。

- `NULL` 值与数值类型的 `0` 或字符串类型的空字符串等值不同
- 使用 `ORDER BY` 排序时，`NULL` 值在升序排序时排在其他值之前，在降序排序时排在其他值之后
- `NULL` 在特定函数中使用需要指定类型，如 `NULL:CHAR`

### 布尔字面量 (Boolean Literals)

常量 `TRUE` 和 `FALSE` 等于 1 和 0，大小写不敏感。

```sql
SELECT TRUE, true, tRuE, FALSE, FaLsE, false FROM dual;
-- 结果: T, T, T, F, F, F
```

---

## 字符集

### 概述

数据库字符集（Character Set）决定数据库中如何存储和解释字符串数据。虚谷数据库支持库级字符集。

### 支持的字符集

| 字符集 | 描述 | 语言/地区 | 字节数 | 备注 |
|--------|------|-----------|--------|------|
| UTF-8 | Unicode 最流行编码方式 | 所有语言 | 1~3 字节 | 推荐国际通用编码 |
| GB18030 | 中国国家强制标准，支持全部 Unicode | 全世界 | 1~2 字节 | 完全兼容 GBK、GB2312 |
| GBK | 向下兼容 GB2312 | 简繁中文 | 1~2 字节 | GB18030 是其超集 |
| GB2312 | 简体中文字符集早期标准 | 简体中文 | 1~2 字节 | GBK 是其扩展 |
| GB13000 | 对 Unicode 1.1 的兼容适配 | 中文 | 1~2 字节 | 后被 GB18030 取代 |
| BIG5 | 繁体中文标准 | 繁体中文 | 1~2 字节 | 与 GB 系列不兼容 |
| LATIN1 | ISO 8859-1 西欧语言 | 英、德、法等 | 1 字节 | 常见老系统默认编码 |
| LATIN2 | ISO 8859-2 中欧语言 | 捷克、波兰等 | 1 字节 | - |
| CP1250 | Windows 编码页 | 中欧语言 | 1 字节 | 类似 Latin2 微软版本 |
| BINARY | 非文本按字节存储 | 任意二进制 | 1 字节 | 适用于原始数据存储 |

### 排序规则

`COLLATE_NAME` 列中：
- `bin` 后缀：区分大小写
- `ci` 后缀：不区分大小写
- 在不指定后缀时，虚谷数据库使用 `bin` 类字符集（区分大小写）

### 库字符集与会话字符集

**库字符集**：用于设置数据库存储和解释字符串数据的表示。
- 系统库 SYSTEM 的字符集为 `UTF8.UTF8_GENERAL_CI`，不可修改
- 新建用户库默认字符集为 `GBK`，可通过修改 `SETUP/xugu.ini` 中 `def_charset` 变量修改

**会话字符集**：用于设置客户端与数据库之间进行会话的字符串数据表示。
- 默认会话字符集为 `GBK`

### 字符集查看与设置

```sql
-- 查看可用字符集
SELECT * FROM SYS_CHARSETS;
SHOW charsets;

-- 查看数据库字符集
SELECT CHAR_SET FROM DBA_DATABASES WHERE DB_NAME='SYSTEM';

-- 查看当前数据库信息（包含字符集）
SHOW DB_INFO;

-- 查看新建库默认字符集
SHOW def_charset;

-- 设置新建库默认字符集
SET def_charset TO UTF8;

-- 查看会话字符集
SHOW CHAR_SET;

-- 设置会话字符集
SET char_set TO UTF8;

-- 创建数据库时指定字符集
CREATE DATABASE db_test CHAR SET 'GBK';
CREATE DATABASE db_test CHARACTER SET 'GBK';
```

### 字符集转换

```sql
-- 字符串数据类型转换
SELECT CONVERT('TEST_CONVERT', 'GBK');

-- RAW 数据类型转换
SELECT to_char(UTL_RAW.CONVERT(UTL_RAW.CAST_TO_RAW('HELLO'), 'GBK', 'UTF8')) result FROM DUAL;
```

> **注意**：字符集转换后，如果会话字符集与转换后字符集不兼容，可能出现乱码现象。切换数据库时，会话字符集不发生变化。

---

## 类型转换

### 概述

在虚谷数据库中，表达式计算通常要求操作数具有相同的数据类型。当表达式中包含不同数据类型的值时，提供两种类型转换机制：

### 显式转换

用户主动使用 SQL 转换函数将值从一种数据类型转换为另一种数据类型。

```sql
-- 使用 CAST 函数
SELECT CAST('123' AS INTEGER);

-- 使用 CONVERT 函数
SELECT CONVERT('TEST', 'GBK');

-- 使用 :: 运算符
SELECT '123'::INTEGER;
```

### 隐式转换

数据库在执行 SQL 语句时自动进行的类型转换，遵循预定义的转换规则和优先级：

- 在 `INSERT`/`UPDATE` 操作期间，会将值转换为受影响列的数据类型
- 在 `SELECT` 操作期间，会将数据从列转换为目标变量的类型
- 操作数值时，通常会调整精度和小数位数以实现最大容量
- 将字符值与数值进行比较时，会将字符数据转换为数值
- 字符值或 `NUMERIC` 与浮点数值之间的转换可能不精确
- 进行赋值时，会将等号 `=` 右侧的值转换为左侧赋值目标的数据类型

---

## 与 Oracle/MySQL/PostgreSQL 基础语法对比

### 关键字与标识符

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 引号标识符 | 双引号 `"` 或反引号 `` ` `` | 双引号 `"` | 反引号 `` ` `` | 双引号 `"` |
| 未加引号标识符大小写 | 解释为大写（PG 兼容模式除外） | 解释为大写 | 不区分大小写（保留原样存储） | 解释为小写 |
| 标识符最大长度 | 127 字节 | 128 字节 | 64 字符 | 63 字节 |
| 关键字过滤功能 | 支持 KEYWORD_FILTER | 不支持 | 不支持 | 不支持 |
| `cata_case_sensitive` 参数 | 支持 | 不支持 | `lower_case_table_names` 类似 | 不支持 |

### 字面量

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| Q 引用语法（`q'<...>'`） | 支持 | 支持 | 不支持 | 不支持（可用 `$$...$$`） |
| 位值字面量 `b'...'` | 支持 | 不支持 | 支持 | 支持 |
| 十六进制 `X'...'` | 支持 | 不支持（用 `HEXTORAW`） | 支持 | 支持 |
| 十六进制 `H'...'` | 支持 | 不支持 | 不支持 | 不支持 |
| 布尔字面量 TRUE/FALSE | 支持（显示为 T/F） | 不直接支持 | 支持（显示为 1/0） | 支持（显示为 t/f） |
| NULL 排序（升序） | 排在最前 | 排在最后 | 排在最前 | 排在最后 |
| INTERVAL 字面量 | 支持 | 支持 | 部分支持 | 支持 |
| `DATE()` 函数构造日期 | 支持 | 使用 `DATE` 关键字 | 使用 `DATE` 关键字 | 使用 `DATE` 关键字 |

### 字符集

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 字符集级别 | 库级 | 库级 + 国家字符集 | 库级 + 表级 + 列级 | 库级 |
| 默认字符集 | GBK（用户库）/ UTF8（系统库） | AL32UTF8（推荐） | utf8mb4（推荐） | UTF8 |
| 字符集查看 | `SHOW charsets` / `SYS_CHARSETS` | `NLS_DATABASE_PARAMETERS` | `SHOW CHARACTER SET` | `SHOW server_encoding` |
| 会话字符集设置 | `SET char_set TO ...` | `ALTER SESSION SET NLS_LANGUAGE` | `SET NAMES ...` | `SET client_encoding TO ...` |
| GB18030 支持 | 原生支持 | 不支持 | 支持 | 不直接支持 |

### 类型转换

| 特性 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| `CAST(... AS ...)` | 支持 | 支持 | 支持 | 支持 |
| `::` 类型转换运算符 | 支持 | 不支持 | 不支持 | 支持 |
| `CONVERT()` 函数 | 支持（字符集转换） | 支持（`CONVERT(..., charset)`） | 支持（字符集转换） | 支持（`CONVERT(..., charset)`） |
| 隐式转换严格程度 | 中等 | 较宽松 | 宽松 | 严格 |

### 迁移注意事项

- **从 Oracle 迁移**：虚谷数据库默认将未加引号标识符解释为大写，与 Oracle 一致。Oracle 的 `q'<...>'` 引用语法在虚谷数据库中直接可用。注意 Oracle 不支持 `::` 类型转换运算符。
- **从 MySQL 迁移**：MySQL 使用反引号作为引号标识符，虚谷数据库同时支持双引号和反引号。MySQL 的 `b'...'` 位值字面量和 `X'...'` 十六进制字面量语法兼容。注意虚谷数据库标识符大小写行为与 MySQL 不同。
- **从 PostgreSQL 迁移**：虚谷数据库支持 `::` 类型转换运算符，与 PostgreSQL 兼容。注意虚谷数据库默认将未加引号标识符解释为大写，而 PostgreSQL 解释为小写。可通过 `def_compatible_mode` 参数启用 PostgreSQL 兼容模式。
