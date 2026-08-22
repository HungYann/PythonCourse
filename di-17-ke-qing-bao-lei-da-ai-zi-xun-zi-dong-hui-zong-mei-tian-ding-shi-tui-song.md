# 第 17 课｜情报雷达：AI 资讯自动汇总，每天定时推送

## 第二阶段：产品设计 × Python 实战

### 第 17 课｜情报雷达：AI 资讯自动汇总，每天定时推送

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～16 课** 🛠️ **工具准备：Python 3.x、pip install feedparser requests anthropic schedule**

***

### 课程导览

| 模块      | 内容                            | 时间    |
| ------- | ----------------------------- | ----- |
| 🔥 热身回顾 | 第 12 课爬虫 + 第 11 课 Claude 调用   | 3 分钟  |
| 🎯 场景拆解 | 信息过载痛点与 RSS + AI 解法           | 5 分钟  |
| 📚 核心知识 | feedparser、定时任务 schedule、邮件发送 | 10 分钟 |
| 💻 代码实战 | 完整 news-radar.py 开发           | 20 分钟 |
| 🔧 封装产出 | 定时脚本 + 推送服务                   | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告                  | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

前几课的工具都是**被动等待触发**：

```
第 11 课：你运行脚本 → 生成纪要
第 12 课：你传入 URL → 生成竞品报告
第 13 课：你上传 CSV → 清洗需求
```

本课第一次做**主动定时任务**：

```
每天早上 8:00，脚本自动：
  ① 从 RSS 源抓取最新资讯
  ② Claude 提炼摘要
  ③ 发送到你的邮箱 / 写入文件
无需人工触发。
```

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
作为产品经理，你需要每天关注：
  - AI 领域动态（AI News RSS）
  - 竞品更新（Hacker News、Product Hunt）
  - 行业报告（少数派、36kr）

每天花 1 小时刷新闻，还容易遗漏重要信息
```

#### 解决思路

```
RSS 源列表（配置文件）
      │
      ▼
feedparser 拉取每个 RSS 的最新文章
      │
      ▼
过滤：只保留今天的新文章
      │
      ▼
Claude API 摘要：每篇文章 1-2 句话
      │
      ▼
汇总 → Markdown 日报文件 + 可选邮件推送
      │
      ▼
schedule 定时：每天 8:00 自动运行
```

***

### 📚 核心知识（10 分钟）

#### 1. feedparser：解析 RSS/Atom

```bash
pip install feedparser
```

```python
import feedparser

# 解析 RSS 源
feed = feedparser.parse("https://hnrss.org/frontpage")

# 查看频道信息
print(feed.feed.title)          # Hacker News Front Page
print(len(feed.entries))        # 文章数量

# 遍历文章
for entry in feed.entries[:5]:
    print(entry.title)          # 文章标题
    print(entry.link)           # 文章链接
    print(entry.published)      # 发布时间（字符串）
    print(entry.summary[:200])  # 摘要（前 200 字符）
```

#### 2. 过滤今天的文章

```python
from datetime import datetime, timezone
import time

def is_today(entry) -> bool:
    """判断 RSS 条目是否是今天发布的"""
    try:
        # feedparser 的 published_parsed 是 time.struct_time
        pub_time = time.mktime(entry.published_parsed)
        pub_dt = datetime.fromtimestamp(pub_time)
        today = datetime.now().date()
        return pub_dt.date() == today
    except (AttributeError, TypeError, OverflowError):
        return True  # 无法判断时间时默认包含
```

#### 3. schedule：Python 定时任务

```bash
pip install schedule
```

```python
import schedule
import time

def job():
    print("执行任务...")

# 每天早上 8:00 运行
schedule.every().day.at("08:00").do(job)

# 每小时运行（开发测试用）
schedule.every(1).hours.do(job)

# 每 30 秒（极速测试）
schedule.every(30).seconds.do(job)

# 主循环
while True:
    schedule.run_pending()
    time.sleep(60)
