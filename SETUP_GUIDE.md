# OpenClaw + 企业微信 + Vercel 部署指南

> 当前电脑配置完成后，可迁移到其他电脑运行

---

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
                   │   电脑A     │
                   │  OpenClaw   │ ← 当前电脑
                   └─────────────┘
                            │
                     迁移到电脑B
```

---

## 准备工作清单

| 序号 | 项目 | 状态 | 备注 |
|------|------|------|------|
| 1 | 备案域名 | ✅ 已准备 | |
| 2 | 企业微信管理员 | ✅ 已准备 | |
| 3 | Node.js 18+ | ⏳ 待确认 | 当前电脑检查 |
| 4 | OpenClaw | ⏳ 待确认 | 当前电脑检查 |

---

## 第一步：检查当前电脑环境

### 1.1 检查 Node.js

```bash
node -v
npm -v
```

**预期结果**：显示版本号，如 `v18.x.x`

### 1.2 检查 OpenClaw

```bash
openclaw gateway status
```

**预期结果**：显示运行状态

### 1.3 检查企业微信插件

```bash
openclaw plugins list
```

---

## 第二步：安装企业微信插件

### 2.1 安装插件

```bash
# 安装企业微信插件（二选一）
openclaw plugins install @sunnoy/wecom

# 或
openclaw plugins install @yanhaidao/wecom
```

### 2.2 配置插件

编辑配置文件 `~/.openclaw/openclaw.json`：

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
      "token": "请在企业微信后台获取",
      "encodingAesKey": "请在企业微信后台获取（43位）",
      "adminUsers": ["你的用户ID"]
    }
  }
}
```

### 2.3 重启 OpenClaw

```bash
openclaw gateway restart
```

---

## 第三步：配置企业微信应用

### 3.1 登录企业微信后台

访问：https://work.weixin.qq.com/

### 3.2 创建自建应用

1. 进入「应用管理」→「创建应用」
2. 选择「自建应用」
3. 填写应用名称（如：AI助手）
4. 上传应用图标

### 3.3 配置可信域名

1. 在应用详情页找到「可信域名」
2. 添加你的域名
3. 下载验证文件

### 3.4 配置 API 接收消息

| 配置项 | 值 | 说明 |
|--------|-----|------|
| URL | `https://你的域名/webhooks/wecom` | 需先配置Vercel |
| Token | 随机字符串 | 记录下来 |
| EncodingAESKey | 43位随机字符串 | 点击生成并记录 |

### 3.5 获取应用凭证

| 凭证 | 位置 | 用途 |
|------|------|------|
| AgentId | 应用详情页 | 应用ID |
| Secret | 应用详情页 | 应用密钥 |

---

## 第四步：配置 Vercel

### 4.1 创建 Vercel 项目

1. 登录 [Vercel](https://vercel.com)
2. 新建项目，导入 `openclaw/openclaw`

### 4.2 配置环境变量

| 变量名 | 值 |
|--------|-----|
| OPENCLAW_SECRET | 随机密钥 |

### 4.3 绑定自有域名

1. Settings → Domains
2. 添加你的域名
3. 按提示配置 DNS

---

## 第五步：连接测试

### 5.1 测试企业微信发送消息

在企业微信中向应用发送一条消息

### 5.2 检查 OpenClaw 日志

```bash
openclaw gateway logs
```

---

## 配置文件备份（重要）

配置完成后，备份以下文件：

```bash
# 备份 OpenClaw 配置
cp ~/.openclaw/openclaw.json ./backup/openclaw_wecom.json

# 导出插件
openclaw plugins export
```

---

## 迁移到其他电脑

当需要迁移到其他电脑时：

### 1. 导出配置

```bash
# 在当前电脑执行
cp ~/.openclaw/openclaw.json ~/wechat-openclaw-config/
```

### 2. 导入配置

```bash
# 在新电脑执行
# 1. 安装 Node.js
# 2. 安装 OpenClaw
# 3. 安装企业微信插件
# 4. 复制配置到 ~/.openclaw/openclaw.json
# 5. 重启 OpenClaw
```

### 3. 检查项

| 项目 | 新电脑检查 |
|------|-----------|
| Node.js 版本 | `node -v` |
| OpenClaw 状态 | `openclaw gateway status` |
| 企业微信连接 | 发送测试消息 |

---

## 常见问题

### Q1: 企业微信提示 URL 无效
- 确认域名已解析到 Vercel
- 确认可信域名已添加
- 验证文件可访问

### Q2: 消息收不到
- 检查 OpenClaw 日志
- 确认 Token 和 EncodingAESKey 一致

### Q3: 如何查看日志

```bash
openclaw gateway logs -f
```

---

## 快速命令参考

```bash
# 启动
openclaw gateway start

# 停止
openclaw gateway stop

# 重启
openclaw gateway restart

# 状态
openclaw gateway status

# 日志
openclaw gateway logs -f

# 插件列表
openclaw plugins list
```

---

*文档更新时间: 2026-02-19*

---

## 第六步：Vercel 配置（URL解决方案）

### 方案说明

由于企业微信需要可访问的URL，需要通过Vercel中转：

```
企业微信 → Vercel域名 → 你的服务器 → OpenClaw
```

### 6.1 Vercel 部署

1. 登录 Vercel
2. 新建项目，导入：`https://github.com/openclaw/openclaw`
3. 配置环境变量：

| 变量 | 值 |
|------|-----|
| OPENCLAW_SECRET | 随机字符串（如：abcd1234） |

### 6.2 绑定自有域名

1. Settings → Domains
2. 添加你的域名（如：ai.yourdomain.com）
3. 按提示配置DNS

### 6.3 配置中转（关键步骤）

创建 Vercel 函数 `/api/wecom.ts`：

```typescript
// api/wecom.ts
export default async function handler(req, res) {
  // 将请求转发到本地OpenClaw
  const response = await fetch('http://你的服务器IP:18789/webhooks/wecom', {
    method: req.method,
    headers: {
      'Content-Type': 'application/json',
      'X-OpenClaw-Secret': process.env.OPENCLAW_SECRET
    },
    body: JSON.stringify(req.body)
  });
  
  const data = await response.json();
  res.status(200).json(data);
}
```

### 6.4 企业微信回调地址

最终企业微信配置的URL为：
```
https://你的域名.com/api/wecom
```

---

## 第七步：完整配置示例

### OpenClaw 配置（最终版）

```json
{
  "channels": {
    "wecom": {
      "enabled": true,
      "token": "你生成的Token",
      "encodingAesKey": "你生成的EncodingAESKey(43位)",
      "adminUsers": ["你的用户ID"]
    }
  }
}
```

### 你需要填写的信息

| 字段 | 来源 | 示例值 |
|------|------|--------|
| token | 企业微信后台生成 | MySecretToken123 |
| encodingAesKey | 企业微信后台生成 | abcdefghijklmnopqrstuvwxyzABCDEFGHIJKL |
| adminUsers | 企业微信用户ID | zhangsan |
| Vercel域名 | 你绑定的域名 | ai.yourdomain.com |
| OPENCLAW_SECRET | 你生成的密钥 | abcd1234 |

