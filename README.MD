# DeepSeek2API - 本地化部署版

一个专注于本地 Python 部署的 DeepSeek API 兼容服务，支持完整的 OpenAI 接口规范。

## ✨ 特性

- ✅ **完全兼容 OpenAI API**：无缝对接现有 OpenAI 客户端和工具
- ✅ **高速流式输出**：支持实时流式响应，提升用户体验
- ✅ **多轮对话支持**：完整的对话历史管理
- ✅ **R1 深度思考**：支持 DeepSeek R1 模型的深度推理模式
- ✅ **多账号轮换**：支持多个 DeepSeek 账号自动切换
- ✅ **工具函数调用**：完整的 OpenAI tools 支持（详见 [TOOLS_SUPPORT.md](TOOLS_SUPPORT.md)）
- ✅ **自动会话清理**：对话结束后自动删除 DeepSeek 会话
- ✅ **本地优先**：专注于本地部署，无外部依赖

## 📋 目录

* [免责声明](#免责声明)
* [快速开始](#快速开始)
  * [环境要求](#环境要求)
  * [安装步骤](#安装步骤)
* [配置说明](#配置说明)
  * [账号配置](#账号配置)
  * [API 密钥](#api-密钥)
* [接口文档](#接口文档)
  * [模型列表](#模型列表)
  * [对话补全](#对话补全)
* [工具调用](#工具调用)
* [高级配置](#高级配置)
  * [Nginx 反向代理](#nginx-反向代理)
* [常见问题](#常见问题)
* [更新日志](#更新日志)

## ⚠️ 免责声明

**重要提示：此项目仅供学习和研究使用**

1. **非官方接口**：本项目使用的是逆向工程实现的 API，非 DeepSeek 官方提供
2. **稳定性风险**：逆向 API 可能随时失效或变更，建议优先使用官方付费 API
3. **使用限制**：仅限个人学习和研究使用，禁止用于商业用途或对外提供服务
4. **账号风险**：使用此服务可能导致 DeepSeek 账号被封禁，请谨慎使用
5. **责任声明**：使用本项目产生的任何后果由使用者自行承担

**官方推荐**：如需稳定可靠的 API 服务，请前往 [DeepSeek 官方平台](https://platform.deepseek.com/) 使用付费 API。

## 🚀 快速开始

### 环境要求

- Python 3.8 或更高版本
- 一个或多个 DeepSeek 账号（邮箱或手机号登录）
- 稳定的网络连接

### 安装步骤

1. **克隆项目**
   ```bash
   git clone https://github.com/1131062103/deepseek2api.git
   cd deepseek2api
   ```

2. **创建虚拟环境**（推荐）
   ```bash
   # Windows
   python -m venv .venv
   .venv\Scripts\activate

   # Linux/Mac
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **安装依赖**
   ```bash
   pip install -r requirements.txt
   ```

4. **配置账号**
   ```bash
   # 复制配置文件模板
   cp config.example.json config.json
   # 编辑 config.json，添加你的账号信息
   ```

5. **启动服务**
   ```bash
   python app.py
   ```

6. **验证服务**
   服务将在 `http://localhost:5001` 启动，访问 `http://localhost:5001` 查看欢迎页面。

## ⚙️ 配置说明

### 账号配置

项目使用 `config.json` 文件进行配置（已添加到 `.gitignore` 中，不会被提交到版本库）。

1. **复制配置模板**
   ```bash
   cp config.example.json config.json
   ```

2. **编辑 `config.json`**
   ```json
   {
     "keys": [
       "your-custom-api-key-1",
       "your-custom-api-key-2"
     ],
     "accounts": [
       {
         "email": "your-email@example.com",
         "password": "your-password",
         "token": ""
       },
       {
         "mobile": "13800138000",
         "password": "your-password",
         "token": ""
       }
     ]
   }
   ```

### 配置项说明

- **`keys`**：API 鉴权密钥列表，客户端调用时需要提供其中一个密钥
  - 格式：`Authorization: Bearer your-custom-api-key`
  - 支持多个密钥，用于不同客户端或用户

- **`accounts`**：DeepSeek 账号列表
  - 支持邮箱登录：`{"email": "...", "password": "...", "token": ""}`
  - 支持手机号登录：`{"mobile": "...", "password": "...", "token": ""}`
  - `token` 字段留空，程序会自动获取和更新
  - 建议配置多个账号，系统会自动轮换使用

### 多账号优势

- **负载均衡**：多个请求可以同时使用不同账号
- **容错能力**：某个账号失效时自动切换到其他账号
- **速率限制**：避免单个账号触发频率限制

## 📡 接口文档

本项目完全兼容 OpenAI API 规范，支持所有标准的 OpenAI 客户端和工具。

### 模型列表

**GET `/v1/models`**

获取支持的模型列表。

```bash
curl http://localhost:5001/v1/models
```

响应示例：
```json
{
  "data": [
    {
      "id": "deepseek-chat",
      "object": "model",
      "created": 1677610602,
      "owned_by": "deepseek"
    },
    {
      "id": "deepseek-reasoner",
      "object": "model",
      "created": 1677610602,
      "owned_by": "deepseek"
    },
    {
      "id": "deepseek-chat-search",
      "object": "model",
      "created": 1677610602,
      "owned_by": "deepseek"
    },
    {
      "id": "deepseek-reasoner-search",
      "object": "model",
      "created": 1677610602,
      "owned_by": "deepseek"
    }
  ],
  "object": "list"
}
```

### 对话补全

**POST `/v1/chat/completions`**

核心对话接口，支持流式和非流式响应。

**请求头**：
```
Authorization: Bearer your-api-key
Content-Type: application/json
```

**请求示例**：
```bash
curl http://localhost:5001/v1/chat/completions \
  -H "Authorization: Bearer your-api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [
      {"role": "user", "content": "你好，请介绍一下你自己"}
    ],
    "stream": false
  }'
```

**参数说明**：
- `model`：模型名称
  - `deepseek-chat` / `deepseek-v3`：标准对话模型
  - `deepseek-reasoner` / `deepseek-r1`：深度思考模型
  - `deepseek-chat-search` / `deepseek-v3-search`：联网搜索模型
  - `deepseek-reasoner-search` / `deepseek-r1-search`：联网搜索+深度思考
- `messages`：对话消息数组
- `stream`：是否启用流式响应（默认 `false`）
- `temperature`：温度参数（0-2）
- `max_tokens`：最大生成 token 数
- `tools`：工具定义数组（详见工具调用）

**响应示例**：
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1738598554,
  "model": "deepseek-chat",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "你好！我是DeepSeek，一个由深度求索公司开发的AI助手..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 50,
    "total_tokens": 60
  }
}
```

## 🔧 工具调用

本项目完整支持 OpenAI 格式的工具调用（function calling），可用于：

- **MCP (Model Context Protocol)** 集成
- **OpenClaw** 本地技能调用
- 自定义工具函数
- 任何符合 OpenAI tools 规范的应用

**详细文档请查看**：[TOOLS_SUPPORT.md](TOOLS_SUPPORT.md)

### 基本使用示例

```python
import requests

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取城市天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "城市名称"}
                },
                "required": ["location"]
            }
        }
    }
]

response = requests.post(
    "http://localhost:5001/v1/chat/completions",
    headers={"Authorization": "Bearer your-api-key"},
    json={
        "model": "deepseek-chat",
        "messages": [{"role": "user", "content": "北京今天天气怎么样？"}],
        "tools": tools
    }
)
```

## 🌐 高级配置

### Nginx 反向代理

如果需要通过 Nginx 代理服务，建议添加以下配置优化流式响应：

```nginx
location / {
    proxy_pass http://localhost:5001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;

    # 流式响应优化
    proxy_buffering off;
    chunked_transfer_encoding on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 120;
}
```

### 生产环境部署

对于生产环境，建议使用进程管理器：

```bash
# 使用 pm2（Node.js 进程管理器）
npm install -g pm2
pm2 start "python app.py" --name deepseek2api

# 或使用 systemd（Linux）
sudo nano /etc/systemd/system/deepseek2api.service
```

systemd 服务文件示例：
```ini
[Unit]
Description=DeepSeek2API Service
After=network.target

[Service]
Type=simple
User=your-user
WorkingDirectory=/path/to/deepseek2api
Environment=PATH=/path/to/venv/bin
ExecStart=/path/to/venv/bin/python app.py
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

## ❓ 常见问题

### Q: 服务启动失败怎么办？
A: 检查以下项目：
1. Python 版本是否为 3.8+
2. 依赖是否安装完整：`pip install -r requirements.txt`
3. 端口 5001 是否被占用
4. `config.json` 文件是否存在且格式正确

### Q: 如何查看日志？
A: 服务日志直接输出到控制台。对于生产环境，可以重定向到文件：
```bash
python app.py > deepseek.log 2>&1 &
```

### Q: 支持哪些客户端？
A: 任何兼容 OpenAI API 的客户端都支持，包括：
- OpenAI Python/JavaScript SDK
- LangChain
- LlamaIndex
- 各种 AI 应用和工具

### Q: 如何更新账号 token？
A: 程序会自动管理 token，当 token 失效时会自动重新登录获取。

## 📝 更新日志

详细更新记录请查看 [CHANGELOG.md](CHANGELOG.md)

### 近期更新
- ✅ 重构工具调用处理，支持完整的 OpenAI tools 功能
- ✅ 优化项目结构，移除不必要的部署文件
- ✅ 改进错误处理和日志输出
- ✅ 增强多账号管理机制

---

**提示**：本项目为学习和研究用途，建议优先使用 DeepSeek 官方 API 服务。

## 📄 许可证

本项目基于 MIT 许可证开源，详见 [LICENSE](LICENSE) 文件。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request 来改进本项目。

## ⭐ 支持

如果这个项目对你有帮助，请给个 Star ⭐
