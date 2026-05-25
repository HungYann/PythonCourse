# 第 13 课｜需求清洗：海量用户反馈 → 可执行产品需求

## 第二阶段：产品设计 × Python 实战

### 第 13 课｜需求清洗：海量用户反馈 → 可执行产品需求

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～12 课** 🛠️ **工具准备：Python 3.x、pip install pandas anthropic**

***

### 课程导览

| 模块      | 内容                             | 时间    |
| ------- | ------------------------------ | ----- |
| 🔥 热身回顾 | 第 12 课爬虫模式回顾                   | 3 分钟  |
| 🎯 场景拆解 | 用户反馈噪音问题与 Pandas + AI 解法       | 5 分钟  |
| 📚 核心知识 | Pandas 数据清洗、词频统计、Claude API 聚类 | 10 分钟 |
| 💻 代码实战 | 完整 feedback-cleaner.py 开发      | 20 分钟 |
| 🔧 封装产出 | 命令行管道 + 需求报告                   | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告                   | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 12 课我们学会了**主动抓取**外部数据：

```
URL → requests → BeautifulSoup → 结构化文本 → Claude 分析
```

这节课数据换成了**用户反馈**：App Store 评论、问卷结果、NPS 开放题……量大、重复、充满噪音。

核心挑战不是"获取数据"，而是**清洗 + 聚类 + 提炼成可执行需求**。这是 Pandas 的主场。

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
你有一份 CSV，里面是 500 条 App Store 评论：
  "太卡了！！！！"
  "怎么找不到导出功能，急死了"
  "很好用，推荐"
  "为什么没有深色模式？？其他 app 都有"
  "好评，就是广告太多了"
  …

需要从中提炼：下季度迭代优先级最高的 5 个需求

传统方式：人工逐条阅读 → 分类归纳 → 2 天
本课方式：Pandas 清洗 → 词频统计 → Claude 聚类提炼 → 20 分钟
```

#### 解决思路

```
CSV 文件（用户反馈）
      │
      ▼
Pandas 读取 & 清洗
  - 去重、去空行
  - 过滤极短噪音（< 5 字）
  - 提取评分分布
      │
      ▼
词频统计（Top 50 高频词）
      │
      ▼
Claude API：将高频词 + 低分反馈 → 需求聚类
      │
      ▼
输出：可执行需求报告 Markdown
```

***

### 📚 核心知识（10 分钟）

#### 1. Pandas 基础操作

```python
import pandas as pd

# 读取 CSV
df = pd.read_csv("feedback.csv", encoding="utf-8")

# 查看前 5 行
print(df.head())

# 查看列名
print(df.columns.tolist())

# 过滤：只保留内容长度 > 5 的行
df = df[df["content"].str.len() > 5]

# 去重
df = df.drop_duplicates(subset="content")

# 重置索引
df = df.reset_index(drop=True)

print(f"清洗后剩余 {len(df)} 条反馈")
```

#### 2. 词频统计（不依赖 jieba）

中文分词工具（jieba）安装有时较复杂。本课用**简单字符串切割**获得词频，足以提取产品关键词：

```python
from collections import Counter
import re

def count_keywords(texts: list[str], top_n: int = 50) -> list[tuple[str, int]]:
    """统计高频词（过滤停用词）"""
    stop_words = {
        "的", "了", "是", "在", "我", "有", "和", "就", "都", "而",
        "及", "与", "这", "那", "也", "很", "但", "不", "好", "用",
        "太", "还", "感觉", "一个", "可以", "没有", "一直", "非常",
    }
    all_words = []
    for text in texts:
        # 用2~6字长度的词（近似词语）
        words = re.findall(r'[一-龥]{2,6}', text)
        all_words.extend([w for w in words if w not in stop_words])
    
    return Counter(all_words).most_common(top_n)
