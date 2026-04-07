---
name: PL/SQL 基础语法
description: |
  虚谷数据库 PL/SQL 基础语法：程序块结构、标识符、数据类型（标量/复合/属性类型）、
  变量声明与赋值、控制结构、游标、异常处理、动态 SQL、批量操作。
---

# PL/SQL 基础语法

## 一、程序块结构

虚谷 PL/SQL 程序由三部分组成：声明部分、可执行部分、异常处理部分。

```sql
DECLARE   -- 声明部分（可选）
  <declarations section>
BEGIN     -- 可执行部分
  <executable command(s)>
EXCEPTION -- 异常处理部分（可选）
  <exception handling>
END;
```

**基本示例：**

```sql
SQL> DECLARE
  str VARCHAR;
BEGIN
  NULL;
END;
/
```

> **提示**：在 xgconsole 控制台执行时，匿名语句块最后需要添加 `/` 标识结束，在其他管理工具执行时无需添加。存在 `BEGIN` 时必须对应一个 `END` 标识结束。

### PL/SQL 程序单元

PL/SQL 单元包括以下任意一种：
- **匿名块**：直接以 DECLARE 或 BEGIN 开始的程序块
- **存储过程**
- **存储函数**
- **触发器**
- **包**：包含包头和包体
- **自定义类型**：包含类型头和类型体

## 二、标识符

### 命名规则

- **合法字符**：字母（A-Z、a-z）、数字（0-9）、下划线（_）、美元符号（$）、中文
- **首字符限制**：必须以字母、中文或特殊字符开头，不能是数字
- **长度限制**：最多 127 个字节
- **大小写敏感性**：默认不区分大小写（如 `EMP_ID` 和 `emp_id` 视为相同），但使用双引号可强制区分
- **保留字冲突**：不能使用 PL/SQL 保留字（如 SELECT、IF、BEGIN 等），除非加双引号（不推荐）

### 作用域与可见性

- **全局变量**：在包头中声明，整个包内可见
- **局部变量**：在存储过程、函数或匿名块中声明，仅在块内可见
- **嵌套块**：内部块可访问外部块的变量，但外部块无法访问内部块的变量

## 三、分隔符

| 分隔符 | 描述 |
|--------|------|
| `+` `-` `*` `/` | 加法（正号）、减法（负号）、乘法、除法 |
| `%` | 属性指示符 |
| `'` | 字符串定界符 |
| `.` | 组件选择器 |
| `(` `)` | 表达式或列表分隔符 |
| `,` | 项目分隔符 |
| `"` | 带引号的标识符分隔符 |
| `@` | 远程访问指示符 |
| `;` | 语句终止符 |
| `:=` | 赋值运算符 |
| `=>` | 关联运算符 |
| `\|\|` | 连接运算符 |
| `<<` `>>` | 标签分隔符 |
| `/*` `*/` | 多行注释分隔符 |
| `--` | 单行注释指示符 |
| `..` | 范围运算符 |
| `<` `>` `<=` `>=` `=` | 关系运算符 |
| `<>` `!=` | 不等于运算符 |

## 四、注释

```sql
SQL> DECLARE
  -- 单行注释
  str VARCHAR;
BEGIN
  /*
   * 多行注释
   */
  NULL;
END;
/
```

## 五、数据类型

### 5.1 标量类型

标量类型是最简单的数据类型，用于存储单个值，包括数值类型、字符类型、日期类型、布尔类型等基础数据类型。

```sql
SQL> DECLARE
  id INT;
  first_name VARCHAR;
  last_name VARCHAR;
BEGIN
  id := 1;
  first_name := '张';
  last_name := '三';
  SEND_MSG(id || ':' || first_name || last_name);
END;
/
-- 输出 1:张三
```

### 5.2 子类型（SUBTYPE）

子类型是其基类型的特定子集，支持基类型的所有操作，但只允许更小范围的值。

**语法：** `SUBTYPE ColumnName IS TypeName`

```sql
SQL> DECLARE
  SUBTYPE id_num IS NUMERIC(5,2);
  SUBTYPE var_name IS VARCHAR(10);
  id id_num;
  var var_name;
BEGIN
  id := 2;
  var := '李四';
  SEND_MSG(id || ':' || var);
END;
/
-- 输出 2:李四
```

