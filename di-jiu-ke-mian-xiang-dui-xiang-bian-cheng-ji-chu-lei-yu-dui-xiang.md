# 第九课：面向对象编程基础（类与对象）

### 第九课：面向对象编程基础（类与对象）

> 🕐 **课程时长：40 分钟** 🎯 **前置知识：已完成第 01～08 课** 🛠️ **工具准备：Python 3.x、终端（Terminal / 命令提示符）、任意代码编辑器**

***

### 课程导览

| 模块       | 内容                                      | 时间    |
| -------- | --------------------------------------- | ----- |
| 🔥 热身回顾  | 从函数到类：代码组织方式的进化                         | 3 分钟  |
| 🏗️ 类与对象 | 概念、"蓝图"比喻、class 定义语法                    | 8 分钟  |
| 🔩 属性与方法 | `__init__`、self、实例方法、类属性                | 10 分钟 |
| ✨ 魔法方法   | `__str__`、`__repr__`、`__len__`、`__eq__` | 7 分钟  |
| 🔒 封装与继承 | 私有属性约定、子类、`super()`                     | 8 分钟  |
| 🏁 综合练习  | OOP 版学生成绩管理系统                           | 4 分钟  |

***

### 🔥 热身回顾（3 分钟）

前八课我们经历了代码组织能力的三次进化：

```
第 6 课：函数          第 8 课：模块             第 9 课：类（面向对象）
──────────────         ─────────────────         ─────────────────────────
def get_grade():  →    grade_utils.py        →   class Student:
def calc_avg():        ├── get_grade()            ├── __init__(self, name)
def print_report()     ├── calc_avg()             ├── add_score(self, s)
                       └── print_report()         ├── average(self)
                                                  └── __str__(self)
```

**函数** 解决重复代码；**模块** 解决文件组织；**类** 解决"数据 + 操作捆绑"——把描述同一事物的属性（数据）和行为（函数）打包在一起。

第八课末尾已经预告了 `Student` 类的雏形，今天我们从头理解它，并扩展为完整的面向对象系统。

***

### 🏗️ 类与对象（8 分钟）

#### 核心比喻：蓝图与建筑

```
类（Class）  ≈ 建筑蓝图       对象（Object）≈ 根据蓝图盖的房子
──────────────────────────    ──────────────────────────────────
Student（类定义）             s1 = Student("小明", 18)
                              s2 = Student("小红", 17)
                              → 每个对象是独立个体，
                                拥有蓝图规定的同一套结构，
                                但数据各自独立
```

```python
# 最简单的类定义
class Dog:
    pass   # 先留空

d1 = Dog()   # 创建对象（实例化）
d2 = Dog()   # 又一个对象

print(type(d1))    # <class '__main__.Dog'>
print(d1 is d2)    # False，两个独立对象
```

***

#### 为什么需要类？

**没有类——用字典代替（数据和行为分离）：**

```python
student1 = {"name": "小明", "age": 18, "scores": []}
student2 = {"name": "小红", "age": 17, "scores": []}

def add_score(student, score):
    student["scores"].append(score)

def average(student):
    scores = student["scores"]
    return sum(scores) / len(scores) if scores else 0

add_score(student1, 92)
print(average(student1))  # 操作和数据分离，容易出错，难以扩展
```

**有了类——数据与行为封装在一起：**

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.scores = []

    def add_score(self, score):
        self.scores.append(score)

    def average(self):
        return sum(self.scores) / len(self.scores) if self.scores else 0

s = Student("小明", 18)
s.add_score(92)
print(s.average())   # 清晰自然，操作跟着对象走
```

***

### 🔩 属性与方法（10 分钟）

#### `__init__`：构造方法

`__init__` 是类的"初始化方法"，在创建对象时**自动调用**，负责设置对象的初始状态：

```python
class Student:
    def __init__(self, name, age, major="未填写"):
        """
        self   → 代表"当前这个对象"（Python 自动传入，不需要手动传）
        name   → 必须提供
        age    → 必须提供
        major  → 可选，默认"未填写"
        """
        self.name   = name    # 实例属性：每个对象各自独立
        self.age    = age
        self.major  = major
        self.scores = []      # 初始化为空列表

