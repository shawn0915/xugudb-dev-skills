---
title: 虚谷数据库数据类型
description: 数值类型、字符类型、日期时间类型、大对象类型、BIT类型、布尔类型、JSON类型、XML类型、GUID类型、数组类型、空间类型、自定义类型详解
tags: xugudb, data-types, number, varchar, timestamp, blob, clob, json, xml, array, spatial, boolean, bit, guid
---

# 虚谷数据库数据类型

## 数值类型

### 整数类型

| 类型 | 字节长度 | 最小值 | 最大值 | 映射类型 |
|------|----------|--------|--------|----------|
| TINYINT | 1 | -128 | 127 | - |
| SMALLINT | 2 | -32768 | 32767 | SHORT |
| INTEGER | 4 | -2147483648 | 2147483647 | INT, PLS_INTEGER, BINARY_INTEGER |
| BIGINT | 8 | -9223372036854775808 | 9223372036854775807 | LONGINT |

```sql
-- 创建整数类型字段表
CREATE TABLE tab_integer_test(col1 INTEGER);
INSERT INTO tab_integer_test VALUES(666);
SELECT * FROM tab_integer_test;

-- 超出范围将报错
INSERT INTO tab_integer_test VALUES(2147483648);
-- Error: [E19230] 字段(col_no:0,col_name:COL1)值错误(数值超界)
```

### 固定精度类型

| 类型 | 字节长度 | 最小值 | 最大值 | 映射类型 |
|------|----------|--------|--------|----------|
| NUMERIC(P,S) | 2~17 | -10^38+1 | 10^38-1 | DECIMAL, NUMBER |

- `P` 为数据精度（总位数），最大支持 38 位
- `S` 为小数位数，取值范围 [0, P]
- 不指定时默认为 NUMERIC(12,0)
- 小数部分超过 S 值定义时，进行四舍五入
- 数值超过定义范围时，数据库抛出错误

```sql
-- 创建固定精度类型字段表
CREATE TABLE tab_numeric_test(id INT, col1 NUMERIC(6,2));
INSERT INTO tab_numeric_test VALUES(1, 6666.66);

-- 小数部分超出精度，四舍五入
INSERT INTO tab_numeric_test VALUES(2, 6666.664);  -- 存储为 6666.66
INSERT INTO tab_numeric_test VALUES(3, 6666.665);  -- 存储为 6666.67

-- 超出定义范围报错
INSERT INTO tab_numeric_test VALUES(5, 99999.99);
-- Error: [E19230] 字段(col_no:1,col_name:COL1)值错误(数值超界)
```

### 浮点类型

| 类型 | 字节长度 | 最小值(近似) | 最大值(近似) | 有效位数 | 映射类型 |
|------|----------|-------------|-------------|----------|----------|
| FLOAT | 4 | -3.402E+38 | 3.402E+38 | 7 | REAL, BINARY_FLOAT |
| DOUBLE | 8 | -1.797E+308 | 1.797E+308 | 16 | BINARY_DOUBLE |

浮点数支持特殊值：正无穷（INF/INFINITY）、负无穷（-INF/-INFINITY）、非数值（NaN）。

```sql
-- 创建浮点类型字段表
CREATE TABLE tab_float_test(id INT, col1 FLOAT);
INSERT INTO tab_float_test VALUES(1, 6666.66);
INSERT INTO tab_float_test VALUES(2, 'INF');
INSERT INTO tab_float_test VALUES(3, '-INF');
INSERT INTO tab_float_test VALUES(4, 'NaN');
SELECT * FROM tab_float_test;
-- 结果: 6.666660e+03, Inf, -Inf, NaN
```

---

## 字符类型

| 类型 | 长度范围 | 存储方式 | 映射类型 |
|------|----------|----------|----------|
| CHAR[(SIZE)] | 1 - 60000 | 固定长度 | NCHAR, CHARACTER |
| VARCHAR[(SIZE)] | 1 - 60000 | 可变长度 | TEXT, VARCHAR2, NVARCHAR, NVARCHAR2 |

### 定长字符类型 CHAR

- SIZE 单位为字符（非字节）
- 不指定长度时默认存储 1 个字符
- 插入时不会用空格填充到 SIZE 长度
- 尾部空格会被去除，首部和中间空格保留
- 超长处理取决于 `str_trunc_warning` 参数：OFF 时报错，ON 时截断并警告

