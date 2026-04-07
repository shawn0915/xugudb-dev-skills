# 虚谷数据库向量函数参考

XuguDB 提供 8 个向量函数，用于距离计算、范数、归一化、维度操作等。

## 距离计算函数

### l2_distance — 欧氏距离（L2 距离）

计算两个向量之间的欧氏距离，表示向量空间中两点之间的直线距离。

**语法**：
```sql
l2_distance(vec1, vec2)
```

**参数**：
- `vec1`, `vec2`：VECTOR、HALFVEC 或 SPARSEVEC 类型，维度必须一致

**返回**：DOUBLE 类型。任一参数为 NULL 则返回 NULL。

**示例**：
```sql
-- vector 类型
SELECT l2_distance('[3, 4]'::vector, '[0, 0]'::vector) AS distance;
-- 结果: 5.000000e+00

-- halfvec 类型
SELECT l2_distance('[1.5, 2.5, 3.5]'::halfvec, '[4.5, 5.5, 6.5]'::halfvec) AS distance;
-- 结果: 5.196152e+00

-- sparsevec 类型
SELECT l2_distance('{1:1, 2:3}/3'::sparsevec, '{1:4, 2:6}/3'::sparsevec) AS distance;
-- 结果: 4.242641e+00
```

---

### l1_distance — 曼哈顿距离（L1 距离）

计算两个向量之间的曼哈顿距离，即对应元素差值的绝对值之和。

**语法**：
```sql
l1_distance(vec1, vec2)
```

**参数**：
- `vec1`, `vec2`：VECTOR、HALFVEC 或 SPARSEVEC 类型，维度必须一致

**返回**：DOUBLE 类型。任一参数为 NULL 则返回 NULL。

**示例**：
```sql
-- vector 类型
SELECT l1_distance('[3, 4]'::vector, '[0, 0]'::vector) AS distance;
-- 结果: 7.000000e+00

-- halfvec 类型
SELECT l1_distance('[1.5, 2.5, 3.5]'::halfvec, '[4.5, 5.5, 6.5]'::halfvec) AS distance;
-- 结果: 9.000000e+00

-- sparsevec 类型
SELECT l1_distance('{1:1, 2:3}/3'::sparsevec, '{1:4, 2:6}/3'::sparsevec) AS distance;
-- 结果: 6.000000e+00
```

---

### cosine_distance — 余弦距离

计算两个向量之间的余弦距离，用于衡量向量方向的差异程度。取值范围 [0, 2]，0 表示方向完全相同，2 表示方向完全相反。

**语法**：
```sql
cosine_distance(vec1, vec2)
```

**参数**：
- `vec1`, `vec2`：VECTOR、HALFVEC 或 SPARSEVEC 类型，维度必须一致

**返回**：DOUBLE 类型。任一参数为 NULL 则返回 NULL。

**示例**：
```sql
-- vector 类型（同方向向量，余弦距离为 0）
SELECT cosine_distance('[3, 4]'::vector, '[6, 8]'::vector) AS distance;
-- 结果: 0.000000e+00

-- halfvec 类型
SELECT cosine_distance('[1, 2, 3]'::halfvec, '[4, 5, 6]'::halfvec) AS distance;
-- 结果: 2.536815e-02

-- sparsevec 类型
SELECT cosine_distance('{1:1, 3:2}/3'::sparsevec, '{1:2, 3:4}/3'::sparsevec) AS distance;
-- 结果: 0.000000e+00
```

---

### inner_product — 内积（点积）

计算两个向量的内积，即对应元素相乘后求和。

**语法**：
```sql
inner_product(vec1, vec2)
```

**参数**：
- `vec1`, `vec2`：VECTOR、HALFVEC 或 SPARSEVEC 类型，维度必须一致

**返回**：DOUBLE 类型。任一参数为 NULL 则返回 NULL。

**示例**：
```sql
-- vector 类型
SELECT inner_product('[3, 4]'::vector, '[5, 6]'::vector) AS product;
-- 结果: 3.900000e+01 (即 3*5 + 4*6 = 39)

-- halfvec 类型
SELECT inner_product('[1.5, 2.5, 3.5]'::halfvec, '[4.5, 5.5, 6.5]'::halfvec) AS product;
-- 结果: 4.325000e+01

-- sparsevec 类型
SELECT inner_product('{1:1, 2:3}/3'::sparsevec, '{1:4, 2:6}/3'::sparsevec) AS product;
-- 结果: 2.200000e+01
```

---

## 范数与归一化函数

### l2_norm — L2 范数（模长）

计算单个向量的欧氏范数（模长），即各元素平方和的平方根。常用于向量归一化、特征缩放。

**语法**：
```sql
l2_norm(vec)
```

**参数**：
- `vec`：VECTOR、HALFVEC 或 SPARSEVEC 类型

**返回**：DOUBLE 类型。输入为 NULL 则返回 NULL。零向量返回 0.0。

