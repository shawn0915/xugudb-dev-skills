# Sequelize ORM 集成虚谷数据库

## 概述

虚谷数据库通过 `xugu-dialect` 适配层集成 Sequelize ORM。该适配层基于 Sequelize 的 mysql dialect 修改，使 Sequelize 能够与虚谷数据库通信。

## 环境要求

- Node.js 运行环境
- Sequelize 6.37.3（推荐版本）
- xugu-dialect（从虚谷官方获取）
- 虚谷数据库 V12.0.0+

## 安装步骤

### 1. 安装依赖

```bash
npm install sequelize dotenv
```

### 2. 部署 xugu-dialect

1. 将 `xugu-dialect` 目录放入 `node_modules/sequelize/lib/dialects/`
2. 备份原 mysql 目录：将 `mysql` 重命名为 `mysql-bak`
3. 确保 `abstract/` 目录保持不变

目录结构：
```
node_modules/sequelize/lib/dialects/
├── abstract/          # 保持不变
├── mysql-bak/         # 原 mysql（备份）
├── xugu-dialect/      # 虚谷适配（替代 mysql）
└── ...
```

### 3. 配置环境变量

创建 `.env` 文件：

```env
DB_NAME=SYSTEM
DB_USER=SYSDBA
DB_PASSWORD=SYSDBA
DB_HOST=192.168.2.216
DB_PORT=5138
```

### 4. 初始化连接

```javascript
// database.js
const { Sequelize } = require('sequelize');
require('dotenv').config();

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASSWORD,
  {
    host: process.env.DB_HOST,
    dialect: 'mysql',  // xugu-dialect 伪装为 mysql
    logging: false,
    define: {
      timestamps: false,
      freezeTableName: true
    }
  }
);

module.exports = sequelize;
```

> 注意：`dialect: 'mysql'` 是因为 xugu-dialect 替换了 mysql 目录

## 模型定义

```javascript
const { DataTypes } = require('sequelize');
const sequelize = require('./database');

const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  name: {
    type: DataTypes.STRING(100),
    allowNull: false
  },
  email: {
    type: DataTypes.STRING(200)
  },
  created: {
    type: DataTypes.DATE,
    defaultValue: DataTypes.NOW
  }
}, {
  tableName: 'users'
});

module.exports = User;
```

## CRUD 操作

### 创建

```javascript
const user = await User.create({
  name: 'Tom',
  email: 'tom@example.com'
});
```

### 查询

```javascript
// 全部
const users = await User.findAll();

// 条件查询
const user = await User.findOne({ where: { name: 'Tom' } });

// 主键查询
const user = await User.findByPk(1);

// 分页
const { count, rows } = await User.findAndCountAll({
  offset: 0,
  limit: 10
});
```

### 更新

```javascript
await User.update(
  { name: 'Jerry' },
  { where: { id: 1 } }
);
```

### 删除

```javascript
await User.destroy({ where: { id: 1 } });
```

## 原始 SQL

对于虚谷特有语法，使用 `sequelize.query()`：

```javascript
const [results] = await sequelize.query(
  'SELECT * FROM test WHERE ROWNUM <= ?',
  { replacements: [10] }
);
```

## 注意事项

- xugu-dialect 需要从虚谷官方渠道获取，npm 上不一定有
- 由于替换了 mysql dialect，同一项目中不能同时连接 MySQL
- 虚谷特有的 SQL 语法（如 `ROWNUM`、`CONNECT BY`）需使用原始 SQL
- 部分 Sequelize 高级功能（如 migration）可能需要额外适配
- Sequelize 版本更新可能导致 xugu-dialect 不兼容，建议锁定版本
- 默认端口 5138，不是 MySQL 的 3306
