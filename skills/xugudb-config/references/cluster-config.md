# 辅助配置文件

虚谷数据库除 xugu.ini 主配置文件外，还包含集群配置、数据文件配置、路径挂载、黑白名单和类型映射等辅助配置文件，均位于 SETUP 目录下。

## cluster.ini 集群配置

cluster.ini 用于配置数据库集群的规模、通信、角色、算力、存储权重等信息。使用 `--init-setup` 参数启动数据库程序可生成模板文件 cluster.ini.temp，修改后重命名为 cluster.ini 即可启用。

**严禁修改 cluster.ini 中的参数格式及顺序**，否则会导致集群配置失效或不可用。

| 场景 | 启动方式 |
|------|----------|
| 缺失或无法打开 cluster.ini | 单机模式启动 |
| 成功解析 cluster.ini | 集群模式启动 |
| 错误的 cluster.ini 格式 | 记载 EVENT.LOG，进程退出 |

### 全局参数

| 参数名 | 类型 | 默认值 | 取值范围 | 说明 |
|--------|------|--------|----------|------|
| MAX_NODES | INTEGER | 32 | [1, 512] | 集群最大节点数，建议按预估规模设值，避免过大值浪费内存。>1 时集群部署 |
| MASTER_GRPS | INTEGER | 1 | 1 | 控制角色组数，不可修改，仅支持 1 组 |
| PROTOCOL | VARCHAR | 'UDP' | {'UDP'} | 节点间通信协议 |
| MSG_PORT_NUM | INTEGER | 1 | [1, 16] | 节点间通讯通道数，需与 PORTS 配置一致 |
| MAX_SEND_WIN | INTEGER | 128 | [1, 2046] | 每通道最大发送窗口大小，网络延迟高时可调大 |
| MSG_HAVE_CRC | INTEGER | 0 | {0, 1} | 是否对节点间消息进行 CRC 校验 |
| MERGE_SMALL_MSG | INTEGER | 0 | {0, 1} | 是否启用消息小包合并（暂未使用） |
| MSG_SIZE | INTEGER | 32768 | [1024, 65536) | 节点间通信消息报文最大长度。MTU>9000 建议设为 9000 |
| TIMEOUT | INTEGER | 10000 | [1, 2147483647] | 消息发送超时（单位 0.1 微秒），高延迟网络适当调大。可通过 `SHOW MSG_TIMEOUT` 查看 |
| RPC_WINDOW | INTEGER | 8 | [0, 63] | RPC 流量控制窗口大小 |
| EJE_WINDOW | INTEGER | 8 | [0, 255] | 并行弹射器流量控制窗口大小 |
| MAX_SHAKE_TIME | INTEGER | 1200 | [1, 2147483647] | 集群启动握手超时时间（秒） |
| MY_NID | INTEGER | 0001 | [1, 2147483647] | 当前节点在集群中的编号，必须与某个节点的 NID 映射 |
| CHECK_RACK | INTEGER | 0 | {0, 1} | 是否检查控制节点机架号。1=每组控制节点机架号必须相等 |
| ADDR_TYPE | VARCHAR | 'IPV4' | {'IPV4', 'DOMAIN'} | 集群 IP 地址协议类型（v12.4.0 引入） |

### 节点参数

每个节点用分号 `;` 分隔，包含以下参数：

| 参数名 | 类型 | 默认值 | 取值范围 | 说明 |
|--------|------|--------|----------|------|
| NID | INTEGER | 0001 | [1, 2147483647] | 节点编号，连续递增，控制节点编号必须连续 |
| RACK | INTEGER | 0001 | - | 机架号 |
| PORTS | VARCHAR | '127.0.0.1:10000' | - | 节点 IPv4/域名和通信端口。多路格式：IP1:PORT1,IP2:PORT2。发送端口=接收端口+20 |
| ROLE | VARCHAR | 'MSQW' 或 'SQWG' | {'M','S','Q','W','G'} 组合 | 节点角色（可组合） |
| LPU | INTEGER | 1 | [1, 255] | 逻辑处理器个数，建议值=物理 CPU 核数-1。仅对 W 角色有效 |
| STORE_WEIGHT | INTEGER | 3 | [1, 10] | 存储权重，可按磁盘容量比或 IO 性能比配置 |
| STATE | VARCHAR | DETECT | {DETECT, INVALID} | 节点状态（暂未使用，建议保持默认） |

### 节点角色说明

| 角色 | 职责 |
|------|------|
| M（控制） | 节点管理、资源管理、故障处理（心跳检测）、存储均衡修复迁移、全局锁管理、全局仲裁。**必须配置 2 个且节点号连续**，建议用 1 和 2 |
| S（存储） | 数据存储，响应工作节点的增删查改等底层数据操作 |
| Q（查询） | 建立 TCP 监听端口，对外服务入口；只有含 Q 的节点才能接受外部客户端连接。**必须与 W 同时配置** |
| W（工作） | 并发并行任务调度与计算处理，生成执行计划，与存储角色交互完成数据读写。**必须与 Q 同时配置** |
| G（变更记载） | 变更数据采集，用于数据备份和增量日志读取。**最多允许配置于 2 个节点** |

### 配置模板示例

