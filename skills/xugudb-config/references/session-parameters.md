# 连接会话参数

连接会话参数的作用范围为当前连接会话，通过 `SET 参数名 TO 值` 或 `SET 参数名 ON/OFF` 修改，通过 `SHOW 参数名` 查看。部分参数也可在连接字符串中指定。

## 连接身份与基本信息

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| DATABASE | VARCHAR(128) | SYSTEM | 否 | 当前连接的数据库名，登录时指定。未指定则默认 SYSTEM，离线库不允许登录 |
| USER | VARCHAR | SYSDBA | 否 | 当前连接的用户名 |
| PASSWORD | VARCHAR | SYSDBA | 否 | 用户身份认证口令，登录时指定 |
| SESSION_USER | VARCHAR | - | 否 | 会话用户名 |
| APP_NAME | VARCHAR(128) | NULL | 是 | 客户端应用名称，方便区分不同应用 |
| DRIVER_VERSION | VARCHAR | 301 | 否 | 通讯协议版本号：302/301/201 |
| LANGUAGE | VARCHAR | PL/SQL | 是 | 存储过程语言类型，当前仅支持 PL/SQL |
| SESSION_CMDSTR | VARCHAR | - | 否 | 会话命令字符串 |

## 字符集与编码

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| CHAR_SET | VARCHAR | GBK | 是 | 客户端字符集编码（等效 CLIENT_ENCODING）。未指定则使用 def_charset |
| CLIENT_ENCODING | VARCHAR | GBK | 是 | 等效 CHAR_SET。支持 UTF8/GBK/GB18030/BINARY 等 |

### 字符集说明

- 只有字符类数据涉及字符集转换（入库：客户端->库，查询：库->客户端）
- 避开字符集转换可减少性能开销
- 客户端实际字符集与连接设置不一致时出现乱码
- 排序规则：ci=不区分大小写，bin=区分大小写，cs=区分大小写
- 不指定 bin/ci 后缀时使用 bin 类字符集
- 查看支持的字符集：`SHOW CHARSETS`

## 兼容模式

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| COMPATIBLE_MODE | VARCHAR | NONE | 是 | 异构库兼容模式。未指定则使用 def_compatible_mode |

| 值 | 标识符处理 |
|----|-----------|
| NONE | 统一转大写 |
| ORACLE | 统一转大写 |
| MYSQL | 不做处理 |
| POSTGRESQL | 统一转小写 |

```sql
SET compatible_mode to 'MYSQL';
```

## 事务控制

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| AUTO_COMMIT | BOOLEAN | TRUE | 是 | 事务自动提交模式。未指定则使用 def_auto_commit |
| ISO_LEVEL | INTEGER | 1 | 是 | 事务隔离级别（等效 SESSION_ISO_LEVEL）。0=只读，1=读已提交，2=可重复读，3=序列化 |
| SESSION_ISO_LEVEL | VARCHAR | - | 是 | 等效 ISO_LEVEL，支持字符串值 |
| STRICT_COMMIT | BOOLEAN | - | 是 | 事务日志是否写实，未指定则使用 strictly_commit |
| TRANS_READONLY | BOOLEAN | - | 是 | 当前事务是否只读 |

```sql
-- 关闭自动提交
SET auto_commit OFF;

-- 设置隔离级别为可重复读
SET ISO_LEVEL TO 2;

-- 查看隔离级别
SHOW TRANSACTION ISOLATION LEVEL;
```

## 模式与Schema

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| SCHEMA | VARCHAR(128) | - | 是 | 当前会话模式（等效 CURRENT_SCHEMA） |
| CURRENT_SCHEMA | VARCHAR(128) | - | 是 | 当前会话模式。不指定则默认与用户同名的模式。设为 DEFAULT 切回用户同名模式 |
| RETURN_SCHEMA | BOOLEAN | - | 是 | 是否返回模式信息 |

```sql
-- 切换模式
SET CURRENT_SCHEMA TO sch_params;
-- 或
ALTER SESSION SET CURRENT_SCHEMA = sch_params;

-- 连接字符串中指定
jdbc:xugu://localhost:5183/db_params?USER=SYSDBA&PASSWORD=SYSDBA&current_schema=sch_params
```

