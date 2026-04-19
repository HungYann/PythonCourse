# 第五课：条件判断与循环控制

### 第五课：Python 基础（从零起飞）- 条件判断与循环控制

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01～04 课** 🛠️ **工具准备：Python 3.x 或** [**在线运行环境**](https://colab.research.google.com/)

***

### 课程导览

| 模块          | 内容                             | 时间    |
| ----------- | ------------------------------ | ----- |
| 🔥 热身回顾     | 程序为什么需要"判断"和"重复"？              | 3 分钟  |
| 🔀 条件判断     | if / elif / else、嵌套、三元表达式      | 12 分钟 |
| 🔁 while 循环 | 基本用法、无限循环、循环控制                 | 8 分钟  |
| 🔂 for 循环   | 遍历序列、range()、enumerate()、zip() | 10 分钟 |
| ⚙️ 循环控制     | break / continue / else / pass | 7 分钟  |
| 🏁 综合练习     | 猜数字游戏 + 成绩统计                   | 5 分钟  |

***

### 🔥 热身回顾（3 分钟）

前四课的代码都是**从上到下直线执行**的——每一行只跑一次，没有选择，没有重复。但真实程序需要做决策、需要重复处理数据：

```python
# 没有判断和循环，只能这样写：
score1 = 95
if score1 >= 60:
    print("小明：及格")

score2 = 45
if score2 >= 60:
    print("小红：及格")

# 如果有 100 个同学……要写 200 行？🤯
```

掌握**条件判断**和**循环**，你的程序才真正"活"起来：

```
顺序执行（目前）        条件判断（本课）        循环（本课）
                       ┌─ True ──> 执行A ─┐    ┌──────────┐
  A → B → C → D       │                   ├──  │ 重复执行  │◄─┐
                       └─ False ─> 执行B ─┘    └──────────┘  │
                                                      └────────┘
```

***

### 🔀 条件判断（12 分钟）

#### 基本 if 语句

```python
score = 85

if score >= 60:
    print("及格了！")
```

**执行逻辑：**

```
score >= 60 ?
    ├── True  → 执行 print("及格了！")
    └── False → 跳过，什么都不做
```

> ⚠️ **注意缩进！** Python 用**4 个空格**的缩进来表示代码块，这是强制规定。缩进错误会导致程序报错。

```python
# ❌ 错误：忘记缩进
if score >= 60:
print("及格了！")   # IndentationError！

# ✅ 正确：4 个空格缩进
if score >= 60:
    print("及格了！")
```

***

#### if / else

```python
score = 45

if score >= 60:
    print("✅ 及格")
else:
    print("❌ 不及格，需要努力！")
```

***

#### if / elif / else

处理多个分支，`elif` 是 "else if" 的缩写：

```python
score = 82

if score >= 90:
    print("优秀 ⭐")
elif score >= 75:
    print("良好 ✅")
elif score >= 60:
    print("及格 📝")
else:
    print("不及格 ❌")

# 输出：良好 ✅
```

**执行逻辑：**

```
score = 82
  │
  ├── >= 90 ? False → 跳过
  ├── >= 75 ? True  → 执行 print("良好 ✅") ──→ 结束，后面不再判断
  ├── >= 60 ?        （不会到达这里）
  └── else           （不会到达这里）
```

> 💡 `elif` 可以有任意多个；`else` 可以省略。一旦某个条件为 `True`，执行完该分支后**立即跳出**整个 if 结构，不再检查后面的条件。

***

#### 条件判断中的运算符

**比较运算符（返回 True / False）**

```python
x = 10

print(x > 5)     # True
print(x < 5)     # False
print(x >= 10)   # True
print(x <= 9)    # False
print(x == 10)   # True（等于用 ==，不是 =）
print(x != 10)   # False（不等于）
```

**逻辑运算符**

```python
age = 20
has_id = True

# and：两个条件都为 True，结果才是 True
print(age >= 18 and has_id)   # True

# or：至少一个条件为 True，结果就是 True
print(age >= 18 or has_id)    # True

# not：取反
print(not has_id)             # False
```

```python
# 实际案例：判断是否可以入场
age = 17
is_vip = True

if age >= 18 or is_vip:
    print("欢迎入场！")
else:
    print("未成年且非 VIP，谢绝入场。")

# 输出：欢迎入场！（因为是 VIP）
```

**成员运算符（in / not in）**

```python
fruits = ["苹果", "香蕉", "橙子"]

if "苹果" in fruits:
    print("有苹果！")       # 输出：有苹果！

if "西瓜" not in fruits:
    print("没有西瓜。")     # 输出：没有西瓜。

# 字符串中也可以用
email = "user@example.com"
if "@" in email:
    print("邮箱格式初步有效")
```

***

#### 嵌套 if

if 语句内部可以再嵌套 if，但**不建议超过 3 层**，否则代码难以阅读：

```python
score = 88
is_homework_done = True

if score >= 60:
    if is_homework_done:
        print("成绩及格，作业完成，可以玩游戏了！🎮")
    else:
        print("成绩及格，但作业没完成，先写作业！📚")
else:
    print("成绩不及格，需要补考。📝")
```

> 💡 **优化技巧：** 嵌套 if 通常可以用 `and` 改写得更扁平：
>
> ```python
> if score >= 60 and is_homework_done:
>     print("可以玩游戏了！🎮")
> elif score >= 60:
>     print("先写作业！📚")
> else:
>     print("需要补考。📝")
> ```

***

#### 三元表达式（条件表达式）

Python 的"一行版 if-else"，适合简单的二选一赋值：

```python
# 语法：值A  if  条件  else  值B
score = 75
result = "及格" if score >= 60 else "不及格"
print(result)   # 输出：及格

# 对比普通写法（等价）
if score >= 60:
    result = "及格"
else:
    result = "不及格"
```

```python
# 更多例子
age = 20
label = "成年人" if age >= 18 else "未成年"

num = -5
abs_val = num if num >= 0 else -num    # 手动实现绝对值

name = input("输入名字：").strip()
display = name if name else "匿名用户"  # 空字符串时使用默认值
```

> ⚠️ **适度使用：** 三元表达式适合简单判断，复杂逻辑还是写完整 if-else，可读性更重要。

***

### 🔁 while 循环（8 分钟）

#### 基本语法

`while` 的含义是"**当……条件为真时，一直重复执行**"：

```python
# 语法
while 条件:
    循环体（条件为 True 时反复执行）
```

```python
count = 1

while count <= 5:
    print(f"第 {count} 次")
    count += 1       # ⚠️ 必须更新条件变量，否则无限循环！

print("循环结束")
```

**输出：**

```
第 1 次
第 2 次
第 3 次
第 4 次
第 5 次
循环结束
```

**执行流程：**

```
count=1 → 1<=5? True  → 打印 → count=2
count=2 → 2<=5? True  → 打印 → count=3
count=3 → 3<=5? True  → 打印 → count=4
count=4 → 4<=5? True  → 打印 → count=5
count=5 → 5<=5? True  → 打印 → count=6
count=6 → 6<=5? False → 退出循环
```

***

#### while 的典型场景：用户输入验证

`while` 非常适合"直到用户输入正确才继续"的场景：

```python
# 持续要求输入，直到输入有效值
while True:
    age_input = input("请输入年龄（1-120）：")

    if age_input.isdigit():
        age = int(age_input)
        if 1 <= age <= 120:
            break                          # 输入有效，跳出循环
        else:
            print("❌ 年龄超出范围，请重新输入。")
    else:
        print("❌ 请输入数字。")

print(f"✅ 你的年龄是 {age} 岁。")
```

***

#### 计数器与累加器

`while` 常见的两种使用模式：

```python
# 模式一：计数器（数"做了几次"）
attempts = 0
while attempts < 3:
    print(f"第 {attempts + 1} 次尝试")
    attempts += 1

# 模式二：累加器（积累结果）
total = 0
n = 1
while n <= 100:
    total += n      # 累加 1 + 2 + 3 + ... + 100
    n += 1

print(f"1 到 100 的总和：{total}")   # 输出：1 到 100 的总和：5050
```

> ⚠️ **无限循环陷阱：** 如果循环条件永远为 `True` 且没有 `break`，程序会卡死。按 `Ctrl + C` 可强制终止。
>
> ```python
> # ❌ 这是无限循环！忘记更新 n
> n = 1
> while n <= 5:
>     print(n)
>     # 忘写 n += 1，n 永远是 1，循环不会停
> ```

***

### 🔂 for 循环（10 分钟）

#### 基本语法

`for` 用于**遍历一个序列**（列表、字符串、字典等），每次取出一个元素：

```python
# 语法
for 变量 in 可迭代对象:
    循环体
```

```python
fruits = ["苹果", "香蕉", "橙子"]

for fruit in fruits:
    print(f"我喜欢吃{fruit}")
```

**输出：**

```
我喜欢吃苹果
我喜欢吃香蕉
我喜欢吃橙子
```

***

#### 遍历字符串

字符串也是可以遍历的序列，每次取出一个字符：

```python
for char in "Python":
    print(char, end=" ")   # end=" " 让输出不换行

# 输出：P y t h o n
```

***

#### range() 函数

`range()` 生成一个数字序列，是 `for` 循环的黄金搭档：

```python
# range(stop)：从 0 到 stop-1
for i in range(5):
    print(i, end=" ")
# 输出：0 1 2 3 4

# range(start, stop)：从 start 到 stop-1
for i in range(2, 6):
    print(i, end=" ")
# 输出：2 3 4 5

# range(start, stop, step)：带步长
for i in range(0, 10, 2):
    print(i, end=" ")
# 输出：0 2 4 6 8

# 倒数（步长为负）
for i in range(5, 0, -1):
    print(i, end=" ")
# 输出：5 4 3 2 1
```

**`range()` 速查：**

| 写法                 | 生成序列           |
| ------------------ | -------------- |
| `range(5)`         | 0, 1, 2, 3, 4  |
| `range(1, 6)`      | 1, 2, 3, 4, 5  |
| `range(0, 10, 2)`  | 0, 2, 4, 6, 8  |
| `range(5, 0, -1)`  | 5, 4, 3, 2, 1  |
| `range(10, 0, -2)` | 10, 8, 6, 4, 2 |

***

#### enumerate()：同时获取索引和值

遍历列表时，如果**同时需要索引和值**，用 `enumerate()` 而不是 `range(len(...))`：

```python
students = ["小明", "小红", "小李", "小张"]

# ❌ 旧写法：通过索引间接取值
for i in range(len(students)):
    print(f"第{i+1}名：{students[i]}")

# ✅ 推荐：enumerate() 直接同时拿到
for i, name in enumerate(students, start=1):   # start=1 让编号从 1 开始
    print(f"第{i}名：{name}")
```

**输出：**

```
第1名：小明
第2名：小红
第3名：小李
第4名：小张
```

***

#### zip()：同时遍历多个序列

```python
names  = ["小明", "小红", "小李"]
scores = [95,     87,     72    ]
cities = ["北京", "上海", "广州"]

for name, score, city in zip(names, scores, cities):
    print(f"{name}（{city}）：{score} 分")
```

**输出：**

```
小明（北京）：95 分
小红（上海）：87 分
小李（广州）：72 分
```

> 💡 `zip()` 以**最短的序列**为准，多余的元素会被忽略。

***

#### 遍历字典

```python
student = {"name": "小明", "age": 18, "score": 95}

# 遍历键（默认）
for key in student:
    print(key)

# 遍历值
for value in student.values():
    print(value)

# 同时遍历键和值（最常用 ✅）
for key, value in student.items():
    print(f"{key} → {value}")
```

**输出：**

```
name → 小明
age → 18
score → 95
```

***

#### for vs while 如何选择？

```
需要遍历一个序列（列表/字符串/字典）？
    └── 用 for

循环次数已知（比如"重复 10 次"）？
    └── 用 for + range()

循环次数未知，取决于某个条件？
    └── 用 while

等待用户输入、等待特定事件？
    └── 用 while True + break
```

***

### ⚙️ 循环控制（7 分钟）

#### break：立即退出循环

```python
# 在列表中找到第一个大于 80 的成绩，找到即停止
scores = [65, 72, 88, 55, 90, 78]

for score in scores:
    if score > 80:
        print(f"找到了：{score}")
        break                   # 找到后立即退出，不再继续
    print(f"{score} 不符合")

# 输出：
# 65 不符合
# 72 不符合
# 找到了：88
```

***

#### continue：跳过本次，继续下一次

```python
# 只打印偶数，跳过奇数
for i in range(1, 11):
    if i % 2 != 0:
        continue       # 奇数：跳过本次循环的剩余部分
    print(i, end=" ")

# 输出：2 4 6 8 10
```

```python
# 跳过不及格的成绩
scores = [95, 42, 87, 55, 76]

print("及格成绩：", end="")
for score in scores:
    if score < 60:
        continue        # 不及格就跳过
    print(score, end=" ")

# 输出：及格成绩：95 87 76
```

***

#### break 和 continue 的区别

```
循环序列：[1, 2, 3, 4, 5]，在 3 时触发

使用 break：
  1 → 2 → 3（触发 break）→ 退出循环
  输出：1 2

使用 continue：
  1 → 2 → 3（触发 continue，跳过 3）→ 4 → 5
  输出：1 2 4 5
```

```python
# 验证：break
for i in [1, 2, 3, 4, 5]:
    if i == 3:
        break
    print(i, end=" ")
# 输出：1 2

# 验证：continue
for i in [1, 2, 3, 4, 5]:
    if i == 3:
        continue
    print(i, end=" ")
# 输出：1 2 4 5
```

***

#### for / while … else

Python 独有的语法：循环**正常结束**（没有被 `break` 打断）时，执行 `else` 块：

```python
# 在列表中查找某个值
numbers = [1, 3, 5, 7, 9]
target = 6

for num in numbers:
    if num == target:
        print(f"找到了：{num}")
        break
else:
    # 循环完整执行完毕，没有触发 break
    print(f"没有找到 {target}")

# 输出：没有找到 6
```

```python
# 找到时触发 break，else 不执行
target = 5
for num in numbers:
    if num == target:
        print(f"找到了：{num}")
        break
else:
    print("没有找到")

# 输出：找到了：5（else 不执行）
```

> 💡 `for...else` / `while...else` 是 Python 特有的，其他语言没有这个语法。常用于"搜索"场景：找到了用 `break`，找不到走 `else`。

***

#### pass：占位符

`pass` 什么都不做，仅用于**语法上需要一个语句但暂时不写内容**的地方：

```python
# 暂时还没想好 else 分支怎么写
score = 85
if score >= 60:
    print("及格")
else:
    pass    # TODO：后续补充不及格的处理逻辑

# 空循环体（占位）
for i in range(10):
    pass    # 先搭结构，内容后续填充
```

***

#### 循环控制关键字速查

| 关键字        | 作用           | 使用场景       |
| ---------- | ------------ | ---------- |
| `break`    | 立即退出整个循环     | 找到目标后提前退出  |
| `continue` | 跳过本次迭代，进入下一次 | 过滤不需要处理的元素 |
| `else`     | 循环正常结束时执行    | 搜索失败的处理逻辑  |
| `pass`     | 什么都不做（占位）    | 代码框架搭建阶段   |

***

### 🏁 综合练习（5 分钟）

#### 练习：猜数字游戏

综合运用本课所有知识点：

```python
# ============================================
# 猜数字游戏
# ============================================
import random

# ---------- 初始化 ----------
secret = random.randint(1, 100)     # 随机生成 1~100 的秘密数字
max_attempts = 7                     # 最多猜 7 次
attempts = 0                         # 已用次数

print("=" * 40)
print(f"{'🎮 猜数字游戏':^34}")
print("=" * 40)
print(f"  我想了一个 1～100 之间的数字")
print(f"  你有 {max_attempts} 次机会，加油！")
print("=" * 40)

# ---------- 主循环 ----------
while attempts < max_attempts:
    attempts += 1
    remaining = max_attempts - attempts   # 剩余次数

    # 获取并验证用户输入
    raw = input(f"\n第 {attempts} 次猜（还剩 {remaining} 次）：")

    if not raw.strip().isdigit():
        print("  ❌ 请输入有效的数字！")
        attempts -= 1                     # 无效输入不计入次数
        continue

    guess = int(raw.strip())

    # 范围检查
    if not (1 <= guess <= 100):
        print("  ❌ 请输入 1～100 之间的数字！")
        attempts -= 1
        continue

    # 判断结果
    if guess == secret:
        print(f"\n  🎉 恭喜你，猜对了！答案就是 {secret}！")
        print(f"  你用了 {attempts} 次猜出来。", end=" ")
        if attempts <= 3:
            print("太厉害了！🏆")
        elif attempts <= 5:
            print("不错哦！👍")
        else:
            print("险险过关！😅")
        break

    elif guess < secret:
        diff = secret - guess
        hint = "差一点点 🔥" if diff <= 5 else ("有点小了 📈" if diff <= 20 else "太小了 ⬆️")
        print(f"  {hint}，再大一点！")

    else:
        diff = guess - secret
        hint = "差一点点 🔥" if diff <= 5 else ("有点大了 📉" if diff <= 20 else "太大了 ⬇️")
        print(f"  {hint}，再小一点！")

else:
    # while 循环正常结束（没有 break），说明用完了所有次数
    print(f"\n  😢 游戏结束，你没有猜对。答案是 {secret}。")
    print("  再来一局？重新运行程序即可！")

print("=" * 40)
```

**示例运行过程：**

```
========================================
          🎮 猜数字游戏          
========================================
  我想了一个 1～100 之间的数字
  你有 7 次机会，加油！
========================================

第 1 次猜（还剩 6 次）：50
  太小了 ⬆️，再大一点！

第 2 次猜（还剩 5 次）：75
  有点大了 📉，再小一点！

第 3 次猜（还剩 4 次）：62
  差一点点 🔥，再大一点！

第 4 次猜（还剩 3 次）：65
  🎉 恭喜你，猜对了！答案就是 65！
  你用了 4 次猜出来。 不错哦！👍
========================================
```

***

### 📝 本课知识总结

```
条件判断
├── if / elif / else（多分支）
├── 比较运算符：> < >= <= == !=
├── 逻辑运算符：and / or / not
├── 成员运算符：in / not in
├── 三元表达式：值A if 条件 else 值B
└── 嵌套 if（建议不超过 3 层）

while 循环
├── 语法：while 条件: 循环体
├── 适合：次数未知、等待条件满足
├── 经典模式：while True + break
└── ⚠️ 必须确保条件最终变为 False

for 循环
├── 语法：for 变量 in 可迭代对象:
├── 适合：遍历序列、已知次数
├── range(start, stop, step)：生成数字序列
├── enumerate(seq, start=0)：同时获取索引和值
└── zip(a, b, c)：并行遍历多个序列

循环控制
├── break     → 立即退出整个循环
├── continue  → 跳过本次，继续下一次
├── for/else  → 循环未被 break 时执行 else
└── pass      → 占位，什么都不做
```

***

### 🎯 课后作业

**练习 1（基础）：** 用 `for` 循环和 `range()` 打印以下图案：

```
*
**
***
****
*****
```

提示：外层循环控制行数，内层用 `*` 乘法或再套一层循环。

**练习 2（进阶）：** 写一个"成绩批量评级"程序：

* 用列表存储 10 个学生的成绩：`[92, 55, 78, 66, 88, 45, 95, 73, 81, 60]`
* 用 `for` 循环遍历，用 `if/elif/else` 判断等级（优秀/良好/及格/不及格）
* 统计各等级人数，最后打印汇总报告
* 提示：用字典存储等级计数 `{"优秀": 0, "良好": 0, ...}`

**练习 3（挑战）：** 写一个"ATM 模拟器"：

* 初始余额 1000 元
* 用 `while True` 主循环，显示菜单：1. 查余额 2. 存款 3. 取款 4. 退出
* 取款时：若余额不足打印提示，不允许取款
* 输入非法选项时打印错误提示并继续循环
* 输入 4 退出，打印最终余额

***

### 🔜 下节课预告

**第六课：函数——让代码可以复用**

```python
# 你将学会把重复的逻辑封装成函数，一次定义，到处调用：
def calculate_grade(score):
    """根据分数返回等级和提示"""
    if score >= 90:
        return "优秀", "🏆 继续保持！"
    elif score >= 75:
        return "良好", "👍 再加把劲！"
    elif score >= 60:
        return "及格", "📝 还需努力！"
    else:
        return "不及格", "❌ 需要补考！"

scores = [95, 72, 55, 88, 66]
for s in scores:
    grade, tip = calculate_grade(s)
    print(f"{s} 分 → {grade}  {tip}")
```

***

> 💬 **有疑问？** 欢迎在评论区留言或加入学习群讨论。
>
> ⭐ 觉得有帮助的话，别忘了分享给朋友！

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 5 课 / 共 10 课 Power by Andrew Liu_





