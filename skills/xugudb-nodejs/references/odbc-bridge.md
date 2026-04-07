# Node.js ODBC 桥接方式

## 概述

通过 `odbc` npm 包 + 虚谷 ODBC 驱动，Node.js 应用可以连接虚谷数据库。这是一种通用的数据库连接方式，不依赖特定数据库驱动。

## 前置条件

1. **安装虚谷 ODBC 驱动**：参考 [xugudb-odbc](../../xugudb-odbc/SKILL.md)
2. **配置系统 DSN** 或使用连接字符串
3. **安装 npm 包**：

```bash
npm install odbc
```

> `odbc` 包需要系统安装了 unixODBC（Linux）或 ODBC Driver Manager（Windows）

## 连接

### 使用 DSN

```javascript
const odbc = require('odbc');

async function connect() {
  const conn = await odbc.connect('DSN=XuguDB;UID=SYSDBA;PWD=SYSDBA');
  return conn;
}
```

### 使用连接字符串

```javascript
const conn = await odbc.connect(
  'DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA'
);
```

### 连接池

```javascript
const pool = await odbc.pool(
  'DSN=XuguDB;UID=SYSDBA;PWD=SYSDBA',
  { initialSize: 5, maxSize: 10 }
);

const conn = await pool.connect();
// 使用完毕后
await conn.close(); // 返回连接池
```

## 查询

```javascript
// 简单查询
const result = await conn.query('SELECT * FROM test');
console.log(result); // 数组形式的行数据

// 参数化查询（防 SQL 注入）
const rows = await conn.query('SELECT * FROM test WHERE id = ?', [1]);

// 获取列信息
console.log(result.columns); // 列名、类型等元信息
```

## 执行 DML

```javascript
// 插入
const insertResult = await conn.query(
  'INSERT INTO test (name, value) VALUES (?, ?)',
  ['hello', 42]
);

// 更新
const updateResult = await conn.query(
  'UPDATE test SET name = ? WHERE id = ?',
  ['world', 1]
);

// 影响行数
console.log(updateResult.count);
```

## 执行 DDL

```javascript
await conn.query('CREATE TABLE demo (id INT, name VARCHAR(100))');
await conn.query('DROP TABLE demo');
```

## 事务

```javascript
await conn.beginTransaction();
try {
  await conn.query('INSERT INTO test VALUES (?, ?)', [1, 'data1']);
  await conn.query('INSERT INTO test VALUES (?, ?)', [2, 'data2']);
  await conn.commit();
} catch (err) {
  await conn.rollback();
  throw err;
}
```

## 存储过程

```javascript
// 使用 CALL 或虚谷语法调用
const result = await conn.query('EXEC proc_name(?, ?)', [param1, param2]);
```

## 关闭连接

```javascript
await conn.close();
```

## 错误处理

```javascript
try {
  const result = await conn.query('SELECT * FROM nonexistent');
} catch (err) {
  console.error('ODBC Error:', err.message);
  console.error('SQL State:', err.odbcErrors?.[0]?.state);
  console.error('Error Code:', err.odbcErrors?.[0]?.code);
}
```

## 注意事项

- `odbc` npm 包是 C++ 原生模块，安装时需要编译环境（node-gyp）
- Windows 上需安装 Visual Studio Build Tools
- Linux 上需安装 unixODBC-dev 和 C++ 编译器
- 虚谷 ODBC 驱动需要与操作系统架构匹配
- 参数化查询使用 `?` 占位符