### 5.3 复合类型

#### 5.3.1 记录类型（RECORD）

记录类型由多种不同基本数据类型或自定义类型的元素组合而成，使用点号 `.` 访问字段成员。

**语法：**

```
TYPE rec_name IS RECORD (
  ColumnName type_name,
  ...
)
```

**初始化方法：**

1. 声明记录类型变量，自动初始化为 EMPTY
2. 使用 `%ROWTYPE` 或 `%TYPE` 与表字段类型关联
3. 使用游标的 `%ROWTYPE` 关联
4. 使用构造函数初始化

```sql
SQL> DECLARE
  TYPE t_rec IS RECORD(
    id NUMERIC,
    name VARCHAR2,
    birthday DATE
  );
  var_rec T_REC;
BEGIN
  var_rec := T_REC(10, 'David', '2025-06-19');
  DBMS_OUTPUT.PUT_LINE('Person Info: (' || var_rec.ID || ', '
    || var_rec.NAME || ', ' || var_rec.BIRTHDAY || ')');
END;
/
-- 输出 Person Info: (10, David, 2025-06-19 00:00:00)
```

**RECORD 与表类型结合使用：**

```sql
SQL> DECLARE
  TYPE type_record IS RECORD (c1 INT, c2 VARCHAR);
  v1 type_record;
  TYPE type_tab_record IS TABLE OF type_record;
  v_tab type_tab_record;
BEGIN
  v_tab := type_tab_record();
  v_tab.EXTEND(1);
  v1 := type_record(1, 'abc');
  v_tab(1) := v1;
  INSERT INTO record_tab VALUES(v_tab(1).c1, v_tab(1).c2);
END;
/
```

> **提示**：不支持为 record 数据添加新成员或删除已有成员。

#### 5.3.2 集合类型

虚谷提供三种集合类型：

| 属性 | 变长数组（VARRAY） | 嵌套表（TABLE） | 索引表（ITABLE） |
|------|-------------------|-----------------|-----------------|
| 可用于SQL | 可用 | 可用 | 不可用 |
| 表字段类型 | 可用 | 可用 | 不可用 |
| 初始化 | 声明时自动完成 | 声明时自动完成 | 声明时自动完成 |
| 是否有界 | 有 size 界 | 可以扩展 | 无界 |
| 索引类型 | 整数 | 整数 | 整数或字符串 |
| 密集或稀疏 | 总是密集的 | 开始密集，删除中间值变得稀疏 | 稀疏 |

**使用集合类型三步骤：** 1. 声明 -> 2. 初始化 -> 3. 赋值

**变长数组（VARRAY）语法：**

```
TYPE type_name IS VARRAY(size) OF type_x
```

```sql
SQL> DECLARE
  TYPE players IS VARRAY(5) OF VARCHAR(20);
  team players := players('Zhangsan', 'Lisi', 'Wangwu', 'Mazi', 'Chouchong');
BEGIN
  FOR i IN 1..5 LOOP
    DBMS_OUTPUT.PUT_LINE(i || '.' || team(i));
  END LOOP;
END;
/
```

**嵌套表（TABLE）语法：**

```
TYPE type_name IS TABLE OF type_x
```

```sql
SQL> DECLARE
  TYPE tab_type IS TABLE OF VARCHAR(20);
  v_names tab_type := tab_type('Alice', 'Bob', 'Charlie');
BEGIN
  FOR i IN v_names.FIRST .. v_names.LAST LOOP
    SEND_MSG(v_names(i));
  END LOOP;
END;
/
```

**索引表（ITABLE / 联合数组）语法：**

```
TYPE type_name IS TABLE OF type_x INDEX BY (PLS_INTEGER | VARCHAR)
```

```sql
SQL> DECLARE
  TYPE type_money IS TABLE OF NUMBER INDEX BY VARCHAR2(64);
  salary type_money;
  n VARCHAR2(64);
BEGIN
  salary('ZhangSan') := 1000;
  salary('LiSi') := 2000;
  salary('WangWu') := 3000;
  n := salary.FIRST;
  WHILE n IS NOT NULL LOOP
    DBMS_OUTPUT.PUT_LINE('Salary of ' || n || ' is ' || salary(n));
    n := salary.Next(n);
  END LOOP;
END;
/
```