```sql
-- 创建 CHAR 类型字段表
CREATE TABLE tab_char(col_1 CHAR, col_2 CHAR(5));
INSERT INTO tab_char VALUES('一', '测试字符');

-- 超长报错（str_trunc_warning OFF 时）
INSERT INTO tab_char VALUES('ab', 'abcd');
-- Error: [E19230] 字段(col_no:0,col_name:COL_1)值错误(数据异常/字串超长)

-- 开启截断警告
SET str_trunc_warning ON;
INSERT INTO tab_char VALUES('cd', 'cdefgh');
-- Warning: [E17090] 数据异常/字串超长（结果截断为 'c' 和 'cdefg'）
```

### 变长字符类型 VARCHAR

- SIZE 单位为字符
- 不指定长度时默认允许存储当前行记录除其他字段所占空间的剩余空间（行记录大小为 64K）
- 尾部空格会被去除
- 超长处理同 CHAR，取决于 `str_trunc_warning` 参数

```sql
-- 创建 VARCHAR 类型字段表
CREATE TABLE tab_varchar(col_1 VARCHAR, col_2 VARCHAR(5));
INSERT INTO tab_varchar VALUES('一', '测试字符');
INSERT INTO tab_varchar VALUES('ab', 'abcd');

-- 超长报错（str_trunc_warning OFF 时）
INSERT INTO tab_varchar VALUES('a', 'abcdef');
-- Error: [E19230] 字段(col_no:1,col_name:COL_2)值错误(数据异常/字串超长)
```

---

## 日期时间类型

### 日期类型 DATE

| 类型 | 格式 | 字节长度 | 范围 |
|------|------|----------|------|
| DATE | YYYY-MM-DD | 4 | '9999-12-31 BC' ~ '9999-12-31' |

- 公元前日期以 BC 表示，公元后以 AD 表示
- Oracle 兼容模式时，DATE 类型映射为 DATETIME 类型

```sql
CREATE TABLE tab_date_test(col1 DATE);
INSERT INTO tab_date_test VALUES('2025-6-20');
SELECT * FROM tab_date_test;
-- 结果: 2025-06-20
```

### 时间类型 TIME

| 类型 | 格式 | 字节长度 | 范围 |
|------|------|----------|------|
| TIME | HH24:MI:SS | 4 | '00:00:00.000' ~ '23:59:59.999' |
| TIME WITH TIME ZONE | HH24:MI:SS +UTC | 6 | '00:00:00.000' ~ '23:59:59.999'，UTC: -12:59 ~ +14:59 |

- 可选 size 值指定小数位秒精度，范围 [0,3]，默认为 0

```sql
CREATE TABLE tab_time_test(col1 TIME, col2 TIME WITH TIME ZONE);
INSERT INTO tab_time_test VALUES('15:16:25', '17:30:29+08:00');
SELECT * FROM tab_time_test;
-- 结果: 15:16:25.000 | 17:30:29 +08:00
```

### 日期时间类型 DATETIME

| 类型 | 格式 | 字节长度 | 范围 |
|------|------|----------|------|
| DATETIME | YYYY-MM-DD HH24:MI:SS | 8 | '9999-12-31 23:59:59.999 BC' ~ '9999-12-31 23:59:59.999' |
| DATETIME WITH TIME ZONE | YYYY-MM-DD HH24:MI:SS +UTC | 10 | 同上，UTC: -12:59 ~ +14:59 |

```sql
CREATE TABLE tab_datetime_test(col1 DATETIME, col2 DATETIME WITH TIME ZONE);
INSERT INTO tab_datetime_test VALUES('2025-06-20 15:16:25', '2025-06-21 17:16:25+08:00');
SELECT * FROM tab_datetime_test;
-- 结果: 2025-06-20 15:16:25.000 | 2025-06-21 17:16:25.000 +08:00
```

### 时间戳类型 TIMESTAMP

| 类型 | 格式 | 字节长度 | 范围 |
|------|------|----------|------|
| TIMESTAMP | YYYY-MM-DD HH24:MI:SS | 8 | '9999-12-31 23:59:59.999 BC' ~ '9999-12-31 23:59:59.999' |
| TIMESTAMP WITH TIME ZONE | YYYY-MM-DD HH24:MI:SS +UTC | 10 | 同上 |

- 可选 size 值指定小数位秒精度，范围 [0,6]，实际精度最大显示 3 位，默认为 3
- 当不插入数据时，数据库会默认插入当前时间（与 DATETIME 不同）

```sql
CREATE TABLE tab_timestamp_test(col1 TIMESTAMP, col2 TIMESTAMP WITH TIME ZONE);
INSERT INTO tab_timestamp_test VALUES('2025-06-20 15:16:25', '2025-06-21 17:16:25+08:00');
SELECT * FROM tab_timestamp_test;
-- 结果: 2025-06-20 15:16:25.000 | 2025-06-21 17:16:25.000 +08:00
```

