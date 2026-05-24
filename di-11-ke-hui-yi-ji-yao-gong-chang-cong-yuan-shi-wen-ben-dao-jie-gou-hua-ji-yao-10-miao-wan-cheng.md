# 第 11 课｜会议纪要工厂：从原始文本到结构化纪要，10 秒完成

## 第二阶段：产品设计 × Python 实战

### 第 11 课｜会议纪要工厂：从原始文本到结构化纪要，10 秒完成

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～10 课（Python 基础）** 🛠️ **工具准备：Python 3.x、pip、Anthropic API Key**

***

### 课程导览

| 模块      | 内容                      | 时间    |
| ------- | ----------------------- | ----- |
| 🔥 热身回顾 | 第 10 课 Streamlit 与本课的关系 | 3 分钟  |
| 🎯 场景拆解 | 会议纪要痛点与 Claude API 解法   | 5 分钟  |
| 📚 核心知识 | Claude API 调用、Prompt 设计 | 10 分钟 |
| 💻 代码实战 | 完整 meeting-notes.py 开发  | 20 分钟 |
| 🔧 封装产出 | 命令行工具 + 输出文件            | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告            | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 10 课我们用 Streamlit 做了一个可视化仪表板——数据在"看"，但还在**被动等待**人工输入。

从第 11 课开始，我们进入**第二阶段：产品设计 × Python 实战**。每节课聚焦一个真实工作场景：

```
第一阶段（01～10）                    第二阶段（11～20）
─────────────────────               ──────────────────────────────
学 Python 语法                →      用 Python 解决真实工作问题
print / input / class         →      Claude API / Pandas / 爬虫
程序自己跑                    →      接入 AI，程序会"思考"
```

今天的场景：**会议纪要**——每周最常见、最耗时的文书任务之一。

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
一次 1 小时的产品评审会结束后，你手头有：
├── 一段 600 字的录音转写文本（错别字多、没有格式）
├── 几条 Slack 消息里散落的关键决策
└── 脑子里还记着的两个行动项

现在需要在 30 分钟内发出一份规范纪要给所有参会人……
```

**传统方式**：手动复制、整理、排版 → 30 分钟\
**本课方式**：Python 脚本 + Claude API → **10 秒**

#### 解决思路

```
原始文本 (txt)
    │
    ▼
Python 读取文件
    │
    ▼
构建 Prompt（告诉 Claude 输出格式）
    │
    ▼
Claude API 调用（claude-opus-4-7）
    │
    ▼
结构化 Markdown 纪要
    │
    ▼
保存到 output/ 目录
```

***

### 📚 核心知识（10 分钟）

#### 1. 安装 Anthropic SDK

```bash
pip install anthropic
```

#### 2. Claude API 最小调用示例

```python
import anthropic

# 自动读取环境变量 ANTHROPIC_API_KEY
client = anthropic.Anthropic()

message = client.messages.create(
    model="claude-opus-4-7",      # 使用最新模型
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "用一句话解释什么是会议纪要"}
    ]
)

print(message.content[0].text)
```

#### 3. System Prompt vs User Message

| 角色          | 用途                     | 比喻      |
| ----------- | ---------------------- | ------- |
| `system`    | 告诉 Claude "你是谁、遵守什么规则" | 岗位职责说明书 |
| `user`      | 本次具体要处理的内容             | 这次的工单   |
| `assistant` | Claude 的回复             | 工单处理结果  |

```python
message = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=2048,
    system="你是专业会议纪要助手，只输出 Markdown 格式，不加任何解释。",  # ← system
    messages=[
        {"role": "user", "content": "原始文本：" + raw_text}               # ← user
    ]
)
```

#### 4. 环境变量设置

```bash
# macOS / Linux（当前终端生效）
export ANTHROPIC_API_KEY="sk-ant-..."

# 永久生效（加入 ~/.zshrc 或 ~/.bashrc）
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.zshrc
source ~/.zshrc