**多维集合：** XuguDB 支持使用集合构建多维集合。

```sql
SQL> DECLARE
  TYPE type_var1 IS VARRAY(3) OF INT;
  TYPE type_var2 IS VARRAY(5) OF type_var1;
  var type_var2 := type_var2(
    type_var1(1,2,3),
    type_var1(4,5,6),
    type_var1(7,8,9)
  );
BEGIN
  FOR i IN 1..3 LOOP
    FOR j IN 1..2 LOOP
      SEND_MSG(VAR(i)(j));
    END LOOP;
  END LOOP;
END;
/
```

#### 5.3.3 集合方法

| 方法 | 描述 | 限制 |
|------|------|------|
| `DELETE` | 删除集合所有元素 | 通用 |
| `DELETE(n)` | 删除下标为 n 的元素 | 对 VARRAY 非法 |
| `DELETE(n,m)` | 删除下标 n~m 的元素 | 对 VARRAY 非法 |
| `TRIM` | 从末端删除 1 个元素 | 对 ITABLE 非法 |
| `TRIM(n)` | 从末端删除 n 个元素 | 对 ITABLE 非法 |
| `EXTEND` | 添加 1 个 NULL 元素 | 对 ITABLE 非法 |
| `EXTEND(n)` | 添加 n 个 NULL 元素 | 对 ITABLE 非法 |
| `EXTEND(n,m)` | 添加 n 个元素，值为索引 m 的值 | 对 ITABLE 非法 |
| `EXISTS(n)` | 下标 n 的元素是否存在 | 通用 |
| `COUNT` / `COUNT()` | 返回元素数目 | 通用 |
| `FIRST` / `FIRST()` | 返回第一个元素索引号 | 通用 |
| `LAST` / `LAST()` | 返回最后一个元素索引号 | 通用 |
| `PRIOR(n)` | 返回元素 n 的前一个索引号 | 通用 |
| `NEXT(n)` | 返回元素 n 的下一个索引号 | 通用 |
| `LIMIT` | 返回 VARRAY 创建时指定的 SIZE | 仅 VARRAY |

> **注意**：`DELETE` 删除元素会保留占位符，`TRIM` 则会销毁占位符。VARRAY 和 TABLE 在赋值前必须初始化，ITABLE 无需初始化。

### 5.4 属性类型

属性类型用于引用其他对象特性的特殊语法：

- **%TYPE**：声明与某个变量或表列相同数据类型的变量
- **%ROWTYPE** / **%ROW TYPE**：声明存储表中一行数据或游标匹配结果的变量

```sql
-- 引用表的行数据
SQL> DECLARE
  emp employees_tab%ROWTYPE;
BEGIN
  SELECT * INTO emp FROM employees_tab WHERE id = 1;
  SEND_MSG(emp.first_name || emp.last_name);
END;
/

-- 引用表的列数据
SQL> DECLARE
  emp employees_tab.first_name%TYPE;
BEGIN
  SELECT first_name INTO emp FROM employees_tab WHERE id = 1;
  SEND_MSG(emp);
END;
/
```

也可以使用自定义类型方式：

```sql
SQL> DECLARE
  TYPE emp_type IS employees_tab%ROWTYPE;
  emp emp_type;
BEGIN
  SELECT * INTO emp FROM employees_tab WHERE id = 1;
  SEND_MSG(emp.first_name || emp.last_name);
END;
/

SQL> DECLARE
  TYPE emp_type IS ROWTYPE OF employees_tab;
  emp emp_type;
BEGIN
  SELECT * INTO emp FROM employees_tab WHERE id = 1;
  SEND_MSG(emp.first_name || emp.last_name);
END;
/
```

## 六、声明部分（DECLARE）

声明部分以关键字 `DECLARE` 开头，定义程序中要使用的变量、类型、游标、异常等。

**语法：**

```
DECLARE
  varname [CONSTANT] TypeName [NOT NULL] [DEFAULT | := def_val]
  | CursorDef
  | ExceptionDef
  | TypeDefStmt
  | PRAGMA EXCEPTION_INIT(errname, integer)
```

### 声明变量

