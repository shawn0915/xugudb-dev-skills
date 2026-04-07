# 数据加密

虚谷数据库支持数据传输加密和透明数据加密（TDE）两种加密方式。

---

## 一、数据传输加密

在驱动与数据库之间传输过程中加密敏感数据，防止数据流被截获导致泄露。

### 1.1 先决条件

安装虚谷数据传输加密动态库：
- **Windows**：将 `xgssl.dll` 放置于 `C:\Windows\System32` 目录
- **Linux**：将 `libxgssl.so` 放置于 `/usr/lib` 或 `/usr/lib64` 目录

### 1.2 各驱动启用方式

| 驱动 | 启用方式 |
|------|---------|
| JDBC | 传输安全配置（驱动属性 `SSL=SSL`） |
| Python | `connect()` 参数 |
| C# | ConnectionString |
| Go | 暂未支持 |
| PHP | `__construct` 参数 |
| Node.js | 暂未支持 |
| ODBC | 配置数据源 |
| NCI | 暂未支持 |
| OCI | 暂未支持 |
| XGCI | `XGCIHandleAttrSet` |

### 1.3 DBeaver 启用示例

1. 在 DBeaver 中修改虚谷连接，配置驱动属性 `SSL=SSL`
2. 重新连接后传输数据即为密文

---

## 二、透明数据加密（TDE）

对存储于文件系统中的静态数据文件执行加密，加解密过程对用户和应用不可见。

### 2.1 启用步骤

1. 使用 ACL_SSO 权限用户创建加密机
2. 对库/用户/表设置加密机

**加密机优先级：** 表级 > 用户级 > 库级

> **重要：** 系统表不支持加密，仅支持用户表。

### 2.2 创建加密机

```sql
-- 使用 SYSSSO 安全管理员登录
SQL> CREATE ENCRYPTOR 'enc_name' BY 'enc_key_value';
```

### 2.3 加密库

对当前会话所在库下表数据加密。

```sql
ENCRYPT DATABASE BY ( encryptor_id | 'encryptor_name' ) [CASCADE] [FORCE]
```

| 参数 | 说明 |
|------|------|
| `CASCADE` | 设置库加密机并加密库下所有已有用户表 |
| `FORCE` | 强制加密（已加密的表先解密再重新加密） |
| 未指定 CASCADE | 仅设置加密机，后续新建表自动使用 |

```sql
-- 创建加密机
SQL> CREATE ENCRYPTOR 'enc_db' BY 'enc_key_value';
-- 创建库并切换
SQL> CREATE DATABASE db_enc;
SQL> USE db_enc;
-- 加密库
SQL> ENCRYPT DATABASE BY 'enc_db';
```

### 2.4 加密用户

```sql
ENCRYPT USER user_name BY ( encryptor_id | 'encryptor_name' ) [CASCADE] [FORCE]
```

```sql
SQL> CREATE ENCRYPTOR 'enc_user' BY 'enc_key_value';
SQL> CREATE USER user_enc IDENTIFIED BY 'Test@2002';
SQL> ENCRYPT USER user_enc BY 'enc_user';
```

### 2.5 加密表

**创建加密表：**
```sql
SQL> CREATE ENCRYPTOR 'enc_tab_new' BY 'enc_key_value';
SQL> CREATE TABLE tab_enc(id INTEGER, name VARCHAR) ENCRYPT BY 'enc_tab_new';
```

**加密已有表：**
```sql
ENCRYPT TABLE table_name BY ( encryptor_id | 'encryptor_name' ) [FORCE]
```

```sql
SQL> CREATE ENCRYPTOR 'enc_tab_old' BY 'enc_key_value';
SQL> CREATE TABLE tab_enc(id INTEGER, name VARCHAR);
SQL> ENCRYPT TABLE tab_enc BY 'enc_tab_old';
```

> **重要：** 启用 CASCADE 或 FORCE 时请谨慎操作，提前评估数据量，避免影响业务。
