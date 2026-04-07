# 访问控制

虚谷数据库通过权限管理、安全策略和黑白名单三重机制实现访问控制。

---

## 一、权限管理

### 1.1 权限分级

**按对象粒度分为四级：**

#### 系统级权限

控制系统范围操作（DATABASE、DIR 仅支持系统级权限控制）：

| 权限 | 说明 |
|------|------|
| `CREATE/ALTER/DROP ANY DATABASE` | 数据库管理 |
| `BACKUP DATABASE` / `BACKUP` | 备份 |
| `RESTORE DATABASE` / `RESTORE` | 恢复 |
| `CREATE/DROP ANY DIR` | 目录管理 |

#### 数据库级权限

格式：`操作 ANY 对象类型`

支持的对象类型：SCHEMA, TABLE, INDEX, VIEW, TRIGGER, PROCEDURE, PACKAGE, JOB, SEQUENCE, DATABASE LINK, SYNONYM, PUBLIC SYNONYM, USER, ROLE, OBJECT, REPLICATION

操作类型：CREATE, ALTER, DROP, SELECT, INSERT, UPDATE, DELETE, EXECUTE, REFERENCES, ENCRYPT

#### 模式级权限

格式：`操作 ANY 对象类型 IN SCHEMA schema_name` 或 `CREATE 对象类型`（默认模式）

支持的对象类型：TABLE, INDEX, VIEW, TRIGGER, PROCEDURE, PACKAGE, SEQUENCE, SYNONYM, OBJECT

> **注意：** JOB, DATABASE LINK, PUBLIC SYNONYM, USER, ROLE, REPLICATION 不支持模式级权限。

#### 对象级权限

格式：`操作 ON [TABLE|VIEW|PROCEDURE|SEQUENCE] 对象名`

支持的操作：ALL/ALL PRIVILEGES, ALTER, DROP, SELECT, INSERT, UPDATE, DELETE, EXECUTE, REFERENCES, INDEX, TRIGGER

**列级权限：** `SELECT/UPDATE (column_list) ON [TABLE] 表名`

### 1.2 管理员权限（三权分立）

| 权限 | 内置角色 | 职责 |
|------|---------|------|
| `DBA` | `DB_ADMIN` | 除审计和安全外的所有权限 |
| `AUDITOR` | `DB_AUDIT_OPER` | 审计权限 |
| `SSO` | `DB_POLICY_OPER` | 安全权限 |

> **注意：** 用户和角色不能同时拥有 DBA、AUDITOR、SSO 中的两项及以上。

### 1.3 权限授予与撤销

**系统级权限：**
```sql
GRANT CREATE ANY DATABASE, BACKUP, CREATE ANY DIR TO usr_test;
REVOKE CREATE ANY DATABASE FROM usr_test;
```

**数据库级权限：**
```sql
GRANT CREATE ANY PACKAGE, ALTER ANY PACKAGE, EXECUTE ANY PACKAGE TO usr_test;
REVOKE CREATE ANY PACKAGE, ALTER ANY PACKAGE FROM usr_test;
```

**模式级权限：**
```sql
-- 指定模式
GRANT CREATE ANY TABLE, SELECT ANY TABLE IN SCHEMA SYSDBA TO role_test;
-- 默认模式
GRANT CREATE TABLE TO usr_test;
```

**对象级权限：**
```sql
-- 表对象
GRANT ALL PRIVILEGES ON TABLE tab_test TO usr_test;
-- 列级权限
GRANT SELECT, UPDATE (a) ON tab_test TO usr_test;
-- 带转授权
GRANT SELECT ON tab_test TO usr_test WITH GRANT OPTION;
-- 撤销（CASCADE 级联撤销转授的权限）
REVOKE SELECT ON tab_test FROM usr_test CASCADE;
-- 仅撤销转授权
REVOKE GRANT OPTION FOR SELECT ON tab_test FROM usr_test;
```

**管理员权限：**
```sql
GRANT DBA TO usr_test;
REVOKE DBA FROM usr_test;
```

### 1.4 权限查询

- 对象属主权限：查询 `SYS_OBJECTS` 获取用户拥有的对象
- 显式授予的权限：查询 `SYS_ACLS`
- 角色继承的权限：先查 `SYS_ROLE_MEMBERS` 获取用户角色，再查 `SYS_ACLS`

### 1.5 权限包含与继承

