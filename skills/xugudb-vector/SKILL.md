---
name: 虚谷数据库向量功能参考
name_for_command: xugudb-vector
description: |
  虚谷数据库向量功能完整参考：VECTOR/HALFVEC/SPARSEVEC 数据类型、向量函数、向量运算符、DiskANN 索引。
  涵盖向量存储、距离计算、相似性检索、ANN 近似最近邻查询等向量数据库核心能力。
---

# 虚谷数据库向量功能参考

XuguDB 向量数据库（V13 新增）提供对高维向量数据的高效存储与检索能力，将向量检索能力引入关系型数据库体系，支持语义检索、RAG、推荐系统等 AI 应用场景。

## 向量数据类型

XuguDB 提供 3 种向量数据类型：

| 类型 | 存储方式 | 精度 | 最大维度 | 格式示例 |
|------|----------|------|----------|----------|
| **VECTOR** | 32 位浮点稠密向量 | 高精度 | 16000 | `[1,2,3,4]` |
| **HALFVEC** | 16 位半精度稠密向量 | 半精度，节省约 50% 存储 | 16000 | `[1,2,3,4]` |
| **SPARSEVEC** | 仅存非零值及索引 | 高精度 | 10 亿 | `{1:1,3:2,6:5}/10` |

每种类型均支持指定维度（如 `VECTOR(3)`）或不指定维度（如 `VECTOR`）两种创建方式。

> 详细参考：[vector-types](skills/xugudb-vector/references/vector-types.md)

## 向量函数

| 函数 | 功能 | 适用类型 | 返回类型 |
|------|------|----------|----------|
| `l2_distance(v1, v2)` | 欧氏距离（L2） | VECTOR/HALFVEC/SPARSEVEC | DOUBLE |
| `l1_distance(v1, v2)` | 曼哈顿距离（L1） | VECTOR/HALFVEC/SPARSEVEC | DOUBLE |
| `cosine_distance(v1, v2)` | 余弦距离 [0,2] | VECTOR/HALFVEC/SPARSEVEC | DOUBLE |
| `inner_product(v1, v2)` | 内积（点积） | VECTOR/HALFVEC/SPARSEVEC | DOUBLE |
| `l2_norm(v)` | L2 范数（模长） | VECTOR/HALFVEC/SPARSEVEC | DOUBLE |
| `l2_normalize(v)` | L2 归一化为单位向量 | VECTOR/HALFVEC/SPARSEVEC | 与输入同类型 |
| `vector_dims(v)` | 获取向量维度数 | VECTOR/HALFVEC | INTEGER |
| `subvector(v, start, count)` | 提取子向量 | VECTOR/HALFVEC | 与输入同类型 |

> 详细参考：[vector-functions](skills/xugudb-vector/references/vector-functions.md)

## 向量运算符

### 距离运算符（用于相似性检索）

| 运算符 | 功能 | 适用类型 | 返回类型 | 对应索引度量 |
|--------|------|----------|----------|-------------|
| `<->` | L2 欧氏距离 | VECTOR/HALFVEC/SPARSEVEC | DOUBLE | `vector_l2_ops` |
| `<#>` | 内积距离（取负） | VECTOR/HALFVEC/SPARSEVEC | DOUBLE | `vector_ip_ops` |
| `<=>` | 余弦距离 | VECTOR/HALFVEC/SPARSEVEC | DOUBLE | `vector_cosine_ops` |
| `<+>` | L1 曼哈顿距离 | VECTOR/HALFVEC/SPARSEVEC | DOUBLE | `vector_l1_ops` |

### 算术运算符

| 运算符 | 功能 | 适用类型 | 返回类型 |
|--------|------|----------|----------|
| `+` | 逐维度加法 | VECTOR/HALFVEC | 与输入同类型 |
| `-` | 逐维度减法 | VECTOR/HALFVEC | 与输入同类型 |
| `*` | 逐维度乘法 | VECTOR/HALFVEC | 与输入同类型 |
| `\|\|` | 向量拼接 | VECTOR/HALFVEC | 与输入同类型 |

