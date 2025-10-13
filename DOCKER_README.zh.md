# Docker 设置文档

Open WebUI 完整的 Docker 设置，包含 Rust 后端、PostgreSQL、Redis、Socket.IO 和前端。

## 文档索引

选择适合您需求的指南：

### [快速入门指南](DOCKER_QUICKSTART.zh.md)
**适合想要快速运行应用的用户**
- 3分钟快速开始
- 基本命令和故障排除
- 常用操作
- 生产部署基础

### [开发指南](DOCKER_DEV.zh.md)
**适合正在开发代码的开发者**
- 本地开发设置
- 在 Docker 中运行服务的同时进行本地开发
- 调试技巧
- 数据库管理
- 性能优化

### [完整设置指南](DOCKER_SETUP.zh.md)
**适合需要详细信息和高级用法**
- 架构说明
- 服务详情
- 卷管理
- 备份和恢复程序
- 生产部署
- 全面故障排除

## 架构概览

```
┌──────────────────────────────────────────────────────────┐
│                    Docker Compose 栈                      │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│  │  前端       │  │  Socket.IO   │  │  Rust 后端      │ │
│  │  (SvelteKit)│──│  桥接        │──│  (Actix-web)    │ │
│  │  端口 3000  │  │  端口 8081   │  │  端口 8080      │ │
│  └──────┬──────┘  └──────┬───────┘  └────────┬─────────┘ │
│         │                │                   │           │
│         └────────────────┴───────────────────┘           │
│                          │                               │
│         ┌────────────────┴────────────────┐              │
│         │                                 │              │
│  ┌──────▼───────┐              ┌─────────▼────────┐     │
│  │  PostgreSQL  │              │      Redis        │     │
│  │  端口 5432   │              │    端口 6379      │     │
│  └──────────────┘              └───────────────────┘     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

## 快速命令

### 首次设置
```bash
# 1. 设置环境
./docker-manage.sh setup

# 2. 启动所有服务
./docker-manage.sh start

# 3. 访问应用
open http://localhost:3000
```

### 日常开发
```bash
# 启动
./docker-manage.sh start

# 查看日志
./docker-manage.sh logs

# 更改后重启
./docker-manage.sh restart rust-backend

# 停止
./docker-manage.sh stop
```

### 维护
```bash
# 检查健康状态
./docker-manage.sh health

# 备份
./docker-manage.sh backup

# 查看状态
./docker-manage.sh status
```

## 包含的内容

### 生产栈 (`docker-compose.yaml`)
- **PostgreSQL 16** - 主数据库
- **Redis 7** - 缓存和会话管理
- **Rust 后端** - API 服务器 (Actix-web)
- **Socket.IO 桥接** - 实时 WebSocket 支持 (Python)
- **前端** - SvelteKit UI 与 Python 后端

### 开发栈 (`docker-compose.dev.yaml`)
- PostgreSQL、Redis、Socket.IO 桥接
- **pgAdmin** - 数据库管理 UI
- **Redis Commander** - Redis 管理 UI
- 优化用于 Rust 后端和前端在 Docker 外运行的本地开发

## 关键特性

### ✅ 数据库迁移
- 启动时自动迁移
- 迁移脚本位于 `rust-backend/migrations/postgres/`
- 基于 SQLx 的迁移系统

### ✅ 数据持久化
- 所有数据存储在 Docker 卷中
- 轻松备份和恢复
- 独立的开发和生产卷

### ✅ 健康检查
- 所有服务都有健康检查
- 正确的启动顺序
- 自动管理依赖关系

### ✅ 实时特性
- Socket.IO 用于 WebSocket 支持
- Redis 支持可扩展性
- 频道和聊天事件

### ✅ 开发工具
- pgAdmin 用于数据库检查
- Redis Commander 用于缓存检查
- 热重载支持
- 详细日志记录

## 要求

- Docker 24.0+
- Docker Compose 2.0+
- 最少 4GB 内存
- 10GB 可用磁盘空间

## 默认端口

| 服务 | 端口 | URL |
|---------|------|-----|
| 前端 | 3000 | http://localhost:3000 |
| Rust 后端 | 8080 | http://localhost:8080 |
| Socket.IO | 8081 | http://localhost:8081 |
| PostgreSQL | 5432 | localhost:5432 |
| Redis | 6379 | localhost:6379 |
| pgAdmin | 5050 | http://localhost:5050 |
| Redis Commander | 8082 | http://localhost:8082 |

所有端口都可通过 `.env` 文件配置。

## 安全注意事项

### 开发环境（默认）
- `env.example` 中使用简单密码
- 开放的 CORS 策略
- 所有功能已启用
- ⚠️ **不适合生产使用**

### 生产环境
必需的更改：
1. 生成强 `WEBUI_SECRET_KEY`
2. 更改 `POSTGRES_PASSWORD`
3. 正确配置 CORS
4. 使用 HTTPS（反向代理）
5. 启用防火墙规则
6. 定期备份

查看[完整设置指南](DOCKER_SETUP.zh.md)了解生产部署。

## 数据流

```
用户请求
    │
    ▼