### 时间间隔类型 INTERVAL

| 类型 | 字节长度 | 范围 | SIZE 范围 |
|------|----------|------|-----------|
| INTERVAL YEAR | 4 | [-999999999, 999999999] | [0, 9] |
| INTERVAL MONTH | 4 | [-999999999, 999999999] | [0, 9] |
| INTERVAL DAY | 4 | [-999999999, 999999999] | [0, 9] |
| INTERVAL HOUR | 4 | [-999999999, 999999999] | [0, 9] |
| INTERVAL MINUTE | 4 | [-999999999, 999999999] | [0, 9] |
| INTERVAL SECOND | 8 | [-999999999.999999, 999999999.999999] | 1:[0,9] 2:[0,6] |
| INTERVAL YEAR TO MONTH | 4 | year:[-99999999,99999999] month:[0,11] | [0, 8] |
| INTERVAL DAY TO HOUR | 4 | day:[-9999999,9999999] hour:[0,23] | [0, 7] |
| INTERVAL DAY TO MINUTE | 4 | day:[-999999,999999] hour:[0,23] minute:[0,59] | [0, 6] |
| INTERVAL DAY TO SECOND | 8 | day:[-999999,999999] hour:[0,23] minute:[0,59] second:[0,59] | 1:[0,6] 2:[0,6] |
| INTERVAL HOUR TO MINUTE | 4 | hour:[-9999999,9999999] minute:[0,59] | [0, 7] |
| INTERVAL HOUR TO SECOND | 8 | hour:[-9999999,9999999] minute:[0,59] second:[0,59] | 1:[0,7] 2:[0,6] |
| INTERVAL MINUTE TO SECOND | 8 | minute:[-9999999,9999999] second:[0,59] | 1:[0,7] 2:[0,6] |

- 不指定 SIZE 时，默认值取 SIZE 范围最大值

```sql
CREATE TABLE tab_interval_test(
    col1 INTERVAL YEAR,
    col2 INTERVAL MONTH,
    col3 INTERVAL DAY,
    col4 INTERVAL HOUR,
    col5 INTERVAL MINUTE,
    col6 INTERVAL SECOND,
    col7 INTERVAL YEAR TO MONTH,
    col8 INTERVAL DAY TO HOUR,
    col9 INTERVAL DAY TO MINUTE,
    col10 INTERVAL DAY TO SECOND,
    col11 INTERVAL HOUR TO MINUTE,
    col12 INTERVAL HOUR TO SECOND,
    col13 INTERVAL MINUTE TO SECOND
);
INSERT INTO tab_interval_test VALUES(
    2025, 1000, 1996, 5200, 12345, 1999.9898,
    '2025-11', '66666 23', '9999 23:56',
    '2222:12:56:24.233', '888:26',
    '9999999:59:59.999999', '9999999:59.999999'
);
```

---

## 大对象类型

| 类型 | 最大长度 | 存储方式 | 数据形式 | 映射类型 |
|------|----------|----------|----------|----------|
| BLOB | 2GB | 可变长度 | 二进制形式数据 | - |
| CLOB | 2GB | 可变长度 | 字符形式文本数据 | NCLOB, LONG |

- 未超过 512 字节存储在行内，超过 512 字节存储在行外
- 可通过 LENGTH 函数查询大对象字段长度

### BLOB 类型

BLOB（Binary Large Object）存储非结构化二进制文件，如图像、声音、视频等。

```sql
CREATE TABLE tab_blob(col_1 BLOB);
-- 通过控制台方式插入文件
INSERT INTO tab_blob VALUES(?);
<# C:\t1.png;

-- 二进制数据直接查询显示为 <BLOB>
SELECT * FROM tab_blob;
-- 通过类型转换函数转为16进制串查看
SELECT rawtohex(col_1) to_hex FROM tab_blob;
```

### CLOB 类型

CLOB（Character Large Object）存储单字节或多字节字符数据，支持固定宽度和可变宽度的字符集。

```sql
CREATE TABLE tab_clob(col_1 CLOB);
-- 通过控制台方式插入文本文件
INSERT INTO tab_clob VALUES(?);
<% C:\clob.txt;

-- 使用 to_char() 函数可将 CLOB 类型字段转为字符串显示
SELECT to_char(col_1) AS col1 FROM tab_clob;
```

---

## 二进制类型

| 类型 | 最大长度 | 存储方式 |
|------|----------|----------|
| BINARY | 64KB | 变长 |

