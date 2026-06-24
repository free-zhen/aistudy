# 第 21 章 · 框架速览：LangChain / LlamaIndex

> 本章目标：在手写理解了原理之后，认识两大主流框架 LangChain 与 LlamaIndex，能看懂别人的代码，并学会判断「何时该用框架、何时该手写」。
> 这是全课**最后一章**。学完，你就走完了从「零后端前端工程师」到「能独立做 LLM 应用」的完整路线。

---

## 本章目标

- [ ] 想清楚**为什么前面 20 章刻意不用框架**，以及为什么现在该了解它们
- [ ] 认识 **LangChain** 的定位与核心概念（Models / Prompts / Chains(LCEL) / Memory / Tools / LangGraph / LangSmith）
- [ ] 认识 **LlamaIndex** 的定位与核心概念（文档加载器 / 索引 / 查询引擎）
- [ ] 用一张对照表，把**自己手写过的每个组件**映射到框架里的对应模块
- [ ] 跑通一个极简的 LangChain / LlamaIndex RAG 示例（对接 DeepSeek，几行代码复刻前面几章的功能）
- [ ] 拿到一套「**框架 vs 手写**」的决策建议
- [ ] 回顾整条学习路线，明确结业后的进阶方向

---

## 核心概念

### 1. 为什么前面不用框架？为什么现在该了解？

你大概率有过这种经历：刚学前端时，有人劝你「别先学 React，先用原生 JS 把 DOM 操作、事件、`fetch` 搞明白」。等你手写过 `document.createElement`、手动 `addEventListener`、自己维护一份状态再 `innerHTML` 渲染之后，再去看 React 的 `useState`、虚拟 DOM、组件化——你会**秒懂它在替你做什么**，也知道它的边界在哪。

本课前 20 章就是这个思路。我们刻意手写了：

- `llm.py` 里的 `ask()`：自己拼 `messages`、自己调 DeepSeek（第 02 / 04 章）
- 提示工程：自己用字符串拼 system prompt、few-shot（第 06 章）
- 多轮记忆：自己用列表 / SQLite 存对话历史（第 07 章）
- Embedding + Chroma：自己算向量、自己存取（第 08 / 09 章）
- RAG：自己切分 → 检索 → 拼 prompt → 生成（第 10 / 11 章）
- Function Calling / Agent：自己定义工具、自己写「调模型 → 解析 → 执行工具 → 再调模型」的循环（第 14 / 15 章）

**为什么当时不用框架？**

| 原因 | 说明 |
|------|------|
| 框架是黑盒 | 不懂原理时，框架替你做的事你看不见，出问题完全无从下手 |
| 调试地狱 | 报错栈里全是框架内部的调用，定位不到自己的逻辑 |
| 概念过载 | 一上来就背 Runnable、Retriever、Chain 这些抽象名词，等于死记硬背 |
| 学不到本质 | RAG 的本质是「检索 + 拼 prompt」，框架一行 `.query()` 就盖住了，你以为学会了其实没有 |

**为什么现在该了解框架了？**

| 原因 | 说明 |
|------|------|
| 生态成熟 | 文档加载器、各种向量库、各家模型的适配早有人写好，不必重复造轮子 |
| 读懂他人代码 | GitHub 上、公司里、教程里的 LLM 项目，大量基于 LangChain / LlamaIndex，看不懂就寸步难行 |
| 快速搭原型 | 验证一个想法时，框架几行代码就能跑通，比手写快十倍 |
| 你已经有底气 | 现在你知道每个 `.query()` 背后是什么，框架对你不再是黑盒，而是**省力的封装** |

> 一句话：**先懂原理，再用框架。** 原理是你的，框架是工具。工具能换，原理跑不掉。

### 2. LangChain：LLM 应用编排框架

**定位**：LangChain 是一个**编排（orchestration）框架**——它本身不提供模型，而是把「模型、提示、检索、工具、记忆」这些零件用统一接口串起来，让你像搭乐高一样组装 LLM 应用。

核心概念（你几乎全都手写过，这里只是给它们起了框架里的名字）：

