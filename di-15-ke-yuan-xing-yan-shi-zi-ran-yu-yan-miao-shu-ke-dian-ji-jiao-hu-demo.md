# 第 15 课｜原型演示：自然语言描述 → 可点击交互 Demo



### 第 15 课｜原型演示：自然语言描述 → 可点击交互 Demo

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～14 课（含第 10 课 Streamlit 基础）** 🛠️ **工具准备：Python 3.x、pip install streamlit anthropic**

***

### 课程导览

| 模块      | 内容                               | 时间    |
| ------- | -------------------------------- | ----- |
| 🔥 热身回顾 | 第 10 课 Streamlit + 第 14 课 PRD 生成 | 3 分钟  |
| 🎯 场景拆解 | 原型演示痛点与 Streamlit + AI 解法        | 5 分钟  |
| 📚 核心知识 | st.session\_state、多轮对话状态管理       | 10 分钟 |
| 💻 代码实战 | 完整 prototype-demo.py 开发          | 20 分钟 |
| 🔧 封装产出 | 可分享 Streamlit App URL            | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告                     | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 10 课我们用 Streamlit 做了数据仪表板——**单向展示**数据。\
第 14 课我们用 PRD 工厂描述功能——**静态文档**。

这节课把两者结合：

```
PRD 描述的功能  →  Streamlit 可交互 Demo  →  AI 对话驱动界面生成
```

产品经理不写一行 HTML/CSS，只需用自然语言描述 UI 交互，AI 实时生成可点击的界面演示。

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
需求评审会上，产品经理需要展示"登录改造"方案：
  - 传统方式：用 Figma 画原型，调整每一个按钮位置 → 3 小时
  - 本课方式：输入"展示微信扫码登录流程，3 个步骤"
              → AI 生成 Streamlit 交互流程演示 → 10 分钟

不是生产级代码，而是能点击、能演示的"概念验证 Demo"
```

#### 解决思路

```
用户在 Streamlit 界面输入 UI 需求描述
            │
            ▼
Claude API 生成 Streamlit 页面代码（字符串）
            │
            ▼
exec() 在当前进程动态执行代码
            │
            ▼
界面实时更新，展示交互效果
            │
            ▼
可保存为独立 demo_xxx.py 文件
```

***

### 📚 核心知识（10 分钟）

#### 1. st.session\_state：跨交互保持状态

Streamlit 每次用户操作都会重新运行整个脚本。`session_state` 是**持久化存储**：

```python
import streamlit as st

# 初始化（只在第一次运行时执行）
if "messages" not in st.session_state:
    st.session_state.messages = []

if "generated_code" not in st.session_state:
    st.session_state.generated_code = ""

# 存储
st.session_state.messages.append({"role": "user", "content": "你好"})

# 读取
for msg in st.session_state.messages:
    print(msg["content"])
```

#### 2. 对话历史渲染

```python
# 渲染所有历史消息
for msg in st.session_state.messages:
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])

# 获取用户输入
if prompt := st.chat_input("描述你想要的 UI..."):
    # 添加到历史
    st.session_state.messages.append({"role": "user", "content": prompt})
    
    # 显示用户消息
    with st.chat_message("user"):
        st.markdown(prompt)
    
    # 生成 AI 回复
    with st.chat_message("assistant"):
        with st.spinner("生成中..."):
            reply = call_claude(st.session_state.messages)
            st.markdown(reply)
    
    st.session_state.messages.append({"role": "assistant", "content": reply})
```

#### 3. exec() 动态执行代码

```python
# 危险！生产环境中不要用
# 但在本地原型演示中，用来即时渲染 AI 生成的 Streamlit 代码

generated_code = '''
import streamlit as st
st.title("微信扫码登录")
st.image("https://placehold.co/200x200?text=QR+Code")
st.caption("请用微信扫描二维码")
'''

try:
    exec(generated_code)
except Exception as e:
    st.error(f"代码执行错误：{e}")
```

> **安全说明**：`exec()` 执行任意代码存在安全风险。本课仅在本地开发环境使用，绝不部署到公网。生产环境应使用沙盒或独立进程。

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
prototype-demo/
├── prototype-demo.py    ← 主程序（Streamlit App）
└── saved_demos/         ← 保存生成的 Demo 代码
```

#### 完整代码：prototype-demo.py