BINARY 类型以行内存储的方式进行二进制数据存储。

```sql
CREATE TABLE tab_binary_test(col1 BINARY);
INSERT INTO tab_binary_test VALUES(1);
-- 控制台输出统一显示为 <BINARY>
SELECT * FROM tab_binary_test;

-- 通过 RAWTOHEX 函数转为16进制串查看
SELECT rawtohex(col1) to_hex FROM tab_binary_test;
-- 结果: 01
```

---

## BIT 类型

### 定长位类型

```
BIT[(SIZE)]
```

- SIZE 取值范围 [1, 60000]，不指定时默认为 1 位

### 变长位类型

```
BIT VARYING[(SIZE)]  或  VARBIT[(SIZE)]
```

- SIZE 取值范围 [1, 60000]，不指定时默认不限制，最大 60000

### 字面量语法

```
b'101'  或  B'101'
```

```sql
CREATE TABLE tab_bit_test(col1 BIT, col2 BIT VARYING, col3 VARBIT);
INSERT INTO tab_bit_test VALUES('1', '1100', '111000');
INSERT INTO tab_bit_test VALUES(b'0', B'0011', B'000111');
SELECT * FROM tab_bit_test;
-- 结果:
-- b'1' | b'1100' | b'111000'
-- b'0' | b'0011' | b'000111'
```

### 位运算符

| 运算表达式 | 结果类型 | 说明 |
|------------|----------|------|
| VARBIT & VARBIT | VARBIT | 按位与 |
| VARBIT \| VARBIT | VARBIT | 按位或 |
| VARBIT ^ VARBIT | VARBIT | 按位异或 |
| ~ VARBIT | VARBIT | 按位取反 |
| VARBIT << INTEGER | VARBIT | 左移位 |
| VARBIT >> INTEGER | VARBIT | 右移位 |

- 任意操作数为 NULL，结果为 NULL
- 操作数长度不等时，右对齐，左侧补 0 值再进行位比较

### BIT 转换规则

BIT 类型与 VARBIT、BINARY、CHAR 之间可互相转换，转换方式取决于源数据位数与目标数据所需位数的大小关系：

- **位数相等**：直接转换
- **源位数小于目标位数**：BIT/VARBIT 右侧补 0 值，BINARY 左侧补 0 值
- **源位数大于目标位数**：BIT/VARBIT 右侧截断，BINARY 报错超长

---

## 布尔类型

| 类型 | 长度 | 存储方式 | 取值 |
|------|------|----------|------|
| BOOLEAN (BOOL) | 1 | 固定长度 | TRUE \| FALSE \| UNKNOWN |

- TRUE, 'TRUE', 'T', '1' 被视为真
- FALSE, 'FALSE', 'F', '0' 被视为假
- -1 以及 -1.xxx 被视为 UNKNOWN
- 非整数插入时舍弃小数部分

```sql
CREATE TABLE tab_bool_test(col1 BOOLEAN);
INSERT INTO tab_bool_test VALUES(1);    -- TRUE
INSERT INTO tab_bool_test VALUES(0);    -- FALSE
INSERT INTO tab_bool_test VALUES(-1);   -- UNKNOWN
SELECT * FROM tab_bool_test;
-- 结果: T | F | U
```

---

## JSON 类型

| 类型 | 最大长度 | 存储方式 | Java 类型 |
|------|----------|----------|-----------|
| JSON | 2GB | 变长（大对象存储） | java.sql.String |

### JSON 数据格式

支持的基础类型：string、bool、number、null。

```
-- string（双引号包裹）
'"中文"'

-- bool
'true'  'false'

-- number
'1'  '-1'  '10.2'

-- null
'null'

-- array
'["abc", true, false, 1, 1.1, null]'

-- object
'{"key1": "value", "key2": true, "key3": false}'
```

### JSON 路径表达式（JSONPath）

| 语法 | 说明 |
|------|------|
| `$` | 代表 JSON 文本本身 |
| `$.key` | 获取对象中键对应的值 |
| `$[N]` | 获取数组中下标为 N 的元素 |
| `$[M to N]` | 获取数组中下标 M 到 N 的元素集合 |
| `$[last]` | 数组最后一个元素 |
| `$[*]` | 通配符，全量元素 |
| `$**` | 深度查找 |

### JSON 增删改查示例

