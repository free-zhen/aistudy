# 第 00 章 · 环境准备与全局 API Key 配置

> 本章目标：装好 Python、注册 DeepSeek 拿到 API Key、配置一份**全局密钥**供之后所有章节共用。
> 这一章不写功能代码，但它是后面一切的地基。**别跳过。**

---

## 本章目标

- [ ] 安装 Python 3.10+ 并能在终端运行
- [ ] 注册 DeepSeek 账号，拿到 API Key，理解计费
- [ ] 把密钥配置到全局 `.env`，并验证能读到它
- [ ] 理解为什么密钥绝对不能写进代码

---

## 核心概念

### 1. 什么是 API Key？（用前端类比）

你在前端调过第三方接口吧？很多接口要带一个 `token` 或 `Authorization` 头来证明「你是谁、你付费了」。**API Key 就是这个凭证**。

调用 DeepSeek 大模型，本质上就是向它的服务器发一个 HTTP 请求，请求头里带上你的 Key。Key 对应你的账户余额——所以 **Key 泄露 = 别人花你的钱**。

### 2. 为什么选 DeepSeek？

| 对比项   | DeepSeek                 | OpenAI / Claude |
| -------- | ------------------------ | --------------- |
| 国内访问 | ✅ 直连，无需翻墙        | ❌ 需要特殊网络 |
| 充值     | ✅ 支付宝/微信，几块钱起 | 需要海外信用卡  |
| 接口风格 | 兼容 OpenAI 标准         | 标准本身        |
| 价格     | 极低，适合大量练习       | 较高            |

对学习者来说 DeepSeek 性价比最高，而且它**兼容 OpenAI 接口**——学会它，将来换任何模型都几乎零成本。

---

## 动手实践

### 步骤 1：安装 Python

前端有 Node.js，后端这门课我们用 **Python**。

1. 打开 https://www.python.org/downloads/ 下载 **3.10 或更高版本**
2. **Windows 安装时务必勾选 `Add Python to PATH`**（这一步漏了后面命令会找不到 python）
3. 验证安装——打开终端（Windows 用 PowerShell，Mac 用 Terminal）：

```bash
python --version
# 期望输出类似：Python 3.12.4
```

> Mac 上如果 `python` 不行，试 `python3 --version`。本课程统一写 `python`，Mac 用户请自行替换为 `python3`。

还要确认 `pip`（Python 的包管理器，相当于 npm）可用：

```bash
pip --version
# 期望输出类似：pip 24.0 from ...
```

### 步骤 2：注册 DeepSeek 并获取 API Key

1. 打开 **https://platform.deepseek.com/**
2. 用手机号/邮箱注册并登录
3. 左侧菜单进入 **「API Keys」**（或「API keys」）页面
4. 点击 **「创建 API Key」**，起个名字（如 `study`），点击创建
5. **立刻复制弹出的 Key**（形如 `sk-xxxxxxxxxxxxxxxx`）——它**只显示这一次**，关掉就再也看不到了。复制下来先存好。

**关于计费：**

- 进入「充值」页面，用支付宝/微信充几块钱就够整门课练习了
- DeepSeek 按 **token**（约等于字数）计费，价格极低；一次普通对话通常不到 1 分钱
- 在「用量」页面能实时看到花了多少钱

> 💡 注册地址再贴一次，方便随时打开：**https://platform.deepseek.com/**

### 步骤 3：配置全局 API Key（核心）

整门课**只配一次**，所有章节共用。

本项目根目录下有 `config/.env.example` 模板。我们把它复制成真正的 `.env`：

```bash
# Windows PowerShell（在项目根目录 d:/project/study/ai 下执行）
Copy-Item config/.env.example .env

# Mac / Linux
cp config/.env.example .env
```

用编辑器打开新生成的 `.env`，把你的真实 Key 填进去：

```ini
DEEPSEEK_API_KEY=sk-你刚才复制的真实密钥
DEEPSEEK_BASE_URL=https://api.deepseek.com
DEEPSEEK_MODEL=deepseek-chat
```

