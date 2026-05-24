# 第 12 课｜竞品逆向：抓取竞品数据，10 分钟摸清产品逻辑

## 第二阶段：产品设计 × Python 实战

### 第 12 课｜竞品逆向：抓取竞品数据，10 分钟摸清产品逻辑

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～11 课** 🛠️ **工具准备：Python 3.x、pip、Anthropic API Key**

***

### 课程导览

| 模块      | 内容                                     | 时间    |
| ------- | -------------------------------------- | ----- |
| 🔥 热身回顾 | 第 11 课封装模式回顾                           | 3 分钟  |
| 🎯 场景拆解 | 竞品分析痛点与爬虫 + AI 解法                      | 5 分钟  |
| 📚 核心知识 | requests、BeautifulSoup、Claude API 图文分析 | 10 分钟 |
| 💻 代码实战 | 完整 competitor-scan.py 开发               | 20 分钟 |
| 🔧 封装产出 | 命令行工具 + Markdown 报告                    | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告                           | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 11 课我们完成了"会议纪要工厂"——核心模式是：

```
原始文本 → Python 读取 → 构建 Prompt → Claude API → 结构化输出 → 保存文件
```

这节课把同样的模式**扩展到网络**：

```
竞品 URL → requests 抓取网页 → BeautifulSoup 提取内容 → Claude API 分析 → 竞品报告
```

差别只有一步：**数据来源从本地文件变成了网络请求**。

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
你负责一款 To-Do 产品，老板问：
  "Todoist、Things 3、TickTick 这三款竞品都有什么核心功能？
   我们差在哪里？下季度该加什么？"

传统方式：打开每个官网 → 逐页截图 → 手动整理 → 写报告 → 2 小时
本课方式：一行命令抓取 + AI 分析 → 10 分钟出报告
```

#### 解决思路

```
competitor-scan.py <url> [url2] [url3]
        │
        ▼
requests 发送 HTTP 请求，获取 HTML
        │
        ▼
BeautifulSoup 提取：标题、导航、功能描述、定价
        │
        ▼
Claude API 分析：核心功能 / 目标用户 / 差异化卖点 / 潜在弱点
        │
        ▼
Markdown 竞品报告 → output/竞品分析_时间戳.md
```

***

### 📚 核心知识（10 分钟）

#### 1. 安装依赖

```bash
pip install requests beautifulsoup4 anthropic
```

#### 2. requests：发送 HTTP 请求

```python
import requests

headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0 Safari/537.36"
}

response = requests.get("https://todoist.com", headers=headers, timeout=10)
print(response.status_code)   # 200 = 成功
print(len(response.text))     # 页面 HTML 长度
```

> **为什么要设置 User-Agent？**\
> 部分网站拒绝没有浏览器标识的请求，伪装成浏览器能提高成功率。

#### 3. BeautifulSoup：解析 HTML

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(response.text, "html.parser")

# 提取标题
title = soup.find("title").get_text(strip=True)

# 提取所有导航链接文字
nav_texts = [a.get_text(strip=True) for a in soup.find_all("a") if a.get_text(strip=True)]

# 提取 meta description（产品一句话介绍）
meta_desc = soup.find("meta", attrs={"name": "description"})
description = meta_desc["content"] if meta_desc else ""

# 提取正文关键段落（h1~h3 标题 + p 段落）
headings = [h.get_text(strip=True) for h in soup.find_all(["h1", "h2", "h3"])]
paragraphs = [p.get_text(strip=True) for p in soup.find_all("p") if len(p.get_text(strip=True)) > 30]
```

#### 4. 控制文本长度再传给 Claude

网页内容往往几万字，全部传给 Claude 会浪费 token。**只取前 3000 字**即可覆盖产品核心信息：

```python
def extract_key_content(soup) -> str:
    parts = []
    parts.append("【页面标题】" + (soup.title.get_text(strip=True) if soup.title else ""))
    meta = soup.find("meta", attrs={"name": "description"})
    if meta:
        parts.append("【产品描述】" + meta.get("content", ""))
    headings = [h.get_text(strip=True) for h in soup.find_all(["h1", "h2", "h3"])[:20]]
    parts.append("【页面标题层级】\n" + "\n".join(headings))
    paras = [p.get_text(strip=True) for p in soup.find_all("p") if len(p.get_text(strip=True)) > 30]
    parts.append("【正文段落摘录】\n" + "\n".join(paras[:30]))
    return "\n\n".join(parts)[:3000]   # 截断到 3000 字
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
competitor-scan/
├── competitor-scan.py    ← 主程序
└── output/               ← 自动创建，存放竞品报告
```