```sql
CREATE TABLE t_json(c_id INT PRIMARY KEY, c_json JSON);
INSERT INTO t_json VALUES(1, '[1,2,3]')(2, '{"中文key": "中文value"}');

-- 查询
SELECT * FROM t_json;

-- 路径表达式查询
SELECT '{"test": 1}'->'$.test';          -- 结果: 1
SELECT '[1,3,5,7]'->'$[1]';              -- 结果: 3
SELECT '[1,3,5,7]'->'$[1 to 3]';         -- 结果: [3, 5, 7]
SELECT '[1,3,5,7]'->'$[last]';           -- 结果: 7

-- 更新
UPDATE t_json SET c_json='{"key": "value"}' WHERE c_id = 1;

-- 删除
DELETE FROM t_json WHERE c_id = 1;
```

### JSON 比较与排序优先级

优先级从高到低：BOOL > ARRAY > OBJECT > STRING > NUMBER > NULL

### JSON 运算符

| 运算符 | 功能 |
|--------|------|
| `->` | 获取指定路径值 |
| `->>` | 获取指定路径值并取消对 JSON 类型的引用 |

### JSON 常用函数

| 函数 | 功能 |
|------|------|
| JSON_ARRAY | 依据参数列表值创建 JSON 数组 |
| JSON_OBJECT | 构建 JSON 对象 |
| JSON_EXTRACT | 返回 JSON 文本中所有指定路径包含的值 |
| JSON_SET | 插入或更新 JSON 文本数据 |
| JSON_INSERT | 将值插入到 JSON 文本指定路径 |
| JSON_REPLACE | 将 JSON 文本指定路径值替换 |
| JSON_REMOVE | 将 JSON 文本指定路径值移除 |
| JSON_CONTAINS | 判断目标 JSON 文本是否包含候选 JSON 文本 |
| JSON_LENGTH | 返回 JSON 文本长度 |
| JSON_DEPTH | 返回 JSON 文本的最大深度 |
| JSON_TYPE | 返回 JSON 文本类型 |
| JSON_VALID | 验证给定值是否是合法 JSON 文本 |
| JSON_KEYS | 返回 JSON 文本顶层对象的所有键 |
| JSON_SEARCH | 根据指定模式搜索 JSON 文本 |
| JSON_VALUE | 根据路径表达式从 JSON 文本获取值 |
| JSON_MERGE | 合并两个或更多的 JSON 文本 |
| JSON_MERGE_PATCH | 按 RFC 7396 规范合并，不保留重复键 |
| JSON_MERGE_PRESERVE | 合并并保留重复键的成员 |
| JSON_ARRAY_APPEND | 将值追加到 JSON 数组指定路径 |
| JSON_ARRAY_INSERT | 将值插入到 JSON 数组指定路径 |
| JSON_OVERLAPS | 判断两个 JSON 文本是否有相同键值对或数组元素 |
| JSON_PRETTY | 格式化 JSON 串 |
| JSON_QUOTE | 将参数包装为 JSON 值 |
| JSON_UNQUOTE | 移除 JSON 文本外层引号 |
| JSON_ARRAYAGG | 将结果集聚合为 JSON 数组 |
| JSON_OBJECTAGG | 将两列聚合为 JSON 对象 |

---

## XML 类型

| 类型 | 最大长度 | 存储方式 |
|------|----------|----------|
| XML (XML TYPE) | 2GB | BLOB 存储 |

- XML 数据类型是数据库自定义基础类型
- 支持在表、视图中创建 XML 类型的列
- 支持创建 XML 类型的常量和变量
- 支持 XPath 路径表达式查询
- 支持 XQuery 查询语言
- 依赖库：xerces-c-3.2.2、XQilla-2.3.4（已静态编译到数据库中）

```sql
CREATE TABLE t_xml(c_id INT PRIMARY KEY, c_xml XML);
INSERT INTO t_xml VALUES(1, '<num>1</num>')(2, '<str>ab</str>');

-- 查询
SELECT * FROM t_xml;
-- 结果: <num>1</num> | <str>ab</str>

-- 更新
UPDATE t_xml SET c_xml='<num>2</num>' WHERE c_id = 1;

-- 删除
DELETE FROM t_xml WHERE c_id = 1;
```

### XPath 支持

| 路径表达式 | 说明 |
|------------|------|
| `/` | 从根节点选取 |
| `//` | 从当前节点选取子孙节点 |
| `.` | 选取当前节点 |
| `..` | 选取父节点 |
| `@` | 选取属性 |
| `*` | 匹配任何元素节点 |
| `@*` | 匹配任何属性节点 |
| `node()` | 匹配任何类型的节点 |

---

## GUID 类型

| 类型 | 长度 | 存储方式 |
|------|------|----------|
| GUID | 16 Byte | 变长 |

