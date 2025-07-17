# RustDesk 自建服务器指南

[![构建状态](https://github.com/wy414012/rustdesk-server/actions/workflows/build.yaml/badge.svg)](https://github.com/wy414012/rustdesk-server/actions/workflows/build.yaml)

[**下载最新版本**](https://github.com/wy414012/rustdesk-server/releases) | 
[**官方文档**](https://rustdesk.com/docs/en/self-host/) | 
[**常见问题解答**](https://github.com/rustdesk/rustdesk/wiki/FAQ)

RustDesk 是一款免费开源的远程桌面软件，通过自建服务器可以获得更好的性能和安全性。

## 快速开始 (Docker方式)

这是最简单的部署方式，适合新手：

1. 确保已安装 Docker 和 docker-compose
2. 创建一个新目录并进入：
   ```bash
   mkdir rustdesk-server && cd rustdesk-server
   ```
3. 创建 `docker-compose.yml` 文件，内容如下：
   ```yaml
   version: '3'
   services:
     hbbs:
       image: wy368/openrustdesk-server:latest
       container_name: hbbs
       ports:
         - 21115:21115
         - 21116:21116
         - 21116:21116/udp
         - 21118:21118
       command: hbbs -r 你的服务器IP:21117
       volumes:
         - ./data:/root
       restart: unless-stopped
     
     hbbr:
       image: wy368/openrustdesk-server:latest
       container_name: hbbr
       ports:
         - 21117:21117
         - 21119:21119
       volumes:
         - ./data:/root
       restart: unless-stopped
   ```
4. 启动服务：
   ```bash
   docker-compose up -d
   ```
5. 在客户端设置中填写你的服务器地址即可使用

> 💡 提示：将"你的服务器IP"替换为你实际的公网IP或域名

## 术语解释

- **hbbs**: RustDesk的ID服务器和会面服务器，负责设备发现和连接建立
- **hbbr**: RustDesk的中继服务器，当直接连接失败时负责转发数据
- **rustdesk-utils**: 命令行工具，用于生成密钥对等管理操作

## 系统要求

- Linux系统(推荐Ubuntu 20.04+或CentOS 7+)
- 至少1GB内存
- 2核CPU
- 10GB可用磁盘空间
- 开放端口21115-21119(TCP和UDP)

## 手动构建指南

1. 安装必要工具：
   ```bash
   # Ubuntu/Debian
   sudo apt update && sudo apt install -y git curl build-essential
   
   # CentOS/RHEL
   sudo yum install -y git curl gcc-c++ make
   ```

2. 安装Rust编程语言：
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   source $HOME/.cargo/env
   ```

3. 获取源代码：
   ```bash
   git clone https://github.com/wy414012/rustdesk-server.git
   cd rustdesk-server
   ```

4. 编译项目：
   ```bash
   cargo build --release
   ```
   编译完成后，在 `target/release` 目录下会生成三个可执行文件。

5. 运行服务器：
   ```bash
   # 启动hbbs(ID服务器)
   ./target/release/hbbs -r 你的服务器IP:21117
   
   # 启动hbbr(中继服务器)
   ./target/release/hbbr
   ```

> 💡 提示：如果不想自己编译，可以直接从[发布页面](https://github.com/wy414012/rustdesk-server/releases)下载预编译的二进制文件。

## 服务器配置

可以通过环境变量或命令行参数配置服务器，以下是常用选项：

### 基本配置

1. **指定服务器地址** (必须)
   ```bash
   ./hbbs -r 你的服务器IP或域名:21117
   ```

2. **修改监听端口** (可选)
   ```bash
   ./hbbs -p 21115 -k 21116 -m 21118
   ./hbbr -p 21117 -m 21119
   ```
   - `-p`: 主端口
   - `-k`: 密钥交换端口
   - `-m`: 管理端口

3. **设置密钥对** (推荐)
   ```bash
   ./rustdesk-utils genkeypair
   ```
   生成的`id_ed25519.pub`和`id_ed25519`文件需要放在客户端使用

### 环境变量配置

| 变量名 | 说明 | 示例 |
|--------|------|------|
| `RUST_LOG` | 设置日志级别 | `RUST_LOG=info` |
| `RUSTDESK_ENCRYPTION` | 启用端到端加密 | `RUSTDESK_ENCRYPTION=1` |

## 测试连接

1. 检查hbbs是否运行：
   ```bash
   telnet 你的服务器IP 21115
   ```

2. 检查hbbr是否运行：
   ```bash
   telnet 你的服务器IP 21117
   ```

3. 查看日志确认无错误：
   ```bash
   docker logs hbbs
   docker logs hbbr
   ```

## 常见问题

❓ **客户端无法连接服务器**
- 检查防火墙是否开放了21115-21119端口
- 确保`-r`参数指定了正确的公网IP或域名
- 检查服务器日志是否有错误

❓ **连接速度慢**
- 尝试使用中继模式(`hbbr`)
- 检查网络带宽是否充足

❓ **如何更新服务器**
```bash
docker-compose pull && docker-compose up -d
```

> 📌 更多问题请参考[官方FAQ](https://github.com/rustdesk/rustdesk/wiki/FAQ)

## api服务器
推荐使用 [lejianwen/rustdesk-api]（https://github.com/lejianwen/rustdesk-api/）

## Docker 镜像

Docker 镜像是在每次 GitHub 发布时自动构建并发布到 [Docker Hub](https://hub.docker.com/r/wy368/openrustdesk-server/) 和 [GitHub Container Registry](https://github.com/wy414012?tab=packages&repo_name=rustdesk-server)。我们有两种类型的镜像。

### 经典镜像

这些镜像是从头开始构建的，包含两个主要的二进制文件（`hbbs` 和 `hbbr`）。它们可以在 [Docker Hub](https://hub.docker.com/r/wy368/openrustdesk-server/) 和 [GitHub Container Registry](https://github.com/wy414012/rustdesk-server/pkgs/container/rustdesk-server) 上找到，支持以下架构：

* amd64
* arm64v8
* armv7

您可以使用 `latest` 标签或主版本标签 `1` 来获取支持的架构：

| 版本       | image:tag                         |
| ------------- | --------------------------------- |
| latest        | `wy368/openrustdesk-server:latest` |
| 主版本 | `wy368/openrustdesk-server:1`      |

您可以直接使用 `docker run` 启动这些镜像，命令如下：

```bash
docker run --name hbbs --net=host -v "$PWD/data:/root" -d wy368/openrustdesk-server:latest hbbs -r <中继服务器IP[:端口]> 
docker run --name hbbr --net=host -v "$PWD/data:/root" -d wy368/openrustdesk-server:latest hbbr 
```

或者不使用 `--net=host`，但 P2P 直接连接将无法工作。

对于使用 SELinux 的系统，需要将 `/root` 替换为 `/root:z` 以确保容器正确运行。或者可以禁用 SELinux 容器隔离，添加选项 `--security-opt label=disable`。

```bash
docker run --name hbbs -p 21115:21115 -p 21116:21116 -p 21116:21116/udp -p 21118:21118 -v "$PWD/data:/root" -d wy368/openrustdesk-server:latest hbbs -r <中继服务器IP[:端口]> 
docker run --name hbbr -p 21117:21117 -p 21119:21119 -v "$PWD/data:/root" -d wy368/openrustdesk-server:latest hbbr 
```

`relay-server-ip` 参数是运行这些容器的服务器的 IP 地址（或 DNS 名称）。可选的 `port` 参数用于当您使用不同于 **21117** 的端口时。

您也可以创建`docker-compose.yml`，使用此配置作为模板：

```yaml
version: '3'

networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: wy368/openrustdesk-server:latest
    command: hbbs -r rustdesk.example.com:21117
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: wy368/openrustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/data
    networks:
      - rustdesk-net
    restart: unless-stopped
```
# 使用
```bash
# 启动并运行在后台
docker compose up -d 
# 关闭
docker compose down

```
编辑第 16 行以指向您的中继服务器（监听端口 21117）。如果需要，您还可以编辑卷行（第 18 行和第 33 行）。

(docker-compose 模板由 @lukebarone 和 @QuiGonLeong 提供)

> [!注意]  
> 在中国，`wy368/openrustdesk-server:latest` 可能需要替换为 Docker Hub 上的最新版本号，例如 `rustdesk-server:1.3.6`。否则，由于镜像加速的原因，可能会拉取旧版本。

> [!注意]  
> 如果您在从 Docker Hub 拉取时遇到问题，请尝试从 [GitHub Container Registry](https://github.com/wy414012/rustdesk-server/pkgs/container/rustdesk-server) 拉取。

## 基于 S6-overlay 的镜像

这些镜像是基于 `busybox:stable` 构建的，并添加了二进制文件（`hbbs` 和 `hbbr`）和 [S6-overlay](https://github.com/just-containers/s6-overlay)。它们可以在 [GitHub Container Registry](https://github.com/wy414012/rustdesk-server/pkgs/container/rustdesk-server) 上找到，支持以下架构：

* amd64
* i386
* arm64v8
* armv7

您可以使用 `latest` 标签或主版本标签 `1` 来获取支持的架构：

| 版本       | image:tag                            |
| ------------- | ------------------------------------ |
| latest        | `wy368/openrustdesk-server:latest` |

您可以直接使用 `docker run` 启动这些镜像，命令如下：

```bash
docker run --name rustdesk-server \ 
  --net=host \
  -e "RELAY=rustdeskrelay.example.com" \
  -e "ENCRYPTED_ONLY=1" \
  -v "$PWD/data:/data" -d wy368/openrustdesk-server:latest
```

或者不使用 `--net=host`，但 P2P 直接连接将无法工作。

```bash
docker run --name rustdesk-server \
  -p 21115:21115 -p 21116:21116 -p 21116:21116/udp \
  -p 21117:21117 -p 21118:21118 -p 21119:21119 \
  -e "RELAY=rustdeskrelay.example.com" \
  -e "ENCRYPTED_ONLY=1" \
  -v "$PWD/data:/data" -d wy368/openrustdesk-server:latest
```

或者您可以创建`docker-compose.yml`文件：

```yaml
version: '3'

networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: wy368/openrustdesk-server-s6:latest
    command: hbbs -r rustdesk.example.com:21117
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: wy368/openrustdesk-server-s6:latest
    command: hbbr
    volumes:
      - ./data:/data
    networks:
      - rustdesk-net
    restart: unless-stopped
```

对于此容器镜像，除了以下 **环境变量** 部分指定的变量外，您还可以使用这些环境变量：

| 变量 | 是否可选 | 描述 |
| --- | --- | --- |
| RELAY | 否 | 运行此容器的机器的 IP 地址/DNS 名称 |
| ENCRYPTED_ONLY | 是 | 如果设置为 **"1"**，则不允许未加密连接 |
| KEY_PUB | 是 | 密钥对的公钥部分 |
| KEY_PRIV | 是 | 密钥对的私钥部分 |

### S6-overlay 基础镜像中的密钥管理

您可以将密钥对保存在 docker 卷中，但最佳实践建议不要将密钥写入文件系统；因此我们提供了几个选项。

在容器启动时，会检查密钥对的存在（`/data/id_ed25519.pub` 和 `/data/id_ed25519`），如果其中一个密钥不存在，则会从环境变量或 docker 秘密中重新创建。
然后会检查密钥对的有效性：如果公钥和私钥不匹配，容器将停止。
如果您没有提供密钥，`hbbs` 将为您生成一个，并将其放置在默认位置。

#### 使用环境变量存储密钥对

您可以使用 docker 环境变量来存储密钥。请参考以下示例：

```bash
docker run --name rustdesk-server \ 
  --net=host \
  -e "RELAY=rustdeskrelay.example.com" \
  -e "ENCRYPTED_ONLY=1" \
  -e "DB_URL=/db/db_v2.sqlite3" \
  -e "KEY_PRIV=FR2j78IxfwJNR+HjLluQ2Nh7eEryEeIZCwiQDPVe+PaITKyShphHAsPLn7So0OqRs92nGvSRdFJnE2MSyrKTIQ==" \
  -e "KEY_PUB=iEyskoaYRwLDy5+0qNDqkbPdpxr0kXRSZxNjEsqykyE=" \
  -v "$PWD/db:/db" -d wy368/openrustdesk-server:latest
```

```yaml
version: '3'

services:
  rustdesk-server:
    container_name: rustdesk-server
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21117:21117
      - 21118:21118
      - 21119:21119
    image: wy368/openrustdesk-server:latest
    environment:
      - "RELAY=rustdesk.example.com:21117"
      - "ENCRYPTED_ONLY=1"
      - "DB_URL=/db/db_v2.sqlite3"
      - "KEY_PRIV=FR2j78IxfwJNR+HjLluQ2Nh7eEryEeIZCwiQDPVe+PaITKyShphHAsPLn7So0OqRs92nGvSRdFJnE2MSyrKTIQ=="
      - "KEY_PUB=iEyskoaYRwLDy5+0qNDqkbPdpxr0kXRSZxNjEsqykyE="
    volumes:
      - ./db:/db
    restart: unless-stopped
```

#### 使用 Docker 秘密存储密钥对

您也可以使用 docker 秘密来存储密钥。
这对于使用 **docker-compose** 或 **Docker Swarm** 的情况非常有用。
请参考以下示例：

```bash
cat secrets/id_ed25519.pub | docker secret create key_pub -
cat secrets/id_ed25519 | docker secret create key_priv -
docker service create --name rustdesk-server \
  --secret key_priv --secret key_pub \
  --net=host \
  -e "RELAY=rustdeskrelay.example.com" \
  -e "ENCRYPTED_ONLY=1" \
  -e "DB_URL=/db/db_v2.sqlite3" \
  --mount "type=bind,source=$PWD/db,destination=/db" \
  wy368/openrustdesk-server:latest
```

```yaml
version: '3'

services:
  rustdesk-server:
    container_name: rustdesk-server
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21117:21117
      - 21118:21118
      - 21119:21119
    image: wy368/openrustdesk-server:latest
    environment:
      - "RELAY=rustdesk.example.com:21117"
      - "ENCRYPTED_ONLY=1"
      - "DB_URL=/db/db_v2.sqlite3"
    volumes:
      - ./db:/db
    restart: unless-stopped
    secrets:
      - key_pub
      - key_priv

secrets:
  key_pub:
    file: secrets/id_ed25519.pub
  key_priv:
    file: secrets/id_ed25519      
```

## 如何创建密钥对

密钥对用于加密；您可以按前面所述提供它，但您需要一种方法来创建它。

您可以使用以下命令生成密钥对：

```bash
/usr/bin/rustdesk-utils genkeypair
```

如果您没有安装 `rustdesk-utils` 包或不想安装，可以使用 docker 执行相同命令：

```bash
docker run --rm --entrypoint /usr/bin/rustdesk-utils  wy368/openrustdesk-server:latest genkeypair
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
| LOGGED_IN_ONLY | hbbs | 如果设置为 **"Y"**，则禁止尚未登录用户发起的远控请求 |
| LIMIT_SPEED | hbbr | 速度限制（Mb/s） |
| PORT | hbbs/hbbr | 监听端口（hbbs 为 21116，hbbr 为 21117） |
| RELAY_SERVERS | hbbs | 运行中继 hbbr 的机器的 IP地址或者，用逗号分隔可以指定多个 |
| RENDZVOUS_SERVERS | hbbs | 运行会面 hbbs 的机器的 IP地址或者，用逗号分隔可以指定多个 |
| RUST_LOG | 所有 | 设置调试级别（error\|warn\|info\|debug\|trace） |
| SINGLE_BANDWIDTH | hbbr | 单个连接的最大带宽（Mb/s） |
| TOTAL_BANDWIDTH | hbbr | 总最大带宽（Mb/s） |