| LangChain 概念 | 它是什么 | 对应你手写过的 |
|----------------|----------|----------------|
| **Models（ChatModel）** | 对各家大模型的统一封装 | `llm.py` 里那段 `OpenAI(...).chat.completions.create()` |
| **Prompts（PromptTemplate）** | 带占位符的提示模板 | 你用 f-string / `.format()` 拼的 prompt |
| **Chains / LCEL** | 把多个步骤用 `\|` 管道串起来 | 你手写的「先做 A 再把结果喂给 B」的流程 |
| **Memory** | 自动管理多轮对话历史 | 第 07 章你用列表 / SQLite 存的对话 |
| **Tools** | 让模型能调用的外部函数 | 第 14 章你定义的 function calling 工具 |

**LCEL（LangChain Expression Language）** 是现在 LangChain 的主推写法，用 `|` 管道符把组件连起来，很像 Unix 管道或 RxJS 的 `pipe`：

```python
chain = prompt | model | output_parser
result = chain.invoke({"question": "什么是 RAG？"})
```

这一行的意思是：把输入塞进 `prompt` 生成完整提示 → 交给 `model` 生成回答 → 用 `output_parser` 把回答整理成你要的格式。**你在第 06、10 章其实手写过一模一样的流程，只是没有这个管道语法糖。**

**LangGraph**：当应用不再是「一条直线」而是有循环、有分支、有状态的**工作流**时（典型就是 Agent——「思考 → 调工具 → 看结果 → 再思考」的循环），用普通 Chain 表达就很别扭。LangGraph 专门用来构建这种**有状态的图（graph）工作流**：节点是步骤，边是流转，还能带条件分支和循环。**这正是第 15 章你手写的那个 Agent 循环**，LangGraph 把它变成了可声明、可可视化、可中断恢复的图。

**LangSmith**：可观测性（observability）平台。你的 Chain 跑起来后，每一步的输入输出、用了多少 token、慢在哪、为什么出错，LangSmith 都能记录和可视化。类比前端的 **Redux DevTools / React DevTools**——让你看见「框架内部到底发生了什么」，这也是从黑盒里把调试能力找回来的关键工具。

### 3. LlamaIndex：偏 RAG / 数据接入的框架

**定位**：如果说 LangChain 是「什么都能编排的通用框架」，那 **LlamaIndex 专精一件事：把你的数据接进 LLM**——也就是 RAG。它对「加载文档 → 建索引 → 检索问答」这条链路封装得最顺手。

核心概念（对应第 10 / 11 章你手写的 RAG 三件套）：

| LlamaIndex 概念 | 它是什么 | 对应你手写过的 |
|-----------------|----------|----------------|
| **文档加载器（Reader / SimpleDirectoryReader）** | 把 PDF / Markdown / 网页等读成统一的 `Document` | 第 11 章你手写的「读文件 + 解析」 |
| **索引（Index）** | 把文档切分、向量化、存进向量库的整套封装 | 第 08–10 章你手写的「切分 + Embedding + 存 Chroma」 |
| **查询引擎（Query Engine）** | 接收问题 → 检索 → 拼 prompt → 调模型 → 返回答案 | 第 10 章你手写的整条 RAG 主流程 |

用 LlamaIndex 做一遍 RAG，往往就是三步：`读目录 → 建索引 → 提问`，每步一行。后面动手实践会真正跑一遍，你会直观看到它替你省了多少代码。

### 4. 手写组件 ↔ 框架模块 映射图

把前面所有章节手写的东西，和框架里的对应模块对照起来看：

