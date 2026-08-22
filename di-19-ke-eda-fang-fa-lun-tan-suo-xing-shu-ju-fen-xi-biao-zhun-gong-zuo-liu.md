# 第 19 课｜EDA 方法论：探索性数据分析标准工作流



## 第二阶段：产品设计 × Python 实战

### 第 19 课｜EDA 方法论：探索性数据分析标准工作流

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～18 课** 🛠️ **工具准备：Python 3.x、pip install pandas matplotlib seaborn jupyter anthropic**

***

### 课程导览

| 模块      | 内容                             | 时间    |
| ------- | ------------------------------ | ----- |
| 🔥 热身回顾 | 第 13/16/18 课数据技能汇总             | 3 分钟  |
| 🎯 场景拆解 | EDA 是什么，产品人为什么要会               | 5 分钟  |
| 📚 核心知识 | 5 步 EDA 工作流 + Pandas Profiling | 10 分钟 |
| 💻 代码实战 | 可复用 EDA 笔记本模板                  | 20 分钟 |
| 🔧 封装产出 | EDA 笔记本模板 + AI 解读函数            | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告（第 20 课大项目）        | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

过去几课我们积累了大量数据处理能力：

```
第 13 课：清洗用户反馈（词频统计、过滤）
第 16 课：可视化业务指标（折线图、柱状图）
第 18 课：从数据库查询数据（SQL + Pandas）
```

但这些都是**针对特定问题**的点状技能。

本课教你一套**通用分析框架**——EDA（探索性数据分析），面对任何新数据集，都能系统性地摸清其规律。

***

### 🎯 场景拆解（5 分钟）

#### 什么是 EDA？

```
EDA = Exploratory Data Analysis（探索性数据分析）

不是为了得出某个预设结论，
而是系统性地"审问"数据：
  - 数据质量如何？（缺失值、异常值）
  - 数据分布是什么样的？（集中趋势、离散程度）
  - 变量之间有什么关系？（相关性）
  - 有什么异常模式？（需要解释的峰值、断层）
```

#### 产品人为什么要 EDA？

```
场景：你拿到一份新的用户行为数据集
  → 没有 EDA：直接建模/出报告，可能基于脏数据得出错误结论
  → 有 EDA：先摸清数据，再分析，结论更可信

EDA 是数据分析的"体检"环节，省不得。
```

***

### 📚 核心知识（10 分钟）

#### 5 步 EDA 标准工作流

```
Step 1: 数据摘要         → 行数、列数、类型、基本统计
Step 2: 缺失值分析       → 哪些列缺失，缺失比例
Step 3: 分布分析         → 数值列直方图，类别列柱状图
Step 4: 相关性分析       → 热力图，找高相关变量对
Step 5: 异常值检测       → IQR 方法，箱线图
```

#### 1. 数据摘要

```python
import pandas as pd

df = pd.read_csv("data.csv")

print(f"形状：{df.shape}")                    # (行数, 列数)
print(df.dtypes)                              # 每列类型
print(df.describe())                          # 数值列统计（均值/标准差/四分位数）
print(df.describe(include="object"))          # 字符串列统计（唯一值数、最高频值）
```

#### 2. 缺失值分析

```python
missing = df.isnull().sum()
missing_pct = (missing / len(df) * 100).round(2)
missing_df = pd.DataFrame({"缺失数": missing, "缺失率%": missing_pct})
missing_df = missing_df[missing_df["缺失数"] > 0].sort_values("缺失率%", ascending=False)
print(missing_df)
```

#### 3. 相关性矩阵

```python
import seaborn as sns
import matplotlib.pyplot as plt

numeric_cols = df.select_dtypes(include="number").columns
corr = df[numeric_cols].corr()

plt.figure(figsize=(10, 8))
sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm",
            center=0, square=True)
plt.title("相关性热力图")
plt.tight_layout()
plt.show()
```

#### 4. IQR 异常值检测

