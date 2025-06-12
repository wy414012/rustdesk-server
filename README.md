# OpenRustDesk Server 文档

> ⚠️ **重要通知**  
> **main分支不提供Docker镜像支持**  
> 如需Docker部署，请使用其他专门分支或参考官方文档获取Docker镜像

RustDesk 自托管服务器实现，提供 ID 注册/中继服务功能。包含以下核心组件：

- `hbbs` - RustDesk ID/会面服务器
- `hbbr` - RustDesk 中继服务器
- `rustdesk-utils` - 命令行工具

支持通过以下方式部署：
1. 直接运行二进制文件
2. .deb 包安装

关键功能：
- 支持多架构（amd64/arm64/armv7）
- 提供密钥对加密功能
- 可通过环境变量灵活配置
- 支持数据库持久化存储

# OpenRustDesk Server

[![构建](https://github.com/wy414012/rustdesk-server/actions/workflows/build.yaml/badge.svg)](https://github.com/wy414012/rustdesk-server/actions/workflows/build.yaml)

[**下载**](https://github.com/wy414012/rustdesk-server/releases)

[**手册**](https://rustdesk.com/docs/en/self-host/)

[**常见问题**](https://github.com/rustdesk/rustdesk/wiki/FAQ)

自托管您自己的 RustDesk 服务器，它是免费和开源的。

## 如何手动构建

```bash
cargo build --release
```

在 `target/release` 目录下会生成三个可执行文件：

- hbbs - RustDesk ID/会面服务器
- hbbr - RustDesk 中继服务器
- rustdesk-utils - RustDesk 命令行工具

您可以在 [Releases](https://github.com/wy414012/rustdesk-server/releases) 页面找到更新的二进制文件。

如果您需要额外功能，[RustDesk Server Pro](https://rustdesk.com/pricing.html) 可能更适合您。

如果您想开发自己的服务器，[rustdesk-server-demo](https://github.com/rustdesk/rustdesk-server-demo) 可能是比这个仓库更好的起点。

## 如何创建密钥对

密钥对用于加密；您可以按前面所述提供它，但您需要一种方法来创建它。

您可以使用以下命令生成密钥对：

```bash
/usr/bin/rustdesk-utils genkeypair
```

输出将类似于以下内容：

```text
公钥:  8BLLhtzUBU/XKAH4mep3p+IX4DSApe7qbAwNH9nv4yA=
私钥:  egAVd44u33ZEUIDTtksGcHeVeAwywarEdHmf99KM5ajwEsuG3NQFT9coAfiZ6nen4hfgNICl7upsDA0f2e/jIA==
```

## .deb 包

每个二进制文件都有单独的 .deb 包，您可以在 [Releases](https://github.com/wy414012/rustdesk-server/releases) 中找到它们。
这些包适用于以下发行版：

- Ubuntu 24.04 LTS
- Ubuntu 22.04 LTS
- Ubuntu 20.04 LTS
- Ubuntu 18.04 LTS
- Debian 12 bookworm
- Debian 11 bullseye
- Debian 10 buster

##### 环境变量

`hbbs` 和 `hbbr` 可以通过以下环境变量进行配置。你可以像往常一样指定这些变量，或者使用 `.env` 文件。

| 变量名 | 适用二进制文件 | 描述 |
| --- | --- | --- |
| ALWAYS_USE_RELAY | hbbs | 如果设置为 **"Y"**，则禁止直接对等连接 |
| DB_URL | hbbs | 数据库文件路径 |
| DOWNGRADE_START_CHECK | hbbr | 下降检查延迟（秒） |
| DOWNGRADE_THRESHOLD | hbbr | 下降检查阈值（bit/ms） |
| KEY | hbbs/hbbr | 如果设置，则强制使用特定密钥；如果设置为 **"_"**，则强制使用任何密钥 |
| LIMIT_SPEED | hbbr | 速度限制（Mb/s） |
| PORT | hbbs/hbbr | 监听端口（hbbs 为 41116，hbbr 为 41117） |
| RELAY_SERVERS | hbbs | 运行中继 hbbr 的机器的 IP地址或者，用逗号分隔可以指定多个 |
| RENDZVOUS_SERVERS | hbbs | 运行会面 hbbs 的机器的 IP地址或者，用逗号分隔可以指定多个 |
| RUST_LOG | 所有 | 设置调试级别（error\|warn\|info\|debug\|trace） |
| SINGLE_BANDWIDTH | hbbr | 单个连接的最大带宽（Mb/s） |
| TOTAL_BANDWIDTH | hbbr | 总最大带宽（Mb/s） |