```

#### 4. 批量摘要（节省 API 调用次数）

不要每篇文章单独调用 Claude，把多篇一起传入：

```python
SUMMARIZE_PROMPT = """你是一位产品经理助手，擅长快速提炼科技资讯。
请为每篇文章生成 1-2 句中文摘要，聚焦：这对产品从业者意味着什么？
输出格式：
1. [文章标题] — [摘要]
2. ...
不加任何解释。"""

# 把多篇文章标题+摘要拼在一起，一次 API 调用
batch_content = "\n\n".join([
    f"标题：{e.title}\n内容：{e.summary[:300]}"
    for e in entries[:10]
])
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
news-radar/
├── news-radar.py         ← 主程序（定时脚本）
├── config.py             ← RSS 源配置
└── output/               ← 自动创建，存放日报
```

#### 配置文件：config.py

```python
# RSS 源配置
RSS_SOURCES = [
    {
        "name": "Hacker News 热帖",
        "url": "https://hnrss.org/frontpage",
        "category": "科技",
        "max_items": 10,
    },
    {
        "name": "Product Hunt 今日产品",
        "url": "https://www.producthunt.com/feed",
        "category": "产品",
        "max_items": 8,
    },
    {
        "name": "The Verge",
        "url": "https://www.theverge.com/rss/index.xml",
        "category": "科技",
        "max_items": 8,
    },
]

# 可选：邮件推送配置（留空则不发邮件）
EMAIL_CONFIG = {
    "smtp_server": "",        # 如：smtp.gmail.com
    "smtp_port": 587,
    "sender": "",             # 你的邮箱
    "password": "",           # 授权码
    "recipient": "",          # 接收邮箱
}

# 运行时间
SCHEDULE_TIME = "08:00"
```

#### 完整代码：news-radar.py

```python
#!/usr/bin/env python3
"""
情报雷达 | news-radar.py
用法：
  立即执行一次：python news-radar.py --now
  启动定时任务：python news-radar.py
依赖：pip install feedparser requests anthropic schedule
"""
import sys
import os
import time
import argparse
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime
import feedparser
import anthropic
import schedule

# 导入配置
try:
    from config import RSS_SOURCES, EMAIL_CONFIG, SCHEDULE_TIME
except ImportError:
    # 内置默认配置
    RSS_SOURCES = [
        {"name": "Hacker News", "url": "https://hnrss.org/frontpage",
         "category": "科技", "max_items": 10},
        {"name": "The Verge", "url": "https://www.theverge.com/rss/index.xml",
         "category": "科技", "max_items": 8},
    ]
    EMAIL_CONFIG = {"smtp_server": "", "sender": "", "password": "", "recipient": ""}
    SCHEDULE_TIME = "08:00"


SYSTEM_PROMPT = """你是一位产品经理的资讯助理，擅长快速提炼科技/产品资讯的关键价值。

请为每篇文章生成 1-2 句中文摘要，重点回答：
- 这是什么事情？
- 对产品从业者意味着什么机会或威胁？

输出格式（严格遵守）：
### [序号]. [原文章标题]
**摘要**：[1-2 句中文摘要]
**关联性**：[对产品工作的启发，一句话]

处理完所有文章后，在末尾添加：
## 📌 今日要点（前 3 条最值得关注的）
1. [要点]
2. [要点]  
3. [要点]"""


# ─────────────────────────────────────────────
#  1. 拉取 RSS
# ─────────────────────────────────────────────
def is_recent(entry, hours: int = 48) -> bool:
    """判断文章是否在 hours 小时内发布"""
    try:
        pub_time = time.mktime(entry.published_parsed)
        age_hours = (time.time() - pub_time) / 3600
        return age_hours <= hours
    except (AttributeError, TypeError, OverflowError):
        return True


