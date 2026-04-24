---
title: "零成本私有算力：GCP + Ollama + Cloudflare 全线打通指南"
description: "从零搭建一条用户浏览器 → Cloudflare Worker → Cloudflare Tunnel → GCP 私有服务器的完整链路，实现私密、安全、零成本的 AI 推理环境。"
date: 2025-03-20
tags:
  - GCP
  - Ollama
  - Cloudflare
  - Self-hosting
  - DevOps
---

## 1. 核心架构（Architecture）

本方案实现的是：**用户浏览器 → Cloudflare Worker → Cloudflare Tunnel → GCP 私有服务器 (Ollama)**。

这种架构的优势在于：

- **极致安全**：服务器无需开启公网端口，无惧扫描攻击。
- **私有私密**：对话数据不经过第三方大模型接口。
- **完全定制**：你可以自由选择任何 Ollama 支持的模型（如 Gemma 4, Llama 3）。

## 2. 实战教程：五步通关

### 第一步：服务器环境配置 (GCP)

在服务器上安装 Ollama 后，必须解除本地监听限制。

修改服务配置：

```bash
sudo systemctl edit ollama.service
```

注入环境变量：

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

重启服务：

```bash
sudo systemctl daemon-reload && sudo systemctl restart ollama
```

### 第二步：搭建数据隧道 (Cloudflare Tunnel)

这是将内网服务安全映射到公网的关键。

认证与创建：

```bash
cloudflared tunnel login
cloudflared tunnel create <name>
```

DNS 绑定：

```bash
cloudflared tunnel route dns <name> <your-subdomain.com>
```

编写 `config.yml`：

```yaml
ingress:
  - hostname: api-paychat.mmljc.com
    service: http://localhost:11434
  - service: http_status:404
```

### 第三步：实现后端代理 (Cloudflare Worker)

为了处理跨域（CORS）并隐藏真实的隧道地址，使用 Worker 作为网关：

- **流式透传**：处理 Ollama 特有的 NDJSON 流式返回。
- **路径映射**：将请求准确投递到隧道的 `/api/chat` 接口。

### 第四步：前端交互设计 (UI)

使用 HTML + JS 编写极简风格的聊天界面。

关键点：使用 `fetch` 的 `reader.read()` 方法处理流式数据，实现打字机效果。

### 第五步：自动化运营 (Systemd)

确保服务器重启后服务自动上线。

```bash
sudo cloudflared tunnel service install
```

迁移配置：将密钥和配置文件移动到 `/etc/cloudflared/`。

设为自启：

```bash
sudo systemctl enable ollama cloudflared
```

## 3. 避坑指南：这些地方最容易"翻车"

在这次实操中，我们总结出了三个核心易错点，请务必关注：

### ⚠️ 易错点 1：Ollama 的"隐身"保护

**现象**：隧道显示连接成功，但访问域名返回 502。

**原因**：Ollama 默认只听 `127.0.0.1`（只跟服务器自己聊天）。

**对策**：必须设置 `OLLAMA_HOST=0.0.0.0`。如果不做这一步，外部流量钻进隧道后，会发现服务器大门紧闭。

### ⚠️ 易错点 2：本地终端 vs. 服务器终端

**现象**：运行命令提示 `Permission denied` 或 `File not found`。

**原因**：开发者容易在本地 Mac/PC 终端里误执行服务器路径命令。

**对策**：时刻确认你的命令行前缀。所有配置文件搬运必须在 SSH 窗口内完成。

### ⚠️ 易错点 3：Systemd 权限"黑洞"

**现象**：手动运行 `cloudflared tunnel run` 没问题，但开机自启就报错。

**原因**：手动运行读取的是 `/home/user/` 下的配置，而系统服务读取的是 `/etc/cloudflared/`。

**对策**：必须将 `.json` 密钥文件和 `config.yml` 拷贝到 `/etc/cloudflared/`，并修改 `config.yml` 内部的 `credentials-file` 为绝对路径。