GUID 是一种通用唯一标识符，由 32 位数字及字符组成，通过 SYS_GUID 函数生成。兼容 Oracle 的 SYS_GUID、MySQL 的 UUID、PostgreSQL 的 GEN_RANDOM_UUID 函数。

```sql
CREATE TABLE tab_guid_test(col1 GUID);
INSERT INTO tab_guid_test VALUES(SYS_GUID());
SELECT * FROM tab_guid_test;
-- 结果: <9A86594BAA38408E93AE31ACEA177C86>
```

---

## 数组类型 (ARRAY)

ARRAY 类型允许将表的列定义为可变长多维数组，最大支持 2GB 输入文本，最大支持六维数组。

### 支持的基础类型

integer, tinyint, smallint, bigint, float, double, numeric, date, datetime, datetime with time zone, time, time with time zone, char, varchar, BLOB, CLOB, interval (各种), point, box, circle, polygon, path, line, lseg, xml

### 字段声明

```sql
-- 一维数组
column_name type_name[]
-- 多维数组
column_name type_name[][]    -- 二维
column_name type_name[][][]  -- 三维

-- 构造函数声明（仅用于一维）
column_name type_name ARRAY
column_name type_name ARRAY[3]
```

### 数组的增删改查

```sql
-- 创建一维数组表
CREATE TABLE test_arr(a INT[]);
-- 字符串方式插入
INSERT INTO test_arr VALUES('{1, 2, 3}');
-- 构造函数方式插入
INSERT INTO test_arr VALUES(ARRAY[1, 2, 3]);

-- 创建多维数组表
CREATE TABLE array_types(
    name VARCHAR,
    int_arr INTEGER[],
    var_arr_2 VARCHAR[][]
);
INSERT INTO array_types VALUES('int', '{1, 2, 3}', '{{1, 2, 3}, {4, 5, 6}, {7, 8, 9}}');

-- 查询
SELECT a FROM test_arr;              -- 结果: {1,2,3}
SELECT a[1] FROM test_arr;           -- 结果: 1（下标访问）
SELECT a[1:2] FROM test_arr;         -- 结果: {1,2}（切片查询）

-- 更新
UPDATE test_arr SET a[1] = 4;        -- 更新指定元素

-- 删除
DELETE test_arr WHERE a = ARRAY[4,2,3];       -- 全等删除
DELETE test_arr WHERE a[1] = 4;               -- 下标条件删除
DELETE test_arr WHERE a[1:2] = ARRAY[1, 2];   -- 切片条件删除
```

---

## 简单空间类型

| 类型 | 字节宽度 | 定义 | 数据格式 |
|------|----------|------|----------|
| POINT | 16 | 平面上的点 | (x, y) |
| LINE | 32 | 无限长的线 | {A, B, C} |
| LSEG | 32 | 有限线段 | [(x1,y1),(x2,y2)] |
| BOX | 32 | 矩形框 | ((x1,y1),(x2,y2)) |
| PATH | 16+16n | 路径（开放或封闭） | 开放:[(x1,y1),...] 封闭:((x1,y1),...) |
| POLYGON | 40+16n | 多边形 | ((x1,y1),...,(xn,yn)) |
| CIRCLE | 24 | 圆 | <(x,y),r> |

```sql
CREATE TABLE tab_geom(
    c_id INT PRIMARY KEY,
    c_pt POINT,
    c_le LINE,
    c_lg LSEG,
    c_bx BOX,
    c_ce CIRCLE,
    c_ph PATH,
    c_pn POLYGON
);
INSERT INTO tab_geom VALUES(
    1, '(1,1)', '(1,1),(2,2)', '(1,1),(3,3)',
    '(1,1),(4,4)', '<(1,2),3>',
    '(1,1),(2,3),(3,1)', '(1,1),(2,3),(3,1),(1,1)'
);
SELECT * FROM tab_geom;

-- 更新点数据
UPDATE tab_geom SET c_pt='(0,1)' WHERE c_id = 1;
```

---

## 自定义类型 (UDT)

XuguDB 自定义类型分为三类：

| 类型 | 说明 | 成员/元素 | 父类 | 构造函数 | 指定容量 | 方法 |
|------|------|-----------|------|----------|----------|------|
| OBJECT | 结构类型 | 有 | 有 | 有 | 否 | 有 |
| VARRAY | 数组类型 | 有 | 无 | 有 | 是 | 无 |
| TABLE | 嵌套表类型 | 有 | 无 | 有 | 否 | 无 |

### 结构类型 (OBJECT)

支持父类继承、构造函数、静态函数、普通成员函数。