## 数据处理

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| EMPTY_STR_AS_NULL | BOOLEAN | FALSE | 是 | 是否将空串作为 NULL。未指定则使用 def_empty_str_as_null |
| IDENTITY_MODE | VARCHAR | DEFAULT | 是 | 自增列填充模式：DEFAULT(NULL 报错)/NULL_AS_AUTO_INCREMENT/ZERO_AS_AUTO_INCREMENT。未指定则使用 def_identity_mode |
| LOB_RET | BOOLEAN | FALSE | 是 | 是否开启大对象描述符。TRUE=大对象作为描述符返回，FALSE=二进制形式返回 |
| RESULT | VARCHAR(128) | DEFAULT | 是 | 数据类型返回值模式。CHAR=所有类型转字符串，DEFAULT=按类型定义发送 |
| RETURN_ROWID | BOOLEAN | - | 是 | 是否返回行 ID |
| RETURN_CURSOR_ID | BOOLEAN | - | 是 | 是否返回游标 ID |
| TIME_FORMAT | VARCHAR | - | 是 | 时间格式设置 |
| DISABLE_BINLOG | BOOLEAN | FALSE | 是 | 是否禁用变更记载 |

```sql
-- 空串作为 NULL
SET EMPTY_STR_AS_NULL to on;

-- 自增列兼容 MySQL（NULL 和 0 用自增值替换）
SET IDENTITY_MODE TO 'ZERO_AS_AUTO_INCREMENT';
```

## 优化器与执行

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| OPTIMIZER_MODE | VARCHAR | ALL_ROWS | 是 | 优化器模式。ALL_ROWS=最佳吞吐量（倾向 HashJoin），FIRST_ROWS(N)=最快返回前 N 条（倾向索引扫描） |
| KEYWORD_FILTER | VARCHAR | NULL | 是 | 关键字过滤，过滤后的关键字在 DML 中当作标识符。多个用逗号分隔，区分大小写 |

### OPTIMIZER_MODE 详解

**ALL_ROWS**：默认模式，优化全部结果的检索时间，适合大数据量查询。

**FIRST_ROWS(N)**：优化前 N 条数据的响应速度，适合在线事务处理和网站场景。根据 N 值决定是否使用索引扫描。

```sql
-- 设置为快速返回前 10 行
SET OPTIMIZER_MODE TO 'FIRST_ROWS(10)';

-- 恢复默认
SET OPTIMIZER_MODE TO 'ALL_ROWS';
```

### KEYWORD_FILTER 说明

过滤后的关键字可作为表名、列名、别名使用（仅在 SELECT/INSERT/UPDATE/DELETE/MERGE/CREATE TABLE 中）。注意：过滤 FROM/GROUP/ORDER/OR/AND 后相关 SELECT 将无法执行。

```sql
-- 连接字符串设置
jdbc:xugu://localhost:5183/SYSTEM?USER=SYSDBA&PASSWORD=SYSDBA&KEYWORD_FILTER=TABLE,FUNCTION

-- SET 命令设置
SET KEYWORD_FILTER TO 'TABLE,FUNCTION';
```

## 游标信息

| 参数名 | 类型 | 默认值 | 可修改 | 说明 |
|--------|------|--------|--------|------|
| CURSORS | RESULTSET | - | 否 | 当前会话的游标信息：CURSOR_NAME/CURSOR_TYPE/FOR_UPDATE/IS_OPENED |

```sql
SHOW CURSORS;
```

## 连接字符串参数汇总

以下参数支持在 JDBC 连接字符串中指定：

```
jdbc:xugu://host:port/database?USER=xxx&PASSWORD=xxx
  &CHAR_SET=UTF8
  &COMPATIBLE_MODE=MYSQL
  &CURRENT_SCHEMA=schema_name
  &IDENTITY_MODE=ZERO_AS_AUTO_INCREMENT
  &APP_NAME=MyApp
  &KEYWORD_FILTER=TABLE,FUNCTION
  &DRIVER_VERSION=302
```

## SET / SHOW 命令语法

```sql
-- 设置参数（以下写法等效）
SET 参数名 TO 值;
SET 参数名 ON;     -- BOOLEAN 类型
SET 参数名 OFF;    -- BOOLEAN 类型
SET 参数名 TO TRUE;
SET 参数名 TO FALSE;

-- 查看参数
SHOW 参数名;

-- 查看所有参数
SHOW SYS_VARS;

-- ALTER SESSION 语法（部分参数支持）
ALTER SESSION SET CURRENT_SCHEMA = schema_name;
```
