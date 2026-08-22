# 第 18 课｜数据仓库接入：SQL + Python 支撑产品数据查询



## 第二阶段：产品设计 × Python 实战

### 第 18 课｜数据仓库接入：SQL + Python 支撑产品数据查询

> 🕐 **课程时长：45 分钟** 🎯 **前置知识：已完成第 01～17 课** 🛠️ **工具准备：Python 3.x、pip install pandas sqlalchemy anthropic**

***

### 课程导览

| 模块      | 内容                                | 时间    |
| ------- | --------------------------------- | ----- |
| 🔥 热身回顾 | 第 16 课 CSV 数据源 → 本课升级数据库          | 3 分钟  |
| 🎯 场景拆解 | 产品数据查询痛点与 SQL + Python 解法         | 5 分钟  |
| 📚 核心知识 | SQLite、pandas.read\_sql、自然语言转 SQL | 10 分钟 |
| 💻 代码实战 | 完整数据查询模块开发                        | 20 分钟 |
| 🔧 封装产出 | 可复用查询封装模块                         | 4 分钟  |
| 🎓 课程小结 | 知识地图 & 下节课预告                      | 3 分钟  |

***

### 🔥 热身回顾（3 分钟）

第 16 课的数据看板用 CSV 作为数据源——每次更新数据都要手动上传文件。

现实中产品数据存储在**数据库**里：用户行为、订单记录、指标数据……

本课目标：

```
第 16 课：CSV 文件 → Pandas → 看板
第 18 课：数据库  → SQL → Pandas → 看板（自动更新）
             +
        自然语言 → AI 生成 SQL → 执行查询
```

***

### 🎯 场景拆解（5 分钟）

#### 真实痛点

```
产品经理想查：
  "上周新注册用户中，完成首次付费的转化率是多少？"
  "iOS 和 Android 用户的 7 日留存率有什么差异？"
  "哪些用户昨天活跃但今天没回来？"

传统方式：找数据分析师写 SQL → 等 2 小时 → 看结果
本课方式：自然语言输入 → AI 生成 SQL → 自动查询 → 结果 + 解读
```

#### 解决思路

```
SQLite 数据库（本地模拟生产环境）
          │
          ▼
自然语言查询需求
          │
          ▼
Claude API：根据表结构生成 SQL
          │
          ▼
pandas.read_sql() 执行查询
          │
          ▼
Pandas DataFrame → 图表 + 文字解读
```

***

### 📚 核心知识（10 分钟）

#### 1. SQLite + SQLAlchemy：本地数据库

```python
import sqlite3
import pandas as pd
from sqlalchemy import create_engine, text

# 方法 1：sqlite3 原生（轻量）
conn = sqlite3.connect("data.db")
df = pd.read_sql("SELECT * FROM users LIMIT 10", conn)
conn.close()

# 方法 2：SQLAlchemy（推荐，可无缝切换 PostgreSQL/MySQL）
engine = create_engine("sqlite:///data.db")
with engine.connect() as conn:
    df = pd.read_sql(text("SELECT * FROM users LIMIT 10"), conn)
```

#### 2. 表结构描述给 Claude

让 AI 生成正确的 SQL，关键是**准确描述表结构**：

```python
def get_schema_description(engine) -> str:
    """提取数据库表结构，生成文字描述"""
    schema_parts = []
    with engine.connect() as conn:
        # SQLite 获取所有表名
        tables = pd.read_sql(
            text("SELECT name FROM sqlite_master WHERE type='table'"), conn
        )
        for table_name in tables["name"]:
            cols = pd.read_sql(text(f"PRAGMA table_info({table_name})"), conn)
            col_descs = ", ".join([
                f"{row['name']} ({row['type']})"
                for _, row in cols.iterrows()
            ])
            schema_parts.append(f"表 {table_name}：{col_descs}")
    return "\n".join(schema_parts)
```

#### 3. 自然语言转 SQL（Text-to-SQL）