### 比较运算符

| 运算符 | 功能 | 适用类型 | 返回类型 |
|--------|------|----------|----------|
| `=` | 逐维度相等 | VECTOR/HALFVEC/SPARSEVEC | BOOLEAN |
| `<>` | 逐维度不等 | VECTOR/HALFVEC/SPARSEVEC | BOOLEAN |
| `<` / `>` / `<=` / `>=` | 字典序比较 | VECTOR/HALFVEC/SPARSEVEC | BOOLEAN |

> 详细参考：[vector-operators](skills/xugudb-vector/references/vector-operators.md)

## DiskANN 索引

XuguDB 支持 DiskANN 向量索引，用于大规模数据集的近似最近邻（ANN）检索。

- **创建语法**：`CREATE INDEX idx ON tbl(col distance_metric) INDEXTYPE IS DISKANN [WITH (r=16, l=64)] [PARALLEL n];`
- **距离度量**：`vector_l2_ops` / `vector_ip_ops` / `vector_cosine_ops` / `vector_l1_ops`
- **构建参数**：`r`（最大邻居数）、`l`（搜索宽度）、`maxc`（访问集大小）、`alpha`（裁剪阈值）
- **查询方式**：`SELECT ... ORDER BY col <-> target_vec LIMIT k;`
- **查询参数**：会话参数 `diskann_ef_search`（默认 64）控制搜索宽度
- **前提条件**：需开启系统参数 `use_index_order`

> 详细参考：[vector-indexes](skills/xugudb-vector/references/vector-indexes.md)

## 快速上手示例

```sql
-- 1. 创建带向量列的表
CREATE TABLE product_features (
    id INT PRIMARY KEY,
    product_name TEXT,
    feature_vector VECTOR(3)
);

-- 2. 插入向量数据
INSERT INTO product_features (id, product_name, feature_vector) VALUES
    (1, '智能手机A', '[1.2, 3.4, 5.6]'),
    (2, '智能手机B', '[1.3, 3.5, 5.7]'),
    (3, '平板电脑A', '[7.8, 8.9, 9.0]'),
    (4, '笔记本电脑A', '[2.1, 4.3, 6.5]');

-- 3. 精确最近邻检索（全量扫描）
SELECT id, product_name, feature_vector <-> '[1.1, 3.3, 5.5]'::vector AS distance
FROM product_features
ORDER BY feature_vector <-> '[1.1, 3.3, 5.5]'::vector
LIMIT 2;

-- 4. 创建 DiskANN 索引加速检索
CREATE INDEX idx ON product_features(feature_vector VECTOR_L2_OPS) INDEXTYPE IS DISKANN;

-- 5. ANN 近似最近邻检索（自动走索引）
SELECT id, product_name, feature_vector <-> '[1.1, 3.3, 5.5]'::vector AS distance
FROM product_features
ORDER BY feature_vector <-> '[1.1, 3.3, 5.5]'::vector
LIMIT 2;
```

## 常见问题

- **索引不生效**：检查 `use_index_order` 参数是否为 true；ORDER BY 必须升序且运算符与索引度量一致
- **召回率不足**：调大 `diskann_ef_search` 会话参数；重建索引时调大 `r` / `l` 参数
- **结果数少于 LIMIT**：DiskANN 为近似索引，搜索范围有限时不会回退全表扫描补齐
- **同一字段多索引**：不允许多个相同类型索引，但允许不同类型索引共存

## 应用场景

- **RAG 搜索增强生成**：向量化存储知识库，语义检索后作为 LLM 上下文
- **个性化推荐**：用户/物品特征向量相似度计算与排序
- **图搜图/文本搜图**：图像/文本特征向量的相似度检索
- **向量标量混合查询**：结合 WHERE 条件过滤与向量相似度排序
