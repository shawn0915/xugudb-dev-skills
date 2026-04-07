# 回收站管理

## 概述

回收站是所有被删除表及其相关依赖对象的逻辑存储容器。当表被意外删除时，可通过回收站恢复到原始状态，无需从备份恢复。

## 一、控制回收站

回收站默认关闭，通过 `enable_recycle` 参数控制（全局参数，立即生效）。

```sql
-- 查看回收站状态
SHOW enable_recycle;

-- 开启回收站
SET enable_recycle ON;

-- 关闭回收站
SET enable_recycle OFF;
```

也可在 `xugu.ini` 中设置 `enable_recycle = true` 在启动时开启。

## 二、删表与回收站

### 语法

```sql
DROP TABLE [IF EXISTS] [schema.]tab_name [RESTRICT | CASCADE] [PURGE];
```

### 行为

| 操作 | 行为 |
|------|------|
| `DROP TABLE tab` | 逻辑删除，表及非外键约束、索引、触发器进入回收站 |
| `DROP TABLE tab PURGE` | 直接物理删除，不进入回收站 |
| `DROP TABLE tab CASCADE` | 强制删除（存在依赖时） |

> **注意**：外键约束会直接删除，不进入回收站。

## 三、查询回收站

### 查询方式

| 对象 | 范围 |
|------|------|
| `SYS_RECYCLEBIN`（系统表） | 所有库中被删除对象 |
| `DBA_RECYCLEBIN`（系统视图） | 当前库所有用户的回收站对象 |
| `USER_RECYCLEBIN`（系统视图） | 当前用户的回收站对象 |
| `RECYCLEBIN`（公共同义词） | 等同 USER_RECYCLEBIN |

回收站中的对象名以 `BIN$` 为前缀。

```sql
-- 查询回收站
SELECT * FROM RECYCLEBIN;
SELECT * FROM DBA_RECYCLEBIN;

-- 查询回收站中的表数据（仅支持 SELECT）
SELECT * FROM "BIN$0800000007";
```

> **注意**：回收站表仅支持 SELECT 查询，不支持其他 DML/DDL 操作。

## 四、恢复表

### 语法

```sql
FLASHBACK TABLE tab_name TO BEFORE DROP [RENAME TO new_name];
```

### 说明

- `tab_name`：原表名或回收站中的表名（`BIN$...`）
- 同名表存在多个时，恢复最迟进入回收站的
- 非外键约束随表自动恢复
- 可通过 `RENAME TO` 避免恢复后重名

### 示例

```sql
-- 按原表名恢复
FLASHBACK TABLE tab_recyc TO BEFORE DROP;

-- 恢复并改名
FLASHBACK TABLE tab_recyc TO BEFORE DROP RENAME TO tab_recyc_new;
```

## 五、清理回收站

### 5.1 清理表

清理表时相关的索引、触发器同时清理。同名表清理最早进入回收站的。

```sql
-- 按原表名清理
PURGE TABLE tab_recyc;

-- 按回收站名清理
PURGE TABLE "BIN$0800000007";
```

### 5.2 清理索引

不支持单独清理主键索引，只能跟随表清理。

```sql
PURGE INDEX tab_name.index_name;
```

### 5.3 清理用户/模式回收站

```sql
-- 清理当前用户下所有回收站对象
PURGE RECYCLEBIN;

-- 清理指定用户的回收站对象（需 DBA 权限）
PURGE USER user_name;
```

### 5.4 其他清理触发

- `DROP DATABASE`：清理该库下所有回收站对象
- `DROP USER ... CASCADE`：清理该用户下所有回收站对象
- `DROP SCHEMA ... CASCADE`：清理该模式下所有回收站对象