````python
SYSTEM_PROMPT = """你是一位数据工程师，擅长根据自然语言需求生成 SQL 查询。

规则：
1. 只输出可直接执行的 SQL，不加解释
2. 使用 SQLite 语法（日期函数用 strftime）
3. 日期范围使用相对表达（如 date('now', '-7 days')）
4. 结果限制在 1000 行以内（加 LIMIT）
5. 代码格式：```sql\n...\n```

数据库结构：
{schema}
"""

def text_to_sql(question: str, schema: str) -> str:
    client = anthropic.Anthropic()
    resp = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=500,
        system=SYSTEM_PROMPT.format(schema=schema),
        messages=[{"role": "user", "content": question}]
    )
    return resp.content[0].text
````

***

### 💻 代码实战（20 分钟）

#### 项目结构

```
db-query/
├── db_query.py           ← 核心查询模块（函数库）
├── setup_demo_db.py      ← 创建演示数据库
├── query_demo.py         ← 命令行查询 Demo
└── data.db               ← SQLite 数据库（自动生成）
```

#### 第一步：创建演示数据库

```python
# setup_demo_db.py
import sqlite3
import random
from datetime import datetime, timedelta

def create_demo_database(db_path: str = "data.db"):
    """创建包含真实感数据的演示数据库"""
    conn = sqlite3.connect(db_path)
    c = conn.cursor()
    
    # 建表
    c.executescript("""
    DROP TABLE IF EXISTS users;
    DROP TABLE IF EXISTS events;
    DROP TABLE IF EXISTS orders;
    
    CREATE TABLE users (
        user_id     INTEGER PRIMARY KEY,
        platform    TEXT,        -- iOS / Android / Web
        country     TEXT,
        register_date TEXT,
        is_premium  INTEGER      -- 0/1
    );
    
    CREATE TABLE events (
        event_id    INTEGER PRIMARY KEY,
        user_id     INTEGER,
        event_name  TEXT,        -- login / view_item / add_cart / purchase
        event_date  TEXT,
        FOREIGN KEY (user_id) REFERENCES users(user_id)
    );
    
    CREATE TABLE orders (
        order_id    INTEGER PRIMARY KEY,
        user_id     INTEGER,
        amount      REAL,
        order_date  TEXT,
        status      TEXT,        -- paid / refunded / pending
        FOREIGN KEY (user_id) REFERENCES users(user_id)
    );
    """)
    
    # 插入用户数据（500 个用户）
    platforms = ["iOS", "Android", "Web"]
    countries = ["CN", "US", "JP", "KR", "SG"]
    base_date = datetime.now() - timedelta(days=90)
    
    users = []
    for i in range(1, 501):
        reg_date = base_date + timedelta(days=random.randint(0, 80))
        users.append((
            i,
            random.choice(platforms),
            random.choice(countries),
            reg_date.strftime("%Y-%m-%d"),
            1 if random.random() < 0.15 else 0
        ))
    c.executemany("INSERT INTO users VALUES (?,?,?,?,?)", users)
    
    # 插入事件数据
    event_names = ["login", "view_item", "add_cart", "purchase"]
    events = []
    eid = 1
    for user_id in range(1, 501):
        # 每个用户有 5-30 次事件
        n_events = random.randint(5, 30)
        for _ in range(n_events):
            evt_date = base_date + timedelta(days=random.randint(0, 89))
            events.append((eid, user_id, random.choice(event_names),
                           evt_date.strftime("%Y-%m-%d")))
            eid += 1
    c.executemany("INSERT INTO events VALUES (?,?,?,?)", events)
    
    # 插入订单数据
    orders = []
    oid = 1
    for user_id in range(1, 501):
        if random.random() < 0.3:  # 30% 用户有订单
            n_orders = random.randint(1, 5)
            for _ in range(n_orders):
                ord_date = base_date + timedelta(days=random.randint(10, 89))
                orders.append((
                    oid, user_id,
                    round(random.uniform(9.9, 199.9), 2),
                    ord_date.strftime("%Y-%m-%d"),
                    random.choice(["paid", "paid", "paid", "refunded", "pending"])
                ))
                oid += 1
    c.executemany("INSERT INTO orders VALUES (?,?,?,?,?)", orders)
    
    conn.commit()
    conn.close()
    print(f"✅ 演示数据库已创建：{db_path}")
    print(f"   - users: {len(users)} 条")
    print(f"   - events: {len(events)} 条")
    print(f"   - orders: {len(orders)} 条")

if __name__ == "__main__":
    create_demo_database()
```

