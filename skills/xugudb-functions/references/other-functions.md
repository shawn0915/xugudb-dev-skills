---
title: 虚谷数据库其他特殊函数参考
description: 比较函数、流程控制函数、加密函数、编码解码函数、BIT 函数、UUID 函数、XML 函数、大对象函数、数组函数、网络地址函数、序列值函数、系统信息函数、系统管理函数、几何函数及其他函数的完整参考
tags: [xugudb, functions, compare, flow-control, encryption, encoding, bit, uuid, xml, lob, array, inet, sequence, system, geometry]
---

# 虚谷数据库其他特殊函数参考

本文档涵盖虚谷数据库中除字符串函数、数学函数、日期时间函数、聚合函数、分析函数和 JSON 函数之外的所有其他系统函数，按功能类别分节。

---

## 目录

1. [比较函数](#1-比较函数)
2. [流程控制函数](#2-流程控制函数)
3. [加密函数](#3-加密函数)
4. [编码解码函数](#4-编码解码函数)
5. [BIT 函数](#5-bit-函数)
6. [UUID 函数](#6-uuid-函数)
7. [XML 函数](#7-xml-函数)
8. [大对象函数](#8-大对象函数)
9. [数组函数](#9-数组函数)
10. [网络地址函数](#10-网络地址函数)
11. [序列值函数](#11-序列值函数)
12. [系统信息函数](#12-系统信息函数)
13. [系统管理函数](#13-系统管理函数)
14. [几何函数](#14-几何函数)
15. [其他函数](#15-其他函数)

---

## 1. 比较函数

比较函数用于值的比较判断，返回比较结果或特殊值处理。

| 函数 | 说明 | 语法 |
|------|------|------|
| COMPARE | 比较两个值，返回比较结果（-1/0/1） | `COMPARE(val1, val2)` |
| INTERVAL | 返回第一个参数在后续有序参数列表中的位置索引 | `INTERVAL(n, n1, n2, ...)` |
| ISNULL | 判断表达式是否为 NULL | `ISNULL(expr)` |
| NUM_NONNULLS | 返回参数列表中非 NULL 值的个数 | `NUM_NONNULLS(val1, val2, ...)` |
| NUM_NULLS | 返回参数列表中 NULL 值的个数 | `NUM_NULLS(val1, val2, ...)` |

### 常用示例

```sql
-- COMPARE：比较两个值
SQL> SELECT COMPARE(10, 20);
+-------+
| EXPR1 |
+-------+
|    -1 |
+-------+

SQL> SELECT COMPARE(20, 20);
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+

-- INTERVAL：返回位置索引
-- 返回 n 在有序列表 (n1, n2, ...) 中的位置
-- 若 n < n1 返回 0，若 n1 <= n < n2 返回 1，以此类推
SQL> SELECT INTERVAL(5, 1, 3, 5, 7);
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- ISNULL：判断是否为 NULL
SQL> SELECT ISNULL(NULL);
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT ISNULL(123);
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+

-- NUM_NONNULLS / NUM_NULLS
SQL> SELECT NUM_NONNULLS(1, NULL, 3, NULL, 5);
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

SQL> SELECT NUM_NULLS(1, NULL, 3, NULL, 5);
+-------+
| EXPR1 |
+-------+
|     2 |
+-------+
```

---

## 2. 流程控制函数

流程控制函数根据条件返回不同的值，常用于 SQL 中的条件逻辑处理。

| 函数 | 说明 | 语法 |
|------|------|------|
| IF | 根据条件返回不同值 | `IF(condition, true_val, false_val)` |
| IFNULL | 若第一个参数为 NULL 则返回第二个参数 | `IFNULL(expr, alt_value)` |
| NULLIF | 若两个参数相等则返回 NULL，否则返回第一个参数 | `NULLIF(expr1, expr2)` |

### 常用示例

```sql
-- IF：条件判断
SQL> SELECT IF(1 > 0, '是', '否');
+-------+
| EXPR1 |
+-------+
| 是    |
+-------+

SQL> SELECT IF(score >= 60, '及格', '不及格') AS result FROM exam;

-- IFNULL：NULL 值替换
SQL> SELECT IFNULL(NULL, '默认值');
+--------+
| EXPR1  |
+--------+
| 默认值 |
+--------+

SQL> SELECT IFNULL(name, '未知') AS display_name FROM users;

-- NULLIF：相等返回 NULL
SQL> SELECT NULLIF(10, 10);
+-------+
| EXPR1 |
+-------+
| NULL  |
+-------+

SQL> SELECT NULLIF(10, 20);
+-------+
| EXPR1 |
+-------+
|    10 |
+-------+

-- 常用于避免除零错误
SQL> SELECT total / NULLIF(count, 0) AS average FROM stats;
```

### 与 Oracle/MySQL/PostgreSQL 对比

| 功能 | 虚谷数据库 | MySQL | PostgreSQL | Oracle |
|------|-----------|-------|------------|--------|
| 条件判断 | IF | IF | 无（用 CASE） | 无（用 CASE/DECODE） |
| NULL 替换 | IFNULL | IFNULL | COALESCE | NVL |
| 相等返 NULL | NULLIF | NULLIF | NULLIF | NULLIF |

---

## 3. 加密函数

加密函数用于对数据进行哈希摘要运算，常用于数据完整性校验和密码存储。

| 函数 | 说明 | 语法 |
|------|------|------|
| MD5 | 计算 MD5 哈希值（128 位） | `MD5(string)` |
| SHA | 计算 SHA-1 哈希值（160 位），SHA1 的别名 | `SHA(string)` |
| SHA1 | 计算 SHA-1 哈希值（160 位） | `SHA1(string)` |
| SHA2 | 计算 SHA-2 系列哈希值 | `SHA2(string, hash_length)` |

### 常用示例

```sql
-- MD5 哈希
SQL> SELECT MD5('hello');
+----------------------------------+
| EXPR1                            |
+----------------------------------+
| 5d41402abc4b2a76b9719d911017c592 |
+----------------------------------+

-- SHA1 哈希
SQL> SELECT SHA1('hello');
+------------------------------------------+
| EXPR1                                    |
+------------------------------------------+
| aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d |
+------------------------------------------+

-- SHA / SHA1 等价
SQL> SELECT SHA('hello');
+------------------------------------------+
| EXPR1                                    |
+------------------------------------------+
| aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d |
+------------------------------------------+

-- SHA2：指定哈希长度（224/256/384/512）
SQL> SELECT SHA2('hello', 256);
+------------------------------------------------------------------+
| EXPR1                                                            |
+------------------------------------------------------------------+
| 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824 |
+------------------------------------------------------------------+
```

---

## 4. 编码解码函数

编码解码函数用于数据的编码转换，包括 Base64、十六进制、压缩等操作。

| 函数 | 说明 | 语法 |
|------|------|------|
| BASE64_DECODE | 对 Base64 编码字符串进行解码 | `BASE64_DECODE(string)` |
| BASE64_ENCODE | 将字符串编码为 Base64 | `BASE64_ENCODE(string)` |
| COMPRESS_FLOATS | 压缩浮点数数组 | `COMPRESS_FLOATS(float_array)` |
| DECODE_PG | PostgreSQL 兼容的解码函数 | `DECODE_PG(string, format)` |
| ENCODE_PG | PostgreSQL 兼容的编码函数 | `ENCODE_PG(data, format)` |
| ESCAPE_DECODE | 对转义编码的字符串进行解码 | `ESCAPE_DECODE(string)` |
| ESCAPE_ENCODE | 将字符串进行转义编码 | `ESCAPE_ENCODE(string)` |
| FROM_BASE64 | 对 Base64 字符串解码（MySQL 兼容别名） | `FROM_BASE64(string)` |
| HEX_DECODE | 将十六进制字符串解码为原始数据 | `HEX_DECODE(hex_string)` |
| HEX_ENCODE | 将数据编码为十六进制字符串 | `HEX_ENCODE(data)` |
| TO_BASE64 | 将数据编码为 Base64（MySQL 兼容别名） | `TO_BASE64(data)` |
| UNCOMPRESS_FLOAT | 解压缩单个浮点数 | `UNCOMPRESS_FLOAT(compressed)` |
| UNCOMPRESS_FLOATS | 解压缩浮点数数组 | `UNCOMPRESS_FLOATS(compressed)` |

### 常用示例

```sql
-- Base64 编解码
SQL> SELECT BASE64_ENCODE('虚谷数据库');
+------------------------+
| EXPR1                  |
+------------------------+
| 6Jma6LC35pWw5o2u5bqT |
+------------------------+

SQL> SELECT BASE64_DECODE('6Jma6LC35pWw5o2u5bqT');
+-----------+
| EXPR1     |
+-----------+
| 虚谷数据库 |
+-----------+

-- TO_BASE64 / FROM_BASE64（MySQL 兼容写法）
SQL> SELECT TO_BASE64('hello');
+----------+
| EXPR1    |
+----------+
| aGVsbG8= |
+----------+

SQL> SELECT FROM_BASE64('aGVsbG8=');
+-------+
| EXPR1 |
+-------+
| hello |
+-------+

-- 十六进制编解码
SQL> SELECT HEX_ENCODE('ABC');
+--------+
| EXPR1  |
+--------+
| 414243 |
+--------+

SQL> SELECT HEX_DECODE('414243');
+-------+
| EXPR1 |
+-------+
| ABC   |
+-------+

-- PostgreSQL 兼容编解码
SQL> SELECT ENCODE_PG('hello'::BINARY, 'hex');
+------------+
| EXPR1      |
+------------+
| 68656c6c6f |
+------------+

SQL> SELECT DECODE_PG('68656c6c6f', 'hex');
+-------+
| EXPR1 |
+-------+
| hello |
+-------+
```

---

## 5. BIT 函数

BIT 函数用于二进制位运算操作。

| 函数 | 说明 | 语法 |
|------|------|------|
| BITAND | 按位与（Oracle 兼容） | `BITAND(val1, val2)` |
| BITTOCHAR | 将 BIT 类型转换为字符串 | `BITTOCHAR(bit_val)` |
| BIT_AND | 按位与 | `BIT_AND(val1, val2)` |
| BIT_CLR | 清除指定位（置 0） | `BIT_CLR(val, bit_pos)` |
| BIT_COUNT | 计算二进制中 1 的个数 | `BIT_COUNT(val)` |
| BIT_LENGTH | 返回值的位长度 | `BIT_LENGTH(val)` |
| BIT_NOT | 按位取反 | `BIT_NOT(val)` |
| BIT_OR | 按位或 | `BIT_OR(val1, val2)` |
| BIT_SET | 设置指定位（置 1） | `BIT_SET(val, bit_pos)` |
| BIT_TEST | 测试指定位是否为 1 | `BIT_TEST(val, bit_pos)` |
| BIT_XOR | 按位异或 | `BIT_XOR(val1, val2)` |
| SHL | 左移位 | `SHL(val, shift_count)` |
| SHR | 右移位 | `SHR(val, shift_count)` |

### 常用示例

```sql
-- BIT_AND / BITAND：按位与
SQL> SELECT BIT_AND(12, 10);
+-------+
| EXPR1 |
+-------+
|     8 |
+-------+

-- BITAND（Oracle 兼容写法）
SQL> SELECT BITAND(12, 10);
+-------+
| EXPR1 |
+-------+
|     8 |
+-------+

-- BIT_OR：按位或
SQL> SELECT BIT_OR(12, 10);
+-------+
| EXPR1 |
+-------+
|    14 |
+-------+

-- BIT_XOR：按位异或
SQL> SELECT BIT_XOR(12, 10);
+-------+
| EXPR1 |
+-------+
|     6 |
+-------+

-- BIT_NOT：按位取反
SQL> SELECT BIT_NOT(0);
+-------+
| EXPR1 |
+-------+
|    -1 |
+-------+

-- BIT_COUNT：计算 1 的个数
SQL> SELECT BIT_COUNT(7);
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- BIT_LENGTH：返回位长度
SQL> SELECT BIT_LENGTH('hello');
+-------+
| EXPR1 |
+-------+
|    40 |
+-------+

-- SHL / SHR：移位
SQL> SELECT SHL(1, 3);
+-------+
| EXPR1 |
+-------+
|     8 |
+-------+

SQL> SELECT SHR(16, 2);
+-------+
| EXPR1 |
+-------+
|     4 |
+-------+

-- BIT_SET / BIT_CLR / BIT_TEST：位操作
SQL> SELECT BIT_SET(0, 3);
+-------+
| EXPR1 |
+-------+
|     8 |
+-------+

SQL> SELECT BIT_CLR(15, 1);
+-------+
| EXPR1 |
+-------+
|    13 |
+-------+

SQL> SELECT BIT_TEST(8, 3);
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

-- BITTOCHAR：BIT 类型转字符串
SQL> SELECT BITTOCHAR(B'10110');
+-------+
| EXPR1 |
+-------+
| 10110 |
+-------+
```

---

## 6. UUID 函数

UUID 函数用于生成全局唯一标识符。

| 函数 | 说明 | 语法 |
|------|------|------|
| GEN_RANDOM_UUID | 生成随机 UUID（PostgreSQL 兼容） | `GEN_RANDOM_UUID()` |
| NEWID | 生成新的 GUID 值（SQL Server 兼容） | `NEWID()` |
| SYS_GUID | 生成系统 GUID（Oracle 兼容） | `SYS_GUID()` |
| SYS_UUID | 生成系统 UUID | `SYS_UUID()` |
| UUID | 生成 UUID | `UUID()` |

### 常用示例

```sql
-- UUID：生成 UUID
SQL> SELECT UUID();
+--------------------------------------+
| EXPR1                                |
+--------------------------------------+
| 550e8400-e29b-41d4-a716-446655440000 |
+--------------------------------------+

-- GEN_RANDOM_UUID：PostgreSQL 兼容
SQL> SELECT GEN_RANDOM_UUID();
+--------------------------------------+
| EXPR1                                |
+--------------------------------------+
| f47ac10b-58cc-4372-a567-0e02b2c3d479 |
+--------------------------------------+

-- SYS_GUID：Oracle 兼容（返回 32 位十六进制字符串）
SQL> SELECT SYS_GUID();
+----------------------------------+
| EXPR1                            |
+----------------------------------+
| A1B2C3D4E5F6078899AABBCCDDEEFF00 |
+----------------------------------+

-- NEWID：SQL Server 兼容
SQL> SELECT NEWID();
+--------------------------------------+
| EXPR1                                |
+--------------------------------------+
| 6ba7b810-9dad-11d1-80b4-00c04fd430c8 |
+--------------------------------------+

-- 在建表中使用
SQL> CREATE TABLE orders (
    id CHAR(36) DEFAULT UUID(),
    order_name VARCHAR(100)
);
```

### 与 Oracle/MySQL/PostgreSQL 对比

| 功能 | 虚谷数据库 | MySQL | PostgreSQL | Oracle |
|------|-----------|-------|------------|--------|
| 生成 UUID | UUID() | UUID() | gen_random_uuid() | SYS_GUID() |
| Oracle 兼容 | SYS_GUID() | 无 | 无 | SYS_GUID() |
| PG 兼容 | GEN_RANDOM_UUID() | 无 | gen_random_uuid() | 无 |
| SQL Server 兼容 | NEWID() | 无 | 无 | 无 |

---

## 7. XML 函数

XML 函数用于 XML 数据的构造、查询和解析操作。

| 函数 | 说明 | 语法 |
|------|------|------|
| EXTRACT (XML) | 从 XML 数据中提取指定 XPath 路径的节点 | `EXTRACT(xml_data, xpath_expr)` |
| EXTRACTVALUE | 从 XML 数据中提取指定路径的文本值 | `EXTRACTVALUE(xml_data, xpath_expr)` |
| XMLATTRIBUTES | 在 XMLELEMENT 中添加 XML 属性 | `XMLATTRIBUTES(expr AS name, ...)` |
| XMLCAST | 将 XML 值转换为 SQL 数据类型 | `XMLCAST(xml_expr AS data_type)` |
| XMLELEMENT | 构造一个 XML 元素 | `XMLELEMENT(NAME name, content)` |
| XMLEXISTS | 判断 XPath 表达式在 XML 中是否存在匹配 | `XMLEXISTS(xpath PASSING xml_data)` |
| XMLFOREST | 将多个列或表达式构造为一组 XML 元素 | `XMLFOREST(expr AS name, ...)` |
| XMLQUERY | 使用 XQuery 表达式查询 XML 数据 | `XMLQUERY(xquery_expr PASSING xml_data)` |
| XMLSEQUENCE | 将 XML 片段转换为行集合 | `XMLSEQUENCE(xml_data)` |
| XMLTABLE | 将 XML 数据映射为关系表行列 | `XMLTABLE(xpath PASSING xml_data COLUMNS ...)` |
| XMLTYPE | 将字符串构造为 XML 类型 | `XMLTYPE(string)` |

### 常用示例

```sql
-- XMLELEMENT：构造 XML 元素
SQL> SELECT XMLELEMENT(NAME "employee", XMLATTRIBUTES(1 AS "id"), '张三');
+-------------------------------------------+
| EXPR1                                     |
+-------------------------------------------+
| <employee id="1">张三</employee>          |
+-------------------------------------------+

-- XMLFOREST：多列构造 XML 元素
SQL> SELECT XMLFOREST(name AS "Name", age AS "Age") FROM employees WHERE id = 1;
+--------------------------------------+
| EXPR1                                |
+--------------------------------------+
| <Name>张三</Name><Age>25</Age>       |
+--------------------------------------+

-- EXTRACTVALUE：提取 XML 文本值
SQL> SELECT EXTRACTVALUE(
    XMLTYPE('<root><name>虚谷数据库</name></root>'),
    '/root/name'
);
+-----------+
| EXPR1     |
+-----------+
| 虚谷数据库 |
+-----------+

-- EXTRACT (XML)：提取 XML 节点
SQL> SELECT EXTRACT(
    XMLTYPE('<root><item>A</item><item>B</item></root>'),
    '/root/item'
);
+-------------------------------+
| EXPR1                         |
+-------------------------------+
| <item>A</item><item>B</item> |
+-------------------------------+

-- XMLEXISTS：判断 XPath 是否匹配
SQL> SELECT XMLEXISTS('/root/name' PASSING XMLTYPE('<root><name>test</name></root>'));
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

-- XMLTABLE：将 XML 映射为表
SQL> SELECT x.*
FROM XMLTABLE(
    '/employees/emp'
    PASSING XMLTYPE('<employees><emp><name>张三</name><age>25</age></emp><emp><name>李四</name><age>30</age></emp></employees>')
    COLUMNS
        name VARCHAR(50) PATH 'name',
        age  INT         PATH 'age'
) x;
+------+-----+
| NAME | AGE |
+------+-----+
| 张三 |  25 |
| 李四 |  30 |
+------+-----+

-- XMLTYPE：构造 XML 类型
SQL> SELECT XMLTYPE('<root><data>hello</data></root>');
+------------------------------------+
| EXPR1                              |
+------------------------------------+
| <root><data>hello</data></root>    |
+------------------------------------+
```

### 与 Oracle/MySQL/PostgreSQL 对比

| 功能 | 虚谷数据库 | Oracle | PostgreSQL | MySQL |
|------|-----------|--------|------------|-------|
| 构造元素 | XMLELEMENT | XMLELEMENT | xmlelement | 无 |
| 构造属性 | XMLATTRIBUTES | XMLATTRIBUTES | xmlattributes | 无 |
| 构造森林 | XMLFOREST | XMLFOREST | xmlforest | 无 |
| 提取节点 | EXTRACT | EXTRACT (已废弃) | xpath | ExtractValue |
| 提取值 | EXTRACTVALUE | EXTRACTVALUE (已废弃) | (xpath(...))[1]::text | ExtractValue |
| 映射为表 | XMLTABLE | XMLTABLE | xmltable | 无 |
| XPath 存在 | XMLEXISTS | XMLEXISTS | xmlexists | 无 |
| 类型构造 | XMLTYPE | XMLTYPE | xml 类型 | 无 |

---

## 8. 大对象函数

大对象函数用于创建和操作 BLOB/CLOB 类型的数据。

| 函数 | 说明 | 语法 |
|------|------|------|
| EMPTY_BLOB | 返回空的 BLOB 值（Oracle 兼容） | `EMPTY_BLOB()` |
| EMPTY_CLOB | 返回空的 CLOB 值（Oracle 兼容） | `EMPTY_CLOB()` |
| TO_BLOB | 将数据转换为 BLOB 类型 | `TO_BLOB(data)` |

### 常用示例

```sql
-- EMPTY_BLOB / EMPTY_CLOB：初始化大对象列
SQL> CREATE TABLE docs (
    id INT PRIMARY KEY,
    content CLOB DEFAULT EMPTY_CLOB(),
    attachment BLOB DEFAULT EMPTY_BLOB()
);

SQL> INSERT INTO docs (id, content) VALUES (1, EMPTY_CLOB());

-- TO_BLOB：转换为 BLOB
SQL> SELECT TO_BLOB('48656C6C6F');
```

---

## 9. 数组函数

数组函数用于操作虚谷数据库的数组类型数据。

| 函数 | 说明 | 语法 |
|------|------|------|
| ARRAY_APPEND | 向数组末尾添加元素 | `ARRAY_APPEND(array, element)` |
| ARRAY_CAT | 拼接两个数组 | `ARRAY_CAT(array1, array2)` |
| ARRAY_DIMS | 返回数组的维度信息 | `ARRAY_DIMS(array)` |
| ARRAY_FILL | 创建用指定值填充的数组 | `ARRAY_FILL(value, dimensions)` |
| ARRAY_LENGTH | 返回数组指定维度的长度 | `ARRAY_LENGTH(array, dimension)` |
| ARRAY_LOWER | 返回数组指定维度的下界 | `ARRAY_LOWER(array, dimension)` |
| ARRAY_POSITION | 返回元素在数组中首次出现的位置 | `ARRAY_POSITION(array, element)` |
| ARRAY_POSITIONS | 返回元素在数组中所有出现位置 | `ARRAY_POSITIONS(array, element)` |
| ARRAY_PREPEND | 向数组头部添加元素 | `ARRAY_PREPEND(element, array)` |
| ARRAY_REMOVE | 从数组中删除所有匹配的元素 | `ARRAY_REMOVE(array, element)` |
| ARRAY_REPLACE | 将数组中所有匹配元素替换为新值 | `ARRAY_REPLACE(array, old_val, new_val)` |
| ARRAY_SAMPLE | 从数组中随机采样指定数量的元素 | `ARRAY_SAMPLE(array, count)` |
| ARRAY_SHUFFLE | 随机打乱数组元素顺序 | `ARRAY_SHUFFLE(array)` |
| ARRAY_UPPER | 返回数组指定维度的上界 | `ARRAY_UPPER(array, dimension)` |
| CARDINALITY | 返回数组中元素的总数 | `CARDINALITY(array)` |
| TRIM_ARRAY | 从数组末尾移除指定数量的元素 | `TRIM_ARRAY(array, count)` |

### 常用示例

```sql
-- ARRAY_APPEND：追加元素
SQL> SELECT ARRAY_APPEND(ARRAY[1,2,3], 4);
+-----------+
| EXPR1     |
+-----------+
| {1,2,3,4} |
+-----------+

-- ARRAY_PREPEND：头部添加
SQL> SELECT ARRAY_PREPEND(0, ARRAY[1,2,3]);
+-----------+
| EXPR1     |
+-----------+
| {0,1,2,3} |
+-----------+

-- ARRAY_CAT：拼接数组
SQL> SELECT ARRAY_CAT(ARRAY[1,2], ARRAY[3,4]);
+-----------+
| EXPR1     |
+-----------+
| {1,2,3,4} |
+-----------+

-- ARRAY_LENGTH：数组长度
SQL> SELECT ARRAY_LENGTH(ARRAY[10,20,30], 1);
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- CARDINALITY：元素总数
SQL> SELECT CARDINALITY(ARRAY[1,2,3,4,5]);
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+

-- ARRAY_POSITION / ARRAY_POSITIONS
SQL> SELECT ARRAY_POSITION(ARRAY['a','b','c','b'], 'b');
+-------+
| EXPR1 |
+-------+
|     2 |
+-------+

SQL> SELECT ARRAY_POSITIONS(ARRAY['a','b','c','b'], 'b');
+-------+
| EXPR1 |
+-------+
| {2,4} |
+-------+

-- ARRAY_REMOVE：删除匹配元素
SQL> SELECT ARRAY_REMOVE(ARRAY[1,2,3,2,1], 2);
+---------+
| EXPR1   |
+---------+
| {1,3,1} |
+---------+

-- ARRAY_REPLACE：替换元素
SQL> SELECT ARRAY_REPLACE(ARRAY[1,2,3,2,1], 2, 9);
+-----------+
| EXPR1     |
+-----------+
| {1,9,3,9,1} |
+-----------+

-- ARRAY_FILL：填充数组
SQL> SELECT ARRAY_FILL(0, ARRAY[5]);
+-------------+
| EXPR1       |
+-------------+
| {0,0,0,0,0} |
+-------------+

-- ARRAY_DIMS：维度信息
SQL> SELECT ARRAY_DIMS(ARRAY[[1,2],[3,4]]);
+-----------+
| EXPR1     |
+-----------+
| [1:2][1:2]|
+-----------+

-- ARRAY_LOWER / ARRAY_UPPER
SQL> SELECT ARRAY_LOWER(ARRAY[10,20,30], 1);
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT ARRAY_UPPER(ARRAY[10,20,30], 1);
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- TRIM_ARRAY：从末尾移除元素
SQL> SELECT TRIM_ARRAY(ARRAY[1,2,3,4,5], 2);
+---------+
| EXPR1   |
+---------+
| {1,2,3} |
+---------+

-- ARRAY_SAMPLE：随机采样
SQL> SELECT ARRAY_SAMPLE(ARRAY[1,2,3,4,5], 3);
+---------+
| EXPR1   |
+---------+
| {2,5,1} |
+---------+

-- ARRAY_SHUFFLE：随机打乱
SQL> SELECT ARRAY_SHUFFLE(ARRAY[1,2,3,4,5]);
+-----------+
| EXPR1     |
+-----------+
| {3,1,5,2,4} |
+-----------+
```

### 与 Oracle/MySQL/PostgreSQL 对比

| 功能 | 虚谷数据库 | PostgreSQL | Oracle | MySQL |
|------|-----------|------------|--------|-------|
| 追加元素 | ARRAY_APPEND | array_append | 无（VARRAY 操作） | 无 |
| 头部添加 | ARRAY_PREPEND | array_prepend | 无 | 无 |
| 拼接数组 | ARRAY_CAT | array_cat / `||` | 无 | 无 |
| 数组长度 | ARRAY_LENGTH | array_length | 无 | 无 |
| 元素总数 | CARDINALITY | cardinality | 无 | 无 |
| 查找位置 | ARRAY_POSITION | array_position | 无 | 无 |
| 删除元素 | ARRAY_REMOVE | array_remove | 无 | 无 |
| 替换元素 | ARRAY_REPLACE | array_replace | 无 | 无 |

> **说明：** 虚谷数据库的数组函数与 PostgreSQL 高度兼容。Oracle 使用 VARRAY/嵌套表实现类似功能，MySQL 不原生支持数组类型。

---

## 10. 网络地址函数

网络地址函数用于 IP 地址与整数之间的转换。

| 函数 | 说明 | 语法 |
|------|------|------|
| INET_ATON | 将 IPv4 地址字符串转换为整数 | `INET_ATON(ip_string)` |
| INET_NTOA | 将整数转换为 IPv4 地址字符串 | `INET_NTOA(integer)` |

### 常用示例

```sql
-- INET_ATON：IP 地址转整数
SQL> SELECT INET_ATON('192.168.1.1');
+------------+
| EXPR1      |
+------------+
| 3232235777 |
+------------+

SQL> SELECT INET_ATON('10.0.0.1');
+-----------+
| EXPR1     |
+-----------+
| 167772161 |
+-----------+

-- INET_NTOA：整数转 IP 地址
SQL> SELECT INET_NTOA(3232235777);
+-------------+
| EXPR1       |
+-------------+
| 192.168.1.1 |
+-------------+

-- 实际应用：存储和查询 IP 地址
SQL> CREATE TABLE access_log (id INT, ip_addr BIGINT);
SQL> INSERT INTO access_log VALUES (1, INET_ATON('192.168.1.100'));
SQL> SELECT id, INET_NTOA(ip_addr) AS ip FROM access_log;
```

---

## 11. 序列值函数

| 函数 | 说明 | 语法 |
|------|------|------|
| CURRVAL | 返回指定序列的当前值（在当前会话中最近一次 NEXTVAL 获取的值） | `CURRVAL('sequence_name')` 或 `sequence_name.CURRVAL` |

### 常用示例

```sql
-- 创建序列
SQL> CREATE SEQUENCE my_seq START WITH 1 INCREMENT BY 1;

-- 获取下一个值
SQL> SELECT my_seq.NEXTVAL;
+---------+
| NEXTVAL |
+---------+
|       1 |
+---------+

-- 获取当前值（必须先调用 NEXTVAL）
SQL> SELECT CURRVAL('my_seq');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

-- 或使用点号语法
SQL> SELECT my_seq.CURRVAL;
+---------+
| CURRVAL |
+---------+
|       1 |
+---------+
```

---

## 12. 系统信息函数

系统信息函数用于获取数据库实例、会话和环境的各种信息。

| 函数 | 说明 |
|------|------|
| CHARSET | 返回当前数据库字符集 |
| COERCIBILITY | 返回字符串排序规则的优先级 |
| COLLATION | 返回字符串的排序规则 |
| CONNECTION_ID | 返回当前连接的 ID |
| CURRENT_CATALOG | 返回当前数据库名 |
| CURRENT_DATABASE | 返回当前数据库名 |
| CURRENT_DB | 返回当前数据库名 |
| CURRENT_ROLE | 返回当前角色 |
| CURRENT_SCHEMA | 返回当前模式名 |
| CURRENT_USER | 返回当前用户名 |
| DATABASE | 返回当前数据库名 |
| DB_ID | 返回数据库 ID |
| DB_NAME | 返回数据库名 |
| FOUND_ROWS | 返回上一条 SELECT 匹配的行数（忽略 LIMIT） |
| HOST_NAME | 返回服务器主机名 |
| LAST_INSERT_ID | 返回最后插入行的自增 ID 值 |
| OBJECT_ID | 返回对象的 ID |
| OBJECT_NAME | 返回对象 ID 对应的名称 |
| ORA_HASH | 计算表达式的 Oracle 兼容哈希值 |
| PG_BACKEND_PID | 返回当前后端进程 ID（PostgreSQL 兼容） |
| ROW_COUNT | 返回上一条 DML 语句影响的行数 |
| SCHEMA_NAME | 返回模式名 |
| SESSION_USER | 返回会话用户名 |
| SYS_CONTEXT | 获取系统上下文信息（Oracle 兼容） |
| SYSTEM_USER | 返回系统用户名 |
| TABLESPACE_NAME | 返回表空间名 |
| USER | 返回当前用户名 |
| USERENV | 返回用户环境信息（Oracle 兼容） |
| USER_ID | 返回用户 ID |
| USER_NAME | 返回用户名 |
| VERSION | 返回数据库版本信息 |
| VERSION_COMMENT | 返回数据库版本注释 |
| VERSION_COMPILE_MACHINE | 返回编译平台信息 |
| VERSION_COMPILE_OS | 返回编译操作系统信息 |
| XG_VERSION | 返回虚谷数据库内部版本号 |
| XUGU_VERSION | 返回虚谷数据库版本信息 |

### 常用示例

```sql
-- 查询当前用户
SQL> SELECT CURRENT_USER;
+--------------+
| CURRENT_USER |
+--------------+
| SYSDBA       |
+--------------+

-- 查询当前数据库
SQL> SELECT CURRENT_DATABASE();
+--------+
| EXPR1  |
+--------+
| SYSTEM |
+--------+

-- 查询数据库版本
SQL> SELECT VERSION();
+-----------------------+
| EXPR1                 |
+-----------------------+
| XuguDB V12.9.0 ...   |
+-----------------------+

SQL> SELECT XUGU_VERSION();
+-----------------------+
| EXPR1                 |
+-----------------------+
| XuguDB V12.9.0 ...   |
+-----------------------+

-- 查询连接 ID
SQL> SELECT CONNECTION_ID();
+-------+
| EXPR1 |
+-------+
|    42 |
+-------+

-- 查询当前模式
SQL> SELECT CURRENT_SCHEMA;
+----------------+
| CURRENT_SCHEMA |
+----------------+
| SYSDBA         |
+----------------+

-- 查询字符集
SQL> SELECT CHARSET('虚谷');
+-------+
| EXPR1 |
+-------+
| UTF8  |
+-------+

-- SYS_CONTEXT（Oracle 兼容）
SQL> SELECT SYS_CONTEXT('USERENV', 'SESSION_USER');
+--------+
| EXPR1  |
+--------+
| SYSDBA |
+--------+

-- USERENV（Oracle 兼容）
SQL> SELECT USERENV('LANGUAGE');
+------------------+
| EXPR1            |
+------------------+
| CHINESE_CHINA.UTF8 |
+------------------+

-- LAST_INSERT_ID
SQL> CREATE TABLE t1 (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50));
SQL> INSERT INTO t1 (name) VALUES ('test');
SQL> SELECT LAST_INSERT_ID();
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

-- ORA_HASH
SQL> SELECT ORA_HASH('hello');
+------------+
| EXPR1      |
+------------+
| 3292163858 |
+------------+

-- OBJECT_ID / OBJECT_NAME
SQL> SELECT OBJECT_ID('SYSDBA', 'my_table');
SQL> SELECT OBJECT_NAME(12345);
```

---

## 13. 系统管理函数

系统管理函数用于数据库管理操作，通常需要管理员权限。

| 函数 | 说明 |
|------|------|
| FLUSH_LOG | 刷新日志缓冲区到磁盘 |
| FLUSH_TRAN | 刷新事务缓冲区 |
| KILL_SESSION | 终止指定的会话 |
| LEVEL_LOCK | 获取指定级别的锁 |
| LEVEL_UNLOCK | 释放指定级别的锁 |
| LOAD_PLUGIN | 加载数据库插件 |
| LOCK_TABLE | 锁定指定表 |
| REMOVE_PLUGIN | 卸载数据库插件 |
| SET_PARALLEL | 设置并行执行度 |
| SET_POLICY | 设置安全策略 |
| SET_TRACE | 设置跟踪级别 |
| SLEEP | 使当前会话暂停指定时间（秒） |
| SWITCH_LOG | 切换日志文件 |
| TABLESPACE_USAGE | 查询表空间使用情况 |
| UNLOCK_TABLE | 解锁指定表 |
| UPDATE_STATS | 更新表或索引的统计信息 |
| VALIDATE_INDEX | 验证索引的完整性 |

### 常用示例

```sql
-- SLEEP：暂停执行
SQL> SELECT SLEEP(3);  -- 暂停 3 秒
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+

-- KILL_SESSION：终止会话
SQL> SELECT KILL_SESSION(42);

-- UPDATE_STATS：更新统计信息
SQL> SELECT UPDATE_STATS('SYSDBA', 'my_table');

-- FLUSH_LOG：刷新日志
SQL> SELECT FLUSH_LOG();

-- SET_TRACE：设置跟踪（调试用）
SQL> SELECT SET_TRACE(1);

-- TABLESPACE_USAGE：查询表空间使用
SQL> SELECT TABLESPACE_USAGE('SYSTEM');

-- VALIDATE_INDEX：验证索引完整性
SQL> SELECT VALIDATE_INDEX('SYSDBA', 'my_index');

-- LOCK_TABLE / UNLOCK_TABLE
SQL> SELECT LOCK_TABLE('SYSDBA', 'my_table');
SQL> SELECT UNLOCK_TABLE('SYSDBA', 'my_table');

-- LEVEL_LOCK / LEVEL_UNLOCK
SQL> SELECT LEVEL_LOCK('my_lock', 10);  -- 获取锁，超时 10 秒
SQL> SELECT LEVEL_UNLOCK('my_lock');
```

---

## 14. 几何函数

几何函数用于操作虚谷数据库的几何数据类型（POINT、LINE、LSEG、BOX、PATH、POLYGON、CIRCLE）。

| 函数 | 说明 |
|------|------|
| AREA | 计算几何对象的面积 |
| CENTER | 返回几何对象的中心点 |
| CIRCLE | 构造圆 |
| DIAMETER | 返回圆的直径 |
| HEIGHT | 返回 BOX 的高度 |
| ISCLOSED | 判断路径是否封闭 |
| ISOPEN | 判断路径是否开放 |
| LENGTH (几何) | 计算几何对象的长度/周长 |
| LINE | 构造直线 |
| LSEG | 构造线段 |
| NPOINTS | 返回几何对象的顶点数 |
| PATH | 构造路径 |
| PCLOSE | 将开放路径转换为封闭路径 |
| POINT | 构造点 |
| POLYGON | 构造多边形 |
| POPEN | 将封闭路径转换为开放路径 |
| RADIUS | 返回圆的半径 |
| SLOPE | 计算线段的斜率 |
| WIDTH | 返回 BOX 的宽度 |
| BOX | 构造矩形 |
| BOUND_BOX | 返回几何对象的外接矩形 |

### 常用示例

```sql
-- POINT：构造点
SQL> SELECT POINT(3.0, 4.0);
+---------+
| EXPR1   |
+---------+
| (3,4)   |
+---------+

-- LINE：构造直线
SQL> SELECT LINE(POINT(0,0), POINT(1,1));

-- LSEG：构造线段
SQL> SELECT LSEG(POINT(0,0), POINT(3,4));

-- BOX：构造矩形
SQL> SELECT BOX(POINT(0,0), POINT(3,4));
+--------------+
| EXPR1        |
+--------------+
| (3,4),(0,0)  |
+--------------+

-- CIRCLE：构造圆
SQL> SELECT CIRCLE(POINT(0,0), 5);
+-------------+
| EXPR1       |
+-------------+
| <(0,0),5>   |
+-------------+

-- AREA：计算面积
SQL> SELECT AREA(BOX(POINT(0,0), POINT(3,4)));
+-------+
| EXPR1 |
+-------+
|    12 |
+-------+

SQL> SELECT AREA(CIRCLE(POINT(0,0), 5));
+-------------------+
| EXPR1             |
+-------------------+
| 78.53981633974483 |
+-------------------+

-- CENTER：获取中心点
SQL> SELECT CENTER(BOX(POINT(0,0), POINT(4,6)));
+---------+
| EXPR1   |
+---------+
| (2,3)   |
+---------+

SQL> SELECT CENTER(CIRCLE(POINT(1,2), 5));
+---------+
| EXPR1   |
+---------+
| (1,2)   |
+---------+

-- HEIGHT / WIDTH：BOX 的高度和宽度
SQL> SELECT HEIGHT(BOX(POINT(0,0), POINT(3,4)));
+-------+
| EXPR1 |
+-------+
|     4 |
+-------+

SQL> SELECT WIDTH(BOX(POINT(0,0), POINT(3,4)));
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- RADIUS / DIAMETER：圆的半径和直径
SQL> SELECT RADIUS(CIRCLE(POINT(0,0), 5));
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+

SQL> SELECT DIAMETER(CIRCLE(POINT(0,0), 5));
+-------+
| EXPR1 |
+-------+
|    10 |
+-------+

-- LENGTH（几何）：计算长度/周长
SQL> SELECT LENGTH(LSEG(POINT(0,0), POINT(3,4)));
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+

-- NPOINTS：返回顶点数
SQL> SELECT NPOINTS(PATH '((0,0),(1,1),(2,0))');
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- SLOPE：计算斜率
SQL> SELECT SLOPE(POINT(0,0), POINT(3,6));
+-------+
| EXPR1 |
+-------+
|     2 |
+-------+

-- ISCLOSED / ISOPEN
SQL> SELECT ISCLOSED(PATH '((0,0),(1,1),(2,0))');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

-- PCLOSE / POPEN：路径开闭转换
SQL> SELECT PCLOSE(PATH '[(0,0),(1,1),(2,0)]');  -- 开放路径转封闭
SQL> SELECT POPEN(PATH '((0,0),(1,1),(2,0))');    -- 封闭路径转开放

-- BOUND_BOX：外接矩形
SQL> SELECT BOUND_BOX(CIRCLE(POINT(0,0), 5));
```

### 与 Oracle/MySQL/PostgreSQL 对比

| 功能 | 虚谷数据库 | PostgreSQL | Oracle | MySQL |
|------|-----------|------------|--------|-------|
| 几何类型 | POINT/LINE/BOX/CIRCLE 等 | 相同类型 | SDO_GEOMETRY | 无原生支持 |
| 面积 | AREA | area | SDO_GEOM.SDO_AREA | 无 |
| 中心 | CENTER | center | SDO_GEOM.SDO_CENTROID | 无 |
| 长度 | LENGTH | length | SDO_GEOM.SDO_LENGTH | 无 |

> **说明：** 虚谷数据库的几何函数与 PostgreSQL 的基础几何类型函数兼容。如需 GIS 级空间功能，请参考空间数据库技能模块。

---

## 15. 其他函数

此类别包含不归属于上述分类的通用函数。

| 函数 | 说明 |
|------|------|
| BIN_TO_NUM | 将二进制位序列转换为数字 |
| CAST | 将表达式转换为指定数据类型 |
| COALESCE | 返回参数列表中第一个非 NULL 值 |
| CONVERT | 转换字符集编码 |
| DECODE | 条件匹配函数（Oracle 兼容） |
| DEFAULT | 返回列的默认值 |
| DUMP | 返回值的内部表示信息 |
| GREATEST | 返回参数列表中的最大值 |
| GROUP_ID | 在 GROUP BY 中返回重复分组的标识 |
| GROUPING | 判断列是否为 ROLLUP/CUBE 产生的汇总行 |
| GROUPING_ID | 返回 GROUPING 值的位图整数 |
| HASH | 计算哈希值 |
| HASHBYTES | 计算指定算法的哈希值 |
| LEAST | 返回参数列表中的最小值 |
| NVL | 若第一个参数为 NULL 则返回第二个参数（Oracle 兼容） |
| NVL2 | 根据第一个参数是否为 NULL 返回不同值（Oracle 兼容） |
| RAISE_APPLICATION_ERROR | 抛出自定义应用错误（Oracle 兼容） |
| RAWTOHEX | 将 RAW 类型转换为十六进制字符串 |
| HEXTORAW | 将十六进制字符串转换为 RAW 类型 |
| ROWIDTOCHAR | 将 ROWID 转换为字符串 |
| SENDMSG | 发送消息 |
| SEQ_NEXTVAL | 获取序列的下一个值 |
| SIZEOF | 返回数据的字节大小 |
| SYS_EXTRACT_UTC | 从带时区的时间戳提取 UTC 时间 |
| TO_SINGLE_BYTE | 将多字节字符转换为单字节字符 |
| TO_MULTI_BYTE | 将单字节字符转换为多字节字符 |
| TYPEOF | 返回表达式的数据类型名称 |
| VSIZE | 返回值的内部存储大小（字节） |
| WIDTH_BUCKET | 计算值在等宽直方图中所属的桶号 |

### 常用示例

```sql
-- CAST：类型转换
SQL> SELECT CAST('2024-01-15' AS DATE);
+------------+
| EXPR1      |
+------------+
| 2024-01-15 |
+------------+

SQL> SELECT CAST(3.14 AS INT);
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

-- COALESCE：返回第一个非 NULL 值
SQL> SELECT COALESCE(NULL, NULL, '第三个', '第四个');
+--------+
| EXPR1  |
+--------+
| 第三个 |
+--------+

-- NVL（Oracle 兼容）
SQL> SELECT NVL(NULL, '默认值');
+--------+
| EXPR1  |
+--------+
| 默认值 |
+--------+

-- NVL2（Oracle 兼容）
SQL> SELECT NVL2('有值', '非空', '为空');
+------+
| EXPR1|
+------+
| 非空 |
+------+

SQL> SELECT NVL2(NULL, '非空', '为空');
+------+
| EXPR1|
+------+
| 为空 |
+------+

-- DECODE（Oracle 兼容）
SQL> SELECT DECODE(2, 1, '一', 2, '二', 3, '三', '其他');
+------+
| EXPR1|
+------+
| 二   |
+------+

-- GREATEST / LEAST
SQL> SELECT GREATEST(10, 20, 5, 30, 15);
+-------+
| EXPR1 |
+-------+
|    30 |
+-------+

SQL> SELECT LEAST(10, 20, 5, 30, 15);
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+

-- BIN_TO_NUM
SQL> SELECT BIN_TO_NUM(1, 0, 1, 1);
+-------+
| EXPR1 |
+-------+
|    11 |
+-------+

-- DUMP：内部表示信息
SQL> SELECT DUMP(12345);
+-------------------------------+
| EXPR1                         |
+-------------------------------+
| Typ=2 Len=4: 195,2,24,46     |
+-------------------------------+

-- VSIZE：内部存储大小
SQL> SELECT VSIZE('hello');
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+

-- SIZEOF：数据字节大小
SQL> SELECT SIZEOF(12345);
+-------+
| EXPR1 |
+-------+
|     4 |
+-------+

-- TYPEOF：返回数据类型
SQL> SELECT TYPEOF(123);
+---------+
| EXPR1   |
+---------+
| INTEGER |
+---------+

SQL> SELECT TYPEOF('hello');
+---------+
| EXPR1   |
+---------+
| CHAR    |
+---------+

-- HASH / HASHBYTES
SQL> SELECT HASH('hello');
+-------------------+
| EXPR1             |
+-------------------+
| 907060870         |
+-------------------+

SQL> SELECT HASHBYTES('SHA2_256', 'hello');

-- CONVERT：字符集转换
SQL> SELECT CONVERT('hello', 'UTF8', 'GBK');

-- RAWTOHEX / HEXTORAW
SQL> SELECT RAWTOHEX('ABC');
+--------+
| EXPR1  |
+--------+
| 414243 |
+--------+

SQL> SELECT HEXTORAW('414243');
+-------+
| EXPR1 |
+-------+
| ABC   |
+-------+

-- WIDTH_BUCKET：直方图分桶
SQL> SELECT WIDTH_BUCKET(75, 0, 100, 10);
+-------+
| EXPR1 |
+-------+
|     8 |
+-------+

-- GROUPING / GROUPING_ID（配合 ROLLUP/CUBE 使用）
SQL> SELECT dept, job, SUM(salary),
       GROUPING(dept) AS g_dept,
       GROUPING(job) AS g_job
FROM employees
GROUP BY ROLLUP(dept, job);

-- SYS_EXTRACT_UTC：提取 UTC 时间
SQL> SELECT SYS_EXTRACT_UTC(SYSTIMESTAMP);

-- RAISE_APPLICATION_ERROR（PL/SQL 中使用）
-- RAISE_APPLICATION_ERROR(-20001, '自定义错误信息');
```

### 与 Oracle/MySQL/PostgreSQL 对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 类型转换 | CAST | CAST | CAST | CAST |
| 首个非 NULL | COALESCE | COALESCE / NVL | COALESCE / IFNULL | COALESCE |
| NULL 替换 | NVL | NVL | IFNULL | COALESCE |
| 条件匹配 | DECODE | DECODE | 无（用 CASE） | 无（用 CASE） |
| 最大值 | GREATEST | GREATEST | GREATEST | GREATEST |
| 最小值 | LEAST | LEAST | LEAST | LEAST |
| 内部表示 | DUMP | DUMP | 无 | 无 |
| 存储大小 | VSIZE | VSIZE | 无 | pg_column_size |
| 直方图分桶 | WIDTH_BUCKET | WIDTH_BUCKET | 无 | width_bucket |
| 分组判断 | GROUPING | GROUPING | GROUPING (8.0+) | GROUPING |