```mermaid
graph LR
    subgraph 你手写过的（本课 02-15 章）
        A1["llm.py 的 ask()<br/>手拼 messages 调 DeepSeek"]
        A2["f-string 拼 prompt"]
        A3["列表 / SQLite 存对话历史"]
        A4["切分 + Embedding + Chroma"]
        A5["检索 → 拼 prompt → 生成<br/>(手写 RAG)"]
        A6["定义工具 + Agent 循环"]
    end

    subgraph LangChain
        B1["ChatModel<br/>(ChatOpenAI 指向 DeepSeek)"]
        B2["PromptTemplate"]
        B3["Memory"]
        B5["Retriever + RetrievalQA"]
        B6["Tools + LangGraph"]
    end

    subgraph LlamaIndex
        C4["Index<br/>(向量化 + 存储封装)"]
        C5["Query Engine"]
        C7["VectorStore 封装<br/>(Chroma/FAISS...)"]
    end

    A1 --> B1
    A2 --> B2
    A3 --> B3
    A4 --> C4
    A4 --> C7
    A5 --> B5
    A5 --> C5
    A6 --> B6
```

读图要点：**左边每一个你亲手写过的组件，右边都有现成封装。** 框架没有发明新原理，它只是把你写过的代码打包成了可复用、可组合的模块。你看框架代码时，脑子里随时能把它「翻译」回左边那套手写逻辑——这就是前 20 章给你的底气。

---

## 动手实践

下面用 LangChain 和 LlamaIndex 各做一遍 RAG，对接 DeepSeek，和你第 10 章的手写版做代码量对比。

> 复用全课的密钥配置：两个框架都通过 **`ChatOpenAI` / OpenAI 兼容接口**指向 DeepSeek 的 `base_url`，密钥仍从根目录 `.env` 读取（第 00 章配好的）。

### 实践 0：安装框架

```bash
# 确保已激活 venv（命令行前缀有 (.venv)）
pip install langchain langchain-openai langchain-community
pip install llama-index llama-index-llms-openai-like llama-index-embeddings-huggingface
```

> 框架依赖较多，安装会比之前慢一些，属正常现象。

### 实践 1：用 LangChain 复刻「调模型 + 提示模板」

先感受 LangChain 怎么把第 02 / 06 章的内容用 LCEL 管道串起来。新建 `lc_basic.py`：

```python
# lc_basic.py —— 用 LangChain 复刻「提示模板 + 调 DeepSeek」
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
import os

load_dotenv()

# ① ChatModel：用 ChatOpenAI 指向 DeepSeek，等价于你 llm.py 里的 client
model = ChatOpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL"),   # https://api.deepseek.com
    model=os.getenv("DEEPSEEK_MODEL"),         # deepseek-chat
)

# ② PromptTemplate：等价于你手写的 f-string 拼 prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个只用一句话回答的助手"),
    ("user", "{question}"),
])

# ③ LCEL 管道：prompt → model → 取出文本。等价于你手写的「拼好 → 调模型 → .content」
chain = prompt | model | StrOutputParser()

print(chain.invoke({"question": "用一句话解释什么是 RAG"}))
```

```bash
python lc_basic.py
```

对照第 02 章的 `ask()`：你会发现 `prompt | model | StrOutputParser()` 这条管道，正是把当时三步手写逻辑（拼 messages、调模型、取 `.content`）用一行声明式地表达出来。

### 实践 2：用 LlamaIndex 几行实现一个完整 RAG

这是本章重头戏。回想第 10 / 11 章，你手写一个 RAG 要做：读文件 → 切分 → 算 Embedding → 存 Chroma → 检索 → 拼 prompt → 调模型，洋洋洒洒上百行。看看 LlamaIndex 要几行。

先准备一点知识库数据。新建 `data/` 目录，放一个 `data/note.txt`：

```text
RAG 全称 Retrieval-Augmented Generation，检索增强生成。
它先从知识库里检索出与问题相关的片段，再把这些片段拼进 prompt 交给大模型回答，
从而让模型能基于「私有的、最新的」资料作答，缓解幻觉问题。
本课程使用 DeepSeek 作为生成模型，使用向量数据库做检索。
```

新建 `li_rag.py`：

