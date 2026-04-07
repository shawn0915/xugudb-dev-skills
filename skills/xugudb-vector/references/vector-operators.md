# 虚谷数据库向量运算符参考

XuguDB 提供 14 个向量运算符，分为距离运算符、算术运算符和比较运算符三类。

## 一、距离运算符

距离运算符用于计算向量间的距离或相似度，是向量检索（KNN / ANN）的核心。结合 `ORDER BY ... LIMIT k` 可实现 Top-K 最近邻查询。

### `<->` — L2 欧氏距离

计算两个向量之间的欧氏距离（Euclidean Distance），值越小表示越相似。

**适用类型**：VECTOR / HALFVEC / SPARSEVEC（左右类型须相同）
**返回类型**：DOUBLE，取值 [0, +inf)
**对应索引度量**：`vector_l2_ops`

```sql
-- 基本用法
SELECT '[1, 2, 3]'::vector <-> '[4, 5, 6]'::vector AS euclidean_distance;
-- 结果: 5.196152e+00

-- halfvec 类型
SELECT '[0.1, 0.2, 0.3]'::halfvec <-> '[0.4, 0.5, 0.6]'::halfvec AS euclidean_distance;
-- 结果: 5.196293e-01

-- sparsevec 类型（未出现维度默认为 0 并参与计算）
SELECT '{1:1, 2:3}/4'::sparsevec <-> '{1:4, 2:6}/4'::sparsevec AS euclidean_distance;
-- 结果: 4.242641e+00

-- KNN 检索：返回最相似的前 2 条记录
SELECT id, product_name, feature_vector <-> '[1.1, 3.3, 5.5]'::vector AS distance
FROM product_features
ORDER BY distance ASC
LIMIT 2;
```

---

### `<#>` — 内积距离

计算两个向量之间的内积（点积）的负值。返回值为内积取负，因此值越小表示内积越大（越相似）。

**适用类型**：VECTOR / HALFVEC / SPARSEVEC（左右类型须相同）
**返回类型**：DOUBLE
**对应索引度量**：`vector_ip_ops`

**输出含义**：
- 正值：两向量在相反方向有投影重叠
- 零值：两向量相互垂直
- 负值：两向量在相同方向有投影重叠（值越小越相似）

```sql
-- vector 类型
SELECT '[1, 2, 3]'::vector <#> '[4, 5, 6]'::vector AS inner_product;
-- 结果: -3.200000e+01 (即 -(1*4+2*5+3*6) = -32)

-- halfvec 类型
SELECT '[0.1, 0.2, 0.3]'::halfvec <#> '[0.4, 0.5, 0.6]'::halfvec AS inner_product;
-- 结果: -3.200147e-01

-- sparsevec 类型（仅对双方均非零的维度计算）
SELECT '{1:1, 2:3, 4:5}/5'::sparsevec <#> '{1:4, 2:5, 3:6}/5'::sparsevec AS inner_product;
-- 结果: -1.900000e+01

-- KNN 检索
SELECT id, username, preference_vector <#> '[3, 4, 5]'::vector AS similarity
FROM user_preferences
ORDER BY similarity ASC
LIMIT 2;
```

---

### `<=>` — 余弦距离

计算两个向量之间的余弦距离，衡量方向的相似性，忽略模长。取值 [0, 2]，0 表示方向完全相同，2 表示完全相反。

**适用类型**：VECTOR / HALFVEC / SPARSEVEC（左右类型须相同）
**返回类型**：DOUBLE，取值 [0, 2]
**对应索引度量**：`vector_cosine_ops`

```sql
-- vector 类型
SELECT '[1, 2, 3]'::vector <=> '[4, 5, 6]'::vector AS cosine_similarity;
-- 结果: 2.536815e-02

-- halfvec 类型
SELECT '[0.1, 0.2, 0.3]'::halfvec <=> '[0.4, 0.5, 0.6]'::halfvec AS cosine_similarity;
-- 结果: 2.536527e-02

-- sparsevec 类型
SELECT '{1:1, 2:3}/4'::sparsevec <=> '{1:4, 2:6}/4'::sparsevec AS cosine_similarity;
-- 结果: 3.523618e-02

-- KNN 检索：余弦距离最小的前 3 条
SELECT id, content, embedding <=> '[2, 3, 4]'::vector AS similarity
FROM documents
ORDER BY similarity ASC
LIMIT 3;
```

---

### `<+>` — L1 曼哈顿距离

计算两个向量之间的 L1 距离（曼哈顿距离），即对应维度值绝对差之和。

**适用类型**：VECTOR / HALFVEC / SPARSEVEC（左右类型须相同）
**返回类型**：DOUBLE，取值 [0, +inf)
**对应索引度量**：`vector_l1_ops`

**SPARSEVEC 特殊行为**：允许维度索引不完全重叠，缺失维度按 0 处理。

