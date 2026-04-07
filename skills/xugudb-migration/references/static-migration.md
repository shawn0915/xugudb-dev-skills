# 静态数据迁移

静态数据迁移是指将以某种结构组织存储的静态数据（数据源），转移到虚谷数据库的过程。

**特征：**

| 特征 | 说明 |
|------|------|
| 数据不变动 | 源数据是静态的，如历史表、归档表、旧版本数据等 |
| 具有一定的结构 | 每行代表一个记录，每列代表一个字段，字段间和行间各有分隔符 |
| 一次性迁移 | 不需要实时同步，只需一次性完整复制 |
| 可择时执行 | 通常在业务低峰期执行 |
| 迁移后不回写 | 目标库数据不再回写到源数据 |
| 数据一致性强 | 数据静态，不需要考虑同步冲突 |

---

## 一、TXT 文件

特指由固定记录分隔符与字段分隔符组织文本数据的文件。

通过虚谷控制台（XGConsole）的 `sqlldr` 或 `tabldr` 命令导入：

- 记录分隔符通过 `rt` 参数指定，如 `rt=X'0d0a'`（即 `\r\n`）
- 字段分隔符通过 `ft` 参数指定，如 `ft=','`
- 不启用 `csv_deal` 参数

```sql
-- sqlldr 方式
SQL> sqlldr table=tab_region datafile=/tmp/region.txt ft='|' rt=X'0a' log=/tmp/load.log errlog=/tmp/loaderr.log mode=replace imp_type=block

-- tabldr 方式（多线程）
SQL> tabldr table=tab_region datafile=/tmp/region.txt ft='|' rt=X'0a' log=/tmp/load.log errlog=/tmp/loaderr.log parse_para=2 ins_para=2 mode=replace
```

---

## 二、CSV 文件

逗号分隔值格式，需启用 `csv_deal` 参数，且文件后缀为 `.csv` 或 `.CSV`。

CSV 格式规则：
- 记录分隔符通过 `rt` 参数指定
- 字段分隔符通过 `ft` 参数指定
- 字段可用 `"` 双引号包裹，包含的分隔符作为字段值保留
- 双引号本身需用 `""` 转义

```sql
-- sqlldr 方式
SQL> sqlldr table=tab_region datafile=/tmp/region.csv ft='|' rt=X'0a' log=/tmp/load.log errlog=/tmp/loaderr.log mode=replace imp_type=block csv_deal=true

-- tabldr 方式
SQL> tabldr table=tab_region datafile=/tmp/region.csv ft='|' rt=X'0a' log=/tmp/load.log errlog=/tmp/loaderr.log parse_para=2 ins_para=2 mode=replace csv_deal=true
```

---

## 三、SQL 文件

通过虚谷控制台 `execfile` 命令执行 SQL 文件中的语句。数据通常以 INSERT 语句形式存储，语法需符合 SQL89/92/99 标准或虚谷语法标准。

```sql
SQL> execfile D:/consel.sql
```

> **注意：** 异构数据库导出的 SQL 文件可能存在不兼容语法，需根据实际情况修改调整。

---

## 四、虚谷备份文件

虚谷数据库备份功能生成的备份文件，仅能由虚谷数据库解析恢复。

| 备份方式 | 说明 |
|---------|------|
| 物理备份 | 全量备份、增量备份 |
| 逻辑备份 | 库级备份、用户级备份、模式级备份、表级备份 |

详细操作请参阅备份恢复文档。

---

## 五、虚谷实例目录

直接复制虚谷数据库部署目录实现迁移。

### 单机目录

1. 停止源目录运行的虚谷数据库单机实例
2. 将单机部署目录中所有文件完整拷贝至目标位置（XGLOG 日志目录可忽略）
3. 修改 `SETUP/xugu.ini` 系统配置文件
4. 启动目标虚谷数据库单机实例

### 集群节点目录

1. 停止集群各节点实例
2. 将集群各节点部署目录完整拷贝至目标位置（XGLOG 日志目录可忽略）
3. 修改 `SETUP/cluster.ini` 集群配置文件
4. 修改 `SETUP/xugu.ini` 系统配置文件
5. 启动目标虚谷数据库集群实例

---

## 六、数据库迁移

支持同构（虚谷）和异构（Oracle/PostgreSQL/MySQL）数据库作为数据源。

### 6.1 通过 TXT 文件（同构）

```sql
-- 1. 导出 TXT 数据
SQL> SELECT * FROM tab_region; >$ /tmp/region.txt;

-- 2. 导入到目标数据库（参照上文 TXT 文件导入方式）
```

> **注意：** Linux 控制台导出以 `\n` 为行分隔符；Windows V2.1.1 之前以 `\r\n`，之后以 `\n`。

### 6.2 通过 CSV 文件（同构/异构）

**同构数据库：**
```sql
-- 导出 CSV 数据
SQL> SELECT * FROM tab_region; >$ /tmp/region.csv;
-- 导入到目标数据库（参照上文 CSV 文件导入方式）
```

**异构数据库：**
1. 使用 DBeaver 连接数据源，导出 CSV 格式数据
2. 使用虚谷控制台导入 CSV 文件

> **注意：** 非虚谷工具生成的 CSV 格式可能无法直接识别，可能需要调整格式。

### 6.3 通过 SQL 文件（同构/异构）

1. 使用 DBeaver 连接数据源，导出 SQL 格式数据
2. 使用虚谷控制台 `execfile` 命令执行

> **注意：** 异构数据库的 SQL 语法可能不兼容，需根据实际情况修改。

### 6.4 通过虚谷备份文件（仅同构）

参阅上文虚谷备份文件部分。

### 6.5 通过虚谷实例目录（仅同构）

参阅上文虚谷实例目录部分。
