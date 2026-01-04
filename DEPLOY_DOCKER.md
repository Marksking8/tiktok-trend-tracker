---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 3045022100a8e8ec23ed0e1ef84e7735c484a296458355886d37f10dd46e152137740fc248022044d7191ba79960b251578783000afabfed697b9b5a13415700fdf2eec15abdcb
    ReservedCode2: 3045022006f72119eb99e702fae142aade895c68da5e62192282ae174443af446d82295d022100ab9f3fd96b741e9f1dfcac5911c71a62bd0ef5851aac6d27df8cf090897c4c1f
---

# TikTok 全球热点追踪系统 - Docker 云服务器部署

## 方案介绍

本方案适用于拥有云服务器（阿里云、腾讯云、华为云、AWS EC2、Google Cloud 等）的用户。通过 Docker 容器化部署，可以快速搭建生产环境。

### 服务器要求
- 操作系统：Ubuntu 20.04/22.04 或 CentOS 7/8
- CPU：最低 1 核（推荐 2 核以上）
- 内存：最低 2GB（推荐 4GB 以上）
- 磁盘：最低 20GB（推荐 50GB 以上）
- 带宽：最低 1Mbps（推荐 5Mbps 以上）

### 所需软件
- Docker Engine 20.10+
- Docker Compose 2.0+
- Git

## 第一步：环境准备

### 1.1 安装 Docker

**Ubuntu 系统：**
```bash
# 更新软件包
sudo apt update

# 安装必要工具
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# 添加 Docker 官方 GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker 仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动 Docker
sudo systemctl start docker
sudo systemctl enable docker

# 添加当前用户到 docker 组（免 sudo 执行 docker 命令）
sudo usermod -aG docker $USER
```

**CentOS 系统：**
```bash
# 安装必要工具
sudo yum install -y yum-utils

# 添加 Docker 仓库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装 Docker
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 启动 Docker
sudo systemctl start docker
sudo systemctl enable docker

# 添加当前用户到 docker 组
sudo usermod -aG docker $USER
```

### 1.2 验证 Docker 安装
```bash
# 检查 Docker 版本
docker --version
docker-compose --version

# 运行测试容器
docker run hello-world
```

### 1.3 安装 Git（如果未安装）
```bash
# Ubuntu
sudo apt install -y git

# CentOS
sudo yum install -y git
```

## 第二步：获取项目代码

### 2.1 克隆仓库
```bash
# 创建工作目录
mkdir -p ~/apps
cd ~/apps

# 克隆项目（替换为您的仓库地址）
git clone https://github.com/your-username/tiktok-trend-tracker.git

# 进入项目目录
cd tiktok-trend-tracker
```

### 2.2 配置环境变量
```bash
# 复制环境变量模板
cp .env.example .env

# 编辑环境变量
nano .env
```

编辑 `.env` 文件：
```env
# 数据库配置
MONGODB_ROOT_USERNAME=admin
MONGODB_ROOT_PASSWORD=your_strong_password_here
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_strong_password_here
POSTGRES_DB=tiktok_analytics
MONGODB_DATABASE=tiktok_analytics

# Redis 配置（可选）
REDIS_PASSWORD=your_redis_password_here

# 应用配置
API_SECRET_KEY=your-very-long-secret-key-at-least-32-characters
DEBUG=false
LOG_LEVEL=info

# 时区配置
TZ=Asia/Shanghai
```

## 第三步：启动服务

### 3.1 启动所有服务
```bash
# 构建并启动所有服务
docker-compose up -d --build

# 查看启动状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 3.2 服务启动顺序
Docker Compose 会按依赖顺序启动服务：
1. **postgresql** - PostgreSQL 数据库
2. **mongodb** - MongoDB 数据库
3. **redis** - Redis 缓存（可选）
4. **app** - FastAPI 应用
5. **nginx** - Web 服务器

### 3.3 验证服务状态
```bash
# 检查容器状态
docker-compose ps

# 预期输出：
# NAME                              STATUS
# tiktok-trend-tracker-app-1        Up
# tiktok-trend-tracker-mongodb-1    Up
# tiktok-trend-tracker-nginx-1      Up
# tiktok-trend-tracker-postgres-1   Up
# tiktok-trend-tracker-redis-1      Up
```

## 第四步：配置防火墙

### 4.1 Ubuntu UFW 防火墙
```bash
# 检查防火墙状态
sudo ufw status

# 开放必要端口
sudo ufw allow 22/tcp       # SSH
sudo ufw allow 80/tcp       # HTTP
sudo ufw allow 443/tcp      # HTTPS
sudo ufw allow 8000/tcp     # API
sudo ufw allow 8080/tcp     # Web

