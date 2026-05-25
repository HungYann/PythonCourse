# 第 14 课｜文档工厂：一句话指令生成完整 PRD

## 第二阶段：产品设计 × Python 实战

### 第 14 课｜文档工厂：一句话指令生成完整 PRD

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～13 课** 🛠️ **工具准备：Python 3.x、pip install anthropic jinja2**

***

### 课程导览

| 模块      | 内容                       | 时间    |
| ------- | ------------------------ | ----- |
| 🔥 热身回顾 | 第 13 课需求清洗的产出物           | 3 分钟  |
| 🎯 场景拆解 | PRD 生成痛点与模板 + Claude 解法  | 5 分钟  |
| 📚 核心知识 | Jinja2 模板、多轮对话、Prompt 工程 | 10 分钟 |
| 💻 代码实战 | 完整 prd-generator.py 开发   | 20 分钟 |
| 🔧 封装产出 | 函数库 + 可复用 PRD 模板         | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告             | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 13 课产出了结构化需求报告，核心痛点和优先级已经清晰了。

但产品经理还需要把需求转化为**完整 PRD**——背景、用户故事、功能规格、验收标准……这份文档往往要写 2-3 小时。

本课目标：输入一句话需求描述（或第 13 课的需求报告），10 分钟输出一份完整 PRD。

```
第 13 课产出          →    第 14 课产出
需求报告（痛点 + 优先级）  →  完整 PRD（用户故事 + 功能规格 + 验收标准）
```

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
需求评审通过了，现在要写 PRD：
  - 背景与目标：为什么做这个？
  - 用户故事：谁在什么场景下需要这个功能？
  - 功能规格：具体交互逻辑是什么？
  - 验收标准：开发怎么知道做完了？
  - 不在范围：什么不做？

写一份完整 PRD 通常需要 2-3 小时
本课方式：输入需求 → 10 分钟出草稿 → 产品经理审核修改 → 发出
```

#### 解决思路

```
需求描述（自然语言）
      │
      ▼
Jinja2 模板：把需求填入结构化 Prompt
      │
      ▼
Claude API（多轮对话）
  第 1 轮：生成 PRD 骨架（标题 + 用户故事 + 功能列表）
  第 2 轮：补充验收标准 + 边界说明
      │
      ▼
合并 → Markdown PRD 文档
      │
      ▼
output/PRD_功能名称_时间戳.md
```

***

### 📚 核心知识（10 分钟）

#### 1. Jinja2：Python 模板引擎

```bash
pip install jinja2
```

Jinja2 让你把变量嵌入字符串模板，避免手拼字符串：

```python
from jinja2 import Template

prompt_template = Template("""
你是一位资深产品经理。
请为以下需求编写完整 PRD：

**需求名称**：{{ feature_name }}
**需求描述**：{{ description }}
**目标用户**：{{ target_user }}
**背景**：{{ background }}
""")

prompt = prompt_template.render(
    feature_name="深色模式",
    description="用户希望 App 支持跟随系统的深色模式切换",
    target_user="夜间使用用户（占 DAU 的 35%）",
    background="竞品 Todoist、Things 3 均已支持，用户反馈中提及 23 次"
)
```

#### 2. 多轮对话（Messages 列表）

Claude API 支持传入历史对话，实现**多轮对话**：

```python
messages = [
    {"role": "user", "content": "请为深色模式写 PRD 骨架"},
]

# 第 1 轮
response1 = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=2000,
    system=SYSTEM_PROMPT,
    messages=messages
)
assistant_reply1 = response1.content[0].text

# 把第 1 轮结果加入历史
messages.append({"role": "assistant", "content": assistant_reply1})

# 第 2 轮：让 Claude 在第 1 轮基础上补充验收标准
messages.append({"role": "user", "content": "现在为每个功能点补充具体的验收标准（AC）"})

response2 = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=2000,
    system=SYSTEM_PROMPT,
    messages=messages
)
```

#### 3. PRD Prompt 设计原则

好的 PRD Prompt 应该明确：

* **角色**：你是产品经理，不是工程师
* **格式**：指定每个区块的标题和内容要求
* **约束**：不写技术实现细节（那是技术方案的事）

```python
SYSTEM_PROMPT = """你是一位经验丰富的产品经理，擅长编写清晰可执行的 PRD。

PRD 编写原则：
1. 用用户视角描述需求，不描述技术实现
2. 验收标准必须可量化、可验证
3. 明确写出"不在本期范围"，避免范围蔓延
4. 功能描述要具体到交互细节（点击什么 → 看到什么 → 发生什么）
"""
```

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
prd-generator/
├── prd-generator.py      ← 主程序
├── templates/
│   └── prd_prompt.j2    ← Jinja2 Prompt 模板
└── output/               ← 自动创建，存放 PRD 文件
```

#### 模板文件：templates/prd\_prompt.j2

