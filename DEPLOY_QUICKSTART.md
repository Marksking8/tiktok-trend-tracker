---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 3046022100d1778160ff1153df2dc38e0b5afb359c4292c9ec881da34a6fb320e28da81b79022100e05d0b58c33b51b1df93957bb1aa0d44767843a063bb7f74de73047ccb63dcea
    ReservedCode2: 304402204c0354447728f0bd89c6a565c12bf292b37e772b1a789ea294ed4536d572dcb9022076e35a35aa0fd441a4168944798423c95e0339ec180e5db64919e295926cd08a
---

# TikTok 全球热点追踪系统 - 快速部署指南

## 概述

本文档提供四种快速部署 TikTok 全球热点追踪系统到公网的方案。您可以根据自身情况选择最适合的方案。所有方案都能让您的系统获得公网访问地址，便于测试和展示。

## 方案对比

| 方案 | 难度 | 费用 | 适合场景 |
|------|------|------|---------|
| Railway | ⭐ | 免费 | 个人测试、快速上手 |
| Render | ⭐⭐ | 免费 | 需要托管数据库 |
| Vercel + Railway | ⭐⭐ | 免费 | 追求最快的前端速度 |
| Docker 云服务器 | ⭐⭐⭐ | 约 200 元/月 | 生产环境、长期运行 |

## 方案一：Railway 部署（推荐新手）

### 部署步骤

**第一步：准备 GitHub 仓库**
1. 将 tiktok-trend-tracker 项目推送到 GitHub
2. 确保项目包含必要文件：`requirements.txt`、`requirements-api.txt`、`src/api.py`、`docker-compose.yml`

**第二步：创建 Railway 账号**
1. 访问 https://railway.app
2. 点击 "Sign Up" 注册账号
3. 使用 GitHub 账号登录

**第三步：部署服务**
1. 在 Railway 控制台点击 "New Project"
2. 选择 "Deploy from GitHub repo"
3. 选择您的 tiktok-trend-tracker 仓库
4. Railway 会自动检测为 Python 应用

**第四步：添加数据库**
1. 点击 "New" → "Database" → "PostgreSQL"
2. 等待数据库创建完成
3. 再次点击 "New" → "Database" → "MongoDB"
4. 等待 MongoDB 创建完成

**第五步：配置环境变量**
1. 进入服务页面，点击 "Variables" 标签
2. 添加以下变量：
   ```
   DATABASE_URL=<从 PostgreSQL 获取>
   MONGODB_URI=<从 MongoDB 获取>
   API_SECRET_KEY=<设置一个复杂的密钥>
   ```

**第六步：获取访问地址**
- 部署完成后，Railway 会提供服务 URL
- 格式类似：`https://tiktok-trend-tracker.up.railway.app`
- API 文档：`https://tiktok-trend-tracker.up.railway.app/docs`
- 健康检查：`https://tiktok-trend-tracker.up.railway.app/api/health`

### 验证部署
```bash
# 测试健康检查
curl https://your-app.up.railway.app/api/health

# 预期输出：{"status":"healthy","timestamp":...}

# 测试统计数据
curl https://your-app.up.railway.app/api/stats
```

### 预计耗时
- 首次部署：5-10 分钟
- 后续更新：1-2 分钟

---

## 方案二：Render 部署

### 部署步骤

**第一步：创建 Web Service**
1. 访问 https://render.com
2. 点击 "New" → "Web Service"
3. 连接 GitHub 仓库
4. 设置：
   - Build Command：`pip install -r requirements.txt -r requirements-api.txt`
   - Start Command：`uvicorn src.api:app --host 0.0.0.0 --port $PORT`

**第二步：创建数据库**
1. 点击 "New" → "PostgreSQL"
2. 点击 "New" → "MongoDB"

**第三步：配置环境变量**
在服务 Environment 页面添加：
```
DATABASE_URL=<PostgreSQL 连接字符串>
MONGODB_URI=<MongoDB 连接字符串>
API_SECRET_KEY=<您的密钥>
```

**第四步：获取访问地址**
- 服务 URL 格式：`https://tiktok-api.onrender.com`

---

## 方案三：Vercel + Railway 混合部署

此方案将前端部署到 Vercel（极速加载），后端部署到 Railway（完整数据库支持）。

### 部署步骤

**第一步：部署后端到 Railway**
按照方案一的步骤部署后端服务