变量声明必须指定名称和数据类型，可选指定初始值。

### 声明常量

常量声明需包含 `CONSTANT` 关键字和初始值（永久值）。

### NOT NULL 约束

指定 NOT NULL 时必须为变量分配初始值（因为标量变量的默认初始值为 NULL）。

### 初始值

使用赋值运算符 `:=` 或关键字 `DEFAULT` 指定初始值。

```sql
SQL> DECLARE
  id INT := 2;
  str CONSTANT VARCHAR NOT NULL DEFAULT 'strings';
  CURSOR cur IS SELECT * FROM user_tables;
  TYPE type_varray_int IS VARRAY(5) OF INTEGER;
  sid id%TYPE := id;
  no_null EXCEPTION;
  PRAGMA EXCEPTION_INIT(no_null, 16005);
BEGIN
  NULL;
END;
/
```

> **注意**：非匿名块中的声明可不使用 `DECLARE` 关键字：
> ```sql
> CREATE OR REPLACE PROCEDURE proc_declare IS
>   str VARCHAR := 'abc';
> BEGIN
>   DBMS_OUTPUT.PUT_LINE(str);
> END;
> /
> ```

## 七、赋值操作

### 7.1 := 操作符赋值

```sql
variable := expr;
```

```sql
SQL> DECLARE
  v_age NUMBER := 25;
  v_name VARCHAR;
BEGIN
  v_name := 'Bob';
  v_name := v_name || (v_age + 1);
  SEND_MSG(v_name);
END;
/
-- 输出 Bob26
```

### 7.2 SELECT INTO 赋值

```sql
SELECT target_list INTO ident_list FROM ...;
```

```sql
SQL> DECLARE
  v_id NUMBER;
  v_name VARCHAR;
BEGIN
  SELECT name, id INTO v_name, v_id FROM tab_selectinto WHERE id = 1;
  SEND_MSG(v_name || v_id);
END;
/
```

### 7.3 BULK COLLECT INTO 赋值

一次性将多行查询结果批量赋值到集合变量。

```sql
SQL> DECLARE
  TYPE ty_tab_bulk IS TABLE OF tab_bulkcollectinto%ROWTYPE;
  tab_bulk ty_tab_bulk;
BEGIN
  SELECT name, id BULK COLLECT INTO tab_bulk FROM tab_bulkcollectinto;
  FOR i IN 1..tab_bulk.count LOOP
    SEND_MSG(tab_bulk(i).name || tab_bulk(i).id);
  END LOOP;
END;
/
```

### 7.4 游标赋值

```sql
FETCH cursor_name [BULK COLLECT] INTO ident_list;
```

## 八、控制结构

### 8.1 条件语句

#### IF-THEN-ELSE

```sql
IF bool_expr THEN
  pl_stmt_list
ELSIF bool_expr THEN
  pl_stmt_list
ELSE
  pl_stmt_list
END IF;
```

```sql
SQL> DECLARE
  v_age NUMBER := 25;
BEGIN
  IF v_age < 18 THEN
    DBMS_OUTPUT.PUT_LINE('未成年人');
  ELSIF v_age >= 18 AND v_age < 65 THEN
    DBMS_OUTPUT.PUT_LINE('成年人');
  ELSE
    DBMS_OUTPUT.PUT_LINE('老年人');
  END IF;
END;
/
-- 输出 成年人
```

#### CASE 语句

**选择型 CASE：**

```sql
SQL> DECLARE
  v_dept_id NUMBER := 20;
  v_dept_name VARCHAR2(50);
BEGIN
  v_dept_name := CASE v_dept_id
    WHEN 10 THEN '人力资源部'
    WHEN 20 THEN '财务部'
    WHEN 30 THEN '市场部'
    ELSE '未知部门'
  END;
  DBMS_OUTPUT.PUT_LINE('部门名称: ' || v_dept_name);
END;
/
```

**搜索型 CASE：**

```sql
SQL> DECLARE
  v_salary NUMBER := 80;
  v_level VARCHAR2(20);
BEGIN
  v_level := CASE
    WHEN v_salary < 50 THEN '低级'
    WHEN v_salary >= 50 AND v_salary < 100 THEN '中级'
    WHEN v_salary >= 100 THEN '高级'
    ELSE '未知'
  END;
  DBMS_OUTPUT.PUT_LINE('等级: ' || v_level);
END;
/
```