#### 完整代码：competitor-scan.py

```python
#!/usr/bin/env python3
"""
竞品逆向扫描工具 | competitor-scan.py
用法：python competitor-scan.py <url1> [url2] [url3]
示例：python competitor-scan.py https://todoist.com https://ticktick.com
依赖：pip install requests beautifulsoup4 anthropic
"""
import sys
import os
import requests
from bs4 import BeautifulSoup
import anthropic
from datetime import datetime


HEADERS = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
                  "AppleWebKit/537.36 (KHTML, like Gecko) "
                  "Chrome/120.0.0.0 Safari/537.36"
}

SYSTEM_PROMPT = """你是一位资深产品分析师。
请根据用户提供的竞品网页内容，输出结构化 Markdown 竞品分析报告。

每个竞品必须包含：
### 🏷️ [产品名称]（[URL]）

**一句话定位**：[产品核心价值主张]

**目标用户**：[用户画像]

**核心功能**（按重要度排序）：
1. [功能]：[说明]
2. ...

**差异化卖点**：
- [与同类产品的独特之处]

**潜在弱点**：
- [可能存在的不足]

---

若分析了多个竞品，最后补充：
## 📊 横向对比
| 维度 | [产品A] | [产品B] | ... |
|------|--------|--------|-----|
| 核心定位 | | | |
| 目标用户 | | | |
| 最大优势 | | | |
| 明显短板 | | | |

## 💡 机会点
- [基于以上分析，你的产品可以在哪里建立差异化优势]

只输出 Markdown，不加任何说明性文字。"""


# ─────────────────────────────────────────────
#  1. 抓取网页并提取关键内容
# ─────────────────────────────────────────────
def fetch_page(url: str) -> tuple[str, str]:
    """抓取网页，返回 (域名, 结构化文本内容)"""
    print(f"  🌐 正在抓取：{url}")
    try:
        resp = requests.get(url, headers=HEADERS, timeout=15)
        resp.raise_for_status()
        resp.encoding = resp.apparent_encoding
    except requests.RequestException as e:
        print(f"  ⚠️  抓取失败：{e}")
        return url, f"【抓取失败】{e}"

    soup = BeautifulSoup(resp.text, "html.parser")

    # 移除 script / style 节点，减少噪音
    for tag in soup(["script", "style", "noscript"]):
        tag.decompose()

    parts = []

    # 标题
    title = soup.title.get_text(strip=True) if soup.title else ""
    parts.append(f"【页面标题】{title}")

    # Meta description
    meta = soup.find("meta", attrs={"name": "description"})
    if meta and meta.get("content"):
        parts.append(f"【产品描述】{meta['content']}")

    # OG description（更丰富的描述）
    og_desc = soup.find("meta", property="og:description")
    if og_desc and og_desc.get("content"):
        parts.append(f"【OG描述】{og_desc['content']}")

    # 标题层级 h1~h3
    headings = [h.get_text(strip=True) for h in soup.find_all(["h1", "h2", "h3"])[:25]
                if h.get_text(strip=True)]
    if headings:
        parts.append("【标题结构】\n" + "\n".join(headings))

    # 段落摘录
    paras = [p.get_text(strip=True) for p in soup.find_all("p")
             if len(p.get_text(strip=True)) > 30][:35]
    if paras:
        parts.append("【正文摘录】\n" + "\n".join(paras))

    # 导航链接（了解产品功能模块）
    nav_links = list(dict.fromkeys(
        a.get_text(strip=True) for a in soup.find_all("a")
        if 3 < len(a.get_text(strip=True)) < 30
    ))[:40]
    if nav_links:
        parts.append("【导航/链接文字】\n" + " | ".join(nav_links))

    content = "\n\n".join(parts)
    return url, content[:4000]  # 截断到 4000 字符


# ─────────────────────────────────────────────
#  2. 调用 Claude API 生成竞品报告
# ─────────────────────────────────────────────
def analyze_competitors(pages: list[tuple[str, str]]) -> str:
    """将多个竞品内容拼接后，一次调用 Claude 生成完整分析"""
    client = anthropic.Anthropic()

    user_content = ""
    for url, content in pages:
        user_content += f"\n\n{'='*50}\n竞品 URL：{url}\n{'='*50}\n{content}"

    print("\n⏳ 正在调用 Claude API 进行竞品分析...")

    message = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=3000,
        system=SYSTEM_PROMPT,
        messages=[
            {"role": "user", "content": f"请分析以下竞品网页内容：{user_content}"}
        ]
    )

    return message.content[0].text


# ─────────────────────────────────────────────
#  3. 保存报告
# ─────────────────────────────────────────────
def save_report(content: str, output_dir: str = "output") -> str:
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = os.path.join(output_dir, f"竞品分析_{timestamp}.md")
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(content)
    return filepath


# ─────────────────────────────────────────────
#  4. 主程序
# ─────────────────────────────────────────────
def main():
    if len(sys.argv) < 2:
        print("用法：python competitor-scan.py <url1> [url2] [url3]")
        print("示例：python competitor-scan.py https://todoist.com https://ticktick.com")
        sys.exit(1)

    urls = sys.argv[1:]
    print(f"📋 待分析竞品：{len(urls)} 个")

    # Step 1: 批量抓取
    pages = []
    for url in urls:
        result = fetch_page(url)
        pages.append(result)

    # Step 2: AI 分析
    report = analyze_competitors(pages)

    # Step 3: 保存
    output_path = save_report(report)

    print(f"\n✅ 报告已保存：{output_path}")
    print("\n" + "=" * 60)
    print(report)
    print("=" * 60)


if __name__ == "__main__":
    main()
```