s1 = Student("小明", 18, "计算机")
s2 = Student("小红", 17)          # major 使用默认值

print(s1.name, s1.major)    # 小明  计算机
print(s2.name, s2.major)    # 小红  未填写
```

***

#### self 是什么？

`self` 代表**调用该方法的对象本身**。Python 自动把调用对象传给 `self`，不需要手动传：

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        import math
        return math.pi * self.radius ** 2

c1 = Circle(5)
c2 = Circle(10)

# c1.area() 等价于 Circle.area(c1)
print(c1.area())    # 78.54（c1 的半径 5）
print(c2.area())    # 314.16（c2 的半径 10）
```

```
c1.area() 调用时：self → c1，self.radius → 5
c2.area() 调用时：self → c2，self.radius → 10
```

***

#### 实例属性 vs 类属性

```python
class Student:
    school = "Python 学院"    # 类属性：所有对象共享

    def __init__(self, name, score):
        self.name  = name    # 实例属性：每个对象独有
        self.score = score

s1 = Student("小明", 90)
s2 = Student("小红", 85)

print(Student.school)   # Python 学院（通过类名访问）
print(s1.school)        # Python 学院（对象访问，向上查找类属性）
print(s1.name)          # 小明
print(s2.name)          # 小红

# 修改类属性（通过类名，所有对象都变）
Student.school = "AI 编程学院"
print(s1.school)        # AI 编程学院
print(s2.school)        # AI 编程学院
```

> ⚠️ **常见误区：** 通过对象赋值给"类属性同名"的属性，实际是给该对象创建了一个新的实例属性，不影响其他对象：
>
> ```python
> s1.school = "专属学院"   # 只给 s1 新建实例属性
> print(s1.school)         # 专属学院（s1 的实例属性）
> print(s2.school)         # AI 编程学院（类属性不变）
> ```

***

#### 实例方法

方法是定义在类内部的函数，第一个参数固定为 `self`：

```python
class Student:
    school = "Python 学院"

    def __init__(self, name, age):
        self.name   = name
        self.age    = age
        self.scores = []

    def add_score(self, score):
        """添加一次成绩（含范围校验）"""
        if not (0 <= score <= 100):
            raise ValueError(f"成绩 {score} 超出 0～100 范围")
        self.scores.append(score)

    def average(self):
        """计算平均分，无成绩时返回 0"""
        return sum(self.scores) / len(self.scores) if self.scores else 0

    def grade(self):
        """根据平均分返回等级"""
        avg = self.average()
        if avg >= 90:   return "优秀"
        elif avg >= 75: return "良好"
        elif avg >= 60: return "及格"
        else:           return "待提高"

    def report(self):
        """打印成绩报告"""
        print(f"{'─' * 30}")
        print(f"  学生：{self.name}（{self.age} 岁）")
        print(f"  学校：{self.school}")
        print(f"  成绩：{self.scores}")
        print(f"  均分：{self.average():.1f}  等级：{self.grade()}")
        print(f"{'─' * 30}")


s = Student("小明", 18)
s.add_score(92)
s.add_score(85)
s.add_score(78)
s.report()
```

输出：

```
──────────────────────────────
  学生：小明（18 岁）
  学校：Python 学院
  成绩：[92, 85, 78]
  均分：85.0  等级：良好
──────────────────────────────
```

***

### ✨ 魔法方法（7 分钟）

魔法方法（Dunder Methods）以双下划线开头和结尾，Python 在特定操作时自动调用：

***

#### `__str__`：定义 print() 的显示内容

```python
class Student:
    def __init__(self, name, age):
        self.name   = name
        self.age    = age
        self.scores = []

    def __str__(self):
        avg = sum(self.scores) / len(self.scores) if self.scores else 0
        return f"Student(name={self.name}, age={self.age}, avg={avg:.1f})"

s = Student("小明", 18)
s.scores = [90, 85]
print(s)        # Student(name=小明, age=18, avg=87.5)
print(str(s))   # 同上
```

***

