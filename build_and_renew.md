# New-API 源码更新与重新构建指南

## 概述

本文档描述如何基于本地源码重新构建并更新 New-API Docker 容器，同时保留现有数据和配置（PostgreSQL、Redis 数据不变，端口 3001 不变）。

## 前置条件

- Docker 和 Docker Compose 已安装
- 当前目录包含 New-API 完整源码
- 已有正在运行的 New-API 容器（数据已存在）

## 更新步骤

### 1. 修改 docker-compose.yml

将 `docker-compose.yml` 中的 `new-api` 服务从使用远程镜像改为从本地源码构建：

```yaml
# 修改前
  new-api:
    image: calciumion/new-api:latest

# 修改后
  new-api:
    build: .
```

### 2. 停止现有容器

```bash
docker-compose down
```

此命令会停止并移除现有容器，但不会删除 volumes 中的数据（PostgreSQL 和 Redis 数据保留在 `pg_data` 和 `redis_data` volume 中）。

### 3. 从源码构建并启动

```bash
docker-compose up -d --build
```

Docker 会根据本地 `Dockerfile` 从源码构建镜像，然后启动所有服务。

### 4. 验证服务

等待几秒后，验证服务是否正常运行：

```bash
# 检查容器状态
docker ps | grep -E "new-api|postgres|redis"

# 检查 API 状态
curl http://localhost:3001/api/status
```

如果返回 JSON 响应且包含 `"success":true`，则表示服务启动成功。

### 5. 恢复 docker-compose.yml（可选）

如需继续使用远程镜像版本进行日常部署，可以将 `docker-compose.yml` 恢复：

```yaml
# 改回远程镜像
  new-api:
    image: calciumion/new-api:latest
```

## 重要配置说明

### 端口保持不变

当前配置中端口映射为 `3001:3000`，外部访问使用 **3001** 端口。

### 数据持久化

- PostgreSQL 数据存储在 `pg_data` volume 中
- Redis 数据存储在默认的 Redis volume 中
- 应用数据挂载在 `./data` 和 `./logs` 目录

### 数据库连接

容器间通过服务名通信：
- New-API 连接 `postgres:5432`
- New-API 连接 `redis`

## 故障排查

### 容器启动失败

```bash
# 查看容器日志
docker-compose logs new-api

# 交互式查看日志
docker-compose logs -f new-api
```

### 数据库连接问题

确保 PostgreSQL 和 Redis 容器先启动，再启动 New-API。`depends_on` 配置已确保启动顺序。

### 重新构建无效

如果修改源码后构建使用了缓存，尝试强制重新构建：

```bash
docker-compose build --no-cache new-api
docker-compose up -d
```

## 自动化脚本

如需频繁更新，可以使用以下脚本：

```bash
#!/bin/bash
set -e

# 修改为本地构建
sed -i 's/image: calciumion\/new-api:latest/build: ./' docker-compose.yml

# 停止现有容器
docker-compose down

# 重新构建并启动
docker-compose up -d --build

# 恢复为镜像模式（可选）
sed -i 's/build: ./# image: calciumion\/new-api:latest/' docker-compose.yml
```

## 注意事项

1. **数据安全**：执行 `docker-compose down` 时数据 volume 不会被删除
2. **构建时间**：首次从源码构建可能需要 5-10 分钟（取决于网络）
3. **Go 版本**：Dockerfile 中使用 `golang:alpine` 镜像，具有较新的 Go 版本
4. **前端构建**：使用 `oven/bun` 构建 React 前端，确保 `web/` 目录完整