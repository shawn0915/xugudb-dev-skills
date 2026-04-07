# 虚谷数据库向量索引参考（DiskANN）

XuguDB 当前支持 DiskANN 向量索引，基于 Microsoft Research 提出的图结构索引算法，利用磁盘存储索引结构，专为大规模向量集合设计。

## 一、检索方式概览

XuguDB 提供两种向量最近邻检索方式：

| 方式 | 说明 | 召回率 | 性能 |
|------|------|--------|------|
| **精确最近邻（ENN）** | 全量扫描，无需索引 | 100% | 随数据规模增长开销迅速增加 |
| **近似最近邻（ANN）** | 基于 DiskANN 索引 | 接近 100%（可调） | 大规模数据下显著优于全量扫描 |

查询规划器会根据代价模型自动选择执行路径。当索引查询成本较低时使用索引，否则仍执行精确检索。

## 二、创建索引

### 2.1 基本语法

```sql
CREATE INDEX index_name ON table_name (column_name distance_metric)
    INDEXTYPE IS DISKANN
    [WITH (param1=value1, param2=value2, ...)]
    [PARALLEL n];
```

**语法要素**：
- `column_name`：向量列名
- `distance_metric`：距离度量方式，格式为 `数据类型_距离度量_ops`
- `INDEXTYPE IS DISKANN`：指定索引类型（对向量类型列，不指定时默认创建 DiskANN 索引）
- `WITH (...)`：可选的构建参数
- `PARALLEL n`：可选的构建并行度

**约束**：
- 不支持多列索引
- 不支持函数形式的列
- 不支持降序列
- 不支持唯一索引、分区索引
- 表必须是堆表
- 同一向量字段不允许创建多个相同类型的索引

### 2.2 完整示例

```sql
-- 创建表
CREATE TABLE items (id INT, embedding VECTOR(3));

-- 创建 DiskANN 索引（L2 距离度量，自定义参数，20 并行度）
CREATE INDEX idx_vec ON items (embedding vector_l2_ops)
    INDEXTYPE IS DISKANN
    WITH (r=16, l=64)
    PARALLEL 20;

-- 简化写法：对向量列不指定索引类型时默认 DiskANN
CREATE INDEX idx_vec ON items (embedding vector_l2_ops);
```

### 2.3 数据类型与距离度量

| 数据类型 | 索引最大维度 | 支持的距离度量 |
|----------|-------------|---------------|
| VECTOR | 2000 | L2、IP、Cosine、L1 |
| HALFVEC | 4000 | L2、IP、Cosine、L1 |
| SPARSEVEC | 1000（非零维） | L2、IP、Cosine、L1 |

距离度量与 ops 名称对应：

| 距离度量 | 含义 | VECTOR ops | HALFVEC ops | SPARSEVEC ops |
|----------|------|-----------|-------------|---------------|
| L2 | 欧氏距离 | `vector_l2_ops` | `halfvec_l2_ops` | `sparsevec_l2_ops` |
| IP | 内积 | `vector_ip_ops` | `halfvec_ip_ops` | `sparsevec_ip_ops` |
| Cosine | 余弦距离 | `vector_cosine_ops` | `halfvec_cosine_ops` | `sparsevec_cosine_ops` |
| L1 | 曼哈顿距离 | `vector_l1_ops` | `halfvec_l1_ops` | `sparsevec_l1_ops` |

## 三、构建参数

通过 `WITH` 子句指定，控制索引图结构的构建过程和质量。

| 参数 | 类型 | 默认值 | 取值范围 | 说明 |
|------|------|--------|----------|------|
| `r` | 整数 | 16 | [4, 128] | 每个节点的最大邻居数。影响图的稠密度 |
| `l` | 整数 | 32 | [8, 512] | 构建阶段搜索宽度。影响邻居连接质量 |
| `maxc` | 整数 | 64 | [16, 2048] | 构建阶段保留的访问集大小。增大可改善长边连接 |
| `alpha` | 浮点 | 1.2 | [1.1, 4.0] | 邻居裁剪距离阈值。调大可保留更多节点 |

**参数调整指导**：
- 优先调整 `r` 和 `l`
- 增大这些参数可提升查询召回率和图质量，但会增加构建耗时、CPU、内存及磁盘占用
- `PARALLEL n`：控制内存构建方式下的并行度，可加快构建速度，但过高可能略微影响图质量
- 构建参数仅影响本次构建过程，不改变构建方式

**示例**：
```sql
-- 高召回率配置
CREATE INDEX idx_high_recall ON items (embedding vector_l2_ops)
    INDEXTYPE IS DISKANN
    WITH (r=64, l=128, maxc=256, alpha=1.5)
    PARALLEL 16;

-- 快速构建配置（默认参数）
CREATE INDEX idx_fast ON items (embedding vector_cosine_ops)
    INDEXTYPE IS DISKANN;
```

