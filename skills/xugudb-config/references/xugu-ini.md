# xugu.ini 系统参数配置

xugu.ini 是虚谷数据库的主配置文件，位于数据库根路径下的 SETUP 目录。包含 13 个参数类别共 160+ 个配置项。

## SQL 引擎参数

控制查询优化器、并行执行、预编译、字典匹配等行为。

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| auto_eje_cast | INTEGER | 10000 | [-1, 2147483647] | 在线修改 | 自动生成并行执行计划的最小查询开销阈值。-1 禁用并行，>=0 根据代价决定 |
| auto_eje_parallel | INTEGER | 4 | [0, 1024] | 在线修改 | 自动并行执行计划的默认并行度，受 max_parallel 和集群 LPU 总数限制 |
| auto_use_eje | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否允许自动生成弹射式并行执行计划 |
| backslash_escapes | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否启用反斜杠转义，影响词法解析和 LIKE 匹配 |
| cata_case_sensitive | BOOLEAN | FALSE | TRUE/FALSE | 离线修改，重启 | 系统字典匹配时英文字母大小写是否敏感 |
| check_unique_mode | INTEGER | 2 | [1, 2] | 在线修改 | 唯一值检测方式。1=操作后检测，2=预检测 |
| ddl_timeout | INTEGER | 2000 | [100, 600000] | 在线修改 | DDL 操作加锁超时时间（毫秒） |
| def_empty_str_as_null | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 是否将长度为 0 的字符串当作 NULL |
| def_index_row_length | INTEGER | 1024 | [256, 4000] | 在线修改 | 索引最大行长限制 |
| def_optimize_mode | INTEGER | 0 | [0, 2147483647] | 在线修改 | 默认优化模式。0=ALL_ROWS，>0 为 FIRST_ROWS(N) |
| delay_check_unique | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否将唯一值检测延迟到事务结束时进行 |
| enable_eje_big_block | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否允许弹射器堆扫描使用大块读策略（预加载一个数据片） |
| enable_find_synonym | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 找不到对象元信息时是否尝试作为同义词查找 |
| enable_histogram | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 预估数据选择率时是否启用直方图优化 |
| enable_stream_import | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 是否启用流式导入功能（LOAD TABLE/IMPORT FROM STREAM/IMPORT FROM SELECT） |
| enable_sys_name_idx | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否启用系统对象名索引加速字典匹配，支持 SYS_OBJECTS/SYS_TABLES 等系统表 |
| error_for_division_zero | - | - | - | 在线修改 | 除数为零时的处理方式 |
| function_stack_size | INTEGER | - | - | - | 函数调用栈大小 |
| group_concat_max_len | INTEGER | - | - | 在线修改 | GROUP_CONCAT 结果最大长度 |
| idx_delay_del_limit | INTEGER | - | - | 在线修改 | 索引延迟删除限制 |
| idx_join_cost | INTEGER | - | - | 在线修改 | 索引连接代价系数 |
| iscan_ini_cost | INTEGER | - | - | 在线修改 | 索引扫描初始代价 |
| max_cursor_num | INTEGER | - | - | 在线修改 | 最大游标数 |
| max_loop_num | INTEGER | - | - | 在线修改 | 最大循环数 |
| max_prepare_num | INTEGER | - | - | 在线修改 | 最大预编译语句数 |
| max_sql_size | INTEGER | - | - | 在线修改 | 最大 SQL 长度 |
| para_eje_seqscan_num | INTEGER | - | - | 在线修改 | 并行执行顺序扫描数 |
| prepare_param_num | INTEGER | - | - | 在线修改 | 预编译参数数量 |
| prepare_reuse | BOOLEAN | - | TRUE/FALSE | 在线修改 | 预编译语句是否重用 |
| proc_reuse_cnt | INTEGER | - | - | 在线修改 | 过程重用计数 |
| select_table_num | INTEGER | - | - | 在线修改 | SELECT 最大表数量 |
| seqscan_skip_err | BOOLEAN | - | TRUE/FALSE | 在线修改 | 顺序扫描时是否跳过错误 |
| str_trunc_warning | BOOLEAN | - | TRUE/FALSE | 在线修改 | 字符串截断时是否发送警告 |
| stream_import_error | INTEGER | - | - | 在线修改 | 流导入错误处理模式 |
| supple_prepare_sql | BOOLEAN | - | TRUE/FALSE | 在线修改 | 补充预编译 SQL |
| support_global_tab | BOOLEAN | - | TRUE/FALSE | 在线修改 | 是否支持全局临时表 |
| tab_rebuild_limit | INTEGER | - | - | 在线修改 | 表重建限制 |
| use_index_order | BOOLEAN | - | TRUE/FALSE | 在线修改 | ORDER BY 排序是否使用索引序（开启后禁止自动并行） |
| week_mode | INTEGER | - | - | 在线修改 | 周模式设置 |