- **权限包含**：拥有表的 SELECT 权限隐式拥有所有列的 SELECT 权限
- **权限继承**：将角色授予用户后，用户继承角色所有权限

```sql
-- 创建角色并授权
CREATE ROLE role_test_1;
GRANT SELECT ANY TABLE TO role_test_1;
-- 将角色授予用户
GRANT ROLE role_test_1 TO usr_test;
-- usr_test 即继承了 SELECT ANY TABLE 权限
```

---

## 二、安全策略（行级访问控制 MAC）

基于安全标记的强制访问控制，由安全等级和安全范畴两部分组成（64位：16位等级 + 48位范畴）。

### 2.1 安全等级

值范围 0-30000。
- **读控制**：用户等级 ≥ 数据行等级才能读取
- **写控制**：用户等级 = 数据行等级才能更新/删除
- **插入**：不受安全等级限制

### 2.2 安全范畴

最多 48 个，通过集合包含关系比较。
- **读控制**：用户范畴集合 ⊇ 数据行范畴集合才能读取
- **写控制**：用户范畴集合 = 数据行范畴集合才能更新/删除

### 2.3 安全策略管理

```sql
-- 创建带等级和范畴的安全策略
CREATE POLICY policy_4
  ADD LEVEL level_1 AS 1,
  ADD LEVEL level_2 AS 2,
  ADD CATEGORY category_1,
  ADD CATEGORY category_2;

-- 修改安全策略
ALTER POLICY policy_4 ADD LEVEL level_3 AS 3, DROP CATEGORY category_1;

-- 删除安全策略
DROP POLICY policy_4;

-- 给用户分配安全策略（SYSSSO 执行）
ALTER USER POLICY usr_1 ADD policy_4 LEVEL level_3 CATEGORY category_2;

-- 修改用户安全等级
ALTER USER POLICY usr_1 ALTER policy_4 LEVEL level_4;

-- 删除用户安全策略
ALTER USER POLICY usr_1 DROP policy_4;
```

> **注意：** 每个用户只能分配一个安全策略。安全策略管理操作由 SYSSSO 或具有安全管理员权限的用户执行。

---

## 三、黑白名单

通过信任策略（白名单）和限制策略（黑名单）控制 IP、用户和数据库的访问。

### 3.1 配置方式

| 方式 | 生效范围 | 生效时间 |
|------|---------|---------|
| 修改 `trust.ini` 配置文件 | 当前节点 | 需重启服务 |
| 执行数据库命令 | 单节点/所有节点 | 立即生效 |

### 3.2 配置文件方式

配置文件位置：`SETUP/trust.ini`

```
-- 白名单：允许特定IP的SYSDBA访问SYSTEM库
trust system sysdba from 192.168.1.100

-- 白名单：允许IP段
trust system sysdba from 192.168.1.100 to 192.168.1.109

-- 黑名单：禁止特定IP用户访问
untrust test guest from 192.168.2.50

-- 黑名单：禁止IP段所有用户
untrust report everyone from 192.168.3.200 to 192.168.3.220
```

> **注意：** 默认配置 `trust everydb everyone from anywhere`，需删除此项白名单才能生效。

### 3.3 数据库命令方式

```sql
-- 添加黑名单：禁止高风险IP所有用户访问所有数据库
ALTER CONNECT POLICY ADD DISABLE 'everyone' LOGIN 'everydb' FROM '192.168.2.239' ON ALL NODE;

-- 添加白名单：允许用户从IP段访问指定库
ALTER CONNECT POLICY ADD ENABLE 'user_white' LOGIN 'monitor_db' FROM '192.168.10.50' TO '192.168.10.60' ON ALL NODE;

-- 删除指定黑名单
ALTER CONNECT POLICY DROP DISABLE 'user_black' LOGIN 'customer_db' FROM 'anywhere' ON NODE 1;

-- 删除所有节点所有黑名单
ALTER CONNECT POLICY DROP ALL DISABLE ON ALL NODE;
```

- 查询黑白名单：`SELECT * FROM sys_black_white_list;`
- 全节点查询：`SELECT * FROM sys_all_black_white_list;`

> **重要：** 删除白名单需谨慎。若删除所有白名单：(1) 当前连接未断开时新增白名单；(2) 用 127.0.0.1 登录；(3) 关闭服务修改 trust.ini 重启。
