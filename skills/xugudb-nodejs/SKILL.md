---
name: 虚谷数据库 Node.js 驱动开发
name_for_command: xugudb-nodejs
description: |
  XuguDB Node.js 开发指南：通过 ODBC 桥接驱动或 Sequelize ORM 连接虚谷数据库。
  涵盖 ODBC 方式的连接管理、CRUD 操作、参数化查询，以及 Sequelize ORM 集成。
  适用于使用 Node.js 构建 Web 应用并连接虚谷数据库的场景。
  注：本技能基于 ODBC 通用接口和官方 Sequelize 集成文档推导，无独立 Node.js 驱动文档。
tags: xugudb, nodejs, javascript, odbc, sequelize, orm, web
---

# 虚谷数据库 Node.js 驱动开发

虚谷数据库目前通过以下方式支持 Node.js 开发：

| 方式 | 说明 | 适用场景 |
|------|------|----------|
| **ODBC 桥接** | 通过 `odbc` npm 包 + 虚谷 ODBC 驱动 | 需要底层控制的场景 |
| **Sequelize ORM** | 通过 xugu-dialect 适配 Sequelize | Web 应用/ORM 偏好 |

## 方式一：ODBC 桥接

### 前置条件

1. 安装虚谷 ODBC 驱动（配置系统 DSN）
2. 安装 npm 包：`npm install odbc`

### 连接

```javascript
const odbc = require('odbc');

// 方式1：使用 DSN
const conn = await odbc.connect('DSN=XuguDB;UID=SYSDBA;PWD=SYSDBA');

// 方式2：使用连接字符串
const conn = await odbc.connect(
  'DRIVER={XuguDB ODBC Driver};SERVER=127.0.0.1;PORT=5138;DATABASE=SYSTEM;UID=SYSDBA;PWD=SYSDBA'
);
```

### CRUD 操作

```javascript
// 查询
const result = await conn.query('SELECT * FROM test');
console.log(result);

// 参数化查询
const rows = await conn.query('SELECT * FROM test WHERE id = ?', [1]);

// 执行 DML
await conn.query('INSERT INTO test (name) VALUES (?)', ['hello']);

// 关闭连接
await conn.close();
```

> 详细参考：[ODBC 连接方式](references/odbc-bridge.md)

## 方式二：Sequelize ORM

### 安装

```bash
npm install sequelize dotenv
# xugu-dialect 需要从虚谷官方获取
```

### 配置步骤

1. 将 `xugu-dialect` 放入 `node_modules/sequelize/lib/dialects/` 目录
2. 替换 `mysql` dialect 目录（备份原 mysql 为 mysql-bak）
3. 配置 `.env` 文件

### .env 配置

```env
DB_NAME=SYSTEM
DB_USER=SYSDBA
DB_PASSWORD=SYSDBA
DB_HOST=192.168.2.216
DB_PORT=5138
```

### 连接初始化

```javascript
const { Sequelize } = require('sequelize');
require('dotenv').config();

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_HOST,
    dialect: 'mysql',  // 使用 mysql dialect（已替换为 xugu-dialect）
    logging: false
  }
);

module.exports = sequelize;
```

### 定义模型

```javascript
const { DataTypes } = require('sequelize');

const User = sequelize.define('User', {
  id: { type: DataTypes.INTEGER, primaryKey: true, autoIncrement: true },
  name: { type: DataTypes.STRING(100) },
  email: { type: DataTypes.STRING(200) }
}, {
  tableName: 'users',
  timestamps: false
});
```

### CRUD 操作

```javascript
// 创建
await User.create({ name: 'Tom', email: 'tom@example.com' });

// 查询
const users = await User.findAll();
const user = await User.findByPk(1);

// 更新
await User.update({ name: 'Jerry' }, { where: { id: 1 } });

// 删除
await User.destroy({ where: { id: 1 } });
```

> 详细参考：[Sequelize ORM 集成](references/sequelize-integration.md)

## 注意事项

- 虚谷数据库暂无原生 Node.js 驱动，需通过 ODBC 或 ORM 适配层访问
- Sequelize 集成使用 mysql dialect 作为基础，部分虚谷特有语法可能需要使用原始 SQL
- ODBC 方式需要系统级安装 ODBC 驱动和配置 DSN
- Sequelize 版本需与 xugu-dialect 版本匹配（推荐 Sequelize 6.37.3）
- 默认端口为 5138

## 工作流程

当用户咨询 Node.js 开发相关问题时：

1. 确定使用方式（ODBC 桥接 / Sequelize ORM）
2. ODBC 方式需确认 ODBC 驱动已安装配置
3. Sequelize 方式需确认 xugu-dialect 已正确部署
4. 提供对应方式的代码示例
5. 对虚谷特有语法建议使用 `sequelize.query()` 原始 SQL

## 参考文档

- [ODBC 桥接方式](references/odbc-bridge.md) — ODBC 驱动配置、odbc npm 包使用、CRUD 示例
- [Sequelize ORM 集成](references/sequelize-integration.md) — xugu-dialect 部署、模型定义、ORM 操作
