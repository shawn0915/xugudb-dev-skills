---
name: 虚谷数据库数据迁移
name_for_command: xugudb-migration
description: |
  虚谷数据库数据迁移完整指南：静态数据迁移（TXT/CSV/SQL/备份文件/实例目录/数据库）、
  对象数据复制（CREATE TABLE AS SELECT / INSERT INTO SELECT / IMPORT SELECT）、在线数据迁移。
---

# 虚谷数据库数据迁移

虚谷数据库支持多种数据迁移方式，覆盖静态迁移、对象复制和在线迁移场景。

## 静态数据迁移

将静态数据源一次性迁移到虚谷数据库，适用于历史表、归档表、旧版本数据等场景。

支持的数据源类型：
- **TXT 文件** — 通过 XGConsole 的 `sqlldr` / `tabldr` 命令导入
- **CSV 文件** — 启用 `csv_deal=true` 参数导入
- **SQL 文件** — 通过 `execfile` 命令执行
- **虚谷备份文件** — 物理备份（全量/增量）、逻辑备份（库/用户/模式/表级）
- **虚谷实例目录** — 直接复制部署目录（单机/集群）
- **数据库** — 同构（虚谷）或异构（Oracle/PostgreSQL/MySQL）通过中间格式迁移

> 详细参考：[static-migration](skills/xugudb-migration/references/static-migration.md)

## 对象数据复制

在同一数据库内复制表结构和数据，适用于数据重组、环境复制、数据归档等场景。

| 方式 | 语法 | 说明 |
|------|------|------|
| 创建并复制 | `CREATE TABLE ... AS SELECT` | 同时复制结构和数据，支持临时表 |
| 插入/替换复制 | `INSERT/REPLACE INTO ... SELECT` | 向已有表插入或替换数据 |
| 流式复制 | `IMPORT [APPEND\|REPLACE] TABLE ... FROM SELECT` | 大数据量高性能导入 |

> 详细参考：[object-replication](skills/xugudb-migration/references/object-replication.md)

## 在线数据迁移

在线数据迁移工具暂未正式发布。如需获取试用版，请通过虚谷官方渠道联系。

## 参考文档

- [静态数据迁移](skills/xugudb-migration/references/static-migration.md) — TXT/CSV/SQL/备份文件/实例目录/数据库迁移
- [对象数据复制](skills/xugudb-migration/references/object-replication.md) — CREATE AS SELECT / INSERT INTO SELECT / IMPORT SELECT
