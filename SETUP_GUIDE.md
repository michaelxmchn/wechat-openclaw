# OpenClaw + 企业微信 + Vercel 部署指南

## 方案架构

```
┌─────────────┐      ┌─────────┐     自有域名      ┌─────────┐
│  企业微信   │ ───► │ Vercel  │ ◄─────────────► │  你的   │
│   (用户)    │      │ API代理 │    your.com    │  域名   │
└─────────────┘      └────┬────┘                 └─────────┘
                          │
                    Webhook URL
                          │
                   ┌──────▼──────┐
                   │   你的电脑   │
                   │  OpenClaw    │
                   └─────────────┘
```

## 准备工作

### 1. 域名要求
- 已在国内备案的域名
- 已解析到你的服务器/VPS

### 2. 服务器
- 一台长期开机的电脑/VPS
- 安装 Node.js 18+

### 3. 企业微信
- 企业微信管理员账号
- 创建一个自建应用

---

## 第一部分：服务器配置

### 1.1 安装 Node.js

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# 验证
node -v
npm -v
```

### 1.2 安装 OpenClaw

```bash
# 安装
npm install -g clawdbot@latest

# 初始化
clawdbot onboard --install-daemon
```

### 1.3 安装企业微信插件

```bash
# 安装插件
openclaw plugins install @sunnoy/wecom

# 或
openclaw plugins install @yanhaidao/wecom
```

### 1.4 配置 OpenClaw

编辑 `~/.openclaw/openclaw.json`：

```json
{
  "plugins": {
    "entries": {
      "wecom": {
        "enabled": true
      }
    }
  },
  "channels": {
    "wecom": {
      "enabled": true,
      "token": "你的Token",
      "encodingAesKey": "你的EncodingAESKey(43位)",
      "adminUsers": ["你的用户ID"],
      "commands": {
        "enabled": true,
        "allowlist": ["/new", "/status", "/help", "/compact"]
      }
    }
  }
}
```

### 1.5 启动 OpenClaw

```bash
# 启动
openclaw gateway start

# 检查状态
openclaw gateway status
```

---

## 第二部分：Vercel 配置

### 2.1 创建 Vercel 项目

1. 登录 [Vercel](https://vercel.com)
2. 创建新项目，导入 `openclaw/openclaw`
3. 配置环境变量

### 2.2 配置环境变量

```env
# 必填
OPENCLAW_SECRET=你的密钥

# 可选
MODEL_PROVIDER=deepseek
MODEL_NAME=deepseek-chat
```

### 2.3 绑定自有域名

1. 在 Vercel 项目中进入 Settings → Domains
2. 添加你的域名
3. 按照提示配置 DNS 记录

---

## 第三部分：企业微信配置

### 3.1 创建应用

1. 登录 [企业微信管理后台](https://work.weixin.qq.com/)
2. 进入「应用管理」→「创建应用」
3. 选择「自建应用」
4. 填写应用信息

### 3.2 配置可信域名

1. 在应用详情页找到「可信域名」
2. 添加你的域名（例如：`yourdomain.com`）
3. 下载验证文件上传到域名根目录

### 3.3 配置接收消息

1. 在应用详情页找到「API接收消息」
2. 配置以下内容：

| 配置项 | 值 |
|--------|-----|
| URL | `https://yourdomain.com/webhooks/wecom` |
| Token | `与OpenClaw配置一致` |
| EncodingAESKey | `与OpenClaw配置一致` |

### 3.4 获取应用凭证

在应用详情页获取：
- AgentId（应用ID）
- Secret（应用密钥）

---

## 第四部分：连接配置

### 4.1 配置 Webhook 中转（可选）

如果 Vercel 无法直接访问本地 OpenClaw，需要中转：

```javascript
// Vercel 函数 /api/wecom.js
export default async function handler(req, res) {
  const response = await fetch('http://你的服务器IP:3000/webhooks/wecom', {
    method: 'POST',
    body: JSON.stringify(req.body),
    headers: {
      'Content-Type': 'application/json'
    }
  });
  
  const data = await response.json();
  res.status(200).json(data);
}
```

### 4.2 更新企业微信回调地址

企业微信回调地址改为：
```
https://yourdomain.com/api/wecom
```

---

## 第五部分：测试

### 5.1 测试企业微信连接

```bash
# 在企业微信中向应用发送消息
# 应该能收到自动回复
```

### 5.2 测试 OpenClaw 功能

```bash
# 检查日志
openclaw gateway logs

# 测试发送消息
```

---

## 常见问题

### Q1: 企业微信提示 URL 无效
- 确认可信域名已正确配置
- 确认验证文件可访问

### Q2: 消息收不到
- 检查服务器防火墙
- 检查 Vercel 日志
- 确认 Token 一致

### Q3: 如何重启 OpenClaw

```bash
openclaw gateway restart
```

---

## 文件结构

```
wechat-openclaw/
├── SETUP_GUIDE.md      # 本指南
├── docker-compose.yml   # Docker部署（可选）
└── .env.example         # 环境变量示例
```

---

## 下一步

完成以上配置后，你可以通过企业微信与 OpenClaw 沟通了！

---

*文档更新时间: 2026-02-19*