## 四、构建方式

DiskANN 索引采用两种构建方式，由系统自动选择：

| 构建方式 | 触发时机 | 行为 |
|----------|----------|------|
| **内存构建** | 创建索引或重建索引时 | 一次性将全部向量数据加载到内存，构建完整图结构后持久化到磁盘 |
| **磁盘构建** | 索引已存在，后续插入或更新数据时 | 增量方式直接更新磁盘上的索引结构 |

## 五、ANN 检索

### 5.1 查询语法

```sql
SELECT ...
FROM table_name
ORDER BY column_name vec_op const_vec
LIMIT k;
```

**向量距离操作符**（必须与索引的 `distance_metric` 匹配）：

| 操作符 | 距离度量 |
|--------|----------|
| `<->` | L2 距离 |
| `<#>` | IP 内积距离 |
| `<=>` | Cosine 余弦距离 |
| `<+>` | L1 曼哈顿距离 |

**注意**：
- `column_name` 和 `const_vec` 的位置可互换
- ORDER BY 必须使用升序排序
- 需开启系统参数 `use_index_order`（必须为 true）

**示例**：
```sql
-- 查询与目标向量 L2 距离最近的 5 条
SELECT * FROM items ORDER BY embedding <-> '[3,1,2]' LIMIT 5;

-- 余弦距离检索
SELECT * FROM items ORDER BY embedding <=> '[3,1,2]' LIMIT 10;

-- 内积检索
SELECT * FROM items ORDER BY embedding <#> '[3,1,2]' LIMIT 5;
```

### 5.2 LIMIT 与搜索宽度

`LIMIT k` 子句的行为：

- 搜索宽度 = `max(k, diskann_ef_search)`
- `diskann_ef_search`：会话参数，默认值 64
- 没有 LIMIT 子句时，搜索宽度等于 `diskann_ef_search`
- 搜索宽度越大，探索节点越多，召回率越高，但查询开销也越大
- k 较小时路径代价小；k 增大时需访问更多节点

**调整搜索宽度**：
```sql
-- 调大搜索宽度以提升召回率
SET diskann_ef_search = 200;
```

### 5.3 强制使用索引

如果查询优化器未选择索引，可使用 Hint 强制：

```sql
-- 使用 Hint 强制向量索引
SELECT /*+ INDEX(items idx_vec) */ *
FROM items
ORDER BY embedding <-> '[3,1,2]'
LIMIT 5;
```

## 六、索引维护

### 6.1 不可达点问题

DiskANN 索引图的边是有向边。如果某节点入度为 0 且不是入口点，则从入口点无法到达该节点，该节点在查询中永远不会被访问。大量不可达点会降低召回率。

**常见产生原因**：
- 使用较小的 `r` 值
- 大量增量插入或更新导致图局部结构破坏
- 数据分布不均

### 6.2 自动修复机制

**内存构建阶段**：在图结构持久化到磁盘前主动检测并修复不可达点。
- 补边：若不可达节点 x 离某节点 y 最近且 y 邻居未满，添加边 y -> x
- 设置额外入口点：若 y 邻居已满，将 x 标记为额外入口点

**动态更新阶段（图修复）**：
- 图修复在下一次查询执行前触发（非异步），由系统定时任务每 10 分钟检测触发条件
- 修复期间所有索引操作（查询、插入、更新等）被阻塞
- 主要工作：扫描索引、找出不可达点、加入为额外入口点

**触发条件**：
- 新增和更新数据累计达到阈值
- 修改后经过一定时间（查询活跃度越高触发越早，最长约一天）

**冷却期**：两小时。一次修复完成后至少等待两小时才允许下一次修复。

## 七、索引不生效排查

如果向量检索未走索引，按以下顺序排查：

1. **检查 `use_index_order` 参数**：必须为 true
2. **检查运算符匹配**：ORDER BY 使用的距离运算符必须与创建索引时的 `distance_metric` 一致
3. **检查排序方向**：ORDER BY 必须使用升序（ASC）
4. **查询优化器选择**：优化器可能因代价估算未选择索引，可使用 Hint 强制

## 八、常见问题

### 同一字段能否创建多个索引？
- 不允许创建多个相同类型的向量索引
- 允许创建多个不同类型的索引，优化器根据代价选择使用哪个

### ANN 结果数量少于 LIMIT？
DiskANN 只在有限搜索范围内查找，如果候选结果不足 LIMIT 数量，不会回退全表扫描补齐。

### 向量标量混合查询结果偏少？
索引先查找相似向量再应用标量过滤，候选结果有限或被过滤后不会补齐。

### 如何提升召回率？
- 调大会话参数 `diskann_ef_search`
- 重建索引时增大 `r` / `l` 参数
- 注意：参数过大会增加构建时间和查询开销，需合理调整