```jinja2
请为以下产品需求编写完整 PRD：

## 需求基本信息
- **功能名称**：{{ feature_name }}
- **需求来源**：{{ source }}
- **目标用户**：{{ target_user }}
- **优先级**：{{ priority }}

## 需求背景
{{ background }}

## 核心需求描述
{{ description }}

{% if constraints %}
## 已知约束条件
{{ constraints }}
{% endif %}

请输出完整 PRD，包含以下所有区块：
1. 需求背景与目标
2. 用户故事（User Stories，至少 3 条）
3. 功能规格（详细交互描述）
4. 验收标准（AC，可量化可验证）
5. 不在本期范围
6. 相关指标（如何衡量功能是否成功）
```

#### 完整代码：prd-generator.py

```python
#!/usr/bin/env python3
"""
PRD 文档工厂 | prd-generator.py
用法：
  交互模式：python prd-generator.py
  快速模式：python prd-generator.py --feature "深色模式" --desc "跟随系统切换"
依赖：pip install anthropic jinja2
"""
import sys
import os
import argparse
from pathlib import Path
import anthropic
from jinja2 import Template
from datetime import datetime


SYSTEM_PROMPT = """你是一位经验丰富的产品经理，擅长编写清晰可执行的 PRD。

PRD 编写原则：
1. 用用户视角描述需求，不描述技术实现细节
2. 验收标准（AC）必须可量化、可验证，使用"Given-When-Then"格式
3. 明确写出"不在本期范围"，避免范围蔓延
4. 功能描述要具体到交互细节

输出格式：纯 Markdown，不加任何解释性开头。"""


PROMPT_TEMPLATE = Template("""
为以下产品需求编写完整 PRD：

## 需求基本信息
- **功能名称**：{{ feature_name }}
- **需求来源**：{{ source }}
- **目标用户**：{{ target_user }}
- **优先级**：{{ priority }}

## 需求背景
{{ background }}

## 核心需求描述
{{ description }}

{% if constraints %}
## 已知约束条件
{{ constraints }}
{% endif %}

请输出完整 PRD，包含：
1. 背景与目标（SMART 目标格式）
2. 用户故事（至少 3 条，格式：作为[用户]，我希望[行为]，以便[价值]）
3. 功能规格（每个功能点用"触发条件 → 系统行为 → 反馈"描述）
4. 验收标准（AC，每条：Given[前提] / When[操作] / Then[预期结果]）
5. 不在本期范围（明确写出不做什么）
6. 成功指标（如何用数据衡量功能上线效果）
""")


# ─────────────────────────────────────────────
#  1. 收集需求信息（交互模式）
# ─────────────────────────────────────────────
def collect_requirements_interactive() -> dict:
    """交互式收集需求信息"""
    print("\n📝 PRD 生成向导（直接回车跳过可选项）\n")
    
    feature_name = input("功能名称（必填）：").strip()
    if not feature_name:
        print("❌ 功能名称不能为空")
        sys.exit(1)
    
    description = input("需求描述（必填，一句话说清楚要做什么）：").strip()
    if not description:
        print("❌ 需求描述不能为空")
        sys.exit(1)
    
    target_user = input("目标用户（选填，如：夜间使用用户）：").strip() or "产品的目标用户"
    source = input("需求来源（选填，如：用户反馈 / 竞品对标）：").strip() or "产品规划"
    priority = input("优先级（选填，P0/P1/P2，默认 P1）：").strip() or "P1"
    background = input("背景补充（选填，为什么要做这个）：").strip() or ""
    constraints = input("约束条件（选填，如：不改动现有数据结构）：").strip() or ""
    
    return {
        "feature_name": feature_name,
        "description": description,
        "target_user": target_user,
        "source": source,
        "priority": priority,
        "background": background or f"用户提出了{feature_name}的需求",
        "constraints": constraints,
    }


# ─────────────────────────────────────────────
#  2. 生成 PRD（两轮对话）
# ─────────────────────────────────────────────
def generate_prd(requirements: dict) -> str:
    """两轮对话生成完整 PRD"""
    client = anthropic.Anthropic()
    
    # 第 1 轮：生成 PRD 主体
    prompt = PROMPT_TEMPLATE.render(**requirements)
    
    print("\n⏳ 第 1/2 轮：生成 PRD 主体...")
    messages = [{"role": "user", "content": prompt}]
    
    resp1 = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=2500,
        system=SYSTEM_PROMPT,
        messages=messages
    )
    prd_body = resp1.content[0].text
    
    # 第 2 轮：补充边界情况和测试建议
    messages.append({"role": "assistant", "content": prd_body})
    messages.append({
        "role": "user",
        "content": (
            "请在现有 PRD 末尾追加两个新区块：\n"
            "7. **边界情况处理**（至少 3 条，描述异常/边缘场景的处理方式）\n"
            "8. **测试建议**（QA 关注的核心测试场景，至少 4 条）\n"
            "只输出这两个新区块的内容，不要重复前面的内容。"
        )
    })
    
    print("⏳ 第 2/2 轮：补充边界情况和测试建议...")
    resp2 = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=1000,
        system=SYSTEM_PROMPT,
        messages=messages
    )
    prd_appendix = resp2.content[0].text
    
    # 合并
    full_prd = prd_body + "\n\n" + prd_appendix
    return full_prd


# ─────────────────────────────────────────────
#  3. 保存 PRD
# ─────────────────────────────────────────────
def save_prd(content: str, feature_name: str, output_dir: str = "output") -> str:
    os.makedirs(output_dir, exist_ok=True)
    safe_name = feature_name.replace("/", "_").replace(" ", "_")
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    filepath = os.path.join(output_dir, f"PRD_{safe_name}_{timestamp}.md")
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(content)
    return filepath


# ─────────────────────────────────────────────
#  4. 主程序
# ─────────────────────────────────────────────
def main():
    parser = argparse.ArgumentParser(description="PRD 文档工厂")
    parser.add_argument("--feature", help="功能名称（快速模式）")
    parser.add_argument("--desc", help="需求描述（快速模式）")
    parser.add_argument("--user", default="目标用户", help="目标用户")
    parser.add_argument("--priority", default="P1", help="优先级")
    args = parser.parse_args()
    
    # 收集需求
    if args.feature and args.desc:
        requirements = {
            "feature_name": args.feature,
            "description": args.desc,
            "target_user": args.user,
            "source": "命令行参数",
            "priority": args.priority,
            "background": f"用户需要{args.feature}功能",
            "constraints": "",
        }
        print(f"🚀 快速模式：{args.feature}")
    else:
        requirements = collect_requirements_interactive()
    
    # 生成 PRD
    prd_content = generate_prd(requirements)
    
    # 保存
    output_path = save_prd(prd_content, requirements["feature_name"])
    
    print(f"\n✅ PRD 已保存：{output_path}")
    print("\n" + "=" * 60)
    print(prd_content)
    print("=" * 60)


if __name__ == "__main__":
    main()
```