```python
def detect_outliers(series: pd.Series) -> dict:
    Q1 = series.quantile(0.25)
    Q3 = series.quantile(0.75)
    IQR = Q3 - Q1
    lower = Q1 - 1.5 * IQR
    upper = Q3 + 1.5 * IQR
    outliers = series[(series < lower) | (series > upper)]
    return {
        "count": len(outliers),
        "pct": len(outliers) / len(series) * 100,
        "lower_bound": lower,
        "upper_bound": upper,
    }
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
eda-notebook/
├── eda_template.py        ← 可复用 EDA 函数库
├── eda_notebook.ipynb     ← Jupyter 笔记本模板
└── samples/
    └── ecommerce.csv      ← 示例电商数据集
```

#### 核心函数库：eda\_template.py

```python
#!/usr/bin/env python3
"""
EDA 分析模板 | eda_template.py
可复用的探索性数据分析函数库

用法：
  from eda_template import EDAAnalyzer
  eda = EDAAnalyzer(df)
  eda.full_report()
"""
import io
import re
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib
import seaborn as sns
import anthropic

matplotlib.rcParams["font.sans-serif"] = ["PingFang SC", "Heiti TC", "Arial Unicode MS"]
matplotlib.rcParams["axes.unicode_minus"] = False
sns.set_theme(style="darkgrid", palette="muted")


class EDAAnalyzer:
    """通用 EDA 分析器"""

    def __init__(self, df: pd.DataFrame, name: str = "数据集"):
        self.df = df.copy()
        self.name = name
        self.numeric_cols = df.select_dtypes(include="number").columns.tolist()
        self.cat_cols = df.select_dtypes(include=["object", "category"]).columns.tolist()
        self._insights_cache: dict[str, str] = {}

    # ── 1. 基础摘要 ──────────────────────────────
    def summary(self) -> dict:
        """返回数据集基础信息"""
        return {
            "行数": len(self.df),
            "列数": len(self.df.columns),
            "数值列": self.numeric_cols,
            "类别列": self.cat_cols,
            "内存占用": f"{self.df.memory_usage(deep=True).sum() / 1024:.1f} KB",
            "总缺失值": int(self.df.isnull().sum().sum()),
        }

    def print_summary(self):
        s = self.summary()
        print(f"\n{'='*50}")
        print(f"📊 {self.name} — 数据摘要")
        print(f"{'='*50}")
        for k, v in s.items():
            print(f"  {k}: {v}")
        print(f"\n描述性统计：")
        print(self.df.describe().round(3))

    # ── 2. 缺失值分析 ─────────────────────────────
    def missing_analysis(self) -> pd.DataFrame:
        missing = self.df.isnull().sum()
        pct = (missing / len(self.df) * 100).round(2)
        result = pd.DataFrame({"缺失数": missing, "缺失率%": pct})
        return result[result["缺失数"] > 0].sort_values("缺失率%", ascending=False)

    def plot_missing(self):
        missing_df = self.missing_analysis()
        if missing_df.empty:
            print("✅ 无缺失值")
            return
        fig, ax = plt.subplots(figsize=(8, max(3, len(missing_df) * 0.4)))
        ax.barh(missing_df.index, missing_df["缺失率%"], color="#f87171")
        ax.set_xlabel("缺失率 (%)")
        ax.set_title(f"{self.name} — 缺失值分析")
        for i, v in enumerate(missing_df["缺失率%"]):
            ax.text(v + 0.5, i, f"{v}%", va="center", fontsize=9)
        plt.tight_layout()
        plt.show()

    # ── 3. 分布分析 ───────────────────────────────
    def plot_distributions(self, max_cols: int = 9):
        cols = self.numeric_cols[:max_cols]
        if not cols:
            print("无数值列")
            return
        n = len(cols)
        ncols = min(3, n)
        nrows = (n + ncols - 1) // ncols
        fig, axes = plt.subplots(nrows, ncols, figsize=(5 * ncols, 4 * nrows))
        axes = np.array(axes).flatten() if n > 1 else [axes]
        for i, col in enumerate(cols):
            axes[i].hist(self.df[col].dropna(), bins=30, color="#60a5fa", edgecolor="white")
            axes[i].set_title(col, fontsize=10)
            axes[i].set_xlabel("")
        for j in range(i + 1, len(axes)):
            axes[j].set_visible(False)
        fig.suptitle(f"{self.name} — 数值列分布", fontsize=13)
        plt.tight_layout()
        plt.show()

    def plot_categorical(self, max_cols: int = 6, top_n: int = 10):
        cols = self.cat_cols[:max_cols]
        if not cols:
            print("无类别列")
            return
        n = len(cols)
        ncols = min(2, n)
        nrows = (n + ncols - 1) // ncols
        fig, axes = plt.subplots(nrows, ncols, figsize=(7 * ncols, 4 * nrows))
        axes = np.array(axes).flatten() if n > 1 else [axes]
        for i, col in enumerate(cols):
            vc = self.df[col].value_counts().head(top_n)
            axes[i].barh(vc.index.astype(str), vc.values, color="#a78bfa")
            axes[i].set_title(col, fontsize=10)
            axes[i].invert_yaxis()
        for j in range(i + 1, len(axes)):
            axes[j].set_visible(False)
        fig.suptitle(f"{self.name} — 类别列分布（Top {top_n}）", fontsize=13)
        plt.tight_layout()
        plt.show()

    # ── 4. 相关性分析 ─────────────────────────────
    def plot_correlation(self):
        if len(self.numeric_cols) < 2:
            print("数值列不足 2 列，无法计算相关性")
            return
        corr = self.df[self.numeric_cols].corr()
        fig, ax = plt.subplots(figsize=(max(6, len(self.numeric_cols)), max(5, len(self.numeric_cols) - 1)))
        sns.heatmap(corr, annot=True, fmt=".2f", cmap="coolwarm",
                    center=0, square=True, ax=ax, annot_kws={"size": 9})
        ax.set_title(f"{self.name} — 相关性热力图")
        plt.tight_layout()
        plt.show()

    # ── 5. 异常值检测 ─────────────────────────────
    def outlier_report(self) -> pd.DataFrame:
        rows = []
        for col in self.numeric_cols:
            s = self.df[col].dropna()
            Q1, Q3 = s.quantile(0.25), s.quantile(0.75)
            IQR = Q3 - Q1
            lower, upper = Q1 - 1.5 * IQR, Q3 + 1.5 * IQR
            n_out = int(((s < lower) | (s > upper)).sum())
            rows.append({
                "列名": col, "异常值数": n_out,
                "异常值率%": round(n_out / len(s) * 100, 2),
                "下界": round(lower, 3), "上界": round(upper, 3),
            })
        return pd.DataFrame(rows).sort_values("异常值率%", ascending=False)

    def plot_boxplots(self, max_cols: int = 8):
        cols = self.numeric_cols[:max_cols]
        if not cols:
            return
        fig, ax = plt.subplots(figsize=(max(8, len(cols) * 1.5), 5))
        self.df[cols].boxplot(ax=ax, vert=True)
        ax.set_title(f"{self.name} — 箱线图（异常值检测）")
        plt.xticks(rotation=30, ha="right")
        plt.tight_layout()
        plt.show()

    # ── 6. AI 解读 ────────────────────────────────
    def ai_insights(self, focus: str = "总体") -> str:
        """用 Claude 生成数据洞察"""
        if focus in self._insights_cache:
            return self._insights_cache[focus]

        s = self.summary()
        missing = self.missing_analysis()
        outliers = self.outlier_report()
        desc = self.df.describe().round(3).to_string()

        context = f"""数据集名称：{self.name}
行数：{s['行数']}，列数：{s['列数']}
数值列：{', '.join(self.numeric_cols)}
类别列：{', '.join(self.cat_cols)}

描述性统计：
{desc}

缺失值情况：
{missing.to_string() if not missing.empty else '无缺失值'}

异常值情况：
{outliers.head(5).to_string()}

分析重点：{focus}"""

        client = anthropic.Anthropic()
        resp = client.messages.create(
            model="claude-opus-4-7",
            max_tokens=600,
            system=(
                "你是一位产品数据分析师，擅长从数据统计特征中发现产品洞察。\n"
                "请根据数据摘要生成 3-5 条关键洞察，每条包含：\n"
                "- 发现了什么（具体数字）\n"
                "- 可能的原因\n"
                "- 建议的产品行动\n"
                "格式：Markdown 列表，简洁直接。"
            ),
            messages=[{"role": "user", "content": context}]
        )
        result = resp.content[0].text
        self._insights_cache[focus] = result
        return result

    # ── 全流程报告 ────────────────────────────────
    def full_report(self, with_ai: bool = True):
        """运行完整 EDA 分析"""
        self.print_summary()
        print("\n📌 缺失值分析")
        self.plot_missing()
        print("\n📌 数值分布")
        self.plot_distributions()
        print("\n📌 类别分布")
        self.plot_categorical()
        print("\n📌 相关性分析")
        self.plot_correlation()
        print("\n📌 异常值检测")
        print(self.outlier_report().to_string())
        self.plot_boxplots()
        if with_ai:
            print("\n🤖 AI 数据洞察")
            print(self.ai_insights())


# ─────────────────────────────────────────────
#  快速入口
# ─────────────────────────────────────────────
if __name__ == "__main__":
    import sys
    if len(sys.argv) < 2:
        print("用法：python eda_template.py <csv文件>")
        sys.exit(1)
    
    df = pd.read_csv(sys.argv[1])
    name = sys.argv[1].split("/")[-1].replace(".csv", "")
    eda = EDAAnalyzer(df, name)
    eda.full_report()
```

