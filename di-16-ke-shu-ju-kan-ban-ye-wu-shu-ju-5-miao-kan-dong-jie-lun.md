# 第 16 课｜数据看板：业务数据 → 5 秒看懂结论



## 第二阶段：产品设计 × Python 实战

###

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～15 课** 🛠️ **工具准备：Python 3.x、pip install pandas matplotlib seaborn streamlit anthropic**

***

### 课程导览

| 模块      | 内容                             | 时间    |
| ------- | ------------------------------ | ----- |
| 🔥 热身回顾 | 第 10 课仪表板 + 本课新增 AI 解读         | 3 分钟  |
| 🎯 场景拆解 | 数据看板痛点与 Pandas + AI 解法         | 5 分钟  |
| 📚 核心知识 | Pandas 聚合、Matplotlib 可视化、AI 解读 | 10 分钟 |
| 💻 代码实战 | 完整 dashboard.py 开发             | 20 分钟 |
| 🔧 封装产出 | 可复用看板模板                        | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告                   | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 10 课的 Streamlit 仪表板是**静态展示**——图表好看，但不告诉你"这个数字意味着什么"。

本课核心升级：

```
第 10 课：数据 → 图表（人看）
第 16 课：数据 → 图表 → AI 解读 → 结论（5 秒看懂）
```

不只是可视化，而是**自动生成管理层能直接用的结论摘要**。

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
周一早上，产品经理要给老板发周报：
  - 上周 DAU 12,340，较上周 -5.2%
  - 留存率 D1: 42%, D7: 18%, D30: 8%
  - 核心功能转化漏斗：注册→激活→付费 = 100%→45%→12%

传统方式：从数据平台导出数据 → Excel 制图 → 手写分析 → 1 小时
本课方式：CSV 放进脚本 → 自动图表 + AI 摘要 → 10 分钟出周报
```

#### 解决思路

```
CSV 业务数据
      │
      ▼
Pandas 计算核心指标（同比/环比/增长率）
      │
      ├─── Matplotlib/Seaborn 生成图表
      │
      └─── Claude API 生成文字结论
                │
                ▼
        Streamlit 看板（图表 + 结论并排）
```

***

### 📚 核心知识（10 分钟）

#### 1. Pandas 计算增长率

```python
import pandas as pd

df = pd.read_csv("metrics.csv")  # 列：date, dau, revenue, retention_d1

# 环比增长率
df["dau_wow"] = df["dau"].pct_change(periods=7)  # 与 7 天前相比

# 滚动平均（平滑曲线）
df["dau_7d_avg"] = df["dau"].rolling(7).mean()

# 同比（与 30 天前比）
df["dau_mom"] = df["dau"].pct_change(periods=30)

# 最新值
latest_dau = df["dau"].iloc[-1]
latest_wow = df["dau_wow"].iloc[-1]
print(f"DAU: {latest_dau:,}（较上周 {latest_wow:+.1%}）")
```

#### 2. Matplotlib 快速绘图

```python
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rcParams["font.family"] = "PingFang SC"  # macOS 中文字体

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

# 折线图：DAU 趋势
axes[0].plot(df["date"], df["dau_7d_avg"], color="#4ade80", linewidth=2)
axes[0].fill_between(df["date"], df["dau_7d_avg"], alpha=0.1, color="#4ade80")
axes[0].set_title("DAU 7日均线")

# 柱状图：日收入
axes[1].bar(df["date"].tail(14), df["revenue"].tail(14), color="#38bdf8")
axes[1].set_title("近 14 天日收入")