**说明**：
- 对于 SPARSEVEC，计算时仅需考虑非零元素（零元素平方后不影响结果）

**示例**：
```sql
-- vector 类型
SELECT l2_norm('[3, 4]'::vector) AS norm;
-- 结果: 5.000000e+00 (即 sqrt(9+16) = 5)

-- halfvec 类型
SELECT l2_norm('[1.5, 2.5, 3.5]'::halfvec) AS norm;
-- 结果: 4.555217e+00

-- sparsevec 类型
SELECT l2_norm('{1:1, 2:3}/3'::sparsevec) AS norm;
-- 结果: 3.162278e+00
```

---

### l2_normalize — L2 归一化

对向量进行 L2 归一化处理，将向量缩放至 L2 范数为 1 的单位向量，保持原向量方向不变。常用于机器学习特征预处理、余弦相似度计算等场景。

**语法**：
```sql
l2_normalize(vec)
```

**参数**：
- `vec`：VECTOR、HALFVEC 或 SPARSEVEC 类型

**返回**：与输入类型相同的向量。输入为 NULL 则返回 NULL。

**说明**：
- 输入向量应为非零向量（零向量归一化后仍为零向量，可能触发数值警告）
- 归一化后维度不变，仅各元素值按比例缩放
- 对于 SPARSEVEC，仅对非零元素缩放，零元素保持为 0

**示例**：
```sql
-- vector 类型
SELECT l2_normalize('[3, 4]'::vector) AS normalized_vec;
-- 结果: [0.6,0.8]

-- halfvec 类型
SELECT l2_normalize('[1.5, 2.5, 3.5]'::halfvec) AS normalized_vec;
-- 结果: [0.3293457,0.5488281,0.7685547]

-- sparsevec 类型
SELECT l2_normalize('{1:1, 2:3}/3'::sparsevec) AS normalized_vec;
-- 结果: {1:0.31622776,2:0.9486833}/3
```

---

## 维度操作函数

### vector_dims — 获取向量维度

获取向量的维度（元素数量），可用于验证向量维度是否符合预期。

**语法**：
```sql
vector_dims(vec)
```

**参数**：
- `vec`：VECTOR 或 HALFVEC 类型

**返回**：INTEGER 类型。输入为 NULL 则返回 NULL。

**注意**：不支持 SPARSEVEC 类型。

**示例**：
```sql
-- vector 类型
SELECT vector_dims('[3, 4, 5, 6]'::vector) AS dims;
-- 结果: 4

-- halfvec 类型
SELECT vector_dims('[1.5, 2.5, 3.5, 4.5, 5.5]'::halfvec) AS dims;
-- 结果: 5
```

---

### subvector — 提取子向量

从输入向量中提取指定维度范围的连续子向量。常用于维度裁剪、特征子集提取。

**语法**：
```sql
subvector(vec, start_dim, count)
```

**参数**：
- `vec`：VECTOR 或 HALFVEC 类型
- `start_dim`：INTEGER，子向量的起始维度（从 1 开始计数）
- `count`：INTEGER，子向量的长度（必须 >= 1）

**返回**：与输入类型相同的向量。任一参数为 NULL 则返回 NULL。

**说明**：
- `start_dim` 小于 1 时自动映射为 1；大于向量维度时报错
- 若请求长度超出剩余元素，自动裁剪至 `min(count, 向量维度 - start_dim + 1)`
- 不支持 SPARSEVEC 类型

**示例**：
```sql
-- vector 类型：从第 2 维开始提取 4 个元素
SELECT subvector('[10, 20, 30, 40, 50]'::vector, 2, 4) AS sub_vec;
-- 结果: [20,30,40,50]

-- halfvec 类型：从第 1 维开始提取 3 个元素
SELECT subvector('[1.1, 2.2, 3.3, 4.4, 5.5, 6.6]'::halfvec, 1, 3) AS sub_vec;
-- 结果: [1.1, 2.2, 3.3]
```

---

## 函数总览表

| 函数 | 适用类型 | 返回类型 | 说明 |
|------|----------|----------|------|
| `l2_distance(v1, v2)` | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 欧氏距离 |
| `l1_distance(v1, v2)` | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 曼哈顿距离 |
| `cosine_distance(v1, v2)` | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 余弦距离 [0,2] |
| `inner_product(v1, v2)` | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 内积（点积） |
| `l2_norm(v)` | VECTOR / HALFVEC / SPARSEVEC | DOUBLE | 欧氏范数 |
| `l2_normalize(v)` | VECTOR / HALFVEC / SPARSEVEC | 与输入同类型 | L2 归一化 |
| `vector_dims(v)` | VECTOR / HALFVEC | INTEGER | 维度数 |
| `subvector(v, start, count)` | VECTOR / HALFVEC | 与输入同类型 | 子向量提取 |

**通用约束**：
- 双参数距离函数要求两个向量维度一致，否则返回错误
- 任一参数为 NULL 时结果为 NULL
