# 存储过程、函数、触发器与包

## 存储过程

### 创建

```sql
CREATE [OR REPLACE] [FORCE|NOFORCE] PROCEDURE [IF NOT EXISTS] [schema.]proc_name
    (param_list)
    [AUTHID DEFAULT|CURRENT_USER|DEFINER]
AS
    [DECLARE variable_list]
BEGIN
    statements;
END [proc_name];
/
```

### 参数模式

| 模式 | 说明 |
|------|------|
| IN | 输入参数（默认） |
| OUT | 输出参数 |
| IN OUT | 输入输出参数 |

### 示例

```sql
CREATE OR REPLACE PROCEDURE proc_add(
    a IN INT,
    b IN INT,
    sum OUT INT
) AS
BEGIN
    sum := a + b;
END;
/

-- 调用
DECLARE
    result INT;
BEGIN
    proc_add(100, 200, result);
    DBMS_OUTPUT.PUT_LINE('结果: ' || result);
END;
/
```

### 管理

```sql
ALTER PROCEDURE proc_name RECOMPILE;
DROP PROCEDURE [IF EXISTS] proc_name;
SELECT * FROM ALL_PROCEDURES;
```

## 存储函数

### 创建

```sql
CREATE [OR REPLACE] FUNCTION [IF NOT EXISTS] [schema.]func_name
    (param_list)
    RETURN return_type
    [AUTHID DEFAULT|CURRENT_USER|DEFINER]
    [PIPELINED]     -- 管道函数
AS
    [DECLARE variable_list]
BEGIN
    statements;
    RETURN value;
END [func_name];
/
```

### 示例

```sql
CREATE OR REPLACE FUNCTION func_add(a INT, b INT)
RETURN INT AS
BEGIN
    RETURN a + b;
END;
/

-- 在 SQL 中使用
SELECT func_add(10, 20) FROM DUAL;  -- 30

-- 在 WHERE 中使用
SELECT * FROM t WHERE func_add(col1, col2) > 100;
```

## 触发器

### 创建

```sql
CREATE [OR REPLACE] [FORCE|NOFORCE] TRIGGER [IF NOT EXISTS] trigger_name
    {BEFORE | AFTER | INSTEAD OF}
    {INSERT | DELETE | UPDATE [OF col_list]}
    [OR {INSERT | DELETE | UPDATE [OF col_list]}]
    ON table_name
    [REFERENCING OLD AS old_alias NEW AS new_alias]
    FOR {EACH ROW | STATEMENT}
    [WHEN (condition)]
    [COMMENT 'description']
    [DECLARE variable_list]
BEGIN
    statements;
END [trigger_name];
/
```

### 触发时机

| 时机 | 说明 |
|------|------|
| BEFORE | 操作执行前触发 |
| AFTER | 操作执行后触发 |
| INSTEAD OF | 替代原操作（用于视图） |

### 触发级别

| 级别 | 说明 |
|------|------|
| FOR EACH ROW | 每行触发 |
| FOR STATEMENT | 每语句触发 |

### 示例

```sql
-- 自动填充时间戳
CREATE OR REPLACE TRIGGER trg_update_time
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    :NEW.update_time := SYSDATE;
END;
/

-- 审计日志
CREATE OR REPLACE TRIGGER trg_audit
AFTER INSERT OR DELETE ON sensitive_table
FOR EACH ROW
BEGIN
    IF INSERTING THEN
        INSERT INTO audit_log VALUES('INSERT', SYSDATE, :NEW.id);
    ELSIF DELETING THEN
        INSERT INTO audit_log VALUES('DELETE', SYSDATE, :OLD.id);
    END IF;
END;
/
```

### 管理

```sql
ALTER TRIGGER trigger_name ENABLE;
ALTER TRIGGER trigger_name DISABLE;
DROP TRIGGER [IF EXISTS] trigger_name;
```

> 注意：批量插入（如 `INSERT INTO t VALUES(1)(2)(3)`）中，FOR EACH ROW 触发器对每行都触发。

## 包（Package）

### 包规范（声明）

```sql
CREATE [OR REPLACE] [FORCE] PACKAGE [schema.]pkg_name
    [AUTHID DEFAULT|CURRENT_USER|DEFINER]
    [COMMENT 'description']
IS | AS
    -- 变量声明
    var1 INT;
    var2 VARCHAR := 'default';

    -- 过程声明
    PROCEDURE proc_name(param1 INT);

    -- 函数声明
    FUNCTION func_name(param1 VARCHAR) RETURN VARCHAR;
END [pkg_name];
/
```

### 包体（实现）

```sql
CREATE [OR REPLACE] PACKAGE BODY [schema.]pkg_name
IS | AS
    [DECLARE variable_list]

    PROCEDURE proc_name(param1 INT)
    AS
    BEGIN
        -- 实现
    END;

    FUNCTION func_name(param1 VARCHAR)
    RETURN VARCHAR
    AS
    BEGIN
        RETURN param1 || '_processed';
    END;

    [BEGIN
        -- 初始化代码
    ]
END [pkg_name];
/
```

### 调用包成员

```sql
-- 调用包中的过程
EXEC pkg_name.proc_name(100);

-- 调用包中的函数
SELECT pkg_name.func_name('test') FROM DUAL;

-- 访问包变量
SELECT pkg_name.var1 FROM DUAL;
```

### 管理

```sql
ALTER PACKAGE pkg_name RECOMPILE;
DROP PACKAGE [BODY] pkg_name;
SELECT * FROM USER_PACKAGES;
```
