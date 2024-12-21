# Open RustDesk 服务器程序文档

- 本仓库代码支持三方api服务器
- 推荐使用https://github.com/lejianwen/rustdesk-api

#### 目录
- [如何手动构建](#如何手动构建)
- [如何创建密钥对](#如何创建密钥对)
- [Debian 包](#debian-包)
- [环境变量](#环境变量)

#### RustDesk 服务器程序

[![构建状态](https://github.com/wy414012/rustdesk-server/actions/workflows/build.yaml/badge.svg)](https://github.com/wy414012/rustdesk-server/actions/workflows/build.yaml)

[**下载**](https://github.com/wy414012/rustdesk-server/releases)

[**手册**](https://rustdesk.com/docs/en/self-host/)

[**常见问题**](https://github.com/rustdesk/rustdesk/wiki/FAQ)

自建你自己的 RustDesk 服务器，它是免费且开源的。

#### master 分支无 Docker

##### 如何手动构建

```bash
cargo build --release
```

在 `target/release` 目录下将生成三个可执行文件：

- `hbbs` - RustDesk ID/会面服务器
- `hbbr` - RustDesk 中继服务器
- `rustdesk-utils` - RustDesk 命令行工具

你可以在 [Releases](https://github.com/wy414012/rustdesk-server/releases) 页面找到更新的二进制文件。

##### 如何创建密钥对

需要一个密钥对用于加密；你可以提供它，如前所述，但你需要一种方法来创建一个。

你可以使用以下命令生成密钥对：

```bash
/usr/bin/rustdesk-utils genkeypair
```

如果你没有安装（或不想安装）`rustdesk-utils` 包，你可以使用 Docker 运行相同的命令：

```bash
docker run --rm --entrypoint /usr/bin/rustdesk-utils wy368/openrustdesk-server:latest genkeypair
```

输出结果如下：

```text
公钥: 8BLLhtzUBU/XKAH4mep3p+IX4DSApe7qbAwNH9nv4yA=
私钥: egAVd44u33ZEUIDTtksGcHeVeAwywarEdHmf99KM5ajwEsuG3NQFT9coAfiZ6nen4hfgNICl7upsDA0f2e/jIA==
```

##### .deb 包

每个二进制文件都有单独的 .deb 包，你可以在 [Releases](https://github.com/wy414012/rustdesk-server/releases) 页面找到它们。
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