```sql
-- 创建 OBJECT 类型
CREATE OR REPLACE TYPE udt_obj_type AS OBJECT(
    n NUMERIC,
    class VARCHAR2,
    type VARCHAR,
    dt DATE
);

-- 使用构造函数
SELECT udt_obj_type(12, '重庆', '成都', '2025-6-14 00:00:00') AS udt_object_type;
-- 结果: [12,重庆,成都,2025-06-14 00:00:00]

-- 继承
CREATE OR REPLACE TYPE under_udt_obj_type UNDER udt_obj_type(region VARCHAR);

-- 在表中使用 UDT
CREATE TABLE obj_tab(id INT, state VARCHAR2, type VARCHAR, udt_obj UDT_OBJ_TYPE);
INSERT INTO obj_tab(id, state, type, udt_obj) VALUES(
    1, '构造函数插入', 'udt_obj_type',
    udt_obj_type(1.0, '一层', 'udt_obj_type', '2025-06-14 00:00:00')
);

-- 自定义构造函数
CREATE OR REPLACE TYPE udt_obj_type_2 AS OBJECT (
    street VARCHAR,
    city VARCHAR,
    CONSTRUCTOR PROCEDURE udt_obj_type_2(street_1 VARCHAR, city_2 VARCHAR)
);
/
CREATE OR REPLACE TYPE BODY udt_obj_type_2 IS
    CONSTRUCTOR PROCEDURE udt_obj_type_2(street_1 VARCHAR, city_2 VARCHAR) AS
    BEGIN
        street := street_1;
        city := city_2;
    END;
END;
/
```

### 数组类型 (VARRAY)

VARRAY 可变数组类型，成员最多包含 65535 个，创建时必须指定容量。

```sql
-- 创建 VARRAY 类型
CREATE OR REPLACE TYPE var_test IS VARRAY(3) OF VARCHAR;
SELECT var_test('四川', '成都', '金石路');
-- 结果: [四川,成都,金石路]

-- 在表中使用
CREATE TABLE udt_varray_tab(id INT, udt_varray_type VAR_TEST);
INSERT INTO udt_varray_tab VALUES(1, VAR_TEST('重庆', '酉阳', '桃花源'));
```

### 嵌套表类型 (TABLE)

嵌套表表示无序的相同类型数组元素的集合。不同于数组的是，删除中间位置的数据后，数据变成离散的。

```sql
-- 创建 TABLE 类型
CREATE OR REPLACE TYPE udt_tab_type IS TABLE OF BIGINT;
SELECT udt_tab_type(1, 2, 3, 4, 5);
-- 结果: [1,2,3,4,5]

-- 在表中使用
CREATE TABLE tab_type(id INT, type_tab UDT_TAB_TYPE);
INSERT INTO tab_type VALUES(1, UDT_TAB_TYPE(1, 2, 3, 4, 5));

-- 通过索引下标查询
SELECT udt_type(1), udt_type(2) FROM udt_tab;
```

### 删除类型

```sql
DROP TYPE person;
-- 存在依赖对象时不允许强制删除，需先修改或删除依赖对象
-- 支持 CASCADE、RESTRICT、CASCADE CONSTRAINTS 选项
```

---

## 与 Oracle/MySQL/PostgreSQL 数据类型对比

### 数值类型对比

| XuguDB | Oracle | MySQL | PostgreSQL | 说明 |
|--------|--------|-------|------------|------|
| TINYINT | - | TINYINT | - | 1 字节整数，Oracle/PG 无直接对应类型 |
| SMALLINT | - | SMALLINT | SMALLINT | 2 字节整数 |
| INTEGER (INT) | NUMBER(10) | INT | INTEGER | 4 字节整数 |
| BIGINT | NUMBER(19) | BIGINT | BIGINT | 8 字节整数 |
| NUMERIC(P,S) | NUMBER(P,S) | DECIMAL(P,S) | NUMERIC(P,S) | 固定精度，XuguDB 最大 38 位 |
| FLOAT | BINARY_FLOAT | FLOAT | REAL | 4 字节单精度浮点 |
| DOUBLE | BINARY_DOUBLE | DOUBLE | DOUBLE PRECISION | 8 字节双精度浮点 |

### 字符类型对比

| XuguDB | Oracle | MySQL | PostgreSQL | 说明 |
|--------|--------|-------|------------|------|
| CHAR(N) | CHAR(N) | CHAR(N) | CHAR(N) | XuguDB 最大 60000 字符，不补空格；Oracle 最大 2000 字节，补空格 |
| VARCHAR(N) | VARCHAR2(N) | VARCHAR(N) | VARCHAR(N) | XuguDB 最大 60000 字符；Oracle VARCHAR2 最大 4000/32767 字节 |
| TEXT (VARCHAR 映射) | CLOB | TEXT | TEXT | XuguDB 中 TEXT 是 VARCHAR 的映射类型 |

