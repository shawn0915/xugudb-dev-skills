# XuguDB Dev Skills

**虚谷数据库 Claude Code 技能包** -- 为 AI 编码助手提供 XuguDB 领域专业知识。

25 个技能模块 / 91 份参考文档 / 覆盖 SQL、PL/SQL、分布式架构、向量检索、空间数据库、8 种语言驱动、Oracle/MySQL/PG 迁移。

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

---

## 安装

### 方式一：npx 一键安装（推荐）

```bash
npx xugudb-dev-skills
```

执行后自动将技能包下载到 Claude Code 的技能目录。

### 方式二：Claude Code 插件市场

```bash
claude install kourou25/xugudb-dev-skills
```

### 方式三：Git 克隆（本地开发）

```bash
# GitHub
git clone https://github.com/kourou25/xugudb-dev-skills.git

# 镜像（国内加速）
git clone https://gitee.com/kourou25/xugudb-dev-skills.git
git clone https://ghproxy.com/https://github.com/kourou25/xugudb-dev-skills.git
```

克隆后将目录放到项目根目录或 Claude Code 技能搜索路径中即可。

### 方式四：直接下载

前往 [Releases](https://github.com/kourou25/xugudb-dev-skills/releases) 下载 zip 包，解压到项目目录。

### 验证安装

在 Claude Code 中输入任意技能命令，如：

```
/xugudb
```

如果返回虚谷数据库概览信息，说明安装成功。也可以用自然语言提问：

```
虚谷数据库怎么创建分布式集群？
```

---

## 技能列表

| 命令 | 名称 | 说明 | 参考文档 |
|------|------|------|:--------:|
| `/xugudb` | 产品概览 | 架构设计、版本选型、快速上手 | 5 |
| `/xugudb-sql` | SQL 语法 | DDL/DML/查询/数据类型/运算符 + Oracle/MySQL/PG 对比 | 8 |
| `/xugudb-plsql` | PL/SQL 编程 | 存储过程、函数、触发器、游标、异常处理 | 1 |
| `/xugudb-functions` | 系统函数 | 字符串/数学/日期/聚合/分析/JSON/XML 等 21 类函数 | 6 |
| `/xugudb-data-dictionary` | 数据字典 | 系统表、系统视图、系统包 | 1 |
| `/xugudb-objects` | 对象管理 | 表/索引/视图/约束/序列/触发器/DBLink 等 | 5 |
| `/xugudb-config` | 系统配置 | xugu.ini 参数、集群配置、类型映射 | 4 |
| `/xugudb-security` | 安全权限 | 认证、权限、角色、审计、加密 | 4 |
| `/xugudb-deployment` | 安装部署 | 标准版/企业版/分布式版/安全版/Docker | 6 |
| `/xugudb-distributed` | 分布式架构 | 集群部署、节点角色（M/QW/S/G）、存算分离/融合 | 2 |
| `/xugudb-ha` | 高可用 | 集群管理、备份恢复、故障切换 | 5 |
| `/xugudb-migration` | 数据迁移 | Oracle/MySQL/PG 迁移到虚谷 | 2 |
| `/xugudb-vector` | 向量功能 | VECTOR/HALFVEC/SPARSEVEC、DiskANN 索引 | 4 |
| `/xugudb-spatial` | 空间数据库 | GIS 几何模型、300+ 空间函数、地图服务集成 | 4 |
| `/xugudb-jdbc` | Java JDBC | 连接池、CRUD、事务、批量操作、SSL | 5 |
| `/xugudb-python` | Python | xgcondb 驱动、参数化查询、游标 | 4 |
| `/xugudb-go` | Go | database/sql 接口、事务、大对象 | 3 |
| `/xugudb-csharp` | C# (.NET) | ADO.NET、XGConnection、DataSet | 3 |
| `/xugudb-php` | PHP | PDO 驱动、参数绑定、存储过程 | 3 |
| `/xugudb-nodejs` | Node.js | ODBC 桥接、Sequelize ORM 集成 | 2 |
| `/xugudb-odbc` | ODBC | DSN 配置、连接字符串、跨语言访问 | 3 |
| `/xugudb-c` | C 语言 | NCI/OCI/XGCI 三种 C 接口 | 3 |
| `/xugudb-ecosystem` | 生态集成 | MyBatis/Hibernate/GORM/Django/Sequelize 等 ORM | 2 |
| `/xugudb-tools` | 客户端工具 | XGConsole 命令行、DBeaver 集成 | 2 |
| `/xugudb-faq` | 常见问题 | FAQ + 完整错误码参考 | 4 |

---

## 其他 IDE 集成

本技能包的 Markdown 文件也可以作为知识库在其他工具中使用：

| 平台 | 集成方式 |
|------|----------|
| **Cursor** | 将 `skills/` 目录加入 Rules 或通过 `@file` 引用 |
| **GitHub Copilot** | 通过 `#file` 引用 SKILL.md 文件 |
| **Windsurf** | 加入项目上下文 |
| **Dify / Coze / FastGPT** | 将 references 目录作为 RAG 知识库导入 |
| **LangChain / LlamaIndex** | 作为文档加载器的数据源 |

---

## 目录结构

```
xugudb-dev-skills/
├── .claude-plugin/
│   └── plugin.json          # 插件元数据
├── skills/
│   ├── xugudb/              # 每个技能一个目录
│   │   ├── SKILL.md         # 技能定义（触发条件、概述）
│   │   └── references/      # 参考文档（详细内容）
│   ├── xugudb-sql/
│   ├── xugudb-distributed/
│   └── ...                  # 共 25 个技能
├── README.md
└── LICENSE
```

---

## 许可证

[Apache License 2.0](LICENSE)

---

## 贡献

欢迎提交 Issue 和 Pull Request。技能文档基于虚谷数据库 V12.9 / V13 官方文档整理。
