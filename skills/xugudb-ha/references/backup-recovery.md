# 备份恢复

## 概述

虚谷数据库提供两种备份恢复方案：

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| **物理备份**（系统级） | 直接备份数据库二进制文件 | 大型数据库快速恢复、高安全需求 |
| **逻辑备份** | 对特定数据库对象备份 | 小量数据迁移、跨平台恢复、特定对象备份 |

### 备份策略

- **全量备份**：完整备份所有数据，增量备份的基础
- **增量备份**：仅备份上次全量/增量以来修改的文件，数据量小、速度快

---

## 一、物理备份恢复（系统级）

### 1.1 定义

物理备份直接复制数据库的二进制文件（数据文件、重做日志文件等），以磁盘块为单位复制。备份文件包含所有库及库中的所有数据。

**权限要求**：必须由 SYSDBA 登录系统库 SYSTEM 执行。

### 1.2 备份能力

| 能力 | 支持情况 |
|------|----------|
| 全量备份 | 支持 |
| 增量备份 | 支持 |
| 单机备份 | 支持 |
| 集群备份 | 3节点及以下支持 |
| 按时间恢复 | 支持（需开启归档） |

### 1.3 物理备份语法

```sql
BACKUP SYSTEM [ALL | INCREMENT] [APPEND] TO backup_file
    [ONLINE | OFFLINE]
    [ENCRYPTOR IS 'key']
    [COMPRESS | NOCOMPRESS];
```

**参数说明**：

| 参数 | 说明 | 默认值 |
|------|------|--------|
| ALL / INCREMENT | ALL 新建备份，INCREMENT 增量备份 | ALL |
| APPEND | 追加到已有备份文件 | - |
| backup_file | 备份文件路径，如 `/BACKUP/SYS.DMP` | - |
| ONLINE / OFFLINE | OFFLINE 会断开除当前外所有连接 | OFFLINE |
| ENCRYPTOR IS | 加密密钥 | - |
| COMPRESS / NOCOMPRESS | 是否压缩 | COMPRESS |

### 1.4 物理备份示例

```sql
-- 全量备份
BACKUP SYSTEM TO '/BACKUP/SYS.DMP';

-- 增量备份（新建文件）
BACKUP SYSTEM INCREMENT TO '/BACKUP/SYS1.DMP';

-- 增量备份（追加到全量文件）
BACKUP SYSTEM INCREMENT APPEND TO '/BACKUP/SYS.DMP';

-- 在线加密压缩备份
BACKUP SYSTEM TO '/BACKUP/SYS.DMP' ONLINE ENCRYPTOR IS 'BACKUP_ENCRY' COMPRESS;
```

### 1.5 物理备份注意事项

1. 备份与恢复时的操作系统和数据库版本必须保持一致
2. 异常中断后不能基于异常文件增量备份，需删除后重新全量备份
3. 必须保证备份磁盘有足够空间
4. 冷备份须在停止服务后进行
5. 多节点全量/增量备份必须在主节点执行，集群节点数不超过3个

### 1.6 物理恢复语法

```sql
RESTORE SYSTEM FROM backup_file
    [ENCRYPTOR IS 'key']
    [UNTIL CURRENT TIME | UNTIL TIME 'timestamp'];
```

### 1.7 物理恢复示例

```sql
-- 系统级恢复
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP';

-- 非追加方式增量恢复（先全量再增量）
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP';
RESTORE SYSTEM FROM '/BACKUP/SYS1.DMP';

-- 追加方式恢复（所有数据在一个文件中）
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP';

-- 解密恢复
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP' ENCRYPTOR IS 'BACKUP_ENCRY';

-- 按时间点恢复（需开启归档 log_archive_mode > 0）
RESTORE SYSTEM FROM '/BACKUP/SYS.DMP' UNTIL TIME '9999-12-31 23:59:59.999';
```

### 1.8 物理恢复注意事项

1. 集群恢复：节点数不超过3个，非活跃主节点须全部离线，恢复完成后服务立即停止，重启集群自动同步
2. 按时间点恢复：须开启归档（`SET log_archive_mode TO 1` 或 `2`），须将未归档 redo 日志拷贝到 ARCH 目录

