---
title: 虚谷数据库字符串函数
description: 74个字符串函数详解，包括长度计算、子串截取、查找定位、大小写转换、填充裁剪、替换、正则表达式、类型转换、拼音等功能分组，含语法、示例及与 Oracle/MySQL/PostgreSQL 对照
tags: xugudb, string-functions, concat, substr, instr, replace, regexp, trim, lpad, length, to_char, translate, upper, lower, pinyin
---

# 虚谷数据库字符串函数

虚谷数据库提供 74 个字符串处理函数，覆盖长度计算、子串截取、查找定位、大小写转换、填充裁剪、替换、正则表达式、类型转换、拼音处理等场景。

---

## 长度计算函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| LENGTH | 返回字符串的字符长度 | `LENGTH(str)` |
| LEN | 返回字符串的字符长度（同 LENGTH） | `LEN(str)` |
| LENGTHB | 返回字符串的字节长度 | `LENGTHB(str)` |
| CHAR_LENGTH | 返回字符串的字符长度 | `CHAR_LENGTH(str)` |
| CHARACTER_LENGTH | 返回字符串的字符长度（同 CHAR_LENGTH） | `CHARACTER_LENGTH(str)` |
| BLEN | 返回字符串的字节长度 | `BLEN(str)` |
| BLENGTH | 返回字符串的字节长度（同 BLEN） | `BLENGTH(str)` |
| OCTET_LENGTH | 返回字符串的字节长度 | `OCTET_LENGTH(str)` |

### LENGTH 示例

```sql
-- 字符长度
SELECT LENGTH('虚谷数据库');
-- 结果: 5

-- 字节长度（UTF-8 编码下中文占 3 字节）
SELECT LENGTHB('虚谷数据库');
-- 结果: 15（取决于字符集）

-- CHAR_LENGTH 与 LENGTH 等价
SELECT CHAR_LENGTH('XuguDB');
-- 结果: 6

-- OCTET_LENGTH 返回字节数
SELECT OCTET_LENGTH('ABC');
-- 结果: 3
```

---

## 子串截取函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| SUBSTR | 从字符串中截取子串（按字符） | `SUBSTR(str, start [, length])` |
| SUBSTRB | 从字符串中截取子串（按字节） | `SUBSTRB(str, start [, length])` |
| SUBSTRING | 截取子串（SQL 标准语法） | `SUBSTRING(str FROM start [FOR length])` |
| SUBSTRING_INDEX | 按分隔符截取子串 | `SUBSTRING_INDEX(str, delim, count)` |
| LEFT | 返回字符串左边指定字符数 | `LEFT(str, len)` |
| LEFTB | 返回字符串左边指定字节数 | `LEFTB(str, len)` |
| RIGHT | 返回字符串右边指定字符数 | `RIGHT(str, len)` |
| RIGHTB | 返回字符串右边指定字节数 | `RIGHTB(str, len)` |
| MID | 从指定位置截取子串（同 SUBSTR） | `MID(str, start, length)` |
| SPLIT_PART | 按分隔符拆分字符串并返回指定部分 | `SPLIT_PART(str, delimiter, part_number)` |
| OVERLAY | 用替换串覆盖字符串中指定位置的内容 | `OVERLAY(str PLACING new_str FROM start [FOR length])` |

### SUBSTR 示例

```sql
-- 从第 2 个字符开始截取 3 个字符
SELECT SUBSTR('虚谷数据库', 2, 3);
-- 结果: 谷数据

-- 不指定长度则截取到末尾
SELECT SUBSTR('XuguDB', 5);
-- 结果: DB

-- SUBSTRING 标准语法
SELECT SUBSTRING('XuguDB' FROM 1 FOR 4);
-- 结果: Xugu

-- LEFT / RIGHT
SELECT LEFT('虚谷数据库', 2);
-- 结果: 虚谷
SELECT RIGHT('虚谷数据库', 3);
-- 结果: 数据库

-- SUBSTRING_INDEX：取第 2 个分隔符之前的内容
SELECT SUBSTRING_INDEX('a-b-c-d', '-', 2);
-- 结果: a-b

-- SPLIT_PART：取分隔后的第 2 部分
SELECT SPLIT_PART('2024-01-15', '-', 2);
-- 结果: 01

-- OVERLAY：替换指定位置内容
SELECT OVERLAY('XuguDB' PLACING '***' FROM 3 FOR 2);
-- 结果: Xu***DB
```