### 关键参数详解

**auto_use_eje（自动并行）**：设为 TRUE 时系统自动判断是否使用并行执行。不可并行的场景包括：auto_eje_cast=-1、use_index_order=true、含 WITH 语句、含文件表、含过程函数调用、含 GROUP_CONCAT/LISTAGG 等本地计算聚合函数、含 LAST_INSERT_ID/RANDOM/CURRENT_USER/SESSION_USER 等本地计算系统函数、存在依赖于上层的子查询表达式。

**cata_case_sensitive（大小写敏感）**：受 def_compatible_mode 影响。NONE/ORACLE 模式下标识符统一转大写，MYSQL 不做处理，POSTGRESQL 统一转小写。TRUE 时对 SYS_DATABASES/SYS_SCHEMAS/SYS_USERS 和 enable_sys_name_idx=true 场景不生效。

**def_empty_str_as_null（空串处理）**：TRUE 时影响 GROUP BY 分组、IS NULL/IS NOT NULL 表达式、CONCAT_WS/CONCAT/SUBSTRING_INDEX/LPAD/RPAD/INSTR/POSITION/LOCATE/STRPOS/SUBSTR/OVERLAY/RTRIM/LTRIM 等系统函数，以及服务端接收参数时将空串作为 NULL。

**ddl_timeout**：支持的 DDL 场景包括创建/删除/修改/重建索引，创建/删除/重编译包/过程/视图/UDT，删除序列，删除/修改/重命名/清理/回收/闪回表，使能/删除触发器，创建/删除/修改存储域，流式导入等。

## 事务子系统参数

控制事务隔离级别、提交策略、检查点、死锁检测等。

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| checkpoint_delay | INTEGER | 15 | [1, 100] | 离线修改，重启 | 检查点延迟时间（分钟），控制脏数据页最大回写延迟。每分钟推进一次，WAL 写入量超过文件大小 4/3 时也触发 |
| def_auto_commit | BOOLEAN | TRUE | TRUE/FALSE | 离线修改，重启 | 事务是否默认自动提交，连接会话未指定 auto_commit 时使用此值 |
| def_iso_level | INTEGER | 1 | {0,1,2,3} | 在线修改 | 默认事务隔离级别：0=只读，1=读已提交，2=重复读，3=序列化 |
| dlock_check_delay | INTEGER | 3000 | [1000, 600000] | 在线修改 | 死锁检测延迟时间（毫秒），事务等锁超过此值后发起死锁仲裁 |
| enable_node_degrade | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 是否启用节点降级防止脑裂。TRUE 时副管理节点降级为只读 |
| exit_when_redo_err | BOOLEAN | TRUE | TRUE/FALSE | 离线修改，重启 | 重放 WAL 失败时是否退出进程。FALSE 时记载 EVENT 日志后继续（无法保障数据一致性） |
| max_trans_modify | INTEGER | 10000 | [0, 2147483647] | 在线修改 | 单个事务最大变更行数。0=不限制，超过时上报 E14024 异常 |
| node_dead_delay | INTEGER | 30 | [1, 600] | 在线修改 | 节点判死心跳次数（一次心跳 2 秒），单机无效 |
| skip_boot_rollback | BOOLEAN | FALSE | TRUE/FALSE | 离线修改，重启 | 重启时是否跳过回滚修复（危险，无法保障数据一致性） |
| strictly_commit | BOOLEAN | FALSE | TRUE/FALSE | 离线修改，重启 | 非只读事务提交是否等候所有 WAL 落盘写实 |
| undo_delay_free | INTEGER | 3000 | [0, 86400000] | 在线修改 | UNDO 项延迟释放时间（毫秒）。0=立即释放 |

## 存储子系统参数