# Windows PowerShell
$env:ANTHROPIC_API_KEY = "sk-ant-..."
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
meeting-notes/
├── meeting-notes.py      ← 主程序
├── samples/
│   └── sample.txt        ← 测试用原始文本
└── output/               ← 自动创建，存放生成的纪要
```

#### 完整代码：meeting-notes.py

```python
#!/usr/bin/env python3
"""
会议纪要工厂 | meeting-notes.py
用法：python meeting-notes.py <原始文本文件>
示例：python meeting-notes.py samples/sample.txt
依赖：pip install anthropic
"""
import sys
import os
import anthropic
from datetime import datetime


# ─────────────────────────────────────────────
#  1. 读取原始文本
# ─────────────────────────────────────────────
def load_raw_text(filepath: str) -> str:
    """读取任意 .txt 文件，返回字符串"""
    if not os.path.exists(filepath):
        print(f"❌ 文件不存在：{filepath}")
        sys.exit(1)
    with open(filepath, "r", encoding="utf-8") as f:
        return f.read()


# ─────────────────────────────────────────────
#  2. 调用 Claude API 生成结构化纪要
# ─────────────────────────────────────────────
SYSTEM_PROMPT = """你是一位专业的会议纪要助手。
请将用户提供的原始会议文本整理为规范 Markdown 格式的会议纪要。

输出必须包含以下结构（无论原文是否完整）：
## 会议纪要 — [主题] — [日期]

**时间**：[从文本中提取，无则写"待确认"]
**参与者**：[从文本中提取姓名]
**会议目的**：[一句话概括]

### 📋 讨论要点
- [要点，每条简短清晰]

### ✅ 关键决策
| 决策内容 | 负责人 | 截止日期 |
|---------|--------|---------|
| [决策] | [人名] | [日期或"待定"] |

### 🎯 行动项
- [ ] [具体任务] — @[负责人] — [截止日期]

### ❓ 待确认事项
- [需要跟进的问题，没有则写"无"]

---
只输出 Markdown，不加任何说明性文字。"""


def generate_meeting_notes(raw_text: str) -> str:
    """调用 Claude API，返回结构化纪要 Markdown"""
    client = anthropic.Anthropic()
    
    print("⏳ 正在调用 Claude API 生成纪要...")
    
    message = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=2048,
        system=SYSTEM_PROMPT,
        messages=[
            {"role": "user", "content": f"请整理以下会议原始文本：\n\n{raw_text}"}
        ]
    )
    
    return message.content[0].text


# ─────────────────────────────────────────────
#  3. 保存纪要到文件
# ─────────────────────────────────────────────
def save_notes(content: str, output_dir: str = "output") -> str:
    """将纪要保存为带时间戳的 Markdown 文件"""
    os.makedirs(output_dir, exist_ok=True)
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = os.path.join(output_dir, f"纪要_{timestamp}.md")
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(content)
    return filepath


# ─────────────────────────────────────────────
#  4. 主程序入口
# ─────────────────────────────────────────────
def main():
    if len(sys.argv) < 2:
        print("用法：python meeting-notes.py <原始文本文件>")
        print("示例：python meeting-notes.py samples/sample.txt")
        sys.exit(1)
    
    # Step 1: 读取原始文本
    raw_text = load_raw_text(sys.argv[1])
    print(f"📄 已读取原始文本（{len(raw_text)} 字）")
    
    # Step 2: 生成纪要
    notes = generate_meeting_notes(raw_text)
    
    # Step 3: 保存文件
    output_path = save_notes(notes)
    
    # Step 4: 打印结果
    print(f"\n✅ 纪要已保存：{output_path}")
    print("\n" + "=" * 60)
    print(notes)
    print("=" * 60)


if __name__ == "__main__":
    main()
```

#### 测试用原始文本（samples/sample.txt）

```
今天下午两点开了个产品需求评审，参加的有产品经理小红、
前端工程师小明、后端工程师老王，还有设计师小美。

