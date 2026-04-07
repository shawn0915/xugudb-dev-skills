---
title: 虚谷数据库数学函数
description: 59个数学函数详解，包括基本运算、三角函数、对数指数、取整舍入、随机数、进制转换等功能分组，含语法、示例及与 Oracle/MySQL/PostgreSQL 对照
tags: xugudb, math-functions, abs, round, ceil, floor, mod, power, log, sqrt, sin, cos, tan, random, trunc, factorial, gcd
---

# 虚谷数据库数学函数

虚谷数据库提供 59 个数学函数，覆盖基本运算、三角函数、对数指数、取整舍入、随机数生成、进制转换等场景。

---

## 基本运算函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| ABS | 返回绝对值 | `ABS(n)` |
| SIGN | 返回数值的符号（-1、0 或 1） | `SIGN(n)` |
| MOD | 返回取模（求余）结果 | `MOD(n, m)` |
| REMAINDER | 返回余数（与 MOD 的舍入方式不同） | `REMAINDER(n, m)` |
| DIV | 返回整数除法结果 | `DIV(n, m)` |
| GCD | 返回最大公约数 | `GCD(a, b)` |
| LCM | 返回最小公倍数 | `LCM(a, b)` |
| FACTORIAL | 返回阶乘 | `FACTORIAL(n)` |
| SQUARE | 返回平方值 | `SQUARE(n)` |
| SQRT | 返回平方根 | `SQRT(n)` |
| CBRT | 返回立方根 | `CBRT(n)` |
| NANVL | 如果值为 NaN 则返回替代值 | `NANVL(n, substitute)` |
| PI | 返回圆周率 π | `PI()` |

### 基本运算示例

```sql
-- 绝对值
SELECT ABS(-42);
-- 结果: 42

-- 符号函数
SELECT SIGN(-15), SIGN(0), SIGN(15);
-- 结果: -1, 0, 1

-- 取模
SELECT MOD(17, 5);
-- 结果: 2

-- REMAINDER 与 MOD 的区别：REMAINDER 使用四舍五入取整
SELECT MOD(11, 4), REMAINDER(11, 4);
-- 结果: 3, -1（REMAINDER = 11 - 4 * ROUND(11/4) = 11 - 4*3 = -1）

-- 整数除法
SELECT DIV(17, 5);
-- 结果: 3

-- 最大公约数和最小公倍数
SELECT GCD(12, 18);
-- 结果: 6
SELECT LCM(12, 18);
-- 结果: 36

-- 阶乘
SELECT FACTORIAL(5);
-- 结果: 120

-- 平方与根
SELECT SQUARE(5);
-- 结果: 25
SELECT SQRT(25);
-- 结果: 5
SELECT CBRT(27);
-- 结果: 3

-- 圆周率
SELECT PI();
-- 结果: 3.141592653589793

-- NaN 值处理
SELECT NANVL('NaN'::DOUBLE, 0);
-- 结果: 0
```

---

## 三角函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| SIN | 正弦（弧度） | `SIN(n)` |
| SIND | 正弦（角度） | `SIND(n)` |
| SINH | 双曲正弦 | `SINH(n)` |
| COS | 余弦（弧度） | `COS(n)` |
| COSD | 余弦（角度） | `COSD(n)` |
| COSH | 双曲余弦 | `COSH(n)` |
| TAN | 正切（弧度） | `TAN(n)` |
| TAND | 正切（角度） | `TAND(n)` |
| TANH | 双曲正切 | `TANH(n)` |
| COT | 余切（弧度） | `COT(n)` |
| COTD | 余切（角度） | `COTD(n)` |
| ASIN | 反正弦（返回弧度） | `ASIN(n)` |
| ASIND | 反正弦（返回角度） | `ASIND(n)` |
| ASINH | 反双曲正弦 | `ASINH(n)` |
| ACOS | 反余弦（返回弧度） | `ACOS(n)` |
| ACOSD | 反余弦（返回角度） | `ACOSD(n)` |
| ACOSH | 反双曲余弦 | `ACOSH(n)` |
| ATAN | 反正切（返回弧度） | `ATAN(n)` |
| ATAND | 反正切（返回角度） | `ATAND(n)` |
| ATANH | 反双曲正切 | `ATANH(n)` |
| ATAN2 | 两参数反正切（返回弧度） | `ATAN2(y, x)` |
| ATAN2D | 两参数反正切（返回角度） | `ATAN2D(y, x)` |
| DEGREES | 弧度转角度 | `DEGREES(radians)` |
| RADIANS | 角度转弧度 | `RADIANS(degrees)` |