控制表空间文件、数据片、数据版本、存储迁移等。

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| auto_extend_dfile | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否允许自动扩展数据表空间文件，配合 max_file_size 使用 |
| block_pctfree | INTEGER | 15 | [0, 99] | 在线修改 | 数据页默认预留空间百分比，适用于堆表/临时表/全局临时表 |
| block_size | INTEGER | 8192 | - | 不可修改 | 数据页大小（固定 8KB） |
| data_file_append_mode | INTEGER | 0 | [0, 1] | 在线修改 | 扩展文件模式。0=写入置零（安全但慢），1=仅设置大小（快但可能空间不足） |
| def_data_space_size | INTEGER | 256 | [64, 2097152] | 离线，首次启动 | 数据表空间文件初始大小（MB），必须是 size_per_chunk 整数倍 |
| def_file_step_size | INTEGER | 64 | [64, 65536] | 离线，首次启动 | 表空间文件自动扩展步长（MB） |
| def_redo_file_size | INTEGER | 500 | [32, 2097152] | 离线修改，重启 | REDO 日志文件大小（MB），固定 2 个文件，每次启动重置 |
| def_temp_space_size | INTEGER | 256 | [64, 2097152] | 离线修改，重启 | 临时表空间文件初始大小（MB），每次启动重置 |
| def_undo_space_size | INTEGER | 256 | [64, 2097152] | 离线，首次启动 | 回滚数据表空间文件初始大小（MB） |
| default_copy_num | INTEGER | 3 | [1, 3] | 在线修改 | 新建表存储默认版本数（单机永远为 1） |
| enable_read_copy2 | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否允许读存储副版本（单机无效） |
| enable_store_migrate | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否根据 STORE_WEIGHT 自动均衡存储分布 |
| index_preload_cnt | INTEGER | 10000 | [0, 2147483647] | 在线修改 | 索引存储数大于此值时预加载索引首块，最多 4 线程 |
| init_data_space_num | INTEGER | 4 | [1, 32] | 离线，首次启动 | 数据表空间文件个数，datafile.ini 优先级更高 |
| init_temp_space_num | INTEGER | 2 | [1, 32] | 离线，首次启动 | 临时表空间文件个数 |
| init_undo_space_num | INTEGER | 2 | [1, 32] | 离线，首次启动 | 回滚表空间文件个数 |
| ioerr_report_mode | INTEGER | 1 | [0, 1] | 在线修改 | I/O 错误处理。0=标记存储版本失效，1=标记表空间存储介质错 |
| max_file_size | INTEGER | -1 | [-1, 2147483647] | 离线，首次启动 | 数据/临时表空间文件最大尺度（MB），-1 不限制 |
| max_hotspot_num | INTEGER | 256 | [1, 1024] | 在线修改 | 最大插入热点数，多热点降低并发写锁争用（临时表无效） |
| max_temp_space_size | INTEGER | -1 | [-1, 2147483647] | 在线修改 | 临时表空间最大尺度（MB），<=0 不限制 |
| ov_reuse | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否允许 OV 存储重用 |
| safely_copy_num | INTEGER | 2 | {1,2,3} | 在线修改 | 存储安全版本数，故障修复时补齐到此数（单机无效） |
| size_per_chunk | INTEGER | 8 | [1, 64] | 离线，首次启动 | 存储片大小（MB），管理的最小单位 |
| store_drop_delay | INTEGER | 16 | [2, 118] | 在线修改 | 局部存储延迟删除时间（分钟） |
| store_maint_span | INTEGER | 0 | [0, 1000] | 在线修改 | 存储维护时间间隔（毫秒） |

## 存储子系统附属参数

控制数据写回策略、AIO、缓存写穿等底层 I/O 行为。

| 参数名 | 类型 | 默认值 | 修改方式 | 说明 |
|--------|------|--------|----------|------|
| cache_undo_wrt | BOOLEAN | - | - | UNDO 缓存写入策略 |
| catalog_write_through | BOOLEAN | - | - | 系统表写穿 |
| data_persistence | INTEGER | - | - | 数据持久化模式 |
| datafile_use_aio | BOOLEAN | - | - | 数据文件是否使用异步 I/O |
| datafile_write_through | BOOLEAN | - | - | 数据文件写穿 |
| gstore_pick_mode | INTEGER | - | - | 全局存储选择模式 |
| major_control_mode | INTEGER | - | - | 主版本控制模式 |
| major_retry_num | INTEGER | - | - | 主版本重试次数 |
| max_write_back_num | INTEGER | - | - | 最大写回数量 |
| redo_write_through | BOOLEAN | - | - | REDO 日志写穿 |
| write_back_thd_num | INTEGER | - | - | 写回线程数 |