```
#MAX_NODES=32 MASTER_GRPS=1 PROTOCOL='UDP' MSG_PORT_NUM=1 MAX_SEND_WIN=128
MSG_HAVE_CRC=0 MERGE_SMALL_MSG=0 MSG_SIZE=32768 TIMEOUT=10000 RPC_WINDOW=8
EJE_WINDOW=8 MAX_SHAKE_TIME=1200 MY_NID=0001 CHECK_RACK=0 ADDR_TYPE='IPV4'
NID=0001 RACK=0001 PORTS='127.0.0.1:10000' ROLE='MSQW' LPU=1 STORE_WEIGHT=3 STATE=DETECT;
NID=0002 RACK=0001 PORTS='127.0.0.1:10000' ROLE='MSQW' LPU=1 STORE_WEIGHT=3 STATE=DETECT;
NID=0003 RACK=0001 PORTS='127.0.0.1:10000' ROLE='SQWG' LPU=1 STORE_WEIGHT=3 STATE=DETECT;
```

## datafile.ini 数据文件配置

datafile.ini 用于指定数据文件的数据库逻辑存储位置和文件个数。**仅当需要指定数据库文件存储位置和文件个数时才需手动创建**。

### 文件格式

```
#DATA_FILES /DATA1/DBDATA1.DBF /DATA2/DBDATA2.DBF
#TEMP_FILES /TEMP1/TEMP1.DBF /TEMP2/TEMP2.DBF
#UNDO_FILES /UNDO1/UNDOFILE1.DBF /UNDO1/UNDOFILE2.DBF
```

| 类型 | 说明 |
|------|------|
| DATA_FILES | 数据文件存放位置及名称 |
| TEMP_FILES | 临时文件存放位置及名称 |
| UNDO_FILES | 回滚文件存放位置及名称 |

路径为数据库逻辑目录，需在 mount.ini 中配置对应的操作系统映射路径。mount.ini 中的映射路径必须已存在（不自动创建）。

datafile.ini 中的文件个数配置优先级高于 xugu.ini 中的 init_data_space_num / init_temp_space_num / init_undo_space_num。

## mount.ini 路径挂载配置

mount.ini 用于存储各类文件的路径映射，支持安装路径的相对路径和操作系统绝对路径。

### 默认映射

```
/XGLOG ./XGLOG
/CATA  ./XHOME/CATA
/DATA  ./XHOME/DATA
/TEMP  ./XHOME/TEMP
/REDO  ./XHOME/REDO
/BACKUP ./BACKUP
/ARCH  ./XHOME/ARCH
/UNDO  ./XHOME/UNDO
/MODI  ./XHOME/XMODI
```

### 映射关系说明

| 逻辑目录 | 物理路径 | 说明 |
|----------|----------|------|
| /XGLOG | ./XGLOG | 系统运行日志（错误日志、事件日志等） |
| /CATA | ./XHOME/CATA | 系统结构性控制文件 |
| /DATA | ./XHOME/DATA | 数据库数据文件 |
| /TEMP | ./XHOME/TEMP | 临时表空间文件 |
| /REDO | ./XHOME/REDO | 重做日志（WAL）文件 |
| /BACKUP | ./BACKUP | 备份文件 |
| /ARCH | ./XHOME/ARCH | 归档日志文件 |
| /UNDO | ./XHOME/UNDO | 回滚日志文件 |
| /MODI | ./XHOME/XMODI | 变更日志文件 |

**注意**：该配置文件数据目录只能映射到一个物理位置。需要将数据文件存储到不同位置时，配合 datafile.ini 使用。./XVOCA 为词表日志目录，存放 xxx.voc 词表文件。

## trust.ini 黑白名单配置

trust.ini 用于对请求连接的客户端进行受信认证。**黑名单优先级高于白名单**。

### 配置格式

```
{trust | untrust} {db_name | everydb} {user_name | everyone} from ip1 [to ip2]
```

| 关键字 | 说明 |
|--------|------|
| trust | 白名单，信任策略中的用户、库和 IP |
| untrust | 黑名单，拒绝策略中的用户、库和 IP |
| db_name / everydb | 影响的数据库（everydb=所有库） |
| user_name / everyone | 影响的用户（everyone=所有用户） |
| ip1 | IP 地址，anywhere 表示所有来访 IP |
| to ip2 | 可选，表示从 ip1 到 ip2 的 IP 段 |

### 示例

```ini
# 默认配置：信任所有来源
trust everydb everyone from anywhere

# 只允许特定 IP 的 SYSDBA 访问 SYSTEM 库
trust system sysdba from 192.168.1.100

# 允许 IP 段访问
trust system sysdba from 192.168.1.100 to 192.168.1.109

# 拒绝某 IP 段
untrust everydb everyone from 10.0.0.1 to 10.0.0.255
```

**注意**：
- 配置仅对当前节点生效，需启动前配置或配置后重启
- 建议所有节点配置一致
- 也可使用具备权限的用户登录后执行命令方式配置

## types.ini 数据类型映射配置

types.ini 用于配置外部类型名到虚谷数据库内置基础数据类型的映射关系。基于此映射，外部类型名将被自动映射为对应的内置基础类型。修改后重启服务端生效。

### 默认映射

| 外部类型名 | 虚谷内置类型 |
|------------|-------------|
| BOOL | BOOLEAN |
| INT | INTEGER |
| SHORT | SMALLINT |
| LONGINT | BIGINT |
| LONG | CLOB |
| REAL | FLOAT |
| DECIMAL | NUMERIC |
| TEXT | VARCHAR |
| NCHAR | CHAR |
| NVARCHAR | VARCHAR |
| NVARCHAR2 | VARCHAR |
| VARCHAR2 | VARCHAR |
| NCLOB | CLOB |
| PLS_INTEGER | INTEGER |
| BINARY_INTEGER | INTEGER |
| BINARY_DOUBLE | DOUBLE |
| BINARY_FLOAT | FLOAT |
| RAW | BINARY |
| XMLTYPE | XML |

格式：每行一个映射关系，左侧为外部类型名，右侧为虚谷内置基础数据类型名。
