## HQ（海问）自助决策咨询网站

面向中国人工智能企业出海与 ESG 创新的自助式决策咨询平台（ENT208TC Group 13）。

### 功能模块

- **注册/登录**：邮箱注册与 JWT 登录（后端 `server/` + SQLite）。
- **AI 问答**：DeepSeek 风格的对话式咨询界面，后端支持对接 OpenAI-compatible API，并在未配置时自动降级为演示模式回复。
- **案例库**：案例列表（分类/搜索/分页）+ 案例详情（Markdown 渲染）。

### 快速开始（本地）

在仓库根目录（`web/`）执行：

```bash
npm run setup
npm run dev
```

- 前端：`http://localhost:5173/`
- 后端：`http://localhost:3000/`

### 文档

- 部署文档：`DEPLOYMENT.md`
- AI API 接入文档：`API_INTEGRATION.md`