**`.env` 是什么？** 它是一个存放「环境变量」的文本文件。环境变量就是程序运行时能读到的配置项。把密钥放这里、而不是写进代码，有两个好处：

1. 代码可以分享/提交，密钥不会跟着泄露
2. 换密钥/换环境只改这一个文件，不用动代码

### 步骤 4：验证能读到密钥

我们装一个能读 `.env` 的小工具 `python-dotenv`：

```bash
pip install python-dotenv
```

> 这里先全局装一下、做一次性验证即可。第 01 章起我们会用**虚拟环境（venv）**规范管理依赖，之后每章用到的包（包括再装一次 `python-dotenv`）都会装在 venv 里，不再污染全局。

在项目根目录新建一个临时文件 `check_env.py`：

```python
# check_env.py —— 验证密钥配置是否成功
from dotenv import load_dotenv  # 读取 .env 的库
import os                        # 访问环境变量的标准库

load_dotenv()  # 把 .env 里的内容加载成环境变量

api_key = os.getenv("DEEPSEEK_API_KEY")  # 读取其中一项

if api_key and api_key.startswith("sk-"):
    # 只打印前后几位，避免把完整密钥暴露在屏幕/截图里
    print(f"✅ 读取成功：{api_key[:6]}...{api_key[-4:]}")
else:
    print("❌ 没读到密钥，请检查 .env 是否存在、DEEPSEEK_API_KEY 是否填对")
```

运行它：

```bash
python check_env.py
# 期望输出：✅ 读取成功：sk-abc...wxyz
```

看到 ✅ 就说明全局密钥配好了。**之后每一章的代码都会用 `load_dotenv()` + `os.getenv()` 这两行来拿密钥，不用再硬编码。**

验证完可以删掉 `check_env.py`（它只是用来确认环境）。

---

## 安全铁律（务必记住）

1. **`.env` 永不提交 Git**——本项目的 `.gitignore` 已经帮你忽略了它，但换项目时要自己加。
2. **Key 不发群、不贴论坛、不写进截图。** 一旦泄露，立刻去 DeepSeek 后台**删除该 Key 并新建一个**。
3. **代码里只写 `os.getenv("DEEPSEEK_API_KEY")`，永远不写 `sk-xxxx` 字面量。**

> 前端同学常犯的错：把 key 直接写进前端 JS。记住——**密钥只能待在后端**。第 04 章会讲为什么前端调大模型必须经过你自己的后端中转。

---

## 常见报错

| 现象                          | 原因                           | 解决                                                         |
| ----------------------------- | ------------------------------ | ------------------------------------------------------------ |
| `python 不是内部或外部命令` | 没勾选 Add to PATH             | 重装 Python 并勾选，或手动加环境变量                         |
| `pip 不是命令`              | 同上 / pip 未安装              | 试 `python -m pip --version`                               |
| `❌ 没读到密钥`             | `.env` 不在根目录 / 名字写错 | 确认文件名就是 `.env`（不是 `.env.txt`），且在项目根目录 |
| `.env` 显示为 `.env.txt`  | Windows 自动加了后缀           | 在资源管理器打开「显示文件扩展名」后改名                     |

---

## 小结

- API Key 是调用大模型的付费凭证，**泄露 = 花你的钱**
- 本课用 DeepSeek：国内直连、便宜、兼容 OpenAI
- 全局密钥放在根目录 `.env`，用 `load_dotenv()` + `os.getenv()` 读取，**只配一次，全课通用**
- 安全铁律：`.env` 不提交、Key 不外泄、代码不硬编码

## 下一章预告

环境就绪。下一章我们用 **JS 工程师能秒懂的方式**速通 Python 语法——你会发现它和 JavaScript 像得超乎想象。

**→ [第 01 章：Python 速成（写给 JS 工程师）](../01-python-for-js-devs/README.md)**