#### 运行效果

```bash
# 交互模式
python prd-generator.py

# 快速模式
python prd-generator.py --feature "深色模式" --desc "App 跟随系统设置自动切换深色/浅色主题"

# 控制台输出：
🚀 快速模式：深色模式

⏳ 第 1/2 轮：生成 PRD 主体...
⏳ 第 2/2 轮：补充边界情况和测试建议...

✅ PRD 已保存：output/PRD_深色模式_20260524_162015.md

============================================================
# PRD：深色模式

## 1. 背景与目标
**背景**：用户反馈（共 23 次提及）及竞品调研显示……
**目标**：90 天内将夜间 DAU 留存提升 8%，NPS 增加 5 分

## 2. 用户故事
- 作为夜间使用用户，我希望 App 跟随系统深色模式切换，以便减少眼睛疲劳
- 作为用户，我希望能手动强制切换深浅模式，以便在特定场景下控制显示效果
...
============================================================
```

***

### 🔧 封装产出（4 分钟）

本课交付物：

```
prd-generator/
├── prd-generator.py      ✅ 命令行工具（支持交互/快速两种模式）
├── templates/
│   └── prd_prompt.j2    ✅ 可复用 Prompt 模板（随时修改格式）
└── output/
    └── PRD_*.md          ✅ 完整 PRD 文档
```

**复用场景**：每次需求评审完，5 分钟输入核心信息，10 分钟出 PRD 草稿，产品经理只需审核修改。下节课我们把这份 PRD 变成**可点击的交互 Demo**。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

Jinja2 模板
├── Template(string).render(**vars)
├── {{ 变量 }} 插值
└── {% if %} 条件块

多轮对话
├── messages 列表累积历史
├── 第 1 轮生成主体
└── 第 2 轮追加补充（不重复）

Prompt 工程
├── System Prompt 定义角色和原则
├── User Prompt 通过模板结构化输入
└── 格式约束（Given-When-Then AC）

argparse 双模式
└── 快速模式（--flag）+ 交互模式（input()）兼容
```

***

### 🎯 课后作业

**练习 1（基础）**：在 `PROMPT_TEMPLATE` 中增加"风险与依赖"区块，让 Claude 识别功能上线的潜在风险点。

**练习 2（进阶）**：支持从第 13 课生成的需求报告（Markdown 文件）中自动提取 P0 需求，批量生成多份 PRD。

**练习 3（挑战）**：把 `prd-generator.py` 改造成 Python 函数库（不依赖命令行），让其他脚本可以 `from prd_generator import generate_prd` 调用，为第 20 课的工具箱做准备。

***

_第二阶段：产品设计 × Python 实战 · 第 14 课 / 共 10 课_
