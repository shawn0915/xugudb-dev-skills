# 驱动安装与环境配置

## 驱动获取方式

### 方式一：产品包获取

Python 驱动压缩包默认在虚谷数据库产品包内，解压产品包后路径为：`Driver/python`。不同操作系统和 CPU 架构需要通过对应的产品包获取。

### 方式二：在线下载

| 操作系统 | CPU | 说明 |
|---------|-----|------|
| Windows | x86, 64-bit | 建议 Windows 10 及以上 |
| Linux | x86, 64-bit | 所有对应 CPU 架构的类 Unix 系统 |
| Linux | ARM, 64-bit | 所有对应 CPU 架构的类 Unix 系统 |

> 如需在其它系统上安装 Python 驱动，请联系虚谷。

## 安装步骤

### Windows 安装

Windows 二进制文件压缩包内容：

```
└── xgcondb
```

将解压文件中的 `xgcondb` 目录放置到项目启动文件同级目录中即可完成安装。

### Linux 安装

Linux 二进制文件压缩包内容：

```
├── libxugusql.so
└── xgcondb
```

安装步骤：
1. 将 `xgcondb` 目录放置到项目启动文件同级目录
2. 将 `libxugusql.so` 文件放置到系统动态库目录下（一般为 `/usr/lib64/` 或 `/usr/lib/`）

```bash
# Linux 安装示例
cp -r xgcondb /path/to/your/project/
cp libxugusql.so /usr/lib64/
ldconfig
```

## 支持的 Python 版本

xgcondb 包内整合了所有支持的 Python 版本二进制文件，切换 Python 版本无需切换驱动包：

- Python 2.7
- Python 3.4
- Python 3.6
- Python 3.7
- Python 3.8
- Python 3.9
- Python 3.10
- Python 3.11

## 验证安装

```python
import xgcondb

# 查看驱动版本
print(xgcondb.version())
# 输出示例：'xgcondb V2.3.7'
```

## 快速入门

```python
import xgcondb

# 1. 创建连接
conn = xgcondb.connect(
    host="127.0.0.1",
    port="5138",
    database="SYSTEM",
    user="SYSDBA",
    password="SYSDBA",
    charset='UTF8'
)

# 2. 创建游标
cur = conn.cursor()

# 3. 执行 SQL
cur.execute("CREATE TABLE test_python (id integer identity PRIMARY KEY, name varchar, age integer);")
cur.execute("INSERT INTO test_python (name, age) VALUES (?, ?)", ('test1', 18))

# 4. 查询数据
cur.execute("SELECT * FROM test_python;")
print(cur.fetchone())   # (1, 'test1', 18)

# 5. 关闭资源
cur.close()
conn.close()
```
