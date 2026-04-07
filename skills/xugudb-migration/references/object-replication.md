# 对象数据复制

在同一数据库中复制"对象"（表）及其数据的过程，包含结构与数据的复制。

**常见应用场景：**
1. 数据表结构优化重组：旧表 → 新表
2. 环境复制：生产环境 → 测试/开发环境；主数据库 → 报表专用数据库
3. 数据归档：活动表 → 历史归档表、冷热数据分级存储

---

## 一、CREATE TABLE AS SELECT

根据查询结果创建新表并复制数据，可仅复制结构或同时复制结构与数据。

### 语法

```sql
CREATE [LOCAL|GLOBAL] [TEMPORARY|TEMP] TABLE [IF NOT EXISTS] [schema_name.]table_name
  [(column_name [, column_name ...])]
AS SELECT ...
```

### 参数

| 参数 | 说明 |
|------|------|
| 未指定 TEMPORARY | 创建为普通表 |
| `TEMPORARY` / `TEMP` / `LOCAL TEMPORARY` / `LOCAL TEMP` | 局部临时表，会话结束时删除结构与数据 |
| `GLOBAL TEMPORARY` / `GLOBAL TEMP` | 全局临时表，会话结束时保留结构、仅删除数据 |
| `column_name` | 可选列名，未指定时使用查询中的列名 |

### 示例

```sql
-- 创建基础表
CREATE TABLE tab_base(id INT, str VARCHAR(20));
INSERT INTO tab_base VALUES(1, 'VALUE BASE');

-- 同时复制表结构和数据
CREATE TABLE tab_copy1 AS SELECT * FROM tab_base;

-- 仅复制表结构（不含数据）
CREATE TABLE tab_copy2 AS SELECT * FROM tab_base WHERE 1=0;
```

---

## 二、INSERT/REPLACE INTO SELECT

将查询结果插入或替换到已有表中。

### 语法

```sql
{INSERT | REPLACE} INTO [schema_name.]table_name [(column_list)]
SELECT ...
```

### 参数

| 参数 | 说明 |
|------|------|
| `INSERT` | 向表中插入数据，不影响原有数据 |
| `REPLACE` | 替换表中数据（遇唯一值冲突时先删除原记录再插入） |
| `column_list` | 可选，仅插入指定列，未指定则插入所有列 |

### 示例

```sql
CREATE TABLE tab_copy3(col1 INT, col2 VARCHAR);
INSERT INTO tab_copy3 VALUES(1,'VALUE1')(2,'VALUE2');

CREATE TABLE tab_copy4(col1 INT, col2 VARCHAR);
CREATE TABLE tab_copy5(col1 INT PRIMARY KEY, col2 VARCHAR);
INSERT INTO tab_copy5 VALUES(5,'VALUE5');

-- INSERT INTO SELECT：复制数据
INSERT INTO tab_copy4 SELECT * FROM tab_copy3;

-- INSERT INTO SELECT：遇唯一值约束冲突则报错
INSERT INTO tab_copy5(col1, col2) SELECT * FROM tab_copy3;
-- 如果主键冲突：Error: [E13001] 违反唯一值约束

-- REPLACE INTO SELECT：遇唯一值约束冲突则先删后插
UPDATE tab_copy3 SET col2='VALUE22' WHERE col1=2;
REPLACE INTO tab_copy5(col1, col2) SELECT * FROM tab_copy3;
-- 结果：col1=2 的记录更新为 VALUE22
```

> **注意：**
> - `INSERT INTO` 遇唯一值约束冲突时，插入失败
> - `REPLACE INTO` 遇唯一值约束冲突时，先删除原记录再插入，操作成功

---

## 三、IMPORT SELECT

将查询结果通过流式导入到已有表中。相较于 INSERT SELECT，省去反复解析 SQL 语句的消耗，**性能更好，适用于大数据量表**。

### 语法

```sql
IMPORT [APPEND|REPLACE] TABLE [schema_name.]table_name [(column_list)]
FROM SELECT ...
```

### 参数

| 参数 | 说明 |
|------|------|
| 未指定 / `APPEND` | 结果追加到目标表中，不影响已有数据（默认） |
| `REPLACE` | 替换表中所有数据（先删除全部原数据） |
| `column_list` | 可选列名别名，字段数须与 SELECT 中一致 |

### 示例

```sql
CREATE TABLE tab_copy6(id INT, name VARCHAR(20));
INSERT INTO tab_copy6 VALUES(1,'one')(2,'two');

CREATE TABLE tab_copy7(id INT PRIMARY KEY, name VARCHAR(20));
INSERT INTO tab_copy7 VALUES(66,'abc');

-- APPEND 模式：追加数据（即使主键冲突也追加成功）
IMPORT APPEND TABLE tab_copy7 FROM SELECT * FROM tab_copy6;

-- REPLACE 模式：替换全部数据
IMPORT REPLACE TABLE tab_copy7 FROM SELECT * FROM tab_copy6;
-- 结果：表中仅剩 (1,'one') 和 (2,'two')
```

> **注意：**
> - `IMPORT APPEND`：追加数据，即使主键冲突也追加成功
> - `IMPORT REPLACE`：替换所有数据，会先删除表中原本所有数据
> - `IMPORT` 不支持目标表出现在查询子句中