## 安全审计参数

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| enable_audit | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 是否开启审计功能。审计内容写入 SYSAUDITOR.SYS_AUDIT_RESULTS（自动扩展分区表） |
| security_level | INTEGER | 0 | {0,1,2} | 离线修改，重启 | 审计安全等级。0=由 enable_audit 和审计规则决定；>0 强制审计新建连接，SYSSSO 审计 SYSDBA 和 SYSAUDITOR 的操作 |

## 系统安全参数（口令策略）

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| pass_mode | INTEGER | 2 | {1,2,3,4} | 在线修改 | 口令复杂度：1=禁止单字符；2=需含两种字符类型；3=必须含数字+字母+特殊符号；4=强口令模式 |
| min_pass_len | INTEGER | 8 | [2, 127] | 在线修改 | 口令最小长度，所有 pass_mode 下生效 |
| min_pass_mixed_case | INTEGER | 0 | [0, 63] | 在线修改 | 口令最少大小写字母数（pass_mode=4 生效），大写和小写需分别满足 |
| min_pass_number | INTEGER | 0 | [0, 127] | 在线修改 | 口令最少数字数（pass_mode=4 生效） |
| min_pass_special_char | INTEGER | 0 | [0, 127] | 在线修改 | 口令最少特殊字符数（pass_mode=4 生效） |
| pass_username_check | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 禁止口令包含用户名或其倒序（pass_mode=4 生效，不区分大小写） |
| weak_pass_dictionary | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 启用弱口令字典检测（pass_mode=4 生效），字典文件 weak_pass_dictionary.txt 位于安装目录 |

### 强口令模式（pass_mode=4）

启用后系统初次启动或新建用户库时为 SYSDBA、SYSSSO、SYSAUDITOR、GUEST 生成随机口令。口令输出位置：`--child` 启动输出到标准输出，`--server` 启动输出到 stdout.txt。首次启动后各用户必须修改系统生成的口令。

## 系统并行度参数

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| cata_hash_size | INTEGER | 8192 | [1024, 131072] | 离线修改，重启 | 系统对象目录 HASH 表桶数，建议设为库内对象数（SYS_OBJECTS 统计）的 1-2 倍 |
| cata_parti_num | INTEGER | 32 | [1, 256] | 离线修改，重启 | 资源分组数（锁、数据页缓存、权限等），高并发建议设为 CPU 核数 |
| lock_hash_size | INTEGER | 8192 | [1024, 131072] | 离线修改，重启 | 锁管理器 HASH 表桶数，建议设为锁数量（SYS_GLOCKS 统计）的 2 倍 |
| max_parallel | INTEGER | 8 | [1, 1024] | 离线修改，重启 | 单机最大并行度。集群下为各节点 cluster.ini 中 LPU 总和 |
| rsync_thd_num | INTEGER | 8 | [1, 256] | 离线修改，重启 | 集群数据同步任务线程数，影响副版本处理效率（单机无效） |
| rtran_thd_num | INTEGER | 8 | [1, 256] | 离线修改，重启 | 代理事务处理线程数，影响分布式事务执行效率（单机无效） |
| task_thd_num | INTEGER | 16 | [4, 10000] | 离线修改，重启 | 任务处理线程数，支持自动扩充。stdout.txt 持续出现 'expand thd' 表示线程不足 |
| tcp_thd_num | INTEGER | 1 | [1, 32] | 离线修改，重启 | TCP 数据接收线程数，增加可提升客户端请求处理能力 |
| thd_bind_mode | INTEGER | 0 | [0, 2] | 离线修改，重启 | 线程绑定 CPU 模式（仅 Linux）。0=不绑定，1=按 NUMA 节点，2=NUMA 分段绑定 |

