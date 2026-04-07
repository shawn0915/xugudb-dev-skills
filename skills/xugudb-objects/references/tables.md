# 表管理

## 创建表

```sql
CREATE [GLOBAL TEMP | TEMP] TABLE [IF NOT EXISTS] [schema.]table_name (
    column_def [, column_def | table_constraint ...]
)
[ON COMMIT DELETE|PRESERVE ROWS]      -- 临时表选项
[PCTFREE n]                           -- 空闲空间百分比
[PARTITION BY RANGE|LIST|HASH (...)]  -- 分区
[SUBPARTITION BY HASH (...)]          -- 子分区
[COMMENT 'description']               -- 注释
[ENCRYPT BY 'encryptor_name'];        -- 加密
```

### 列定义

```sql
column_name type_name
    [IDENTITY(start, step)]   -- 自增列
    [PRIMARY KEY]             -- 主键
    [NOT NULL]                -- 非空
    [DEFAULT expr]            -- 默认值
    [CHECK (condition)]       -- 检查约束
    [REFERENCES parent(col)]  -- 外键
    [COMMENT 'desc']          -- 列注释
```

### 完整建表示例

```sql
CREATE TABLE IF NOT EXISTS employees (
    employee_id INT IDENTITY(1,1) PRIMARY KEY,
    employee_name VARCHAR(50) NOT NULL
        CONSTRAINT ck_name CHECK (LENGTH(employee_name) >= 2),
    email VARCHAR(100) DEFAULT 'someone@example.com',
    hire_date DATE NOT NULL DEFAULT SYSDATE,
    salary NUMERIC(10,2) CHECK (salary > 0),
    department_id NUMERIC(4,0)
        CONSTRAINT fk_dept REFERENCES departments(department_id),
    CONSTRAINT uk_email UNIQUE (email),
    CONSTRAINT ck_salary CHECK (salary <= 100000)
)
PARTITION BY RANGE (salary) PARTITIONS (
    part1 VALUES LESS THAN (5000),
    part2 VALUES LESS THAN (10000),
    part3 VALUES LESS THAN (MAXVALUES)
)
SUBPARTITION BY HASH (employee_id) SUBPARTITIONS 2
PCTFREE 10
COMMENT '员工表';
```

## 临时表

```sql
-- 本地临时表（默认会话结束时删除数据）
CREATE TEMP TABLE temp_data (id INT, val VARCHAR(100));

-- 全局临时表（需先开启）
SET support_global_tab ON;
CREATE GLOBAL TEMP TABLE global_temp (id INT)
ON COMMIT PRESERVE ROWS;  -- 提交后保留数据
-- ON COMMIT DELETE ROWS;  -- 提交后删除数据（默认）
```

> 注意：全局临时表需要设置 `support_global_tab` 参数为 ON

## 分区表

### 范围分区（RANGE）

```sql
CREATE TABLE orders (
    id INT, order_date DATE, amount NUMERIC(10,2)
)
PARTITION BY RANGE (order_date) PARTITIONS (
    p2024 VALUES LESS THAN ('2025-01-01'),
    p2025 VALUES LESS THAN ('2026-01-01'),
    p_max VALUES LESS THAN (MAXVALUES)
);
```

### 列表分区（LIST）

```sql
CREATE TABLE regions (
    id INT, region VARCHAR(20), data VARCHAR(100)
)
PARTITION BY LIST (region) PARTITIONS (
    p_east VALUES ('北京', '上海'),
    p_west VALUES ('成都', '重庆'),
    p_other VALUES (OTHERVALUES)
);
```

### 哈希分区（HASH）

```sql
CREATE TABLE logs (id INT, msg VARCHAR(200))
PARTITION BY HASH (id) PARTITIONS 4;

-- 或指定分区名
PARTITION BY HASH (id) PARTITIONS (p1, p2, p3, p4);
```

### 分区管理

```sql
-- 添加分区
ALTER TABLE orders ADD PARTITION p2026 VALUES LESS THAN ('2027-01-01');

-- 删除分区
ALTER TABLE orders DROP PARTITION p2024 REBUILD GLOBAL INDEX;

-- 截断分区
ALTER TABLE orders TRUNCATE PARTITION p2024 REBUILD GLOBAL INDEX;

-- 设置分区在线/离线
ALTER TABLE orders SET PARTITION p2025 OFFLINE;
```

