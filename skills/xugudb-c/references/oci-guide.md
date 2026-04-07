# OCI (Oracle Call Interface) 兼容接口指南

## 概述

虚谷数据库提供与 Oracle OCI 兼容的 C 接口（XuguDB-OCI），API 函数签名与 Oracle OCI 完全一致。这使得基于 Oracle OCI 开发的 C/C++ 应用可以低成本迁移到虚谷数据库。

## 与 Oracle OCI 的兼容性

- 函数签名完全一致：`OCIEnvCreate`、`OCIStmtPrepare`、`OCIStmtExecute` 等
- 句柄类型一致：`OCI_HTYPE_ENV`、`OCI_HTYPE_ERROR`、`OCI_HTYPE_STMT` 等
- 数据类型一致：`SQLT_INT`、`SQLT_STR`、`SQLT_ODT` 等
- 返回值一致：`OCI_SUCCESS`、`OCI_ERROR` 等

## 迁移步骤

从 Oracle OCI 迁移到虚谷 OCI：

1. **替换头文件**：`#include "oci.h"`（使用虚谷提供的 oci.h）
2. **替换库文件**：链接虚谷 OCI 库（替换 Oracle 的 libclntsh）
3. **修改连接串**：改为虚谷格式 `host:port/database`
4. **编译链接**：使用虚谷 OCI 库重新编译

> 应用层代码通常不需要修改

## 连接串格式差异

| Oracle OCI | 虚谷 OCI |
|------------|---------|
| `//host:port/service_name` 或 TNS 名 | `host:port/database` |
| 如 `//192.168.1.1:1521/orcl` | 如 `192.168.1.1:5138/SYSTEM` |

## 快速示例

```c
#include "oci.h"

OCIEnv* env;
OCIError* err;
OCISvcCtx* ctx;
OCIStmt* stmt;

// 初始化（与 Oracle 代码完全一致）
OCIEnvCreate(&env, OCI_DEFAULT, NULL, NULL, NULL, NULL, 0, NULL);
OCIHandleAlloc(env, &err, OCI_HTYPE_ERROR, 0, NULL);

// 连接（仅连接串不同）
OCILogon(env, err, &ctx, "SYSDBA", 6, "SYSDBA", 6,
         "127.0.0.1:5138/SYSTEM", 22);

// 执行 SQL（与 Oracle 完全一致）
OCIHandleAlloc(env, &stmt, OCI_HTYPE_STMT, 0, NULL);
OCIStmtPrepare(stmt, err, "SELECT id FROM test", 20, OCI_NTV_SYNTAX, OCI_DEFAULT);

int id;
OCIDefine* def;
OCIDefineByPos(stmt, &def, err, 1, &id, 4, SQLT_INT, NULL, NULL, NULL, OCI_DEFAULT);
OCIStmtExecute(ctx, stmt, err, 0, 0, NULL, NULL, OCI_DEFAULT);

while (OCIStmtFetch(stmt, err, 1, OCI_FETCH_NEXT, OCI_DEFAULT) == OCI_SUCCESS) {
    printf("%d\n", id);
}

// 释放
OCIHandleFree(stmt, OCI_HTYPE_STMT);
OCILogoff(ctx, err);
OCIHandleFree(err, OCI_HTYPE_ERROR);
OCIHandleFree(env, OCI_HTYPE_ENV);
```

## 已实现的 Oracle OCI API

| 类别 | 函数 |
|------|------|
| 环境管理 | OCIEnvCreate, OCIHandleAlloc, OCIHandleFree |
| 连接管理 | OCIServerAttach, OCISessionBegin, OCILogon, OCILogoff |
| 属性操作 | OCIAttrSet, OCIAttrGet |
| SQL 执行 | OCIStmtPrepare, OCIStmtExecute, OCIStmtFetch |
| 参数绑定 | OCIBindByName, OCIBindByPos |
| 结果定义 | OCIDefineByPos |
| 事务控制 | OCITransCommit, OCITransRollback |
| 错误处理 | OCIErrorGet |

## 注意事项

- 虚谷 OCI 兼容层覆盖了常用 OCI API，但部分高级特性可能未实现
- 连接串格式与 Oracle 不同，不支持 TNS 名称解析
- 如果使用了 Oracle 专有扩展（如 OCI 连接池、TAF），需要评估兼容性
- NCI 和 OCI 接口功能等价，新项目建议使用 NCI 以避免与 Oracle 库冲突