#### `__repr__`：开发调试用的字符串表示

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age  = age

    def __repr__(self):
        """在交互式解释器、repr()、容器中显示"""
        return f"Student(name='{self.name}', age={self.age})"

    def __str__(self):
        """友好显示，print() 优先使用"""
        return f"{self.name}（{self.age}岁）"

s = Student("小明", 18)
print(s)           # 小明（18岁）                  ← __str__
print(repr(s))     # Student(name='小明', age=18)  ← __repr__

# 在列表中显示时用 __repr__
students = [Student("小明", 18), Student("小红", 17)]
print(students)    # [Student(name='小明', age=18), Student(name='小红', age=17)]
```

***

#### `__len__` 和 `__eq__`：自定义运算符行为

```python
class Student:
    def __init__(self, name, score):
        self.name  = name
        self.score = score

    def __eq__(self, other):
        """obj1 == obj2：名字和分数都相同才视为相等"""
        if not isinstance(other, Student):
            return NotImplemented
        return self.name == other.name and self.score == other.score

    def __lt__(self, other):
        """obj1 < obj2：支持按分数排序"""
        return self.score < other.score

    def __str__(self):
        return f"{self.name}({self.score}分)"


class GradeBook:
    def __init__(self, class_name):
        self.class_name = class_name
        self._students  = []

    def add_student(self, student):
        self._students.append(student)

    def __len__(self):
        """len(gradebook) 返回学生数量"""
        return len(self._students)

    def __str__(self):
        return f"GradeBook(班级={self.class_name}, 学生数={len(self)})"


gb = GradeBook("高一(3)班")
gb.add_student(Student("小明", 92))
gb.add_student(Student("小红", 85))
print(len(gb))    # 2
print(gb)         # GradeBook(班级=高一(3)班, 学生数=2)

s1 = Student("小明", 92)
s2 = Student("小明", 92)
s3 = Student("小红", 85)
print(s1 == s2)   # True
print(s1 == s3)   # False

# 利用 __lt__ 排序
students = [Student("小明", 92), Student("小红", 85), Student("小亮", 78)]
print(sorted(students))              # [小亮(78分), 小红(85分), 小明(92分)]
print(sorted(students, reverse=True))  # [小明(92分), 小红(85分), 小亮(78分)]
```

**常用魔法方法速查：**

| 魔法方法                       | 触发时机                        |
| -------------------------- | --------------------------- |
| `__init__(self, ...)`      | 创建对象 `cls(...)`             |
| `__str__(self)`            | `print(obj)`、`str(obj)`     |
| `__repr__(self)`           | `repr(obj)`、交互式解释器          |
| `__len__(self)`            | `len(obj)`                  |
| `__eq__(self, other)`      | `obj1 == obj2`              |
| `__lt__(self, other)`      | `obj1 < obj2`，支持 `sorted()` |
| `__add__(self, other)`     | `obj1 + obj2`               |
| `__contains__(self, item)` | `item in obj`               |
| `__getitem__(self, key)`   | `obj[key]`                  |
| `__iter__(self)`           | `for x in obj`              |

***

### 🔒 封装与继承（8 分钟）

#### 封装：隐藏内部细节

封装把数据和操作包在一起，通过方法控制读写，防止外部直接破坏内部状态：

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner    = owner
        self._balance = balance    # 单下划线：约定"受保护"，外部不建议直接访问

    def deposit(self, amount):
        if amount <= 0:
            raise ValueError("存款金额必须大于 0")
        self._balance += amount
        print(f"✅ 存入 {amount} 元，余额：{self._balance} 元")

    def withdraw(self, amount):
        if amount <= 0:
            raise ValueError("取款金额必须大于 0")
        if amount > self._balance:
            raise ValueError(f"余额不足（当前：{self._balance} 元）")
        self._balance -= amount
        print(f"✅ 取出 {amount} 元，余额：{self._balance} 元")

    @property
    def balance(self):
        """只读属性：外部可以读余额，但不能直接改"""
        return self._balance

    def __str__(self):
        return f"账户[{self.owner}]，余额：{self._balance} 元"


acc = BankAccount("小明", 1000)
acc.deposit(500)          # ✅ 存入 500 元，余额：1500 元
acc.withdraw(200)         # ✅ 取出 200 元，余额：1300 元
print(acc.balance)        # 1300（通过 @property 读取）
# acc.balance = 99999    # ❌ AttributeError：只读属性
```