#### 运行示例

```bash
python competitor-scan.py https://todoist.com https://ticktick.com

# 控制台输出：
📋 待分析竞品：2 个
  🌐 正在抓取：https://todoist.com
  🌐 正在抓取：https://ticktick.com

⏳ 正在调用 Claude API 进行竞品分析...
✅ 报告已保存：output/竞品分析_20260524_151203.md

============================================================
### 🏷️ Todoist（https://todoist.com）

**一句话定位**：面向效率人士的跨平台任务管理工具，以简洁和自然语言输入见长

**目标用户**：个人效率提升者、远程团队协作者、GTD 方法论实践者

**核心功能**：
1. 自然语言任务创建：输入"明天下午三点开会"自动解析为任务+提醒
2. 项目与标签系统：按项目/标签组织任务，支持子任务嵌套
3. 多平台同步：iOS/Android/Web/桌面端无缝同步
4. Karma 生产力分数：游戏化机制增强坚持动力

**差异化卖点**：
- 自然语言解析在同类产品中最为准确
- Inbox 零收件箱理念，降低决策负担

**潜在弱点**：
- 免费版项目数量限制（5个）
- 日历视图功能较弱
...
============================================================
```

***

### 🔧 封装产出（4 分钟）

本课交付物：

```
competitor-scan/
├── competitor-scan.py    ✅ 命令行工具（支持多 URL）
└── output/
    └── 竞品分析_*.md    ✅ 结构化竞品报告
```

**复用场景**：做竞品调研时，把竞品官网 URL 列表粘贴进命令，10 分钟出报告。下节课我们将处理"更乱的数据"——海量用户反馈文本。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

网络请求
├── requests.get(url, headers, timeout)
├── response.status_code / response.text
└── User-Agent 模拟浏览器

HTML 解析
├── BeautifulSoup(html, "html.parser")
├── soup.find() / find_all()
├── .get_text(strip=True)
└── meta["content"] 提取描述

工程技巧
├── 截断内容（[:4000]）控制 token 消耗
├── 批量抓取 + 一次 API 调用
└── 异常处理（requests.RequestException）
```

***

### 🎯 课后作业

**练习 1（基础）**：在 `SYSTEM_PROMPT` 中增加"定价策略"维度，让 Claude 分析竞品定价层级（免费版/专业版/企业版）。

**练习 2（进阶）**：抓取竞品的 App Store / Google Play 页面，提取用户评论中的高频词，添加到竞品分析报告的"用户口碑"区块。

**练习 3（挑战）**：支持 `--compare` 参数，分析完所有竞品后，额外生成一份"差距矩阵"CSV 文件，列出你产品与每个竞品的功能差距项。

***

_第二阶段：产品设计 × Python 实战 · 第 12 课 / 共 10 课_
