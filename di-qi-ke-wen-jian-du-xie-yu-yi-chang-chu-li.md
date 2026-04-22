# 第七课：文件读写与异常处理

### 第七课：Python 基础（从零起飞）- 文件读写与异常处理

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01～06 课** 🛠️ **工具准备：Python 3.x 或** [**在线运行环境**](https://colab.research.google.com/)

***

### 课程导览

| 模块         | 内容                            | 时间    |
| ---------- | ----------------------------- | ----- |
| 🔥 热身回顾    | 数据为什么需要保存到文件？                 | 3 分钟  |
| 📂 文件读写基础  | open()、读取模式、写入模式、with 语句      | 10 分钟 |
| 📋 文件操作进阶  | 按行读写、CSV 处理、路径操作              | 8 分钟  |
| 🛡️ 异常处理基础 | try / except / else / finally | 10 分钟 |
| ⚡ 异常进阶     | 常见异常类型、raise、自定义异常            | 4 分钟  |
| 🏁 综合练习    | 成绩持久化系统（文件 + 异常）              | 5 分钟  |

***

### 🔥 热身回顾（3 分钟）

第六课写的成绩管理系统有一个致命缺陷——**程序一关闭，数据就消失了**。每次运行都要重新输入所有数据：

```python
# 第六课的成绩数据：写死在代码里，无法持久化
scores = {
    "小明": 92, "小红": 85, "小李": 73,
    "小张": 95, "小王": 68, "小赵": 55,
}
# 程序退出 → 数据全部丢失 💀
```

本课学习**文件读写**，让数据在程序关闭后依然存在；同时学习**异常处理**，让程序面对错误时不崩溃，优雅地恢复：

```
没有文件 & 异常处理              有了文件 & 异常处理
──────────────────────          ──────────────────────────────
数据只在内存中                   数据保存到 .txt / .csv 文件
程序崩溃 → 整个退出          →  发生错误 → 捕获 → 提示用户
文件不存在 → 报错退出            文件不存在 → 自动创建
```

***

### 📂 文件读写基础（10 分钟）

#### open()：打开文件的钥匙

一切文件操作都从 `open()` 开始：

```python
# 语法
文件对象 = open(文件路径, 模式, encoding="utf-8")
```

**常用模式速查表：**

| 模式     | 含义     | 文件不存在时 | 文件已存在时   |
| ------ | ------ | ------ | -------- |
| `"r"`  | 只读（默认） | ❌ 报错   | ✅ 读取     |
| `"w"`  | 只写     | ✅ 创建   | ⚠️ 清空后写入 |
| `"a"`  | 追加     | ✅ 创建   | ✅ 在末尾添加  |
| `"r+"` | 读写     | ❌ 报错   | ✅ 读写     |
| `"x"`  | 独占创建   | ✅ 创建   | ❌ 报错     |

> ⚠️ **处理中文文件必须指定 `encoding="utf-8"`**，否则在 Windows 上会出现乱码。

***

#### with 语句：推荐写法

`with` 会在代码块结束后**自动关闭文件**，无论是否发生异常，是操作文件的最佳实践：

```python
# ❌ 旧写法：容易忘记关闭，出错时文件可能损坏
f = open("hello.txt", "w", encoding="utf-8")
f.write("你好，世界！")
f.close()    # 必须手动关闭，一旦出错就漏掉了

# ✅ 推荐：with 语句自动关闭
with open("hello.txt", "w", encoding="utf-8") as f:
    f.write("你好，世界！")
# 离开 with 块后文件自动关闭，无论是否报错
```

***

#### 写入文件

```python
# write()：写入字符串
with open("note.txt", "w", encoding="utf-8") as f:
    f.write("第一行内容\n")       # \n 是换行符
    f.write("第二行内容\n")
    f.write("第三行内容\n")

# writelines()：写入字符串列表（不自动加换行）
lines = ["苹果\n", "香蕉\n", "橙子\n"]
with open("fruits.txt", "w", encoding="utf-8") as f:
    f.writelines(lines)

# 追加模式：不覆盖原内容，在末尾添加
with open("note.txt", "a", encoding="utf-8") as f:
    f.write("这是追加的第四行\n")
```

***

#### 读取文件

```python
# read()：一次性读取全部内容（返回字符串）
with open("note.txt", "r", encoding="utf-8") as f:
    content = f.read()
    print(content)
```

```python
# readline()：每次读取一行
with open("note.txt", "r", encoding="utf-8") as f:
    line1 = f.readline()    # 读第一行（含 \n）
    line2 = f.readline()    # 读第二行
    print(line1.strip())    # strip() 去掉末尾换行符
```

```python
# readlines()：读取所有行，返回列表
with open("note.txt", "r", encoding="utf-8") as f:
    lines = f.readlines()   # ['第一行\n', '第二行\n', ...]
    for line in lines:
        print(line.strip())
```

```python
# 最Pythonic的写法：直接遍历文件对象（内存友好）
with open("note.txt", "r", encoding="utf-8") as f:
    for line in f:           # 逐行读取，不一次性载入内存
        print(line.strip())
```

**读取方式对比：**

```
read()        → 适合小文件，一次全部读入内存
readline()    → 适合逐行手动控制，精确处理
readlines()   → 适合需要列表操作所有行
直接遍历 for  → 适合大文件，逐行处理，内存最省（推荐 ✅）
```

***

### 📋 文件操作进阶（8 分钟）

#### 处理 CSV 格式数据

CSV（逗号分隔值）是最常见的数据交换格式，手动处理也很简单：

```python
# 写入 CSV
students = [
    ("小明", 18, 92),
    ("小红", 17, 85),
    ("小李", 19, 73),
]

with open("students.csv", "w", encoding="utf-8") as f:
    f.write("姓名,年龄,成绩\n")            # 写入表头
    for name, age, score in students:
        f.write(f"{name},{age},{score}\n")
```

```python
# 读取 CSV（跳过表头）
with open("students.csv", "r", encoding="utf-8") as f:
    header = f.readline().strip()          # 读并跳过表头
    print(f"字段：{header}")

    for line in f:
        name, age, score = line.strip().split(",")
        print(f"{name}（{age}岁）：{score} 分")
```

**输出：**

```
字段：姓名,年龄,成绩
小明（18岁）：92 分
小红（17岁）：85 分
小李（19岁）：73 分
```

***

#### os 模块：文件路径与目录操作

```python
import os

# 检查文件是否存在
print(os.path.exists("note.txt"))      # True / False

# 检查是否是文件 / 目录
print(os.path.isfile("note.txt"))      # True（是文件）
print(os.path.isdir("myfolder"))       # True（是目录）

# 获取文件大小（字节）
print(os.path.getsize("note.txt"))

# 路径拼接（跨平台安全写法）
path = os.path.join("data", "scores", "2024.csv")
print(path)   # data/scores/2024.csv（Linux/Mac）
              # data\scores\2024.csv（Windows）

# 创建目录（exist_ok=True：已存在不报错）
os.makedirs("data/output", exist_ok=True)

# 列出目录内容
files = os.listdir(".")     # 当前目录所有文件和文件夹
print(files)
```

> 💡 **始终用 `os.path.join()` 拼接路径**，不要手动写 `/` 或 `\`，这样代码在 Windows 和 Mac/Linux 上都能正常运行。

***

#### 文件编码问题

```python
# 常见编码
# utf-8    → 国际通用，推荐始终使用
# gbk      → 老版 Windows 中文系统默认
# utf-8-sig → 带 BOM 的 UTF-8，Excel 打开中文不乱码

# 写入供 Excel 读取的 CSV（用 utf-8-sig 避免乱码）
with open("report.csv", "w", encoding="utf-8-sig") as f:
    f.write("姓名,成绩\n")
    f.write("小明,92\n")
```

***

### 🛡️ 异常处理基础（10 分钟）

#### 什么是异常？

程序运行时遇到错误会抛出**异常（Exception）**，未处理的异常会让程序直接崩溃：

```python
# 这三行代码各自会引发什么异常？
print(10 / 0)           # ZeroDivisionError：除以零
print(int("abc"))       # ValueError：无法转换
print(open("no.txt"))   # FileNotFoundError：文件不存在
```

```
错误发生
    │
    ├── 没有异常处理 → 程序崩溃，打印红色错误信息，后续代码不执行
    └── 有异常处理   → 捕获错误，执行备用逻辑，程序继续运行
```

***

#### try / except：捕获异常

```python
# 基本语法
try:
    可能出错的代码
except 异常类型:
    出错时执行的代码
```

```python
# 示例：安全地把字符串转为整数
user_input = input("请输入一个数字：")

try:
    number = int(user_input)
    print(f"你输入的数字是：{number}")
except ValueError:
    print("❌ 输入的不是有效数字！")

print("程序继续运行...")    # 无论是否出错，这行都会执行
```

**执行流程：**

```
try 块开始
    │
    ├── 正常执行 → 跳过 except → 继续后面的代码
    │
    └── 发生异常 → 跳转到匹配的 except → 执行错误处理 → 继续后面的代码
```

***

#### 捕获多种异常

```python
def safe_divide(a, b):
    try:
        result = a / b
        print(f"{a} ÷ {b} = {result}")
    except ZeroDivisionError:
        print("❌ 除数不能为 0！")
    except TypeError:
        print("❌ 参数必须是数字！")

safe_divide(10, 2)      # 10 ÷ 2 = 5.0
safe_divide(10, 0)      # ❌ 除数不能为 0！
safe_divide(10, "a")    # ❌ 参数必须是数字！
```

```python
# 同时捕获多种异常（用元组）
try:
    value = int(input("输入："))
    result = 100 / value
except (ValueError, ZeroDivisionError):
    print("❌ 输入无效或为零")
```

```python
# 捕获所有异常（用 Exception，不推荐滥用）
try:
    risky_operation()
except Exception as e:        # as e：把异常对象赋给变量
    print(f"发生了错误：{e}")  # 打印具体错误信息
    print(f"错误类型：{type(e).__name__}")
```

***

#### else 与 finally

```python
try:
    number = int(input("输入数字："))
    result = 100 / number
except ValueError:
    print("❌ 不是数字")
except ZeroDivisionError:
    print("❌ 不能为零")
else:
    # 只有 try 块没有发生异常时才执行
    print(f"✅ 计算成功：{result}")
finally:
    # 无论是否发生异常，都会执行（常用于清理资源）
    print("--- 计算结束 ---")
```

**四个子句的执行规律：**

```
情况一：try 正常执行
  try ✅ → else ✅ → finally ✅    （except 跳过）

情况二：try 发生异常且被捕获
  try ❌ → except ✅ → finally ✅  （else 跳过）

情况三：try 发生异常但未被捕获
  try ❌ → finally ✅ → 异常继续向上抛出
```

```python
# finally 的典型用途：确保资源被释放
def read_file_safe(filename):
    f = None
    try:
        f = open(filename, "r", encoding="utf-8")
        return f.read()
    except FileNotFoundError:
        print(f"❌ 文件 {filename} 不存在")
        return None
    finally:
        if f:
            f.close()   # 无论成功与否都关闭文件
```

> 💡 实际上使用 `with open(...)` 就能自动完成 `finally` 的工作，所以更推荐 `with` 语句。

***

#### 常见异常类型速查

| 异常类型                | 触发场景         | 示例                      |
| ------------------- | ------------ | ----------------------- |
| `ValueError`        | 值的类型正确但内容不合法 | `int("abc")`            |
| `TypeError`         | 操作或函数应用于错误类型 | `"1" + 1`               |
| `ZeroDivisionError` | 除以零          | `10 / 0`                |
| `FileNotFoundError` | 文件或目录不存在     | `open("no.txt")`        |
| `IndexError`        | 序列索引超出范围     | `lst[99]`（lst 只有 3 个元素） |
| `KeyError`          | 字典中键不存在      | `d["x"]`（x 不在字典里）       |
| `AttributeError`    | 对象没有该属性或方法   | `"hello".push()`        |
| `NameError`         | 变量名未定义       | `print(undefined_var)`  |
| `PermissionError`   | 没有文件操作权限     | 写入只读文件                  |

***

### ⚡ 异常进阶（4 分钟）

#### raise：主动抛出异常

有时需要在业务逻辑中主动触发异常，表示"这个情况不被允许"：

```python
def set_age(age):
    if not isinstance(age, int):
        raise TypeError(f"年龄必须是整数，收到了 {type(age).__name__}")
    if age < 0 or age > 150:
        raise ValueError(f"年龄 {age} 超出合理范围（0～150）")
    print(f"年龄设置为：{age}")

set_age(18)        # ✅ 年龄设置为：18
set_age("十八")    # ❌ TypeError: 年龄必须是整数，收到了 str
set_age(200)       # ❌ ValueError: 年龄 200 超出合理范围（0～150）
```

***

#### 自定义异常

继承 `Exception` 类，创建有业务含义的自定义异常：

```python
# 定义自定义异常
class ScoreError(Exception):
    """成绩相关的异常"""
    pass

class NegativeScoreError(ScoreError):
    """成绩不能为负数"""
    pass

class ScoreOverflowError(ScoreError):
    """成绩超过最大值"""
    pass

# 使用自定义异常
def validate_score(score):
    if score < 0:
        raise NegativeScoreError(f"成绩不能为负数：{score}")
    if score > 100:
        raise ScoreOverflowError(f"成绩不能超过 100：{score}")
    return score

# 捕获时可以按层级捕获
try:
    validate_score(150)
except NegativeScoreError as e:
    print(f"负数错误：{e}")
except ScoreOverflowError as e:
    print(f"溢出错误：{e}")   # 输出：溢出错误：成绩不能超过 100：150
except ScoreError as e:
    print(f"通用成绩错误：{e}")  # 捕获所有 ScoreError 的子类
```

***

### 🏁 综合练习（5 分钟）

#### 练习：成绩持久化系统

将第六课的成绩管理系统升级，加入文件读写和异常处理，让数据真正被保存下来：

```python
# ============================================
# 成绩持久化管理系统
# ============================================
import os

DATA_FILE = "scores.csv"    # 数据文件路径


# ---------- 文件读写函数 ----------
def load_scores():
    """从文件加载成绩，文件不存在则返回空字典"""
    if not os.path.exists(DATA_FILE):
        print("💡 数据文件不存在，将新建。")
        return {}

    scores = {}
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            for line in f:
                line = line.strip()
                if not line:
                    continue            # 跳过空行
                name, score = line.split(",")
                scores[name] = int(score)
        print(f"✅ 已加载 {len(scores)} 条成绩记录。")
    except ValueError as e:
        print(f"⚠️ 文件格式错误，部分数据跳过：{e}")
    except PermissionError:
        print(f"❌ 没有读取 {DATA_FILE} 的权限！")
    return scores


def save_scores(scores):
    """将成绩字典保存到文件"""
    try:
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            for name, score in scores.items():
                f.write(f"{name},{score}\n")
        print(f"✅ 已保存 {len(scores)} 条记录到 {DATA_FILE}。")
    except PermissionError:
        print(f"❌ 没有写入 {DATA_FILE} 的权限！")


# ---------- 数据操作函数 ----------
def add_score(scores, name, score_str):
    """添加或更新成绩，包含输入校验"""
    try:
        score = int(score_str)
        if not (0 <= score <= 100):
            raise ValueError(f"成绩须在 0～100 之间，收到：{score}")
        scores[name.strip()] = score
        print(f"✅ 已记录：{name} → {score} 分")
    except ValueError as e:
        print(f"❌ 成绩无效：{e}")


def show_scores(scores):
    """展示所有成绩，按分数降序"""
    if not scores:
        print("📭 暂无成绩记录。")
        return
    print("\n" + "=" * 30)
    print(f"{'📊 当前成绩':^26}")
    print("=" * 30)
    for name, score in sorted(scores.items(), key=lambda x: x[1], reverse=True):
        bar = "█" * (score // 10)   # 简易进度条
        print(f"  {name:<6} {score:>3} 分  {bar}")
    print("=" * 30 + "\n")


# ---------- 主程序 ----------
def main():
    scores = load_scores()

    while True:
        print("1. 录入成绩  2. 查看成绩  3. 保存退出")
        choice = input("请选择：").strip()

        if choice == "1":
            name  = input("  姓名：").strip()
            score = input("  成绩：").strip()
            add_score(scores, name, score)

        elif choice == "2":
            show_scores(scores)

        elif choice == "3":
            save_scores(scores)
            print("👋 再见！")
            break

        else:
            print("⚠️ 请输入 1、2 或 3。")


main()
```

**示例运行过程：**

```
💡 数据文件不存在，将新建。
1. 录入成绩  2. 查看成绩  3. 保存退出
请选择：1
  姓名：小明
  成绩：92
✅ 已记录：小明 → 92 分
请选择：1
  姓名：小红
  成绩：abc
❌ 成绩无效：invalid literal for int() with base 10: 'abc'
请选择：2

==============================
         📊 当前成绩         
==============================
  小明     92 分  █████████
==============================

请选择：3
✅ 已保存 1 条记录到 scores.csv。
👋 再见！
```

***

### 📝 本课知识总结

```
文件读写
├── open(路径, 模式, encoding="utf-8")
├── 模式：r（只读）/ w（覆盖写）/ a（追加）/ x（新建）
├── with open(...) as f:  → 推荐，自动关闭
├── 读取：f.read() / f.readline() / f.readlines() / for line in f
├── 写入：f.write(str) / f.writelines(list)
└── os.path：exists() / isfile() / join() / getsize()

异常处理
├── try:      → 放可能出错的代码
├── except 类型 as e: → 捕获指定异常
├── else:     → try 无异常时执行
├── finally:  → 无论如何都执行（清理资源）
├── raise     → 主动抛出异常
└── 自定义异常：class MyError(Exception): pass

常见异常
├── ValueError        → 值不合法
├── TypeError         → 类型错误
├── ZeroDivisionError → 除以零
├── FileNotFoundError → 文件不存在
├── IndexError        → 索引越界
└── KeyError          → 字典键不存在
```

***

### 🎯 课后作业

**练习 1（基础）：** 写一个"安全文件读取器"：

* 接受用户输入的文件名
* 用 `try/except` 处理 `FileNotFoundError` 和 `PermissionError`
* 文件存在则打印行数和内容；文件不存在则提示"文件不存在，是否新建？（y/n）"
* 用户选 y 则创建并写入一行默认内容

**练习 2（进阶）：** 写一个"CSV 成绩分析器"：

* 读取一个包含"姓名,成绩"的 CSV 文件（自己先写入几条数据）
* 跳过表头，逐行解析，用 `try/except` 跳过格式错误的行
* 计算平均分、最高分、最低分
* 将分析结果追加写入同一文件末尾（用 `"a"` 模式）

**练习 3（挑战）：** 写一个"单词频率统计器"：

* 读取一个英文文本文件（自己创建，写几句英文）
* 将所有内容转为小写，去除标点符号（提示：用 `str.replace()` 或 `str.translate()`）
* 统计每个单词出现次数，存入字典
* 将统计结果按频率降序写入新文件 `word_count.txt`
* 全程使用异常处理，确保文件操作出错时程序不崩溃

***

### 🔜 下节课预告

**第八课：模块与包、pip 使用**

```python
# 你将学会复用别人写好的代码，以及管理项目依赖：

# 使用标准库模块
import random
import datetime
import json

print(random.choice(["石头", "剪刀", "布"]))
print(datetime.date.today())

# 将字典保存为 JSON 文件（比 CSV 更灵活）
scores = {"小明": 92, "小红": 85}
with open("scores.json", "w", encoding="utf-8") as f:
    json.dump(scores, f, ensure_ascii=False, indent=2)

# 在终端安装第三方库（pip）
# pip install requests
import requests
response = requests.get("https://api.github.com")
print(response.status_code)   # 200
```

***

> 💬 **有疑问？** 欢迎在评论区留言或加入学习群讨论。
>
> ⭐ 觉得有帮助的话，别忘了分享给朋友！

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 7 课 / 共 10 课 Power by Andrew Liu_
