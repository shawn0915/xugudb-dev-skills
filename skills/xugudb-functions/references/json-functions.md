---
title: 虚谷数据库 JSON 函数参考
description: JSON 构造、查询提取、修改、验证、搜索函数的完整参考，包括 JSON_ARRAY、JSON_OBJECT、JSON_EXTRACT、JSON_SET、JSON_VALID 等 28 个函数的语法、参数说明与 SQL 示例，以及与 Oracle/MySQL/PostgreSQL 对比
tags: [xugudb, json, json-functions, json_extract, json_object, json_array, json_set, json_value, json_merge, json_valid]
---

# 虚谷数据库 JSON 函数参考

虚谷数据库提供 28 个 JSON 函数，覆盖 JSON 数据的构造、查询提取、修改、验证与搜索等场景。JSON 函数主要操作 JSON 类型或可被解析为 JSON 的字符串。

> **提示：** 虚谷数据库同时支持 `->` 和 `->>` JSON 运算符（参见运算符参考），可与 JSON 函数配合使用。

---

## 目录

1. [JSON 构造函数](#1-json-构造函数)
2. [JSON 查询与提取函数](#2-json-查询与提取函数)
3. [JSON 修改函数](#3-json-修改函数)
4. [JSON 验证与类型函数](#4-json-验证与类型函数)
5. [JSON 搜索与比较函数](#5-json-搜索与比较函数)
6. [JSON 格式化与转换函数](#6-json-格式化与转换函数)
7. [JSON 合并函数](#7-json-合并函数)
8. [JSON Schema 验证函数](#8-json-schema-验证函数)
9. [JSON 函数速查表](#9-json-函数速查表)
10. [与 Oracle/MySQL/PostgreSQL 对比](#10-与-oraclemysqlpostgresql-对比)

---

## 1. JSON 构造函数

### JSON_ARRAY

**功能描述：** 根据给定的值列表构造一个 JSON 数组。

**语法格式：**

```sql
JSON_ARRAY(val1 [, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| val1, val2, ... | 任意数量的值，可以是字符串、数字、布尔值、NULL 或其他 JSON 值 |

**返回类型：** JSON

**示例：**

```sql
-- 构造简单 JSON 数组
SQL> SELECT JSON_ARRAY(1, 2, 'abc', NULL);
+-------------------+
| EXPR1             |
+-------------------+
| [1, 2, "abc", null] |
+-------------------+

-- 嵌套 JSON 数组
SQL> SELECT JSON_ARRAY(1, JSON_ARRAY(2, 3));
+-------------+
| EXPR1       |
+-------------+
| [1, [2, 3]] |
+-------------+
```

---

### JSON_OBJECT

**功能描述：** 根据给定的键值对构造一个 JSON 对象。

**语法格式：**

```sql
JSON_OBJECT(key1, val1 [, key2, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| key | 键名，字符串类型 |
| val | 对应的值，可以是字符串、数字、布尔值、NULL 或其他 JSON 值 |

**返回类型：** JSON

**示例：**

```sql
-- 构造 JSON 对象
SQL> SELECT JSON_OBJECT('name', '张三', 'age', 25);
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 25} |
+-----------------------------+

-- 嵌套对象
SQL> SELECT JSON_OBJECT('user', JSON_OBJECT('name', '李四', 'role', 'admin'));
+------------------------------------------+
| EXPR1                                    |
+------------------------------------------+
| {"user": {"name": "李四", "role": "admin"}} |
+------------------------------------------+
```

---

## 2. JSON 查询与提取函数

### JSON_EXTRACT

**功能描述：** 从 JSON 文档中提取指定路径的值。返回结果保留 JSON 格式。

**语法格式：**

```sql
JSON_EXTRACT(json_doc, path [, path2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | JSON 文档（JSON 类型或可解析为 JSON 的字符串） |
| path | JSON 路径表达式，以 `$` 开头 |

**返回类型：** JSON。若路径不存在或输入为 NULL，返回 NULL。

**示例：**

```sql
-- 提取对象中的字段
SQL> SELECT JSON_EXTRACT('{"name":"张三","age":25}', '$.name');
+--------+
| EXPR1  |
+--------+
| "张三" |
+--------+

-- 提取数组元素
SQL> SELECT JSON_EXTRACT('[10,20,30]', '$[1]');
+-------+
| EXPR1 |
+-------+
| 20    |
+-------+

-- 提取多个路径
SQL> SELECT JSON_EXTRACT('{"a":1,"b":2,"c":3}', '$.a', '$.c');
+--------+
| EXPR1  |
+--------+
| [1, 3] |
+--------+
```

---

### JSON_VALUE

**功能描述：** 从 JSON 文档中提取指定路径的标量值，返回字符串类型（非 JSON 格式）。与 JSON_EXTRACT 不同，JSON_VALUE 对字符串结果不保留引号。

**语法格式：**

```sql
JSON_VALUE(json_doc, path)
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | JSON 文档 |
| path | JSON 路径表达式 |

**返回类型：** CHAR。若路径不存在或结果为非标量值（对象/数组），返回 NULL。

**示例：**

```sql
-- 提取标量值（不带引号）
SQL> SELECT JSON_VALUE('{"name":"张三","age":25}', '$.name');
+------+
| EXPR1|
+------+
| 张三 |
+------+

-- 与 JSON_EXTRACT 对比
SQL> SELECT JSON_EXTRACT('{"name":"张三"}', '$.name') AS extracted,
            JSON_VALUE('{"name":"张三"}', '$.name') AS value_of;
+-----------+----------+
| EXTRACTED | VALUE_OF |
+-----------+----------+
| "张三"    | 张三     |
+-----------+----------+
```

---

### JSON_KEYS

**功能描述：** 返回 JSON 对象中的所有顶层键名组成的 JSON 数组。如果指定了路径，则返回该路径对应对象的键名。

**语法格式：**

```sql
JSON_KEYS(json_doc [, path])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | JSON 文档 |
| path | 可选，JSON 路径表达式，指向一个 JSON 对象 |

**返回类型：** JSON（数组）。若输入为 NULL 或路径指向非对象，返回 NULL。

**示例：**

```sql
SQL> SELECT JSON_KEYS('{"name":"张三","age":25,"city":"北京"}');
+---------------------------+
| EXPR1                     |
+---------------------------+
| ["name", "age", "city"]   |
+---------------------------+

SQL> SELECT JSON_KEYS('{"a":{"x":1,"y":2},"b":3}', '$.a');
+------------+
| EXPR1      |
+------------+
| ["x", "y"] |
+------------+
```

---

### JSON_LENGTH

**功能描述：** 返回 JSON 文档的长度。对于对象返回键数量，对于数组返回元素数量，对于标量值返回 1。

**语法格式：**

```sql
JSON_LENGTH(json_doc [, path])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | JSON 文档 |
| path | 可选，JSON 路径表达式 |

**返回类型：** INTEGER

**示例：**

```sql
SQL> SELECT JSON_LENGTH('{"a":1,"b":2,"c":3}');
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+

SQL> SELECT JSON_LENGTH('[1,2,3,4,5]');
+-------+
| EXPR1 |
+-------+
|     5 |
+-------+

SQL> SELECT JSON_LENGTH('{"a":[1,2,3]}', '$.a');
+-------+
| EXPR1 |
+-------+
|     3 |
+-------+
```

---

### JSON_DEPTH

**功能描述：** 返回 JSON 文档的最大嵌套深度。空数组、空对象或标量值的深度为 1。

**语法格式：**

```sql
JSON_DEPTH(json_doc)
```

**返回类型：** INTEGER

**示例：**

```sql
SQL> SELECT JSON_DEPTH('{}');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_DEPTH('{"a":{"b":{"c":1}}}');
+-------+
| EXPR1 |
+-------+
|     4 |
+-------+

SQL> SELECT JSON_DEPTH('[1,[2,[3]]]');
+-------+
| EXPR1 |
+-------+
|     4 |
+-------+
```

---

### JSON_TYPE

**功能描述：** 返回 JSON 值的类型，结果为描述类型的字符串。

**语法格式：**

```sql
JSON_TYPE(json_val)
```

**返回类型：** CHAR，可能的返回值包括：OBJECT、ARRAY、STRING、INTEGER、DOUBLE、BOOLEAN、NULL。

**示例：**

```sql
SQL> SELECT JSON_TYPE('{"a":1}');
+--------+
| EXPR1  |
+--------+
| OBJECT |
+--------+

SQL> SELECT JSON_TYPE('[1,2]');
+-------+
| EXPR1 |
+-------+
| ARRAY |
+-------+

SQL> SELECT JSON_TYPE('"hello"');
+--------+
| EXPR1  |
+--------+
| STRING |
+--------+

SQL> SELECT JSON_TYPE('123');
+---------+
| EXPR1   |
+---------+
| INTEGER |
+---------+
```

---

## 3. JSON 修改函数

### JSON_SET

**功能描述：** 在 JSON 文档中设置指定路径的值。如果路径已存在则替换，不存在则插入。

**语法格式：**

```sql
JSON_SET(json_doc, path, val [, path2, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | 原始 JSON 文档 |
| path | JSON 路径表达式 |
| val | 要设置的新值 |

**返回类型：** JSON

**示例：**

```sql
-- 替换已有值
SQL> SELECT JSON_SET('{"name":"张三","age":25}', '$.age', 26);
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 26} |
+-----------------------------+

-- 插入新键值
SQL> SELECT JSON_SET('{"name":"张三"}', '$.age', 25);
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 25} |
+-----------------------------+
```

---

### JSON_INSERT

**功能描述：** 向 JSON 文档中插入新值。仅在路径不存在时插入，已存在的路径不做修改。

**语法格式：**

```sql
JSON_INSERT(json_doc, path, val [, path2, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | 原始 JSON 文档 |
| path | JSON 路径表达式 |
| val | 要插入的值 |

**返回类型：** JSON

**示例：**

```sql
-- 路径不存在，插入成功
SQL> SELECT JSON_INSERT('{"name":"张三"}', '$.age', 25);
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 25} |
+-----------------------------+

-- 路径已存在，不做修改
SQL> SELECT JSON_INSERT('{"name":"张三","age":25}', '$.age', 30);
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 25} |
+-----------------------------+
```

---

### JSON_REPLACE

**功能描述：** 替换 JSON 文档中已有路径的值。仅在路径存在时替换，不存在的路径不做处理。

**语法格式：**

```sql
JSON_REPLACE(json_doc, path, val [, path2, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | 原始 JSON 文档 |
| path | JSON 路径表达式 |
| val | 要替换的新值 |

**返回类型：** JSON

**示例：**

```sql
-- 路径存在，替换成功
SQL> SELECT JSON_REPLACE('{"name":"张三","age":25}', '$.age', 30);
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 30} |
+-----------------------------+

-- 路径不存在，不做处理
SQL> SELECT JSON_REPLACE('{"name":"张三"}', '$.age', 25);
+-----------------+
| EXPR1           |
+-----------------+
| {"name": "张三"} |
+-----------------+
```

---

### JSON_REMOVE

**功能描述：** 从 JSON 文档中删除指定路径的数据。

**语法格式：**

```sql
JSON_REMOVE(json_doc, path [, path2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | 原始 JSON 文档 |
| path | 要删除的路径表达式 |

**返回类型：** JSON

**示例：**

```sql
SQL> SELECT JSON_REMOVE('{"name":"张三","age":25,"city":"北京"}', '$.city');
+-----------------------------+
| EXPR1                       |
+-----------------------------+
| {"name": "张三", "age": 25} |
+-----------------------------+

-- 删除数组元素
SQL> SELECT JSON_REMOVE('[1,2,3,4]', '$[1]');
+-----------+
| EXPR1     |
+-----------+
| [1, 3, 4] |
+-----------+
```

---

### JSON_ARRAY_APPEND

**功能描述：** 向 JSON 文档中指定路径的数组末尾追加值。如果路径指向的是标量值或对象，则自动将其包装为数组后追加。

**语法格式：**

```sql
JSON_ARRAY_APPEND(json_doc, path, val [, path2, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | 原始 JSON 文档 |
| path | JSON 路径表达式，指向一个数组 |
| val | 要追加的值 |

**返回类型：** JSON

**示例：**

```sql
SQL> SELECT JSON_ARRAY_APPEND('[1,2,3]', '$', 4);
+--------------+
| EXPR1        |
+--------------+
| [1, 2, 3, 4] |
+--------------+

SQL> SELECT JSON_ARRAY_APPEND('{"items":[1,2]}', '$.items', 3);
+--------------------+
| EXPR1              |
+--------------------+
| {"items": [1, 2, 3]} |
+--------------------+
```

---

### JSON_ARRAY_INSERT

**功能描述：** 在 JSON 数组的指定位置插入值，后续元素向后移动。

**语法格式：**

```sql
JSON_ARRAY_INSERT(json_doc, path, val [, path2, val2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | 原始 JSON 文档 |
| path | JSON 路径表达式，需包含数组索引，如 `$[1]` |
| val | 要插入的值 |

**返回类型：** JSON

**示例：**

```sql
SQL> SELECT JSON_ARRAY_INSERT('[1,2,3]', '$[1]', 'new');
+---------------------+
| EXPR1               |
+---------------------+
| [1, "new", 2, 3]    |
+---------------------+

SQL> SELECT JSON_ARRAY_INSERT('{"items":["a","b"]}', '$.items[0]', 'first');
+-------------------------------+
| EXPR1                         |
+-------------------------------+
| {"items": ["first", "a", "b"]} |
+-------------------------------+
```

---

## 4. JSON 验证与类型函数

### JSON_VALID

**功能描述：** 检查给定的字符串是否为有效的 JSON 文档。

**语法格式：**

```sql
JSON_VALID(val)
```

**返回类型：** INTEGER，有效 JSON 返回 1，无效返回 0，NULL 输入返回 NULL。

**示例：**

```sql
SQL> SELECT JSON_VALID('{"name":"张三"}');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_VALID('{invalid}');
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+

SQL> SELECT JSON_VALID(NULL);
+-------+
| EXPR1 |
+-------+
| NULL  |
+-------+
```

---

## 5. JSON 搜索与比较函数

### JSON_CONTAINS

**功能描述：** 判断 JSON 文档是否包含指定的候选值。可选地在指定路径下进行判断。

**语法格式：**

```sql
JSON_CONTAINS(target, candidate [, path])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| target | 被搜索的 JSON 文档 |
| candidate | 要查找的候选 JSON 值 |
| path | 可选，在 target 中搜索的路径 |

**返回类型：** INTEGER，包含返回 1，不包含返回 0。

**示例：**

```sql
SQL> SELECT JSON_CONTAINS('[1,2,3,4,5]', '3');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_CONTAINS('{"a":1,"b":2}', '1', '$.a');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_CONTAINS('[1,2,3]', '[1,3]');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+
```

---

### JSON_CONTAINS_PATH

**功能描述：** 判断 JSON 文档中是否存在指定的路径。可通过 `one` 或 `all` 模式指定匹配规则。

**语法格式：**

```sql
JSON_CONTAINS_PATH(json_doc, one_or_all, path [, path2, ...])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | JSON 文档 |
| one_or_all | `'one'` 表示任意一个路径存在即返回 1；`'all'` 表示所有路径都存在才返回 1 |
| path | 要检查的路径表达式 |

**返回类型：** INTEGER

**示例：**

```sql
SQL> SELECT JSON_CONTAINS_PATH('{"a":1,"b":2,"c":3}', 'one', '$.a', '$.d');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_CONTAINS_PATH('{"a":1,"b":2,"c":3}', 'all', '$.a', '$.d');
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+
```

---

### JSON_SEARCH

**功能描述：** 在 JSON 文档中搜索给定的字符串值，返回匹配的路径。

**语法格式：**

```sql
JSON_SEARCH(json_doc, one_or_all, search_str [, escape_char [, path]])
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| json_doc | JSON 文档 |
| one_or_all | `'one'` 返回第一个匹配路径；`'all'` 返回所有匹配路径 |
| search_str | 要搜索的字符串，支持 `%`（匹配任意字符序列）和 `_`（匹配单个字符）通配符 |
| escape_char | 可选，转义字符，默认为 `\` |
| path | 可选，限定搜索范围的路径 |

**返回类型：** JSON（路径字符串或路径数组）

**示例：**

```sql
SQL> SELECT JSON_SEARCH('{"a":"hello","b":"world","c":"hello"}', 'one', 'hello');
+-------+
| EXPR1 |
+-------+
| "$.a" |
+-------+

SQL> SELECT JSON_SEARCH('{"a":"hello","b":"world","c":"hello"}', 'all', 'hello');
+------------------+
| EXPR1            |
+------------------+
| ["$.a", "$.c"]   |
+------------------+

-- 使用通配符
SQL> SELECT JSON_SEARCH('["apple","banana","avocado"]', 'all', 'a%');
+------------------+
| EXPR1            |
+------------------+
| ["$[0]", "$[2]"] |
+------------------+
```

---

### JSON_OVERLAPS

**功能描述：** 判断两个 JSON 文档是否存在重叠部分（即是否有共同的键值对或数组元素）。

**语法格式：**

```sql
JSON_OVERLAPS(json_doc1, json_doc2)
```

**返回类型：** INTEGER，有重叠返回 1，无重叠返回 0。

**示例：**

```sql
SQL> SELECT JSON_OVERLAPS('[1,2,3]', '[3,4,5]');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_OVERLAPS('[1,2]', '[3,4]');
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+
```

---

### MEMBER OF

**功能描述：** 判断一个值是否为 JSON 数组的成员。

**语法格式：**

```sql
val MEMBER OF(json_array)
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| val | 要检查的值 |
| json_array | JSON 数组 |

**返回类型：** INTEGER，是成员返回 1，否则返回 0。

**示例：**

```sql
SQL> SELECT 3 MEMBER OF('[1,2,3,4,5]');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT 'abc' MEMBER OF('["abc","def"]');
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+
```

---

## 6. JSON 格式化与转换函数

### JSON_PRETTY

**功能描述：** 将 JSON 文档格式化为易于阅读的缩进形式。

**语法格式：**

```sql
JSON_PRETTY(json_doc)
```

**返回类型：** CHAR（格式化后的 JSON 字符串）

**示例：**

```sql
SQL> SELECT JSON_PRETTY('{"name":"张三","age":25,"address":{"city":"北京"}}');
+--------------------------------------+
| EXPR1                                |
+--------------------------------------+
| {                                    |
|   "name": "张三",                     |
|   "age": 25,                         |
|   "address": {                       |
|     "city": "北京"                    |
|   }                                  |
| }                                    |
+--------------------------------------+
```

---

### JSON_QUOTE

**功能描述：** 将字符串值用双引号包裹，使其成为合法的 JSON 字符串值。特殊字符会被转义。

**语法格式：**

```sql
JSON_QUOTE(string)
```

**返回类型：** CHAR

**示例：**

```sql
SQL> SELECT JSON_QUOTE('hello');
+---------+
| EXPR1   |
+---------+
| "hello" |
+---------+

SQL> SELECT JSON_QUOTE('say "hi"');
+----------------+
| EXPR1          |
+----------------+
| "say \"hi\""   |
+----------------+
```

---

### JSON_UNQUOTE

**功能描述：** 去掉 JSON 字符串值的外层引号。如果输入不是 JSON 字符串类型，则原样返回。

**语法格式：**

```sql
JSON_UNQUOTE(json_val)
```

**返回类型：** CHAR

**示例：**

```sql
SQL> SELECT JSON_UNQUOTE('"hello"');
+-------+
| EXPR1 |
+-------+
| hello |
+-------+

-- 配合 JSON_EXTRACT 使用
SQL> SELECT JSON_UNQUOTE(JSON_EXTRACT('{"name":"张三"}', '$.name'));
+------+
| EXPR1|
+------+
| 张三 |
+------+
```

---

## 7. JSON 合并函数

### JSON_MERGE

**功能描述：** 合并两个或多个 JSON 文档。此函数是 JSON_MERGE_PRESERVE 的别名，已不建议使用，推荐使用 JSON_MERGE_PRESERVE 或 JSON_MERGE_PATCH。

**语法格式：**

```sql
JSON_MERGE(json_doc1, json_doc2 [, json_doc3, ...])
```

**返回类型：** JSON

---

### JSON_MERGE_PRESERVE

**功能描述：** 合并两个或多个 JSON 文档，保留所有值。对于相同键，将两个值合并为数组。

**语法格式：**

```sql
JSON_MERGE_PRESERVE(json_doc1, json_doc2 [, json_doc3, ...])
```

**返回类型：** JSON

**合并规则：**
- 两个数组合并为一个数组（拼接）
- 两个对象合并为一个对象，相同键的值合并为数组
- 标量值合并为数组

**示例：**

```sql
-- 合并两个数组
SQL> SELECT JSON_MERGE_PRESERVE('[1,2]', '[3,4]');
+--------------+
| EXPR1        |
+--------------+
| [1, 2, 3, 4] |
+--------------+

-- 合并两个对象（相同键合并为数组）
SQL> SELECT JSON_MERGE_PRESERVE('{"a":1}', '{"a":2,"b":3}');
+----------------------+
| EXPR1                |
+----------------------+
| {"a": [1, 2], "b": 3}|
+----------------------+
```

---

### JSON_MERGE_PATCH

**功能描述：** 合并两个或多个 JSON 文档，使用"补丁"语义。对于相同键，后一个值覆盖前一个值（符合 RFC 7396）。

**语法格式：**

```sql
JSON_MERGE_PATCH(json_doc1, json_doc2 [, json_doc3, ...])
```

**返回类型：** JSON

**示例：**

```sql
-- 合并对象（相同键被覆盖）
SQL> SELECT JSON_MERGE_PATCH('{"a":1,"b":2}', '{"a":10,"c":3}');
+------------------------+
| EXPR1                  |
+------------------------+
| {"a": 10, "b": 2, "c": 3} |
+------------------------+

-- 使用 null 删除键
SQL> SELECT JSON_MERGE_PATCH('{"a":1,"b":2}', '{"b":null}');
+----------+
| EXPR1    |
+----------+
| {"a": 1} |
+----------+
```

---

## 8. JSON Schema 验证函数

### JSON_SCHEMA_VALID

**功能描述：** 检查 JSON 文档是否符合给定的 JSON Schema 定义。

**语法格式：**

```sql
JSON_SCHEMA_VALID(schema, json_doc)
```

**输入参数：**

| 参数 | 说明 |
|------|------|
| schema | JSON Schema 文档 |
| json_doc | 要验证的 JSON 文档 |

**返回类型：** INTEGER，符合返回 1，不符合返回 0。

**示例：**

```sql
SQL> SELECT JSON_SCHEMA_VALID(
    '{"type":"object","properties":{"name":{"type":"string"},"age":{"type":"integer"}}}',
    '{"name":"张三","age":25}'
);
+-------+
| EXPR1 |
+-------+
|     1 |
+-------+

SQL> SELECT JSON_SCHEMA_VALID(
    '{"type":"object","properties":{"age":{"type":"integer"}}}',
    '{"age":"not_a_number"}'
);
+-------+
| EXPR1 |
+-------+
|     0 |
+-------+
```

---

### JSON_SCHEMA_VALIDATION_REPORT

**功能描述：** 检查 JSON 文档是否符合 JSON Schema，并返回详细的验证报告。

**语法格式：**

```sql
JSON_SCHEMA_VALIDATION_REPORT(schema, json_doc)
```

**返回类型：** JSON（包含 valid 字段和错误详情）

**示例：**

```sql
SQL> SELECT JSON_SCHEMA_VALIDATION_REPORT(
    '{"type":"object","properties":{"age":{"type":"integer"}},"required":["age"]}',
    '{"name":"张三"}'
);
+----------------------------------------------+
| EXPR1                                        |
+----------------------------------------------+
| {"valid": false, "reason": "required property 'age' is missing", ...} |
+----------------------------------------------+
```

---

## 9. JSON 函数速查表

### 构造函数

| 函数 | 说明 | 示例 |
|------|------|------|
| JSON_ARRAY | 构造 JSON 数组 | `JSON_ARRAY(1,2,3)` → `[1,2,3]` |
| JSON_OBJECT | 构造 JSON 对象 | `JSON_OBJECT('k','v')` → `{"k":"v"}` |

### 查询与提取函数

| 函数 | 说明 | 示例 |
|------|------|------|
| JSON_EXTRACT | 按路径提取值（保留 JSON 格式） | `JSON_EXTRACT(doc,'$.a')` |
| JSON_VALUE | 按路径提取标量值（返回字符串） | `JSON_VALUE(doc,'$.a')` |
| JSON_KEYS | 获取对象的键名列表 | `JSON_KEYS('{"a":1}')` → `["a"]` |
| JSON_LENGTH | 获取元素个数 | `JSON_LENGTH('[1,2,3]')` → `3` |
| JSON_DEPTH | 获取嵌套深度 | `JSON_DEPTH('{"a":1}')` → `2` |
| JSON_TYPE | 获取值的类型 | `JSON_TYPE('"abc"')` → `STRING` |

### 修改函数

| 函数 | 说明 | 路径已存在 | 路径不存在 |
|------|------|-----------|-----------|
| JSON_SET | 设置值 | 替换 | 插入 |
| JSON_INSERT | 插入值 | 不处理 | 插入 |
| JSON_REPLACE | 替换值 | 替换 | 不处理 |
| JSON_REMOVE | 删除路径 | 删除 | 不处理 |
| JSON_ARRAY_APPEND | 向数组末尾追加值 | - | - |
| JSON_ARRAY_INSERT | 在数组指定位置插入 | - | - |

### 搜索与比较函数

| 函数 | 说明 |
|------|------|
| JSON_CONTAINS | 判断是否包含指定值 |
| JSON_CONTAINS_PATH | 判断是否存在指定路径 |
| JSON_SEARCH | 按字符串值搜索路径 |
| JSON_OVERLAPS | 判断两个 JSON 是否有重叠 |
| MEMBER OF | 判断值是否为数组成员 |

### 合并函数

| 函数 | 说明 |
|------|------|
| JSON_MERGE | JSON_MERGE_PRESERVE 的别名（已废弃） |
| JSON_MERGE_PRESERVE | 合并 JSON，保留重复键的所有值 |
| JSON_MERGE_PATCH | 合并 JSON，后值覆盖前值（RFC 7396） |

### 验证函数

| 函数 | 说明 |
|------|------|
| JSON_VALID | 检查是否为有效 JSON |
| JSON_SCHEMA_VALID | 按 JSON Schema 验证 |
| JSON_SCHEMA_VALIDATION_REPORT | 按 JSON Schema 验证并返回报告 |

### 格式化与转换函数

| 函数 | 说明 |
|------|------|
| JSON_PRETTY | 格式化 JSON 为易读形式 |
| JSON_QUOTE | 将字符串包装为 JSON 字符串 |
| JSON_UNQUOTE | 去掉 JSON 字符串的引号 |

---

## 10. 与 Oracle/MySQL/PostgreSQL 对比

### JSON 函数对照表

| 功能 | 虚谷数据库 | MySQL | PostgreSQL | Oracle |
|------|-----------|-------|------------|--------|
| 构造数组 | JSON_ARRAY | JSON_ARRAY | json_build_array | JSON_ARRAY |
| 构造对象 | JSON_OBJECT | JSON_OBJECT | json_build_object | JSON_OBJECT |
| 路径提取（JSON） | JSON_EXTRACT | JSON_EXTRACT | `->` / jsonb_extract_path | JSON_QUERY |
| 路径提取（标量） | JSON_VALUE | JSON_VALUE (8.0.21+) | `->>` / jsonb_extract_path_text | JSON_VALUE |
| 获取键名 | JSON_KEYS | JSON_KEYS | json_object_keys | 无直接等价 |
| 获取长度 | JSON_LENGTH | JSON_LENGTH | json_array_length | JSON_QUERY + LENGTH |
| 获取深度 | JSON_DEPTH | JSON_DEPTH | 无直接等价 | 无直接等价 |
| 获取类型 | JSON_TYPE | JSON_TYPE | json_typeof | 无直接等价 |
| 设置值 | JSON_SET | JSON_SET | jsonb_set | JSON_TRANSFORM |
| 插入值 | JSON_INSERT | JSON_INSERT | jsonb_insert | JSON_TRANSFORM |
| 替换值 | JSON_REPLACE | JSON_REPLACE | jsonb_set | JSON_TRANSFORM |
| 删除值 | JSON_REMOVE | JSON_REMOVE | `jsonb - key` / jsonb_strip_nulls | JSON_TRANSFORM |
| 数组追加 | JSON_ARRAY_APPEND | JSON_ARRAY_APPEND | jsonb_insert / `||` | 无直接等价 |
| 数组插入 | JSON_ARRAY_INSERT | JSON_ARRAY_INSERT | jsonb_insert | 无直接等价 |
| 包含判断 | JSON_CONTAINS | JSON_CONTAINS | `@>` 运算符 | JSON_EXISTS |
| 路径存在 | JSON_CONTAINS_PATH | JSON_CONTAINS_PATH | `?` / `?|` / `?&` 运算符 | JSON_EXISTS |
| 值搜索 | JSON_SEARCH | JSON_SEARCH | 无直接等价 | JSON_EXISTS（配合条件） |
| 重叠判断 | JSON_OVERLAPS | JSON_OVERLAPS (8.0.17+) | `?|` 运算符 | 无直接等价 |
| 成员判断 | MEMBER OF | MEMBER OF (8.0.17+) | `@>` 运算符 | 无直接等价 |
| 合并（保留） | JSON_MERGE_PRESERVE | JSON_MERGE_PRESERVE | `||` 运算符 | 无直接等价 |
| 合并（补丁） | JSON_MERGE_PATCH | JSON_MERGE_PATCH | jsonb_merge_patch (16+) | JSON_MERGEPATCH |
| 验证 | JSON_VALID | JSON_VALID | 类型转换验证 | IS JSON 条件 |
| 格式化 | JSON_PRETTY | JSON_PRETTY | jsonb_pretty | JSON_SERIALIZE(...PRETTY) |
| 引号包装 | JSON_QUOTE | JSON_QUOTE | to_json | 无直接等价 |
| 去引号 | JSON_UNQUOTE | JSON_UNQUOTE | `->>` 运算符 | 无直接等价 |
| Schema 验证 | JSON_SCHEMA_VALID | JSON_SCHEMA_VALID (8.0.17+) | 无内置支持 | 无内置支持 |
| Schema 报告 | JSON_SCHEMA_VALIDATION_REPORT | JSON_SCHEMA_VALIDATION_REPORT (8.0.17+) | 无内置支持 | 无内置支持 |

### 主要差异说明

1. **路径语法：** 虚谷数据库和 MySQL 使用 `$.key` 风格的 JSON 路径；PostgreSQL 使用 `->` / `->>` 运算符链或 `'{key1,key2}'` 数组风格路径；Oracle 使用类似 SQL/JSON 标准的路径。

2. **运算符支持：** 虚谷数据库同时支持 `->` 和 `->>` 运算符（类似 MySQL/PostgreSQL），可替代部分函数调用。

3. **合并语义：** JSON_MERGE_PRESERVE（保留所有值）和 JSON_MERGE_PATCH（后值覆盖）的区别在各数据库中一致，但函数命名不同。

4. **从 MySQL 迁移：** 虚谷数据库的 JSON 函数命名与 MySQL 高度一致，大部分可直接使用。

5. **从 Oracle 迁移：** Oracle 主要使用 JSON_VALUE、JSON_QUERY、JSON_EXISTS 和 JSON_TABLE，需改写为虚谷对应函数。

6. **从 PostgreSQL 迁移：** PostgreSQL 的 jsonb 运算符风格需改写为虚谷的函数调用风格，但 `->` 和 `->>` 运算符可直接使用。