````python
#!/usr/bin/env python3
"""
原型演示生成器 | prototype-demo.py
运行：streamlit run prototype-demo.py
依赖：pip install streamlit anthropic
"""
import os
import re
from datetime import datetime
import streamlit as st
import anthropic


# ─────────────────────────────────────────────
#  配置
# ─────────────────────────────────────────────
st.set_page_config(
    page_title="原型演示生成器",
    page_icon="🎨",
    layout="wide"
)

SYSTEM_PROMPT = """你是一位专业的 Streamlit UI 原型工程师。
用户会用自然语言描述 UI 原型需求，你需要生成对应的 Streamlit Python 代码。

规则：
1. 只输出 Python 代码，不要任何解释
2. 代码必须是完整可运行的 Streamlit 片段（不包含 import streamlit as st，因为已经导入）
3. 使用 Streamlit 原生组件实现交互：按钮、表单、选项卡、进度条等
4. 不要使用外部图片 URL，用 st.empty() 或 emoji 代替
5. 代码简洁，专注于演示核心交互流程
6. 不要在代码中使用 st.set_page_config()（已在主程序设置）
7. 用中文标签和内容

代码格式：
```python
# 你的代码
````

"""

## ─────────────────────────────────────────────

## Claude API 调用

## ─────────────────────────────────────────────

def call\_claude(messages: list\[dict]) -> str: client = anthropic.Anthropic() response = client.messages.create( model="claude-opus-4-7", max\_tokens=2000, system=SYSTEM\_PROMPT, messages=messages ) return response.content\[0].text

def extract\_code(text: str) -> str: """从 AI 回复中提取 Python 代码块""" pattern = r"`python\n(.*?)`" match = re.search(pattern, text, re.DOTALL) if match: return match.group(1).strip() # 如果没有代码块标记，尝试直接返回 return text.strip()

def save\_demo(code: str, name: str = "demo") -> str: """保存 Demo 代码为独立文件""" os.makedirs("saved\_demos", exist\_ok=True) timestamp = datetime.now().strftime("%Y%m%d\_%H%M%S") safe\_name = name.replace(" ", "_").replace("/", "_")\[:20] filepath = f"saved\_demos/{safe\_name}\_{timestamp}.py" header = ( "import streamlit as st\n" "st.set\_page\_config(page\_title='Demo', layout='wide')\n\n" ) with open(filepath, "w", encoding="utf-8") as f: f.write(header + code) return filepath

## ─────────────────────────────────────────────

## 主界面

## ─────────────────────────────────────────────

st.title("🎨 原型演示生成器") st.caption("用自然语言描述你的 UI，AI 即时生成可交互的 Streamlit 原型")

## 初始化 session\_state

if "messages" not in st.session\_state: st.session\_state.messages = \[] if "current\_code" not in st.session\_state: st.session\_state.current\_code = "" if "demo\_name" not in st.session\_state: st.session\_state.demo\_name = "我的原型"

## 左右分栏：左边对话，右边预览

col\_chat, col\_preview = st.columns(\[1, 1], gap="large")

with col\_chat: st.subheader("💬 需求对话")

```
# 渲染历史消息
chat_container = st.container(height=400)
with chat_container:
    if not st.session_state.messages:
        st.info("👋 试试输入：「展示一个用户登录表单，有用户名密码输入框和登录按钮」")
    for msg in st.session_state.messages:
        with st.chat_message(msg["role"]):
            if msg["role"] == "assistant":
                # AI 回复只显示摘要，不显示完整代码
                st.markdown("✅ 已生成原型代码，请查看右侧预览")
            else:
                st.markdown(msg["content"])

# 输入框
prompt = st.chat_input("描述你想要的 UI 原型...")

if prompt:
    # 添加用户消息
    st.session_state.messages.append({"role": "user", "content": prompt})
    
    # 调用 Claude
    with st.spinner("⏳ AI 正在生成原型代码..."):
        reply = call_claude([
            m for m in st.session_state.messages
            if m["role"] == "user"  # 只传用户消息历史，节省 token
        ])
    
    # 提取代码
    code = extract_code(reply)
    st.session_state.current_code = code
    st.session_state.messages.append({"role": "assistant", "content": reply})
    
    # 自动命名
    if len(st.session_state.messages) == 2:
        st.session_state.demo_name = prompt[:20]
    
    st.rerun()
```

