# 身份鉴别与认证

虚谷数据库采用基于口令的强身份认证机制，通过多因素安全验证流程确认用户身份。

## 一、口令策略参数

身份认证相关参数在配置文件 `xugu.ini` 中，可通过 `SET`/`SHOW` 命令查看和修改。

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `min_pass_len` | 8 | 用户口令最小长度 |
| `pass_mode` | 2 | 用户口令模式 |
| `conn_fail_cnt` | 3 | 禁止登录的失败次数 |
| `min_pass_number` | 0 | 口令中最少数字数 |
| `min_pass_mixed_case` | 0 | 口令中最少大小写字母数 |
| `min_pass_special_char` | 0 | 口令中最少特殊字符数 |
| `pass_username_check` | true | 口令-用户名匹配检测 |
| `weak_pass_dictionary` | true | 弱口令字典检测 |

```sql
-- 查看口令模式
SQL> SHOW PASS_MODE;

-- 修改口令模式
SQL> SET PASS_MODE TO 3;
```

> **注意：** 只有数据库管理员有权限修改数据库参数。命令方式修改直接生效，修改 `xugu.ini` 需重启。

## 二、认证时机

虚谷数据库在以下操作时进行认证：
1. 用户登录数据库
2. 切换数据库
3. 客户端连接断开后自动重连

使用 SSL 连接时，认证逻辑发生在 SSL 握手成功后。

## 三、认证方式

校验流程：
1. 校验用户口令（口令复杂度）
2. 校验登录 IP（黑白名单规则）
3. 校验用户合规性（是否被锁定、是否在有效期、口令是否失效）

通过所有鉴别后用户可成功登录。

## 四、失败处理

- 默认以错误口令登录失败 **3 次**后锁定该 IP
- 锁定后 **3 分钟**内此 IP 任何用户（即使输入正确口令）无法登录
- 可通过 `conn_fail_cnt` 参数自定义锁定失败次数
- 通过系统表 `SYS_ALL_FORBIDDEN_IPS` 查看失败情况

## 五、口令存储

用户口令使用加密存储在系统表 `SYS_USERS` 中：

```sql
SQL> SELECT db_id, user_id, user_name, password FROM SYS_USERS WHERE IS_ROLE = false;
-- PASSWORD 字段显示为 <BINARY>（加密存储）
```

系统内置用户：SYSDBA（系统管理员）、SYSSSO（安全管理员）、SYSAUDITOR（审计管理员）、GUEST（来宾用户）。