#### 核心模块：db\_query.py

````python
#!/usr/bin/env python3
"""
数据查询封装模块 | db_query.py
提供：
  - DataQueryEngine：数据库连接 + 查询
  - text_to_sql()：自然语言转 SQL
  - query_and_explain()：查询 + AI 解读
依赖：pip install pandas sqlalchemy anthropic
"""
import re
import pandas as pd
import anthropic
from sqlalchemy import create_engine, text, inspect


SCHEMA_SYSTEM = """你是一位数据工程师，擅长根据自然语言需求生成 SQLite SQL 查询。

规则：
1. 只输出可直接执行的 SQL，不加解释
2. 使用 SQLite 语法：日期用 strftime，字符串连接用 ||
3. 相对日期：date('now','-7 days')、date('now','-1 month')
4. 结果最多 1000 行（加 LIMIT 1000）
5. 格式：只输出 SQL 语句，不要代码块标记

数据库结构：
{schema}"""

EXPLAIN_SYSTEM = """你是一位产品数据分析师。
根据 SQL 查询结果，用 2-4 句话给出产品视角的解读：
- 数字说明了什么？
- 和正常水平相比如何？
- 产品可以采取什么行动？
语言简洁，面向产品经理。"""


class DataQueryEngine:
    """数据库查询引擎"""

    def __init__(self, db_url: str = "sqlite:///data.db"):
        self.engine = create_engine(db_url)
        self._schema = None

    @property
    def schema(self) -> str:
        if self._schema is None:
            self._schema = self._get_schema()
        return self._schema

    def _get_schema(self) -> str:
        """获取数据库表结构描述"""
        inspector = inspect(self.engine)
        parts = []
        for table_name in inspector.get_table_names():
            cols = inspector.get_columns(table_name)
            col_descs = ", ".join([f"{c['name']}({str(c['type'])})" for c in cols])
            parts.append(f"表 {table_name}：{col_descs}")
        return "\n".join(parts)

    def run_sql(self, sql: str) -> pd.DataFrame:
        """执行 SQL，返回 DataFrame"""
        with self.engine.connect() as conn:
            return pd.read_sql(text(sql), conn)

    def safe_run_sql(self, sql: str) -> tuple[pd.DataFrame | None, str]:
        """安全执行 SQL，返回 (DataFrame, error_msg)"""
        # 安全检查：只允许 SELECT
        sql_upper = sql.strip().upper()
        if not sql_upper.startswith("SELECT") and not sql_upper.startswith("WITH"):
            return None, "只允许 SELECT 查询"
        try:
            df = self.run_sql(sql)
            return df, ""
        except Exception as e:
            return None, str(e)


def text_to_sql(question: str, schema: str) -> str:
    """自然语言 → SQL"""
    client = anthropic.Anthropic()
    resp = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=600,
        system=SCHEMA_SYSTEM.format(schema=schema),
        messages=[{"role": "user", "content": question}]
    )
    sql = resp.content[0].text.strip()
    # 移除可能的代码块标记
    sql = re.sub(r"```sql\n?", "", sql)
    sql = re.sub(r"```\n?", "", sql)
    return sql.strip()


def explain_result(question: str, sql: str, df: pd.DataFrame) -> str:
    """对查询结果生成 AI 解读"""
    client = anthropic.Anthropic()
    context = (
        f"查询问题：{question}\n"
        f"执行的 SQL：{sql}\n"
        f"结果行数：{len(df)}\n"
        f"结果预览：\n{df.head(10).to_string(index=False)}"
    )
    resp = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=300,
        system=EXPLAIN_SYSTEM,
        messages=[{"role": "user", "content": context}]
    )
    return resp.content[0].text


