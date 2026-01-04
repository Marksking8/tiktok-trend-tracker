---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 3046022100b9116f0587f97bce5587d6b08f53d137f063ad14b82d960c46620a60d6dfb3ef022100bace32bd4161a95ad1ab3c1a53c484dc60792666ee2e55c4846830af6338de85
    ReservedCode2: 3046022100b4e97436e72150dae127a25917deb01f597a3d9c4a9aa589951490a973d728f00221009ed7beb997b2071001f04f4d9ae3666b9d2d223d58e8d9d529a5d2c329a11d1b
---

# TikTok 全球热点追踪系统 - Render 部署配置

## 快速部署到 Render

### 步骤 1：准备 GitHub 仓库
1. 将项目推送到 GitHub 仓库
2. 确保文件结构完整：
   - `requirements.txt` - Python 依赖
   - `requirements-api.txt` - API 额外依赖
   - `src/api.py` - FastAPI 应用入口
   - `docker-compose.yml` - Docker 配置

### 步骤 2：创建 Render 账号并连接 GitHub
1. 访问 https://render.com
2. 点击 "Sign Up" 注册账号
3. 选择 "Continue with GitHub" 授权访问仓库
4. 选择您的 tiktok-trend-tracker 仓库

### 步骤 3：创建 Web Service
1. 在 Render 控制台，点击 "New" → "Web Service"
2. 选择 GitHub 仓库
3. 配置构建和运行设置：

**基础配置：**
- Name：`tiktok-api`（或其他名称）
- Environment：`Python 3`
- Region：选择最近的区域（如 Singapore 或 Frankfurt）

**构建配置：**
- Build Command：`pip install -r requirements.txt -r requirements-api.txt`
- Start Command：`uvicorn src.api:app --host 0.0.0.0 --port $PORT`

**高级配置：**
- Instance Type：Free（测试用）
- Health Check Path：`/api/health`

4. 点击 "Create Web Service"

### 步骤 4：创建数据库服务
Render 平台提供托管数据库：

**PostgreSQL 数据库：**
1. 点击 "New" → "PostgreSQL"
2. 设置名称：`tiktok-db`
3. 选择 Free 计划
4. 点击 "Create Database"
5. 创建完成后，复制 Internal Database URL

**MongoDB 数据库：**
1. 点击 "New" → "MongoDB"
2. 设置名称：`tiktok-mongo`
3. 选择 Free 计划
4. 点击 "Create Database"
5. 复制 Connection URI

### 步骤 5：配置环境变量
1. 在 Web Service 页面，点击 "Environment" 标签
2. 添加以下环境变量：

```
# 数据库连接
DATABASE_URL=postgresql://user:password@hostname:5432/dbname
MONGODB_URI=mongodb+srv://user:password@hostname/dbname

# 应用配置
API_SECRET_KEY=your-secure-secret-key-here
DEBUG=false
LOG_LEVEL=info
```

3. 点击 "Save Changes"
4. 服务会自动重启

### 步骤 6：获取访问地址
部署完成后，在控制台查看：
- Service URL：API 服务地址（如 `https://tiktok-api.onrender.com`）
- API 文档：`https://tiktok-api.onrender.com/docs`
- 健康检查：`https://tiktok-api.onrender.com/api/health`

## Render 免费计划说明

### 免费额度
- 每月 750 小时运行时间
- 500MB 数据库存储
- 单个 Web Service
- 单个数据库实例

### 注意事项
1. **休眠机制**：15 分钟无活动后自动休眠
2. **唤醒时间**：休眠后首次请求需要 30-60 秒
3. **构建时间**：每次部署有 30 分钟限制
4. **带宽限制**：每月 100GB 免费带宽

### 成本说明
- 超出免费额度后按使用付费
- 查看 Render 定价页面了解详情

## 部署验证

部署完成后，访问以下地址验证：

```bash
# 健康检查
curl https://your-service-url.onrender.com/api/health

# 统计数据
curl https://your-service-url.onrender.com/api/stats

# 关键词列表
curl https://your-service-url.onrender.com/api/keywords?top_n=10
```

## 常见问题解决

### 问题 1：构建失败
**症状**：部署日志显示依赖安装错误
**解决**：
- 检查 requirements.txt 语法
- 确保包版本兼容
- 查看具体错误日志

### 问题 2：数据库连接失败
**症状**：应用日志显示连接超时
**解决**：
- 确认环境变量正确设置
- 检查数据库服务状态
- 验证连接字符串格式

### 问题 3：服务频繁休眠
**症状**：免费实例经常需要唤醒
**解决**：
- 考虑升级到 Paid Plan
- 使用 Keep-alive ping 服务
- 评估是否需要更高配置

### 问题 4：CORS 错误
**症状**：前端无法调用 API
**解决**：
- 检查 API 是否设置 CORS 头
- 确认前端请求地址正确
- 验证浏览器控制台错误信息

## 监控和维护

### 查看日志
1. 在 Render 控制台选择服务
2. 点击 "Logs" 标签查看实时日志
3. 可以搜索特定关键词

### 重启服务
1. 在服务页面点击 "Settings"
2. 滚动到 "Actions" 部分
3. 点击 "Restart"

### 部署更新
1. 推送代码到 GitHub
2. Render 自动触发部署
3. 在 "Deployments" 标签查看进度

## 高级配置

### 自定义域名
1. 在服务 Settings 中找到 "Custom Domains"
2. 添加您的域名
3. 配置 DNS 记录指向 Render
4. SSL 证书会自动配置

### 环境变量管理
- 使用 .env 文件进行本地开发
- 在 Render 控制台管理生产环境变量
- 敏感信息不要提交到 GitHub

### 性能优化
- 使用 Gunicorn 作为生产服务器：`gunicorn -w 4 -k uvicorn.workers.UvicornWorker src.api:app`
- 启用响应压缩
- 配置缓存策略
