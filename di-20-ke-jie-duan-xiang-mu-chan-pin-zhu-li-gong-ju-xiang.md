# 第 20 课｜阶段项目——产品助理工具箱 🏗



## 第二阶段：产品设计 × Python 实战

### 第 20 课｜阶段项目——产品助理工具箱 🏗

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～19 课（第二阶段全部）** 🛠️ **工具准备：Python 3.x、pip install streamlit anthropic pandas requests beautifulsoup4 feedparser**

***

### 课程导览

| 模块      | 内容                    | 时间    |
| ------- | --------------------- | ----- |
| 🔥 阶段回顾 | 第 11～19 课成果盘点         | 5 分钟  |
| 🎯 项目设计 | 多页工具箱整体架构             | 5 分钟  |
| 💻 代码实战 | 整合所有工具，统一入口           | 30 分钟 |
| 🔧 上线部署 | Streamlit 本地运行 + 分享链接 | 3 分钟  |
| 🎓 阶段小结 | 第二阶段收获 & 第三阶段预告       | 2 分钟  |

***

### 🔥 阶段回顾（5 分钟）

第二阶段（第 11～19 课）我们为每个工作场景写了一个独立脚本：

```
第 11 课：meeting-notes.py     会议纪要工厂
第 12 课：competitor-scan.py   竞品逆向分析
第 13 课：feedback-cleaner.py  需求清洗
第 14 课：prd-generator.py     PRD 文档工厂
第 15 课：prototype-demo.py    原型演示生成器
第 16 课：dashboard.py         业务数据看板
第 17 课：news-radar.py        情报雷达
第 18 课：db_query.py          数据库查询
第 19 课：eda_template.py      EDA 分析器
```

问题：9 个独立脚本，散落在不同文件夹，要用哪个还得想。

**本课目标**：把所有工具整合进一个 **Streamlit 多页应用**，一个界面，所有工具。

***

### 🎯 项目设计（5 分钟）

#### 应用架构

```
product-toolbox/
├── Home.py                      ← 首页（工具导航）
├── pages/
│   ├── 1_📝_会议纪要.py
│   ├── 2_🔍_竞品分析.py
│   ├── 3_📊_需求清洗.py
│   ├── 4_📄_PRD生成.py
│   ├── 5_🎨_原型演示.py
│   ├── 6_📈_数据看板.py
│   └── 7_📡_情报雷达.py
├── tools/                       ← 复用第 11-19 课的核心函数
│   ├── meeting_notes.py
│   ├── competitor.py
│   ├── feedback.py
│   ├── prd.py
│   └── eda.py
└── .streamlit/
    └── config.toml              ← 主题配置
```

#### 多页应用原理

Streamlit 自动把 `pages/` 目录下的文件转化为侧边栏导航：

```
文件名格式：[序号]_[emoji]_[显示名].py
例如：1_📝_会议纪要.py → 侧边栏显示 "📝 会议纪要"
```

***

### 💻 代码实战（30 分钟）

#### 第一步：主题配置

```toml
# .streamlit/config.toml
[theme]
base = "dark"
primaryColor = "#4ade80"
backgroundColor = "#0f0f23"
secondaryBackgroundColor = "#1a1a2e"
textColor = "#e0e0ff"
font = "sans serif"
```

#### 第二步：首页

```python
# Home.py
import streamlit as st

st.set_page_config(
    page_title="产品助理工具箱",
    page_icon="🛠️",
    layout="wide",
    initial_sidebar_state="expanded"
)

st.title("🛠️ 产品助理工具箱")
st.caption("Python × Claude AI — 第二阶段成果展示")

st.markdown("""
AI 驱动的产品工作流工具集，覆盖产品经理日常 80% 的重复性工作。

**一次封装，无限复用。**
""")

# 工具卡片网格
tools = [
    ("📝", "会议纪要", "原始文本 → 结构化纪要，10 秒完成", "第 11 课"),
    ("🔍", "竞品分析", "抓取竞品网站，AI 生成对比报告", "第 12 课"),
    ("📊", "需求清洗", "用户反馈 CSV → 可执行产品需求", "第 13 课"),
    ("📄", "PRD 生成", "一句话需求 → 完整 PRD 文档", "第 14 课"),
    ("🎨", "原型演示", "自然语言 → 可点击交互 Demo", "第 15 课"),
    ("📈", "数据看板", "业务 CSV → 图表 + AI 解读", "第 16 课"),
    ("📡", "情报雷达", "RSS 聚合 → 每日 AI 资讯摘要", "第 17 课"),
]

col1, col2, col3 = st.columns(3)
cols = [col1, col2, col3]

for i, (icon, name, desc, source) in enumerate(tools):
    with cols[i % 3]:
        st.markdown(f"""
<div style="background:#1a1a2e;border:1px solid #333366;border-radius:8px;
            padding:16px;margin-bottom:12px;">
  <div style="font-size:32px">{icon}</div>
  <div style="font-size:16px;font-weight:bold;color:#4ade80">{name}</div>
  <div style="font-size:13px;color:#8888aa;margin:6px 0">{desc}</div>
  <div style="font-size:11px;color:#555577">{source}</div>
</div>
""", unsafe_allow_html=True)

st.divider()
st.markdown("**使用方式**：点击左侧导航栏选择工具")
```