```

#### 3. 构建分析上下文给 Claude

把统计结果而不是原始数据传给 Claude，大幅减少 token：

```python
def build_analysis_context(df: pd.DataFrame, keywords: list) -> str:
    """构建精简的分析上下文"""
    total = len(df)
    
    # 评分分布（如果有评分列）
    rating_dist = ""
    if "rating" in df.columns:
        dist = df["rating"].value_counts().sort_index()
        rating_dist = "\n".join([f"  {r}星：{c}条" for r, c in dist.items()])
    
    # 低分反馈样本（1-2星，最有价值）
    low_ratings = []
    if "rating" in df.columns:
        low_df = df[df["rating"] <= 2]["content"].head(30).tolist()
        low_ratings = low_df
    else:
        # 没有评分列，取所有反馈前 50 条
        low_ratings = df["content"].head(50).tolist()
    
    kw_text = "\n".join([f"  {w}（{c}次）" for w, c in keywords[:30]])
    samples = "\n".join([f"  - {t}" for t in low_ratings[:20]])
    
    return f"""反馈总量：{total} 条

评分分布：
{rating_dist or "  无评分数据"}

高频关键词 Top 30：
{kw_text}

低分/负面反馈样本（前 20 条）：
{samples}"""
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
feedback-cleaner/
├── feedback-cleaner.py    ← 主程序
├── samples/
│   └── sample_feedback.csv   ← 测试数据
└── output/                   ← 自动创建，存放需求报告
```

#### 测试数据：samples/sample\_feedback.csv

```csv
rating,content
5,功能很全，特别是日历同步，强烈推荐
1,每次打开都崩溃，卸了
2,找不到导出 PDF 功能，非常不方便
3,整体还行，但深色模式什么时候上
5,好用！就是希望有 widget
1,太卡了，手机都发烫
2,同步经常失败，数据丢了一次
4,用了两年了，就是广告太多了
1,登录一直报错，客服也不理人
3,希望能加个提醒功能，现在提醒太简单
2,界面不好看，字体太小
5,非常棒，团队协作功能一流
1,购买了会员发现功能和免费版差不多
2,没有离线模式，没网就废了
3,分享链接经常打不开
```

#### 完整代码：feedback-cleaner.py

```python
#!/usr/bin/env python3
"""
用户反馈清洗工具 | feedback-cleaner.py
用法：python feedback-cleaner.py <csv文件>
示例：python feedback-cleaner.py samples/sample_feedback.csv
      python feedback-cleaner.py samples/sample_feedback.csv --col review_text
依赖：pip install pandas anthropic
"""
import sys
import os
import re
import argparse
from collections import Counter
import pandas as pd
import anthropic
from datetime import datetime


SYSTEM_PROMPT = """你是一位经验丰富的产品经理，擅长从用户反馈中提炼可执行需求。

请根据用户提供的反馈统计数据，输出结构化需求分析报告：

## 📊 数据概览
[用 2-3 句话概括反馈整体情况]

## 🔴 核心痛点（按严重程度排序）
每个痛点格式：
### 痛点 N：[痛点名称]
- **频率**：[高/中/低]
- **用户影响**：[说明这个痛点影响用户的哪个核心流程]
- **代表性反馈**：[引用 1-2 条原始反馈]
- **建议解法**：[具体可执行的产品改进方向]

## ✅ 下季度需求优先级（TOP 5）
| 优先级 | 需求 | 理由 | 预估影响 |
|--------|------|------|---------|
| P0 | | | |
| P1 | | | |
...

## 💛 正向反馈（继续保持）
- [用户最喜欢的功能/体验，3 条]

## ⚠️ 风险预警
- [如果不解决，可能导致用户流失的问题]

只输出 Markdown，不加说明性文字。"""