---

## 二、逻辑备份恢复

逻辑备份支持库级、用户级、模式级、表级四种层级，不受单机/集群限制。

### 2.1 备份能力矩阵

| 能力 | 库级 | 用户级 | 模式级 | 表级 |
|------|------|--------|--------|------|
| 全量备份 | 支持 | 支持 | 支持 | 支持 |
| 增量备份 | 不支持 | 不支持 | 不支持 | 不支持 |
| 库级恢复 | 支持 | - | - | - |
| 用户级恢复 | 支持 | 支持 | - | - |
| 模式级恢复 | 支持 | 支持 | 支持 | - |
| 表级恢复 | 支持 | 支持 | 支持 | 支持 |

### 2.2 库级逻辑备份

```sql
-- 备份（SYSDBA 登录目标库）
BACKUP DATABASE TO '/BACKUP/DB.EXP' [ENCRYPTOR IS 'key'] [COMPRESS];

-- 恢复（须在空库中执行）
RESTORE DATABASE db_name FROM '/BACKUP/DB.EXP'
    [ENCRYPTOR IS 'key'] [WITH REINDEX];
```

**注意**：
- 库级恢复不支持在非空库下操作，需先创建新库
- 恢复目标库和备份库字符集必须一致
- `WITH REINDEX` 可在恢复时重建索引，数据量大时可之后手动重建
- 若 `security_level > 0`，创建库时会产生审计表导致库非空，需特殊处理

### 2.3 用户级逻辑备份

```sql
-- 备份（SYSDBA 登录用户所在库）
BACKUP USER user_name TO '/BACKUP/USR.EXP' [ENCRYPTOR IS 'key'] [COMPRESS];

-- 恢复
RESTORE USER user_name FROM '/BACKUP/USR.EXP' [ENCRYPTOR IS 'key'];
```

**注意**：
- 仅备份用户下的表对象，其他对象和表依赖不备份
- 恢复时默认重建索引
- 恢复时对应模式不存在会创建同名模式

### 2.4 模式级逻辑备份

```sql
-- 备份（SYSDBA 登录模式所在库）
BACKUP SCHEMA schema_name TO '/BACKUP/SCH.EXP' [ENCRYPTOR IS 'key'] [COMPRESS];

-- 恢复（支持改名）
RESTORE SCHEMA schema_name [RENAME TO new_name] FROM '/BACKUP/SCH.EXP'
    [ENCRYPTOR IS 'key'];
```

**注意**：
- 仅备份表对象
- 恢复时 `schema_name` 必须与备份中的模式名一致
- `RENAME TO` 可指定新模式名，不存在则自动创建

### 2.5 表级逻辑备份

```sql
-- 备份（支持追加多表到同一文件）
BACKUP TABLE [schema.]tab_name [APPEND] TO '/BACKUP/TB.EXP'
    [ENCRYPTOR IS 'key'] [COMPRESS];

-- 恢复（支持改名）
RESTORE TABLE [schema.]tab_name [RENAME TO [schema.]new_name]
    FROM '/BACKUP/TB.EXP' [ENCRYPTOR IS 'key'];
```

**注意**：
- 备份包含表的结构（字段、约束、索引）和数据
- 恢复时允许目标表已存在，但表结构须完全一致
- 恢复时默认重建索引

---

## 三、备份恢复对比

| 维度 | 物理备份 | 逻辑备份 |
|------|----------|----------|
| 粒度 | 整个系统 | 库/用户/模式/表 |
| 速度 | 快（直接复制文件） | 较慢（解析对象） |
| 集群支持 | ≤3节点 | 不限 |
| 跨平台 | 不支持 | 支持 |
| 跨版本 | 不支持 | 支持 |
| 增量备份 | 支持 | 不支持 |
| 按时间恢复 | 支持 | 不支持 |
| 推荐场景 | 大型数据库容灾 | 小量数据迁移（<5GB） |
