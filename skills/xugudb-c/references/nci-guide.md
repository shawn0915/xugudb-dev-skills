# NCI (Native Call Interface) 接口指南

## 概述

NCI 是虚谷数据库的原生 C 接口，API 命名风格与 Oracle OCI 一致（仅将 OCI 前缀替换为 NCI），便于 Oracle 开发者快速上手。

## 环境要求

- C/C++ 编译器
- NCI 头文件（nci.h）和库文件
- 平台：Windows x86-64 / Linux x86-64 / Linux ARM-64

## 核心句柄类型

| 句柄类型 | 常量 | 说明 |
|----------|------|------|
| NCIEnv | NCI_HTYPE_ENV | 环境句柄 |
| NCIError | NCI_HTYPE_ERROR | 错误句柄 |
| NCIServer | NCI_HTYPE_SERVER | 服务器句柄 |
| NCISvcCtx | NCI_HTYPE_SVCCTX | 服务上下文句柄 |
| NCISession | NCI_HTYPE_SESSION | 会话句柄 |
| NCIStmt | NCI_HTYPE_STMT | 语句句柄 |

## 初始化

```c
NCIEnv* env;
NCIError* err;
NCIServer* server;
NCISvcCtx* ctx;
NCISession* session;
NCIStmt* stmt;

// 创建环境
NCIEnvCreate(&env, NCI_DEFAULT, NULL, NULL, NULL, NULL, 0, NULL);

// 分配各类句柄
NCIHandleAlloc(env, &err, NCI_HTYPE_ERROR, 0, NULL);
NCIHandleAlloc(env, &server, NCI_HTYPE_SERVER, 0, NULL);
NCIHandleAlloc(env, &ctx, NCI_HTYPE_SVCCTX, 0, NULL);
NCIHandleAlloc(env, &session, NCI_HTYPE_SESSION, 0, NULL);
NCIHandleAlloc(env, &stmt, NCI_HTYPE_STMT, 0, NULL);
```

## 连接方式

### 方式一：分步连接（ServerAttach + SessionBegin）

```c
// 连接服务器（host:port/database 格式）
const char* dblink = "127.0.0.1:5138/SYSTEM";
NCIServerAttach(server, err, dblink, strlen(dblink), NCI_DEFAULT);

// 设置用户名密码
NCIAttrSet(session, NCI_HTYPE_SESSION, user, strlen(user), NCI_ATTR_USERNAME, err);
NCIAttrSet(session, NCI_HTYPE_SESSION, pwd, strlen(pwd), NCI_ATTR_PASSWORD, err);

// 关联上下文
NCIAttrSet(ctx, NCI_HTYPE_SVCCTX, server, 0, NCI_ATTR_SERVER, err);
NCIAttrSet(ctx, NCI_HTYPE_SVCCTX, session, 0, NCI_ATTR_SESSION, err);

// 开始会话
NCISessionBegin(ctx, err, session, NCI_CRED_RDBMS, NCI_DEFAULT);
```

### 方式二：简化连接（NCILogon）

```c
NCILogon(env, err, &ctx, user, strlen(user), pwd, strlen(pwd), db, strlen(db));
// db 格式: "127.0.0.1:5138/SYSTEM"
```

## 执行 SQL

### DDL/DML（iters=1）

```c
const char* sql = "CREATE TABLE test(id INT, name VARCHAR(20))";
NCIStmtPrepare(stmt, err, sql, strlen(sql), NCI_NTV_SYNTAX, NCI_DEFAULT);
NCIStmtExecute(ctx, stmt, err, 1, 0, NULL, NULL, NCI_DEFAULT);
// DDL/DML 的 iters 参数设为 1
```

### DQL 查询（iters=0 + DefineByPos + Fetch）