```sql
-- vector 类型
SELECT '[1.2, 3.4, 5.6]'::vector <+> '[1.0, 3.6, 5.8]'::vector AS l1_distance;
-- 结果: 6.000001e-01

-- halfvec 类型
SELECT '[0.8, 1.6, 2.4]'::halfvec <+> '[0.9, 1.5, 2.5]'::halfvec AS l1_distance;
-- 结果: 2.993164e-01

-- sparsevec 类型
SELECT '{1:2.0, 3:4.0, 5:6.0}/6'::sparsevec <+> '{1:1.5, 2:3.0, 5:6.5}/6'::sparsevec AS l1_distance;
-- 结果: 8.000000e+00
```

---

## 二、算术运算符

算术运算符对向量进行逐维度运算或拼接，仅支持 VECTOR 和 HALFVEC 类型。

### `+` — 向量加法

逐维度加法，维度必须一致。

**适用类型**：VECTOR / HALFVEC
**返回类型**：与输入同类型

```sql
-- vector 加法
SELECT '[1, 2, 3]'::vector + '[4, 5, 6]'::vector AS vector_sum;
-- 结果: [5,7,9]

-- halfvec 加法
SELECT '[0.1, 0.2, 0.3]'::halfvec + '[0.4, 0.5, 0.6]'::halfvec AS vector_sum;
-- 结果: [0.5,0.7001953,0.9003906]

-- 特征融合示例
SELECT id, title_embedding + content_embedding AS fused_embedding
FROM article_features;
```

---

### `-` — 向量减法

逐维度减法，维度必须一致。结果中即使为 0 也会显式存储。

**适用类型**：VECTOR / HALFVEC
**返回类型**：与输入同类型

```sql
-- vector 减法
SELECT '[2.5, 3.8, 1.2]'::vector - '[1.1, 2.0, 0.5]'::vector AS vector_diff;
-- 结果: [1.4,1.8,0.70000005]

-- halfvec 减法
SELECT '[0.3, 0.6, 0.9]'::halfvec - '[0.1, 0.2, 0.3]'::halfvec AS halfvec_diff;
-- 结果: [0.20007324,0.40014648,0.5996094]

-- 从综合向量中剥离标题特征
SELECT id, combined_vec - title_vec AS content_vec
FROM news_vectors;
```

---

### `*` — 向量乘法

逐维度乘法（Hadamard 积），维度必须一致。

**适用类型**：VECTOR / HALFVEC
**返回类型**：与输入同类型

```sql
-- vector 逐元素乘法（特征加权）
SELECT '[5, 3, 4]'::vector * '[0.2, 0.5, 0.3]'::vector AS weighted;
-- 结果: [1,1.5,1.2]

-- halfvec 逐元素乘法
SELECT '[5, 3, 4]'::halfvec * '[0.2, 0.5, 0.3]'::halfvec AS weighted;
-- 结果: [1,1.5,1.2001953]

-- 在查询中使用
SELECT id, raw_features * weight_vector AS weighted_features
FROM feature_weights;
```

---

### `||` — 向量拼接

将两个向量首尾拼接为一个新向量，新向量维度等于两个输入向量维度之和。不要求维度一致。

**适用类型**：VECTOR / HALFVEC
**返回类型**：与输入同类型

```sql
-- vector 拼接（2维 + 3维 = 5维）
SELECT '[1.2, 3.4]'::vector || '[5.6, 2.8, 4.0]'::vector AS concatenated;
-- 结果: [1.2,3.4,5.6,2.8,4]

-- halfvec 拼接
SELECT '[0.8, 1.6]'::halfvec || '[2.4, 3.2]'::halfvec AS concatenated;
-- 结果: [0.8, 1.6, 2.4, 3.2]
```

**注意**：拼接顺序影响结果，`a || b` 与 `b || a` 生成的向量维度顺序不同。

---

## 三、比较运算符

比较运算符用于向量的相等性判断和字典序比较，支持 VECTOR / HALFVEC / SPARSEVEC 三种类型，返回 BOOLEAN。

### `=` — 相等比较

逐维度判断所有维度值是否完全一致。

```sql
SELECT '[1.2, 3.4, 5.6]'::vector = '[1.2, 3.4, 5.6]'::vector AS result;
-- 结果: T

-- halfvec 类型基于半精度表示后的值比较
SELECT '[0.8, 1.6, 2.4]'::halfvec = '[0.8000000119, 1.6, 2.4]'::halfvec AS result;
-- 结果: T

-- sparsevec 维度总长度不同时不相等
SELECT '{1:2.0, 3:4.0}/5'::sparsevec = '{1:2.0, 3:4.0}/6'::sparsevec AS result;
-- 结果: F
```

---

### `<>` — 不等比较

判断两个向量是否不完全一致，任一维度不等即返回 true。