```python
# li_rag.py —— 用 LlamaIndex 几行实现完整 RAG（对接 DeepSeek）
from dotenv import load_dotenv
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings
from llama_index.llms.openai_like import OpenAILike
from llama_index.embeddings.huggingface import HuggingFaceEmbedding
import os

load_dotenv()

# 全局配置：把生成模型和 Embedding 模型都指向 DeepSeek（OpenAI 兼容接口）
Settings.llm = OpenAILike(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    api_base=os.getenv("DEEPSEEK_BASE_URL"),
    model=os.getenv("DEEPSEEK_MODEL"),
    is_chat_model=True,
)
# Embedding 用第 08 章的本地 BGE 模型（DeepSeek 不提供 embedding 接口），首次运行会自动下载
Settings.embed_model = HuggingFaceEmbedding(model_name="BAAI/bge-small-zh-v1.5")

# ① 文档加载器：读 data/ 下所有文件 —— 对应你第 11 章手写的「读文件 + 解析」
documents = SimpleDirectoryReader("data").load_data()

# ② 建索引：切分 + 向量化 + 入库，全包了 —— 对应第 08-10 章手写的一整套
index = VectorStoreIndex.from_documents(documents)

# ③ 查询引擎：检索 → 拼 prompt → 调模型 —— 对应第 10 章手写的 RAG 主流程
query_engine = index.as_query_engine()

resp = query_engine.query("RAG 是什么？本课程用什么做生成模型？")
print(resp.response)                 # .response 才是答案文本（query 返回的是 Response 对象）
# resp.source_nodes 能看到检索到的原文片段，对应你第 10 章手写的检索结果
```

```bash
python li_rag.py
```

> 说明：DeepSeek 不提供 embedding 接口，所以这里 Embedding 直接用第 08 章的本地模型 `BAAI/bge-small-zh-v1.5`（首次运行自动下载），生成仍走 DeepSeek。这样整段代码**开箱即可跑通**，向量选型也和你前面手写 RAG 完全一致。如果你有专门的 embedding 服务，也可换回 `OpenAILikeEmbedding` 指向它，逻辑结构不变。

### 实践 3：代码量对比

| 实现方式 | 大致行数 | 你需要亲手处理的事 |
|----------|----------|--------------------|
| 手写版（第 10 / 11 章） | 100+ 行 | 切分逻辑、Embedding 调用、Chroma 存取、检索 top-k、prompt 拼接、调模型，全部自己写 |
| LlamaIndex 版（本章） | ~15 行 | 只需「读目录 → 建索引 → 提问」三步，其余框架全包 |

**结论一目了然**：搭原型、跑通验证，框架快得多。但请注意——正因为框架把那 100 行藏进了 `from_documents()` 和 `query()` 里，**如果你没手写过，出问题时根本不知道里面发生了什么**。这就是前 20 章的价值：你现在能透过这两行，看见底下那 100 行。

### 实践 4：什么时候用框架，什么时候手写？

| 场景 | 建议 | 理由 |
|------|------|------|
| 快速验证一个想法 / 做 Demo | **用框架** | 几行跑通，省时间 |
| 标准 RAG / 标准 Agent 流程 | **用框架** | 都是成熟套路，没必要重造轮子 |
| 接入多种数据源 / 多种向量库 | **用框架** | 加载器、VectorStore 适配现成 |
| 核心业务逻辑、要求精确可控 | **手写或薄封装** | 框架的默认行为不一定符合你的业务，黑盒难调优 |
| 对成本 / 延迟 / token 极度敏感 | **手写** | 框架常有隐藏的额外调用，手写能精确控制每一次请求 |
| 团队要长期维护、避免框架锁定 | **手写或自建薄层** | 框架版本升级常有 breaking change，核心链路自己掌控更稳 |

> 实战里最常见的是**混合**：用框架的加载器和向量库封装省力，但 RAG 的检索策略、prompt、Agent 的决策循环这些**核心环节自己手写**，既快又可控。这正是「懂原理 + 会框架」的人才有的选择权。

---

## 常见报错