## 系统缓冲区参数

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| data_buff_mem | INTEGER | 256 | [8, 2147483647] | 离线修改，重启 | 数据页缓存总内存（MB），建议配置为物理内存的 60%。查看：`SHOW MEM_STATUS` |
| ena_share_sga | BOOLEAN | TRUE | TRUE/FALSE | 离线修改，重启 | 数据页缓存是否与 SGA 共享。TRUE 时 SGA 页 8KB 可互相借用，FALSE 时 SGA 页 256KB 各自独立 |
| hash_parti_num | INTEGER | 16 | [16, 256] | 在线修改 | 内存不足时转入 HASH 分组处理的分组数 |
| max_hash_mem | INTEGER | 32 | [32, 1048576] | 离线修改，重启 | 单个 HASH 计算节点最大内存（MB） |
| max_hash_size | INTEGER | 3000000 | [1048576, 134217728] | 在线修改 | 散列计算最大 HASH 槽数，一个槽 8 Byte。过小导致 HASH 碰撞影响效率 |
| max_malloc_once | INTEGER | 512 | [64, 4095] | 在线修改 | 最大单次内存分配大小（MB），超过抛出 E19116 |
| min_hash_size | INTEGER | 100 | [1, 500000000] | 在线修改 | 散列计算最小 HASH 槽数，修正数据量预估误差 |
| swap_buff_mem | INTEGER | 128 | [64, 1048576] | 离线修改，重启 | 交换缓冲区内存（MB），与临时表空间文件交换数据，页大小 256KB |
| system_sga_mem | INTEGER | - | - | 离线修改，重启 | 系统全局排序内存（MB） |
| xlog_buff_mem | INTEGER | - | - | 离线修改，重启 | WAL 日志缓冲区内存 |

## 网络侦听参数

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| listen_port | INTEGER | 5138 | [1024, 65535] | 离线修改，重启 | 数据库服务监听端口，端口冲突导致启动失败 |
| max_conn_num | INTEGER | 1000 | [0, 10000] | 在线修改 | SELECT 监听模型下单节点最大连接数，对 POLL 模型不生效 |
| max_act_conn_num | INTEGER | 0 | [0, 1024] | 离线修改，重启 | 最大活动事务连接数。0=不区分；>0 优先保证活跃事务连接的 TCP 监听 |
| max_idle_time | INTEGER | 3600 | [0, 86400] | 在线修改 | 空闲连接最大闲置时间（秒），超时自动清理并记载 TRACE 日志 |
| login_timeout | INTEGER | 30 | [10, 300] | 在线修改 | 登录认证超时时间（秒），判定窗口为 TCP 握手后到身份校验前 |
| nio_timeout | INTEGER | 100 | [10, 10000] | 离线修改，重启 | 网络读写超时（秒），接收超时报 E1010，发送超时报 E1011 |
| conn_fail_cnt | INTEGER | 3 | [2, 100] | 在线修改 | 3 分钟内连续认证失败达此阈值，禁止该 IP 3 分钟。记录在 SYS_FORBIDDEN_IPS |
| session_per_user | INTEGER | 1000 | [0, 10000] | 在线修改 | 用户连接配额。超过上报 E18019。在线修改不影响已有用户和连接 |
| def_charset | VARCHAR | GBK | {UTF8,GBK,GB18030,BINARY} | 在线修改 | 新建用户库和新建连接的默认字符集 |
| def_timefmt | VARCHAR | 'YYYY-MM-DD HH24:MI:SS' | 两种格式 | 在线修改 | 新建连接和 TO_DATE/TO_CHAR 的默认时间格式 |
| def_timezone | VARCHAR | 'GMT+08:00' | ['GMT-12:59','GMT+14:59'] | 在线修改 | 默认时区，必须以 GMT 前缀。启动时确定，在线更改不同步系统时钟 |
| send_warning | BOOLEAN | TRUE | TRUE/FALSE | 在线修改 | 是否向客户端发送警告级别异常（除零、字符串截断、配额不足等） |
| use_std_nio | BOOLEAN | TRUE | TRUE/FALSE | 离线修改，重启 | 网络监听模型。TRUE=SELECT（最多 1024 fd），FALSE=POLL（最多 10240 fd）。Windows 仅支持 SELECT |

