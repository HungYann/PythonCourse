# 第八课：模块与包、pip 使用

### 第八课：Python 基础（从零起飞）- 模块与包、pip 使用

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01～07 课** 🛠️ **工具准备：Python 3.x、终端（Terminal / 命令提示符）、**[**pip**](https://pip.pypa.io)

***

### 课程导览

| 模块           | 内容                                 | 时间    |
| ------------ | ---------------------------------- | ----- |
| 🔥 热身回顾      | 代码复用的三个层次：函数 → 模块 → 包              | 3 分钟  |
| 📦 模块基础      | import、from...import、as 别名、自定义模块   | 10 分钟 |
| 🗂️ 包与项目结构   | 包的概念、`__init__.py`、相对导入            | 7 分钟  |
| 🔧 常用标准库     | random、datetime、json、os、math       | 8 分钟  |
| 🌐 pip 与第三方库 | pip 安装/卸载/查询、虚拟环境、requirements.txt | 7 分钟  |
| 🏁 综合练习      | JSON 成绩管理系统 + 第三方库初体验              | 5 分钟  |

***

### 🔥 热身回顾（3 分钟）

前七课我们用**函数**解决了代码重复的问题。但如果有几百个函数，全写在一个文件里会变得混乱难维护——这时就需要**模块**：

```
代码复用的三个层次：

函数（第 6 课）         模块（本课）              包（本课）
─────────────          ──────────────────        ─────────────────────
def greet():    →      grade_utils.py       →    my_project/
    ...                ├── def get_grade()        ├── utils/
                       ├── def calc_stats()       │   ├── grade.py
                       └── def print_report()     │   └── file_io.py
                                                  ├── main.py
                                                  └── __init__.py
```

第七课的成绩系统文件操作函数，今天我们把它们整理成可复用的模块。同时学会用 `pip` 调用全球开发者写好的第三方库——**不重复造轮子**。

***

### 📦 模块基础（10 分钟）

#### 什么是模块？

模块就是一个 `.py` 文件。任何 Python 文件都可以作为模块被其他文件导入使用。

```
my_project/
├── math_utils.py   ← 这就是一个模块
└── main.py         ← 主程序，导入并使用模块
```

```python
# math_utils.py（模块文件）
PI = 3.14159

def circle_area(r):
    """计算圆面积"""
    return PI * r ** 2

def circle_perimeter(r):
    """计算圆周长"""
    return 2 * PI * r
```

***

#### import：导入整个模块

```python
# main.py
import math_utils           # 导入整个模块

print(math_utils.PI)                     # 访问模块变量：3.14159
print(math_utils.circle_area(5))         # 调用模块函数：78.53975
print(math_utils.circle_perimeter(5))   # 31.4159
```

***

#### from ... import：导入指定内容

```python
# 只导入需要的内容，使用时不需要模块名前缀
from math_utils import circle_area, PI

print(PI)               # 3.14159（直接用，不需要 math_utils.PI）
print(circle_area(5))   # 78.53975
```

```python
# 导入全部内容（不推荐：容易造成名称冲突）
from math_utils import *

print(circle_perimeter(3))   # 可以直接用，但不清楚来源
```

***

#### as：给模块或名称起别名

```python
# 给模块起别名（长模块名常用）
import math_utils as mu

print(mu.circle_area(5))     # 输出：78.53975

# 给导入的名称起别名
from math_utils import circle_area as area

print(area(5))               # 输出：78.53975
```

> 💡 **import vs from...import 如何选？**
>
> ```
> import module          → 命名空间清晰，知道函数来自哪里（推荐）
> from module import f   → 使用频繁时方便，但大量导入时来源不清晰
> from module import *   → 尽量避免，污染命名空间
> ```

***

#### 自定义模块：`__name__` 的作用

每个模块都有一个 `__name__` 属性。**直接运行时** `__name__ == "__main__"`，**被导入时** `__name__` 是模块文件名：

```python
# grade_utils.py

def get_grade(score):
    """根据分数返回等级"""
    if score >= 90:   return "优秀 ⭐"
    elif score >= 75: return "良好 ✅"
    elif score >= 60: return "及格 📝"
    else:             return "不及格 ❌"


def calc_average(scores):
    """计算平均分"""
    return sum(scores) / len(scores) if scores else 0


# ✅ 关键写法：只有直接运行该文件时才执行测试代码
if __name__ == "__main__":
    # 这里写测试代码，被 import 时不会执行
    print("=== 模块自测 ===")
    print(get_grade(92))    # 优秀 ⭐
    print(get_grade(55))    # 不及格 ❌
    print(calc_average([90, 85, 72]))  # 82.33...
```

```python
# main.py：导入时，grade_utils 的 if __name__ 块不会执行
import grade_utils

score = 88
print(grade_utils.get_grade(score))   # 良好 ✅
```

**`__name__` 值对照：**

```
运行方式                    __name__ 的值
──────────────────────     ──────────────
python grade_utils.py  →   "__main__"
import grade_utils     →   "grade_utils"
```

***

#### 模块搜索路径

Python 导入模块时按以下顺序查找：

```
1. 当前目录
2. 环境变量 PYTHONPATH 中的目录
3. Python 标准库目录
4. site-packages（pip 安装的第三方库）
```

```python
import sys
print(sys.path)    # 查看完整搜索路径列表
```

***

### 🗂️ 包与项目结构（7 分钟）

#### 什么是包？

包是**含有 `__init__.py` 文件的目录**，用于组织多个相关模块：

```
score_project/
├── main.py                  ← 主入口
├── utils/                   ← 这是一个包
│   ├── __init__.py          ← 标识这是包（可以为空）
│   ├── grade.py             ← 成绩工具模块
│   └── file_io.py           ← 文件操作模块
└── data/
    └── scores.csv
```

```python
# utils/grade.py
def get_grade(score):
    if score >= 90: return "优秀"
    elif score >= 75: return "良好"
    elif score >= 60: return "及格"
    else: return "不及格"
```

```python
# utils/file_io.py
import os

def load_csv(filepath):
    if not os.path.exists(filepath):
        return {}
    data = {}
    with open(filepath, "r", encoding="utf-8") as f:
        for line in f:
            name, score = line.strip().split(",")
            data[name] = int(score)
    return data
```

```python
# main.py：导入包中的模块
from utils.grade import get_grade
from utils.file_io import load_csv

scores = load_csv("data/scores.csv")
for name, score in scores.items():
    print(f"{name}：{get_grade(score)}")
```

***

#### `__init__.py` 的用途

`__init__.py` 在包被导入时自动执行，可以用来简化导入路径：

```python
# utils/__init__.py（配置对外暴露的接口）
from .grade import get_grade
from .file_io import load_csv, save_csv

# 现在 main.py 可以这样导入（更简洁）
# from utils import get_grade, load_csv
```

```python
# main.py（简化后的导入）
from utils import get_grade, load_csv    # 不需要写完整路径

scores = load_csv("data/scores.csv")
print(get_grade(88))
```

***

### 🔧 常用标准库（8 分钟）

Python 内置了大量"开箱即用"的标准库，无需安装直接 `import`：

***

#### random：随机数

```python
import random

# 生成随机整数
print(random.randint(1, 100))         # 1 到 100 随机整数

# 生成随机浮点数
print(random.random())                # 0.0 到 1.0

# 从序列中随机选择
fruits = ["苹果", "香蕉", "橙子", "葡萄"]
print(random.choice(fruits))          # 随机选一个
print(random.sample(fruits, 2))       # 随机选 2 个（不重复）

# 随机打乱列表（原地修改）
numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)
print(numbers)                        # 输出：[3, 1, 5, 2, 4]（每次不同）

# 固定随机种子（让结果可复现，调试时有用）
random.seed(42)
print(random.randint(1, 100))         # 固定输出：52
```

***

#### datetime：日期与时间

```python
import datetime

# 获取今天的日期
today = datetime.date.today()
print(today)                          # 输出：2024-03-15

# 获取当前时间
now = datetime.datetime.now()
print(now)                            # 输出：2024-03-15 14:30:25.123456

# 格式化输出
print(now.strftime("%Y年%m月%d日 %H:%M:%S"))
# 输出：2024年03月15日 14:30:25

# 字符串解析为日期
d = datetime.datetime.strptime("2024-01-15", "%Y-%m-%d")
print(d.year, d.month, d.day)         # 2024 1 15

# 日期计算
one_week = datetime.timedelta(days=7)
next_week = today + one_week
print(f"一周后：{next_week}")

deadline = datetime.date(2024, 12, 31)
days_left = (deadline - today).days
print(f"距离年底还有 {days_left} 天")
```

***

#### json：数据序列化

JSON 是比 CSV 更灵活的数据格式，支持嵌套结构：

```python
import json

# Python 对象 → JSON 字符串
data = {
    "students": [
        {"name": "小明", "score": 92, "passed": True},
        {"name": "小红", "score": 85, "passed": True},
    ],
    "total": 2
}

json_str = json.dumps(data, ensure_ascii=False, indent=2)
print(json_str)
```

```json
{
  "students": [
    {
      "name": "小明",
      "score": 92,
      "passed": true
    },
    ...
  ],
  "total": 2
}
```

```python
# JSON 字符串 → Python 对象
parsed = json.loads(json_str)
print(parsed["students"][0]["name"])   # 输出：小明

# 写入 JSON 文件
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# 读取 JSON 文件
with open("data.json", "r", encoding="utf-8") as f:
    loaded = json.load(f)
print(loaded["total"])                 # 输出：2
```

**JSON ↔ Python 类型对照：**

| JSON             | Python           |
| ---------------- | ---------------- |
| `object {}`      | `dict`           |
| `array []`       | `list`           |
| `string`         | `str`            |
| `number`         | `int` / `float`  |
| `true` / `false` | `True` / `False` |
| `null`           | `None`           |

***

#### math：数学运算

```python
import math

print(math.pi)            # 3.141592653589793
print(math.e)             # 2.718281828459045
print(math.sqrt(16))      # 4.0（平方根）
print(math.floor(3.7))    # 3（向下取整）
print(math.ceil(3.2))     # 4（向上取整）
print(math.log(100, 10))  # 2.0（以 10 为底的对数）
print(math.factorial(5))  # 120（5 的阶乘）
print(math.gcd(12, 8))    # 4（最大公约数）
```

***

#### 标准库速查

| 库             | 主要用途                        |
| ------------- | --------------------------- |
| `random`      | 随机数生成、随机选择、洗牌               |
| `datetime`    | 日期时间处理、格式化、计算               |
| `json`        | JSON 数据读写与序列化               |
| `os`          | 文件路径、目录操作、环境变量              |
| `math`        | 数学函数（sqrt、log、floor 等）      |
| `sys`         | Python 解释器信息、命令行参数          |
| `re`          | 正则表达式匹配与替换                  |
| `collections` | 高级数据结构（Counter、defaultdict） |
| `itertools`   | 迭代器工具（排列组合、无限序列）            |
| `pathlib`     | 面向对象的路径操作（现代写法）             |

***

### 🌐 pip 与第三方库（7 分钟）

#### pip 是什么？

`pip` 是 Python 的包管理器，用于从 [PyPI](https://pypi.org)（Python 包索引，拥有 50 万+ 第三方库）安装、升级、卸载包。

***

#### 常用 pip 命令

```bash
# 安装包
pip install requests               # 安装最新版
pip install requests==2.31.0       # 安装指定版本
pip install requests>=2.28         # 安装 >=2.28 的版本

# 升级包
pip install --upgrade requests

# 卸载包
pip uninstall requests

# 查看已安装的包
pip list

# 查看某个包的详情（版本、依赖等）
pip show requests

# 搜索包（旧版 pip 支持，新版建议直接去 pypi.org 搜索）
pip search keyword

# 导出当前环境的所有依赖（用于分享项目）
pip freeze > requirements.txt

# 从文件批量安装依赖
pip install -r requirements.txt
```

> 💡 **国内用户加速：** 默认从国外服务器下载，可能较慢。临时使用国内镜像：
>
> ```bash
> pip install requests -i https://pypi.tuna.tsinghua.edu.cn/simple
> ```
>
> 永久设置（推荐）：
>
> ```bash
> pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
> ```

***

#### 虚拟环境：隔离项目依赖

不同项目可能需要同一个库的不同版本，**虚拟环境**让每个项目拥有独立的依赖空间：

```bash
# 创建虚拟环境
python -m venv myenv          # 在当前目录创建 myenv 文件夹

# 激活虚拟环境
source myenv/bin/activate      # Mac / Linux
myenv\Scripts\activate         # Windows

# 激活后命令行前会出现 (myenv) 前缀
(myenv) $ pip install requests   # 安装到虚拟环境，不影响全局

# 退出虚拟环境
deactivate
```

**虚拟环境的意义：**

```
没有虚拟环境                      有虚拟环境
──────────────────────────       ─────────────────────────────
全局 Python                      project_A/  ← requests 2.28
├── requests 2.31 (项目A用)      project_B/  ← requests 2.31
└── requests 2.28 (项目B用)      → 两个项目互不干扰 ✅
→ 版本冲突，无法共存 ❌
```

***

#### requirements.txt：依赖声明文件

```bash
# 导出当前环境依赖
pip freeze > requirements.txt
```

```
# requirements.txt 内容示例
requests==2.31.0
pandas==2.1.0
numpy==1.26.0
```

```bash
# 在新环境中一键安装所有依赖
pip install -r requirements.txt
```

> 💡 **项目工作流：** 创建虚拟环境 → 安装依赖 → 开发 → `pip freeze > requirements.txt` → 提交到 Git → 他人克隆后 `pip install -r requirements.txt` 即可复现环境。

***

#### 常用第三方库推荐

| 库            | 用途                | 安装命令                     |
| ------------ | ----------------- | ------------------------ |
| `requests`   | HTTP 请求，爬虫/API 调用 | `pip install requests`   |
| `pandas`     | 数据分析与处理           | `pip install pandas`     |
| `numpy`      | 数值计算、矩阵运算         | `pip install numpy`      |
| `matplotlib` | 数据可视化、图表绘制        | `pip install matplotlib` |
| `flask`      | 轻量级 Web 框架        | `pip install flask`      |
| `fastapi`    | 现代高性能 Web API     | `pip install fastapi`    |
| `sqlalchemy` | 数据库 ORM           | `pip install sqlalchemy` |
| `pillow`     | 图像处理              | `pip install pillow`     |
| `rich`       | 终端美化输出            | `pip install rich`       |
| `pytest`     | 单元测试框架            | `pip install pytest`     |

***

### 🏁 综合练习（5 分钟）

#### 练习：JSON 成绩管理系统升级版

将第七课的 CSV 成绩系统升级为 JSON 格式，并用 `datetime` 记录操作时间，同时初体验第三方库 `rich` 的美化终端输出：

```python
# ============================================
# JSON 成绩管理系统（模块化 + 标准库）
# ============================================
import json
import os
import datetime

DATA_FILE = "scores.json"


def load_data():
    """从 JSON 文件加载数据"""
    if not os.path.exists(DATA_FILE):
        return {"records": [], "last_updated": None}
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except (json.JSONDecodeError, IOError) as e:
        print(f"⚠️ 数据文件损坏，重置：{e}")
        return {"records": [], "last_updated": None}


def save_data(data):
    """将数据保存到 JSON 文件"""
    data["last_updated"] = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"✅ 已保存（{data['last_updated']}）")


def add_record(data, name, score):
    """添加成绩记录"""
    try:
        score = int(score)
        if not (0 <= score <= 100):
            raise ValueError("成绩须在 0～100")
    except ValueError as e:
        print(f"❌ 输入有误：{e}")
        return

    # 检查是否已存在，存在则更新
    for record in data["records"]:
        if record["name"] == name:
            record["score"] = score
            record["updated"] = datetime.date.today().isoformat()
            print(f"✅ 已更新：{name} → {score} 分")
            return

    data["records"].append({
        "name": name,
        "score": score,
        "updated": datetime.date.today().isoformat()
    })
    print(f"✅ 已新增：{name} → {score} 分")


def show_data(data):
    """展示所有成绩记录"""
    records = data["records"]
    if not records:
        print("📭 暂无记录。")
        return

    sorted_records = sorted(records, key=lambda r: r["score"], reverse=True)
    last = data.get("last_updated", "未知")

    print(f"\n{'=' * 38}")
    print(f"{'📊 成绩报告':^34}")
    print(f"  最后保存：{last}")
    print(f"{'=' * 38}")
    for i, r in enumerate(sorted_records, 1):
        bar = "█" * (r["score"] // 10)
        print(f"  {i}. {r['name']:<6} {r['score']:>3} 分  {bar}")
    print(f"{'=' * 38}\n")


def main():
    data = load_data()
    print(f"📂 已加载 {len(data['records'])} 条记录。\n")

    while True:
        print("1. 录入  2. 查看  3. 保存退出")
        choice = input("请选择：").strip()
        if choice == "1":
            name  = input("  姓名：").strip()
            score = input("  成绩：").strip()
            add_record(data, name, score)
        elif choice == "2":
            show_data(data)
        elif choice == "3":
            save_data(data)
            break
        else:
            print("⚠️ 请输入 1、2 或 3。")


main()
```

***

### 📝 本课知识总结

```
模块
├── import module              → 导入整个模块，用 module.func() 调用
├── from module import func    → 导入指定内容，直接用 func() 调用
├── import module as alias     → 给模块起别名
└── if __name__ == "__main__": → 模块自测代码块，被导入时不执行

包
├── 含 __init__.py 的目录即为包
├── from package.module import func  → 导入包中的模块
└── __init__.py 可配置对外接口，简化导入路径

常用标准库
├── random   → 随机数、random.choice()、shuffle()
├── datetime → 日期时间、strftime()、timedelta
├── json     → json.dumps/loads（字符串）/ json.dump/load（文件）
├── math     → sqrt、pi、floor、ceil、factorial
└── os       → path.exists、path.join、makedirs、listdir

pip
├── pip install 包名           → 安装
├── pip uninstall 包名         → 卸载
├── pip list                   → 查看已安装
├── pip freeze > requirements.txt  → 导出依赖
└── pip install -r requirements.txt → 批量安装

虚拟环境
├── python -m venv 环境名      → 创建
├── source 环境名/bin/activate → 激活（Mac/Linux）
├── 环境名\Scripts\activate    → 激活（Windows）
└── deactivate                 → 退出
```

***

### 🎯 课后作业

**练习 1（基础）：** 创建自己的工具模块 `my_utils.py`：

* 在模块中定义至少 3 个函数：`celsius_to_fahrenheit(c)`、`is_prime(n)`、`reverse_string(s)`
* 在同一文件末尾用 `if __name__ == "__main__":` 写测试
* 在 `main.py` 中用 `import my_utils` 导入并调用

**练习 2（进阶）：** 用 `datetime` 和 `json` 写一个"日记本"程序：

* 每次运行显示今天日期
* 用户可以写入当天日记（存储到 `diary.json`，以日期为键）
* 可以查看历史某天的日记（输入日期字符串）
* 日记以 JSON 格式持久化，程序重启后数据不丢失

**练习 3（挑战）：** 安装并体验 `requests` 库：

* `pip install requests`
* 调用公开 API：`https://api.github.com/users/python`
* 用 `json` 解析返回的数据，打印 Python 官方 GitHub 账号的公开仓库数量、粉丝数等信息
* 用 `try/except` 处理网络请求失败的情况

***

### 🔜 下节课预告

**第九课：面向对象编程（OOP）基础**

```python
# 你将学会用"类"来建模现实世界的事物：

class Student:
    """学生类"""
    def __init__(self, name, age):
        self.name = name        # 实例属性
        self.age  = age
        self.scores = []

    def add_score(self, score):
        """添加成绩"""
        self.scores.append(score)

    def average(self):
        """计算平均分"""
        return sum(self.scores) / len(self.scores) if self.scores else 0

    def __str__(self):
        return f"学生({self.name}, {self.age}岁, 均分:{self.average():.1f})"


s = Student("小明", 18)
s.add_score(92)
s.add_score(85)
print(s)   # 学生(小明, 18岁, 均分:88.5)
```

***

> 💬 **有疑问？** 欢迎在评论区留言或加入学习群讨论。
>
> ⭐ 觉得有帮助的话，别忘了分享给朋友！

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 8 课 / 共 10 课 Power by Andrew Liu_