> **注意**：CASE 语句中 `WHEN NULL` 不会匹配 NULL 值，需使用 `IS NULL`。

### 8.2 循环语句

#### LOOP（无条件循环）

```sql
LOOP
  pl_stmt_list
  EXIT WHEN condition;
END LOOP;
```

```sql
SQL> DECLARE
  v_counter NUMBER := 1;
BEGIN
  LOOP
    DBMS_OUTPUT.PUT_LINE('当前数值为：' || v_counter);
    v_counter := v_counter + 1;
    EXIT WHEN v_counter > 5;
  END LOOP;
END;
/
```

#### WHILE 循环

```sql
WHILE bool_expr LOOP
  pl_stmt_list
END LOOP;
```

#### FOR 循环

**数值范围型：**

```sql
BEGIN
  FOR i IN 1..5 LOOP
    DBMS_OUTPUT.PUT_LINE('正向数值：' || i);
  END LOOP;
  FOR i IN REVERSE 5..1 LOOP
    DBMS_OUTPUT.PUT_LINE('反向数值：' || i);
  END LOOP;
END;
/
```

**游标型：**

```sql
SQL> DECLARE
  CURSOR cur IS SELECT * FROM tab_forcur;
BEGIN
  FOR i IN cur LOOP         -- 使用显式游标
    SEND_MSG(i.table_name);
  END LOOP;
  FOR j IN (SELECT * FROM tab_forcur) LOOP  -- 使用隐式游标
    SEND_MSG(j.table_type);
  END LOOP;
END;
/
```

#### FORALL（批量 DML）

FORALL 类似 FOR 循环，但只能执行一个 DML 语句（UPDATE/INSERT/DELETE/EXECUTE），用于批量操作以减少数据库交互次数。

**语法：**

```
FORALL i IN lower_bound..higher_bound
  dml_stmt;

FORALL i IN INDICES OF collection [BETWEEN lower AND higher]
  dml_stmt;

FORALL i IN VALUES OF collection
  dml_stmt;
```

```sql
SQL> BEGIN
  FORALL i IN 1..5
    INSERT INTO tab_forall VALUES(i, 'test' || i);
END;
/
```

**INDICES OF / VALUES OF 示例：**

```sql
SQL> DECLARE
  TYPE ty_forall2 IS VARRAY(100) OF INT;
  tab_ty_forall2 ty_forall2 := ty_forall2(1, 3, 2, 5, 3);
BEGIN
  FORALL i IN INDICES OF tab_ty_forall2   -- i 取值为 1,2,3,4,5
    INSERT INTO tab_forall2 (id) VALUES (tab_ty_forall2(i));
  FORALL i IN VALUES OF tab_ty_forall2    -- i 取值为 1,3,2,5,3
    INSERT INTO tab_forall3 (id) VALUES (tab_ty_forall2(i));
END;
/
```

### 8.3 跳转语句

| 语句 | 说明 |
|------|------|
| `CONTINUE` | 跳过本次循环，继续下一次循环 |
| `EXIT [WHEN bool_expr]` | 退出当前循环 |
| `GOTO label` | 无条件跳转到标签位置 |
| `RETURN [expr]` | 结束当前子程序，返回结果 |

**GOTO 示例：**

```sql
SQL> DECLARE
  v_num INT := 1;
BEGIN
  <<loop_start>>
  IF v_num <= 5 THEN
    DBMS_OUTPUT.PUT_LINE('当前数字为: ' || v_num);
    v_num := v_num + 1;
    GOTO loop_start;
  END IF;
END;
/
```

> **注意**：GOTO 不允许从外层跳转到块、循环、IF 语句的内层。

## 九、游标

### 9.1 游标类型

| 类型 | 说明 | 关闭方式 |
|------|------|----------|
| 隐式游标 | 由数据库自动创建并管理，名为 SQL | 系统自动关闭 |
| 显式游标 | 由用户显式声明、打开、提取、关闭 | 手动关闭 |
| 游标变量 | 变量形式的游标句柄，支持动态绑定查询 | 手动关闭 |

### 9.2 游标属性

