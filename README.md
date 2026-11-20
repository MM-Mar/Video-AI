<div align="center">
  <h1>🎬 Video-AI</h1>
  <p align="center">
    <img src="https://img.shields.io/badge/Java-21-E76F00?style=flat-square&logo=openjdk" alt="Java 21">
    <img src="https://img.shields.io/badge/Spring%20Boot-3.5-6DB33F?style=flat-square&logo=springboot" alt="Spring Boot">
    <img src="https://img.shields.io/badge/Vue-3-42B883?style=flat-square&logo=vuedotjs" alt="Vue 3">
    <img src="https://img.shields.io/badge/MySQL-8-4479A1?style=flat-square&logo=mysql" alt="MySQL">
    <img src="https://img.shields.io/badge/Redis-7-DC382D?style=flat-square&logo=redis" alt="Redis">
    <img src="https://img.shields.io/badge/MIT-License-blue?style=flat-square" alt="MIT">
  </p>
  <p align="center">
    <strong>面向长视频内容理解的 Video Agent</strong><br/>
    将长视频转化为可检索、可追溯、可追问的结构化知识
  </p>
</div>

---

## ✨ 核心特性

| 特性 | 说明 |
|------|------|
| 🎬 **可靠任务链路** | 分片上传 + 断点续传 + 异步削峰，提交即返回 |
| 🧩 **多模态上下文** | ASR + OCR + 关键帧融合，统一时序结构 |
| 🔁 **证据约束 AgentLoop** | Planner → Executor → Critic，每条结论必绑时间戳证据 |
| 🔍 **长视频检索** | Embedding + 关键词混合召回，Qdrant 向量库 |
| 🛡️ **成本护栏** | 令牌桶限流 + 指数退避重试 + Agent 轮次上限 |
| 🔄 **断点恢复** | MySQL Checkpoint + Redis 热缓存，任务中断可续跑 |

## 🖼️ 项目预览

**登录注册**

![登录](docs/images/login-register.png)

**视频工作台**

![工作台](docs/images/video-library.png)

**Agent 目标输入**

![Agent](docs/images/agent-compose.png)

用户完成登录后，上传视频 → 选择视频 → 输入分析目标 → 手动或自动选择分析模式。工作台展示结构化结论、时间戳证据、执行计划、阶段轨迹与质量评估，并支持基于同一视频继续追问。

## 📐 系统架构

```mermaid
flowchart TD
    A[用户上传视频 + 分析目标] --> B[Vue 前端]
    B --> C[Spring Boot API]
    C --> D[RocketMQ 异步投递]
    D --> E[分析消费者]
    
    E --> F[VideoContext 构建]
    F --> F1[FFmpeg 音频分段]
    F --> F2[关键帧抽取]
    F1 --> F3[ASR 语音识别]
    F2 --> F4[OCR 文字识别]
    
    F3 --> G[MySQL + Redis Checkpoint]
    F4 --> G
    
    E --> H[混合检索]
    H --> H1[Qdrant 语义召回]
    H --> H2[关键词匹配]
    
    H --> I[AgentLoop]
    I --> I1[Planner 任务拆解]
    I --> I2[Executor 生成结论]
    I --> I3[Critic 校验证据]
    I3 -- "未通过且未超两轮" --> I
    
    I --> G
    G --> J[SSE 推送前端]
    J --> B
```

## 🧩 四类分析模式

| 模式 | 产物 |
|------|------|
| **通用** | 结构化结论 + 时间戳证据 + 建议 |
| **学习** | 大纲 + 自测题 + 易错点 |
| **审查** | 逻辑漏洞 + 存疑结论定位 |
| **创作** | 爆点提取 + 标题 + 口播脚本 |

---

## 🛠️ 技术栈

| 层次 | 技术 |
|------|------|
| 前端 | Vue 3、Vite、SSE、Marked |
| 后端 | Java 21、Spring Boot 3.5、Undertow、MyBatis-Plus |
| 中间件 | RocketMQ、Redis + Redisson、MySQL、MinIO、Qdrant |
| 视频处理 | FFmpeg、Tesseract OCR、yt-dlp |
| AI | LangChain4j、DeepSeek、TeleSpeechASR、BGE-M3 |

