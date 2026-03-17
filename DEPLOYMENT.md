## HQ（海问）网站部署文档

本项目为 **前后端一体化部署**：

- 开发模式：Vite 前端（`5173`）代理到 Express 后端（`3000`）。
- 生产模式：Express 会托管 `client/dist` 静态文件（见 `server/index.js`），访问一个端口即可。

---

## 环境要求

- Node.js：建议 **18+**（推荐 20 LTS）
- npm：随 Node 安装
- Windows / macOS / Linux 均可

---

## 一键初始化（推荐）

在项目根目录 `web/`：

```bash
npm run setup
```

该命令会：

- 安装根目录依赖（仅 `concurrently`）
- 安装 `server/` 依赖
- 安装 `client/` 依赖
- 执行 `npm run seed`（写入案例库演示数据到 `server/hq.db`）

---

## 本地开发运行（开发模式）

在项目根目录 `web/`：

```bash
npm run dev
```

访问：

- 前端：`http://localhost:5173/`
- 后端 API：`http://localhost:3000/api`

说明：

- 前端通过 `client/vite.config.js` 的 proxy 把 `/api` 转发到后端。

---

## 生产部署（单机/云服务器）

### 1）配置环境变量

在 `server/` 下复制并修改：

```bash
cd server
copy .env.example .env
```

至少需要修改：

- `JWT_SECRET`：生产环境必须更换为强随机字符串
- `AI_API_BASE_URL / AI_API_KEY / AI_MODEL`：如需真实 AI 问答（不配置也可运行，系统会返回演示模式回复）

### 2）构建前端

在项目根目录 `web/`：

```bash
npm run build
```

产物输出到：`client/dist`

### 3）启动后端（同时托管前端）

在项目根目录 `web/`：

```bash
npm start
```

默认端口：`http://localhost:3000/`

---

## 常见部署方式

### 方式 A：直接在云服务器运行 Node

- 把整个 `web/` 上传到服务器
- 执行 `npm run setup`
- 执行 `npm run build`
- 执行 `npm start`

建议：

- 用反向代理（Nginx）把域名转发到 `localhost:3000`
- 用进程守护（如 PM2）保持服务常驻

### 方式 B：容器化（可选）

如果你们后续需要 Docker，我也可以补一份 `Dockerfile` + `docker-compose.yml`，把数据库文件卷挂载出来，便于持久化。

---

## 数据与持久化

- SQLite 数据库文件：`server/hq.db`
- 需要持久化部署时，请确保该文件不会在更新代码时被覆盖/删除。

