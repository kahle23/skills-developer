# 数据结构

Python 内置数据结构的选择和操作是日常开发中最频繁的部分。

---

# 一、数据结构选择指南

| 需求 | 首选 | 说明 |
|------|------|------|
| 有序可重复 | `list` | 随机访问 O(1)，插入/删除尾部 O(1) |
| 键值映射 | `dict` | 查找/插入 O(1)，3.7+ 保持插入顺序 |
| 去重 | `set` | 查找/插入 O(1)，无序不可重复 |
| 不可变去重 | `frozenset` | 可作为字典 key |
| 不可变序列 | `tuple` | 内存占用小于 list |
| 有序不可重复 | `dict.fromkeys()` | 利用 dict 保持插入顺序去重 |
| 计数 | `collections.Counter` | 统计元素出现次数 |
| 双端队列 | `collections.deque` | 队列两端 O(1) 操作 |
| 命名元组 | `collections.namedtuple` / `typing.NamedTuple` | 轻量级不可变对象 |

```python
from collections import Counter, deque, defaultdict

# Counter — 计数
word_counts = Counter(text.split())
top3 = word_counts.most_common(3)

# deque — 高效双端队列
queue = deque([1, 2, 3])
queue.appendleft(0)   # 左侧插入 O(1)
queue.popleft()       # 左侧弹出 O(1)

# defaultdict — 默认值字典
groups = defaultdict(list)
for user in users:
    groups[user.dept_id].append(user)
```

---

# 二、列表操作

## 2.1 创建与初始化

```python
# 基本创建
nums = [1, 2, 3]
names = ["alice", "bob"]

# 重复列表
zeros = [0] * 10

# 列表乘法的坑 — 嵌套列表不要用乘法
# BAD — 三个引用指向同一个列表
matrix = [[0] * 3] * 3
matrix[0][0] = 1  # 所有行都变了！

# GOOD
matrix = [[0] * 3 for _ in range(3)]
```

## 2.2 常用操作

```python
# 条件删除
users = [u for u in users if u.age >= 18]   # 推导式（创建新列表）
# 或原地删除
users[:] = [u for u in users if u.age >= 18]

# 排序
users.sort(key=lambda u: u.name)                          # 原地排序
users.sort(key=lambda u: (u.age, u.name))                 # 多字段
users.sort(key=lambda u: u.age, reverse=True)             # 倒序
sorted_users = sorted(users, key=lambda u: u.create_time)  # 返回新列表

# 查找
first_match = next((u for u in users if u.is_active), None)  # 找不到返回 None
```

## 2.3 切片

```python
nums = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

nums[2:5]      # [2, 3, 4]       起始到结束（不含）
nums[:5]       # [0, 1, 2, 3, 4] 从头
nums[5:]       # [5, 6, 7, 8, 9] 到尾
nums[-3:]      # [7, 8, 9]       最后3个
nums[::2]      # [0, 2, 4, 6, 8] 步长2
nums[::-1]     # [9, 8, ..., 0]  反转

# 切片赋值（原地替换）
nums[1:3] = [10, 20, 30]
```

> **注意**：切片是浅拷贝，嵌套结构修改会互相影响。

---

# 三、字典操作

## 3.1 常用模式

```python
# 安全获取
name = user.get("name", "未知")
age = user.get("age") or 0       # None/0 兜底

# 设置默认值
user.setdefault("tags", []).append("vip")

# 合并字典（3.9+）
merged = {**defaults, **overrides}
merged = defaults | overrides     # 3.9+ 运算符

# 遍历
for key, value in user.items():
    ...
for key in user:                  # 等价于 user.keys()
    ...
```

## 3.2 高级模式

```python
from collections import defaultdict

# 分组
groups = defaultdict(list)
for user in users:
    groups[user.dept_id].append(user)
# 结果: {1: [u1, u2], 2: [u3]}

# 计数
counts = defaultdict(int)
for word in words:
    counts[word] += 1

# 字典推导式
user_map = {u.id: u for u in users}
name_map = {u.id: u.name for u in users}

# 过滤
filtered = {k: v for k, v in config.items() if v is not None}

# 反转 key/value
reversed_map = {v: k for k, v in original.items()}
```

## 3.3 3.7+ 保持插入顺序

```python
# 有序去重（保持首次出现顺序）
unique_names = list(dict.fromkeys(names))

# 有序去重并保留最后出现顺序
unique_names = list(dict.fromkeys(reversed(names)))
```

---

# 四、集合操作

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# 集合运算
intersection = a & b      # {3, 4}     交集
union = a | b             # {1,2,3,4,5,6} 并集
difference = a - b        # {1, 2}     差集
symmetric = a ^ b         # {1, 2, 5, 6} 对称差集

# 子集/超集判断
{1, 2}.issubset(a)        # True
a.issuperset({1, 2})      # True
```

> **性能提示**：判断元素是否存在的场景，`set` 的 `in` 操作是 O(1)，远快于 `list` 的 O(n)。详见 `06-performance.md`。

---

# 五、推导式

## 5.1 列表推导式

```python
# 基本
squares = [x * x for x in range(10)]

# 带条件
evens = [x for x in nums if x % 2 == 0]

# 带变换
names = [u.name.upper() for u in users if u.is_active]

# 嵌套（谨慎使用，超过两层考虑拆分）
flat = [n for row in matrix for n in row]
```

## 5.2 字典推导式

```python
user_map = {u.id: u.name for u in users}
squares = {x: x * x for x in range(5)}
```

## 5.3 集合推导式

```python
unique_tags = {tag for article in articles for tag in article.tags}
```

## 5.4 何时用推导式

| 场景 | 推荐 |
|------|------|
| 简单映射/过滤（一行能看懂） | 推导式 |
| 多次循环或复杂条件 | 普通 for 循环 |
| 有副作用（修改外部状态） | 普通 for 循环 |
| 数据量极大 | 生成器表达式 `(...)` |

```python
# GOOD — 简单明了
names = [u.name for u in users if u.is_active]

# BAD — 太复杂，可读性差
result = [transform(x) for row in matrix for x in row if x > 0 and validate(x)]

# GOOD — 拆成普通循环
result = []
for row in matrix:
    for x in row:
        if x > 0 and validate(x):
            result.append(transform(x))
```

---

# 六、解包与拼接

```python
# 星号解包
first, *rest = [1, 2, 3, 4]      # first=1, rest=[2,3,4]
first, *middle, last = [1, 2, 3, 4]  # first=1, middle=[2,3], last=4

# 字典解包
def create_user(name, age, **kwargs):
    ...
config = {"name": "张三", "age": 18, "email": "..."}
create_user(**config)

# 多变量交换（不需要临时变量）
a, b = b, a

# 多返回值
def get_user_info(uid):
    return name, age, email  # 返回 tuple

name, age, email = get_user_info(uid)
```

---

# 七、枚举

```python
from enum import Enum

class OrderStatus(Enum):
    PENDING = "pending"
    PAID = "paid"
    SHIPPED = "shipped"
    COMPLETED = "completed"
    CANCELLED = "cancelled"

# 使用
status = OrderStatus.PAID
print(status.value)        # "paid"
print(status.name)         # "PAID"

# 值转枚举
status = OrderStatus("paid")

# IntEnum — 支持比较运算
from enum import IntEnum
class Priority(IntEnum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
```
