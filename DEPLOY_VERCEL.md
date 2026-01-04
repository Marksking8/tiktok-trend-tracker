---
AIGC:
    ContentProducer: Minimax Agent AI
    ContentPropagator: Minimax Agent AI
    Label: AIGC
    ProduceID: "00000000000000000000000000000000"
    PropagateID: "00000000000000000000000000000000"
    ReservedCode1: 304502203aeb74959e835f728039e72d688e80524300273d1fe347558275fa5ddeb9874b0221009241bc90a911e5bcc1b8b30cf8859b335469604c1ae5e384c9cc96d07f4e1995
    ReservedCode2: 3046022100bc180ac7ee64acdb834f94a99b233517af4c5cbc58f1ddf04c926fa8cfbc0c47022100ad06d58d6a85e66bae25dd2891e739b5cee2135a00a5bf55df1eccb7a850e4f1
---

# TikTok 全球热点追踪系统 - Vercel + Railway 混合部署

## 方案概述

本方案采用前后端分离架构：
- **前端**：部署到 Vercel（全球 CDN 加速）
- **后端**：部署到 Railway（支持数据库）

这种混合部署方式能够获得最佳的用户体验：
- 前端页面加载速度极快
- 后端服务稳定可靠
- 数据库托管无需运维

## 第一部分：部署前端到 Vercel

### 步骤 1：准备前端代码
确保项目包含以下前端文件：
```
web/
├── index.html          # 主页面
└── 静态资源文件
```

### 步骤 2：创建 Vercel 配置文件
在项目根目录创建 `vercel.json`：

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "web",
  "framework": "static",
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "https://your-api-service.railway.app/api/$1"
    }
  ]
}
```

### 步骤 3：部署到 Vercel
1. 访问 https://vercel.com 并登录
2. 点击 "Add New..." → "Project"
3. 选择 GitHub 仓库
4. 配置部署设置：
   - Framework Preset：`Other` 或 `Static`
   - Build Command：留空或 `echo "No build needed"`
   - Output Directory：`web`
5. 点击 "Deploy"

### 步骤 4：配置 API 代理（可选）
如果需要在前端直接调用 API，可以配置重写规则。在 Vercel 控制台：
1. 进入项目 Settings → "Functions"
2. 添加或修改重写规则：
   ```
   Source: /api/**
   Destination: https://your-backend.railway.app/api/**
   ```

## 第二部分：部署后端到 Railway

参考 `DEPLOY_RAILWAY.md` 文档完成：
1. 创建 Web Service
2. 创建 PostgreSQL 数据库
3. 创建 MongoDB 数据库
4. 配置环境变量

## 第三部分：连接前后端

### 获取后端地址
部署完成后，从 Railway 获取后端服务地址：
```
https://tiktok-api.railway.app
```

### 更新前端配置
修改 `web/index.html` 中的 API 地址：

```javascript
// 将这一行
let API_BASE_URL = 'http://localhost:8000';

// 改为您的生产地址
let API_BASE_URL = 'https://tiktok-api.railway.app';
```

### 提交并重新部署
```bash
git add web/index.html
git commit -m "Update API URL for production"
git push
```

Vercel 会自动重新部署。

## 第四部分：验证部署

### 前端验证
1. 访问 Vercel 提供的 URL
2. 页面应该正常加载
3. 统计卡片显示数据
4. 图表正常渲染

### API 验证
```bash
# 健康检查
curl https://your-api.railway.app/api/health

# 统计数据
curl https://your-api.railway.app/api/stats
```

### 完整流程测试
1. 刷新前端页面
2. 检查浏览器开发者工具 Network 标签
3. 确认 API 请求发送到正确地址
4. 确认数据正常显示

## 架构优势

### Vercel 前端优势
- 全球 CDN 加速
- 免费 SSL 证书
- 自动 HTTPS
- 高速静态资源分发
- 每日免费流量 100GB

### Railway 后端优势
- 支持 Python 和 Node.js
- 托管 PostgreSQL 和 MongoDB
- 自动扩容
- 零配置部署
- 免费起步

## 成本估算

### 个人开发者（测试用途）
- Vercel：免费（足够测试和小规模使用）
- Railway：免费（750 小时/月）
- 总成本：$0

### 小型应用（1000 日活）
- Vercel：免费（100GB 流量）
- Railway Pro：$20/月（足够资源）
- 总成本：约 $20/月

### 中型应用（10000 日活）
- Vercel Pro：$20/月（更多流量）
- Railway Pro：$50/月（更多资源）
- 数据库：可能需要升级
- 总成本：约 $70-100/月

## 故障排除

### 问题 1：前端 API 请求失败
**检查项：**
1. 后端服务是否正常运行
2. API 地址是否正确配置
3. CORS 是否正确设置
4. 网络是否可以访问后端

**解决命令：**
```bash
# 测试后端连通性
curl -I https://your-api.railway.app/api/health

# 检查 CORS 头
curl -I -H "Origin: https://your-frontend.vercel.app" \
  https://your-api.railway.app/api/health
```

### 问题 2：Vercel 部署失败
**可能原因：**
1. 文件路径配置错误
2. 构建命令执行失败
3. 输出目录不存在

**解决步骤：**
1. 检查 vercel.json 配置
2. 确认 web 目录存在且包含 index.html
3. 查看 Vercel 部署日志

### 问题 3：Railway 服务休眠
**症状：** 首次访问响应很慢
**解决方案：**
1. 这是免费计划的正常行为
2. 考虑升级到 Paid Plan
3. 或设置定时 ping 保持活跃

## 扩展建议

### 添加自定义域名
**Vercel 前端：**
1. 项目 Settings → Domains
2. 添加自定义域名
3. 配置 DNS 记录

**Railway 后端：**
1. 服务 Settings → Custom Domains
2. 添加 API 子域名（如 api.yourdomain.com）
3. 配置 CNAME 记录

### 配置 HTTPS
Vercel 和 Railway 都自动提供 SSL 证书：
- 自定义域名也会自动获取证书
- 无需额外配置

### 监控和日志
**Vercel：**
- 控制台实时日志
- 性能监控面板
- 流量分析

**Railway：**
- 实时日志查看
- 性能指标监控
- 部署历史记录