def fetch_rss(source: dict) -> list[dict]:
    """拉取单个 RSS 源，返回文章列表"""
    print(f"  📡 拉取：{source['name']}")
    try:
        feed = feedparser.parse(source["url"])
        entries = []
        for entry in feed.entries[:source["max_items"]]:
            if not is_recent(entry, hours=48):
                continue
            entries.append({
                "title": entry.get("title", "无标题"),
                "link": entry.get("link", ""),
                "summary": entry.get("summary", "")[:400],
                "category": source["category"],
                "source": source["name"],
            })
        print(f"    ✅ 获取 {len(entries)} 篇")
        return entries
    except Exception as e:
        print(f"    ⚠️  失败：{e}")
        return []


def fetch_all_sources() -> dict[str, list[dict]]:
    """拉取所有 RSS 源，按分类分组"""
    print(f"\n📡 开始拉取资讯（共 {len(RSS_SOURCES)} 个源）...")
    by_category: dict[str, list] = {}
    for source in RSS_SOURCES:
        articles = fetch_rss(source)
        cat = source["category"]
        if cat not in by_category:
            by_category[cat] = []
        by_category[cat].extend(articles)
    
    total = sum(len(v) for v in by_category.values())
    print(f"📊 共获取 {total} 篇文章")
    return by_category


# ─────────────────────────────────────────────
#  2. Claude API 批量摘要
# ─────────────────────────────────────────────
def summarize_articles(articles: list[dict], category: str) -> str:
    """批量摘要一个分类的文章"""
    if not articles:
        return ""
    
    client = anthropic.Anthropic()
    
    batch = "\n\n".join([
        f"文章 {i+1}：\n标题：{a['title']}\n来源：{a['source']}\n内容：{a['summary']}"
        for i, a in enumerate(articles[:12])
    ])
    
    print(f"  ⏳ AI 处理 [{category}] {len(articles)} 篇...")
    
    resp = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=1500,
        system=SYSTEM_PROMPT,
        messages=[{"role": "user", "content": f"请处理以下 [{category}] 类文章：\n\n{batch}"}]
    )
    return resp.content[0].text


# ─────────────────────────────────────────────
#  3. 生成日报
# ─────────────────────────────────────────────
def generate_daily_report(by_category: dict[str, list[dict]]) -> str:
    today = datetime.now().strftime("%Y-%m-%d %A")
    
    header = f"""# 📡 情报雷达日报
**日期**：{today}
**生成时间**：{datetime.now().strftime("%H:%M:%S")}
**数据来源**：{', '.join(src['name'] for src in RSS_SOURCES)}

---
"""
    sections = [header]
    
    print("\n🤖 开始 AI 摘要...")
    for category, articles in by_category.items():
        if not articles:
            continue
        section_title = f"\n## {category} ({len(articles)} 篇)\n\n"
        summary = summarize_articles(articles, category)
        sections.append(section_title + summary)
    
    # 添加原文链接附录
    sections.append("\n\n---\n## 🔗 原文链接\n")
    all_articles = [a for articles in by_category.values() for a in articles]
    for i, a in enumerate(all_articles, 1):
        sections.append(f"{i}. [{a['title']}]({a['link']}) — {a['source']}\n")
    
    return "\n".join(sections)


# ─────────────────────────────────────────────
#  4. 保存 + 发送
# ─────────────────────────────────────────────
def save_report(content: str) -> str:
    os.makedirs("output", exist_ok=True)
    date_str = datetime.now().strftime("%Y%m%d_%H%M")
    filepath = f"output/日报_{date_str}.md"
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(content)
    return filepath


def send_email(content: str, subject: str) -> bool:
    """发送邮件（可选）"""
    cfg = EMAIL_CONFIG
    if not cfg.get("smtp_server") or not cfg.get("sender"):
        return False
    
    try:
        msg = MIMEMultipart("alternative")
        msg["Subject"] = subject
        msg["From"] = cfg["sender"]
        msg["To"] = cfg["recipient"]
        msg.attach(MIMEText(content, "plain", "utf-8"))
        
        with smtplib.SMTP(cfg["smtp_server"], cfg.get("smtp_port", 587)) as server:
            server.starttls()
            server.login(cfg["sender"], cfg["password"])
            server.sendmail(cfg["sender"], cfg["recipient"], msg.as_string())
        return True
    except Exception as e:
        print(f"  ⚠️  邮件发送失败：{e}")
        return False


