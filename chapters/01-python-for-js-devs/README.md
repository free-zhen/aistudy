# 第 01 章 · Python 速成（写给 JS 工程师）

> 本章目标：用 JavaScript 做对照，30 分钟掌握后面够用的 Python 语法。
> 好消息：你已经会编程，这一章只是「换个语法」，不是「从头学」。

---

## 本章目标

- [ ] 看懂并写出 Python 的变量、函数、条件、循环
- [ ] 掌握 Python 的核心数据结构：list / dict（对应 JS 的数组 / 对象）
- [ ] 理解 Python 用**缩进**代替 `{}` 这件大事
- [ ] 会用虚拟环境 venv（相当于 Python 版的 `node_modules`）
- [ ] 会用 pip 装包（相当于 npm install）

---

## 核心概念：JS vs Python 速查表

最大的不同只有一个：**Python 用缩进表示代码块，没有 `{}`，语句结尾不写 `;`**。

```python
# Python
def greet(name):
    if name:
        print(f"Hello, {name}")
    else:
        print("Hello, stranger")
```

```javascript
// JavaScript
function greet(name) {
  if (name) {
    console.log(`Hello, ${name}`);
  } else {
    console.log("Hello, stranger");
  }
}
```

逐项对照：

| 概念 | JavaScript | Python |
|------|-----------|--------|
| 变量声明 | `let x = 1` | `x = 1`（不用关键字） |
| 常量 | `const x = 1` | `X = 1`（约定大写，无强制） |
| 字符串模板 | `` `Hi ${name}` `` | `f"Hi {name}"` |
| 函数 | `function f(a) {}` | `def f(a):` |
| 箭头函数 | `(a) => a + 1` | `lambda a: a + 1` |
| 数组 | `[1, 2, 3]` | `[1, 2, 3]`（叫 list） |
| 对象/字典 | `{ a: 1 }` | `{"a": 1}`（叫 dict，键要引号） |
| null | `null` / `undefined` | `None` |
| 布尔 | `true` / `false` | `True` / `False`（首字母大写） |
| 与或非 | `&& \|\| !` | `and or not` |
| 注释 | `// 注释` | `# 注释` |
| 相等比较 | `===` | `==`（Python 没有 `===`；`==` 比较值，`is` 比较身份） |
| 打印 | `console.log(x)` | `print(x)` |
| 数组长度 | `arr.length` | `len(arr)` |
| 遍历 | `for (const x of arr)` | `for x in arr:` |

---

## 动手实践

新建文件 `learn.py`，跟着敲（每段都能 `python learn.py` 运行看结果）。

### 1. 变量与字符串

```python
name = "前端工程师"
age = 3
# f-string —— 和 JS 模板字符串几乎一样，只是前面加个 f
print(f"我是{name}，转后端第 {age} 天")
```

### 2. 列表 list（≈ JS 数组）

```python
langs = ["js", "python"]
langs.append("sql")        # 对应 arr.push()
print(langs)               # ['js', 'python', 'sql']
print(len(langs))          # 3，对应 arr.length
print(langs[0])            # 'js'，索引访问一样

# 列表推导式：Python 的招牌写法，相当于 JS 的 map/filter
nums = [1, 2, 3, 4]
doubled = [n * 2 for n in nums]          # [2, 4, 6, 8] —— 类似 nums.map(n => n*2)
evens = [n for n in nums if n % 2 == 0]  # [2, 4]     —— 类似 nums.filter(...)
print(doubled, evens)
```

### 3. 字典 dict（≈ JS 对象）

```python
user = {"name": "Tom", "age": 18}   # 注意：键必须带引号
print(user["name"])                  # 取值用方括号，不是点号
user["email"] = "tom@x.com"          # 加字段
print(user.get("phone"))             # 取不存在的键返回 None，不会报错（推荐用 get）

# 遍历字典
for key, value in user.items():
    print(key, "=", value)
```

