# 常见问题与解决

## ImportError: libxugusql.so: cannot open shared object file

**错误信息：**

```
ImportError: libxugusql.so: cannot open shared object file: No such file or directory
```

**原因：** 在 Linux/Unix 系统中使用 XuguDB Python 驱动时，未将 `libxugusql.so` 放到系统动态库目录。

**解决方法：**

将 `libxugusql.so` 放置到系统动态库文件读取目录下，一般为 `/usr/lib64/` 或 `/usr/lib/`：

```bash
# 复制动态库
cp libxugusql.so /usr/lib64/

# 刷新动态库缓存
ldconfig

# 验证
ldconfig -p | grep xugusql
```

也可以通过设置环境变量指定库路径：

```bash
export LD_LIBRARY_PATH=/path/to/libxugusql:$LD_LIBRARY_PATH
```

## ImportError: No module named 'xgcondb'

**原因：** `xgcondb` 目录未放置到项目启动文件同级目录中。

**解决方法：**

确保项目目录结构如下：

```
your_project/
├── your_script.py    # 你的 Python 脚本
└── xgcondb/          # 虚谷 Python 驱动包
```

或者将 xgcondb 所在目录添加到 Python 路径：

```python
import sys
sys.path.insert(0, '/path/to/xgcondb/parent/directory')
import xgcondb
```

## Python 版本兼容性

xgcondb 包内整合了所有支持的 Python 版本二进制文件（2.7, 3.4, 3.6-3.11），切换 Python 版本无需切换驱动包。

如果在不支持的 Python 版本上运行，请联系虚谷获取对应版本的驱动。

## 连接超时或连接失败

**排查步骤：**

1. 确认数据库服务已启动且端口可达：
   ```bash
   telnet <host> 5138
   ```

2. 确认连接参数正确（host、port、database、user、password）

3. 使用 `ping()` 方法检测连接状态：
   ```python
   if not conn.ping(True):
       print("连接已断开，重连失败")
   ```

4. 如需 SSL 连接，确保 `usessl` 参数设置正确：
   ```python
   conn = xgcondb.connect(..., usessl="true")
   ```

## 字符编码问题

默认客户端编码为 `utf8`。如果出现乱码，请检查 `charset` 参数：

```python
conn = xgcondb.connect(
    host="127.0.0.1", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA",
    charset='UTF8'  # 显式指定编码
)
```

## 游标线程安全问题

连接对象是线程安全的，但游标对象不是。在多线程环境中，应为每个线程创建独立的游标：

```python
import threading
import xgcondb

conn = xgcondb.connect(
    host="127.0.0.1", port="5138",
    database="SYSTEM", user="SYSDBA", password="SYSDBA"
)

def worker():
    cur = conn.cursor()  # 每个线程创建自己的游标
    try:
        cur.execute("SELECT * FROM my_table")
        rows = cur.fetchall()
        print(rows)
    finally:
        cur.close()

threads = [threading.Thread(target=worker) for _ in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()

conn.close()
```

## 事务未提交导致数据丢失

默认情况下连接不是自动提交模式。执行 INSERT/UPDATE/DELETE 后需要手动 `commit()`：

```python
conn.begin()
cur.execute("INSERT INTO t VALUES(1, 'data')")
conn.commit()  # 必须提交，否则数据不会持久化
```

或者设置自动提交：

```python
conn.autocommit(True)
```
