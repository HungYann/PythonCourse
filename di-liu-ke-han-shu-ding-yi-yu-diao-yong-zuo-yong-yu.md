# 第六课：函数定义与调用、作用域

### 第六课：Python 基础（从零起飞）- 函数定义与调用、作用域

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01～05 课** 🛠️ **工具准备：Python 3.x 或** [**在线运行环境**](https://colab.research.google.com/)

***

### 课程导览

| 模块       | 内容                | 时间    |
| -------- | ----------------- | ----- |
| 🔥 热身回顾  | 为什么需要函数？重复代码的问题   | 3 分钟  |
| 🧱 函数基础  | 定义、调用、参数、返回值      | 12 分钟 |
| 🎛️ 参数进阶 | 默认参数、关键字参数、可变参数   | 10 分钟 |
| 🔭 作用域   | 局部变量、全局变量、LEGB 规则 | 8 分钟  |
| 🏁 综合练习  | 函数化成绩管理系统         | 7 分钟  |

***

### 🔥 热身回顾（3 分钟）

第五课我们写了猜数字游戏。回顾其中的"提示"逻辑，它在猜小了和猜大了两个分支里**各写了一遍**几乎相同的代码：

```python
# 第五课猜数字游戏中的重复逻辑
elif guess < secret:
    diff = secret - guess
    hint = "差一点点 🔥" if diff <= 5 else ("有点小了 📈" if diff <= 20 else "太小了 ⬆️")
    print(f"  {hint}，再大一点！")
else:
    diff = guess - secret
    hint = "差一点点 🔥" if diff <= 5 else ("有点大了 📉" if diff <= 20 else "太大了 ⬇️")
    print(f"  {hint}，再小一点！")
```

两段逻辑高度重复——计算差值、判断远近、拼接提示语，只有方向文字不同。若要修改距离阈值，就得改两处。用函数封装后，只需改一处：

```python
# ✅ 用函数封装重复逻辑：定义一次，两处调用
def get_proximity_hint(diff):
    """根据差值距离返回远近提示"""
    if diff <= 5:    return "差一点点 🔥"
    elif diff <= 20: return "有点偏了 📈"
    else:            return "差很远 ⬆️"

# 猜小了和猜大了各调用一次，不再重复写判断逻辑
hint = get_proximity_hint(abs(guess - secret))
```

函数是**代码复用**的基本单元，也是从"写脚本"迈向"写程序"的第一步：

```
没有函数                     有了函数
─────────────────────        ─────────────────────────────
代码 A                       def get_grade(score): ...
代码 B（重复 A）          →  
代码 C（重复 A）             get_grade(92)  ← 调用
代码 D（重复 A）             get_grade(67)  ← 调用
                             get_grade(55)  ← 调用
```

***

### 🧱 函数基础（12 分钟）

#### 定义与调用

```python
# 定义语法
def 函数名(参数列表):
    """文档字符串（可选，描述函数用途）"""
    函数体
    return 返回值     # 可选
```

```python
# 最简单的函数：无参数，无返回值
def say_hello():
    print("你好，Python！")

say_hello()    # 调用：输出 → 你好，Python！
say_hello()    # 可以多次调用
```

> 💡 定义函数只是"写好了菜谱"，**调用**才是"真正开始做菜"。定义时函数体不会执行，只有被调用时才运行。

***

#### 参数：向函数传递数据

```python
# 带参数的函数
def greet(name):
    print(f"你好，{name}！欢迎来到 Python 世界！")

greet("小明")    # 输出：你好，小明！欢迎来到 Python 世界！
greet("小红")    # 输出：你好，小红！欢迎来到 Python 世界！
```

```python
# 多个参数
def introduce(name, age, city):
    print(f"我叫{name}，{age}岁，来自{city}。")

introduce("小明", 18, "北京")
# 输出：我叫小明，18岁，来自北京。
```

**参数传递图示：**

```
introduce("小明", 18, "北京")
              │      │      │
              ▼      ▼      ▼
def introduce(name, age, city):
    # name = "小明"
    # age  = 18
    # city = "北京"
```

***

#### 返回值：从函数取出结果

`return` 语句将结果"送出"函数，调用方可以接收并使用：

```python
def add(a, b):
    result = a + b
    return result          # 将结果返回给调用方

total = add(3, 5)          # 接收返回值
print(total)               # 输出：8
print(add(10, 20))         # 直接使用：输出 30
```

```python
# 没有 return，函数默认返回 None
def say_hi():
    print("Hi!")

result = say_hi()          # 输出：Hi!
print(result)              # 输出：None
```

**return 的两个作用：**

```
return 的作用
├── 1. 把值传递给调用方
└── 2. 立即结束函数执行（return 后面的代码不再运行）
```

```python
def check_age(age):
    if age < 0:
        return "年龄无效"     # 提前返回，下面的代码不执行
    if age < 18:
        return "未成年"
    return "成年"             # 正常返回

print(check_age(-1))   # 输出：年龄无效
print(check_age(15))   # 输出：未成年
print(check_age(20))   # 输出：成年
```

***

#### 返回多个值

Python 函数可以同时返回多个值（本质是返回一个元组）：

```python
def get_min_max(numbers):
    return min(numbers), max(numbers)   # 返回元组

low, high = get_min_max([3, 1, 4, 1, 5, 9, 2, 6])
print(f"最小值：{low}，最大值：{high}")
# 输出：最小值：1，最大值：9
```

***

#### 文档字符串（Docstring）

用三引号写在函数体第一行，描述函数用途、参数和返回值：

```python
def calculate_area(radius):
    """
    计算圆的面积。

    参数：
        radius (float): 圆的半径，必须为正数

    返回：
        float: 圆的面积
    """
    return 3.14159 * radius ** 2

# 查看文档
help(calculate_area)
print(calculate_area.__doc__)
```

> 💡 养成写 Docstring 的好习惯——三个月后的你会感谢现在的自己。

***

### 🎛️ 参数进阶（10 分钟）

#### 默认参数

定义函数时给参数设置默认值，调用时可以省略该参数：

```python
def greet(name, greeting="你好"):    # greeting 有默认值
    print(f"{greeting}，{name}！")

greet("小明")              # 使用默认值：输出 → 你好，小明！
greet("小红", "早上好")    # 覆盖默认值：输出 → 早上好，小红！
greet("小李", "晚安")      # 输出：晚安，小李！
```

```python
# 实用案例：连接数据库
def connect_db(host, port=3306, charset="utf8"):
    print(f"连接 {host}:{port}，编码：{charset}")

connect_db("localhost")                        # 用两个默认值
connect_db("192.168.1.1", 5432)               # 覆盖 port
connect_db("127.0.0.1", charset="utf8mb4")    # 只覆盖 charset
```

> ⚠️ **默认参数必须放在非默认参数后面！**
>
> ```python
> # ❌ 错误：非默认参数在默认参数后面
> def greet(greeting="你好", name):   # SyntaxError!
>     pass
>
> # ✅ 正确：默认参数放最后
> def greet(name, greeting="你好"):
>     pass
> ```

***

#### 关键字参数

调用时用 `参数名=值` 的形式传参，可以不按顺序：

```python
def order_coffee(size, milk, sugar):
    print(f"尺寸：{size}，牛奶：{milk}，糖：{sugar}")

# 位置参数（必须按顺序）
order_coffee("大杯", "全脂", "少糖")

# 关键字参数（顺序随意）
order_coffee(sugar="无糖", size="中杯", milk="燕麦奶")

# 混合使用（位置参数必须在关键字参数前）
order_coffee("大杯", sugar="无糖", milk="脱脂")
```

***

#### 可变位置参数 `*args`

当参数数量不确定时，用 `*args` 接收任意数量的位置参数，函数内部得到一个**元组**：

```python
def total(*args):
    """计算任意数量数字的总和"""
    print(f"收到参数：{args}")     # args 是元组
    return sum(args)

print(total(1, 2))               # 收到参数：(1, 2)  → 3
print(total(1, 2, 3, 4, 5))     # 收到参数：(1, 2, 3, 4, 5) → 15
print(total())                   # 收到参数：()  → 0
```

***

#### 可变关键字参数 `**kwargs`

用 `**kwargs` 接收任意数量的关键字参数，函数内部得到一个**字典**：

```python
def show_info(**kwargs):
    """打印任意个键值对信息"""
    print(f"收到参数：{kwargs}")   # kwargs 是字典
    for key, value in kwargs.items():
        print(f"  {key}: {value}")

show_info(name="小明", age=18, city="北京")
# 收到参数：{'name': '小明', 'age': 18, 'city': '北京'}
#   name: 小明
#   age: 18
#   city: 北京
```

***

#### 参数类型总览

```python
def demo(a, b, c=10, *args, **kwargs):
    print(f"普通参数: a={a}, b={b}")
    print(f"默认参数: c={c}")
    print(f"可变位置: args={args}")
    print(f"可变关键字: kwargs={kwargs}")

demo(1, 2, 3, 4, 5, x=6, y=7)
# 普通参数: a=1, b=2
# 默认参数: c=3
# 可变位置: args=(4, 5)
# 可变关键字: kwargs={'x': 6, 'y': 7}
```

**参数定义顺序规则：**

```
def func( 普通参数, 默认参数=值, *args, **kwargs )
            │          │           │         │
            必须        可选       0到多个   0到多个
          按顺序      按顺序      位置参数   关键字参数
```

***

#### 参数速查表

| 类型    | 语法                | 函数内类型    | 示例             |
| ----- | ----------------- | -------- | -------------- |
| 普通参数  | `def f(x)`        | 直接使用     | `f(1)`         |
| 默认参数  | `def f(x=0)`      | 直接使用     | `f()` 或 `f(5)` |
| 关键字参数 | 调用时 `f(x=1)`      | 直接使用     | `f(x=1)`       |
| 可变位置  | `def f(*args)`    | 元组 tuple | `f(1,2,3)`     |
| 可变关键字 | `def f(**kwargs)` | 字典 dict  | `f(a=1,b=2)`   |

***

### 🔭 作用域（8 分钟）

#### 什么是作用域？

作用域决定了**变量在哪里可以被访问**。Python 中变量不是在所有地方都能用的：

```python
def my_func():
    x = 10          # 局部变量，只在函数内部存在
    print(x)        # ✅ 函数内部可以访问

my_func()           # 输出：10
print(x)            # ❌ NameError：函数外部无法访问 x
```

***

#### 局部变量 vs 全局变量

```python
message = "我是全局变量"     # 全局变量：在函数外定义

def show():
    tip = "我是局部变量"     # 局部变量：在函数内定义
    print(message)           # ✅ 函数内可以读取全局变量
    print(tip)               # ✅ 函数内可以读取局部变量

show()
print(message)               # ✅ 全局变量在函数外可访问
print(tip)                   # ❌ NameError：局部变量在外部不可访问
```

***

#### 局部变量遮蔽全局变量

函数内定义了和全局变量同名的变量时，函数内使用的是局部版本：

```python
name = "全局小明"

def greet():
    name = "局部小红"       # 新建了一个局部变量，与全局同名
    print(f"函数内：{name}")  # 输出：函数内：局部小红

greet()
print(f"函数外：{name}")      # 输出：函数外：全局小明（全局变量未被改变）
```

**内存示意：**

```
全局空间             函数内部空间
─────────────        ─────────────────
name = "全局小明"    name = "局部小红"  ← 两个独立的 name
                     （函数结束后消失）
```

***

#### global 关键字：在函数内修改全局变量

如果确实需要在函数内修改全局变量，用 `global` 声明：

```python
count = 0                # 全局变量

def increment():
    global count         # 声明：我要修改的是全局的 count
    count += 1
    print(f"函数内 count = {count}")

increment()              # 输出：函数内 count = 1
increment()              # 输出：函数内 count = 2
print(f"函数外 count = {count}")  # 输出：函数外 count = 2
```

```python
# ❌ 不用 global，直接修改全局变量会报错
count = 0

def bad_increment():
    count += 1           # UnboundLocalError！
    # Python 认为 count 是局部变量，但使用前未赋值

bad_increment()
```

> ⚠️ **尽量避免使用 global！** 全局变量会让代码难以追踪和调试。更好的做法是通过**参数传入、返回值传出**：
>
> ```python
> # ✅ 推荐写法：通过参数和返回值传递数据
> def increment(count):
>     return count + 1
>
> count = 0
> count = increment(count)   # count = 1
> count = increment(count)   # count = 2
> ```

***

#### LEGB 规则：Python 查找变量的顺序

当 Python 遇到一个变量名，会按照 **L → E → G → B** 的顺序依次查找：

```
L  Local（局部）      → 当前函数内部
E  Enclosing（闭包）  → 外层函数（嵌套函数时）
G  Global（全局）     → 模块顶层
B  Built-in（内置）   → Python 内置名称（print、len 等）
```

```python
x = "全局 x"                    # G：全局

def outer():
    x = "外层函数 x"             # E：闭包层

    def inner():
        x = "内层函数 x"         # L：局部
        print(x)                 # 找到 L，输出：内层函数 x

    inner()
    print(x)                     # 找到 E，输出：外层函数 x

outer()
print(x)                         # 找到 G，输出：全局 x
```

**查找顺序图示：**

```
inner() 中访问变量 x
    │
    ▼
L: inner 内部有 x？ → 有 → 使用 "内层函数 x"，停止查找
    │（若无，继续向上）
    ▼
E: outer 内部有 x？ → 有 → 使用 "外层函数 x"，停止查找
    │（若无，继续向上）
    ▼
G: 全局有 x？       → 有 → 使用 "全局 x"，停止查找
    │（若无，继续向上）
    ▼
B: 内置中有 x？     → 无 → NameError
```

***

#### nonlocal 关键字（了解）

在嵌套函数中修改外层函数的变量，用 `nonlocal`：

```python
def make_counter():
    count = 0               # 外层函数变量

    def increment():
        nonlocal count      # 声明修改的是外层的 count
        count += 1
        return count

    return increment

counter = make_counter()
print(counter())   # 输出：1
print(counter())   # 输出：2
print(counter())   # 输出：3
```

> 💡 这个模式叫**闭包（Closure）**，是函数式编程的基础概念，第七课会深入讲解。

***

### 🏁 综合练习（7 分钟）

#### 练习：函数化成绩管理系统

将第四课的成绩管理程序重构为函数版本，感受函数带来的代码组织优势：

```python
# ============================================
# 函数化成绩管理系统
# ============================================

# ---------- 工具函数 ----------
def get_grade(score):
    """根据分数返回等级标签"""
    if score >= 90:   return "优秀 ⭐"
    elif score >= 75: return "良好 ✅"
    elif score >= 60: return "及格 📝"
    else:             return "不及格 ❌"


def calc_stats(scores_dict):
    """计算并返回统计数据"""
    values = list(scores_dict.values())
    return {
        "count":   len(values),
        "average": sum(values) / len(values),
        "highest": max(values),
        "lowest":  min(values),
    }


def print_divider(char="=", width=36):
    """打印分隔线"""
    print(char * width)


def print_header(title, width=36):
    """打印标题区域"""
    print_divider("=", width)
    print(f"{title:^{width - 2}}")
    print_divider("=", width)


def print_stats(stats):
    """打印统计信息"""
    print(f"  参考人数：{stats['count']} 人")
    print(f"  平均分：  {stats['average']:.1f} 分")
    print(f"  最高分：  {stats['highest']} 分")
    print(f"  最低分：  {stats['lowest']} 分")
    print_divider("-")


def print_ranking(scores_dict):
    """按成绩降序打印排名"""
    sorted_items = sorted(scores_dict.items(), key=lambda x: x[1], reverse=True)
    print(f"  {'排名':<4} {'姓名':<6} {'成绩':>5}   等级")
    print_divider("-")
    for rank, (name, score) in enumerate(sorted_items, start=1):
        grade = get_grade(score)
        print(f"  {rank:<4} {name:<6} {score:>5} 分  {grade}")


def print_pass_fail(scores_dict):
    """打印及格/不及格分组"""
    pass_set = {n for n, s in scores_dict.items() if s >= 60}
    fail_set  = set(scores_dict.keys()) - pass_set
    print_divider("-")
    print(f"  ✅ 及格（{len(pass_set)} 人）：{', '.join(sorted(pass_set))}")
    if fail_set:
        print(f"  ❌ 不及格（{len(fail_set)} 人）：{', '.join(sorted(fail_set))}")
    print_divider("=")


# ---------- 主程序 ----------
def main():
    scores = {
        "小明": 92, "小红": 85, "小李": 73,
        "小张": 95, "小王": 68, "小赵": 55,
    }

    stats = calc_stats(scores)

    print_header("📊 成绩统计报告")
    print_stats(stats)
    print_ranking(scores)
    print_pass_fail(scores)


main()    # 调用主函数，启动程序
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
  排名   姓名     成绩   等级
------------------------------------
  1    小张       95 分  优秀 ⭐
  2    小明       92 分  优秀 ⭐
  3    小红       85 分  良好 ✅
  4    小李       73 分  及格 📝
  5    小王       68 分  及格 📝
  6    小赵       55 分  不及格 ❌
------------------------------------
  ✅ 及格（5 人）：小张、小明、小红、小李、小王
  ❌ 不及格（1 人）：小赵
====================================
```

> 💡 **对比第四课的版本：** 代码总行数相近，但每个函数职责单一，任何一处逻辑出了问题，只需修改对应的函数，不需要在一整段代码里翻找。这就是**函数化编程**的核心价值。

***

### 📝 本课知识总结

```
函数基础
├── 定义：def 函数名(参数): 函数体
├── 调用：函数名(实参)
├── 参数：将数据传入函数
├── return：将结果传出函数，并结束函数执行
├── 多返回值：return a, b  →  本质是元组
└── Docstring：三引号写在函数体第一行

参数类型
├── 普通参数：def f(x, y)      → 调用时按顺序传入
├── 默认参数：def f(x, y=0)    → 调用时可省略
├── 关键字参数：f(y=1, x=2)    → 调用时可不按顺序
├── 可变位置：def f(*args)      → 函数内得到元组
└── 可变关键字：def f(**kwargs) → 函数内得到字典

作用域 LEGB
├── L Local    → 当前函数内部（优先）
├── E Enclosing → 外层函数（嵌套时）
├── G Global   → 模块顶层
└── B Built-in → print、len 等内置名称

关键字
├── global   → 在函数内声明并修改全局变量
└── nonlocal → 在嵌套函数内修改外层函数变量
```

***

### 🎯 课后作业

**练习 1（基础）：** 写一个 `calc(a, b, op="+")` 函数：

* 支持 `+` `-` `*` `/` 四种运算，通过 `op` 参数指定
* 除法时若除数为 0，返回字符串 `"错误：除数不能为 0"`
* 调用示例：`calc(10, 3, "*")` → `30`，`calc(10, 0, "/")` → 错误提示

**练习 2（进阶）：** 写一个 `stats(*numbers)` 函数：

* 接受任意数量的数字参数
* 返回一个字典，包含：`count`（个数）、`total`（总和）、`average`（均值）、`maximum`（最大值）、`minimum`（最小值）
* 参数为空时返回 `None` 并打印提示
* 调用示例：`stats(3, 1, 4, 1, 5, 9, 2, 6)`

**练习 3（挑战）：** 写一个简易"函数式计算器"：

* 定义 `add`、`subtract`、`multiply`、`divide` 四个函数
* 再定义一个 `calculate(func, *args)` 函数，接收一个函数和任意参数，调用并返回结果
* 最后用 `while True` 主循环，让用户选择运算类型和输入数字，调用 `calculate` 完成计算
* 提示：函数也可以作为参数传递给另一个函数！

***

### 🔜 下节课预告

**第七课：文件读写与异常处理**

```python
# 你将学会把数据保存到文件，以及优雅地处理程序错误：

# 写入文件
with open("scores.txt", "w", encoding="utf-8") as f:
    f.write("小明,95\n小红,87\n小李,72\n")

# 读取文件
with open("scores.txt", "r", encoding="utf-8") as f:
    for line in f:
        name, score = line.strip().split(",")
        print(f"{name}：{score} 分")

# 异常处理：程序出错时不崩溃
try:
    result = int(input("输入一个数字："))
    print(f"结果：{100 / result}")
except ValueError:
    print("❌ 输入的不是数字！")
except ZeroDivisionError:
    print("❌ 不能除以零！")
```

***

> 💬 **有疑问？** 欢迎在评论区留言或加入学习群讨论。
>
> ⭐ 觉得有帮助的话，别忘了分享给朋友！

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 6 课 / 共 10 课 Power by Andrew Liu_
