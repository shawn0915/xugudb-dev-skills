# 索引管理

## 概述

索引用于加速 WHERE、JOIN、IN、EXISTS 等查询条件的匹配。虚谷数据库使用 B+ 树作为默认索引结构。

## 创建索引

```sql
CREATE [UNIQUE] INDEX [IF NOT EXISTS] index_name
ON [schema.]table_name (index_keys)
[index_type]
[partition_clause]
[ONLINE]
[PARALLEL n]
[NOWAIT | WAIT n];
```

### 索引键类型

```sql
-- 单列索引
CREATE INDEX idx_name ON t (name);

-- 复合索引
CREATE INDEX idx_name_age ON t (name, age);

-- 降序索引
CREATE INDEX idx_date_desc ON t (create_date DESC);

-- 函数索引
CREATE INDEX idx_upper_name ON t (UPPER(name));

-- 唯一索引
CREATE UNIQUE INDEX idx_unique_email ON t (email);
```

### IF NOT EXISTS

```sql
-- 如果索引已存在不报错
CREATE INDEX IF NOT EXISTS idx_name ON t (name);
```

### 示例

```sql
-- 创建表
CREATE TABLE employees (id INT, name VARCHAR(100), birth DATE);

-- 唯一索引（约束唯一性）
CREATE UNIQUE INDEX idx_emp_id ON employees (id);
INSERT INTO employees VALUES (1, 'Tom', '2000-01-01');
INSERT INTO employees VALUES (1, 'Jerry', '2001-01-01');  -- Error: [E13001]

-- 普通索引
CREATE INDEX idx_emp_name ON employees (name);

-- 复合索引
CREATE INDEX idx_emp_name_birth ON employees (name, birth);
```

## 删除索引

```sql
DROP INDEX index_name;
DROP INDEX IF EXISTS index_name;
```

## 索引选择建议

| 场景 | 建议 |
|------|------|
| WHERE 等值查询 | 单列索引 |
| WHERE 多列组合 | 复合索引（高选择性列在前） |
| ORDER BY / GROUP BY | 与排序列匹配的索引 |
| 唯一性约束 | UNIQUE 索引 |
| 函数条件（如 UPPER） | 函数索引 |
| 大量写入的表 | 减少索引数量 |
| 覆盖查询 | 包含所有查询列的复合索引 |

## 注意事项

- 索引会占用存储空间并影响写入性能
- 每个表的索引数量应控制在合理范围内
- 主键自动创建唯一索引
- UNIQUE 约束自动创建唯一索引
- 删除约束时可选择保留或删除关联索引（KEEP INDEX / DROP INDEX）