### 日期时间类型对比

| XuguDB | Oracle | MySQL | PostgreSQL | 说明 |
|--------|--------|-------|------------|------|
| DATE | DATE (含时间) | DATE | DATE | Oracle 的 DATE 包含时间；XuguDB 在 Oracle 兼容模式下 DATE 映射为 DATETIME |
| TIME | - | TIME | TIME | Oracle 无独立 TIME 类型 |
| DATETIME | DATE / TIMESTAMP | DATETIME | TIMESTAMP | XuguDB DATETIME 精确到毫秒 |
| TIMESTAMP | TIMESTAMP | TIMESTAMP | TIMESTAMP | XuguDB TIMESTAMP 未插入数据时默认当前时间 |
| DATETIME WITH TIME ZONE | TIMESTAMP WITH TIME ZONE | - | TIMESTAMP WITH TIME ZONE | MySQL 无带时区的日期时间类型 |
| INTERVAL | INTERVAL YEAR TO MONTH / INTERVAL DAY TO SECOND | - | INTERVAL | XuguDB 支持 13 种间隔类型；MySQL 不支持 INTERVAL 类型存储 |

### 大对象与二进制类型对比

| XuguDB | Oracle | MySQL | PostgreSQL | 说明 |
|--------|--------|-------|------------|------|
| BLOB | BLOB | LONGBLOB | BYTEA | XuguDB 最大 2GB |
| CLOB | CLOB | LONGTEXT | TEXT | XuguDB 最大 2GB；NCLOB/LONG 为映射类型 |
| BINARY | RAW | BINARY/VARBINARY | BYTEA | XuguDB 最大 64KB，行内存储 |

### 特殊类型对比

| XuguDB | Oracle | MySQL | PostgreSQL | 说明 |
|--------|--------|-------|------------|------|
| BOOLEAN | - | BOOLEAN (TINYINT(1)) | BOOLEAN | XuguDB 支持 TRUE/FALSE/UNKNOWN 三值；Oracle 无 BOOLEAN 列类型 |
| BIT / VARBIT | - | BIT | BIT / BIT VARYING | XuguDB 最大 60000 位；MySQL BIT 最大 64 位 |
| JSON | - (需用 CLOB) | JSON | JSON / JSONB | XuguDB 支持 JSONPath 和丰富的 JSON 函数 |
| XML | XMLTYPE | - (需用 TEXT) | XML | XuguDB 支持 XPath 和 XQuery |
| GUID | RAW(16) + SYS_GUID() | - (UUID 函数) | UUID | XuguDB 兼容三种数据库的 UUID 生成函数 |
| ARRAY | - (VARRAY/TABLE) | - | ARRAY | XuguDB 支持最大六维数组，语法兼容 PostgreSQL |
| POINT/LINE/BOX/CIRCLE/PATH/POLYGON/LSEG | SDO_GEOMETRY | - (需 Spatial 扩展) | 原生支持 | XuguDB 简单空间类型语法兼容 PostgreSQL |
| OBJECT (UDT) | OBJECT TYPE | - | COMPOSITE TYPE | XuguDB 支持继承、成员函数，语法兼容 Oracle |
| VARRAY (UDT) | VARRAY | - | - | XuguDB 语法兼容 Oracle |
| TABLE (UDT 嵌套表) | NESTED TABLE | - | - | XuguDB 语法兼容 Oracle |

### 迁移注意事项

1. **从 Oracle 迁移**：XuguDB 的 NUMBER 是 NUMERIC 的映射类型，VARCHAR2 是 VARCHAR 的映射类型，NCLOB/LONG 是 CLOB 的映射类型。Oracle 兼容模式下 DATE 自动映射为 DATETIME。OBJECT/VARRAY/NESTED TABLE 等 UDT 语法基本兼容。
2. **从 MySQL 迁移**：XuguDB 的 TEXT 是 VARCHAR 的映射类型。MySQL 的 TINYINT(1) 布尔语义需改为 BOOLEAN。MySQL 无 INTERVAL 存储类型。BIT 类型位数上限差异较大（XuguDB 60000 vs MySQL 64）。
3. **从 PostgreSQL 迁移**：ARRAY 类型语法高度兼容。简单空间类型语法兼容。BIT VARYING/VARBIT 语法兼容。PostgreSQL 的 JSONB 在 XuguDB 中统一为 JSON 类型。
