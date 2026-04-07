# 安全审计

虚谷数据库提供数据库内部审计机制，通过定义系统级、语句级、对象级审计项，监控用户操作行为。

---

## 一、审计概述

### 审计方式
通过监控、记录和分析数据库操作行为，实现安全合规、风险追踪与责任追溯。

### 审计价值
1. 安全事件（数据篡改、泄露）发生后提供追责依据
2. 针对风险行为进行实时告警

### 审计术语

| 术语 | 说明 |
|------|------|
| 审计管理员（SYSAUDITOR） | 负责审计设置权的管理和维护 |
| 审计员 | 具备审计权的普通用户 |
| 审计权 | 可定义/删除审计项的权限 |
| 审计项 | 审计的类目 |

---

## 二、审计功能管理

### 2.1 审计开关

通过系统参数 `enable_audit` 控制：

```sql
-- 开启审计
SQL> SET enable_audit ON;

-- 关闭审计
SQL> SET enable_audit OFF;

-- 查看审计开关状态（T：开启，F：关闭（默认））
SQL> SHOW enable_audit;
```

### 2.2 授予/回收审计设置权

```sql
-- 授予普通用户审计设置权（SYSAUDITOR 执行）
SQL> GRANT AUDITOR TO u1;

-- 回收审计设置权
SQL> REVOKE AUDITOR FROM u1;
```

查询用户权限：`SELECT * FROM SYSDBA.SYS_ACLS WHERE grantee_id = <user_id>;`
（AUTHORITY & 0x10000000 = 268435456 表示具有审计设置权）

> **注意：** 审计员只有定义审计项的权限，访问审计结果表需审计管理员单独授权。

### 2.3 审计项定义

```sql
AuditStmt ::= AuditOper AuditItems [BY 'UserName'] [WHENEVER [NO] SUCCESSFUL]
```

- `AUDIT`：定义审计项
- `NOAUDIT`：取消审计项
- `BY 'UserName'`：指定用户，省略则针对当前库所有用户
- `WHENEVER SUCCESSFUL`：仅审计成功操作
- `WHENEVER NO SUCCESSFUL`：仅审计失败操作

### 2.4 审计记载

审计记录写入 `SYSAUDITOR.SYS_AUDIT_RESULTS`（自动扩展分区表，分区间隔 7 天）。该表在定义审计项时由系统自动创建。

---

## 三、审计范围设置

### 3.1 系统级审计

无需定义审计项，系统自动具备：

| 审计项 | 说明 |
|--------|------|
| CONNECT | 对所有客户端连接请求审计（`security_level > 0`） |

### 3.2 语句级审计（全局）

```sql
-- 审计所有用户对任何表的 UPDATE 操作
SQL> AUDIT UPDATE TABLE WHENEVER SUCCESSFUL;

-- 取消审计
SQL> NOAUDIT UPDATE TABLE WHENEVER SUCCESSFUL;
```

**支持的语句级审计项：**
DATABASE, STORAGE ZONE, LOGIN, CONNECT, USER, ROLE, GRANT, REVOKE, AUDIT, NOAUDIT, SCHEMA, TABLE, VIEW, INDEX, PROCEDURE, PACKAGE, TYPE, DATABASE LINK, SYNONYM, TRIGGER, SEQUENCE, TABLESPACE, POLICY, USER POLICY, TABLE POLICY, INSERT TABLE, INSERT ANY TABLE, UPDATE TABLE, UPDATE ANY TABLE, DELETE TABLE, DELETE ANY TABLE, SELECT TABLE, SELECT ANY TABLE, LOCK TABLE, LOCK ANY TABLE, EXECUTE PROCEDURE, EXECUTE ANY PROCEDURE

### 3.3 对象级审计（局部）

```sql
-- 审计用户 u2 对 u2.t1 表的 DELETE 操作
SQL> AUDIT DELETE ON u2.t1 BY u2 WHENEVER SUCCESSFUL;

-- 审计对象的所有操作
SQL> AUDIT ALL ON table_name;
```

支持的对象级审计操作：INSERT, UPDATE, DELETE, SELECT, EXECUTE

---

## 四、审计信息查询

### 4.1 审计项查询

```sql
-- 查看已定义的审计项（SYSDBA 执行）
SQL> SELECT * FROM SYSDBA.SYS_AUDIT_DEFS;
```

> **注意：** SYSAUDITOR 无法直接查看审计项，只能由 SYSDBA 查看。

### 4.2 审计内容查询

```sql
SQL> SELECT * FROM SYSAUDITOR.SYS_AUDIT_RESULTS;
```

OBJ_TYPE 字段取值：库(1)、表空间(2)、模式(4)、表(5)、过程/函数(7)、序列值(8)、视图(9)、索引(10)、触发器(11)、数据库LINK(12)、同义词(15)、用户(16)、角色(17)、包(18)、自定义类型(19)、安全策略(23)、用户策略(24)、表策略(25)、审计项(26)、公共同义词(35)、存储域(36)

---

## 五、审计信息安全

### 5.1 审计数据高可用
审计结果表是堆表，集群模式下最多支持三个存储副本。

### 5.2 审计数据备份

```sql
SQL> SELECT * FROM sys_audit_results; >$ /home/DBMS/out_audit.exp;
```

---

## 六、审计日志维护

审计管理员对审计结果表的操作权限：

| 操作 | 审计员 | 审计管理员 |
|------|--------|-----------|
| CREATE | × | √ |
| SELECT | 需授权 | √ |
| INSERT/UPDATE/DELETE | × | × |
| TRUNCATE | × | √ |
| DROP | × | × |
| DROP PARTITION | × | √（除初始化分区 PINIT 外） |