plt.tight_layout()
plt.savefig("charts/metrics.png", dpi=150, bbox_inches="tight")
```

#### 3. 把数据摘要传给 Claude 生成结论

```python
def build_metrics_summary(df: pd.DataFrame) -> str:
    latest = df.iloc[-1]
    prev_week = df.iloc[-8] if len(df) >= 8 else df.iloc[0]
    
    return f"""最新业务数据摘要（{latest['date']}）：

核心指标：
- DAU：{latest['dau']:,.0f}（较上周 {(latest['dau']/prev_week['dau']-1):+.1%}）
- 日收入：¥{latest['revenue']:,.0f}
- D1 留存：{latest['retention_d1']:.1%}
- D7 留存：{latest['retention_d7']:.1%}

近 7 天趋势：
{df.tail(7)[['date','dau','revenue']].to_string(index=False)}"""
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
data-dashboard/
├── dashboard.py          ← 主程序（Streamlit App）
├── samples/
│   └── metrics.csv       ← 测试业务数据
└── charts/               ← 自动创建，存放图表文件
```

#### 测试数据：samples/metrics.csv

```csv
date,dau,revenue,retention_d1,retention_d7,retention_d30
2026-05-01,11200,45000,0.42,0.18,0.08
2026-05-02,11450,47200,0.43,0.19,0.08
2026-05-03,10890,42100,0.41,0.17,0.07
2026-05-04,10560,38900,0.40,0.16,0.07
2026-05-05,9870,35200,0.38,0.15,0.06
2026-05-06,10230,39800,0.39,0.16,0.07
2026-05-07,11100,44500,0.42,0.18,0.08
2026-05-08,12340,51200,0.44,0.19,0.09
2026-05-09,12580,53400,0.45,0.20,0.09
2026-05-10,13100,56800,0.46,0.21,0.10
```

#### 完整代码：dashboard.py

```python
#!/usr/bin/env python3
"""
业务数据看板 | dashboard.py
运行：streamlit run dashboard.py
依赖：pip install pandas matplotlib seaborn streamlit anthropic
"""
import os
import io
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib
import seaborn as sns
import streamlit as st
import anthropic
from datetime import datetime

# 中文字体
matplotlib.rcParams["font.sans-serif"] = ["PingFang SC", "Heiti TC", "Arial Unicode MS"]
matplotlib.rcParams["axes.unicode_minus"] = False

SYSTEM_PROMPT = """你是一位数据分析师，擅长从业务数据中提炼管理层洞察。
请根据数据摘要生成简洁的分析结论，格式：

## 📊 本周数据摘要
[2-3 句话总结整体情况，使用具体数字]

## 🔴 需要关注
- [问题点 + 数据支撑 + 建议行动]

## 🟢 表现亮眼
- [亮点 + 数据支撑]

## 🎯 下周重点
- [建议重点关注的 2-3 个方向]

语言简洁，管理层 30 秒内看完。只输出 Markdown，不加解释。"""


# ─────────────────────────────────────────────
#  1. 数据加载与计算
# ─────────────────────────────────────────────
@st.cache_data
def load_data(filepath: str) -> pd.DataFrame:
    df = pd.read_csv(filepath)
    df["date"] = pd.to_datetime(df["date"])
    df = df.sort_values("date").reset_index(drop=True)
    
    # 计算环比
    df["dau_wow"] = df["dau"].pct_change(7).fillna(0)
    df["revenue_wow"] = df["revenue"].pct_change(7).fillna(0)
    df["dau_7d_avg"] = df["dau"].rolling(7, min_periods=1).mean()
    return df


def compute_summary(df: pd.DataFrame) -> dict:
    latest = df.iloc[-1]
    prev_week = df.iloc[-8] if len(df) >= 8 else df.iloc[0]
    return {
        "date": latest["date"].strftime("%Y-%m-%d"),
        "dau": int(latest["dau"]),
        "dau_wow": latest["dau_wow"],
        "revenue": int(latest["revenue"]),
        "revenue_wow": latest["revenue_wow"],
        "retention_d1": latest["retention_d1"],
        "retention_d7": latest["retention_d7"],
        "retention_d30": latest.get("retention_d30", None),
    }


