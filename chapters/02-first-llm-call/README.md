# 第 02 章 · 第一次调用 LLM API

> 本章目标：用 Python 调通 DeepSeek，让 AI 真正回答你的问题。
> 这是整门课第一个「跑起来有 AI 味」的时刻。

---

## 本章目标

- [ ] 用官方 SDK 调通一次 DeepSeek 对话
- [ ] 看懂 `messages` 结构（system / user / assistant 三种角色）
- [ ] 理解 token 是什么、怎么影响计费
- [ ] 实现**流式输出**（像 ChatGPT 那样一个字一个字蹦出来）
- [ ] 把这些封装成一个可复用的函数

---

## 核心概念

### 1. 调用大模型 = 发一个 HTTP 请求

调 DeepSeek 本质上就是：把你的问题打包成 JSON，带上 API Key，POST 给它的服务器，它返回 AI 生成的文字。

你完全可以用第 01 章装的 `requests` 手写这个 HTTP 请求。但 DeepSeek **兼容 OpenAI 接口**，所以我们直接用更方便的 `openai` SDK——它帮你处理了请求细节。

```bash
# 确保已激活 venv，然后安装
pip install openai python-dotenv
```

### 2. messages 与三种角色

和大模型对话，输入是一个 **消息列表**，每条消息有个 `role`：

| role | 含义 | 类比 |
|------|------|------|
| `system` | 给 AI 设定身份/规则（开场前的导演说明） | 「你是一个简洁的编程助手」 |
| `user` | 用户说的话 | 你的提问 |
| `assistant` | AI 之前说过的话 | 用于多轮对话时回传上下文 |

```python
messages = [
    {"role": "system", "content": "你是一个只用一句话回答的助手"},
    {"role": "user", "content": "什么是 API？"},
]
```

> 第 07 章会讲：多轮对话其实就是不断往这个列表里追加 user / assistant 消息。

### 3. token 是什么

模型不是按「字数」而是按 **token** 计费和限制长度。token 是文本的最小单位，大致：

- 英文：1 token ≈ 0.75 个单词
- 中文：1 个汉字 ≈ 1～2 token

**输入（你的 messages）+ 输出（AI 的回答）都算 token。** 这就是为什么 prompt 不要写太啰嗦——既费钱又占上下文。每次响应里都能拿到本次用了多少 token。

---

## 动手实践

### 实践 1：最小可运行的一次对话

新建 `hello_llm.py`：

```python
# hello_llm.py —— 第一次调用大模型
from dotenv import load_dotenv
from openai import OpenAI
import os

load_dotenv()  # 加载根目录 .env 里的密钥（第 00 章配好的）

# 创建客户端：把 base_url 指向 DeepSeek，就能用 OpenAI 的 SDK 调 DeepSeek
client = OpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL"),  # https://api.deepseek.com
)

# 发起一次对话
response = client.chat.completions.create(
    model=os.getenv("DEEPSEEK_MODEL"),   # deepseek-chat
    messages=[
        {"role": "system", "content": "你是一个简洁专业的编程助手"},
        {"role": "user", "content": "用一句话解释什么是后端"},
    ],
)

# 取出 AI 的回答
answer = response.choices[0].message.content
print("AI 回答：", answer)

# 看看这次花了多少 token
usage = response.usage
print(f"输入 {usage.prompt_tokens} + 输出 {usage.completion_tokens} = 共 {usage.total_tokens} token")
```

运行：

```bash
python hello_llm.py
```

期望看到 AI 的回答和 token 统计。**恭喜——你刚刚完成了第一次 AI 调用。**

> `response.choices[0]` 为什么有个 `[0]`？因为理论上能让模型一次生成多个候选答案，默认只要第一个。

### 实践 2：流式输出（像 ChatGPT 那样逐字显示）

上面的写法要等 AI 全部生成完才一次性返回。设 `stream=True` 就能一个片段一个片段地接收，体验好得多。

新建 `stream_llm.py`：

```python
# stream_llm.py —— 流式输出
from dotenv import load_dotenv
from openai import OpenAI
import os

load_dotenv()
client = OpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL"),
)

stream = client.chat.completions.create(
    model=os.getenv("DEEPSEEK_MODEL"),
    messages=[{"role": "user", "content": "用三句话讲讲 RAG 是什么"}],
    stream=True,   # ← 关键
)

# 逐块接收并打印（end="" 表示不换行，flush=True 表示立即显示）
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        print(delta, end="", flush=True)
print()  # 结束后换行
```

运行后你会看到文字像打字机一样蹦出来。**第 04 章我们会把这个流式能力做成后端接口，第 05 章在前端页面上呈现这个打字机效果。**

### 实践 3：封装成可复用函数

后面每章都要调模型，先封装好，避免重复。新建 `llm.py`：

```python
# llm.py —— 全课通用的大模型调用封装
from dotenv import load_dotenv
from openai import OpenAI
import os

load_dotenv()

_client = OpenAI(
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url=os.getenv("DEEPSEEK_BASE_URL"),
)
_model = os.getenv("DEEPSEEK_MODEL")


def ask(question: str, system: str = "你是一个乐于助人的助手") -> str:
    """问一个问题，返回完整答案字符串。"""
    response = _client.chat.completions.create(
        model=_model,
        messages=[
            {"role": "system", "content": system},
            {"role": "user", "content": question},
        ],
    )
    return response.choices[0].message.content or ""   # 内容可能为 None，兜底成空串


# 直接运行本文件时做个自测
if __name__ == "__main__":
    print(ask("你好，简单介绍下你自己"))
```

```bash
python llm.py
```

> `if __name__ == "__main__":` 是 Python 的惯用法：意思是「只有直接运行这个文件时才执行下面的代码，被别处 import 时不执行」。类似一个内置的自测入口。

---

## 常见报错

| 现象 | 原因 | 解决 |
|------|------|------|
| `AuthenticationError` / 401 | Key 错或没读到 | 检查 `.env` 的 `DEEPSEEK_API_KEY`，重跑第 00 章的验证 |
| `Insufficient Balance` / 余额不足 | 没充值 | 去 DeepSeek 后台充几块钱 |
| `Connection error` | 网络问题 / base_url 写错 | 确认 `DEEPSEEK_BASE_URL=https://api.deepseek.com` |
| `ModuleNotFoundError: openai` | 没装包 / 没激活 venv | 确认 `(.venv)` 后 `pip install openai` |
| 返回很慢 | 正常，模型在生成 | 用 `stream=True` 提升体感 |

---

## 小结

- 调大模型 = 带着 API Key 发 HTTP 请求；用 `openai` SDK + DeepSeek 的 `base_url` 最省事
- 输入是 `messages` 列表，`system` 设定身份、`user` 是提问、`assistant` 是 AI 历史回复
- 按 token 计费，输入输出都算；`response.usage` 能看用量
- `stream=True` 实现逐字输出
- 已经把调用封装进 `llm.py` 的 `ask()`，后面直接复用

## 下一章预告

现在你能在**自己的脚本**里调 AI 了。但要做成一个网页应用，前端得能调到它——而前端**不能直接拿着你的密钥**调大模型（会泄露）。所以我们需要一个「后端」当中间人。

下一章 🐍 正式补后端：**HTTP 到底是什么、后端在干嘛，并用 FastAPI 写出你的第一个 API 接口。**

**→ [第 03 章：HTTP 与 FastAPI 入门](../03-backend-http-fastapi/README.md)**