#### 第三步：各工具页面（以会议纪要为例）

```python
# pages/1_📝_会议纪要.py
import streamlit as st
import sys, os
sys.path.append(os.path.dirname(os.path.dirname(__file__)))

from tools.meeting_notes import generate_meeting_notes, save_notes

st.set_page_config(page_title="会议纪要", page_icon="📝")
st.title("📝 会议纪要工厂")
st.caption("原始文本 → 结构化 Markdown 纪要，10 秒完成")

# 输入区
input_method = st.radio("输入方式", ["文本粘贴", "上传 .txt 文件"], horizontal=True)

raw_text = ""
if input_method == "文本粘贴":
    raw_text = st.text_area(
        "粘贴会议录音转写或聊天记录",
        height=200,
        placeholder="例如：今天下午两点开了产品评审会，参加的有小红、小明……"
    )
else:
    uploaded = st.file_uploader("上传 .txt 文件", type="txt")
    if uploaded:
        raw_text = uploaded.read().decode("utf-8")
        st.text_area("文件内容预览", raw_text[:500] + ("..." if len(raw_text) > 500 else ""),
                     height=150, disabled=True)

if st.button("🚀 生成纪要", disabled=not raw_text, use_container_width=True):
    with st.spinner("⏳ Claude 正在处理..."):
        notes = generate_meeting_notes(raw_text)
    
    st.success("✅ 生成完成！")
    st.markdown(notes)
    
    # 下载按钮
    st.download_button(
        "💾 下载 Markdown",
        data=notes,
        file_name="会议纪要.md",
        mime="text/markdown"
    )
```

#### 第四步：工具函数层（tools/meeting\_notes.py）

```python
# tools/meeting_notes.py
# 从第 11 课提取核心逻辑，移除 sys.argv 依赖

import os
import anthropic
from datetime import datetime

SYSTEM_PROMPT = """你是一位专业的会议纪要助手。
请将用户提供的原始会议文本整理为规范 Markdown 格式的会议纪要。

输出必须包含以下结构：
## 会议纪要 — [主题] — [日期]

**时间**：[从文本中提取，无则写"待确认"]
**参与者**：[从文本中提取姓名]
**会议目的**：[一句话概括]

### 📋 讨论要点
- [要点，每条简短清晰]

### ✅ 关键决策
| 决策内容 | 负责人 | 截止日期 |
|---------|--------|---------|

### 🎯 行动项
- [ ] [具体任务] — @[负责人] — [截止日期]

### ❓ 待确认事项
- [需要跟进的问题]

只输出 Markdown，不加任何说明性文字。"""


def generate_meeting_notes(raw_text: str) -> str:
    client = anthropic.Anthropic()
    message = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=2048,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": f"请整理以下会议原始文本：\n\n{raw_text}"}]
    )
    return message.content[0].text


def save_notes(content: str, output_dir: str = "output") -> str:
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = os.path.join(output_dir, f"纪要_{timestamp}.md")
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(content)
    return filepath
```

#### 第五步：竞品分析页面

```python
# pages/2_🔍_竞品分析.py
import streamlit as st
import sys, os
sys.path.append(os.path.dirname(os.path.dirname(__file__)))

from tools.competitor import fetch_page, analyze_competitors

st.set_page_config(page_title="竞品分析", page_icon="🔍")
st.title("🔍 竞品逆向分析")
st.caption("输入竞品 URL，AI 生成结构化竞品报告")

url_input = st.text_area(
    "竞品 URL（每行一个，最多 5 个）",
    placeholder="https://todoist.com\nhttps://ticktick.com",
    height=120
)

if st.button("🚀 开始分析", disabled=not url_input, use_container_width=True):
    urls = [u.strip() for u in url_input.strip().splitlines() if u.strip()][:5]
    
    progress = st.progress(0, text="正在抓取竞品页面...")
    pages = []
    for i, url in enumerate(urls):
        with st.spinner(f"抓取 {url}..."):
            pages.append(fetch_page(url))
        progress.progress((i + 1) / (len(urls) + 1), text=f"已抓取 {i+1}/{len(urls)}")
    
    progress.progress(0.9, text="AI 分析中...")
    report = analyze_competitors(pages)
    progress.progress(1.0, text="完成！")
    
    st.success("✅ 分析完成！")
    st.markdown(report)
    st.download_button("💾 下载报告", data=report, file_name="竞品分析.md", mime="text/markdown")
```