通过 `cursor_name%attribute` 获取：

| 属性 | 说明 |
|------|------|
| `%FOUND` | 最近一次成功获取记录时返回 TRUE |
| `%NOTFOUND` | 与 FOUND 相反 |
| `%ISOPEN` | 游标处于打开状态时返回 TRUE |
| `%ROWCOUNT` | 返回已读取的记录个数 |

### 9.3 隐式游标

隐式游标由系统自动命名为 `SQL`，每次执行 SELECT 或 DML 时自动打开。

```sql
SQL> DECLARE
BEGIN
  UPDATE test_cur SET id = id + 1 WHERE id > 1;
  SEND_MSG('ISOPEN: ' || to_char(SQL%ISOPEN));
  SEND_MSG('ISFOUND: ' || to_char(SQL%FOUND));
  SEND_MSG('TOTAL UPDATE:' || SQL%ROWCOUNT || ' ROWS');
  ROLLBACK;
END;
/
-- 输出
-- ISOPEN: FALSE
-- ISFOUND: TRUE
-- TOTAL UPDATE:2 ROWS
```

> **注意**：隐式游标的 `%ISOPEN` 始终返回 FALSE，因为隐式游标在命令执行后自动关闭。

### 9.4 显式游标

**使用步骤：** 定义 -> 打开 -> 提取 -> 关闭

**定义语法：**

```sql
-- 方式1：CURSOR IS
CURSOR cursor_name(args) IS SelectStmt;

-- 方式2：DECLARE CURSOR FOR
DECLARE cursor_name(args) CURSOR FOR SelectStmt;
```

**完整示例：**

```sql
SQL> DECLARE
  CURSOR cur(inid INT) IS
    SELECT * FROM test_cur WHERE id > inid;
  oid INT;
  oname VARCHAR;
BEGIN
  IF NOT cur%ISOPEN THEN
    OPEN cur(1);
  END IF;
  FETCH cur INTO oid, oname;
  WHILE cur%FOUND LOOP
    SEND_MSG('ID IS:' || oid || ' NAME IS:' || oname);
    FETCH cur INTO oid, oname;
  END LOOP;
  SEND_MSG('TOTAL FETCH :' || cur%ROWCOUNT || ' ROWS');
  CLOSE cur;
END;
/
```

**BULK COLLECT 批量返回：**

```sql
SQL> DECLARE
  CURSOR cur IS SELECT * FROM test_cur;
  TYPE trow IS TABLE OF cur%ROWTYPE;
  otab trow;
BEGIN
  OPEN cur;
  FETCH cur BULK COLLECT INTO otab;
  CLOSE cur;
  FOR i IN 1 .. otab.COUNT LOOP
    SEND_MSG(otab(i).id || '|' || otab(i).name);
  END LOOP;
END;
/
```

### 9.5 游标变量（SYS_REFCURSOR / REF CURSOR）

游标变量可在运行时动态绑定任意查询，常用于存储过程/函数的参数。

```sql
SQL> DECLARE
  i INT;
  v_chr VARCHAR;
  cur_sysref SYS_REFCURSOR;
BEGIN
  OPEN cur_sysref FOR SELECT name FROM test_cur;
  i := 1;
  LOOP
    FETCH cur_sysref INTO v_chr;
    EXIT WHEN cur_sysref%NOTFOUND;
    SEND_MSG('Row ' || i || ' is: ' || v_chr);
    i := i + 1;
  END LOOP;
  CLOSE cur_sysref;
END;
/
```

### 9.6 游标使用注意事项

1. **及时关闭**：显式游标和游标变量占用会话资源，使用完必须 CLOSE
2. **限制批量大小**：大数据集场景使用 BULK COLLECT 时，建议搭配 LIMIT 分批提取
3. **避免游标泄漏**：使用 CURSOR...IS 定义时，把所有操作放在同一 PL/SQL 块

## 十、异常处理

### 10.1 异常处理结构

```sql
DECLARE
  -- 异常定义
BEGIN
  -- 可能引发异常的语句
EXCEPTION
  WHEN exception_name1 THEN
    statement1;
  WHEN exception_name2 THEN
    statement2;
  WHEN OTHERS THEN
    -- 处理所有未被前面 WHEN 子句捕获的异常
END;
/
```