## 系统运行(跟踪)日志参数

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| error_level | INTEGER | 3 | [0, 10] | 在线修改 | 错误日志最低记载等级。0=不记载，1=仅系统级异常，2=不记载警告，>=3 记载所有 |
| errlog_size | INTEGER | 100 | [1, 1024] | 在线修改 | 日志分割长度（MB），ERROR/EVENT/COMMAND/TRACE/SLOWSQL.LOG 超过此值自动归档 |
| reg_command | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 记录所有 SQL 命令到 COMMAND.LOG（UTF8 编码，>16KB 截断） |
| reg_ddl | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 记录 DDL 命令到 COMMAND.LOG（无视 reg_command 设置） |
| slow_sql_time | INTEGER | - | - | 在线修改 | 慢 SQL 阈值（毫秒），超过记录到 SLOWSQL.LOG |
| core_dump | BOOLEAN | FALSE | TRUE/FALSE | 离线修改，重启 | 崩溃时是否产生 Core Dump。FALSE 时写堆栈跟踪日志 |
| is_zh | BOOLEAN | FALSE | TRUE/FALSE | 离线修改，重启 | 标准输出日志是否使用中文 |
| license_prompt_day | INTEGER | 30 | [0, 1000] | 在线修改 | 许可证到期提前提示天数，0=不提示 |
| trace_login | BOOLEAN | - | TRUE/FALSE | 在线修改 | 是否跟踪登录 |

### 日志文件说明

所有日志超过 errlog_size 自动归档，格式 `ERROR_yy_mm_dd_hh_mi_ss.LOG`。可通过系统表 SYS_COMMAND_LOG / SYS_ALL_COMMAND_LOG 查看命令日志。

## 数据备份与同步复制参数

| 参数名 | 类型 | 默认值 | 修改方式 | 说明 |
|--------|------|--------|----------|------|
| enable_recycle | BOOLEAN | - | 在线修改 | 是否启用回收站 |
| log_archive_mode | BOOLEAN | - | - | 是否启用归档模式 |
| log_supplement | BOOLEAN | - | - | 是否启用日志补充 |
| max_allow_lob_len | INTEGER | - | - | 最大允许 LOB 长度 |
| modify_log_parti_num | INTEGER | - | - | 变更日志分区数 |
| modify_log_pice_len | INTEGER | - | - | 变更日志片段长度 |
| trigger_modify_log_level | INTEGER | - | - | 触发变更日志级别 |
| ignore_when_meta_err | BOOLEAN | - | - | 元数据错误时是否忽略 |

## 系统分析参数

| 参数名 | 类型 | 默认值 | 修改方式 | 说明 |
|--------|------|--------|----------|------|
| enable_analyze | BOOLEAN | - | 在线修改 | 是否启用自动分析 |
| analyze_mode | INTEGER | - | 在线修改 | 分析模式 |
| analyze_level | INTEGER | - | 在线修改 | 分析级别 |
| analyze_threshold | INTEGER | - | 在线修改 | 分析阈值 |
| analyze_time | VARCHAR | - | 在线修改 | 分析时间 |
| enable_monitor | BOOLEAN | - | 在线修改 | 是否启用监控 |
| histogram_bucket | INTEGER | - | 在线修改 | 直方图桶数 |
| debug_flag | INTEGER | - | 在线修改 | 调试标志 |

## 兼容性参数

| 参数名 | 类型 | 默认值 | 取值范围 | 修改方式 | 说明 |
|--------|------|--------|----------|----------|------|
| def_compatible_mode | VARCHAR | NONE | {NONE,ORACLE,MYSQL,POSTGRESQL} | 在线修改 | 异构库兼容模式，影响后续新连接 |
| def_group_by_mode | INTEGER | 0 | {0,1} | 在线修改 | GROUP BY 模式。0=SQL 标准（非聚合列必须在 GROUP BY 中），1=MySQL 方言 |
| def_identity_mode | INTEGER | 0 | {0,1,2} | 在线修改 | 自增列模式。0=NULL 报 E16005，1=NULL 用自增值，2=NULL 或 0 用自增值 |
| use_old_product_name | BOOLEAN | FALSE | TRUE/FALSE | 在线修改 | 产品名称。FALSE=XuguDB，TRUE=XuGu SQL Server |

### 兼容模式详解

**ORACLE**：标识符转大写；兼容 VAR_POP/VARIANCE/STDDEV 等聚合；支持强制创建存储过程（编译失败不报错）；COALESCE 忽略空值；LEAST/GREATEST 含 NULL 返回 NULL；兼容 DATE 类型。

**MYSQL**：兼容二进制与字符串类型转换；ACOS 参数越界返回 NULL；TRUNCATE 重置自增序列；非字段表达式无别名时展示表达式；兼容部分日期格式。

**POSTGRESQL**：标识符转小写；浮点数向整型转换截断边界值小数；十六进制字母小写（HEX 函数）。