## 修改表

```sql
-- 添加列
ALTER TABLE t ADD COLUMN new_col VARCHAR(100);

-- 修改列类型
ALTER TABLE t ALTER COLUMN col SET DATA TYPE BIGINT;

-- 删除列
ALTER TABLE t DROP COLUMN old_col [CASCADE];

-- 添加约束
ALTER TABLE t ADD CONSTRAINT pk PRIMARY KEY (id);
ALTER TABLE t ADD CONSTRAINT uk UNIQUE (email);
ALTER TABLE t ADD CONSTRAINT fk FOREIGN KEY (dept_id) REFERENCES dept(id);
ALTER TABLE t ADD CONSTRAINT ck CHECK (age > 0);

-- 启用/禁用约束
ALTER TABLE t ENABLE CONSTRAINT cons_name;
ALTER TABLE t DISABLE CONSTRAINT cons_name;

-- 删除约束
ALTER TABLE t DROP CONSTRAINT cons_name [KEEP INDEX | DROP INDEX];

-- 转移所有权
ALTER TABLE t OWNER TO new_user;

-- 设置表缓存
ALTER TABLE t SET CACHE BY (col1, col2);
ALTER TABLE t SET CACHE OFF;

-- 设置表在线/离线
ALTER TABLE t SET ONLINE;
ALTER TABLE t SET OFFLINE;

-- 重建堆
ALTER TABLE t REBUILD HEAP;
```

## 约束详解

### 主键约束

```sql
-- 内联
CREATE TABLE t (id INT PRIMARY KEY, name VARCHAR(50));

-- 外联
CREATE TABLE t (
    id INT,
    name VARCHAR(50),
    PRIMARY KEY (id)
);

-- ALTER 添加
ALTER TABLE t ADD CONSTRAINT pk_id PRIMARY KEY (id);
```

### 外键约束

```sql
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT CONSTRAINT fk_user REFERENCES users(id)
);
```

### 唯一约束

```sql
ALTER TABLE t ADD CONSTRAINT uk_email UNIQUE (email);
```

### 检查约束

```sql
ALTER TABLE t ADD CONSTRAINT ck_age CHECK (age >= 0 AND age <= 150);
```

### 约束管理

```sql
-- 查看约束
SELECT cons_name, cons_type, define, enable, valid
FROM SYS_CONSTRAINTS c
JOIN SYS_TABLES t ON c.table_id = t.table_id
WHERE t.table_name = 'TABLE_NAME';

-- 禁用/启用
ALTER TABLE t DISABLE CONSTRAINT cons_name;
ALTER TABLE t ENABLE CONSTRAINT cons_name;

-- 删除（保留或删除关联索引）
ALTER TABLE t DROP CONSTRAINT cons_name KEEP INDEX;
ALTER TABLE t DROP CONSTRAINT cons_name DROP INDEX;
```

## 删除表

```sql
-- 基本删除
DROP TABLE table_name;

-- 带 IF EXISTS
DROP TABLE IF EXISTS table_name;

-- CASCADE（级联删除依赖对象）
DROP TABLE table_name CASCADE;

-- CASCADE CONSTRAINTS（仅级联删除约束）
DROP TABLE table_name CASCADE CONSTRAINTS;

-- RESTRICT（有依赖时报错，默认）
DROP TABLE table_name RESTRICT;

-- PURGE（跳过回收站，永久删除）
DROP TABLE table_name PURGE;
```

## 截断表

```sql
-- 截断表（删除所有数据，不可回滚）
TRUNCATE TABLE table_name;

-- 带等待选项
TRUNCATE TABLE table_name NOWAIT;
TRUNCATE TABLE table_name WAIT 10;
```

## 锁表

```sql
-- 共享锁
LOCK TABLE t IN SHARE MODE;

-- 排他锁
LOCK TABLE t IN EXCLUSIVE MODE;

-- 行共享锁
LOCK TABLE t IN ROW SHARE MODE;

-- 行排他锁
LOCK TABLE t IN ROW EXCLUSIVE MODE;

-- 默认排他锁
LOCK TABLE t;

-- 带等待选项
LOCK TABLE t IN SHARE MODE NOWAIT;
LOCK TABLE t IN SHARE MODE WAIT 10;
```