---

## 查找定位函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| INSTR | 查找子串在字符串中的位置（按字符） | `INSTR(str, substr [, start [, occurrence]])` |
| INSTRB | 查找子串在字符串中的位置（按字节） | `INSTRB(str, substr [, start [, occurrence]])` |
| LOCATE | 查找子串在字符串中的位置 | `LOCATE(substr, str [, start])` |
| POSITION | 查找子串在字符串中的位置（SQL 标准） | `POSITION(substr IN str)` |
| STRPOS | 返回子串在字符串中首次出现的位置 | `STRPOS(str, substr)` |
| FIELD | 返回字符串在参数列表中的位置索引 | `FIELD(str, str1, str2, ...)` |
| FIND_IN_SET | 在逗号分隔的字符串列表中查找 | `FIND_IN_SET(str, strlist)` |
| ELT | 返回参数列表中指定索引位置的字符串 | `ELT(index, str1, str2, ...)` |
| STRCMP | 比较两个字符串的大小 | `STRCMP(str1, str2)` |
| STARTS_WITH | 判断字符串是否以指定前缀开头 | `STARTS_WITH(str, prefix)` |
| STROF | 返回子串在字符串中的位置 | `STROF(str, substr)` |

### INSTR 示例

```sql
-- 查找子串位置
SELECT INSTR('虚谷数据库管理系统', '数据库');
-- 结果: 3

-- 从第 4 个字符开始查找
SELECT INSTR('abcabc', 'bc', 4);
-- 结果: 5

-- 查找第 2 次出现的位置
SELECT INSTR('abcabcabc', 'abc', 1, 2);
-- 结果: 4

-- LOCATE 语法（注意参数顺序与 INSTR 不同）
SELECT LOCATE('DB', 'XuguDB');
-- 结果: 5

-- POSITION 标准语法
SELECT POSITION('谷' IN '虚谷数据库');
-- 结果: 2

-- FIND_IN_SET
SELECT FIND_IN_SET('b', 'a,b,c,d');
-- 结果: 2

-- STARTS_WITH
SELECT STARTS_WITH('XuguDB', 'Xugu');
-- 结果: TRUE
```

---

## 大小写转换函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| UPPER | 将字符串转为大写 | `UPPER(str)` |
| UCASE | 将字符串转为大写（同 UPPER） | `UCASE(str)` |
| LOWER | 将字符串转为小写 | `LOWER(str)` |
| LCASE | 将字符串转为小写（同 LOWER） | `LCASE(str)` |
| INITCAP | 将每个单词首字母转为大写 | `INITCAP(str)` |

```sql
SELECT UPPER('xugudb');
-- 结果: XUGUDB

SELECT LOWER('XUGUDB');
-- 结果: xugudb

SELECT INITCAP('hello world');
-- 结果: Hello World
```

---

## 填充与裁剪函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| LPAD | 在字符串左侧填充到指定长度 | `LPAD(str, len [, pad_str])` |
| RPAD | 在字符串右侧填充到指定长度 | `RPAD(str, len [, pad_str])` |
| LTRIM | 去除字符串左侧的指定字符（默认空格） | `LTRIM(str [, trim_chars])` |
| RTRIM | 去除字符串右侧的指定字符（默认空格） | `RTRIM(str [, trim_chars])` |
| BTRIM | 去除字符串两侧的指定字符（默认空格） | `BTRIM(str [, trim_chars])` |
| TRIM | 去除字符串两侧/左侧/右侧的字符 | `TRIM([LEADING\|TRAILING\|BOTH] [trim_char] FROM str)` |
| HEADING | 去除字符串开头的空格 | `HEADING(str)` |
| TAILING | 去除字符串末尾的空格 | `TAILING(str)` |
| SPACE | 返回指定数量的空格字符串 | `SPACE(n)` |

