# Docker 部署文档 - 端口 3005（调试环境）

## 概述

本文档描述如何在端口 3005 部署一个独立的调试环境，不影响端口 3001 的生产环境。

## 配置文件

配置文件：`docker-compose-local-3005.yml`

**特点：**
- 使用 **SQLite** 数据库（不影响生产的 PostgreSQL）
- 使用独立的 **redis-local-3005** 容器（不影响生产的 Redis）
- 端口 **3005**（不影响生产的 3001 端口）
- 数据存储在 `./data-local-3005/` 目录

## 构建和启动命令

```bash
# 进入项目目录
cd /home/hxp/code/new-api

# 构建并启动（首次运行或代码修改后）
docker-compose -f docker-compose-local-3005.yml up -d --build

# 查看日志
docker-compose -f docker-compose-local-3005.yml logs -f

# 停止服务
docker-compose -f docker-compose-local-3005.yml down

# 重启服务
docker-compose -f docker-compose-local-3005.yml restart
```

## 访问地址

| 环境 | 地址 |
|------|------|
| 调试环境 | http://localhost:3005 |
| 生产环境 | http://localhost:3001 |

## 容器列表

| 容器名称 | 用途 | 端口 |
|---------|------|------|
| new-api | 生产环境 | 3001 |
| postgres | 生产数据库 | 5432 (内部) |
| redis | 生产缓存 | 6379 (内部) |
| new-api-local-3005 | 调试环境 | 3005 |
| redis-local-3005 | 调试缓存 | 6379 (内部) |

## 数据目录

| 环境 | 数据目录 |
|------|---------|
| 生产数据 | `./data/` |
| 调试数据 | `./data-local-3005/` |

## 注意事项

1. 两个环境完全独立，互不影响
2. 调试环境使用 SQLite，生产环境使用 PostgreSQL
3. 首次构建需要较长时间（约 5-10 分钟）
4. 修改代码后需要重新构建镜像

## 新功能说明

本次修改添加了用户渠道管理功能：

### 功能特性

1. **管理员权限**：管理员（role >= 10）可以管理所有渠道
2. **用户渠道管理权限**：管理员可以通过用户编辑界面开启普通用户的 `can_manage_channels` 权限
3. **渠道所有权**：
   - `UserId = nil` 表示系统共享渠道，所有用户可用
   - `UserId = 具体用户ID` 表示该渠道只属于指定用户
4. **渠道使用**：用户只能使用自己的渠道和共享渠道
5. **渠道管理**：用户只能编辑/删除自己的渠道

### 数据库变更

- `channels` 表新增 `user_id` 字段
- `users` 表新增 `can_manage_channels` 字段
- `abilities` 表新增 `user_id` 字段