#### 运行方式

```bash
# 对任意 CSV 文件运行 EDA
python eda_template.py samples/ecommerce.csv

# 在 Python 脚本中复用
from eda_template import EDAAnalyzer
import pandas as pd

df = pd.read_csv("your_data.csv")
eda = EDAAnalyzer(df, "你的数据集")
eda.full_report()

# 只看缺失值
print(eda.missing_analysis())

# 只要 AI 洞察
print(eda.ai_insights(focus="用户留存"))
```

***

### 🔧 封装产出（4 分钟）

```
eda-notebook/
├── eda_template.py        ✅ 可复用 EDA 函数库（EDAAnalyzer 类）
│   ├── summary()          基础摘要
│   ├── missing_analysis() 缺失值分析
│   ├── plot_distributions() 数值分布图
│   ├── plot_categorical() 类别分布图
│   ├── plot_correlation() 相关性热力图
│   ├── outlier_report()   异常值报告
│   ├── ai_insights()      AI 数据洞察
│   └── full_report()      一键全流程
```

**复用方式**：只需 `from eda_template import EDAAnalyzer`，面对任何 CSV 数据集，三行代码完成完整 EDA。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

EDA 5步工作流
├── Step 1: describe() 基础统计
├── Step 2: isnull().sum() 缺失值
├── Step 3: hist() 分布可视化
├── Step 4: corr() + heatmap 相关性
└── Step 5: IQR boxplot 异常值

Seaborn
├── sns.heatmap() 热力图
├── sns.set_theme() 主题设置
└── 与 Matplotlib 协同使用

工程设计
├── EDAAnalyzer 类封装所有分析步骤
├── _insights_cache 缓存 AI 结果，避免重复调用
└── full_report() 一键运行所有步骤
```

***

### 🎯 课后作业

**练习 1（基础）**：为 `EDAAnalyzer` 增加 `time_series_plot()` 方法，自动检测日期列并绘制时间序列趋势图。

**练习 2（进阶）**：把 `EDAAnalyzer` 集成到 Streamlit 应用中，用户上传 CSV 后，页面展示所有分析图表和 AI 洞察。

**练习 3（挑战）**：实现"增量 EDA"——支持对比两个不同时间段的数据集，高亮分布发生显著变化的列（数据漂移检测）。

***

_第二阶段：产品设计 × Python 实战 · 第 19 课 / 共 10 课_
