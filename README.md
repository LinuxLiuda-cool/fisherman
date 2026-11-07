# FisherMan（渔翁）

使用 Rust 构建的，面向定制化的轻量化 APP 运行托管平台。

FisherMan（简称 fm），把「应用」当作可被操控的对象：fm 是鱼竿，应用是水里的鱼。鱼不因鱼竿收线而消失 —— 与应用之间完全解耦，fm 停止运行，应用继续运行；fm 重启后重新接管。

> 项目当前处于开发阶段，接口与数据表仍可能变动。

---

## 核心特性

- **控制面 / 数据面分离**：fm 与调度单元完全解耦，自身可崩溃可重启，启动时通过 takeover 重新接管应用。
- **两态分离**：数据库只存「意图态 Desired + 静态元数据」，运行中 / 发布中 / 回滚中等瞬态一律实时观测，不落库。
- **声明式生命周期**：注册 / 发布 / 回滚 / 删除 / 控制 / 定时发布统一走 life_cycle，细化到最小调度单元。
- **版本化存储**：软件包全版本存储用户可控，每次发布即一个版本，可回滚。
- **保活与自愈**：单一状态机收口运行期巡检（总闸 / 执行开关 / 易失暂停三轴），异常死亡、自愈动作、熔断均可追溯。
- **分层日志留存**：应用 → 版本 → 每次运行 → 分段，支持在线切片（FALLOC_FL_COLLAPSE_RANGE 前缀摘除），不丢字节、不阻塞应用。
- **附件（annex）共享数据区**：跨应用共享的持久化目录，多对多绑定到应用虚拟根，默认只读，改动留痕可恢复。
- **Runtime 管理**：对支持的应用类型支持 Runtime 的快速构建、使用、管理。
- **完整 Web 控制台**：概览、应用控制台、运行池、配置中心、仓库、证书、附件、运行记录、操作日志、系统管理等。

## 支持的应用类型

| stype | 说明 | 依赖 runtime | 控制动作 |
|---|---|---|---|
| java | Spring Boot 等可执行 Jar | JDK | start / stop / restart |
| static | HTML / 前端静态资源 | nginx | enable / disable |
| container | 容器应用（docker / podman / ctr） | 容器化引擎 | start / stop / restart |
| python | CPython 应用 | Python解释器 | start / stop / restart |
| nodejs | Node.js 应用 | Node.js | start / stop / restart |
| php | PHP CLI 应用 | PHP编译器 | start / stop / restart |
| csharp | .NET 应用 | .NET Runtime | start / stop / restart |
| onefile | 单二进制程序（免 runtime） | — | start / stop / restart |

runtime 缺失时对应类型不可注册；已注册应用降级为只读能力（删除 / 日志 / 仓库）。

## 技术栈

- 后端：Rust（edition 2024）· axum · tokio · sea-orm 2.0 · systemd（D-Bus）
- 数据库：SQLite（WAL，可选 SQLCipher 加密）/ MySQL
- 前端：React 19 · Umi Max · Ant Design 5 · TypeScript · Monaco Editor
- 构建 / 部署：Cargo · bun · 静态链接（musl）· systemd

## 快速开始

### 环境要求

- Linux（x86_64 / aarch64），需 root 运行（启动前校验）
- systemd 环境（服务托管与状态探测依赖）

### 安装

```bash
tar -xJf fisherman-.tar.xz && cd fisherman-
bash update.sh
```

`update.sh` 会依次完成：停止旧服务 → 备份旧二进制与数据 → 覆盖 `sbin/`  → 安装配置（已存在的 `/usr/local/FisherMan/conf/fm.toml` 不覆盖）→ 启动服务。

配置与数据目录：

- 安装目录：`/usr/local/FisherMan`（`sbin/fm`、`conf/fm.toml`）
- 数据目录：由 `server.path.data_root` 决定，首次启动自动建库并执行迁移

访问 `http://<服务器 IP>:10586/`。

也可直接前台运行（要求 systemd 环境）：

```bash
/usr/local/FisherMan/sbin/fm daemon -d
```

### 配置文件

配置文件为 `fm.toml`（示例见 `conf/fm.toml`），修改后需重启服务。

```toml
[server]
ips = "0.0.0.0"        # 监听地址，支持字符串或列表
port = 10586           # 监听端口（Web 控制台与 /api 共用）
run_level = "INFO"     # INFO | DEBUG

[server.path]
data_root = "/data/FisherMan"   # 数据根目录（日志、存储、运行池等）

[auth]
key = "<请替换为随机密钥>"       # 接口加密密钥，同时用于 SQLite 加密
access_time_out = 600
refresh_time_out = 3600

[db]
type = "sqlite"        # sqlite | mysql
# type = "mysql" 时补充 host / port / username / password / database
```

> 安全提示：`auth.key` 参与接口加密与数据库加密，部署前务必替换为随机值，且不要提交到版本库。

## 路线图

- [x] java / static 全生命周期闭环，运行池与版本回滚
- [x] 附件（annex）池、绑定、文件操作与修改记录
- [x] 证书模块（国密算法套件）
- [x] 配置中心与配置注入
- [x] 定时发布（延迟触发）
- [x] 分层日志留存与历史日志检索
- [x] 保活体系重建（状态机 + 自愈留痕）
- [ ] python / 容器化 支持
- [ ] NodeJS / 单二进制程序 支持
- [ ] PHP / C# 支持
- [ ] 容器托管增强（镜像仓库、网络）
- [ ] 节点对等联邦与跨节点迁移
- [ ] 全量审计与精细权限（四维授权）
- [ ] 编排自愈与调度 DAG

## 贡献

欢迎通过 Issue 反馈问题与建议。
