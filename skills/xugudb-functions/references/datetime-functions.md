---
title: 虚谷数据库日期时间函数
description: 虚谷数据库（XuguDB）日期时间函数完整参考，包括获取当前时间、日期构造、日期提取、日期运算、日期转换、日期截断等 100 个函数，与 Oracle/MySQL/PostgreSQL 对比
tags: [xugudb, datetime, date, time, timestamp, interval, 日期, 时间, 函数]
---

# 虚谷数据库日期时间函数

虚谷数据库提供了 100 个日期时间相关函数，涵盖获取当前时间、日期构造、日期提取、日期运算、日期转换、日期截断等功能。这些函数同时兼容 Oracle、MySQL 和 PostgreSQL 的常用日期时间函数语法，便于数据库迁移。

---

## 目录

1. [获取当前时间函数](#1-获取当前时间函数)
2. [日期构造函数](#2-日期构造函数)
3. [日期提取函数](#3-日期提取函数)
4. [日期运算函数](#4-日期运算函数)
5. [日期转换函数](#5-日期转换函数)
6. [日期截断与舍入函数](#6-日期截断与舍入函数)
7. [其他日期时间函数](#7-其他日期时间函数)
8. [与 Oracle/MySQL/PostgreSQL 对比](#8-与-oraclemysqlpostgresql-对比)

---

## 1. 获取当前时间函数

获取系统当前的日期、时间或时间戳。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `CURRENT_DATE` | DATE | 返回当前日期（SQL 标准） |
| `CURRENT_TIME` | TIME | 返回当前时间（SQL 标准） |
| `CURRENT_TIMESTAMP` | DATETIME/TIMESTAMP | 返回当前日期时间（SQL 标准） |
| `CURDATE()` | DATE | 返回当前日期（MySQL 兼容） |
| `CURTIME()` | TIME | 返回当前时间（MySQL 兼容） |
| `NOW()` | DATETIME | 返回当前日期时间（MySQL 兼容） |
| `SYSDATE` | DATETIME | 返回系统当前日期时间（Oracle 兼容），每次调用实时获取 |
| `SYSTIME` | TIME | 返回系统当前时间 |
| `SYSTIMESTAMP` | TIMESTAMP WITH TIME ZONE | 返回带时区的系统时间戳（Oracle 兼容） |
| `CLOCK_TIMESTAMP()` | TIMESTAMP | 返回实际的当前时间（每次调用都重新获取，PostgreSQL 兼容） |
| `STATEMENT_TIMESTAMP()` | TIMESTAMP | 返回当前语句开始时的时间戳（PostgreSQL 兼容） |
| `LOCALTIME` | TIME | 返回本地当前时间（无时区，SQL 标准） |
| `LOCALTIMESTAMP` | TIMESTAMP | 返回本地当前时间戳（无时区，SQL 标准） |
| `UTC_DATE()` | DATE | 返回 UTC 日期（MySQL 兼容） |
| `UTC_TIME()` | TIME | 返回 UTC 时间（MySQL 兼容） |
| `UTC_TIMESTAMP()` | DATETIME | 返回 UTC 日期时间（MySQL 兼容） |

### SQL 示例

```sql
-- 获取当前日期
SELECT CURRENT_DATE;
-- 结果: 2024-07-12

-- 获取当前日期时间
SELECT NOW();
-- 结果: 2024-07-12 16:30:45

-- SYSDATE 在同一语句中每次调用返回实时时间
SELECT SYSDATE, SYSDATE FROM DUAL;
-- 两个值可能不同（毫秒级差异）

-- CURRENT_TIMESTAMP 在同一语句中返回相同值
SELECT CURRENT_TIMESTAMP, CURRENT_TIMESTAMP FROM DUAL;
-- 两个值相同（语句级快照）

-- CLOCK_TIMESTAMP 与 STATEMENT_TIMESTAMP 对比
SELECT CLOCK_TIMESTAMP(), STATEMENT_TIMESTAMP();
-- CLOCK_TIMESTAMP 实时更新，STATEMENT_TIMESTAMP 语句开始时固定

-- 获取 UTC 时间
SELECT UTC_DATE(), UTC_TIME(), UTC_TIMESTAMP();

-- MySQL 兼容写法
SELECT CURDATE(), CURTIME(), NOW();

-- Oracle 兼容写法
SELECT SYSDATE, SYSTIMESTAMP FROM DUAL;
```

> **注意**：`SYSDATE` 和 `CLOCK_TIMESTAMP()` 每次调用都返回实时时间，而 `CURRENT_TIMESTAMP`、`NOW()` 和 `STATEMENT_TIMESTAMP()` 在同一语句中返回相同的时间戳（语句级快照）。这一行为与 Oracle 和 PostgreSQL 一致。

---

## 2. 日期构造函数

根据给定的年、月、日、时、分、秒等分量构造日期时间值。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `MAKEDATE(year, dayofyear)` | DATE | 根据年份和年内第 N 天构造日期（MySQL 兼容） |
| `MAKETIME(hour, minute, second)` | TIME | 根据时分秒构造时间（MySQL 兼容） |
| `MAKE_DATE(year, month, day)` | DATE | 根据年月日构造日期（PostgreSQL 兼容） |
| `MAKE_TIME(hour, minute, second)` | TIME | 根据时分秒构造时间（PostgreSQL 兼容） |
| `MAKE_TIMESTAMP(year, month, day, hour, min, sec)` | TIMESTAMP | 根据各分量构造时间戳（PostgreSQL 兼容） |
| `MAKE_TIMESTAMPTZ(year, month, day, hour, min, sec [, tz])` | TIMESTAMP WITH TIME ZONE | 根据各分量构造带时区时间戳（PostgreSQL 兼容） |

### SQL 示例

```sql
-- MySQL 风格：根据年份和天数构造日期
SELECT MAKEDATE(2024, 100);
-- 结果: 2024-04-09（第 100 天）

-- MySQL 风格：构造时间
SELECT MAKETIME(14, 30, 45);
-- 结果: 14:30:45

-- PostgreSQL 风格：根据年月日构造日期
SELECT MAKE_DATE(2024, 7, 15);
-- 结果: 2024-07-15

-- PostgreSQL 风格：构造时间
SELECT MAKE_TIME(10, 30, 0);
-- 结果: 10:30:00

-- PostgreSQL 风格：构造时间戳
SELECT MAKE_TIMESTAMP(2024, 7, 15, 10, 30, 0);
-- 结果: 2024-07-15 10:30:00

-- 构造带时区的时间戳
SELECT MAKE_TIMESTAMPTZ(2024, 7, 15, 10, 30, 0, 'Asia/Shanghai');
-- 结果: 2024-07-15 10:30:00+08
```

---

## 3. 日期提取函数

从日期时间值中提取年、月、日、时、分、秒等分量。

### 3.1 通用提取函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `EXTRACT(field FROM source)` | NUMERIC | SQL 标准提取函数，field 可为 YEAR/MONTH/DAY/HOUR/MINUTE/SECOND 等 |
| `DATE_PART(field, source)` | NUMERIC | PostgreSQL 风格提取函数，功能同 EXTRACT |

### 3.2 单字段提取函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `YEAR(date)` | INTEGER | 提取年份 |
| `MONTH(date)` | INTEGER | 提取月份（1-12） |
| `DAY(date)` | INTEGER | 提取日（1-31） |
| `HOUR(datetime)` | INTEGER | 提取小时（0-23） |
| `MINUTE(datetime)` | INTEGER | 提取分钟（0-59） |
| `SECOND(datetime)` | NUMERIC | 提取秒（0-59，可含小数） |
| `QUARTER(date)` | INTEGER | 提取季度（1-4） |
| `WEEK(date)` | INTEGER | 提取周数 |
| `MICROSECOND(datetime)` | INTEGER | 提取微秒部分 |

### 3.3 日期名称与编号函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `DAYNAME(date)` | VARCHAR | 返回星期名称（如 Monday） |
| `DAYOFWEEK(date)` | INTEGER | 返回星期编号（1=Sunday, 7=Saturday，MySQL 兼容） |
| `DAYOFYEAR(date)` | INTEGER | 返回年内第几天（1-366） |
| `MONTHNAME(date)` | VARCHAR | 返回月份名称（如 January） |
| `WEEKDAY(date)` | INTEGER | 返回星期编号（0=Monday, 6=Sunday，MySQL 兼容） |
| `WEEKOFYEAR(date)` | INTEGER | 返回年内第几周（MySQL 兼容） |
| `YEARWEEK(date [, mode])` | INTEGER | 返回年份和周数组合值（如 202428，MySQL 兼容） |

### 3.4 GET 系列提取函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `GETYEAR(datetime)` | INTEGER | 提取年份 |
| `GETMONTH(datetime)` | INTEGER | 提取月份 |
| `GETDAY(datetime)` | INTEGER | 提取日 |
| `GETHOUR(datetime)` | INTEGER | 提取小时 |
| `GETMINUTE(datetime)` | INTEGER | 提取分钟 |
| `GETSECOND(datetime)` | INTEGER | 提取秒 |
| `GETDATE()` | DATE | 返回当前日期（SQL Server 兼容） |
| `GETTIME()` | TIME | 返回当前时间 |

### 3.5 EXTRACT 系列函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `EXTRACT_YEAR(datetime)` | INTEGER | 等价于 EXTRACT(YEAR FROM datetime) |
| `EXTRACT_MONTH(datetime)` | INTEGER | 等价于 EXTRACT(MONTH FROM datetime) |
| `EXTRACT_DAY(datetime)` | INTEGER | 等价于 EXTRACT(DAY FROM datetime) |
| `EXTRACT_HOUR(datetime)` | INTEGER | 等价于 EXTRACT(HOUR FROM datetime) |
| `EXTRACT_MINUTE(datetime)` | INTEGER | 等价于 EXTRACT(MINUTE FROM datetime) |
| `EXTRACT_SECOND(datetime)` | NUMERIC | 等价于 EXTRACT(SECOND FROM datetime) |

### SQL 示例

```sql
-- EXTRACT 标准语法
SELECT EXTRACT(YEAR FROM TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 2024

SELECT EXTRACT(MONTH FROM DATE '2024-07-15');
-- 结果: 7

SELECT EXTRACT(HOUR FROM TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 10

-- DATE_PART（PostgreSQL 风格）
SELECT DATE_PART('year', TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 2024

-- 单字段提取
SELECT YEAR(DATE '2024-07-15'), MONTH(DATE '2024-07-15'), DAY(DATE '2024-07-15');
-- 结果: 2024, 7, 15

-- 季度和周
SELECT QUARTER(DATE '2024-07-15');
-- 结果: 3

SELECT WEEK(DATE '2024-07-15');
-- 结果: 28

-- 星期名称和编号
SELECT DAYNAME(DATE '2024-07-15');
-- 结果: Monday

SELECT DAYOFWEEK(DATE '2024-07-15');
-- 结果: 2（Monday，Sunday=1）

SELECT WEEKDAY(DATE '2024-07-15');
-- 结果: 0（Monday，Monday=0）

-- 年内天数
SELECT DAYOFYEAR(DATE '2024-07-15');
-- 结果: 197

-- YEARWEEK（MySQL 兼容）
SELECT YEARWEEK(DATE '2024-07-15');
-- 结果: 202429

-- GET 系列
SELECT GETYEAR(NOW()), GETMONTH(NOW()), GETDAY(NOW());

-- EXTRACT 系列
SELECT EXTRACT_YEAR(NOW()), EXTRACT_MONTH(NOW());
```

---

## 4. 日期运算函数

对日期时间值进行加减运算、计算时间差等。

### 4.1 日期加法函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `ADDDATE(date, INTERVAL expr unit)` | DATETIME | 日期加上时间间隔（MySQL 兼容） |
| `ADDDATE(date, days)` | DATE | 日期加上天数（MySQL 兼容） |
| `ADDTIME(datetime, time)` | DATETIME | 日期时间加上时间值（MySQL 兼容） |
| `ADD_MONTHS(date, n)` | DATE | 日期加上 N 个月（Oracle 兼容） |
| `DATE_ADD(date, INTERVAL expr unit)` | DATETIME | 日期加上时间间隔（MySQL 兼容，同 ADDDATE） |

### 4.2 日期减法函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `DATE_SUB(date, INTERVAL expr unit)` | DATETIME | 日期减去时间间隔（MySQL 兼容） |
| `SUBDATE(date, INTERVAL expr unit)` | DATETIME | 日期减去时间间隔（MySQL 兼容，同 DATE_SUB） |
| `SUBDATE(date, days)` | DATE | 日期减去天数（MySQL 兼容） |
| `SUBTIME(datetime, time)` | DATETIME | 日期时间减去时间值（MySQL 兼容） |

### 4.3 日期差值函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `DATEDIFF(date1, date2)` | INTEGER | 返回两个日期之间的天数差（MySQL 兼容） |
| `TIMEDIFF(time1, time2)` | TIME | 返回两个时间之间的差值（MySQL 兼容） |
| `TIMESTAMPADD(unit, n, datetime)` | DATETIME | 在时间戳上加指定单位的数值（JDBC 兼容） |
| `TIMESTAMPDIFF(unit, datetime1, datetime2)` | BIGINT | 返回两个时间戳之间指定单位的差值（MySQL 兼容） |
| `MONTHS_BETWEEN(date1, date2)` | NUMERIC | 返回两个日期之间的月数差（Oracle 兼容） |

### 4.4 周期运算函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `PERIOD_ADD(period, n)` | INTEGER | 在年月周期值上加 N 个月（MySQL 兼容，period 格式 YYYYMM） |
| `PERIOD_DIFF(period1, period2)` | INTEGER | 返回两个年月周期值之间的月数差（MySQL 兼容） |

### 4.5 重叠判断

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `(start1, end1) OVERLAPS (start2, end2)` | BOOLEAN | 判断两个时间区间是否重叠（SQL 标准） |

### SQL 示例

```sql
-- 日期加减（MySQL 兼容）
SELECT ADDDATE('2024-07-15', INTERVAL 10 DAY);
-- 结果: 2024-07-25

SELECT ADDDATE('2024-07-15', 10);
-- 结果: 2024-07-25

SELECT DATE_ADD('2024-07-15', INTERVAL 3 MONTH);
-- 结果: 2024-10-15

SELECT DATE_SUB('2024-07-15', INTERVAL 1 YEAR);
-- 结果: 2023-07-15

-- 加减时间
SELECT ADDTIME('2024-07-15 10:00:00', '02:30:00');
-- 结果: 2024-07-15 12:30:00

SELECT SUBTIME('2024-07-15 10:00:00', '02:30:00');
-- 结果: 2024-07-15 07:30:00

-- Oracle 兼容：加月
SELECT ADD_MONTHS(DATE '2024-01-31', 1);
-- 结果: 2024-02-29（自动处理月末）

-- 日期差值
SELECT DATEDIFF('2024-07-15', '2024-01-01');
-- 结果: 196

SELECT TIMEDIFF('12:00:00', '08:30:00');
-- 结果: 03:30:00

-- TIMESTAMPDIFF 按指定单位计算差值
SELECT TIMESTAMPDIFF(MONTH, '2024-01-01', '2024-07-15');
-- 结果: 6

SELECT TIMESTAMPDIFF(HOUR, '2024-07-15 08:00:00', '2024-07-15 17:30:00');
-- 结果: 9

-- MONTHS_BETWEEN（Oracle 兼容）
SELECT MONTHS_BETWEEN(DATE '2024-07-15', DATE '2024-01-15');
-- 结果: 6

-- 周期运算（MySQL 兼容）
SELECT PERIOD_ADD(202407, 3);
-- 结果: 202410

SELECT PERIOD_DIFF(202407, 202401);
-- 结果: 6

-- 时间区间重叠判断
SELECT (DATE '2024-01-01', DATE '2024-06-30') OVERLAPS (DATE '2024-03-01', DATE '2024-09-30');
-- 结果: TRUE
```

---

## 5. 日期转换函数

在日期时间类型与字符串、数值之间进行格式化转换。

### 5.1 字符串转日期时间

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `TO_DATE(string, format)` | DATE | 按格式将字符串转为日期（Oracle 兼容） |
| `TO_TIMESTAMP(string, format)` | TIMESTAMP | 按格式将字符串转为时间戳（Oracle/PG 兼容） |
| `TO_TIMESTAMPZ(string, format)` | TIMESTAMP WITH TIME ZONE | 按格式将字符串转为带时区时间戳 |
| `TO_XDATE(string, format)` | DATETIME | 按格式将字符串转为虚谷 DATETIME 类型 |

### 5.2 日期时间转字符串

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `TO_CHAR(datetime, format)` | VARCHAR | 按格式将日期时间转为字符串（Oracle 兼容） |
| `TO_NCHAR(datetime, format)` | NVARCHAR | 按格式将日期时间转为国际化字符串 |
| `STROF(datetime, format)` | VARCHAR | 按格式将日期时间转为字符串 |
| `DATE_FORMAT(date, format)` | VARCHAR | MySQL 风格日期格式化 |
| `TIME_FORMAT(time, format)` | VARCHAR | MySQL 风格时间格式化 |
| `GET_FORMAT(type, locale)` | VARCHAR | 获取指定地区的日期时间格式字符串（MySQL 兼容） |

### 5.3 UNIX 时间戳转换

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `FROM_UNIXTIME(unix_timestamp [, format])` | DATETIME/VARCHAR | Unix 时间戳转日期时间（MySQL 兼容） |
| `UNIX_TIMESTAMP([datetime])` | BIGINT | 日期时间转 Unix 时间戳（MySQL 兼容） |

### 5.4 天数/秒数转换

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `FROM_DAYS(n)` | DATE | 从天数（距公元 0 年）转为日期（MySQL 兼容） |
| `TO_DAYS(date)` | INTEGER | 日期转为天数（距公元 0 年，MySQL 兼容） |
| `TO_SECONDS(datetime)` | BIGINT | 日期时间转为秒数（距公元 0 年，MySQL 兼容） |
| `SEC_TO_TIME(seconds)` | TIME | 秒数转时间值（MySQL 兼容） |
| `TIME_TO_SEC(time)` | INTEGER | 时间值转秒数（MySQL 兼容） |

### SQL 示例

```sql
-- TO_DATE（Oracle 兼容）
SELECT TO_DATE('2024-07-15', 'YYYY-MM-DD');
-- 结果: 2024-07-15

SELECT TO_DATE('15/07/2024', 'DD/MM/YYYY');
-- 结果: 2024-07-15

-- TO_TIMESTAMP
SELECT TO_TIMESTAMP('2024-07-15 10:30:45', 'YYYY-MM-DD HH24:MI:SS');
-- 结果: 2024-07-15 10:30:45

-- TO_CHAR（Oracle 兼容格式化输出）
SELECT TO_CHAR(SYSDATE, 'YYYY"年"MM"月"DD"日"');
-- 结果: 2024年07月15日

SELECT TO_CHAR(SYSDATE, 'YYYY-MM-DD HH24:MI:SS');
-- 结果: 2024-07-15 10:30:45

SELECT TO_CHAR(SYSDATE, 'Day, DD Month YYYY');
-- 结果: Monday, 15 July 2024

-- DATE_FORMAT（MySQL 兼容格式）
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');
-- 结果: 2024-07-15 10:30:45

SELECT DATE_FORMAT(NOW(), '%W, %d %M %Y');
-- 结果: Monday, 15 July 2024

-- TIME_FORMAT
SELECT TIME_FORMAT('10:30:45', '%H时%i分%s秒');
-- 结果: 10时30分45秒

-- GET_FORMAT
SELECT GET_FORMAT(DATE, 'ISO');
-- 结果: %Y-%m-%d

-- UNIX 时间戳互转
SELECT UNIX_TIMESTAMP('2024-07-15 10:30:00');
-- 结果: 1721036200

SELECT FROM_UNIXTIME(1721036200);
-- 结果: 2024-07-15 10:30:00

SELECT FROM_UNIXTIME(1721036200, '%Y年%m月%d日');
-- 结果: 2024年07月15日

-- 天数转换
SELECT TO_DAYS('2024-07-15');
-- 结果: 739071

SELECT FROM_DAYS(739071);
-- 结果: 2024-07-15

-- 秒数转时间
SELECT SEC_TO_TIME(3661);
-- 结果: 01:01:01

SELECT TIME_TO_SEC('01:01:01');
-- 结果: 3661
```

### 常用格式化字符

#### Oracle 风格格式符（TO_CHAR / TO_DATE）

| 格式符 | 说明 | 示例 |
|--------|------|------|
| `YYYY` | 四位年份 | 2024 |
| `YY` | 两位年份 | 24 |
| `MM` | 两位月份 | 07 |
| `DD` | 两位日期 | 15 |
| `HH24` | 24 小时制小时 | 16 |
| `HH` / `HH12` | 12 小时制小时 | 04 |
| `MI` | 分钟 | 30 |
| `SS` | 秒 | 45 |
| `FF` / `FF6` | 微秒 | 123456 |
| `Day` | 星期全名 | Monday |
| `DY` | 星期缩写 | Mon |
| `Month` | 月份全名 | July |
| `MON` | 月份缩写 | Jul |
| `Q` | 季度 | 3 |
| `WW` | 年中的第几周 | 28 |
| `AM` / `PM` | 上午/下午标志 | PM |

#### MySQL 风格格式符（DATE_FORMAT / TIME_FORMAT）

| 格式符 | 说明 | 示例 |
|--------|------|------|
| `%Y` | 四位年份 | 2024 |
| `%y` | 两位年份 | 24 |
| `%m` | 两位月份 | 07 |
| `%d` | 两位日期 | 15 |
| `%H` | 24 小时制小时 | 16 |
| `%h` / `%I` | 12 小时制小时 | 04 |
| `%i` | 分钟 | 30 |
| `%s` / `%S` | 秒 | 45 |
| `%f` | 微秒 | 123456 |
| `%W` | 星期全名 | Monday |
| `%a` | 星期缩写 | Mon |
| `%M` | 月份全名 | July |
| `%b` | 月份缩写 | Jul |
| `%j` | 年中的第几天 | 197 |
| `%U` | 年中的第几周（Sunday 起始） | 28 |
| `%u` | 年中的第几周（Monday 起始） | 29 |
| `%p` | AM/PM | PM |
| `%T` | 24 小时制时间（HH:MM:SS） | 16:30:45 |
| `%r` | 12 小时制时间 | 04:30:45 PM |

---

## 6. 日期截断与舍入函数

对日期时间值按指定精度进行截断或四舍五入。

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `DATE_TRUNC(field, source)` | TIMESTAMP | 按指定精度截断日期时间（PostgreSQL 兼容） |
| `TRUNC(datetime [, format])` | DATE/DATETIME | 截断日期时间到指定精度（Oracle 兼容） |
| `ROUND(datetime [, format])` | DATE/DATETIME | 对日期时间按指定精度四舍五入（Oracle 兼容） |

### SQL 示例

```sql
-- DATE_TRUNC（PostgreSQL 风格）
SELECT DATE_TRUNC('month', TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 2024-07-01 00:00:00（截断到月份开始）

SELECT DATE_TRUNC('year', TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 2024-01-01 00:00:00

SELECT DATE_TRUNC('hour', TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 2024-07-15 10:00:00

SELECT DATE_TRUNC('day', TIMESTAMP '2024-07-15 10:30:45');
-- 结果: 2024-07-15 00:00:00

-- TRUNC（Oracle 风格）
SELECT TRUNC(SYSDATE, 'MM');
-- 结果: 2024-07-01（截断到月初）

SELECT TRUNC(SYSDATE, 'YYYY');
-- 结果: 2024-01-01（截断到年初）

SELECT TRUNC(SYSDATE, 'DD');
-- 结果: 2024-07-15（截断到天）

SELECT TRUNC(SYSDATE);
-- 结果: 2024-07-15（默认截断到天，去掉时间部分）

-- ROUND（Oracle 风格）
SELECT ROUND(DATE '2024-07-15', 'MM');
-- 结果: 2024-08-01（15日为月中，四舍五入到下月初）

SELECT ROUND(DATE '2024-07-14', 'MM');
-- 结果: 2024-07-01（14日四舍五入到月初）

SELECT ROUND(DATE '2024-07-15', 'YYYY');
-- 结果: 2025-01-01（7月已过半年，四舍五入到下一年）
```

### DATE_TRUNC 支持的截断精度

| 精度值 | 说明 |
|--------|------|
| `microseconds` | 微秒 |
| `milliseconds` | 毫秒 |
| `second` | 秒 |
| `minute` | 分钟 |
| `hour` | 小时 |
| `day` | 天 |
| `week` | 周（从 Monday 开始） |
| `month` | 月 |
| `quarter` | 季度 |
| `year` | 年 |

---

## 7. 其他日期时间函数

| 函数 | 返回类型 | 说明 |
|------|----------|------|
| `LAST_DAY(date)` | DATE | 返回指定日期所在月的最后一天（Oracle/MySQL 兼容） |
| `NEXT_DAY(date, weekday)` | DATE | 返回指定日期之后的第一个指定星期几的日期（Oracle 兼容） |
| `NUMTODSINTERVAL(n, unit)` | INTERVAL DAY TO SECOND | 数值转为日时间间隔（Oracle 兼容） |
| `NUMTOYMINTERVAL(n, unit)` | INTERVAL YEAR TO MONTH | 数值转为年月时间间隔（Oracle 兼容） |
| `DATE(expr)` | DATE | 提取日期部分或将表达式转为日期 |
| `TIME(expr)` | TIME | 提取时间部分或将表达式转为时间 |
| `TIMESTAMP(expr)` | TIMESTAMP | 将表达式转为时间戳 |
| `SYSDATETIME()` | DATETIME | 返回系统日期时间（SQL Server 兼容） |
| `TIMEOFDAY()` | VARCHAR | 返回当前日期时间的文本表示（PostgreSQL 兼容） |

### SQL 示例

```sql
-- LAST_DAY
SELECT LAST_DAY(DATE '2024-02-15');
-- 结果: 2024-02-29（闰年）

SELECT LAST_DAY(DATE '2024-07-15');
-- 结果: 2024-07-31

-- NEXT_DAY（Oracle 兼容）
SELECT NEXT_DAY(DATE '2024-07-15', 'FRIDAY');
-- 结果: 2024-07-19

-- 数值转间隔（Oracle 兼容）
SELECT NUMTODSINTERVAL(1.5, 'DAY');
-- 结果: INTERVAL '1 12:00:00' DAY TO SECOND

SELECT NUMTOYMINTERVAL(18, 'MONTH');
-- 结果: INTERVAL '1-6' YEAR TO MONTH

-- DATE / TIME / TIMESTAMP 构造
SELECT DATE('2024-07-15 10:30:00');
-- 结果: 2024-07-15

SELECT TIME('2024-07-15 10:30:00');
-- 结果: 10:30:00

SELECT TIMESTAMP('2024-07-15', '10:30:00');
-- 结果: 2024-07-15 10:30:00

-- TIMEOFDAY
SELECT TIMEOFDAY();
-- 结果: Mon Jul 15 16:30:45.123456 2024 CST

-- 业务场景：生成月度报表日期范围
SELECT
    TRUNC(SYSDATE, 'MM') AS month_start,
    LAST_DAY(SYSDATE) AS month_end;
-- 结果: 2024-07-01, 2024-07-31

-- 业务场景：计算下一个工作日（周一至周五）
SELECT
    CASE DAYOFWEEK(CURRENT_DATE)
        WHEN 6 THEN ADDDATE(CURRENT_DATE, 3)  -- 周五 -> 下周一
        WHEN 7 THEN ADDDATE(CURRENT_DATE, 2)  -- 周六 -> 下周一
        ELSE ADDDATE(CURRENT_DATE, 1)          -- 其他 -> 次日
    END AS next_workday;
```

---

## 8. 与 Oracle/MySQL/PostgreSQL 对比

### 8.1 获取当前时间对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 当前日期 | `CURRENT_DATE` / `CURDATE()` | `CURRENT_DATE` / `SYSDATE`（含时间） | `CURRENT_DATE` / `CURDATE()` | `CURRENT_DATE` |
| 当前时间 | `CURRENT_TIME` / `CURTIME()` | 不支持 | `CURRENT_TIME` / `CURTIME()` | `CURRENT_TIME` |
| 当前时间戳 | `NOW()` / `CURRENT_TIMESTAMP` / `SYSDATE` | `SYSDATE` / `SYSTIMESTAMP` | `NOW()` / `CURRENT_TIMESTAMP` | `NOW()` / `CURRENT_TIMESTAMP` |
| 实时时间戳 | `CLOCK_TIMESTAMP()` / `SYSDATE` | `SYSDATE` | 不支持 | `CLOCK_TIMESTAMP()` |
| 语句时间戳 | `STATEMENT_TIMESTAMP()` | 不支持 | 不支持 | `STATEMENT_TIMESTAMP()` |
| UTC 时间 | `UTC_DATE()` / `UTC_TIME()` / `UTC_TIMESTAMP()` | `SYS_EXTRACT_UTC(SYSTIMESTAMP)` | `UTC_DATE()` / `UTC_TIME()` / `UTC_TIMESTAMP()` | `CURRENT_TIMESTAMP AT TIME ZONE 'UTC'` |

### 8.2 日期提取对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| SQL 标准提取 | `EXTRACT(field FROM ...)` | `EXTRACT(field FROM ...)` | `EXTRACT(field FROM ...)` | `EXTRACT(field FROM ...)` |
| 单字段函数 | `YEAR()` / `MONTH()` / `DAY()` | 不支持（使用 EXTRACT 或 TO_CHAR） | `YEAR()` / `MONTH()` / `DAY()` | `DATE_PART()` |
| 星期函数 | `DAYOFWEEK()` / `WEEKDAY()` / `DAYNAME()` | `TO_CHAR(d,'D')` / `TO_CHAR(d,'DAY')` | `DAYOFWEEK()` / `WEEKDAY()` / `DAYNAME()` | `EXTRACT(DOW FROM ...)` / `TO_CHAR(d,'Day')` |
| DATE_PART | `DATE_PART()` | 不支持 | 不支持 | `DATE_PART()` |

### 8.3 日期运算对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 加月 | `ADD_MONTHS()` / `DATE_ADD()` | `ADD_MONTHS()` | `DATE_ADD()` / `ADDDATE()` | `date + INTERVAL 'n months'` |
| 加天 | `ADDDATE()` / `DATE_ADD()` / `date + n` | `date + n` | `DATE_ADD()` / `ADDDATE()` | `date + n` / `date + INTERVAL` |
| 日期差（天） | `DATEDIFF()` | `date1 - date2` | `DATEDIFF()` | `date1 - date2` |
| 月份差 | `MONTHS_BETWEEN()` / `TIMESTAMPDIFF(MONTH,...)` | `MONTHS_BETWEEN()` | `TIMESTAMPDIFF(MONTH,...)` / `PERIOD_DIFF()` | `EXTRACT(YEAR FROM age(...)) * 12 + EXTRACT(MONTH FROM age(...))` |
| 时间差 | `TIMEDIFF()` / `TIMESTAMPDIFF()` | `date1 - date2`（返回天数） | `TIMEDIFF()` / `TIMESTAMPDIFF()` | `time1 - time2` |
| OVERLAPS | 支持 | 不支持 | 不支持 | 支持 |

### 8.4 日期格式化对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 日期转字符串 | `TO_CHAR()` / `DATE_FORMAT()` | `TO_CHAR()` | `DATE_FORMAT()` | `TO_CHAR()` |
| 字符串转日期 | `TO_DATE()` / `STR_TO_DATE()` | `TO_DATE()` | `STR_TO_DATE()` | `TO_DATE()` / `TO_TIMESTAMP()` |
| 格式符风格 | Oracle 风格 + MySQL 风格均支持 | Oracle 风格（YYYY-MM-DD） | MySQL 风格（%Y-%m-%d） | Oracle 风格（YYYY-MM-DD） |
| UNIX 时间戳 | `UNIX_TIMESTAMP()` / `FROM_UNIXTIME()` | 需手动计算 | `UNIX_TIMESTAMP()` / `FROM_UNIXTIME()` | `EXTRACT(EPOCH FROM ...)` / `TO_TIMESTAMP()` |

### 8.5 日期截断对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 截断 | `TRUNC()` / `DATE_TRUNC()` | `TRUNC()` | 不支持（需用 DATE_FORMAT 模拟） | `DATE_TRUNC()` |
| 四舍五入 | `ROUND()` | `ROUND()` | 不支持 | 不支持 |

### 8.6 其他函数对比

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 月末日期 | `LAST_DAY()` | `LAST_DAY()` | `LAST_DAY()` | 无内置函数（需表达式模拟） |
| 下一个星期几 | `NEXT_DAY()` | `NEXT_DAY()` | 不支持 | 不支持 |
| 构造日期 | `MAKE_DATE()` / `MAKEDATE()` | 不支持 | `MAKEDATE()` | `MAKE_DATE()` |
| 构造时间戳 | `MAKE_TIMESTAMP()` | 不支持 | 不支持 | `MAKE_TIMESTAMP()` |
| 数值转间隔 | `NUMTODSINTERVAL()` / `NUMTOYMINTERVAL()` | `NUMTODSINTERVAL()` / `NUMTOYMINTERVAL()` | 不支持 | 不支持 |

> **迁移提示**：虚谷数据库同时兼容 Oracle 风格（`TO_CHAR`/`TO_DATE`/`ADD_MONTHS`/`TRUNC`/`LAST_DAY`/`NEXT_DAY`）和 MySQL 风格（`DATE_FORMAT`/`DATE_ADD`/`DATEDIFF`/`CURDATE`/`NOW`）以及 PostgreSQL 风格（`DATE_TRUNC`/`DATE_PART`/`MAKE_DATE`/`MAKE_TIMESTAMP`）的日期函数。从这三种数据库迁移时，大部分日期时间函数可直接使用，无需修改。仅需注意格式符的差异：Oracle 使用 `YYYY-MM-DD HH24:MI:SS`，MySQL 使用 `%Y-%m-%d %H:%i:%s`，虚谷数据库两种格式均支持。