> ⚠️ JS 习惯写 `user.name`，Python 字典必须写 `user["name"]`。点号在 Python 里是给「对象的属性/方法」用的。

### 4. 函数

```python
def add(a, b=10):       # b=10 是默认参数，和 JS 一样
    return a + b

print(add(5))           # 15
print(add(5, 20))       # 25

# 关键字参数（调用时指定参数名，后面调 API 经常用）
print(add(a=1, b=2))    # 3
```

### 5. 条件与循环

```python
score = 85
if score >= 90:
    print("A")
elif score >= 60:       # 注意是 elif，不是 else if
    print("及格")
else:
    print("不及格")

# 计数循环：range(5) 产生 0,1,2,3,4
for i in range(5):
    print(i)

# while 一样
n = 3
while n > 0:
    print(n)
    n -= 1              # Python 没有 n--，用 n -= 1
```

### 6. 处理报错（后面调 API 会用）

```python
try:
    result = 10 / 0
except Exception as e:        # 对应 JS 的 catch (e)
    print("出错了:", e)
finally:
    print("无论如何都会执行")
```

---

## 虚拟环境 venv（≈ node_modules）

前端每个项目有自己的 `node_modules`，互不干扰。Python 用 **虚拟环境（venv）** 达到同样效果：每个项目独立装包，不污染全局。

**强烈建议：本课程从这一章起就用虚拟环境。**

```bash
# 1. 在项目根目录创建虚拟环境（生成一个 .venv 文件夹）
python -m venv .venv

# 2. 激活它
#    Windows PowerShell：
.\.venv\Scripts\Activate.ps1
#    Mac / Linux：
source .venv/bin/activate

# 激活成功后，命令行前面会出现 (.venv) 字样
```

> 如果 PowerShell 报「禁止运行脚本」，执行一次：
> `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`，输入 Y 回车，再激活。

激活后装包，就只装进这个项目：

```bash
pip install python-dotenv requests   # 一次装多个，空格隔开
```

记录依赖（相当于 package.json）：

```bash
pip freeze > requirements.txt   # 导出当前所有依赖
# 别人/换电脑后恢复依赖：
pip install -r requirements.txt
```

| 前端概念 | Python 对应 |
|---------|------------|
| `node_modules/` | `.venv/` |
| `npm install xxx` | `pip install xxx` |
| `package.json` 的 dependencies | `requirements.txt` |
| `npm install`（装全部） | `pip install -r requirements.txt` |

---

## 常见报错

| 现象 | 原因 | 解决 |
|------|------|------|
| `IndentationError` | 缩进不一致（混用空格/Tab） | 统一用 4 个空格；VS Code 会自动处理 |
| `KeyError: 'xxx'` | 取了字典里不存在的键 | 用 `dict.get("xxx")` 代替 `dict["xxx"]` |
| `NameError: name 'x' is not defined` | 变量没定义就用 | 检查拼写/是否在作用域内 |
| `pip` 装的包 import 不到 | 没激活 venv 就装了 / 装错环境 | 确认命令行前有 `(.venv)` |
| PowerShell 无法激活 venv | 脚本执行策略限制 | 见上面的 `Set-ExecutionPolicy` |

---

## 小结

- Python 和 JS 思路一致，主要差别是**缩进代替大括号、不写分号**
- list ≈ 数组，dict ≈ 对象（但键要引号、取值用 `[]`）
- 列表推导式 `[x for x in arr]` 是 Python 版的 map/filter
- venv = Python 的 node_modules，pip = npm，requirements.txt = package.json
- 别背语法，需要时回来查这张对照表即可

## 下一章预告

语法过关。下一章我们就用这点 Python，**第一次调通 DeepSeek 大模型**——你会亲手让 AI 回答你的问题，并理解 messages、token、流式输出这些核心概念。

**→ [第 02 章：第一次调用 LLM API](../02-first-llm-call/README.md)**