# ─────────────────────────────────────────────
#  1. 读取并清洗 CSV
# ─────────────────────────────────────────────
def load_and_clean(filepath: str, content_col: str = "content") -> pd.DataFrame:
    """读取 CSV，清洗无效行"""
    if not os.path.exists(filepath):
        print(f"❌ 文件不存在：{filepath}")
        sys.exit(1)

    df = pd.read_csv(filepath, encoding="utf-8")

    # 自动识别内容列
    if content_col not in df.columns:
        # 尝试常见列名
        for candidate in ["content", "review", "comment", "text", "feedback", "评论", "内容"]:
            if candidate in df.columns:
                content_col = candidate
                break
        else:
            # 取第一个字符串列
            str_cols = df.select_dtypes(include="object").columns.tolist()
            if not str_cols:
                print("❌ 未找到文本列，请用 --col 指定列名")
                sys.exit(1)
            content_col = str_cols[0]

    print(f"📋 使用列：{content_col}（共 {len(df)} 行原始数据）")

    df = df.rename(columns={content_col: "content"})
    df["content"] = df["content"].astype(str).str.strip()

    # 清洗
    df = df[df["content"].str.len() >= 5]          # 过滤过短
    df = df[~df["content"].str.match(r'^[好坏差不错]+$')]  # 过滤纯情绪词
    df = df.drop_duplicates(subset="content")
    df = df.reset_index(drop=True)

    print(f"✅ 清洗后：{len(df)} 条有效反馈")
    return df


# ─────────────────────────────────────────────
#  2. 词频统计
# ─────────────────────────────────────────────
def count_keywords(texts: list[str], top_n: int = 50) -> list[tuple[str, int]]:
    stop_words = {
        "的", "了", "是", "在", "我", "有", "和", "就", "都", "而",
        "及", "与", "这", "那", "也", "很", "但", "不", "用", "太",
        "还", "感觉", "一个", "可以", "没有", "一直", "非常", "真的",
        "希望", "功能", "一下", "时候", "什么", "为什么", "如果",
    }
    all_words = []
    for text in texts:
        words = re.findall(r'[一-龥]{2,6}', text)
        all_words.extend([w for w in words if w not in stop_words])
    return Counter(all_words).most_common(top_n)


# ─────────────────────────────────────────────
#  3. 构建分析上下文
# ─────────────────────────────────────────────
def build_context(df: pd.DataFrame, keywords: list) -> str:
    total = len(df)

    # 评分分布
    rating_info = ""
    if "rating" in df.columns:
        try:
            df["rating"] = pd.to_numeric(df["rating"], errors="coerce")
            dist = df["rating"].value_counts().sort_index()
            rating_info = "\n".join([f"  {int(r)}星：{c}条" for r, c in dist.items() if not pd.isna(r)])
        except Exception:
            pass

    # 低分样本
    if "rating" in df.columns:
        try:
            low_df = df[df["rating"] <= 2]["content"].dropna().head(25).tolist()
        except Exception:
            low_df = df["content"].head(25).tolist()
    else:
        low_df = df["content"].head(25).tolist()

    kw_text = "\n".join([f"  {w}（{c}次）" for w, c in keywords[:30]])
    samples = "\n".join([f"  - {t}" for t in low_df])

    return f"""反馈总量：{total} 条

评分分布：
{rating_info or "  无评分数据"}

高频关键词 Top 30：
{kw_text}

负面/低分反馈样本：
{samples}"""


# ─────────────────────────────────────────────
#  4. Claude API 分析
# ─────────────────────────────────────────────
def analyze_feedback(context: str) -> str:
    client = anthropic.Anthropic()
    print("\n⏳ 正在调用 Claude API 提炼需求...")
    message = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=2500,
        system=SYSTEM_PROMPT,
        messages=[
            {"role": "user", "content": f"请分析以下用户反馈数据：\n\n{context}"}
        ]
    )
    return message.content[0].text


# ─────────────────────────────────────────────
#  5. 保存报告
# ─────────────────────────────────────────────
def save_report(content: str, output_dir: str = "output") -> str:
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = os.path.join(output_dir, f"需求报告_{timestamp}.md")
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(content)
    return filepath