---

## 🚀 快速开始

### 环境要求

| 组件 | 版本 | 用途 |
|------|------|------|
| JDK | 21 | 后端运行 |
| Node.js | 22 | 前端构建 |
| Docker | Compose v2 | 中间件编排 |
| FFmpeg | — | 音视频处理 |
| Tesseract | 含 `chi_sim` `eng` 语言包 | OCR |
| yt-dlp | 可选 | 解析在线视频链接 |

### 1. 准备配置

```bash
# Windows PowerShell
Copy-Item .env.example .env

# macOS / Linux
cp .env.example .env
```

编辑 `.env`，至少设置：

- 数据库密码（`DB_PASSWORD`）
- MinIO / Qdrant 密码
- **`SILICONFLOW_API_KEY`**（AI 接口必需）

> ⚠️ `.env` 已在 `.gitignore` 中，密钥只保存在本地，不要提交到仓库。

### 2. 启动中间件

```bash
# macOS / Linux
./scripts/dev-up.sh

# Windows：手动启动，或修改脚本后用 Git Bash / WSL 运行
docker compose up -d mysql redis minio qdrocketmq-namesrv rmqbroker
```

脚本会等待 MySQL、Redis、MinIO、Qdrant 和 RocketMQ 全部就绪。所有服务默认只监听 `127.0.0.1`，不会暴露到局域网。

### 3. 启动后端

```bash
# macOS / Linux
set -a && source .env && set +a
cd server && ./mvnw spring-boot:run

# Windows PowerShell
$env:DB_URL = "jdbc:mysql://localhost:3306/media_db?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf-8&allowPublicKeyRetrieval=true"
# （复制 .env 里的其他变量到当前会话）
cd server && .\mvnw.cmd spring-boot:run
```

后端启动后访问 `http://localhost:9090/health`，应返回 `{"code":0,"message":"success","data":"UP"}`。

> 💡 推荐在 IDE 中安装 `.env files` / `EnvFile` 插件，Run Configuration 里直接关联 `.env`，免去手动 set 环境变量。

### 4. 启动前端

```bash
# macOS / Linux
set -a && source .env && set +a
cd client && npm ci && npm run dev

# Windows PowerShell
cd client && npm ci && npm run dev
```

打开 `http://localhost:5173` 即可使用。

> 🔬 只想看前端 UI：`http://localhost:5173/?demo` 进入 Demo 模式，使用内置示例数据，不依赖后端。

---

## ❓ 常见问题

| 现象 | 处理 |
|------|------|
| 后端连不上 MySQL / Redis | `docker compose ps` 确认服务健康，检查 `.env` 密码是否一致 |
| 前端无法连接后端 | 先访问 `/health` 确认后端存活，再检查 `VITE_DEV_PROXY_TARGET` |
| 视频解析提示命令不存在 | 确认 `ffmpeg` / `tesseract` 在终端可用，必要时设置 `FFMPEG_DIR` / `OCR_COMMAND` |
| AI 接口 401 或模型不可用 | 检查 `SILICONFLOW_API_KEY` 和模型名称，修改后重启后端 |
| Maven 提示 `maven-default-http-blocker` | `cd server && ./mvnw -s .mvn/central-settings.xml spring-boot:run` |

### 停止 & 清理

```bash
# 停止但保留数据
docker compose down

# 完全重置（⚠️ 会删除所有视频数据）
docker compose down --volumes
```

---

## 📁 项目结构

```
Video-AI/
├── client/              # Vue 3 前端工作台
├── server/              # Spring Boot 后端 + Video Agent
├── docker-compose.yml   # 中间件编排
├── scripts/             # 开发辅助脚本
└── .env.example         # 配置模板（不含真实密钥）
```

## 📄 License

[MIT License](LICENSE)