### 三角函数示例

```sql
-- 正弦（弧度）
SELECT SIN(PI() / 2);
-- 结果: 1

-- 正弦（角度）
SELECT SIND(90);
-- 结果: 1

-- 余弦
SELECT COS(0);
-- 结果: 1
SELECT COSD(60);
-- 结果: 0.5

-- 正切
SELECT TAN(PI() / 4);
-- 结果: 1（近似值）

-- 反三角函数
SELECT ASIN(1);
-- 结果: 1.5707963267948966（即 π/2）
SELECT ASIND(1);
-- 结果: 90

-- 两参数反正切
SELECT ATAN2(1, 1);
-- 结果: 0.7853981633974483（即 π/4）
SELECT ATAN2D(1, 1);
-- 结果: 45

-- 弧度角度转换
SELECT DEGREES(PI());
-- 结果: 180
SELECT RADIANS(180);
-- 结果: 3.141592653589793

-- 双曲函数
SELECT SINH(1), COSH(1), TANH(1);
-- 结果: 1.1752011936438014, 1.5430806348152437, 0.7615941559557649
```

---

## 对数与指数函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| EXP | 返回 e 的 n 次方 | `EXP(n)` |
| LN | 返回自然对数 | `LN(n)` |
| LOG | 返回指定底数的对数 | `LOG(base, n)` |
| LOG2 | 返回以 2 为底的对数 | `LOG2(n)` |
| LOG10 | 返回以 10 为底的对数 | `LOG10(n)` |
| POW | 返回 x 的 y 次方 | `POW(x, y)` |
| POWER | 返回 x 的 y 次方（同 POW） | `POWER(x, y)` |
| ERF | 误差函数 | `ERF(n)` |
| ERFC | 互补误差函数 | `ERFC(n)` |

### 对数与指数示例

```sql
-- e 的幂
SELECT EXP(1);
-- 结果: 2.718281828459045

-- 自然对数
SELECT LN(EXP(1));
-- 结果: 1

-- 指定底数的对数
SELECT LOG(10, 1000);
-- 结果: 3

-- 以 2 为底
SELECT LOG2(8);
-- 结果: 3

-- 以 10 为底
SELECT LOG10(100);
-- 结果: 2

-- 幂运算
SELECT POWER(2, 10);
-- 结果: 1024

-- 误差函数
SELECT ERF(1);
-- 结果: 0.8427007929497149（近似值）
```

---

## 取整与舍入函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| CEIL | 向上取整（返回不小于参数的最小整数） | `CEIL(n)` |
| CEILING | 向上取整（同 CEIL） | `CEILING(n)` |
| FLOOR | 向下取整（返回不大于参数的最大整数） | `FLOOR(n)` |
| ROUND | 四舍五入到指定小数位 | `ROUND(n [, decimal_places])` |
| ROUND_TIES_TO_EVEN | 银行家舍入法（四舍六入五成双） | `ROUND_TIES_TO_EVEN(n [, decimal_places])` |
| TRUNC | 截断到指定小数位（不做舍入） | `TRUNC(n [, decimal_places])` |
| TRUNCATE | 截断到指定小数位（同 TRUNC） | `TRUNCATE(n [, decimal_places])` |

### 取整与舍入示例

