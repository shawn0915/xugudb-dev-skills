---
name: xugudb
name_for_command: xugudb
description: |
  虚谷数据库（XuguDB）开发技能包。适用场景：虚谷数据库的SQL开发、PL/SQL编程、分布式集群管理、数据迁移（Oracle/MySQL/PG）、向量检索、空间数据库、多语言驱动开发（PHP/Node.js/C/ODBC）。
---

# 虚谷数据库（XuguDB）技能包

虚谷数据库是成都虚谷伟业科技有限公司自主原创的高性能关系型数据库，基于20多年数据库研发经验，核心代码自有率99.6%。

## 技能模块索引

### 📋 基础开发
| 命令 | 说明 | 参考文档 |
|------|------|:--------:|
| `/xugudb` | 产品概览、架构设计、版本选型 | 5 |
| `/xugudb-sql` | DDL/DML/查询/数据类型/运算符 | 8 |
| `/xugudb-plsql` | 存储过程、函数、触发器、游标 | 1 |
| `/xugudb-functions` | 字符串/数学/日期/JSON/XML等21类函数 | 6 |

### ⚙️ 配置运维
| 命令 | 说明 | 参考文档 |
|------|------|:--------:|
| `/xugudb-objects` | 表/索引/视图/约束/DBLink等对象管理 | 5 |
| `/xugudb-config` | xugu.ini参数、集群配置 | 4 |
| `/xugudb-data-dictionary` | 系统表、系统视图、系统包 | 1 |
| `/xugudb-security` | 认证、权限、角色、审计、加密 | 4 |
| `/xugudb-deployment` | 标准版/企业版/分布式版/安全版/Docker | 6 |
| `/xugudb-ha` | 集群管理、备份恢复、故障切换 | 5 |

### 🌐 高级特性
| 命令 | 说明 | 参考文档 |
|------|------|:--------:|
| `/xugudb-distributed` | 分布式架构、节点角色(M/QW/S/G)、存算融合 | 2 |
| `/xugudb-migration` | Oracle/MySQL/PG迁移到虚谷 | 2 |
| `/xugudb-vector` | VECTOR/HALFVEC/SPARSEVEC、DiskANN索引 | 4 |
| `/xugudb-spatial` | GIS几何模型、300+空间函数、地图服务集成 | 4 |

### 💻 驱动开发
| 命令 | 说明 | 参考文档 |
|------|------|:--------:|
| `/xugudb-jdbc` | Java连接池、CRUD、事务、批量操作、SSL | 5 |
| `/xugudb-python` | xgcondb驱动、参数化查询、游标 | 4 |
| `/xugudb-go` | database/sql接口、事务、大对象 | 3 |
| `/xugudb-csharp` | ADO.NET、XGConnection、DataSet | 3 |
| `/xugudb-php` | PDO驱动、参数绑定、存储过程 | 2 |
| `/xugudb-nodejs` | ODBC桥接、Sequelize ORM集成 | 2 |
| `/xugudb-odbc` | DSN配置、连接字符串、跨语言访问 | 2 |
| `/xugudb-c` | NCI/OCI/XGCI三种C接口 | 3 |

### 🔗 生态集成
| 命令 | 说明 | 参考文档 |
|------|------|:--------:|
| `/xugudb-ecosystem` | MyBatis/Hibernate/GORM/Django等ORM集成 | 2 |
| `/xugudb-tools` | XGConsole命令行、DBeaver集成 | 2 |
| `/xugudb-faq` | FAQ + 完整错误码参考 | 4 |

---

## 快速参考

### 产品定位
- **国产自主可控**：通过EAL4+安全认证、国密算法、涉密认证
- **原生分布式**：Shared-Nothing架构，支持1~512节点
- **多模兼容**：兼容Oracle/MySQL/PostgreSQL语法
- **HTAP混合负载**：同时支持OLTP和OLAP场景

### 架构特点
虚谷采用**单进程多线程 + Shared-Nothing**架构，集群由四类角色组成：