def query_and_explain(question: str, engine: DataQueryEngine) -> dict:
    """完整流程：自然语言 → SQL → 查询 → 解读"""
    print(f"\n🔍 问题：{question}")
    
    # Step 1: 生成 SQL
    sql = text_to_sql(question, engine.schema)
    print(f"📝 生成 SQL：\n{sql}")
    
    # Step 2: 执行查询
    df, err = engine.safe_run_sql(sql)
    if err:
        return {"question": question, "sql": sql, "error": err}
    
    print(f"📊 查询结果：{len(df)} 行")
    
    # Step 3: AI 解读
    explanation = explain_result(question, sql, df)
    
    return {
        "question": question,
        "sql": sql,
        "data": df,
        "explanation": explanation,
    }
````

#### 命令行 Demo：query\_demo.py

```python
#!/usr/bin/env python3
"""
自然语言查询 Demo | query_demo.py
用法：python query_demo.py "上周新注册用户数"
"""
import sys
from db_query import DataQueryEngine, query_and_explain


def main():
    engine = DataQueryEngine()
    
    if len(sys.argv) > 1:
        question = " ".join(sys.argv[1:])
        result = query_and_explain(question, engine)
        
        if "error" in result:
            print(f"❌ 错误：{result['error']}")
        else:
            print(f"\n✅ AI 解读：\n{result['explanation']}")
            print(f"\n数据预览：\n{result['data'].head(10).to_string()}")
    else:
        # 交互模式
        print("📊 数据查询助手（输入 quit 退出）")
        print(f"数据库结构：\n{engine.schema}\n")
        
        while True:
            question = input("\n请输入查询需求：").strip()
            if question.lower() in ("quit", "exit", "q"):
                break
            if not question:
                continue
            
            result = query_and_explain(question, engine)
            if "error" in result:
                print(f"❌ {result['error']}")
            else:
                print(f"\n🤖 解读：{result['explanation']}")
                print(f"\n{result['data'].head(10).to_string()}")


if __name__ == "__main__":
    main()
```

#### 运行方式

```bash
# 第一步：创建演示数据库
python setup_demo_db.py

# 第二步：查询（命令行模式）
python query_demo.py "iOS 和 Android 用户各有多少？"
python query_demo.py "最近 7 天每天的新注册用户数"
python query_demo.py "付费用户的平均订单金额"

# 交互模式
python query_demo.py
```

***

### 🔧 封装产出（4 分钟）

```
db-query/
├── db_query.py           ✅ 可复用查询模块（函数库）
│   ├── DataQueryEngine   数据库连接 + 查询
│   ├── text_to_sql()     自然语言转 SQL
│   └── query_and_explain() 全流程封装
├── setup_demo_db.py      ✅ 演示数据生成
└── query_demo.py         ✅ 命令行/交互查询 Demo
```

第 20 课的"产品助理工具箱"将直接 `from db_query import DataQueryEngine` 复用本课模块。

***

### 🎓 课程小结（3 分钟）

```
本课学到的技能

SQLAlchemy
├── create_engine("sqlite:///data.db")
├── inspect(engine).get_table_names()
├── pd.read_sql(text(sql), conn)
└── 只允许 SELECT 的安全检查

Text-to-SQL
├── 表结构描述给 Claude
├── 提取 SQL（移除代码块标记）
└── safe_run_sql() 错误处理

工程设计
├── DataQueryEngine 类封装数据库操作
├── schema 缓存（@property + 懒加载）
└── 返回 dict 聚合所有结果
```

***

### 🎯 课后作业

**练习 1（基础）**：在 `query_demo.py` 中增加查询历史功能，把每次查询的问题、SQL、结果行数保存到 `query_history.json`。

**练习 2（进阶）**：把 `DataQueryEngine` 接入第 16 课的 Streamlit 看板，让看板支持自然语言查询。

**练习 3（挑战）**：支持切换数据库——通过环境变量 `DATABASE_URL` 支持 PostgreSQL，让同一份代码可以连接云端数据库（如 Supabase 免费版）。

***

_第二阶段：产品设计 × Python 实战 · 第 18 课 / 共 10 课_