### 10.2 预定义异常

| 异常标识 | 异常信息 | 错误码 |
|----------|----------|--------|
| ZERO_DIVIDE | 除数为 0 | 19005 |
| NO_DATA_FOUND | 查询无结果 | 19009 |
| TOO_MANY_ROWS | 查询结果太多 | 19010 |
| DUP_VAL_ON_INDEX | 违反唯一值约束 | 13001 |

```sql
SQL> DECLARE
  x INT := 2;
  y INT := 0;
BEGIN
  x := x / y;
EXCEPTION
  WHEN ZERO_DIVIDE THEN
    SEND_MSG('除数为0');
END;
/

-- 也可使用错误码捕获
EXCEPTION
  WHEN 19005 THEN
    SEND_MSG('除数为0');
```

### 10.3 自定义异常

自定义异常必须在 DECLARE 部分使用 `EXCEPTION` 关键字声明，使用 `RAISE` 或 `THROW` 抛出。

```sql
SQL> CREATE OR REPLACE PROCEDURE pro_1(a INT) IS
DECLARE
  e1 EXCEPTION;
  e2 EXCEPTION;
BEGIN
  IF a > 5 THEN
    THROW EXCEPTION e1;
  ELSE
    THROW e2;
  END IF;
EXCEPTION
  WHEN e1 THEN send_msg('抛出异常1');
  WHEN e2 THEN send_msg('抛出异常2');
  WHEN OTHERS THEN send_msg('其它抛出异常');
END;
/
```

### 10.4 非预定义异常（PRAGMA EXCEPTION_INIT）

将自定义异常名称与特定错误号关联：

```sql
SQL> DECLARE
  no_null EXCEPTION;
  PRAGMA EXCEPTION_INIT(no_null, 16005);
BEGIN
  INSERT INTO tb_ex VALUES(NULL);
EXCEPTION
  WHEN no_null THEN
    DBMS_OUTPUT.PUT_LINE(SQLCODE);
    DBMS_OUTPUT.PUT_LINE(SQLERRM);
END;
/
-- 输出：16005  字段 ID 不能取空值
```

### 10.5 RAISE_APPLICATION_ERROR

自定义错误码：

```sql
SQL> DECLARE
  a NUMERIC(4,0) := NULL;
BEGIN
  IF a IS NULL THEN
    RAISE_APPLICATION_ERROR(9999, 'A为空');
  END IF;
END;
/
-- 输出 Error: [E9999 L5 C1] A为空
```

### 10.6 异常处理函数

| 函数 | 说明 |
|------|------|
| `SQLCODE` | 返回当前异常的错误码 |
| `SQLERRM` | 返回当前异常的错误信息 |
| `RAISE_APPLICATION_ERROR` | 抛出自定义错误码和消息 |
| `DBMS_UTILITY.FORMAT_ERROR_STACK` | 返回错误堆栈 |
| `DBMS_UTILITY.FORMAT_ERROR_BACKTRACE` | 返回错误回溯 |

### 10.7 异常重新引发（RAISE）

在异常处理块中使用 `RAISE;`（不带异常名）可将异常传播到外层块：

```sql
SQL> DECLARE
  sal_too_high EXCEPTION;
  cur_sal NUMBER := 8000;
  max_sal NUMBER := 6000;
  err_oneous_sal NUMBER;
BEGIN
  BEGIN
    IF cur_sal > max_sal THEN
      RAISE sal_too_high;
    END IF;
  EXCEPTION
    WHEN sal_too_high THEN
      err_oneous_sal := cur_sal;
      DBMS_OUTPUT.PUT_LINE('Salary:' || err_oneous_sal || ' is out of range.');
      RAISE;  -- 重新引发，传播到外层
  END;
EXCEPTION
  WHEN sal_too_high THEN
    cur_sal := max_sal;
    DBMS_OUTPUT.PUT_LINE('Revising salary from ' || err_oneous_sal || ' to ' || cur_sal || '.');
END;
/
```

## 十一、SQL 执行

PL/SQL 中可直接执行 DML、DDL、事务控制、查询、批量操作等 SQL 语句。

### 11.1 自治事务（匿名事务）

独立于主事务的事务，使用 `PRAGMA AUTONOMOUS_TRANSACTION` 标识：

