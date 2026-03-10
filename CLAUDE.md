# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目简介

ServerStatus-Rust 是一个用 Rust 重写的服务器监控工具，包含服务端和客户端。通过 gRPC/HTTP 协议上报服务器状态数据，支持多平台（Linux、MacOS、Windows、Android、Raspberry Pi）。

## 构建命令

```bash
# 构建所有组件
cargo build --release

# 构建单个组件
cargo build --release -p stat_server
cargo build --release -p stat_client

# 运行测试
cargo test --verbose

# 代码检查
cargo clippy

# 安装组件
cargo install stat_server
cargo install stat_client
```

## 运行命令

### 服务端
```bash
# 正常运行
./stat_server -c config.toml

# 调试模式
RUST_BACKTRACE=1 RUST_LOG=trace ./stat_server -c config.toml

# 测试配置文件
./stat_server -c config.toml -t

# 发送测试通知
./stat_server -c config.toml --notify-test
```

### 客户端
```bash
# HTTP 协议上报
./stat_client -a "http://127.0.0.1:8080/report" -u h1 -p p1

# gRPC 协议上报
./stat_client -a "grpc://127.0.0.1:9394" -u h1 -p p1

# 动态注册模式
./stat_client -a "http://127.0.0.1:8080/report" -g g1 -p pp --alias "$(hostname)"

# 查看帮助
./stat_client -h
```

## Docker 部署
```bash
docker-compose up -d
```

## 代码架构

### Workspace 结构
```
.
├── common/      # 共享代码（protobuf 定义、工具函数）
├── server/      # 服务端（Axum HTTP + Tonic gRPC）
└── client/      # 客户端（数据采集和上报）
```

### common (`common/src/lib.rs`)
- `server_status` 模块：protobuf 生成的 gRPC 代码
- `utils` 模块：工具函数（如 `bytes2human` 流量单位转换）
- protobuf 定义在 `common/proto/server_status.proto`

### server (`server/src/`)
核心模块：
- `main.rs` - 入口，初始化配置、通知器、HTTP/gRPC 服务
- `config.rs` - TOML 配置文件解析
- `grpc.rs` - gRPC 报告接收服务
- `http.rs` - HTTP 服务（前端页面、API）
- `auth.rs` / `jwt.rs` - JWT 认证
- `stats.rs` - 主机状态管理
- `payload.rs` - 数据结构定义
- `notifier/` - 通知模块（Telegram、微信、邮件、Webhook）
- `assets.rs` - 静态资源嵌入
- `jinja.rs` - Jinja2 模板引擎

### client (`client/src/`)
核心模块：
- `main.rs` - 入口，CLI 参数解析，上报逻辑
- `status.rs` / `sys_info.rs` - 系统信息采集（CPU、内存、磁盘、网络）
- `vnstat.rs` - vnstat 流量统计
- `geoip/` - IP 地理位置查询（多源支持）
- `grpc.rs` - gRPC 客户端

## 配置说明

服务端配置文件 `config.toml` 支持：
- `hosts` - 静态主机列表配置
- `hosts_group` - 动态注册模式配置
- `tgbot` / `wechat` / `email` / `webhook` - 通知渠道配置
- `grpc_tls` - gRPC TLS/mTLS 配置

## 开发注意事项

- Rust 最低版本：1.94
- Protocol Buffers：使用 `tonic-prost` 和 `tonic-build` 生成代码
- 前端资源通过 `rust-embed` 嵌入二进制
- 日志使用 `pretty_env_logger`，通过 `RUST_LOG` 环境变量控制