```sql
-- 向上取整
SELECT CEIL(4.1), CEIL(-4.9);
-- 结果: 5, -4

-- 向下取整
SELECT FLOOR(4.9), FLOOR(-4.1);
-- 结果: 4, -5

-- 四舍五入
SELECT ROUND(3.14159, 2);
-- 结果: 3.14
SELECT ROUND(3.145, 2);
-- 结果: 3.15

-- 四舍五入到整数
SELECT ROUND(3.5), ROUND(4.5);
-- 结果: 4, 5

-- 银行家舍入法（五成双）
SELECT ROUND_TIES_TO_EVEN(3.5), ROUND_TIES_TO_EVEN(4.5);
-- 结果: 4, 4（恰好为 .5 时，舍入到最近的偶数）

-- ROUND 支持负数位：对整数部分舍入
SELECT ROUND(1234.5, -2);
-- 结果: 1200

-- 截断（不做舍入）
SELECT TRUNC(3.14159, 2);
-- 结果: 3.14
SELECT TRUNC(3.999, 0);
-- 结果: 3

-- 截断到十位
SELECT TRUNC(1234.5, -2);
-- 结果: 1200
```

---

## 随机数函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| RAND | 返回 [0, 1) 之间的随机浮点数 | `RAND([seed])` |
| RANDOM | 返回随机整数 | `RANDOM()` |
| RANDOM_NORMAL | 返回正态分布随机数 | `RANDOM_NORMAL([mean, stddev])` |
| SRAND | 设置随机数种子 | `SRAND(seed)` |
| SETSEED | 设置随机数种子（PostgreSQL 兼容） | `SETSEED(seed)` |

### 随机数示例

```sql
-- 0 到 1 之间的随机数
SELECT RAND();
-- 结果: 0.7231742029971469（每次不同）

-- 设置种子后生成可复现的随机数
SELECT SETSEED(0.42);
SELECT RANDOM();

-- 随机整数
SELECT RANDOM();

-- 正态分布随机数（均值 0，标准差 1）
SELECT RANDOM_NORMAL();

-- 指定均值和标准差的正态分布
SELECT RANDOM_NORMAL(100, 15);

-- 生成指定范围的随机整数（例如 1-100）
SELECT FLOOR(RAND() * 100) + 1;
```

---

## 进制转换函数

| 函数名 | 功能 | 语法 |
|--------|------|------|
| CONV | 进制转换 | `CONV(n, from_base, to_base)` |

### 进制转换示例

```sql
-- 十进制转二进制
SELECT CONV(255, 10, 2);
-- 结果: 11111111

-- 十进制转十六进制
SELECT CONV(255, 10, 16);
-- 结果: FF

-- 十六进制转十进制
SELECT CONV('FF', 16, 10);
-- 结果: 255

-- 八进制转十进制
SELECT CONV('77', 8, 10);
-- 结果: 63

-- 二进制转十六进制
SELECT CONV('11111111', 2, 16);
-- 结果: FF
```

---

## 与 Oracle / MySQL / PostgreSQL 函数名对照表