> 💡 **Python 封装约定：**
>
> ```
> self.name    → 公开属性，外部随意访问
> self._name   → "受保护"，约定不直接访问（靠自觉）
> self.__name  → Python 名称改写为 _ClassName__name，更难从外部直接访问
> ```

***

#### 继承：子类复用父类

子类获得父类的全部属性和方法，并可以新增或覆盖：

```python
class Animal:
    """父类（基类）"""
    def __init__(self, name, age):
        self.name = name
        self.age  = age

    def speak(self):
        return "..."

    def info(self):
        return f"{self.name}（{self.age}岁）说：{self.speak()}"


class Dog(Animal):          # Dog 继承 Animal
    def speak(self):        # 覆盖父类方法（方法重写）
        return "汪汪！"

    def fetch(self):        # 子类独有方法
        return f"{self.name} 去捡球了！"


class Cat(Animal):
    def __init__(self, name, age, indoor=True):
        super().__init__(name, age)    # super() 调用父类 __init__
        self.indoor = indoor           # 子类新增属性

    def speak(self):
        return "喵～"

    def info(self):                    # 覆盖并扩展父类 info
        base     = super().info()
        location = "室内猫" if self.indoor else "流浪猫"
        return f"{base}（{location}）"


dog = Dog("旺财", 3)
cat = Cat("咪咪", 2, indoor=True)

print(dog.info())    # 旺财（3岁）说：汪汪！
print(dog.fetch())   # 旺财 去捡球了！
print(cat.info())    # 咪咪（2岁）说：喵～（室内猫）

# isinstance 检查继承关系
print(isinstance(dog, Dog))      # True
print(isinstance(dog, Animal))   # True（Dog 是 Animal 的子类）
print(isinstance(dog, Cat))      # False
```

***

#### 继承在成绩系统中的应用

```python
class Person:
    """人员基类"""
    def __init__(self, name, age):
        self.name = name
        self.age  = age

    def __str__(self):
        return f"{self.name}（{self.age}岁）"


class Student(Person):
    def __init__(self, name, age, student_id):
        super().__init__(name, age)
        self.student_id = student_id
        self.scores     = {}    # {"语文": 90, "数学": 85}

    def add_score(self, subject, score):
        self.scores[subject] = score

    def average(self):
        return sum(self.scores.values()) / len(self.scores) if self.scores else 0

    def __str__(self):
        return (f"[学生] {super().__str__()} | "
                f"学号:{self.student_id} | 均分:{self.average():.1f}")


class Teacher(Person):
    def __init__(self, name, age, subject):
        super().__init__(name, age)
        self.subject  = subject
        self.students = []

    def assign_student(self, student):
        self.students.append(student)

    def class_average(self):
        if not self.students: return 0
        return sum(s.average() for s in self.students) / len(self.students)

    def __str__(self):
        return (f"[教师] {super().__str__()} | "
                f"科目:{self.subject} | 班级均分:{self.class_average():.1f}")


t  = Teacher("王老师", 35, "Python")
s1 = Student("小明", 18, "S001")
s2 = Student("小红", 17, "S002")

s1.add_score("Python", 92); s1.add_score("数学", 85)
s2.add_score("Python", 78); s2.add_score("数学", 91)
t.assign_student(s1); t.assign_student(s2)

print(s1)    # [学生] 小明（18岁） | 学号:S001 | 均分:88.5
print(s2)    # [学生] 小红（17岁） | 学号:S002 | 均分:84.5
print(t)     # [教师] 王老师（35岁） | 科目:Python | 班级均分:86.5
```

***

### 🏁 综合练习（4 分钟）

#### OOP 版学生管理系统

将前几课的成绩系统用面向对象彻底重构，结合第八课的 JSON 存储：