with col\_preview: st.subheader("🖥️ 实时预览")

```
if st.session_state.current_code:
    # 操作按钮行
    btn_col1, btn_col2, btn_col3 = st.columns(3)
    with btn_col1:
        if st.button("🔄 重新生成", use_container_width=True):
            if st.session_state.messages:
                last_user = next(
                    (m["content"] for m in reversed(st.session_state.messages)
                     if m["role"] == "user"), None
                )
                if last_user:
                    with st.spinner("重新生成中..."):
                        reply = call_claude([{"role": "user", "content": last_user}])
                        st.session_state.current_code = extract_code(reply)
                    st.rerun()
    with btn_col2:
        if st.button("💾 保存 Demo", use_container_width=True):
            path = save_demo(st.session_state.current_code, st.session_state.demo_name)
            st.success(f"已保存：{path}")
    with btn_col3:
        if st.button("🗑️ 清空", use_container_width=True):
            st.session_state.messages = []
            st.session_state.current_code = ""
            st.rerun()
    
    # 预览区域
    st.divider()
    preview_tab, code_tab = st.tabs(["📱 预览效果", "📄 查看代码"])
    
    with preview_tab:
        try:
            exec(st.session_state.current_code)
        except Exception as e:
            st.error(f"预览错误：{e}")
            st.code(st.session_state.current_code, language="python")
    
    with code_tab:
        st.code(st.session_state.current_code, language="python")
else:
    st.empty()
    st.markdown("""
    **🚀 快速开始示例：**
    
    在左侧输入框中试试：
    - 「展示微信扫码登录的三步流程」
    - 「创建一个任务管理看板，有待办/进行中/已完成三列」
    - 「设计一个数据看板，有三张卡片显示日活、留存、收入」
    - 「展示一个用户注册表单，有表单验证」
    """)
```

````

### 运行方式

```bash
streamlit run prototype-demo.py
````

浏览器自动打开 `http://localhost:8501`。

**演示对话示例**：

```
你：展示微信扫码登录的三步流程，有进度指示

AI 生成：
  步骤 1/3 — 打开微信 → 扫一扫
  [进度条 33%]
  [QR Code 占位符]
  [等待扫码...] 按钮

  步骤 2/3 — 微信内确认登录
  [进度条 66%]
  [手机图标 + "请在手机上点击确认"]

  步骤 3/3 — 登录成功
  [进度条 100%]
  [✅ 登录成功！] 绿色提示
  [进入应用] 按钮
```

***

### 🔧 封装产出（4 分钟）

本课交付物：

```
prototype-demo/
├── prototype-demo.py    ✅ 可运行的 Streamlit 原型生成器
└── saved_demos/
    └── *.py             ✅ 可独立运行的 Demo 文件
```

**部署为可分享 URL**：

```bash
# 方法 1：本地网络分享（同一 WiFi 下所有人可访问）
streamlit run prototype-demo.py --server.address 0.0.0.0

# 方法 2：Streamlit Cloud（需要 GitHub 仓库，第 38 课详细讲）
# 上传代码到 GitHub → streamlit.io/cloud 一键部署 → 得到公开 URL
```

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

st.session_state
├── 跨交互保持状态（消息历史、生成代码）
├── if "key" not in st.session_state: 初始化
└── st.rerun() 触发重新渲染

对话界面
├── st.chat_message(role) 消息气泡
├── st.chat_input() 输入框
└── st.container(height=n) 限高滚动区域

动态代码执行
├── extract_code() 正则提取代码块
├── exec(code) 动态执行（仅本地）
└── try/except 捕获执行错误

布局
├── st.columns([1,1]) 左右分栏
└── st.tabs(["预览", "代码"]) 标签页
```

***

### 🎯 课后作业

**练习 1（基础）**：在预览区增加"导出为 PNG"功能，用 `st.download_button` 提供截图下载（可以用 Pillow 库截取 `st.container` 的内容）。

**练习 2（进阶）**：支持"修改需求"的连续对话——当用户说"把登录按钮改成绿色"时，AI 在现有代码基础上修改，而不是重新生成。

**练习 3（挑战）**：把生成的 Demo 代码包装成真实可运行的 Streamlit 多页应用，每个功能页一个文件，自动生成导航菜单。

***

_第二阶段：产品设计 × Python 实战 · 第 15 课 / 共 10 课_
