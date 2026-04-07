# PHP PDO 驱动安装与环境搭建

## 环境要求

- PHP 运行环境（PHP 7.x+ 或 PHP 8.x）
- XuguDB-PDO 扩展（pdo_xugusql）
- 虚谷数据库服务端已安装并启动

## 平台支持

| 平台 | CPU 架构 | 扩展文件 |
|------|----------|---------|
| Windows | x86, 64-bit | pdo_xugusql.dll |
| Linux | x86, 64-bit | pdo_xugusql.so |
| Linux | ARM, 64-bit | pdo_xugusql.so |

## Windows 安装

1. 安装 PHP（下载地址：https://windows.php.net/downloads/releases/）
2. 解压到安装目录
3. 将 `pdo_xugusql.dll` 复制到 PHP 的 `ext` 目录
4. 将 `php.ini-development` 或 `php.ini-production` 复制为 `php.ini`
5. 编辑 `php.ini`：

```ini
# 取消注释并设置扩展目录
extension_dir = "ext"

# 在文件末尾添加
extension=pdo_xugusql
```

6. 验证安装：`php -m` 查看 PHP Modules 中是否包含 `pdo_xugusql`

## Linux 安装

1. 安装 PHP（编译安装或包管理器安装）

```bash
# 编译安装示例
tar -xzf php-x.x.x.tar.gz
cd php-x.x.x
./configure --prefix=/usr/local/php
make && make install
```

2. 配置 `php.ini`

```bash
# 查找 php.ini 路径
php --ini | grep php.ini

# 复制配置文件模板
cp php.ini-development /usr/local/php/lib/php.ini
```

3. 将 `pdo_xugusql.so` 放入 PHP 扩展目录

4. 编辑 `php.ini` 添加：

```ini
extension=pdo_xugusql
```

5. 验证：`php -m` 查看是否包含 `pdo_xugusql`

## 注意事项

- CPU 架构必须匹配（64-bit PHP 需使用 64-bit 扩展）
- XuguDB-PDO 基于标准 PHP PDO 接口，使用方式与其他 PDO 驱动类似
- 驱动名称为 `"xugusql"`，在 PDO DSN 中使用