### TRIM / LPAD 示例

```sql
-- TRIM 去除两侧空格
SELECT TRIM('  XuguDB  ');
-- 结果: XuguDB

-- TRIM 去除指定字符
SELECT TRIM(BOTH '*' FROM '***XuguDB***');
-- 结果: XuguDB

-- LTRIM / RTRIM
SELECT LTRIM('  XuguDB');
-- 结果: XuguDB
SELECT RTRIM('XuguDB  ');
-- 结果: XuguDB

-- LTRIM 去除指定字符集合
SELECT LTRIM('xxyyXuguDB', 'xy');
-- 结果: XuguDB

-- LPAD 左填充
SELECT LPAD('42', 6, '0');
-- 结果: 000042

-- RPAD 右填充
SELECT RPAD('XuguDB', 10, '*');
-- 结果: XuguDB****

-- BTRIM 去除两侧指定字符
SELECT BTRIM('xxXuguDBxx', 'x');
-- 结果: XuguDB
```

---

## 替换与拼接函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| CONCAT | 拼接两个或多个字符串 | `CONCAT(str1, str2 [, ...])` |
| CONCAT_WS | 用分隔符拼接多个字符串 | `CONCAT_WS(separator, str1, str2 [, ...])` |
| REPLACE | 替换字符串中所有匹配的子串 | `REPLACE(str, old_str, new_str)` |
| TRANSLATE | 逐字符替换 | `TRANSLATE(str, from_chars, to_chars)` |
| INSERT | 在字符串指定位置插入/替换内容 | `INSERT(str, pos, len, new_str)` |
| STUFF | 在字符串指定位置删除并插入内容（同 INSERT） | `STUFF(str, pos, len, new_str)` |
| REPEAT | 将字符串重复指定次数 | `REPEAT(str, count)` |
| REPLICATE | 将字符串重复指定次数（同 REPEAT） | `REPLICATE(str, count)` |
| REVERSE | 反转字符串 | `REVERSE(str)` |
| REVERSE_STR | 反转字符串（同 REVERSE） | `REVERSE_STR(str)` |

### CONCAT / REPLACE 示例

```sql
-- CONCAT 拼接
SELECT CONCAT('虚谷', '数据库');
-- 结果: 虚谷数据库

-- 也可以使用 || 运算符拼接
SELECT '虚谷' || '数据库';
-- 结果: 虚谷数据库

-- CONCAT_WS 带分隔符拼接
SELECT CONCAT_WS('-', '2024', '01', '15');
-- 结果: 2024-01-15

-- REPLACE 替换
SELECT REPLACE('Hello World', 'World', 'XuguDB');
-- 结果: Hello XuguDB

-- TRANSLATE 逐字符替换
SELECT TRANSLATE('abc123', 'abc', 'ABC');
-- 结果: ABC123

-- INSERT / STUFF
SELECT INSERT('XuguDB', 5, 2, 'Database');
-- 结果: XuguDatabase

-- REPEAT
SELECT REPEAT('AB', 3);
-- 结果: ABABAB

-- REVERSE
SELECT REVERSE('ABCDE');
-- 结果: EDCBA
```

---

## 正则表达式函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| REGEXP_LIKE | 判断字符串是否匹配正则表达式 | `REGEXP_LIKE(str, pattern [, match_param])` |
| REGEXP_REPLACE | 用正则表达式替换匹配的子串 | `REGEXP_REPLACE(str, pattern, replace_str [, position [, occurrence [, match_param]]])` |
| REGEXP_SUBSTR | 提取匹配正则表达式的子串 | `REGEXP_SUBSTR(str, pattern [, position [, occurrence [, match_param]]])` |
| REGEXP_INSTR | 返回匹配正则表达式的子串的位置 | `REGEXP_INSTR(str, pattern [, position [, occurrence [, return_option [, match_param]]]])` |
| REGEXP_COUNT | 返回正则表达式匹配的次数 | `REGEXP_COUNT(str, pattern [, position [, match_param]])` |