# ─────────────────────────────────────────────
#  2. 图表生成（返回 BytesIO）
# ─────────────────────────────────────────────
def make_trend_chart(df: pd.DataFrame) -> bytes:
    fig, axes = plt.subplots(1, 2, figsize=(12, 3.5))
    fig.patch.set_facecolor("#0f0f23")
    
    for ax in axes:
        ax.set_facecolor("#1a1a2e")
        ax.tick_params(colors="#aaaaaa", labelsize=9)
        for spine in ax.spines.values():
            spine.set_edgecolor("#333366")
    
    # DAU 趋势
    dates = df["date"]
    axes[0].plot(dates, df["dau_7d_avg"], color="#4ade80", linewidth=2, label="7日均线")
    axes[0].plot(dates, df["dau"], color="#4ade80", linewidth=1, alpha=0.4, linestyle="--")
    axes[0].fill_between(dates, df["dau_7d_avg"], alpha=0.15, color="#4ade80")
    axes[0].set_title("DAU 趋势", color="white", fontsize=11)
    axes[0].legend(facecolor="#1a1a2e", labelcolor="white", fontsize=8)
    
    # 日收入柱状图
    last_14 = df.tail(14)
    bar_colors = ["#38bdf8" if v >= 0 else "#f87171"
                  for v in last_14["revenue_wow"]]
    axes[1].bar(range(len(last_14)), last_14["revenue"], color=bar_colors)
    axes[1].set_xticks(range(len(last_14)))
    axes[1].set_xticklabels(
        [d.strftime("%m/%d") for d in last_14["date"]],
        rotation=45, ha="right", fontsize=7
    )
    axes[1].set_title("近 14 天日收入", color="white", fontsize=11)
    
    plt.tight_layout(pad=1.5)
    buf = io.BytesIO()
    plt.savefig(buf, format="png", dpi=130, bbox_inches="tight",
                facecolor=fig.get_facecolor())
    plt.close(fig)
    buf.seek(0)
    return buf.read()


def make_retention_chart(df: pd.DataFrame) -> bytes:
    fig, ax = plt.subplots(figsize=(7, 3.5))
    fig.patch.set_facecolor("#0f0f23")
    ax.set_facecolor("#1a1a2e")
    
    last_14 = df.tail(14)
    ax.plot(last_14["date"], last_14["retention_d1"],
            label="D1 留存", color="#a78bfa", linewidth=2, marker="o", markersize=4)
    ax.plot(last_14["date"], last_14["retention_d7"],
            label="D7 留存", color="#f472b6", linewidth=2, marker="s", markersize=4)
    if "retention_d30" in last_14.columns:
        ax.plot(last_14["date"], last_14["retention_d30"],
                label="D30 留存", color="#fb923c", linewidth=2, marker="^", markersize=4)
    
    ax.yaxis.set_major_formatter(matplotlib.ticker.PercentFormatter(1.0))
    ax.set_title("留存率趋势", color="white")
    ax.tick_params(colors="#aaaaaa")
    ax.legend(facecolor="#1a1a2e", labelcolor="white")
    for spine in ax.spines.values():
        spine.set_edgecolor("#333366")
    
    plt.tight_layout()
    buf = io.BytesIO()
    plt.savefig(buf, format="png", dpi=130, bbox_inches="tight",
                facecolor=fig.get_facecolor())
    plt.close(fig)
    buf.seek(0)
    return buf.read()


# ─────────────────────────────────────────────
#  3. Claude AI 结论生成
# ─────────────────────────────────────────────
def generate_insights(df: pd.DataFrame, summary: dict) -> str:
    tail_text = df.tail(7)[["date", "dau", "revenue",
                             "retention_d1", "retention_d7"]].to_string(index=False)
    context = f"""最新数据（{summary['date']}）：
- DAU：{summary['dau']:,}（较上周 {summary['dau_wow']:+.1%}）
- 日收入：¥{summary['revenue']:,}（较上周 {summary['revenue_wow']:+.1%}）
- D1 留存：{summary['retention_d1']:.1%}
- D7 留存：{summary['retention_d7']:.1%}

近 7 天明细：
{tail_text}"""
    
    client = anthropic.Anthropic()
    resp = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=800,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": context}]
    )
    return resp.content[0].text


# ─────────────────────────────────────────────
#  4. 主界面
# ─────────────────────────────────────────────
st.set_page_config(page_title="业务数据看板", page_icon="📊", layout="wide")

# 自定义黑色主题
st.markdown("""
<style>
.stApp { background-color: #0f0f23; color: #e0e0ff; }
.metric-card {
    background: #1a1a2e; border-radius: 8px;
    padding: 16px; text-align: center;
    border: 1px solid #333366;
}
.metric-value { font-size: 28px; font-weight: bold; }
.metric-label { font-size: 12px; color: #8888aa; }
.positive { color: #4ade80; }
.negative { color: #f87171; }
</style>
""", unsafe_allow_html=True)

st.title("📊 业务数据看板")

