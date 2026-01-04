---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 30450221009e944d1c9a5597519139c4c56727d48c066a38f5ba7bb72c1f8ced1044c495d20220399339f5afe242fae75495cf52f156074975c891fb71984b3793bc8c1c33e382
    ReservedCode2: 3045022069378b2989d81a5ca7575babb9e974e4c5c2afe820ebbecc1e745ce780214678022100b0a20042ac2e4fbb48862c2adcfdf4f0f9011f98d07b36ebf41bb001f768f964
---

# TikTok 全球热点追踪系统 - Railway 部署配置

## 快速部署到 Railway

### 步骤 1：准备 GitHub 仓库
1. 将项目推送到 GitHub 仓库
2. 确保包含以下文件：
   - `requirements.txt`
   - `requirements-api.txt`
   - `docker-compose.yml`
   - `src/api.py`

### 步骤 2：部署到 Railway
1. 访问 https://railway.app 并登录
2. 点击 "New Project"
3. 选择 "Deploy from GitHub repo"
4. 选择您的 tiktok-trend-tracker 仓库
5. Railway 会自动检测服务类型

### 步骤 3：配置服务
Railway 会自动创建 Web 服务，但需要添加数据库：

1. 在 Railway 控制台中，点击 "New" → "Database" → "PostgreSQL"
2. 创建 PostgreSQL 数据库
3. 创建 "New" → "Database" → "MongoDB"
4. 创建 MongoDB 数据库

### 步骤 4：配置环境变量
在服务设置中添加以下环境变量：

```
DATABASE_URL=postgresql://...
MONGODB_URI=mongodb://...
API_SECRET_KEY=your-secret-key
```

### 步骤 5：部署完成
- Railway 会自动构建和部署
- 部署完成后会获得一个公共 URL，如：`https://tiktok-trend-tracker.up.railway.app`
- API 文档地址：`https://tiktok-trend-tracker.up.railway.app/docs`
- 健康检查：`https://tiktok-trend-tracker.up.railway.app/api/health`

## Railway 部署特点

✅ 自动构建和部署
✅ 免费额度足够测试使用
✅ 支持 PostgreSQL 和 MongoDB
✅ 自动生成 SSL 证书
✅ 支持自定义域名

## 注意事项

1. 首次部署可能需要 5-10 分钟
2. 免费账户有每月 500 小时的运行时间限制
3. 服务在 15 分钟无活动后会自动休眠（唤醒可能需要 30 秒）
4. 唤醒后的第一次请求可能较慢

## 故障排除

### 构建失败
- 检查 requirements.txt 路径是否正确
- 确保所有依赖包版本兼容
- 查看构建日志定位具体错误

### 数据库连接失败
- 检查环境变量是否正确设置
- 确认数据库服务已启动
- 查看应用日志获取详细错误信息

### 服务无法启动
- 检查端口配置是否正确
- 确认没有端口冲突
- 查看应用日志获取启动错误

## 监控和维护

- 在 Railway 控制台查看服务状态
- 查看实时日志进行调试
- 可以手动重启服务
- 可以在控制台执行数据库迁移