```sql
SQL> CREATE PROCEDURE proc_anonymous_insert() AS
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
  INSERT INTO tab_person VALUES('Bob', 20);
  COMMIT;
END;
/

-- 主事务
SQL> BEGIN
  INSERT INTO tab_person VALUES('Alice', 10);
  proc_anonymous_insert();  -- 自治事务中的提交不影响主事务
  ROLLBACK;                 -- 只回滚主事务中的操作
END;
/
-- 结果：只有 Bob 的记录保留
```

### 11.2 动态 SQL（EXECUTE IMMEDIATE）

```sql
-- 基本用法
EXECUTE IMMEDIATE 'SQL语句';

-- 带参数绑定 (USING)
EXECUTE IMMEDIATE 'SELECT ... WHERE col = :param'
  USING param_value;

-- 带返回值 (INTO)
EXECUTE IMMEDIATE 'SELECT COUNT(*) FROM ...'
  RETURNING INTO num_var;

-- USING + INTO 组合
EXECUTE IMMEDIATE 'SELECT COUNT(*) FROM user_tables WHERE table_name = :name'
  USING 'TTT' INTO num_var;

-- RETURNING BULK COLLECT INTO（批量返回）
EXECUTE IMMEDIATE 'SELECT * FROM t1'
  RETURNING BULK COLLECT INTO v1, v2, v3;

-- LIMIT 限制返回条数
CALL IMMEDIATE v_sql RETURNING INTO v1, v2, v3 LIMIT 1;
```

## 十二、输出

### DBMS_OUTPUT.PUT_LINE

系统包，用于向控制台输出文本信息：

```sql
SQL> DECLARE
  str VARCHAR2(200);
  id INT := 100;
BEGIN
  dbms_output.put_line('直接输出字符常量');
  dbms_output.put_line(id);
  dbms_output.put_line('拼接：' || str || id);
END;
/
```

### SEND_MSG

虚谷独有的系统函数，向客户端发送消息，参数类型为 VARCHAR，返回类型为 BOOLEAN：

```sql
SQL> BEGIN
  send_msg('直接输出字符常量');
END;
/
```

### RETURN

存储函数中必须使用 RETURN 返回值，返回类型需与函数定义一致。

## 与 Oracle/MySQL/PG 语法对比

| 特性 | XuguDB | Oracle | MySQL | PostgreSQL |
|------|--------|--------|-------|------------|
| 块结构 | DECLARE/BEGIN/EXCEPTION/END | 相同 | BEGIN/END | DO $$ DECLARE/BEGIN/EXCEPTION/END $$ |
| 变量赋值 | `:=` | `:=` | `SET @var = val` / `=` | `:=` |
| SELECT INTO | 支持 | 支持 | `SELECT ... INTO` | 支持 |
| BULK COLLECT | 支持 | 支持 | 不支持 | 不支持 |
| FORALL | 支持 | 支持 | 不支持 | 不支持 |
| 游标 FOR 循环 | FOR i IN cursor LOOP | 相同 | 不支持 | FOR rec IN cursor LOOP |
| SYS_REFCURSOR | 支持 | 支持 | 不支持 | REFCURSOR |
| RAISE/THROW | 两者均支持 | 仅 RAISE | SIGNAL | 仅 RAISE |
| PRAGMA EXCEPTION_INIT | 支持 | 支持 | 不支持 | 不支持 |
| RAISE_APPLICATION_ERROR | 支持 | 支持 | SIGNAL SQLSTATE | RAISE EXCEPTION |
| EXECUTE IMMEDIATE | 支持(含USING/INTO) | 支持 | PREPARE/EXECUTE | EXECUTE(不同语法) |
| 自治事务 | PRAGMA AUTONOMOUS_TRANSACTION | 相同 | 不支持 | 不支持 |
| %TYPE/%ROWTYPE | 支持 | 支持 | 不支持 | 支持 |
| VARRAY/TABLE/ITABLE | 支持 | 支持 | 不支持 | ARRAY(有限) |
| DBMS_OUTPUT | 支持 | 支持 | 不支持 | 需扩展 |
| SEND_MSG | 虚谷独有 | 不支持 | 不支持 | 不支持 |
