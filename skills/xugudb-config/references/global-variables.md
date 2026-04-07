# 内置全局变量

内置全局变量是在编译和安装时确定的预设参数，通过 `SHOW 变量名` 查看。大部分不可修改，少数支持在线修改。通过 `SHOW SYS_VARS` 可查看所有系统变量和参数。

## 版本与构建信息

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| VERSION | VARCHAR | 否 | 数据库发布版本（如 "XuguDB 12.0.0"），受 use_old_product_name 影响 |
| VERSION_NUM | INTEGER | 否 | 内核版本号的数字形式（如 1209006） |
| DB_KERNEL | VARCHAR | 否 | 内核版本号（如 "12.9.6"） |
| BUILD_TIME | VARCHAR | 否 | 数据库构建时间（如 "2025-07-14 12:00:00 GA"） |
| LICENSE_TYPE | VARCHAR | 否 | 许可类型：试用版本/个人版本/标准版本/企业版本/分布式集群版本/通用版本/敏捷版本 |
| LICENSE_INFO | RESULTSET | 否 | 许可详细信息（license_type/version/gen_date/expire_date/cpu_num/user_num/功能项等） |

## 系统环境信息

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| OS_TYPE | VARCHAR | 否 | 操作系统类型（LINUX32/LINUX64） |
| MACHINE_TYPE | VARCHAR | 否 | CPU 架构（X86/X86_64） |
| INSTALL_PATH | VARCHAR | 否 | 数据库当前节点安装路径 |
| WORK_PATH | VARCHAR | 否 | 数据库当前节点工作路径（数据文件的父级目录） |

## 数据库信息查询

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| DB_INFO | RESULTSET | 否 | 当前连接的逻辑库信息：DB_NAME/DB_ID/DB_OWNER/DB_CHARSET/DB_TIMEZ |
| DATA_TYPES | RESULTSET | 否 | 支持的基础数据类型，等效 SYSDBA.SYS_DATATYPES |
| CHARSETS | RESULTSET | 否 | 支持的字符集列表，等效 SYSDBA.SYS_CHARSETS。排序规则后缀：_bin=大小写敏感，_ci=不敏感，_cs=敏感 |
| METHODS | RESULTSET | 否 | 支持的内置函数，等效 SYSDBA.SYS_METHODS。含 OPTION 字段标识函数特性 |
| OPERATORS | RESULTSET | 否 | 支持的操作符，等效 SYSDBA.SYS_OPERATORS |
| SERVER_INFO | RESULTSET | 否 | 数据库服务信息：SERVER_NAME/SERVER_IP/SERVER_PORT |
| CATA_CAPS | BOOLEAN | 否 | 大小写是否不敏感，与 cata_case_sensitive 取值相反 |

## 系统运行状态

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| RUN_INFO | RESULTSET | 否 | 系统运行状态，等效 SYSDBA.SYS_RUN_INFO。含 TCP 请求次数、缓存命中、磁盘读写、网络读写、锁信息等 |
| MEM_STATUS | RESULTSET | 否 | 内存状态，等效 SYSDBA.SYS_MEM_STATUS。含数据缓存/SGA/交换内存的总量/空闲/脏页/峰值等 |
| THD_STATUS | RESULTSET | 否 | 所有线程状态，等效 SYSDBA.SYS_THD_STATUS。含线程号/类型/状态/事务ID/等候资源/内存等 |
| CTL_VARS | RESULTSET | 否 | 核心控制信息，等效 SYSDBA.SYS_CTL_VARS。含事务号/checkpoint 点位/日志文件号/初建时间等 |
| SYS_VARS | RESULTSET | 否 | 所有配置参数信息，等效 SYSDBA.SYS_VARS。含参数名/是否全局/访问权限/描述 |

## 集群信息

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| CLUSTERS | RESULTSET | 否 | 集群信息，等效 SYSDBA.SYS_CLUSTERS。含节点 ID/机架号/IP/端口/角色/状态/LPU/存储权重/存储数等 |
| CLUSTER_FAULT_LEVEL | INTEGER | 是（在线修改） | 集群错误级别。0=正常，1=只读。集群降级为只读时，在主 Master 执行 `SET CLUSTER_FAULT_LEVEL TO 0` 可恢复 |
| GSTORES | RESULTSET | 否 | 集群全局存储信息，等效 SYSDBA.SYS_GSTORES。含全局存储号/版本节点/版本局部存储号等 |
| MSG_TIMEOUT | INTEGER | 否 | 当前节点消息超时重发时间（微秒），对应 cluster.ini 中的 TIMEOUT |

## 安全信息

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| ENCRYPTORS | RESULTSET | 否 | 加密机信息，等效 SYSSSO.SYS_ENCRYPTORS。仅 SSO 权限用户可访问 |

## 调试与控制

| 变量名 | 类型 | 可修改 | 说明 |
|--------|------|--------|------|
| DEBUG_PRINT | BOOLEAN | 是（在线修改） | 是否打印调试信息到 stdout.txt，用于分析网络重发/丢包 |
| EXCLUDE_ERRNO | INTEGER | 是（在线修改） | 设置不记载 ERROR 日志的异常号，每次只能设一个，编号不超过 1280 |

## 使用示例

```sql
-- 查看数据库版本
SHOW VERSION;

-- 查看内存状态
SHOW MEM_STATUS;

-- 查看集群信息
SHOW CLUSTERS;

-- 查看所有配置参数
SHOW SYS_VARS;

-- 查看当前库信息
SHOW DB_INFO;

-- 查看线程状态
SHOW THD_STATUS;

-- 查看运行信息
SHOW RUN_INFO;

-- 开启调试信息打印
SET DEBUG_PRINT ON;

-- 排除特定错误号不记入日志
SET EXCLUDE_ERRNO TO 2;

-- 恢复集群只读状态
SET CLUSTER_FAULT_LEVEL TO 0;
```