```sql
SELECT '[1.2, 3.4, 5.6]'::vector <> '[1.2, 3.5, 5.6]'::vector AS result;
-- 结果: T

SELECT '[0.1, 0.2]'::halfvec <> '[0.1, 0.2]'::halfvec AS result;
-- 结果: F

-- sparsevec 索引位置不同
SELECT '{1:2.0, 3:4.0}/4'::sparsevec <> '{1:2.0, 4:4.0}/4'::sparsevec AS result;
-- 结果: T
```

---

### `<` — 小于（字典序）

从左到右逐维度比较，遇到第一个不相等维度时判断左值是否小于右值。所有维度相等返回 false。

```sql
SELECT '[1.2, 3.4, 5.6]'::vector < '[2.1, 3.0, 6.0]'::vector AS result;
-- 结果: T (第一维 1.2 < 2.1，直接返回)

-- 结合 WHERE 子句
SELECT id, sensor_id, value_vec
FROM sensor_data
WHERE (value_vec < '[2.0, 4.0]'::vector) = true;
```

---

### `>` — 大于（字典序）

从左到右逐维度比较，遇到第一个不相等维度时判断左值是否大于右值。所有维度相等返回 false。

```sql
SELECT '[1.5, 3.6, 5.8]'::vector > '[1.2, 3.5, 6.0]'::vector AS result;
-- 结果: T (第一维 1.5 > 1.2，直接返回)

-- sparsevec 类型，未存储维度按 0 处理
SELECT '{1:2.5, 3:4.5}/5'::sparsevec > '{1:2.0, 3:4.0, 5:0.5}/6'::sparsevec AS result;
-- 结果: T
```

---

### `<=` — 小于等于（字典序）

字典序小于等于比较。所有维度相等时返回 true。

```sql
SELECT '[1.2, 3.4, 5.6]'::vector <= '[1.2, 3.5, 5.6]'::vector AS result;
-- 结果: T

SELECT '[0.8, 1.6, 2.4]'::halfvec <= '[0.8, 1.6, 2.4]'::halfvec AS result;
-- 结果: T
```

---

### `>=` — 大于等于（字典序）

字典序大于等于比较。所有维度相等时返回 true。

```sql
SELECT '[2.0, 3.5, 5.8]'::vector >= '[2.0, 3.4, 6.0]'::vector AS result;
-- 结果: T (第二维 3.5 > 3.4，直接返回)

SELECT '[0.8, 1.6, 2.4]'::halfvec >= '[0.8, 1.6, 2.4]'::halfvec AS result;
-- 结果: T
```

---

## 运算符总览表

| 运算符 | 功能 | 适用类型 | 返回类型 | 维度要求 |
|--------|------|----------|----------|----------|
| `<->` | L2 欧氏距离 | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 一致 |
| `<#>` | 内积距离（取负） | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 一致 |
| `<=>` | 余弦距离 | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 一致 |
| `<+>` | L1 曼哈顿距离 | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 一致 |
| `+` | 逐维度加法 | VECTOR / HALFVEC | 与输入同类型 | 一致 |
| `-` | 逐维度减法 | VECTOR / HALFVEC | 与输入同类型 | 一致 |
| `*` | 逐维度乘法 | VECTOR / HALFVEC | 与输入同类型 | 一致 |
| `\|\|` | 向量拼接 | VECTOR / HALFVEC | 与输入同类型 | 无要求 |
| `=` | 相等比较 | VECTOR / HALFVEC / SPARSEVEC | BOOLEAN | 一致 |
| `<>` | 不等比较 | VECTOR / HALFVEC / SPARSEVEC | BOOLEAN | 一致 |
| `<` | 小于（字典序） | VECTOR / HALFVEC / SPARSEVEC | BOOLEAN | 允许不一致 |
| `>` | 大于（字典序） | VECTOR / HALFVEC / SPARSEVEC | BOOLEAN | 允许不一致 |
| `<=` | 小于等于（字典序） | VECTOR / HALFVEC / SPARSEVEC | BOOLEAN | 允许不一致 |
| `>=` | 大于等于（字典序） | VECTOR / HALFVEC / SPARSEVEC | BOOLEAN | 允许不一致 |

## 距离运算符与索引度量对应关系

在创建 DiskANN 索引和执行 ANN 检索时，距离运算符必须与索引的距离度量一致：

| 运算符 | 距离度量名称 | 索引 ops 名称 |
|--------|-------------|--------------|
| `<->` | L2 欧氏距离 | `vector_l2_ops` / `halfvec_l2_ops` / `sparsevec_l2_ops` |
| `<#>` | 内积 | `vector_ip_ops` / `halfvec_ip_ops` / `sparsevec_ip_ops` |
| `<=>` | 余弦距离 | `vector_cosine_ops` / `halfvec_cosine_ops` / `sparsevec_cosine_ops` |
| `<+>` | L1 曼哈顿距离 | `vector_l1_ops` / `halfvec_l1_ops` / `sparsevec_l1_ops` |