#### 第六步：数据看板页面

```python
# pages/6_📈_数据看板.py
import streamlit as st
import pandas as pd
import sys, os
sys.path.append(os.path.dirname(os.path.dirname(__file__)))

from tools.eda import EDAAnalyzer

st.set_page_config(page_title="数据看板", page_icon="📈", layout="wide")
st.title("📈 数据看板 + AI 解读")

uploaded = st.file_uploader("上传 CSV 数据文件", type="csv")

if uploaded:
    df = pd.read_csv(uploaded)
    st.success(f"✅ 已加载：{len(df)} 行 × {len(df.columns)} 列")
    
    eda = EDAAnalyzer(df, uploaded.name)
    
    tab1, tab2, tab3, tab4 = st.tabs(["📊 数据摘要", "📈 分布图表", "🔗 相关性", "🤖 AI 洞察"])
    
    with tab1:
        s = eda.summary()
        col1, col2, col3 = st.columns(3)
        col1.metric("行数", s["行数"])
        col2.metric("列数", s["列数"])
        col3.metric("缺失值总数", s["总缺失值"])
        st.dataframe(df.describe().round(3), use_container_width=True)
    
    with tab2:
        st.pyplot(eda._make_dist_fig())
    
    with tab3:
        st.pyplot(eda._make_corr_fig())
    
    with tab4:
        if st.button("🔄 生成 AI 洞察"):
            with st.spinner("分析中..."):
                insights = eda.ai_insights()
            st.markdown(insights)
```

#### 运行方式

```bash
# 安装所有依赖
pip install streamlit anthropic pandas requests beautifulsoup4 feedparser sqlalchemy

# 启动工具箱
streamlit run Home.py

# 浏览器自动打开 http://localhost:8501
```

***

### 🔧 上线部署（3 分钟）

#### 本地局域网分享

```bash
# 同一 WiFi 下所有人可访问
streamlit run Home.py --server.address 0.0.0.0
# 把 http://你的IP:8501 分享给同事
```

#### Streamlit Cloud 公开部署

```bash
# 1. 把代码推送到 GitHub
git init && git add . && git commit -m "产品助理工具箱"
git remote add origin https://github.com/你的用户名/product-toolbox.git
git push -u origin main

# 2. 在 streamlit.io/cloud 绑定 GitHub 仓库
# 3. 设置 ANTHROPIC_API_KEY 环境变量
# 4. 点击 Deploy → 得到公开 URL

# 第 38 课会详细讲 vercel-deploy Skill 的一键部署流程
```

***

### 🎓 阶段小结（2 分钟）

```
第二阶段学习成果（第 11～20 课）

场景覆盖
├── 会议纪要工厂    → meeting-notes.py
├── 竞品逆向分析    → competitor-scan.py
├── 需求清洗        → feedback-cleaner.py
├── PRD 文档工厂    → prd-generator.py
├── 原型演示生成器  → prototype-demo.py
├── 数据看板        → dashboard.py
├── 情报雷达        → news-radar.py
├── 数据库查询      → db_query.py
└── EDA 分析器      → eda_template.py

整合产出
└── 产品助理工具箱  → 多页 Streamlit 应用

核心能力
├── Claude API 调用（多轮对话、Prompt 设计）
├── Pandas 数据处理（清洗、聚合、可视化）
├── 网络爬取（requests + BeautifulSoup）
├── 数据库操作（SQLite + Text-to-SQL）
└── Streamlit 应用开发（多页、session_state）
```

#### 第三阶段预告

```
到目前为止，每个工具还是"单独运行"的脚本。

第三阶段（第 21～27 课）：
  把这些脚本"一次封装"为 Skill——
  一个触发词，自动调用正确的工具，
  输出标准化结果，无限复用。

下节课：Skill 系统初探
  ~/.claude/commands/ 目录
  SKILL.md 格式
  触发词原理
```

***

### 🎯 课后作业

**练习 1（基础）**：在工具箱首页增加"最近使用"功能——用 `st.session_state` 记录用户访问过哪些工具，首页顶部显示最近 3 个。

**练习 2（进阶）**：在工具箱中增加"工具输出历史"侧边栏——每次生成的报告自动保存，用户可以在同一会话内查看历史结果。

**练习 3（挑战）**：把工具箱部署到 Streamlit Cloud，在 GitHub README 中贴上公开访问 URL，完成第二阶段的公开展示。

***

_第二阶段：产品设计 × Python 实战 · 第 20 课（阶段项目） / 共 10 课_