```c
const char* sql = "SELECT id, name FROM test";
NCIStmtPrepare(stmt, err, sql, strlen(sql), NCI_NTV_SYNTAX, NCI_DEFAULT);

// 定义输出列
int id;
char name[20];
NCIDefine* def[2];
NCIDefineByPos(stmt, &def[0], err, 1, &id, 4, SQLT_INT, NULL, NULL, NULL, NCI_DEFAULT);
NCIDefineByPos(stmt, &def[1], err, 2, name, sizeof(name), SQLT_CHR, NULL, NULL, NULL, NCI_DEFAULT);

// 执行查询（iters=0 表示 SELECT）
NCIStmtExecute(ctx, stmt, err, 0, 0, NULL, NULL, NCI_DEFAULT);

// 遍历结果
while (NCIStmtFetch(stmt, err, 1, NCI_FETCH_NEXT, NCI_DEFAULT) == NCI_SUCCESS) {
    printf("%d, %s\n", id, name);
}
```

## 参数绑定

### 按名绑定（BindByName）

```c
const char* sql = "INSERT INTO test VALUES(:id, :name, :created)";
NCIStmtPrepare(stmt, err, sql, strlen(sql), NCI_NTV_SYNTAX, NCI_DEFAULT);

NCIBind* bind[3];
int id = 1;
const char* name = "Tom";
NCIDate created = { 2000, 1, 1 };

NCIBindByName(stmt, &bind[0], err, "id", 2, &id, 4, SQLT_INT,
              NULL, NULL, NULL, 0, NULL, NCI_DEFAULT);
NCIBindByName(stmt, &bind[1], err, "name", 4, name, sizeof(name), SQLT_STR,
              NULL, NULL, NULL, 0, NULL, NCI_DEFAULT);
NCIBindByName(stmt, &bind[2], err, "created", 7, &created, sizeof(created), SQLT_ODT,
              NULL, NULL, NULL, 0, NULL, NCI_DEFAULT);

NCIStmtExecute(ctx, stmt, err, 1, 0, NULL, NULL, NCI_DEFAULT);
```

## 常用数据类型常量

| 常量 | 说明 |
|------|------|
| SQLT_INT | 整数 |
| SQLT_FLT | 浮点数 |
| SQLT_STR | 字符串（NULL 结尾） |
| SQLT_CHR | 字符数组 |
| SQLT_ODT | NCIDate 日期 |
| SQLT_BLOB | BLOB |
| SQLT_CLOB | CLOB |

## 属性常量

| 常量 | 说明 |
|------|------|
| NCI_ATTR_USERNAME | 用户名 |
| NCI_ATTR_PASSWORD | 密码 |
| NCI_ATTR_SERVER | 服务器句柄 |
| NCI_ATTR_SESSION | 会话句柄 |

## 错误处理

```c
sb4 errCode;
char errMsg[128];
NCIErrorGet(err, 1, NULL, &errCode, errMsg, 128, NCI_HTYPE_ERROR);
printf("[%d] %s\n", errCode, errMsg);
```

## 资源释放

```c
NCIHandleFree(stmt, NCI_HTYPE_STMT);
NCILogoff(ctx, err);  // 或 NCISessionEnd + NCIServerDetach
NCIHandleFree(ctx, NCI_HTYPE_SVCCTX);
NCIHandleFree(err, NCI_HTYPE_ERROR);
NCIHandleFree(env, NCI_HTYPE_ENV);
```

## 核心 API 参考

| 函数 | 说明 |
|------|------|
| NCIEnvCreate | 创建环境句柄 |
| NCIHandleAlloc | 分配句柄 |
| NCIHandleFree | 释放句柄 |
| NCIServerAttach | 连接服务器 |
| NCILogon / NCILogoff | 简化登录/退出 |
| NCISessionBegin / NCISessionEnd | 开始/结束会话 |
| NCIAttrSet / NCIAttrGet | 设置/获取属性 |
| NCIStmtPrepare | 预编译 SQL |
| NCIStmtExecute | 执行 SQL |
| NCIStmtFetch | 获取结果行 |
| NCIBindByName / NCIBindByPos | 参数绑定 |
| NCIDefineByPos | 定义输出列 |
| NCIErrorGet | 获取错误信息 |
| NCITransCommit | 提交事务 |
| NCITransRollback | 回滚事务 |
