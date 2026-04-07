# C# (.NET) 驱动安装与环境搭建

## 环境要求

- .NET Framework 或 .NET Core 运行环境
- XuguClient 驱动包（XuguClient.dll + 原生库）
- 虚谷数据库服务端已安装并启动

## 平台支持

| 平台 | CPU 架构 | 原生库文件 |
|------|----------|-----------|
| Windows | x86, 64-bit | xugusql.dll |
| Linux | x86, 64-bit | libxugusql.so |
| Linux | ARM, 64-bit | libxugusql.so |

> Windows 需 Windows 10 及以上版本

## Windows 安装

1. 获取 `XuguClient.dll`（托管程序集）和 `xugusql.dll`（原生库）
2. 在项目中添加 `XuguClient.dll` 引用（右键"引用" → "添加引用"）
3. 将 `xugusql.dll` 放到 BIN 目录（与生成的 exe 文件同级目录）

## Linux 安装

1. 获取 `XuguClient.dll` 和 `libxugusql.so`
2. 将两个文件放到与应用程序 exe 同级目录

## 注意事项

- CPU 架构必须匹配（64-bit 应用需使用 64-bit 驱动）
- XuguClient 实现标准 ADO.NET 接口，使用方式类似 SqlClient
- 主要命名空间：需引用 XuguClient 中的类型（XGConnection、XGCommand 等）
