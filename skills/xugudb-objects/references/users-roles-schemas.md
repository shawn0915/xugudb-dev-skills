# 用户、角色与模式管理

## 库管理

```sql
-- 创建数据库
CREATE DATABASE [IF NOT EXISTS] db_name
    [CHARACTER SET 'UTF8']
    [TIME ZONE 'GMT+08:00']
    [ENCRYPT BY 'encryptor_name'];

-- 删除数据库
DROP DATABASE db_name;

-- 查看数据库列表
SELECT * FROM SYS_DATABASES;
```

> 只有 SYSDBA 可以在 SYSTEM 库中创建新数据库

## 用户管理

### 预置用户

| 用户 | 角色 | 说明 |
|------|------|------|
| SYSDBA | 数据库管理员 | 最高权限，管理用户/库/表等 |
| SYSAUDITOR | 审计管理员 | 审计策略管理 |
| SYSSSO | 安全管理员 | 安全策略/加密/标签管理 |
| GUEST | 普通用户 | 预置普通用户 |

### 创建用户

```sql
CREATE USER user_name IDENTIFIED BY 'password'
    [LOGIN alias_name]                    -- 登录别名
    [DEFAULT ROLE role1, role2]           -- 默认角色
    [VALID UNTIL 'datetime']             -- 账户有效期
    [ACCOUNT LOCK | UNLOCK]              -- 锁定/解锁
    [PASSWORD EXPIRE]                    -- 密码立即过期
    [QUOTA size ON tablespace]           -- 空间配额
    [ENCRYPT BY 'encryptor'];            -- 加密
```

### 修改用户

```sql
ALTER USER user_name IDENTIFIED BY 'new_password';
ALTER USER user_name ACCOUNT LOCK;
ALTER USER user_name VALID UNTIL '2026-12-31';
ALTER USER user_name DEFAULT ROLE role_name;
```

### 删除用户

```sql
DROP USER user_name [CASCADE];
```

### 密码策略

通过系统参数控制密码复杂度：

| 参数 | 说明 |
|------|------|
| pass_mode | 密码模式（1-4） |
| min_pass_len | 最小密码长度（默认 8） |
| min_pass_number | 最少数字数 |
| min_pass_mixed_case | 最少大小写混合 |
| min_pass_special_char | 最少特殊字符数 |
| pass_username_check | 是否检查用户名相似 |
| weak_pass_dictionary | 弱密码字典 |

## 角色管理

### 预置角色

| 角色 | 说明 |
|------|------|
| DB_ADMIN | DBA 角色（SYSDBA 持有） |
| DB_POLICY_ADMIN | 安全策略管理角色（SYSSSO） |
| DB_POLICY_OPER | 安全策略操作角色 |
| DB_AUDIT_ADMIN | 审计管理角色（SYSAUDITOR） |
| DB_AUDIT_OPER | 审计操作角色 |
| PUBLIC | 公共角色（所有用户自动拥有） |

### 自定义角色

```sql
-- 创建角色
CREATE ROLE app_reader;

-- 授予权限
GRANT SELECT ON schema.table TO app_reader;
GRANT SELECT ON schema.view TO app_reader;

-- 将角色授予用户
GRANT app_reader TO user1, user2;

-- 删除角色
DROP ROLE app_reader;
```

## 模式管理

每个用户创建时自动创建同名模式。也可手动创建模式。

```sql
-- 创建模式
CREATE SCHEMA schema_name [AUTHORIZATION user_name];

-- 切换当前模式
SET [CURRENT] SCHEMA schema_name;
SET SCHEMA DEFAULT;  -- 切回默认模式

-- 查看当前模式
SELECT CURRENT_SCHEMA();

-- 修改模式
ALTER SCHEMA schema_name RENAME TO new_name;
ALTER SCHEMA schema_name OWNER TO new_owner;

-- 删除模式
DROP SCHEMA schema_name [CASCADE];

-- 查看模式列表
SELECT * FROM DBA_SCHEMAS;
SELECT * FROM USER_SCHEMAS;
```

## 查看对象信息

```sql
-- 查看所有表
SELECT * FROM ALL_TABLES;
SELECT * FROM DBA_TABLES;
SELECT * FROM USER_TABLES;

-- 查看列信息
SELECT * FROM SYS_COLUMNS WHERE table_id = ...;

-- 查看约束
SELECT * FROM SYS_CONSTRAINTS WHERE table_id = ...;

-- 查看用户
SELECT * FROM ALL_USERS;

-- 查看模式
SELECT * FROM DBA_SCHEMAS;

-- 查看序列
SELECT * FROM USER_SEQUENCES;
```
