---
name: 虚谷数据库安全权限管理
name_for_command: xugudb-security
description: |
  虚谷数据库安全权限完整参考：身份鉴别认证、权限管理（系统/数据库/模式/对象级）、
  角色管理、安全策略（行级访问控制）、黑白名单、数据加密（传输/透明）、安全审计。
---

# 虚谷数据库安全权限管理

虚谷数据库提供完整的安全管理体系，包括身份认证、访问控制、数据加密和安全审计四大模块。

## 身份鉴别与认证

基于口令的强身份认证机制，支持多因素安全验证：口令复杂度校验、登录 IP 校验、用户有效期校验。

- 口令策略参数：`min_pass_len`（最小长度）、`pass_mode`（口令模式）、`conn_fail_cnt`（失败锁定次数）等
- 认证时机：登录、切换数据库、断连自动重连
- 失败处理：默认 3 次失败后锁定 IP 3 分钟
- 口令存储：加密存储在系统表 `SYS_USERS` 中

> 详细参考：[authentication](skills/xugudb-security/references/authentication.md)

## 访问控制

### 权限管理

按对象粒度分为四级权限体系：

| 权限级别 | 说明 |
|---------|------|
| 系统级权限 | CREATE/ALTER/DROP ANY DATABASE, BACKUP/RESTORE 等 |
| 数据库级权限 | CREATE/ALTER/DROP/SELECT/INSERT/UPDATE/DELETE ANY + 对象类型 |
| 模式级权限 | ... ANY ... IN SCHEMA schema_name |
| 对象级权限 | ... ON 具体对象（含列级权限） |

管理员权限：DBA（数据库管理）、AUDITOR（审计）、SSO（安全）三权分立。

### 安全策略（行级访问控制 MAC）

基于安全标记的强制访问控制，通过安全等级（0-30000）和安全范畴（最多48个）实现细粒度行级控制。

### 黑白名单

双重防护机制，支持配置文件（`trust.ini`）和数据库命令两种方式管理。

> 详细参考：[access-control](skills/xugudb-security/references/access-control.md)

## 数据加密

| 加密类型 | 说明 |
|---------|------|
| 数据传输加密 | 驱动层 SSL 加密，需安装 `xgssl.dll` / `libxgssl.so` |
| 透明数据加密（TDE） | 静态数据文件加密，支持库级/用户级/表级，优先级：表 > 用户 > 库 |

> 详细参考：[encryption](skills/xugudb-security/references/encryption.md)

## 安全审计

通过定义系统级、语句级、对象级审计项，监控用户操作行为。

- 审计开关：`SET enable_audit ON/OFF`
- 三级审计：系统级（自动）、语句级、对象级
- 审计结果表：`SYSAUDITOR.SYS_AUDIT_RESULTS`（自动扩展分区表，7天一分区）
- 审计角色：SYSAUDITOR（审计管理员）管理审计设置权

> 详细参考：[audit](skills/xugudb-security/references/audit.md)

## 参考文档

- [身份鉴别认证](skills/xugudb-security/references/authentication.md) — 口令策略、认证机制、失败处理
- [访问控制](skills/xugudb-security/references/access-control.md) — 权限管理、安全策略、角色、黑白名单
- [数据加密](skills/xugudb-security/references/encryption.md) — 传输加密、透明数据加密（TDE）
- [安全审计](skills/xugudb-security/references/audit.md) — 审计开关、审计项定义、审计查询、日志维护
