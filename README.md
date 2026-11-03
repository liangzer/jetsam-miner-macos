# JetsamForge · Jetsam (JTM) 挖矿工具包 for macOS

> **Apple Silicon 原生 arm64 预编译包 —— 解压即挖。** 不需要 Rust 工具链，不需要花 90 分钟自己生成四份证明矩阵。
>
> **Keywords**: Jetsam / JTM / Jetsam miner / Jetsam mining / macOS miner / Apple Silicon miner / arm64 / crypto mining / macOS 挖矿 / 苹果芯片挖矿 / Docker

---

## 这是什么

[Jetsam](https://github.com/jetsam-chain/jetsam) 是一条用 Rust 写的 PoW 链（JTM）。官方只发 Linux / Windows 二进制，**没有 macOS 版**，而且自建节点要先跑一个 90 分钟的证明矩阵生成流程。

这个仓库把这两件事都做完了：

- 编译好的 **macOS arm64 原生节点 + 矿工**（M1 / M2 / M3 / M4 / M5 全部适用）
- **四份 HistoryStep 证明矩阵**（v1 / v1.3 / v1.5 / v1.6）全部内嵌进二进制，开机即用
- 附带**图形控制台**（浏览器打开 `127.0.0.1:8080`）和**终端仪表盘**两种用法
- 另有一个 **Docker 镜像包**，任何平台都能跑

## 下载

到 [Releases](https://github.com/liangzer/jetsam-miner-macos/releases/latest) 拿最新版（当前 **v1.6.1**）：

| 文件 | 适用 | 大小 |
|---|---|---|
| `jetsam-macos-arm64.tar.gz` | Apple Silicon（M1/M2/M3/M4/M5） | ~106 MB |
| `jetsam-docker.tar.gz` | 任意平台（Docker） | ~121 MB |

每个附件旁边都有 SHA256，下完可以自己核对：

```bash
shasum -a 256 jetsam-macos-arm64.tar.gz
```

## 用法一：图形控制台（推荐新手）

```bash
tar -xzf jetsam-macos-arm64.tar.gz && cd jetsam-macos-arm64
./start-miner.sh          # 浏览器自动打开 127.0.0.1:8080
```

页面上能看：区块高度、同步状态、算力、线程数、钱包地址、挖到的区块。

## 用法二：终端命令行（日志看得清）

```bash
cd jetsam-macos-arm64
./start-cli.sh            # 前台跑，日志直接打在终端
# 或者
./watch.sh                # 另开一个窗口看实时仪表盘
```

## 用法三：Docker（Linux / NAS / 群晖都能跑）

```bash
docker load -i jetsam-docker.tar.gz
docker run -d --name jtm \
  -p 8080:8080 -p 9701:9701 \
  -v ~/jetsam-data:/data \
  jetsam-miner:v1.6.1
```

## 钱包与数据

- 数据目录在 `~/jetsam-data`（或启动时用 `--data-dir` 指定）
- 钱包文件 `wallet.key` 就是这个币的私钥，**自己备份好**
- 升级只换二进制，钱包和链数据全保留，**不用重新同步**

## 版本与网络

当前发布基于官方 **v1.6.1**（2026-10-30 的安全修复版，**不是硬分叉**）：

- v1.6 共识规则已于 **2026-10-30 06:10 北京时间** 在 block **46,080** 激活
- v1.6.1 修了 gossipsub PRUNE backoff 溢出（CVE-2026-33040 / CVE-2026-34219，一条恶意消息就能打崩节点）
- **v1.5.x 及更早的版本已孤立**，挖出来的块主网不认

## 系统要求

- macOS 12+，Apple Silicon（M 系列）
- 磁盘 5 GB 起（链数据会涨）
- 内存 8 GB 够用

## 关于挖矿收益（说实话）

- M4 / 9 线程实测约 **5.3 kH/s**
- 出块是概率事件，平均 **3–4 天** 出一个，区块奖励约 **25 JTM**
- **JTM 目前没有交易所、没有报价**，挖出来的币暂时无法变现

这不是能赚钱的生意，是给想跑一条 PoW 链的人省掉编译和配置的麻烦。

## 构建信息

二进制来自 Jetsam 官方源码 **v1.6.1**（Apache-2.0），未签名、未经审计，自负风险。

本机只做了三处**不改共识行为**的适配：

1. macOS 上关掉 jemalloc 的 `background_threads`（macOS 不支持 pthread 后台线程，会刷警告）
2. internal miner 预留核数 `2 → 1`（10 核 M4 从 8 线程提到 9 线程）
3. release profile 改 `lto = "fat"` + `panic = "abort"`（与官方发布版一致）

## License

Apache-2.0。上游代码见 [jetsam-chain/jetsam](https://github.com/jetsam-chain/jetsam)。
