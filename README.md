# AI 全栈学习项目 · 从前端到 LLM 应用开发

> 写给**只会 JavaScript、零后端基础**的前端工程师。
> 目标：用 **Python + DeepSeek**，从零搭出一个完整的 **RAG 知识库问答系统**（毕业项目）。

---

## 这是什么

这是一套「随时可学」的渐进式课程。每一章是一个独立目录，里面有完整的讲解、可运行的代码和动手练习。你不需要先懂后端——后端知识会**穿插**在需要它的地方讲（标了 🐍 的章节）。

学习方式很简单：**从第 00 章开始，按顺序读 `README.md`，跟着敲代码。**

---

## 学习路线图（22 章 + 附录）

> 分两部分：**基础篇（00–13）** 带你从零做出并部署完整 RAG 毕业项目；**进阶篇（14–21）** 对标业界标准，补齐 Agent、进阶 RAG、评估、安全、生产化等"从能跑到好用、能上线"的关键能力。基础篇学完即可独立做项目，进阶篇可按需选学。

### 基础篇 · 从零到毕业项目

| 章节 | 主题 | 你会学到 | 后端补给 |
|------|------|----------|:---:|
| [第 00 章](chapters/00-env-and-apikey/README.md) | **环境准备与全局 API Key 配置** | 装 Python、注册 DeepSeek、配置全局密钥 | |
| [第 01 章](chapters/01-python-for-js-devs/README.md) | **Python 速成（写给 JS 工程师）** | 用 JS 对比快速掌握 Python 语法、venv、pip | |
| [第 02 章](chapters/02-first-llm-call/README.md) | **第一次调用 LLM API** | 用 Python 调通 DeepSeek，理解 messages/token/流式 | |
| [第 03 章](chapters/03-backend-http-fastapi/README.md) | **HTTP 与 FastAPI 入门** | 后端到底是什么、写出第一个 API 接口 | 🐍 |
| [第 04 章](chapters/04-llm-backend-api/README.md) | **把 LLM 包成自己的后端 API** | FastAPI + DeepSeek，做 `/chat` 流式接口 | 🐍 |
| [第 05 章](chapters/05-frontend-integration/README.md) | **前端接入后端** | 用你会的 JS 写聊天界面、CORS、流式渲染 | |
| [第 06 章](chapters/06-prompt-engineering/README.md) | **提示工程** | system prompt、few-shot、让 AI 输出 JSON | |
| [第 07 章](chapters/07-memory-and-persistence/README.md) | **多轮对话与记忆** | 会话状态管理 + SQLite 数据持久化 | 🐍 |
| [第 08 章](chapters/08-embeddings/README.md) | **Embedding 与文本向量** | 文本如何变成向量、相似度怎么算 | |
| [第 09 章](chapters/09-vector-database/README.md) | **向量数据库** | Chroma 入门，存储与检索 | |
| [第 10 章](chapters/10-rag-fundamentals/README.md) | **RAG 原理与最小实现** | 切分→检索→拼 prompt→生成的完整链路 | |
| [第 11 章](chapters/11-capstone-backend/README.md) | **毕业项目① RAG 后端** | 上传→解析→向量化→入库→检索→问答 | 🐍 |
| [第 12 章](chapters/12-capstone-frontend/README.md) | **毕业项目② RAG 前端** | 上传界面 + 聊天 + 引用来源展示 | |
| [第 13 章](chapters/13-deployment/README.md) | **部署上线** | 环境变量、Docker 简介、部署到云 | 🐍 |

### 进阶篇 · 对标业界，从"能跑"到"能上线"

| 章节 | 主题 | 你会学到 |
|------|------|----------|
| [第 14 章](chapters/14-function-calling/README.md) | **Function Calling 工具调用** | 让模型调用你定义的函数，Agent 的地基 |
| [第 15 章](chapters/15-ai-agent/README.md) | **从零写 AI Agent** | ReAct 思考-行动-观察循环，让 AI 会做事 |
| [第 16 章](chapters/16-mcp/README.md) | **MCP（Model Context Protocol）** | 工具/上下文接入的事实标准，Claude 生态原生 |
| [第 17 章](chapters/17-advanced-rag/README.md) | **进阶 RAG** | 重排序、混合检索、查询改写、拒答防幻觉 |
| [第 18 章](chapters/18-evaluation-and-testing/README.md) | **评估与测试** | 评测集、检索指标、LLM-as-judge、回归测试 |
| [第 19 章](chapters/19-security-prompt-injection/README.md) | **安全与提示注入防护** | 直接/间接注入、OWASP LLM Top 10、纵深防御 |
| [第 20 章](chapters/20-production-optimization/README.md) | **生产化** | 成本/Token 优化、缓存、限流、重试、可观测 |
| [第 21 章](chapters/21-frameworks-langchain-llamaindex/README.md) | **框架速览** | LangChain / LlamaIndex，何时用、怎么对接 |

> 📎 [附录：后端知识地图与排错手册](appendix/backend-knowledge-map.md) —— 后端全景图、常见报错速查，随时查阅。

**学习阶段：**
1. **起步（00–02）**：环境 + Python + 调通大模型
2. **打通前后端（03–05）**：理解后端 + 做出能用的全栈聊天应用
3. **RAG 核心知识（06–10）**：提示工程、记忆、向量、检索
4. **毕业项目与上线（11–13）**：把所有知识组装成 RAG 系统并部署
5. **进阶篇（14–21）**：Agent / MCP / 进阶 RAG / 评估 / 安全 / 生产化 / 框架（按需选学）

---

## 全局 API Key 配置（重要）

**所有章节共用同一份密钥配置**，只需配置一次。

1. 复制模板：把 `config/.env.example` 复制为项目根目录的 `.env`
2. 填入你的 DeepSeek API Key（怎么注册和拿 key 见 [第 00 章](chapters/00-env-and-apikey/README.md)）
3. 之后每章的代码都会自动从 `.env` 读取密钥，不用每次硬编码

```
config/.env.example   ← 模板（已提交，不含真实密钥）
.env                  ← 你的真实密钥（已被 .gitignore 忽略，绝不会泄露）
```

> ⚠️ **安全铁律**：`.env` 永远不要提交到 Git、不要发给别人、不要写进代码。详见第 00 章。

---

## 开始之前你需要准备

- 一台能上网的电脑（Windows / Mac 都行，本课程命令以 Windows PowerShell 为主，会标注差异）
- 一个代码编辑器（推荐 VS Code，你大概率已经在用）
- 一点点耐心：每章大约 1–3 小时，建议每天 1 章

**现在就开始 → [第 00 章：环境准备与全局 API Key 配置](chapters/00-env-and-apikey/README.md)**

> 基础篇 00–13 + 进阶篇 14–21 + 附录均已就绪。先按顺序学完基础篇做出毕业项目，再按需选学进阶篇。

---

## 项目结构

```
ai/
├── README.md                 # 你正在看的总路线图
├── config/
│   └── .env.example          # API Key 配置模板
├── .gitignore                # 保护 .env 不被提交
├── chapters/                 # 22 章学习内容（基础篇 00-13 + 进阶篇 14-21）
│   ├── 00-env-and-apikey/
│   ├── 01-python-for-js-devs/
│   └── ...
├── appendix/                 # 附录
└── .workflow/                # 课程生成计划（可忽略）
```
