---
name: 虚谷数据库系统函数
name_for_command: xugudb-functions
description: |
  虚谷数据库 472 个系统函数完整参考：字符串、数学、日期时间、聚合、分析、JSON、XML、数组、加密、编码、系统信息等 21 类函数。
  包含与 Oracle/MySQL/PostgreSQL 的函数名对照和迁移指南。
---

# 虚谷数据库系统函数

虚谷数据库提供 472 个内置系统函数，覆盖 21 个功能类别，高度兼容 Oracle/MySQL/PostgreSQL 函数体系。

## 函数分类概览

| 类别 | 函数数 | 说明 |
|------|--------|------|
| 字符串函数 | 74 | 长度、截取、查找、替换、正则、拼音等 |
| 数学函数 | 59 | 基本运算、三角函数、对数指数、取整、随机数 |
| 日期时间函数 | 100 | 当前时间、构造、提取、运算、转换、截断 |
| 聚合函数 | 24 | AVG/SUM/COUNT/MAX/MIN/MEDIAN/LISTAGG/GROUP_CONCAT 等 |
| 分析函数 | 8 | ROW_NUMBER/RANK/DENSE_RANK/LAG/LEAD/FIRST_VALUE 等 |
| JSON 函数 | 28 | 构造/查询/修改/验证/搜索/合并/Schema 验证 |
| 比较函数 | 5 | COMPARE/INTERVAL/ISNULL/NUM_NONNULLS/NUM_NULLS |
| 流程控制函数 | 3 | IF/IFNULL/NULLIF |
| 加密函数 | 4 | MD5/SHA/SHA1/SHA2 |
| 编码解码函数 | 13 | BASE64/HEX/ESCAPE 编解码 |
| BIT 函数 | 13 | 按位与或非异或、位设置/测试/计数、移位 |
| UUID 函数 | 5 | NEWID/SYS_GUID/UUID/GEN_RANDOM_UUID |
| XML 函数 | 11 | XMLELEMENT/XMLFOREST/XMLTABLE/XMLQUERY 等 |
| 大对象函数 | 3 | EMPTY_BLOB/EMPTY_CLOB/TO_BLOB |
| 数组函数 | 16 | ARRAY_APPEND/CAT/FILL/POSITION/REMOVE/REPLACE 等 |
| 网络地址函数 | 2 | INET_ATON/INET_NTOA |
| 序列值函数 | 1 | CURRVAL |
| 系统信息函数 | 36 | CURRENT_USER/DATABASE/VERSION/安全标签等 |
| 系统管理函数 | 17 | 文件管理/日志/License/备份相关 |
| 几何函数 | 21 | AREA/CENTER/POINT/POLYGON/LINE 等 |
| 其他函数 | 29 | DECODE/HEX/RAISE_APPLICATION_ERROR/会话变量等 |

## 常用函数速查

### 字符串处理
```sql
SELECT CONCAT('Hello', ' ', 'XuguDB');           -- 字符串拼接
SELECT SUBSTR('XuguDB', 1, 4);                    -- 截取子串 → 'Xugu'
SELECT REPLACE('Hello World', 'World', 'XuguDB'); -- 替换
SELECT TRIM('  text  ');                           -- 去除两端空格
SELECT REGEXP_REPLACE('abc123', '[0-9]+', '');     -- 正则替换
SELECT LENGTH('虚谷数据库');                        -- 字符长度
```

### 日期时间
```sql
SELECT SYSDATE;                                    -- 当前日期时间
SELECT EXTRACT(YEAR FROM SYSDATE);                 -- 提取年份
SELECT ADD_MONTHS(SYSDATE, 3);                     -- 加3个月
SELECT DATEDIFF(DATE '2024-12-31', DATE '2024-01-01'); -- 日期差
SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS'); -- 日期格式化
```

### 聚合与分析
```sql
SELECT dept_id, AVG(salary), MEDIAN(salary) FROM emp GROUP BY dept_id;
SELECT name, salary, RANK() OVER (ORDER BY salary DESC) FROM emp;
SELECT name, salary, LAG(salary) OVER (ORDER BY hire_date) AS prev_salary FROM emp;
```

### JSON
```sql
SELECT JSON_EXTRACT('{"name":"XuguDB","ver":12}', '$.name');  -- 提取值
SELECT JSON_SET('{"a":1}', '$.b', 2);                          -- 设置值
SELECT JSON_VALID('{"key":"value"}');                           -- 验证
```

## 与 Oracle/MySQL/PG 函数兼容性

| 场景 | XuguDB | Oracle | MySQL | PostgreSQL |
|------|--------|--------|-------|------------|
| 字符串拼接 | CONCAT / `\|\|` | `\|\|` | CONCAT | `\|\|` |
| 正则 | REGEXP_LIKE/REPLACE/SUBSTR | 同名 | REGEXP 运算符 | ~ 运算符 |
| NVL | NVL / IFNULL | NVL | IFNULL | COALESCE |
| 分组拼接 | LISTAGG / GROUP_CONCAT / WM_CONCAT | LISTAGG | GROUP_CONCAT | STRING_AGG |
| 日期加月 | ADD_MONTHS | ADD_MONTHS | DATE_ADD | + INTERVAL |
| 日期格式化 | TO_CHAR | TO_CHAR | DATE_FORMAT | TO_CHAR |
| DECODE | DECODE | DECODE | 不支持(用CASE) | 不支持(用CASE) |
| JSON 操作 | JSON_EXTRACT / -> / ->> | JSON_VALUE | JSON_EXTRACT / -> / ->> | -> / ->> |
| UUID | SYS_GUID / NEWID / UUID | SYS_GUID | UUID() | gen_random_uuid() |
| 数组 | ARRAY_APPEND/CAT/POSITION 等 | VARRAY(受限) | 不支持 | 同名函数 |
| 分析函数 | ROW_NUMBER/RANK/LAG/LEAD | 同名 | 同名(8.0+) | 同名 |

## 参考文档

- [字符串函数](skills/xugudb-functions/references/string-functions.md) — 74 个函数：长度、截取、查找、替换、正则、拼音
- [数学函数](skills/xugudb-functions/references/math-functions.md) — 59 个函数：运算、三角、对数、取整、随机
- [日期时间函数](skills/xugudb-functions/references/datetime-functions.md) — 100 个函数：当前时间、提取、运算、转换
- [聚合与分析函数](skills/xugudb-functions/references/aggregate-functions.md) — 24 聚合 + 8 分析函数
- [JSON 函数](skills/xugudb-functions/references/json-functions.md) — 28 个函数：构造、查询、修改、验证
- [其他函数](skills/xugudb-functions/references/other-functions.md) — 比较、流程控制、加密、编码、BIT、UUID、XML、数组、系统信息等