# ─────────────────────────────────────────────
#  5. 主任务
# ─────────────────────────────────────────────
def run_radar():
    print(f"\n{'='*50}")
    print(f"🚀 情报雷达启动 — {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print("="*50)
    
    # 拉取资讯
    by_category = fetch_all_sources()
    
    if not any(by_category.values()):
        print("⚠️  未获取到任何文章，跳过本次运行")
        return
    
    # 生成日报
    report = generate_daily_report(by_category)
    
    # 保存
    filepath = save_report(report)
    print(f"\n✅ 日报已保存：{filepath}")
    
    # 发送邮件（如果配置了）
    today = datetime.now().strftime("%Y-%m-%d")
    if send_email(report, f"📡 情报雷达日报 {today}"):
        print("📧 日报已发送至邮箱")
    
    print("\n" + "="*50)
    print(report[:500] + "...[截断]")
    print("="*50)


# ─────────────────────────────────────────────
#  6. 入口
# ─────────────────────────────────────────────
def main():
    parser = argparse.ArgumentParser(description="情报雷达")
    parser.add_argument("--now", action="store_true", help="立即执行一次")
    parser.add_argument("--time", default=SCHEDULE_TIME, help=f"定时执行时间（默认 {SCHEDULE_TIME}）")
    args = parser.parse_args()
    
    if args.now:
        run_radar()
        return
    
    print(f"⏰ 情报雷达已启动，每天 {args.time} 自动运行")
    print("Ctrl+C 停止\n")
    
    schedule.every().day.at(args.time).do(run_radar)
    
    # 启动时立即运行一次
    run_radar()
    
    while True:
        schedule.run_pending()
        time.sleep(60)


if __name__ == "__main__":
    main()
```

#### 运行方式

```bash
# 立即执行一次（测试）
python news-radar.py --now

# 启动定时任务（每天 08:00 自动运行）
python news-radar.py

# 自定义时间
python news-radar.py --time 09:30
```

***

### 🔧 封装产出（4 分钟）

```
news-radar/
├── news-radar.py         ✅ 定时脚本
├── config.py             ✅ RSS 源配置（随时增减）
└── output/
    └── 日报_*.md         ✅ 每日情报日报
```

**让脚本常驻运行**（macOS）：

```bash
# 方法：nohup 后台运行
nohup python news-radar.py > logs/radar.log 2>&1 &

# 查看日志
tail -f logs/radar.log
```

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

feedparser
├── feedparser.parse(url) 解析 RSS/Atom
├── feed.entries 获取文章列表
└── entry.title / .link / .summary / .published_parsed

定时任务
├── schedule.every().day.at("08:00").do(func)
├── schedule.run_pending()
└── nohup 后台常驻

Python 邮件发送
├── smtplib.SMTP + starttls()
├── MIMEMultipart("alternative")
└── server.sendmail()

设计模式
└── 配置与逻辑分离（config.py）
    方便随时更换 RSS 源，不改代码
```

***

### 🎯 课后作业

**练习 1（基础）**：在 `config.py` 中增加 3 个你关注领域的 RSS 源，定制属于自己的情报雷达。

**练习 2（进阶）**：支持微信公众号推送——用 Server 酱（https://sct.ftqq.com）的 Webhook API，将日报推送到微信。

**练习 3（挑战）**：给日报加上**关键词过滤**——读取用户定义的关键词列表（如 `["AI", "大模型", "Anthropic"]`），只保留包含关键词的文章，并在摘要中高亮关键词。

***

_第二阶段：产品设计 × Python 实战 · 第 17 课 / 共 10 课_