```python
# ============================================
# OOP 学生管理系统（OOP + JSON + datetime）
# ============================================
import json
import os
import datetime


class Student:
    """学生类：封装信息与成绩操作"""

    def __init__(self, name, student_id):
        self.name       = name
        self.student_id = student_id
        self.scores     = {}    # {"语文": 90, "数学": 85}
        self.created_at = datetime.date.today().isoformat()

    def add_score(self, subject, score):
        if not (0 <= score <= 100):
            raise ValueError(f"成绩须在 0～100，收到：{score}")
        self.scores[subject] = score

    def average(self):
        return sum(self.scores.values()) / len(self.scores) if self.scores else 0

    def grade(self):
        avg = self.average()
        if avg >= 90:   return "优秀 ⭐"
        elif avg >= 75: return "良好 ✅"
        elif avg >= 60: return "及格 📝"
        else:           return "待提高 ❌"

    def to_dict(self):
        """序列化为字典（用于 JSON 存储）"""
        return {
            "name":       self.name,
            "student_id": self.student_id,
            "scores":     self.scores,
            "created_at": self.created_at,
        }

    @classmethod
    def from_dict(cls, data):
        """从字典反序列化（从 JSON 读取）"""
        s = cls(data["name"], data["student_id"])
        s.scores     = data.get("scores", {})
        s.created_at = data.get("created_at", "")
        return s

    def __str__(self):
        scores_str = "  ".join(f"{k}:{v}" for k, v in self.scores.items())
        return (f"[{self.student_id}] {self.name:<6} "
                f"均分:{self.average():.1f}  {self.grade()}  {scores_str}")

    def __repr__(self):
        return f"Student(name='{self.name}', id='{self.student_id}')"


class GradeBook:
    """成绩册类：管理多名学生，支持 JSON 持久化"""

    DATA_FILE = "gradebook.json"

    def __init__(self, class_name):
        self.class_name = class_name
        self._students  = {}    # {student_id: Student}

    # ── CRUD ──────────────────────────────────────────────

    def add_student(self, name, student_id):
        if student_id in self._students:
            print(f"⚠️  学号 {student_id} 已存在")
            return
        self._students[student_id] = Student(name, student_id)
        print(f"✅ 已添加：{name}（{student_id}）")

    def add_score(self, student_id, subject, score):
        student = self._students.get(student_id)
        if not student:
            print(f"❌ 找不到学号 {student_id}")
            return
        try:
            student.add_score(subject, score)
            print(f"✅ 已录入：{student.name} - {subject} - {score} 分")
        except ValueError as e:
            print(f"❌ {e}")

    # ── 统计 ──────────────────────────────────────────────

    def class_average(self):
        students = list(self._students.values())
        if not students: return 0
        return sum(s.average() for s in students) / len(students)

    def ranking(self):
        return sorted(self._students.values(), key=lambda s: s.average(), reverse=True)

    def show_report(self):
        if not self._students:
            print("📭 暂无学生记录。")
            return
        ranked = self.ranking()
        print(f"\n{'═' * 55}")
        print(f"{'📊 ' + self.class_name + ' 成绩报告':^50}")
        print(f"  班级均分：{self.class_average():.1f} 分")
        print(f"{'═' * 55}")
        for i, s in enumerate(ranked, 1):
            bar = "█" * int(s.average() // 10)
            print(f"  {i:>2}. {s}  {bar}")
        print(f"{'═' * 55}\n")

    # ── 持久化 ────────────────────────────────────────────

    def save(self):
        data = {
            "class_name": self.class_name,
            "saved_at":   datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "students":   [s.to_dict() for s in self._students.values()],
        }
        with open(self.DATA_FILE, "w", encoding="utf-8") as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
        print(f"✅ 已保存 {len(self._students)} 条记录 → {self.DATA_FILE}")

    @classmethod
    def load(cls):
        if not os.path.exists(cls.DATA_FILE):
            name = input("班级名称：").strip() or "默认班级"
            return cls(name)
        with open(cls.DATA_FILE, "r", encoding="utf-8") as f:
            data = json.load(f)
        gb = cls(data["class_name"])
        for sd in data["students"]:
            s = Student.from_dict(sd)
            gb._students[s.student_id] = s
        print(f"📂 已加载 {len(gb._students)} 条记录（{data.get('saved_at', '未知')}）")
        return gb

    def __len__(self):
        return len(self._students)

    def __str__(self):
        return f"GradeBook(班级={self.class_name}, 学生数={len(self)})"


# ── 主程序 ──────────────────────────────────────────────

def main():
    gb = GradeBook.load()
    print(f"\n{gb}\n")

    menu = """
╔══════════════════════╗
║  1. 添加学生          ║
║  2. 录入成绩          ║
║  3. 查看报告          ║
║  4. 保存并退出        ║
╚══════════════════════╝"""

    while True:
        print(menu)
        choice = input("请选择：").strip()

        if choice == "1":
            name = input("  姓名：").strip()
            sid  = input("  学号：").strip()
            gb.add_student(name, sid)

        elif choice == "2":
            sid     = input("  学号：").strip()
            subject = input("  科目：").strip()
            try:
                score = int(input("  成绩：").strip())
                gb.add_score(sid, subject, score)
            except ValueError:
                print("❌ 成绩须为整数")

        elif choice == "3":
            gb.show_report()

        elif choice == "4":
            gb.save()
            print("👋 再见！")
            break

        else:
            print("⚠️  请输入 1～4")


if __name__ == "__main__":
    main()
```