# 启用防火墙
sudo ufw enable
```

### 4.2 阿里云安全组
1. 登录阿里云控制台
2. 进入 ECS 管理页面
3. 找到对应实例，点击 "安全组"
4. 添加入方向规则：
   - 端口：80, 443, 8000, 8080
   - 协议：TCP
   - 授权对象：0.0.0.0/0

### 4.3 腾讯云安全组
1. 登录腾讯云控制台
2. 进入 CVM 管理页面
3. 找到对应实例，点击 "安全组"
4. 添加入站规则：
   - 协议：TCP
   - 端口：80, 443, 8000, 8080
   - 来源：0.0.0.0/0

## 第五步：访问应用

服务启动后，通过以下地址访问：

| 服务 | 地址 | 说明 |
|------|------|------|
| Web 界面 | http://your-server-ip:8080 | 前端可视化页面 |
| API 服务 | http://your-server-ip:8000 | 后端 API 接口 |
| API 文档 | http://your-server-ip:8000/docs | Swagger 文档 |
| 健康检查 | http://your-server-ip:8000/api/health | 服务状态 |

## 第六步：配置域名（可选）

### 6.1 准备域名
1. 购买域名（阿里云、腾讯云、GoDaddy 等）
2. 完成域名解析

### 6.2 配置 Nginx 域名
编辑 `nginx.conf`：
```nginx
server {
    listen 80;
    server_name api.yourdomain.com;

    location / {
        proxy_pass http://app:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

server {
    listen 80;
    server_name www.yourdomain.com;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

### 6.3 配置 SSL 证书（推荐）
使用 Let's Encrypt 免费证书：
```bash
# 安装 Certbot
sudo apt install -y certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# 自动续期测试
sudo certbot renew --dry-run
```

## 日常运维

### 查看日志
```bash
# 实时查看所有日志
docker-compose logs -f

# 查看特定服务日志
docker-compose logs -f app

# 查看最近 100 行日志
docker-compose logs --tail 100
```

### 重启服务
```bash
# 重启所有服务
docker-compose restart

# 重启特定服务
docker-compose restart app

# 重启并重新构建
docker-compose up -d --build
```

### 停止服务
```bash
# 停止所有服务（保留数据）
docker-compose stop

# 停止并删除容器
docker-compose down

# 停止并删除容器、网络、卷
docker-compose down -v
```

### 更新应用
```bash
# 拉取最新代码
git pull origin main

# 重新构建并启动
docker-compose up -d --build

# 清理旧镜像
docker image prune -a
```

### 数据备份
```bash
# 备份 PostgreSQL
docker exec tiktok-trend-tracker-postgres-1 pg_dump -U postgres tiktok_analytics > backup.sql

# 备份 MongoDB
docker exec tiktok-trend-tracker-mongodb-1 mongodump --out=/backup
docker cp tiktok-trend-tracker-mongodb-1:/backup ./mongo-backup
```

## 性能优化

### 1. 调整 Docker 内存限制
在 `docker-compose.yml` 中添加：
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

### 2. 配置 Nginx 缓存
```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
    expires 1d;
    add_header Cache-Control "public, immutable";
}
```

### 3. 优化数据库连接
在应用配置中设置连接池大小：
```python
# src/database.py 或配置文件中
SQLALCHEMY_ENGINE_OPTIONS = {
    "pool_size": 10,
    "pool_recycle": 3600,
    "pool_pre_ping": True
}
```

## 故障排查

### 问题 1：端口被占用
```bash
# 查看占用端口的进程
sudo lsof -i :8000

# 修改端口或停止占用进程
```

### 问题 2：数据库连接失败
```bash
# 检查数据库容器状态
docker-compose ps postgres mongodb

# 查看数据库日志
docker-compose logs postgres
docker-compose logs mongodb

# 检查连接字符串是否正确
cat .env | grep DATABASE
```

### 问题 3：内存不足
```bash
# 查看 Docker 内存使用
docker stats

# 清理未使用的资源
docker system prune -a
```

### 问题 4：磁盘空间不足
```bash
# 查看磁盘使用情况
df -h

# 清理 Docker 资源
docker system df
docker system prune --volumes
```

## 监控建议

### 基础监控
- 使用 Docker Stats 监控容器资源使用
- 配置日志轮转防止日志文件过大

### 建议监控工具
- **Portainer**：Docker 可视化管理界面
- **Grafana + Prometheus**：完整监控解决方案
- **UptimeRobot**：网站可用性监控

## 成本估算

| 项目 | 费用（阿里云为例） |
|------|-------------------|
| 2核2G云服务器 | 约 80-100 元/月 |
| 数据盘 50GB | 约 10 元/月 |
| 带宽 5Mbps | 约 100 元/月 |
| 域名 | 约 50 元/年 |
| **总计** | **约 200 元/月** |

## 总结

通过本指南，您可以在云服务器上成功部署 TikTok 全球热点追踪系统。Docker 容器化部署具有以下优势：

- **快速部署**：几分钟内完成环境搭建
- **环境一致**：开发环境和生产环境保持一致
- **易于迁移**：容器可轻松迁移到其他服务器
- **便于扩展**：可水平扩展应对更大负载

如有问题，请参考故障排查章节或提交 Issue 寻求帮助。