`match_param` 常用值：`'i'` 不区分大小写，`'c'` 区分大小写，`'m'` 多行模式。

### REGEXP_REPLACE 示例

```sql
-- 判断是否匹配
SELECT REGEXP_LIKE('XuguDB123', '^[A-Za-z]+[0-9]+$');
-- 结果: TRUE

-- 正则替换：去除非数字字符
SELECT REGEXP_REPLACE('电话: 010-12345678', '[^0-9]', '');
-- 结果: 01012345678

-- 正则提取：提取第一个数字串
SELECT REGEXP_SUBSTR('订单号ABC-12345-XY', '[0-9]+');
-- 结果: 12345

-- 正则定位
SELECT REGEXP_INSTR('abc 123 def 456', '[0-9]+');
-- 结果: 5

-- 正则计数
SELECT REGEXP_COUNT('aaa bbb aaa ccc aaa', 'aaa');
-- 结果: 3

-- 不区分大小写匹配
SELECT REGEXP_LIKE('XuguDB', 'xugudb', 'i');
-- 结果: TRUE
```

---

## 类型转换函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| TO_CHAR | 将数值/日期转换为格式化字符串 | `TO_CHAR(value [, format])` |
| TO_NCHAR | 将值转换为 NCHAR 类型字符串 | `TO_NCHAR(value)` |
| CONVERT | 转换字符串的字符集编码 | `CONVERT(str, dest_charset [, source_charset])` |
| CHR | 返回 ASCII 码值对应的字符 | `CHR(code)` |
| ASCII | 返回字符的 ASCII 码值 | `ASCII(char)` |
| ATOF | 将字符串转换为浮点数 | `ATOF(str)` |
| ATOL | 将字符串转换为整数 | `ATOL(str)` |
| BIN | 返回整数的二进制字符串表示 | `BIN(n)` |
| TO_HEX | 返回整数的十六进制字符串表示 | `TO_HEX(n)` |
| RAWTOHEX | 将原始值转换为十六进制字符串 | `RAWTOHEX(raw)` |
| ROWIDTOCHAR | 将 ROWID 转换为字符串 | `ROWIDTOCHAR(rowid)` |
| NVL | 如果第一个参数为 NULL 则返回第二个参数 | `NVL(expr1, expr2)` |
| NVL2 | 根据第一个参数是否为 NULL 返回不同值 | `NVL2(expr1, expr2, expr3)` |

### TO_CHAR 示例

```sql
-- 数值转字符串
SELECT TO_CHAR(12345.67);
-- 结果: 12345.67

-- 带格式的数值转换
SELECT TO_CHAR(12345.67, '99,999.99');
-- 结果:  12,345.67

-- 日期转字符串
SELECT TO_CHAR(CURRENT_TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS');
-- 结果: 2024-01-15 14:30:00（示例值）

-- CHR 和 ASCII
SELECT CHR(65);
-- 结果: A
SELECT ASCII('A');
-- 结果: 65

-- NVL 空值替换
SELECT NVL(NULL, '默认值');
-- 结果: 默认值

-- NVL2 条件返回
SELECT NVL2('有值', '非空', '为空');
-- 结果: 非空
SELECT NVL2(NULL, '非空', '为空');
-- 结果: 为空

-- BIN 和 TO_HEX
SELECT BIN(255);
-- 结果: 11111111
SELECT TO_HEX(255);
-- 结果: FF
```

---

## 拼音函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| PINYIN | 返回中文字符串的拼音（全拼） | `PINYIN(str)` |
| PINYIN1 | 返回中文字符串的拼音首字母 | `PINYIN1(str)` |

```sql
-- 获取全拼
SELECT PINYIN('虚谷数据库');
-- 结果: xugushujuku

-- 获取拼音首字母
SELECT PINYIN1('虚谷数据库');
-- 结果: xgsjk

-- 可用于拼音排序
SELECT name FROM users ORDER BY PINYIN(name);

-- 可用于拼音模糊查询
SELECT name FROM users WHERE PINYIN1(name) LIKE 'z%';
```

---