**第二步：配置 Vercel**
1. 访问 https://vercel.com
2. 导入 GitHub 仓库
3. 设置 Output Directory 为 `web`
4. 点击 Deploy

**第三步：更新前端配置**
修改 `web/index.html` 中的 API 地址：
```javascript
let API_BASE_URL = 'https://your-railway-app.up.railway.app';
```

**第四步：提交并重新部署**
```bash
git add web/index.html
git commit -m "Update API URL"
git push
```

### 访问地址
- 前端：`https://your-vercel-app.vercel.app`
- 后端：`https://your-railway-app.up.railway.app`

---

## 方案四：Docker 云服务器部署

此方案适合有云服务器的用户，支持长期稳定运行。

### 部署步骤

**第一步：准备云服务器**
1. 购买云服务器（阿里云、腾讯云、AWS 等）
2. 安装 Ubuntu 20.04 或 22.04
3. 开放端口：80、443、8000、8080

**第二步：安装 Docker**
```bash
# 安装 Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# 验证安装
docker --version
```

**第三步：部署项目**
```bash
# 克隆项目
git clone https://github.com/your-username/tiktok-trend-tracker.git
cd tiktok-trend-tracker

# 配置环境变量
cp .env.example .env
nano .env

# 启动服务
docker-compose up -d --build
```

**第四步：配置域名和 SSL（可选）**
- 购买域名并解析到服务器 IP
- 使用 Let's Encrypt 配置免费 SSL 证书

### 访问地址
- Web 界面：`http://your-server-ip:8080`
- API 服务：`http://your-server-ip:8000`

---

## 部署验证清单

无论使用哪种方案，部署完成后请验证以下项目：

### 1. 服务健康检查
```bash
curl https://your-api-url/api/health
# 预期：{"status":"healthy","timestamp":...}
```

### 2. 统计数据接口
```bash
curl https://your-api-url/api/stats
# 预期：包含 total_videos、total_views、total_likes 等字段
```

### 3. 关键词列表
```bash
curl https://your-api-url/api/keywords?top_n=10
# 预期：返回关键词数组
```

### 4. 标签列表
```bash
curl https://your-api-url/api/hashtags?limit=10
# 预期：返回标签数组
```

### 5. 视频列表
```bash
curl https://your-api-url/api/videos?limit=10
# 预期：返回视频数组
```

### 6. Web 界面访问
1. 浏览器访问前端 URL
2. 页面正常加载
3. 统计卡片显示数据
4. 图表正常渲染
5. 刷新按钮可用

---

## 常见问题

### Q1：部署后页面显示 API 未连接？
**原因**：前端配置的 API 地址与实际后端地址不匹配
**解决**：修改 `web/index.html` 中的 `API_BASE_URL` 为实际后端地址

### Q2：首次访问很慢？
**原因**：免费实例在无访问时会休眠
**解决**：这是正常行为，唤醒后会自动恢复

### Q3：数据库连接失败？
**原因**：环境变量未正确配置
**解决**：检查 Railway/Render 控制台中的环境变量是否正确

### Q4：端口被占用？
**原因**：服务器上已有服务使用相同端口
**解决**：修改 `docker-compose.yml` 中的端口映射

### Q5：如何更新部署？
**方法**：推送代码到 GitHub，平台会自动重新部署

---

## 免费额度说明

| 平台 | 免费额度 | 备注 |
|------|---------|------|
| Railway | 500 小时/月 | 超出按量付费 |
| Render | 750 小时/月 | 单个服务 |
| Vercel | 100GB 带宽/月 | 足够小规模使用 |
| Docker | 取决于服务器 | 需自行购买服务器 |

---

## 技术支持

如果部署过程中遇到问题：

1. **查看日志**：在平台控制台查看应用日志
2. **检查文档**：阅读详细的部署文档
   - `DEPLOY_RAILWAY.md`
   - `DEPLOY_RENDER.md`
   - `DEPLOY_VERCEL.md`
   - `DEPLOY_DOCKER.md`
3. **提交 Issue**：在 GitHub 仓库提交问题

---

## 下一步

部署成功后，您可以：

1. **测试完整功能**：访问 Web 界面，验证所有功能
2. **分享给他人**：分享公网访问地址
3. **配置自定义域名**：提升专业形象
4. **扩展功能**：根据需求添加新功能

祝您部署顺利！