# 文件上传 or 使用样本
with st.sidebar:
    st.header("📂 数据源")
    uploaded = st.file_uploader("上传 CSV（列：date, dau, revenue, retention_d1, retention_d7）",
                                 type="csv")
    use_sample = st.checkbox("使用样本数据", value=True)
    ai_insights = st.checkbox("启用 AI 数据解读", value=True)

# 加载数据
if uploaded:
    df = load_data(uploaded)
elif use_sample and os.path.exists("samples/metrics.csv"):
    df = load_data("samples/metrics.csv")
else:
    st.info("👆 请上传 CSV 文件或勾选"使用样本数据"")
    st.stop()

summary = compute_summary(df)

# 顶部核心指标卡片
st.subheader(f"📅 截至 {summary['date']} 的最新数据")
c1, c2, c3, c4 = st.columns(4)

def metric_color(val: float) -> str:
    return "positive" if val >= 0 else "negative"

for col, label, value, change in [
    (c1, "DAU", f"{summary['dau']:,}", summary["dau_wow"]),
    (c2, "日收入", f"¥{summary['revenue']:,}", summary["revenue_wow"]),
    (c3, "D1 留存", f"{summary['retention_d1']:.1%}", 0),
    (c4, "D7 留存", f"{summary['retention_d7']:.1%}", 0),
]:
    with col:
        arrow = "↑" if change > 0 else ("↓" if change < 0 else "")
        change_str = f"{change:+.1%}" if change != 0 else ""
        col_class = metric_color(change)
        col.markdown(f"""
<div class="metric-card">
  <div class="metric-label">{label}</div>
  <div class="metric-value">{value}</div>
  <div class="{col_class}">{arrow} {change_str}</div>
</div>""", unsafe_allow_html=True)

st.divider()

# 图表 + AI 解读并排
chart_col, insight_col = st.columns([3, 2])

with chart_col:
    st.subheader("📈 趋势图表")
    trend_img = make_trend_chart(df)
    st.image(trend_img, use_container_width=True)
    
    retention_img = make_retention_chart(df)
    st.image(retention_img, use_container_width=True)

with insight_col:
    st.subheader("🤖 AI 数据解读")
    if ai_insights:
        if "insights" not in st.session_state:
            st.session_state.insights = ""
        
        if st.button("🔄 生成/刷新解读", use_container_width=True):
            with st.spinner("AI 分析中..."):
                st.session_state.insights = generate_insights(df, summary)
        
        if st.session_state.insights:
            st.markdown(st.session_state.insights)
        else:
            st.info("点击上方按钮生成 AI 解读")
    else:
        st.info("在左侧勾选"启用 AI 数据解读"")
```

#### 运行方式

```bash
streamlit run dashboard.py
```

***

### 🔧 封装产出（4 分钟）

```
data-dashboard/
├── dashboard.py          ✅ 可复用看板模板
├── samples/
│   └── metrics.csv       ✅ 测试数据
└── charts/               ✅ 图表输出目录
```

**复用场景**：把 `metrics.csv` 替换成任意业务 CSV，换一下列名映射，10 分钟出定制看板。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

Pandas 指标计算
├── pct_change(n) 环比增长率
├── rolling(7).mean() 滚动均值
└── to_string() 转文本传给 AI

Matplotlib 可视化
├── subplots(1, 2) 多图布局
├── fill_between() 面积图
├── BytesIO 内存图片传给 Streamlit
└── rcParams 中文字体设置

Streamlit 看板
├── @st.cache_data 缓存数据加载
├── st.columns(n) 多列布局
├── st.image(bytes) 展示图表
└── 文件上传 + 样本数据双模式
```

***

### 🎯 课后作业

**练习 1（基础）**：在看板顶部增加日期范围选择器（`st.date_input`），让用户选择分析区间。

**练习 2（进阶）**：支持导出功能——把图表和 AI 解读合并为一份 PDF 周报，用 `reportlab` 或 `weasyprint` 生成。

**练习 3（挑战）**：接入真实数据源——用 SQLite 替代 CSV，支持 `pandas.read_sql()` 查询，并在第 18 课中深入学习数据仓库接入。

***

_第二阶段：产品设计 × Python 实战 · 第 16 课 / 共 10 课_