| 角色 | 职责 |
|------|------|
| 主控管理(M) | 集群管理、心跳检测、全局锁仲裁 |
| 工作计算(QW) | 接收用户请求、SQL解析执行、结果返回 |
| 存储管理(S) | 数据持久化、8MB CHUNK自动切片、多副本 |
| 变更收集(G) | 事务级变更信息收集（可选部署） |

### 常用指标

| 指标 | 限制 |
|------|------|
| 最大节点数 | 512 |
| 单节点连接数 | 10,000 |
| 单库表数量 | 21亿 |
| 字段数 | 2,040 |
| 记录长度 | 64KB |
| JOIN表数 | 64 |
| 大对象 | 2GB |
| SQL长度 | 2MB |

### 与Oracle/MySQL/PG关键差异

| 特性 | XuguDB特点 | 迁移注意 |
|------|-------------|----------|
| 分布式架构 | 原生Shared-Nothing | 无需额外中间件 |
| 自动分片 | 物理CHUNK自动切片，无需分片键 | 无需设计分片策略 |
| 锁机制 | 行级锁，不允许锁升级 | 无表锁升级风险 |
| 隔离级别 | 基于MVCC的Read Committed | 与Oracle默认级别一致 |
| PL/SQL | 兼容Oracle PL/SQL | 存储过程可较低成本迁移 |

---

## 详细文档索引

按需加载以下子技能获取详细参考：

**基础开发：**
- [xugudb](skills/xugudb/SKILL.md) — 产品简介、架构、版本、核心技术
- [xugudb-sql](skills/xugudb-sql/SKILL.md) — 完整SQL语法参考
- [xugudb-plsql](skills/xugudb-plsql/SKILL.md) — PL/SQL编程指南
- [xugudb-functions](skills/xugudb-functions/SKILL.md) — 21类系统函数

**配置运维：**
- [xugudb-objects](skills/xugudb-objects/SKILL.md) — 数据库对象管理
- [xugudb-config](skills/xugudb-config/SKILL.md) — 系统配置与调优
- [xugudb-data-dictionary](skills/xugudb-data-dictionary/SKILL.md) — 数据字典
- [xugudb-security](skills/xugudb-security/SKILL.md) — 安全与权限管理
- [xugudb-deployment](skills/xugudb-deployment/SKILL.md) — 安装部署指南
- [xugudb-ha](skills/xugudb-ha/SKILL.md) — 高可用与备份恢复

**高级特性：**
- [xugudb-distributed](skills/xugudb-distributed/SKILL.md) — 分布式集群部署
- [xugudb-migration](skills/xugudb-migration/SKILL.md) — 异构数据库迁移
- [xugudb-vector](skills/xugudb-vector/SKILL.md) — AI向量检索功能
- [xugudb-spatial](skills/xugudb-spatial/SKILL.md) — GIS空间数据库

**驱动开发：**
- [xugudb-jdbc](skills/xugudb-jdbc/SKILL.md) — Java数据库开发
- [xugudb-python](skills/xugudb-python/SKILL.md) — Python数据库开发
- [xugudb-go](skills/xugudb-go/SKILL.md) — Go数据库开发
- [xugudb-csharp](skills/xugudb-csharp/SKILL.md) — C#数据库开发
- [xugudb-php](skills/xugudb-php/SKILL.md) — PHP数据库开发
- [xugudb-nodejs](skills/xugudb-nodejs/SKILL.md) — Node.js数据库开发
- [xugudb-odbc](skills/xugudb-odbc/SKILL.md) — ODBC数据库访问
- [xugudb-c](skills/xugudb-c/SKILL.md) — C语言数据库开发

**生态集成：**
- [xugudb-ecosystem](skills/xugudb-ecosystem/SKILL.md) — ORM框架集成
- [xugudb-tools](skills/xugudb-tools/SKILL.md) — 工具与集成环境
- [xugudb-faq](skills/xugudb-faq/SKILL.md) — 常见问题与错误码