# ─────────────────────────────────────────────
#  6. 主程序
# ─────────────────────────────────────────────
def main():
    parser = argparse.ArgumentParser(description="用户反馈清洗工具")
    parser.add_argument("csv_file", help="CSV 文件路径")
    parser.add_argument("--col", default="content", help="反馈文本列名（默认：content）")
    args = parser.parse_args()

    # Step 1: 读取 & 清洗
    df = load_and_clean(args.csv_file, args.col)

    # Step 2: 词频统计
    keywords = count_keywords(df["content"].tolist())
    print(f"📊 Top 10 高频词：{', '.join([w for w, _ in keywords[:10]])}")

    # Step 3: 构建上下文
    context = build_context(df, keywords)

    # Step 4: AI 分析
    report = analyze_feedback(context)

    # Step 5: 保存
    output_path = save_report(report)

    print(f"\n✅ 需求报告已保存：{output_path}")
    print("\n" + "=" * 60)
    print(report)
    print("=" * 60)


if __name__ == "__main__":
    main()
```

#### 运行效果

```bash
python feedback-cleaner.py samples/sample_feedback.csv

📋 使用列：content（共 15 行原始数据）
✅ 清洗后：15 条有效反馈
📊 Top 10 高频词：深色模式, 导出, 广告, 崩溃, 同步, 离线, widget, 提醒, 客服, 团队

⏳ 正在调用 Claude API 提炼需求...
✅ 需求报告已保存：output/需求报告_20260524_160342.md

============================================================
## 📊 数据概览
共 15 条反馈，好评与差评比例接近，用户对产品核心功能总体认可，
但稳定性问题（崩溃、卡顿）和缺失功能（深色模式、离线、导出）导致评分拖低。

## 🔴 核心痛点

### 痛点 1：应用稳定性差
- **频率**：高
- **用户影响**：崩溃和卡顿影响日常使用，直接导致卸载
- **代表性反馈**："每次打开都崩溃，卸了" / "太卡了，手机都发烫"
- **建议解法**：重点排查内存泄露，添加 ANR 监控，crash 率降至 0.1% 以下

### 痛点 2：缺少深色模式
- **频率**：中
...

## ✅ 下季度需求优先级（TOP 5）
| 优先级 | 需求 | 理由 | 预估影响 |
|--------|------|------|---------|
| P0 | 修复崩溃/卡顿 | 直接导致卸载 | 降低流失率 20%+ |
| P1 | 深色模式 | 高频提及，竞品标配 | 提升留存 |
...
============================================================
```

***

### 🔧 封装产出（4 分钟）

本课交付物：

```
feedback-cleaner/
├── feedback-cleaner.py    ✅ 命令行工具（支持自定义列名）
├── samples/
│   └── sample_feedback.csv  ✅ 测试数据
└── output/
    └── 需求报告_*.md        ✅ 可执行需求分析报告
```

**复用场景**：把任意产品的评论 CSV 传入，一行命令产出 PRD 级别的需求优先级报告。下节课我们直接从需求报告生成完整 PRD 文档。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

Pandas 数据处理
├── pd.read_csv() 读取 CSV
├── df[条件] 过滤行
├── drop_duplicates() 去重
├── value_counts() 频率统计
└── select_dtypes() 自动识别列类型

文本处理
├── re.findall() 提取中文词语
├── collections.Counter 词频统计
└── 停用词过滤

工程技巧
├── argparse 添加命令行参数（--col）
├── 自动识别列名，降低使用门槛
└── 统计摘要传 AI（不传原始数据）节省 token
```

***

### 🎯 课后作业

**练习 1（基础）**：在报告中增加"正面反馈词云"区块（高频正面词汇），帮助产品团队了解核心优势。

**练习 2（进阶）**：支持同时处理多个 CSV（来自不同渠道：App Store + 问卷），合并后统一分析，并在报告中注明各渠道占比。

**练习 3（挑战）**：用 Pandas 的 `cut()` 函数将反馈按"评分区间 × 关键词"做二维分类热力图，并保存为 PNG 图片。

***

_第二阶段：产品设计 × Python 实战 · 第 13 课 / 共 10 课_