| 功能 | 虚谷数据库 | Oracle | MySQL | PostgreSQL |
|------|-----------|--------|-------|------------|
| 绝对值 | ABS | ABS | ABS | ABS |
| 符号 | SIGN | SIGN | SIGN | SIGN |
| 取模 | MOD | MOD | MOD / % | MOD / % |
| 整数除法 | DIV | TRUNC(a/b) | DIV | DIV（运算符） |
| 最大公约数 | GCD | 无（需自行实现） | 无 | GCD |
| 最小公倍数 | LCM | 无 | 无 | LCM |
| 阶乘 | FACTORIAL | 无 | 无 | FACTORIAL |
| 平方 | SQUARE | POWER(n,2) | POW(n,2) | POWER(n,2) |
| 平方根 | SQRT | SQRT | SQRT | SQRT |
| 立方根 | CBRT | POWER(n,1/3) | POW(n,1/3) | CBRT |
| 向上取整 | CEIL / CEILING | CEIL | CEIL / CEILING | CEIL / CEILING |
| 向下取整 | FLOOR | FLOOR | FLOOR | FLOOR |
| 四舍五入 | ROUND | ROUND | ROUND | ROUND |
| 银行家舍入 | ROUND_TIES_TO_EVEN | 无 | 无 | 无（虚谷特有） |
| 截断 | TRUNC / TRUNCATE | TRUNC | TRUNCATE | TRUNC |
| 幂运算 | POW / POWER | POWER | POW / POWER | POWER |
| 自然对数 | LN | LN | LN | LN |
| 指定底数对数 | LOG | LOG | LOG | LOG |
| 以 10 为底 | LOG10 | LOG(10,n) | LOG10 | LOG |
| 以 2 为底 | LOG2 | LOG(2,n) | LOG2 | LOG |
| 指数 | EXP | EXP | EXP | EXP |
| 正弦 | SIN / SIND | SIN | SIN | SIN / SIND |
| 余弦 | COS / COSD | COS | COS | COS / COSD |
| 正切 | TAN / TAND | TAN | TAN | TAN / TAND |
| 余切 | COT / COTD | 1/TAN(n) | COT | COT / COTD |
| 反正弦 | ASIN / ASIND | ASIN | ASIN | ASIN / ASIND |
| 反余弦 | ACOS / ACOSD | ACOS | ACOS | ACOS / ACOSD |
| 反正切 | ATAN / ATAND | ATAN | ATAN | ATAN / ATAND |
| 两参数反正切 | ATAN2 / ATAN2D | ATAN2 | ATAN2 | ATAN2 / ATAN2D |
| 双曲正弦 | SINH | SINH | 无 | SINH |
| 双曲余弦 | COSH | COSH | 无 | COSH |
| 双曲正切 | TANH | TANH | 无 | TANH |
| 弧度转角度 | DEGREES | 180/PI()*n | DEGREES | DEGREES |
| 角度转弧度 | RADIANS | PI()/180*n | RADIANS | RADIANS |
| 圆周率 | PI | ACOS(-1) | PI() | PI() |
| 随机数 | RAND / RANDOM | DBMS_RANDOM.VALUE | RAND | RANDOM |
| 随机数种子 | SRAND / SETSEED | DBMS_RANDOM.SEED | 无（RAND(seed)） | SETSEED |
| 正态分布随机数 | RANDOM_NORMAL | 无 | 无 | 无（虚谷特有） |
| 进制转换 | CONV | 无（TO_CHAR 部分支持） | CONV | 无（TO_HEX 等） |
| NaN 值处理 | NANVL | NANVL | 无 | 无 |
| 误差函数 | ERF / ERFC | 无 | 无 | 无（虚谷特有） |
| 余数 | REMAINDER | REMAINDER | 无 | 无 |

> **迁移提示**：
> - 从 Oracle 迁移时，ABS、ROUND、TRUNC、MOD、POWER、SQRT、LN、三角函数等可直接使用。Oracle 的 LOG(base, n) 语法与虚谷兼容。NANVL 函数可直接迁移。
> - 从 MySQL 迁移时，ABS、CEIL、FLOOR、ROUND、TRUNCATE、MOD、POW、SQRT、LOG、LOG2、LOG10、三角函数等均可直接使用。CONV 进制转换函数可直接迁移。
> - 从 PostgreSQL 迁移时，ABS、CEIL、FLOOR、ROUND、TRUNC、SQRT、CBRT、LN、LOG、POWER、GCD、LCM、FACTORIAL、三角函数的角度版本（SIND/COSD 等）均可直接使用。SETSEED 随机数种子函数兼容。
> - ROUND_TIES_TO_EVEN（银行家舍入）、RANDOM_NORMAL（正态分布随机数）、ERF/ERFC（误差函数）是虚谷数据库的特色函数。
