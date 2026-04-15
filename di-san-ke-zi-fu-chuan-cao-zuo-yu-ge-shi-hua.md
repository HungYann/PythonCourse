# 第三课：字符串操作与格式化

### 第三课：Python 基础（从零起飞）- 字符串操作与格式化

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01、02 课** 🛠️ **工具准备：Python 3.x 或** [**在线运行环境**](https://colab.research.google.com/)

***

### 课程导览

| 模块       | 内容                           | 时间    |
| -------- | ---------------------------- | ----- |
| 🔥 热身回顾  | 字符串是什么？快速复习                  | 3 分钟  |
| ✂️ 索引与切片 | 精准提取字符串中的内容                  | 8 分钟  |
| 🔧 常用方法  | 大小写、查找、替换、分割等                | 12 分钟 |
| 🎨 格式化输出 | % / format() / f-string 三种方式 | 10 分钟 |
| 🏁 综合练习  | 动手写一个文本处理小程序                 | 7 分钟  |

***

### 🔥 热身回顾（3 分钟）

字符串（`str`）是 Python 中使用最频繁的数据类型之一。**任何被引号包裹的内容都是字符串。**

```python
s1 = "双引号字符串"
s2 = '单引号字符串'
s3 = """三引号字符串
可以跨越多行
非常适合写长文本"""

print(type(s1))   # 输出：<class 'str'>
```

> 💡 单引号和双引号效果完全相同，选一种风格坚持使用即可。三引号用于多行文本。

***

### ✂️ 索引与切片（8 分钟）

#### 索引：取出单个字符

字符串本质上是一串有序的字符，每个字符都有一个编号，叫做**索引（index）**。

```
字 符 串：  P  y  t  h  o  n
正向索引：  0  1  2  3  4  5
反向索引： -6 -5 -4 -3 -2 -1
```

```python
s = "Python"

# 正向索引（从 0 开始）
print(s[0])    # 输出：P
print(s[1])    # 输出：y
print(s[5])    # 输出：n

# 反向索引（从 -1 开始）
print(s[-1])   # 输出：n（最后一个）
print(s[-2])   # 输出：o（倒数第二个）
```

> ⚠️ **越界会报错！** 如果索引超过字符串长度，Python 会抛出 `IndexError`。

***

#### 切片：取出一段字符

切片语法：`字符串[start : end : step]`

* `start`：起始位置（**包含**）
* `end`：结束位置（**不包含**）
* `step`：步长（默认为 1）

```python
s = "Hello, Python!"
#    0123456789...

print(s[0:5])     # 输出：Hello（取第 0~4 位）
print(s[7:13])    # 输出：Python
print(s[7:])      # 输出：Python!（省略 end，取到末尾）
print(s[:5])      # 输出：Hello（省略 start，从头开始）
print(s[:])       # 输出：Hello, Python!（完整复制）
```

**使用步长**

```python
s = "0123456789"

print(s[::2])     # 输出：02468（每隔一个取一个）
print(s[1::2])    # 输出：13579
print(s[::-1])    # 输出：9876543210（步长为 -1，即反转！）
```

> 🏆 **反转字符串的最 Pythonic 写法：** `s[::-1]`，记住它！

**切片速查表**

| 写法        | 含义          |
| --------- | ----------- |
| `s[2:5]`  | 第 2、3、4 位   |
| `s[:3]`   | 前 3 个字符     |
| `s[3:]`   | 第 3 位到末尾    |
| `s[-3:]`  | 最后 3 个字符    |
| `s[:-1]`  | 除最后一个外的所有字符 |
| `s[::-1]` | 整个字符串反转     |

***

### 🔧 常用方法（12 分钟）

Python 字符串内置了大量实用方法。记住：**字符串是不可变的**，这些方法不会修改原字符串，而是返回一个新字符串。

```python
s = "  Hello, Python!  "
```

#### 大小写转换

```python
s = "Hello, Python!"

print(s.upper())        # 输出：HELLO, PYTHON!
print(s.lower())        # 输出：hello, python!
print(s.title())        # 输出：Hello, Python!（每个单词首字母大写）
print(s.capitalize())   # 输出：Hello, python!（仅句首大写）
print(s.swapcase())     # 输出：hELLO, pYTHON!（大小写互换）
```

***

#### 去除空白

```python
s = "  hello  "

print(s.strip())        # 输出：hello（去除两端空格）
print(s.lstrip())       # 输出：hello  （只去左边）
print(s.rstrip())       # 输出：  hello（只去右边）
```

> 💡 处理用户输入时，`strip()` 几乎是必用的！用户很容易误输入多余空格。

***

#### 查找与判断

```python
s = "Hello, Python!"

# 查找位置（找不到返回 -1）
print(s.find("Python"))     # 输出：7
print(s.find("Java"))       # 输出：-1

# 统计出现次数
print(s.count("l"))         # 输出：2

# 判断包含关系（推荐用 in）
print("Python" in s)        # 输出：True
print("Java" in s)          # 输出：False
print("Java" not in s)      # 输出：True
```

```python
# 判断开头/结尾
url = "https://www.python.org"

print(url.startswith("https"))   # 输出：True
print(url.endswith(".org"))      # 输出：True
```

```python
# 判断字符串内容组成
print("12345".isdigit())     # 输出：True（全是数字）
print("hello".isalpha())     # 输出：True（全是字母）
print("hello123".isalnum())  # 输出：True（全是字母或数字）
print("   ".isspace())       # 输出：True（全是空白）
```

***

#### 替换

```python
s = "I love Java, Java is great!"

# replace(旧, 新)
print(s.replace("Java", "Python"))
# 输出：I love Python, Python is great!

# 限制替换次数
print(s.replace("Java", "Python", 1))
# 输出：I love Python, Java is great!
```

***

#### 分割与拼接

```python
# split：将字符串按分隔符切成列表
sentence = "apple,banana,orange,grape"
fruits = sentence.split(",")
print(fruits)           # 输出：['apple', 'banana', 'orange', 'grape']
print(fruits[0])        # 输出：apple

# 按空格分割（默认）
words = "Hello Python World".split()
print(words)            # 输出：['Hello', 'Python', 'World']

# join：将列表用指定字符拼接成字符串（split 的逆操作）
result = " - ".join(fruits)
print(result)           # 输出：apple - banana - orange - grape

result2 = "".join(["P", "y", "t", "h", "o", "n"])
print(result2)          # 输出：Python
```

***

#### 对齐与填充

```python
s = "Python"

print(s.center(20))          # 输出：       Python       （居中，总宽 20）
print(s.center(20, "*"))     # 输出：*******Python*******（用 * 填充）
print(s.ljust(20, "-"))      # 输出：Python--------------（左对齐）
print(s.rjust(20, "-"))      # 输出：--------------Python（右对齐）
print("42".zfill(5))         # 输出：00042（左边补 0）
```

***

#### 方法速查表

| 方法              | 功能      | 示例                                 |
| --------------- | ------- | ---------------------------------- |
| `upper()`       | 全部大写    | `"hi".upper()` → `"HI"`            |
| `lower()`       | 全部小写    | `"HI".lower()` → `"hi"`            |
| `strip()`       | 去两端空白   | `" hi ".strip()` → `"hi"`          |
| `find(x)`       | 查找位置    | `"hi".find("i")` → `1`             |
| `count(x)`      | 统计次数    | `"hello".count("l")` → `2`         |
| `replace(a,b)`  | 替换内容    | `"hi".replace("i","ey")` → `"hey"` |
| `split(x)`      | 分割为列表   | `"a,b".split(",")` → `['a','b']`   |
| `join(list)`    | 列表拼为字符串 | `",".join(['a','b'])` → `"a,b"`    |
| `startswith(x)` | 判断开头    | `"hi".startswith("h")` → `True`    |
| `endswith(x)`   | 判断结尾    | `"hi".endswith("i")` → `True`      |

***

### 🎨 字符串格式化（10 分钟）

Python 有三种格式化字符串的方式，从旧到新依次演进。

***

#### 方式一：% 占位符（旧式，了解即可）

```python
name = "小明"
age = 18
score = 95.5

print("我叫 %s，今年 %d 岁，成绩 %.1f 分" % (name, age, score))
# 输出：我叫 小明，今年 18 岁，成绩 95.5 分
```

| 占位符    | 含义       |
| ------ | -------- |
| `%s`   | 字符串      |
| `%d`   | 整数       |
| `%f`   | 浮点数      |
| `%.2f` | 保留 2 位小数 |

***

#### 方式二：`.format()` 方法（较新，仍常见）

```python
name = "小明"
age = 18

# 按位置
print("我叫 {}，今年 {} 岁".format(name, age))
# 输出：我叫 小明，今年 18 岁

# 按索引
print("我叫 {0}，{0} 今年 {1} 岁".format(name, age))
# 输出：我叫 小明，小明 今年 18 岁

# 按关键字
print("我叫 {name}，今年 {age} 岁".format(name="小红", age=16))
# 输出：我叫 小红，今年 16 岁
```

**格式控制**

```python
pi = 3.14159265

print("{:.2f}".format(pi))       # 输出：3.14（保留 2 位小数）
print("{:10.2f}".format(pi))     # 输出：      3.14（宽度 10，右对齐）
print("{:<10.2f}".format(pi))    # 输出：3.14      （左对齐）
print("{:^10.2f}".format(pi))    # 输出：   3.14   （居中）
print("{:0>8d}".format(42))      # 输出：00000042（用 0 填充，右对齐）
```

***

#### 方式三：f-string（Python 3.6+，推荐 ✅）

f-string 是最简洁、最直观的方式，**日常开发首选**。

```python
name = "小明"
age = 18
score = 95.5678

# 基本用法：在引号前加 f，变量用 {} 包裹
print(f"我叫{name}，今年{age}岁")
# 输出：我叫小明，今年18岁

# 直接在 {} 内写表达式
print(f"明年我 {age + 1} 岁")
# 输出：明年我 19 岁

# 格式控制（在变量后加 :）
print(f"成绩：{score:.2f}")       # 输出：成绩：95.57
print(f"成绩：{score:10.2f}")     # 输出：成绩：     95.57
print(f"{'居中':^20}")            # 输出：        居中        
```

**f-string 调试技巧（Python 3.8+）**

```python
x = 42
y = 3.14

# 在变量后加 = 号，自动打印变量名和值（调试神器！）
print(f"{x = }")      # 输出：x = 42
print(f"{y = :.2f}")  # 输出：y = 3.14
```

***

#### 三种方式对比

```python
name = "Python"
version = 3.12

# 方式一：% 占位符
print("语言：%s，版本：%.2f" % (name, version))

# 方式二：.format()
print("语言：{}，版本：{:.2f}".format(name, version))

# 方式三：f-string（推荐）
print(f"语言：{name}，版本：{version:.2f}")

# 三种方式输出完全相同：
# 语言：Python，版本：3.12
```

> 🏆 **建议：** 新代码一律使用 f-string，只需记住 `f"...{变量}..."` 即可。

***

#### 常用格式说明符速查

| 格式     | 含义        | 示例                 | 结果          |
| ------ | --------- | ------------------ | ----------- |
| `:.2f` | 保留 2 位小数  | `f"{3.14159:.2f}"` | `3.14`      |
| `:d`   | 整数        | `f"{42:d}"`        | `42`        |
| `:08d` | 宽度 8，补零   | `f"{42:08d}"`      | `00000042`  |
| `:>10` | 宽度 10，右对齐 | `f"{'hi':>10}"`    | `hi`        |
| `:<10` | 宽度 10，左对齐 | `f"{'hi':<10}"`    | `hi`        |
| `:^10` | 宽度 10，居中  | `f"{'hi':^10}"`    | `hi`        |
| `:,`   | 千分位分隔符    | `f"{1000000:,}"`   | `1,000,000` |
| `:.0%` | 百分比       | `f"{0.856:.0%}"`   | `86%`       |

***

### 🏁 综合练习（7 分钟）

#### 练习：用户信息卡片生成器

综合运用今天所有知识，写一个程序：

```python
# ================================
# 用户信息卡片生成器
# ================================

# 获取用户输入
raw_name = input("请输入姓名（可含空格）：")
raw_email = input("请输入邮箱：")
raw_score = input("请输入成绩（0-100）：")

# 数据清洗
name = raw_name.strip()             # 去除多余空格
email = raw_email.strip().lower()   # 去空格并转小写
score = float(raw_score.strip())

# 数据处理
name_upper = name.upper()           # 姓名转大写
is_pass = score >= 60               # 是否及格
grade = "优秀" if score >= 90 else ("良好" if score >= 75 else ("及格" if score >= 60 else "不及格"))

# 检查邮箱格式（简单验证）
is_valid_email = "@" in email and email.endswith((".com", ".cn", ".org", ".edu"))

# 提取邮箱域名
domain = email.split("@")[-1] if "@" in email else "未知"

# 生成信息卡片
width = 34
print()
print("=" * width)
print(f"{'用 户 信 息 卡 片':^{width - 2}}")
print("=" * width)
print(f"  姓名：{name_upper}")
print(f"  邮箱：{email}")
print(f"  域名：{domain}")
print(f"  邮箱有效：{'✅' if is_valid_email else '❌'}")
print("-" * width)
print(f"  成绩：{score:.1f} 分")
print(f"  等级：{grade}")
print(f"  状态：{'✅ 通过' if is_pass else '❌ 未通过'}")
print("=" * width)
```

**示例输出：**

```
请输入姓名（可含空格）：  Zhang San  
请输入邮箱：ZhangSan@Gmail.COM
请输入成绩（0-100）：87.5

==================================
       用 户 信 息 卡 片        
==================================
  姓名：ZHANG SAN
  邮箱：zhangsan@gmail.com
  域名：gmail.com
  邮箱有效：✅
----------------------------------
  成绩：87.5 分
  等级：良好
  状态：✅ 通过
==================================
```

***

### 📝 本课知识总结

```
字符串索引与切片
├── 正向索引：s[0], s[1], ...
├── 反向索引：s[-1], s[-2], ...
└── 切片：s[start:end:step]，s[::-1] 反转

常用方法
├── 大小写：upper() / lower() / title()
├── 清理：strip() / lstrip() / rstrip()
├── 查找：find() / count() / in / startswith() / endswith()
├── 修改：replace()
└── 分割：split() / join()

格式化方式
├── % 占位符：旧式，了解即可
├── .format()：较新，仍常见
└── f-string（推荐）：f"{变量:.格式}"
    ├── :.2f  → 保留小数位
    ├── :>10  → 右对齐
    ├── :,    → 千分位
    └── :.0%  → 百分比
```

***

### 🎯 课后作业

**练习 1（基础）：** 给定字符串 `s = " Hello, World! "`，完成以下操作：

1. 去除两端空格
2. 全部转为大写
3. 将 `World` 替换为 `Python`
4. 按 `,` 分割，打印每一部分

**练习 2（进阶）：** 写一个"手机号打码"程序：

* 接收用户输入的 11 位手机号
* 将中间 4 位替换为 `****`
* 输出：`138****5678`
* 提示：用切片 + 字符串拼接，或用 `replace()`

**练习 3（挑战）：** 写一个简单的"词频统计"：

* 接收一段英文句子（全部转小写处理）
* 统计其中字母 `e` 出现的次数
* 统计单词个数（按空格分割）
* 打印最长的单词是哪个及其长度

***

### 🔜 下节课预告

**第四课：列表、元组、字典、集合**

```python
# 你将学会管理一组数据：
students = ["小明", "小红", "小李"]
scores = {"小明": 95, "小红": 87, "小李": 72}

for name in students:
    print(f"{name} 的成绩是 {scores[name]} 分")
```

***

> 💬 **有疑问？** 欢迎在评论区留言或加入学习群讨论。
>
> ⭐ 觉得有帮助的话，别忘了分享给朋友！

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 3 课 / 共 10 课 Power by Andrew Liu_