***

### 📝 本课知识总结

```
面向对象编程（OOP）

类（Class）
├── class 类名:              → 定义类
├── __init__(self, ...)      → 构造方法，创建对象时自动调用
├── self                     → 当前对象的引用
├── 实例属性 self.xxx        → 每个对象独有
├── 类属性 ClassName.xxx     → 所有对象共享
└── 实例方法 def f(self):    → 定义行为

魔法方法
├── __str__    → print(obj) 显示的内容
├── __repr__   → repr(obj)、容器中的调试显示
├── __len__    → len(obj)
├── __eq__     → obj1 == obj2
└── __lt__     → obj1 < obj2，支持 sorted()

封装
├── self._attr   → 约定受保护，外部不建议直接访问
├── @property    → 只读属性装饰器
└── 通过方法控制读写，保护数据完整性

继承
├── class 子类(父类):        → 继承父类所有属性和方法
├── super().__init__(...)    → 调用父类构造方法
├── 方法重写                 → 子类覆盖父类同名方法
└── isinstance(obj, cls)    → 检查继承关系（含继承链）
```

***

### 🎯 课后作业

**练习 1（基础）：** 定义一个 `Rectangle`（矩形）类：

* 属性：`width`（宽）、`height`（高）
* 方法：`area()`（面积）、`perimeter()`（周长）、`is_square()`（是否为正方形）
* 魔法方法：`__str__` 显示 `矩形(宽=x, 高=y)`，`__eq__` 面积相等则视为相等

**练习 2（进阶）：** 扩展本课的 `Student` 和 `GradeBook`：

* `GradeBook.top_students(n=3)`：返回均分前 n 名的学生列表
* `GradeBook.subject_average(subject)`：计算某科目全班平均分
* `Student.highest_score()` 和 `Student.lowest_score()`：返回最高/最低分科目名和分数

**练习 3（挑战）：** 设计一个 `Library`（图书馆）系统：

* `Book` 类：书名、作者、ISBN、是否在馆
* `Library` 类：支持 `borrow(isbn)`、`return_book(isbn)`、`search(keyword)` 操作
* 数据用 JSON 持久化，支持关键词搜索书名或作者

***

### 🔜 下节课预告

**第十课：阶段项目——交互小工具或游戏**

```
整合前九课所学，独立完成一个完整的 Python 项目：

课程 01～09 技能树
├── 01  变量 & 数据类型        ✅
├── 02  条件判断               ✅
├── 03  循环                  ✅
├── 04  字符串操作             ✅
├── 05  列表 & 字典            ✅
├── 06  函数                  ✅
├── 07  文件 I/O              ✅
├── 08  模块 & pip            ✅
└── 09  面向对象（OOP）        ✅ ← 今天刚完成

```

***

> 💬 **有疑问？** 欢迎在评论区留言或加入学习群讨论。
>
> ⭐ 觉得有帮助的话，别忘了分享给朋友！

***

_最后更新：2026 年 | Python 基础（从零起飞）系列 第 9 课 / 共 10 课 Power by Andrew Liu_