## 其他函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| STROF | 返回子串在字符串中的位置 | `STROF(str, substr)` |

---

## 与 Oracle / MySQL / PostgreSQL 函数名对照表

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 字符串长度（字符） | LENGTH / LEN / CHAR_LENGTH | LENGTH | LENGTH / CHAR_LENGTH | LENGTH / CHAR_LENGTH |
| 字符串长度（字节） | LENGTHB / OCTET_LENGTH | LENGTHB | OCTET_LENGTH | OCTET_LENGTH |
| 子串截取 | SUBSTR / SUBSTRING | SUBSTR | SUBSTR / SUBSTRING | SUBSTR / SUBSTRING |
| 子串查找 | INSTR / LOCATE / POSITION | INSTR | INSTR / LOCATE / POSITION | POSITION / STRPOS |
| 字符串拼接 | CONCAT / \|\| | CONCAT / \|\| | CONCAT | CONCAT / \|\| |
| 带分隔符拼接 | CONCAT_WS | 无（需手动拼接） | CONCAT_WS | CONCAT_WS |
| 替换 | REPLACE | REPLACE | REPLACE | REPLACE |
| 逐字符替换 | TRANSLATE | TRANSLATE | 无 | TRANSLATE |
| 转大写 | UPPER / UCASE | UPPER | UPPER / UCASE | UPPER |
| 转小写 | LOWER / LCASE | LOWER | LOWER / LCASE | LOWER |
| 首字母大写 | INITCAP | INITCAP | 无 | INITCAP |
| 左填充 | LPAD | LPAD | LPAD | LPAD |
| 右填充 | RPAD | RPAD | RPAD | RPAD |
| 去除空格 | TRIM / LTRIM / RTRIM / BTRIM | TRIM / LTRIM / RTRIM | TRIM / LTRIM / RTRIM | TRIM / LTRIM / RTRIM / BTRIM |
| 正则匹配 | REGEXP_LIKE | REGEXP_LIKE | REGEXP | 无（使用 ~ 运算符） |
| 正则替换 | REGEXP_REPLACE | REGEXP_REPLACE | REGEXP_REPLACE | REGEXP_REPLACE |
| 正则提取 | REGEXP_SUBSTR | REGEXP_SUBSTR | REGEXP_SUBSTR（8.0+） | REGEXP_MATCH / SUBSTRING |
| 正则定位 | REGEXP_INSTR | REGEXP_INSTR | REGEXP_INSTR（8.0+） | 无 |
| 正则计数 | REGEXP_COUNT | REGEXP_COUNT（11g+） | 无 | 无 |
| 类型转字符串 | TO_CHAR | TO_CHAR | 无（使用 CAST/FORMAT） | TO_CHAR |
| 空值替换 | NVL / NVL2 | NVL / NVL2 | IFNULL / COALESCE | COALESCE |
| ASCII 值 | ASCII | ASCII | ASCII | ASCII |
| 反转字符串 | REVERSE | REVERSE（PL/SQL） | REVERSE | REVERSE |
| 重复字符串 | REPEAT / REPLICATE | RPAD 模拟 | REPEAT | REPEAT |
| 按分隔符拆分 | SPLIT_PART | 无（需 REGEXP_SUBSTR） | SUBSTRING_INDEX | SPLIT_PART |
| 拼音转换 | PINYIN / PINYIN1 | 无 | 无 | 无（虚谷特有） |

> **迁移提示**：
> - 从 Oracle 迁移时，INSTR、SUBSTR、REPLACE、TRANSLATE、NVL、NVL2、TO_CHAR、REGEXP 系列函数可直接使用，语法基本兼容。
> - 从 MySQL 迁移时，注意 LOCATE 的参数顺序与 INSTR 不同；MySQL 的 IFNULL 在虚谷中对应 NVL。
> - 从 PostgreSQL 迁移时，SPLIT_PART、BTRIM、POSITION 等函数可直接使用；PG 的 `~` 正则运算符需改为 REGEXP_LIKE。
> - PINYIN / PINYIN1 是虚谷数据库特有的中文拼音处理函数，其他数据库无直接对应。
