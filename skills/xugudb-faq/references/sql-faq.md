# SQL 常见问题

## 字符串与转义

### 单引号转义

SQL 中字符串使用单引号，字符串内的单引号用两个单引号转义：

```sql
INSERT INTO emp(name) VALUES ('O''Reilly');

-- PL/SQL 中
EXECUTE DBMS_SCHEDULER.CREATE_JOB('test1', 'plsql_block',
    'declare begin insert into test1 values(11,''xiaohei'',15,''beijing'');end;',
    ...);
```

## GROUP BY 相关

### 取每组最大/最新记录

```sql
-- 方式1：窗口函数 ROW_NUMBER（推荐）
SELECT c1, c2 FROM (
    SELECT c1, c2, ROW_NUMBER() OVER (PARTITION BY c1 ORDER BY c2 DESC) AS rn
    FROM t_exp2
) t WHERE rn = 1;

-- 方式2：子查询 + JOIN
SELECT e.* FROM t_exp2 e
JOIN (SELECT c1, MAX(c2) AS max_c2 FROM t_exp2 GROUP BY c1) m
ON e.c1 = m.c1 AND e.c2 = m.max_c2;
```

> - 数据量 <500 万行：窗口函数更简洁
> - 数据量 >500 万行：JOIN 方式可能更快（避免全表排序）

### GROUP BY 取非聚合列

虚谷数据库要求 SELECT 的非聚合列必须出现在 GROUP BY 中（与 MySQL 的 `ONLY_FULL_GROUP_BY` 行为一致）：

```sql
-- 错误
SELECT id, name FROM test GROUP BY id;

-- 正确：使用窗口函数
SELECT id, name FROM (
    SELECT id, name, ROW_NUMBER() OVER (PARTITION BY id ORDER BY id) rn
    FROM test
) t WHERE t.rn = 1;
```

## 条件判断

### IF / DECODE / CASE

```sql
-- CASE WHEN（推荐，标准 SQL）
SELECT CASE WHEN status = 1 THEN '启用' ELSE '禁用' END AS status_text FROM t;

-- DECODE（Oracle 兼容）
SELECT DECODE(status, 1, '启用', '禁用') AS status_text FROM t;
```

## 序列使用

```sql
-- 创建序列
CREATE SEQUENCE seq_id START WITH 1 INCREMENT BY 1 CACHE 20;

-- 获取下一个值
SELECT seq_id.NEXTVAL FROM DUAL;

-- 在 INSERT 中使用
INSERT INTO t (id, name) VALUES (seq_id.NEXTVAL, 'test');

-- IDENTITY 列（更简洁）
CREATE TABLE t (id INT IDENTITY(1,1) PRIMARY KEY, name VARCHAR(100));
```

## 字符串拼接

```sql
-- CONCAT 函数
SELECT CONCAT(first_name, ' ', last_name) FROM emp;

-- || 运算符
SELECT first_name || ' ' || last_name FROM emp;
```

## EXPLAIN 查看执行计划

```sql
EXPLAIN SELECT * FROM t WHERE id = 1;
```

## 行号与分页

```sql
-- ROWNUM 伪列
SELECT * FROM t WHERE ROWNUM <= 10;

-- ROW_NUMBER 窗口函数
SELECT * FROM (
    SELECT t.*, ROW_NUMBER() OVER (ORDER BY id) AS rn FROM t
) WHERE rn BETWEEN 11 AND 20;
```

## 大文本（CLOB/BLOB）操作

```sql
-- 查询 CLOB 长度
SELECT LENGTH(clob_col) FROM t;

-- CLOB 转 VARCHAR
SELECT CAST(clob_col AS VARCHAR(1000)) FROM t;
```

## IF NOT EXISTS / IF EXISTS

```sql
-- 建表
CREATE TABLE IF NOT EXISTS t (id INT);

-- 删表
DROP TABLE IF EXISTS t;

-- 建索引
CREATE INDEX IF NOT EXISTS idx_name ON t (col);

-- 删索引
DROP INDEX IF EXISTS idx_name;
```

## 排除特定错误

```sql
-- SET EXCLUDE_ERRNO 忽略特定错误
SET EXCLUDE_ERRNO = 'E5021';  -- 忽略"表不存在"错误
DROP TABLE t;  -- 表不存在时不报错
SET EXCLUDE_ERRNO = '';  -- 清除忽略设置
```
