# 第四课：列表、元组、字典、集合

### 第四课：Python 基础（从零起飞） - 列表、元组、字典、集合

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01、02、03 课** 🛠️ **工具准备：Python 3.x 或** [**在线运行环境**](https://colab.research.google.com/)

***

### 课程导览

| 模块      | 内容             | 时间    |
| ------- | -------------- | ----- |
| 🔥 热身回顾 | 为什么需要"容器"？     | 3 分钟  |
| 📋 列表   | 创建、索引、常用方法、推导式 | 12 分钟 |
| 🔒 元组   | 创建、不可变性、解包     | 5 分钟  |
| 📖 字典   | 创建、访问、常用方法、遍历  | 12 分钟 |
| 🎯 集合   | 创建、去重、集合运算     | 6 分钟  |
| 🏁 综合练习 | 学生成绩管理小程序      | 7 分钟  |

***

### 🔥 热身回顾（3 分钟）

前三课我们学会了用变量存储**单个数据**。但如果要管理一个班 30 位同学的成绩，总不能写 30 个变量吧？

```python
# ❌ 不好的做法：每人一个变量
score_xiaoming = 95
score_xiaohong = 87
score_xiaoli   = 72
# ...还有 27 个...

# ✅ 好的做法：用一个"容器"装所有数据
scores = [95, 87, 72, ...]
```

Python 提供了四种内置"容器"类型，今天全部掌握：

| 类型         | 符号           | 特点          | 典型用途           |
| ---------- | ------------ | ----------- | -------------- |
| 列表 `list`  | `[]`         | 有序、可修改      | 存储一组同类数据       |
| 元组 `tuple` | `()`         | 有序、**不可修改** | 存储固定数据（坐标、RGB） |
| 字典 `dict`  | `{}`         | 键值对、可修改     | 存储有名字的数据       |
| 集合 `set`   | `{}`/`set()` | 无序、**不重复**  | 去重、集合运算        |

***

### 📋 列表（12 分钟）

#### 什么是列表？

列表就像一排有编号的**储物格**，每格可以放任意类型的数据。

```
索引：   0      1      2      3      4
     ┌──────┬──────┬──────┬──────┬──────┐
     │ "小明" │ "小红" │ "小李" │ "小张" │ "小王" │
     └──────┴──────┴──────┴──────┴──────┘
```

#### 创建列表

```python
# 空列表
empty = []

# 字符串列表
students = ["小明", "小红", "小李"]

# 数字列表
scores = [95, 87, 72, 68, 100]

# 混合类型（可以，但不推荐）
mixed = ["Python", 3, True, 3.14]

# 用 list() 把其他类型转换为列表
chars = list("Python")
print(chars)   # 输出：['P', 'y', 't', 'h', 'o', 'n']
```

***

#### 索引与切片

列表的索引规则和字符串**完全一致**（还记得第三课吗？）：

```python
fruits = ["苹果", "香蕉", "橙子", "葡萄", "西瓜"]
#          0       1       2       3       4
#         -5      -4      -3      -2      -1

# 正向索引
print(fruits[0])     # 输出：苹果
print(fruits[2])     # 输出：橙子

# 反向索引
print(fruits[-1])    # 输出：西瓜
print(fruits[-2])    # 输出：葡萄

# 切片（和字符串切片语法完全相同）
print(fruits[1:3])   # 输出：['香蕉', '橙子']
print(fruits[:2])    # 输出：['苹果', '香蕉']
print(fruits[2:])    # 输出：['橙子', '葡萄', '西瓜']
print(fruits[::-1])  # 输出：['西瓜', '葡萄', '橙子', '香蕉', '苹果']（反转！）
```

***

#### 修改列表

列表是**可变的**，可以随意增删改：

```python
students = ["小明", "小红", "小李"]

# 修改元素
students[1] = "小花"
print(students)   # 输出：['小明', '小花', '小李']

# 追加到末尾
students.append("小张")
print(students)   # 输出：['小明', '小花', '小李', '小张']

# 插入到指定位置
students.insert(1, "小王")
print(students)   # 输出：['小明', '小王', '小花', '小李', '小张']

# 删除指定值（第一个匹配项）
students.remove("小花")
print(students)   # 输出：['小明', '小王', '小李', '小张']

# 弹出并返回末尾元素
last = students.pop()
print(last)       # 输出：小张
print(students)   # 输出：['小明', '小王', '小李']

# 弹出并返回指定位置的元素
first = students.pop(0)
print(first)      # 输出：小明
```

***

#### 常用方法

```python
nums = [3, 1, 4, 1, 5, 9, 2, 6, 5]

# 排序（直接修改原列表）
nums.sort()
print(nums)              # 输出：[1, 1, 2, 3, 4, 5, 5, 6, 9]

# 降序排列
nums.sort(reverse=True)
print(nums)              # 输出：[9, 6, 5, 5, 4, 3, 2, 1, 1]

# 反转列表
nums.reverse()
print(nums)              # 输出：[1, 1, 2, 3, 4, 5, 5, 6, 9]

# 统计某值出现次数
print(nums.count(5))     # 输出：2

# 查找某值的索引（找不到会报错）
print(nums.index(4))     # 输出：4

# 获取长度
print(len(nums))         # 输出：9

# 清空列表
# nums.clear()           # 清空后 nums = []

# 合并两个列表
a = [1, 2, 3]
b = [4, 5, 6]
a.extend(b)
print(a)                 # 输出：[1, 2, 3, 4, 5, 6]

# 也可以用 + 号合并（返回新列表，不修改原列表）
c = [1, 2] + [3, 4]
print(c)                 # 输出：[1, 2, 3, 4]
```

> 💡 `sort()` 直接修改原列表；如果想保留原列表，用 `sorted(列表)` 返回新列表。

***

#### 遍历列表

```python
fruits = ["苹果", "香蕉", "橙子"]

# 方式一：直接遍历值
for fruit in fruits:
    print(fruit)

# 方式二：遍历索引（用 range）
for i in range(len(fruits)):
    print(f"第{i+1}个：{fruits[i]}")

# 方式三：同时获取索引和值（推荐 ✅）
for i, fruit in enumerate(fruits):
    print(f"第{i+1}个：{fruit}")
```

输出：

```
第1个：苹果
第2个：香蕉
第3个：橙子
```

***

#### 列表推导式（List Comprehension）

列表推导式是 Python 最受欢迎的特性之一，用一行代码生成新列表。

```python
# 普通写法：生成 1~10 的平方列表
squares = []
for i in range(1, 11):
    squares.append(i ** 2)
print(squares)   # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

# 推导式写法（等价，但更简洁）
squares = [i ** 2 for i in range(1, 11)]
print(squares)   # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

```python
# 带条件的推导式：只保留偶数的平方
even_squares = [i ** 2 for i in range(1, 11) if i % 2 == 0]
print(even_squares)   # 输出：[4, 16, 36, 64, 100]

# 处理字符串列表
words = ["  apple  ", "  banana  ", "  orange  "]
clean = [w.strip().upper() for w in words]
print(clean)          # 输出：['APPLE', 'BANANA', 'ORANGE']
```

> 🏆 **推导式公式：** `[表达式 for 变量 in 可迭代对象 if 条件]` 条件部分可以省略。

***

#### 列表方法速查表

| 方法             | 功能               | 示例                   |
| -------------- | ---------------- | -------------------- |
| `append(x)`    | 末尾追加             | `lst.append(4)`      |
| `insert(i, x)` | 指定位置插入           | `lst.insert(0, "a")` |
| `remove(x)`    | 删除第一个 x          | `lst.remove("a")`    |
| `pop(i)`       | 删除并返回第 i 个（默认末尾） | `lst.pop()`          |
| `sort()`       | 原地排序             | `lst.sort()`         |
| `reverse()`    | 原地反转             | `lst.reverse()`      |
| `index(x)`     | 查找 x 的索引         | `lst.index(3)`       |
| `count(x)`     | 统计 x 出现次数        | `lst.count(1)`       |
| `extend(lst2)` | 合并另一个列表          | `lst.extend([4,5])`  |
| `clear()`      | 清空列表             | `lst.clear()`        |
| `len(lst)`     | 获取长度             | `len(lst)`           |

***

### 🔒 元组（5 分钟）

#### 什么是元组？

元组和列表非常相似，区别只有一点：**元组一旦创建，不能修改**。

```python
# 用圆括号创建
point = (3, 7)
colors = ("red", "green", "blue")
person = ("小明", 18, "北京")

# 空元组
empty = ()

# 只有一个元素的元组（注意必须加逗号！）
single = (42,)         # ✅ 这是元组
not_tuple = (42)       # ❌ 这是整数 42，括号只是运算符

print(type(single))    # 输出：<class 'tuple'>
print(type(not_tuple)) # 输出：<class 'int'>
```

***

#### 访问元组

访问方式和列表完全相同，支持索引和切片：

```python
colors = ("red", "green", "blue")

print(colors[0])      # 输出：red
print(colors[-1])     # 输出：blue
print(colors[1:])     # 输出：('green', 'blue')
print(len(colors))    # 输出：3
print("red" in colors)# 输出：True
```

***

#### 元组解包（Tuple Unpacking）

元组最强大的用法之一——一次性把多个值"拆"给多个变量：

```python
point = (10, 20)
x, y = point              # 解包！
print(f"x={x}, y={y}")   # 输出：x=10, y=20

# 函数返回多个值（本质上就是元组）
def get_min_max(numbers):
    return min(numbers), max(numbers)   # 返回元组

low, high = get_min_max([3, 1, 4, 1, 5, 9])
print(f"最小值：{low}，最大值：{high}")   # 输出：最小值：1，最大值：9

# 交换两个变量（Python 独特写法）
a, b = 1, 2
a, b = b, a
print(a, b)   # 输出：2 1
```

> 💡 **何时用元组，何时用列表？**
>
> * 数据会变化（增删改）→ 用**列表**
> * 数据固定不变（坐标、颜色、配置项）→ 用**元组**
> * 函数需要返回多个值 → 用**元组**

***

### 📖 字典（12 分钟）

#### 什么是字典？

字典存储**键值对（key-value）**，就像真实的字典——用"词条"（键）查找"解释"（值）。

```
{
  "小明": 95,    ← key: value
  "小红": 87,
  "小李": 72
}
```

```python
# 创建字典
scores = {
    "小明": 95,
    "小红": 87,
    "小李": 72
}

# 空字典
empty = {}

# 用 dict() 创建
info = dict(name="小明", age=18, city="北京")
print(info)   # 输出：{'name': '小明', 'age': 18, 'city': '北京'}
```

> 💡 字典的键（key）必须是**不可变类型**（字符串、数字、元组），值（value）可以是任意类型。

***

#### 访问与修改

```python
scores = {"小明": 95, "小红": 87, "小李": 72}

# 访问：用 [] 取值（键不存在会报 KeyError）
print(scores["小明"])         # 输出：95

# 安全访问：用 get()（键不存在返回 None 或默认值，不报错）
print(scores.get("小张"))     # 输出：None
print(scores.get("小张", 0))  # 输出：0（指定默认值）

# 修改已有的值
scores["小明"] = 100
print(scores["小明"])         # 输出：100

# 添加新键值对
scores["小张"] = 88
print(scores)
# 输出：{'小明': 100, '小红': 87, '小李': 72, '小张': 88}

# 删除键值对
del scores["小李"]
print(scores)
# 输出：{'小明': 100, '小红': 87, '小张': 88}

# 弹出并返回值
val = scores.pop("小红")
print(val)      # 输出：87

# 检查键是否存在（用 in）
print("小明" in scores)   # 输出：True
print("小红" in scores)   # 输出：False
```

***

#### 常用方法

```python
student = {
    "name": "小明",
    "age": 18,
    "city": "北京",
    "score": 95
}

# 获取所有键
print(student.keys())
# 输出：dict_keys(['name', 'age', 'city', 'score'])

# 获取所有值
print(student.values())
# 输出：dict_values(['小明', 18, '北京', 95])

# 获取所有键值对（每个元素是元组）
print(student.items())
# 输出：dict_items([('name', '小明'), ('age', 18), ('city', '北京'), ('score', 95)])

# 合并字典（update）
extra = {"grade": "高三", "hobby": "编程"}
student.update(extra)
print(student)
# 输出：{'name': '小明', 'age': 18, 'city': '北京', 'score': 95, 'grade': '高三', 'hobby': '编程'}

# 获取长度
print(len(student))   # 输出：6
```

***

#### 遍历字典

```python
scores = {"小明": 95, "小红": 87, "小李": 72}

# 遍历所有键
for name in scores:
    print(name)

# 遍历所有值
for score in scores.values():
    print(score)

# 同时遍历键和值（最常用 ✅）
for name, score in scores.items():
    print(f"{name}：{score} 分")
```

输出：

```
小明：95 分
小红：87 分
小李：72 分
```

***

#### 嵌套字典

字典的值也可以是字典，用于存储更复杂的数据：

```python
students = {
    "小明": {"age": 18, "score": 95, "city": "北京"},
    "小红": {"age": 17, "score": 87, "city": "上海"},
}

# 访问嵌套数据
print(students["小明"]["score"])   # 输出：95
print(students["小红"]["city"])    # 输出：上海

# 遍历嵌套字典
for name, info in students.items():
    print(f"{name}：{info['age']}岁，成绩{info['score']}，来自{info['city']}")
```

***

#### 字典方法速查表

| 方法                    | 功能        | 示例                     |
| --------------------- | --------- | ---------------------- |
| `d[key]`              | 取值（不存在报错） | `d["name"]`            |
| `d.get(key, default)` | 安全取值      | `d.get("age", 0)`      |
| `d[key] = val`        | 添加/修改     | `d["x"] = 1`           |
| `del d[key]`          | 删除键值对     | `del d["x"]`           |
| `d.pop(key)`          | 删除并返回值    | `d.pop("x")`           |
| `d.keys()`            | 所有键       | `list(d.keys())`       |
| `d.values()`          | 所有值       | `list(d.values())`     |
| `d.items()`           | 所有键值对     | `for k,v in d.items()` |
| `d.update(d2)`        | 合并字典      | `d.update({"a":1})`    |
| `key in d`            | 判断键是否存在   | `"name" in d`          |

***

### 🎯 集合（6 分钟）

#### 什么是集合？

集合有两大特点：

1. **元素不重复**（自动去除重复项）
2. **无序**（不能用索引访问）

```python
# 用 {} 创建集合（注意：{} 是空字典，空集合必须用 set()）
fruits = {"苹果", "香蕉", "橙子", "苹果", "香蕉"}
print(fruits)   # 输出：{'苹果', '香蕉', '橙子'}（重复项被自动去除）

# 空集合（不能用 {}）
empty = set()

# 把列表转为集合（常用于去重！）
numbers = [1, 2, 2, 3, 3, 3, 4]
unique = set(numbers)
print(unique)   # 输出：{1, 2, 3, 4}

# 再转回列表
unique_list = list(set(numbers))
print(unique_list)   # 输出：[1, 2, 3, 4]（顺序可能不同）
```

> ⚠️ 集合是**无序的**，每次打印顺序可能不同，不能用 `s[0]` 取值。

***

#### 集合的增删与判断

```python
s = {"苹果", "香蕉", "橙子"}

# 添加元素
s.add("葡萄")
print(s)   # 输出：{'苹果', '香蕉', '橙子', '葡萄'}

# 删除元素（不存在会报错）
s.remove("香蕉")

# 安全删除（不存在不报错）
s.discard("西瓜")   # 没有西瓜，但不报错

# 判断成员
print("苹果" in s)   # 输出：True
print(len(s))        # 输出：3
```

***

#### 集合运算

集合最强大的地方是支持数学集合运算，非常直观：

```python
python_fans = {"小明", "小红", "小李", "小张"}   # 学 Python 的同学
ai_fans     = {"小红", "小张", "小王", "小赵"}   # 学 AI 的同学

# 并集（两组中所有人）
print(python_fans | ai_fans)
# {'小明', '小红', '小李', '小张', '小王', '小赵'}

# 交集（两组都有的人）
print(python_fans & ai_fans)
# {'小红', '小张'}

# 差集（只学 Python、不学 AI 的人）
print(python_fans - ai_fans)
# {'小明', '小李'}

# 对称差集（只属于其中一组的人）
print(python_fans ^ ai_fans)
# {'小明', '小李', '小王', '小赵'}
```

**集合运算速查**

| 运算   | 符号   | 方法                        | 含义       |
| ---- | ---- | ------------------------- | -------- |
| 并集   | `\|` | `.union()`                | 合并所有元素   |
| 交集   | `&`  | `.intersection()`         | 共同拥有的元素  |
| 差集   | `-`  | `.difference()`           | 前者有、后者没有 |
| 对称差集 | `^`  | `.symmetric_difference()` | 只属于其中一个  |

***

#### 四种容器横向对比

```python
# 列表：有序、可重复、可修改
lst   = [1, 2, 2, 3]       # ✅ 有索引  ✅ 可修改  ✅ 允许重复

# 元组：有序、可重复、不可修改
tpl   = (1, 2, 2, 3)       # ✅ 有索引  ❌ 不可修改 ✅ 允许重复

# 字典：有序（Python 3.7+）、键不重复、可修改
dct   = {"a": 1, "b": 2}   # ✅ 键查找  ✅ 可修改  ❌ 键不重复

# 集合：无序、不重复、可修改
st    = {1, 2, 3}           # ❌ 无索引  ✅ 可修改  ❌ 不重复
```

|            | 有序 | 可修改 | 允许重复  | 查找方式 |
| ---------- | -- | --- | ----- | ---- |
| 列表 `list`  | ✅  | ✅   | ✅     | 索引   |
| 元组 `tuple` | ✅  | ❌   | ✅     | 索引   |
| 字典 `dict`  | ✅  | ✅   | 键❌ 值✅ | 键    |
| 集合 `set`   | ❌  | ✅   | ❌     | `in` |

***

### 🏁 综合练习（7 分钟）

#### 练习：学生成绩管理系统

综合运用今天所学的四种容器，写一个小型成绩管理程序：

```python
# ============================================
# 学生成绩管理系统
# ============================================

# ---------- 数据准备 ----------
# 用字典存储学生成绩（姓名 → 成绩）
scores = {
    "小明": 92,
    "小红": 85,
    "小李": 73,
    "小张": 95,
    "小王": 68,
    "小赵": 55,
}

# ---------- 基础统计 ----------
score_list = list(scores.values())    # 所有成绩转为列表

total   = sum(score_list)
average = total / len(score_list)
highest = max(score_list)
lowest  = min(score_list)

print("=" * 36)
print(f"{'📊 成绩统计报告':^30}")
print("=" * 36)
print(f"  参考人数：{len(scores)} 人")
print(f"  平均分：  {average:.1f} 分")
print(f"  最高分：  {highest} 分")
print(f"  最低分：  {lowest} 分")
print("-" * 36)

# ---------- 等级划分（用字典存等级规则） ----------
def get_grade(score):
    if score >= 90:
        return "优秀 ⭐"
    elif score >= 75:
        return "良好 ✅"
    elif score >= 60:
        return "及格 📝"
    else:
        return "不及格 ❌"

# ---------- 打印个人成绩 ----------
print(f"  {'姓名':<6} {'成绩':>6} {'等级'}")
print("-" * 36)

# 按成绩从高到低排序（sorted 不修改原字典）
sorted_scores = sorted(scores.items(), key=lambda x: x[1], reverse=True)

for rank, (name, score) in enumerate(sorted_scores, start=1):
    grade = get_grade(score)
    print(f"  {rank}. {name:<5} {score:>5} 分   {grade}")

# ---------- 集合应用：分组 ----------
print("-" * 36)
pass_set = {name for name, s in scores.items() if s >= 60}
fail_set = set(scores.keys()) - pass_set

print(f"  ✅ 及格人数：{len(pass_set)} 人 → {', '.join(sorted(pass_set))}")
print(f"  ❌ 不及格：  {len(fail_set)} 人 → {', '.join(sorted(fail_set))}")
print("=" * 36)
```

**示例输出：**

```
====================================
          📊 成绩统计报告           
====================================
  参考人数：6 人
  平均分：  78.0 分
  最高分：  95 分
  最低分：  55 分
------------------------------------
  姓名     成绩 等级
------------------------------------
  1. 小张     95 分   优秀 ⭐
  2. 小明     92 分   优秀 ⭐
  3. 小红     85 分   良好 ✅
  4. 小李     73 分   及格 📝
  5. 小王     68 分   及格 📝
  6. 小赵     55 分   不及格 ❌
------------------------------------
  ✅ 及格人数：5 人 → 小张、小明、小红、小李、小王
  ❌ 不及格：  1 人 → 小赵
====================================
```

***

### 📝 本课知识总结

```
列表 list []
├── 创建：[1, 2, 3] 或 list()
├── 索引/切片：与字符串相同
├── 增：append() / insert()
├── 删：remove() / pop() / del
├── 查：index() / count() / in
├── 改：lst[i] = 新值
├── 排序：sort() / sorted()
└── 推导式：[x*2 for x in lst if x > 0]

元组 tuple ()
├── 创建：(1, 2, 3) 或 单元素必须加逗号 (1,)
├── 不可修改，支持索引/切片
└── 解包：a, b = (1, 2)

字典 dict {}
├── 创建：{"key": value} 或 dict()
├── 读：d[key] / d.get(key, default)
├── 写：d[key] = value / d.update()
├── 删：del d[key] / d.pop(key)
├── 遍历：for k, v in d.items()
└── 查：key in d

集合 set
├── 创建：{1, 2, 3} 或 set()，空集合必须用 set()
├── 特点：无序、不重复，常用于去重
├── 增删：add() / remove() / discard()
└── 运算：| 并集  & 交集  - 差集  ^ 对称差集
```

***

### 🎯 课后作业

**练习 1（基础）：** 有一个列表 `nums = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]`，完成：

1. 用集合对其去重，再转回列表并升序排列，打印结果
2. 统计数字 `5` 出现了几次
3. 用列表推导式，生成一个只包含其中偶数的新列表

**练习 2（进阶）：** 写一个"购物车"程序：

* 用列表存储购物车中的商品名
* 用字典存储商品价格（`{"苹果": 3.5, "香蕉": 2.0, "橙子": 4.5}`）
* 程序循环接收用户输入的商品名，加入购物车（输入 `q` 退出）
* 最后打印购物车清单，并计算总价
* 提示：用 `dict.get()` 处理商品不存在的情况

**练习 3（挑战）：** 写一个"词频统计"程序：

* 给定一段英文文本（字符串）
* 将文本按空格分割为单词列表，全部转为小写
* 用字典统计每个单词出现的次数
* 找出出现次数最多的前 3 个单词并打印
* 提示：可用 `sorted(word_count.items(), key=lambda x: x[1], reverse=True)`

***

### 🔜 下节课预告

**第五课：**&#x6761;件判断与循环控制

```python
# 成绩列表
scores = [95, 72, 55, 88]

# 使用循环逐个处理成绩
for s in scores:
    # 条件判断
    if s >= 90:
        print(f"{s} 分 → 优秀")
    elif s >= 60:
        print(f"{s} 分 → 及格")
    else:
        print(f"{s} 分 → 不及格")
```

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 4 课 / 共 10 课 Power by Andrew Liu_