前端（浏览器）
    │
    ├─► REST API ──────► Rust 后端 ──┬─► PostgreSQL
    │                                └─► Redis
    │
    └─► WebSocket ─────► Socket.IO ──┬─► Rust 后端
                         桥接         └─► Redis
```

## 故障排除

### 服务无法启动
```bash
./docker-manage.sh health
docker-compose logs
```

### 端口冲突
```bash
lsof -i :3000
lsof -i :8080
```

### 数据库问题
```bash
./docker-manage.sh shell postgres
docker-compose logs postgres
```

### 重置所有内容
```bash
./docker-manage.sh clean  # ⚠️ 删除所有数据
./docker-manage.sh start
```

## 📖 文档结构

```
DOCKER_README.zh.md        ← 您在这里（概览）
├── DOCKER_QUICKSTART.zh.md   ← 3分钟快速入门
├── DOCKER_SETUP.zh.md        ← 完整设置指南
└── DOCKER_DEV.zh.md          ← 开发工作流程
```

## 🛠️ 文件概览

### 配置
- `docker-compose.yaml` - 生产栈
- `docker-compose.dev.yaml` - 开发栈
- `env.example` - 环境变量模板
- `.dockerignore` - Docker 构建忽略规则

### Dockerfiles
- `Dockerfile` - 前端（SvelteKit + Python）
- `rust-backend/Dockerfile` - Rust 后端
- `rust-backend/Dockerfile.socketio` - Socket.IO 桥接

### 脚本
- `docker-manage.sh` - 管理辅助脚本
- `rust-backend/build.sh` - Rust 构建脚本

### 文档
- `DOCKER_README.zh.md` - 本文件
- `DOCKER_QUICKSTART.zh.md` - 快速入门
- `DOCKER_SETUP.zh.md` - 完整指南
- `DOCKER_DEV.zh.md` - 开发指南

## 🚀 开始使用

### 新用户（只想运行它）
1. 阅读 [DOCKER_QUICKSTART.zh.md](DOCKER_QUICKSTART.zh.md)
2. 运行 `./docker-manage.sh setup`
3. 运行 `./docker-manage.sh start`
4. 打开 http://localhost:3000

### 开发者（正在开发代码）
1. 阅读 [DOCKER_DEV.zh.md](DOCKER_DEV.zh.md)
2. 启动基础设施：`docker-compose -f docker-compose.dev.yaml up -d`
3. 本地运行 Rust 后端：`cd rust-backend && cargo run`
4. 本地运行前端：`npm run dev`
5. 访问 http://localhost:5173

### 系统管理员（生产环境）
1. 阅读 [DOCKER_SETUP.zh.md](DOCKER_SETUP.zh.md)
2. 使用生产值配置 `.env`
3. 设置反向代理（nginx/Traefik/Caddy）
4. 配置 SSL/TLS
5. 设置备份
6. 使用 `docker-compose up -d` 部署

## 💡 提示

- 使用 `./docker-manage.sh` 方便操作
- 使用 `./docker-manage.sh logs [service]` 检查日志
- 使用 `./docker-manage.sh backup` 定期备份
- 使用开发 compose 进行本地开发
- 启用工具配置进行调试：`--profile tools`

## 🆘 支持

- **日志**：`./docker-manage.sh logs`
- **状态**：`./docker-manage.sh status`
- **健康**：`./docker-manage.sh health`
- **帮助**：`./docker-manage.sh help`

详细信息请参阅具体指南：
- 快速入门：`DOCKER_QUICKSTART.zh.md`
- 开发：`DOCKER_DEV.zh.md`
- 完整指南：`DOCKER_SETUP.zh.md`

---

**选择您的路径：**
- 🚀 想现在就运行？ → [快速入门](DOCKER_QUICKSTART.zh.md)
- 🔧 开发者？ → [开发指南](DOCKER_DEV.zh.md)
- 📖 需要详细信息？ → [完整设置](DOCKER_SETUP.zh.md)