主要讨论了下季度的三个需求：
第一个是用户登录改造，支持微信扫码和手机验证码两种方式，
优先级最高，小明负责，定在下周五上线。

第二个是数据看板，要展示日活、留存、收入三个核心指标，
老王负责后端接口，小红写 PRD，三周内搞定。

第三个是暗黑模式，小美说设计稿已经出了一半，
但前端排期满了，这个先搁置，等下下季度再说。

另外老王提到现在线上有个偶发的登录 Bug，
用户反馈说微信授权后有时候会白屏，
下周一需要定位问题，老王和小明一起排查。

下次开会定在下周四上午十点，地点会议室 B。
```

#### 运行效果

```bash
python meeting-notes.py samples/sample.txt

# 输出：
📄 已读取原始文本（312 字）
⏳ 正在调用 Claude API 生成纪要...
✅ 纪要已保存：output/纪要_20260524_143025.md

============================================================
## 会议纪要 — 产品需求评审会 — 2026-05-24

**时间**：下午两点
**参与者**：小红（产品经理）、小明（前端）、老王（后端）、小美（设计）
**会议目的**：评审下季度三个产品需求并分配责任人

### 📋 讨论要点
- 用户登录改造：支持微信扫码 + 手机验证码
- 数据看板：展示日活、留存、收入三项指标
- 暗黑模式：设计稿完成一半，暂缓排期
- 线上登录 Bug：微信授权后偶发白屏

### ✅ 关键决策
| 决策内容 | 负责人 | 截止日期 |
|---------|--------|---------|
| 登录改造上线 | 小明 | 下周五 |
| 数据看板开发 | 老王（接口）+ 小红（PRD） | 三周内 |
| 暗黑模式 | 暂缓 | 下下季度 |

### 🎯 行动项
- [ ] 完成登录改造（微信 + 手机验证码）— @小明 — 下周五
- [ ] 编写数据看板 PRD — @小红 — 待定
- [ ] 开发数据看板后端接口 — @老王 — 三周内
- [ ] 排查线上登录白屏 Bug — @老王 + @小明 — 下周一

### ❓ 待确认事项
- 数据看板前端排期待确认
- 下次会议：下周四上午 10 点，会议室 B
============================================================
```

***

### 🔧 封装产出（4 分钟）

本课交付物：

```
meeting-notes/
├── meeting-notes.py     ✅ 命令行工具（可直接运行）
├── samples/
│   └── sample.txt       ✅ 测试样本
└── output/
    └── 纪要_*.md        ✅ 生成的结构化纪要
```

**复用场景**：每次开完会，把录音转写或聊天记录保存为 `.txt`，一行命令搞定。下节课我们会把这个脚本进一步封装成 Skill，用触发词自动调用。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

文件操作
└── open() + read() → 读取任意 .txt 文件

Claude API
├── anthropic.Anthropic() 初始化客户端
├── client.messages.create(model, system, messages)
├── System Prompt 控制输出格式
└── message.content[0].text 提取结果

工程习惯
├── os.makedirs(exist_ok=True) 安全创建目录
├── 时间戳命名输出文件，避免覆盖
└── sys.argv 接收命令行参数
```

***

### 🎯 课后作业

**练习 1（基础）**：修改 `SYSTEM_PROMPT`，让输出额外增加一个 "## 💡 会议效率评估" 区块，包含：会议是否有明确结论（是/否）、行动项是否都有负责人（是/否）。

**练习 2（进阶）**：支持批量处理——遍历 `samples/` 目录下所有 `.txt` 文件，为每个文件生成对应纪要，文件名保持对应关系（如 `samples/proj_a.txt` → `output/纪要_proj_a_时间戳.md`）。

**练习 3（挑战）**：增加 `--lang` 参数，支持 `--lang en` 输出英文纪要。提示：在 `system` prompt 末尾加上语言要求即可。

***

_第二阶段：产品设计 × Python 实战 · 第 11 课 / 共 40 课_&#x20;
