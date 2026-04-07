# 视图、序列与其他对象

## 视图

### 创建视图

```sql
CREATE [OR REPLACE] [FORCE|NOFORCE] VIEW [IF NOT EXISTS] [schema.]view_name [(col_list)]
AS select_stmt
[WITH READ ONLY | WITH CHECK OPTION]
[COMMENT 'description'];
```

- **OR REPLACE**：已存在则替换
- **FORCE**：即使引用的表不存在也创建（视图标记为无效）
- **NOFORCE**：引用的表必须存在（默认）
- **WITH READ ONLY**：只读视图
- **WITH CHECK OPTION**：INSERT/UPDATE 必须满足视图的 WHERE 条件

### 示例

```sql
-- 简单视图
CREATE VIEW emp_view (id, name, dept)
AS SELECT employee_id, employee_name, department
FROM employees;

-- 只读视图
CREATE VIEW readonly_view AS SELECT * FROM sensitive_data WITH READ ONLY;

-- 多表连接视图
CREATE VIEW order_detail AS
SELECT u.name, o.product, o.amount
FROM users u JOIN orders o ON u.id = o.user_id
WITH CHECK OPTION
COMMENT '订单详情视图';
```

### 管理视图

```sql
-- 替换视图
CREATE OR REPLACE VIEW emp_view AS SELECT * FROM employees;

-- 删除视图
DROP VIEW [IF EXISTS] view_name;

-- 查询视图信息
SELECT view_name, define, option, valid FROM SYS_VIEWS;
SELECT * FROM SYS_VIEW_COLUMNS WHERE view_id = ...;
```

## 序列

### 创建序列

```sql
CREATE SEQUENCE [IF NOT EXISTS] [schema.]seq_name
    [START WITH n]              -- 起始值（默认 1）
    [INCREMENT BY n]            -- 步长（默认 1）
    [MINVALUE n | NOMINVALUE]   -- 最小值
    [MAXVALUE n | NOMAXVALUE]   -- 最大值（默认 9223372036854775807）
    [CACHE n | NOCACHE]         -- 缓存大小
    [CYCLE | NOCYCLE]           -- 是否循环
    [COMMENT 'desc'];
```

### 使用序列

```sql
SELECT seq_name.NEXTVAL FROM DUAL;   -- 获取下一个值
SELECT seq_name.CURRVAL FROM DUAL;   -- 获取当前值

-- INSERT 中使用
INSERT INTO t (id, name) VALUES (seq_name.NEXTVAL, 'test');
```

### IDENTITY 列（替代序列的自增方式）

```sql
CREATE TABLE t (
    id INT IDENTITY(1, 1) PRIMARY KEY,  -- 从 1 开始，步长 1
    name VARCHAR(100)
);
INSERT INTO t (name) VALUES ('auto_id');  -- id 自动生成
```

### 管理序列

```sql
-- 修改序列
ALTER SEQUENCE seq_name INCREMENT BY 2;

-- 删除序列
DROP SEQUENCE [IF EXISTS] seq_name;

-- 查询序列
SELECT * FROM USER_SEQUENCES;
```

## 同义词

```sql
-- 创建同义词（为对象起别名）
CREATE [OR REPLACE] [PUBLIC] SYNONYM [schema.]syn_name
FOR [schema.]object_name;

-- 示例
CREATE SYNONYM syn_emp FOR schema1.employees;
SELECT * FROM syn_emp;  -- 等同于 SELECT * FROM schema1.employees

-- 公共同义词（所有用户可用，需 CREATE PUBLIC SYNONYM 权限）
CREATE PUBLIC SYNONYM pub_emp FOR employees;

-- 删除同义词
DROP [PUBLIC] SYNONYM syn_name;
```

> 注意：同义词不能指向触发器。

## DBLink（数据库链接）

```sql
-- 创建 ODBC DBLink
CREATE [PUBLIC] DATABASE LINK link_name
FOR db_type (CONNECT TO 'host:port/dbname')
USER user_name IDENTIFIED BY password
USING 'odbc_driver_info';

-- 创建 XGCI DBLink（连接其他虚谷数据库）
CREATE DATABASE LINK link_name
FOR XUGU (CONNECT TO 'host:port/dbname')
USER user_name IDENTIFIED BY password;

-- 使用 DBLink
SELECT * FROM table_name@link_name;

-- 删除 DBLink
DROP DATABASE LINK link_name;
```

支持的远程数据库类型：Oracle、MySQL、PostgreSQL 等（通过 ODBC）、虚谷（通过 XGCI）。

## 定时作业

```sql
-- 创建定时作业
EXEC DBMS_SCHEDULER.CREATE_JOB(
    job_name        => 'job_name',
    job_type        => 'stored_procedure',  -- 或 plsql_block, plsql_command
    job_action      => 'proc_name',         -- 存储过程名或 PL/SQL 块
    start_date      => SYSTIMESTAMP,
    repeat_interval => 'FREQ=DAILY;BYHOUR=2;BYMINUTE=0',
    enabled         => TRUE,
    auto_drop       => TRUE,
    comments        => '每日凌晨2点执行'
);

-- job_type 说明：
-- stored_procedure: 执行存储过程
-- plsql_block: 执行 PL/SQL 匿名块
-- plsql_command: 执行 SQL（DML/DDL）
```

## 加密机

```sql
-- 创建加密机（需 SYSSSO 权限）
CREATE ENCRYPTOR 'enc_name' BY 'encryption_key';

-- 使用加密机加密表
CREATE TABLE secret_data (id INT, data VARCHAR(200))
ENCRYPT BY 'enc_name';

-- 使用加密机加密备份
BACKUP SYSTEM TO '/backup/sys.dmp' ENCRYPTOR IS 'enc_name';

-- 查看加密机
SELECT * FROM sys_encryptors;
```
