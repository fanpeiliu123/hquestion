## HQ（海问）AI 问答 API 接入技术文档（一步步）

本项目的 AI 问答走 **后端转发**（推荐），前端只调用你们自己的 `/api/chat/...`，避免把第三方 API Key 暴露到浏览器。

后端在 `server/routes/chat.js` 已经预留 **OpenAI-compatible** 的调用方式：当 `AI_API_BASE_URL` 或 `AI_API_KEY` 未配置时，会自动使用演示模式（mock）回复，保证前端可跑通。

---

## 1. 架构概览（你需要改哪里）

- 前端：`client/src/api/index.js`
  - 发送消息：`POST /api/chat/conversations/:id/messages`
- 后端：`server/routes/chat.js`
  - `callAIAPI(messages)` 负责把对话历史转成第三方 AI API 请求
  - `SYSTEM_PROMPT` 已为“海问”场景定制（出海/ESG/合规/案例）

---

## 2. 配置环境变量（最重要）

在 `server/` 目录下复制 `.env.example`：

```bash
cd server
copy .env.example .env
```

然后编辑 `server/.env`（至少改 `JWT_SECRET`；接 AI 再改 AI 相关配置）：

- **JWT_SECRET**：登录令牌签名密钥
- **AI_API_BASE_URL**：第三方 API 基地址（OpenAI-compatible）
- **AI_API_KEY**：第三方 Key
- **AI_MODEL**：模型名（取决于供应商）

示例（DeepSeek）：

```env
AI_API_BASE_URL=https://api.deepseek.com/v1
AI_API_KEY=你的key
AI_MODEL=deepseek-chat
```

示例（通义千问兼容模式）：

```env
AI_API_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
AI_API_KEY=sk-xxx
AI_MODEL=qwen-plus
```

---

## 3. 验证后端是否读到配置

启动后端：

```bash
cd ..\web
npm run dev:server
```

如果没有配置 `AI_API_*`，你仍然可以聊天，只是回复会带“演示模式”提示。

---

## 4. 前端到后端的调用链（已实现）

前端调用封装在：

- `client/src/api/index.js`
  - `chat.createConversation(title)`
  - `chat.sendMessage(id, content)`

前端页面：

- `client/src/pages/Chat.jsx`

说明：

- 没有对话 id 时，前端会先创建对话，再发送第一条消息。
- 后端会把用户消息写入 `messages` 表，然后调用 AI，再写入 assistant 回复。

---

## 5. 后端到第三方 AI 的请求格式（当前实现）

后端目前按 OpenAI 的 `chat.completions` 兼容格式请求：

- URL：`\${AI_API_BASE_URL}/chat/completions`
- Header：
  - `Authorization: Bearer ${AI_API_KEY}`
  - `Content-Type: application/json`
- Body（关键字段）：
  - `model`
  - `messages`: `[{role:'system', content: SYSTEM_PROMPT}, ...history]`
  - `temperature`
  - `max_tokens`

如果你的供应商是 OpenAI-compatible（DeepSeek / Qwen compatible / Moonshot / OpenAI 等），通常无需改动，只要对齐 `base_url`、`model` 和 key。

---

## 6.（可选）如何支持流式输出（Streaming / 打字机效果）

当前接口是一次性返回整段文本，前端已预留“思考中…”的乐观 UI。

如果你们要做 DeepSeek/ChatGPT 那样的**流式输出**，推荐方案：

### 方案 A：后端 SSE（推荐）

1）新增一个接口：`GET /api/chat/conversations/:id/stream?prompt=...` 或 `POST /stream`
2）后端设置：

- `res.setHeader('Content-Type', 'text/event-stream')`
- `res.setHeader('Cache-Control', 'no-cache')`
- `res.flushHeaders()`

3）调用第三方的 stream 模式（不同供应商字段可能是 `stream: true`），逐段把 token 写给前端：

- `res.write("data: ...\n\n")`

4）前端用 `EventSource` 或 `fetch + ReadableStream` 接收并增量渲染。

### 方案 B：前端直连第三方（不推荐）

会暴露 API Key，除非你们有自己的 token 交换/签名网关。

如果你们决定上流式，我可以在现有代码上直接补齐 SSE 版本（后端 + 前端增量渲染）。

---

## 7. 错误处理与降级策略（已内置）

后端 `callAIAPI()` 里包含三层兜底：

- **未配置 `AI_API_BASE_URL` 或 `AI_API_KEY`**：直接返回 mock 回复
- **第三方返回非 2xx**：记录错误并返回 mock 回复
- **网络/异常**：记录错误并返回 mock 回复

这保证了演示与答辩时，即使第三方不可用，也能稳定展示完整功能链路。