| 现象 | 原因 | 解决 |
|------|------|------|
| `ModuleNotFoundError: langchain_openai` | 没装对应子包 | `pip install langchain-openai`（LangChain 拆成了多个子包） |
| `ModuleNotFoundError: llama_index.llms.openai_like` | 没装 OpenAI-like 适配包 | `pip install llama-index-llms-openai-like llama-index-embeddings-openai-like` |
| `AuthenticationError` / 401 | Key 没读到或 base_url 不对 | 确认 `.env` 里 `DEEPSEEK_API_KEY` / `DEEPSEEK_BASE_URL`，参考第 00 章 |
| Embedding 相关报错 / 首次卡在下载 | 在线拉 `BAAI/bge-small-zh-v1.5` 失败 | 同第 17 章：PowerShell 先 `$env:HF_ENDPOINT="https://hf-mirror.com"` 再重跑 |
| `SimpleDirectoryReader` 报找不到文件 | `data/` 目录不存在或为空 | 确认在脚本同级建了 `data/` 并放入 `note.txt` |
| 框架升级后代码报错 | 框架有 breaking change | 锁定版本（`pip freeze`），升级时对照官方迁移文档 |
| 一行 `.query()` 行为不符预期但看不出原因 | 框架默认配置是黑盒 | 接入 **LangSmith** 或打开框架的 verbose 日志，看每一步的真实输入输出 |

---

## 小结

- 前 20 章刻意手写一切，是为了**先懂原理**——框架是黑盒，不懂原理时调试如盲人摸象；现在你懂了，框架才从黑盒变成省力工具
- **LangChain** 是通用编排框架：Models / Prompts / Chains(LCEL 管道) / Memory / Tools；**LangGraph** 构建有状态的 Agent 工作流（即你手写的 Agent 循环）；**LangSmith** 提供可观测性
- **LlamaIndex** 专精 RAG / 数据接入：文档加载器 → 索引 → 查询引擎，三步搭出一个 RAG
- 你手写过的每个组件，框架里都有对应模块（见映射图与对照表）——框架没发明新原理，只是打包了你写过的逻辑
- 同一个 RAG，手写要 100+ 行，LlamaIndex 约 15 行：**原型用框架，核心可控处手写**，混合使用是最务实的姿势

---

## 结业与进阶方向

**恭喜你——读到这里，你已经走完了整门课。** 🎉

回头看看这条路线，你从一个**只会 JavaScript、零后端基础**的前端工程师出发：

1. **起步**：装好 Python、调通了第一次 DeepSeek 对话，理解了 messages / token / 流式（00–02）
2. **打通前后端**：搞懂了后端是什么，用 FastAPI 把 LLM 包成自己的 API，再用熟悉的 JS 接上前端，做出了能用的全栈聊天应用（03–05）
3. **RAG 核心**：掌握了提示工程、多轮记忆、Embedding、向量数据库，亲手实现了完整的 RAG 链路（06–10）
4. **毕业项目与上线**：把所有知识组装成一个 RAG 知识库问答系统并部署上线（11–13）
5. **进阶**：Function Calling、Agent 等更高级的能力，直到本章——认识框架、学会取舍（14–21）

你不只是「会用」这些技术，你**手写过它们的内核**。这意味着：当框架出问题、当教程语焉不详、当业务需要定制时，你有能力深入下去——这是只会调框架 API 的人不具备的。

**结业后往哪走？** 给你几个方向：

- **深入 Agent**：用 LangGraph 构建多步骤、多工具、可中断恢复的复杂 Agent，研究 ReAct、Plan-and-Execute 等模式
- **生产化打磨**：评估（eval）、可观测（LangSmith 等）、缓存、限流、降本——把 Demo 变成扛得住真实流量的产品（可回顾第 20 章的优化思路）
- **多模态 / 更多模型**：接入语音、图像，或对比不同模型的能力与成本
- **补齐后端工程**：你已经会写 API 和部署，但数据库设计、并发、消息队列、可观测性这些后端硬功夫值得系统补强

最后一站，去**附录**把后端全景图和排错手册过一遍——它会帮你把这门课里零散补给的后端知识连成一张完整的地图，也是你日后排查问题的速查表。

**← 上一章：[第 20 章：生产环境优化](../20-production-optimization/README.md)**

**→ 结业进阶：[附录 · 后端知识地图与排错手册](../../appendix/backend-knowledge-map.md)**

> 课程到此结束。但你的 AI 全栈之路，才刚刚开始。带着「懂原理」这个最大的底气，去造点真东西吧。
