# 性能优化

本文档收录 Python 日常开发中常见的性能优化技巧，以实用为主。

---

# 一、字符串优化

## 1.1 字符串拼接

```python
# BAD — 循环中用 +，每次创建新字符串
result = ""
for s in items:
    result += s

# GOOD — join（性能最优）
result = "".join(items)
result = ", ".join(items)    # 带分隔符

# GOOD — f-string（少量拼接，可读性好）
message = f"用户 {user.name}（ID: {user.id}）"
```

> **拼接性能**：`"".join()` > f-string > `%` 格式化 > `+` 拼接。大量拼接一律用 join。

## 1.2 字符串格式化选择

| 场景 | 推荐 | 示例 |
|------|------|------|
| 少量拼接（<5个变量） | f-string | `f"name={name}"` |
| 日志输出 | `%` 占位符 | `logger.info("id=%s", id)` |
| 大量拼接 | `"".join()` | `"".join(parts)` |
| 模板/多行 | f-string | `f"""..."""` |

---

# 二、数据结构选择

## 2.1 列表 vs 集合查找

```python
# BAD — list.in 是 O(n)
if user_id in id_list:
    ...

# GOOD — set.in 是 O(1)
if user_id in id_set:
    ...
```

> 反复判断元素是否存在的场景，将 list 转成 set。

## 2.2 推导式 vs 生成器

```python
# 数据量小（< 10000）— 推导式更快
squares = [x * x for x in range(1000)]

# 数据量大 — 生成器节省内存
squares = (x * x for x in range(1000000))

# 生成器 vs 列表：内存 O(1) vs O(n)，但生成器只能遍历一次
```

## 2.3 deque vs list

```python
from collections import deque

# BAD — list 头部插入/删除是 O(n)
items.insert(0, item)
items.pop(0)

# GOOD — deque 两端操作都是 O(1)
queue = deque([1, 2, 3])
queue.appendleft(0)   # 左侧插入
queue.popleft()       # 左侧弹出
```

---

# 三、循环优化

## 3.1 提取循环不变量

```python
# BAD — 每次循环都编译正则
for text in texts:
    match = re.search(r"\d+", text)

# GOOD — 编译一次，复用
pattern = re.compile(r"\d+")
for text in texts:
    match = pattern.search(text)
```

## 3.2 用内置函数替代循环

```python
# 用内置函数（C 实现，比 Python 循环快）

# GOOD — sum / max / min / any / all
total = sum(nums)                           # 比 for 循环累加快
has_active = any(u.is_active for u in users) # 找到第一个 True 就返回
all_valid = all(u.is_valid for u in users)   # 找到第一个 False 就返回
oldest = max(users, key=lambda u: u.age)

# BAD — 手写循环
total = 0
for n in nums:
    total += n
```

## 3.3 避免循环内重复计算

```python
# BAD — 每次循环都调用函数
for item in items:
    if get_threshold() > item.value:
        ...

# GOOD — 提取到循环外
threshold = get_threshold()
for item in items:
    if threshold > item.value:
        ...
```

---

# 四、字典 / 集合进阶

## 4.1 defaultdict 避免重复判空

```python
from collections import defaultdict

# BAD — 每次都要判断 key 是否存在
groups = {}
for user in users:
    dept_id = user.dept_id
    if dept_id not in groups:
        groups[dept_id] = []
    groups[dept_id].append(user)

# GOOD — defaultdict 自动初始化
groups = defaultdict(list)
for user in users:
    groups[user.dept_id].append(user)
```

## 4.2 Counter 统计

```python
from collections import Counter

# 统计词频
words = text.split()
word_counts = Counter(words)
top10 = word_counts.most_common(10)
```

---

# 五、I/O 优化

## 5.1 批量读写

```python
# BAD — 逐行写入
for line in lines:
    f.write(line + "\n")

# GOOD — 批量写入
f.writelines(line + "\n" for line in lines)
f.write("\n".join(lines))
```

## 5.2 大文件逐行处理

```python
# GOOD — for 逐行读（不一次性加载到内存）
with open(path, "r", encoding="utf-8") as f:
    for line in f:
        process(line)

# BAD — readlines() 加载全部到内存
lines = f.readlines()
```

---

# 六、内存优化

## 6.1 生成器

```python
# 生成器 — 惰性计算，节省内存
def read_large_file(path):
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            yield line.strip()

# 链式处理（每一步都是生成器）
lines = read_large_file(path)
valid_lines = (l for l in lines if l.strip())
results = (parse(l) for l in valid_lines)
for r in results:
    save(r)  # 全程只保留当前元素在内存中
```

## 6.2 slots 优化

```python
# 默认类用 __dict__ 存储属性，内存开销大
class Point:
    __slots__ = ("x", "y")  # 固定属性，节省约 40% 内存
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

---

# 七、缓存

```python
from functools import lru_cache, cache

# lru_cache — 纯函数缓存（参数相同则返回缓存结果）
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# cache — 无限缓存（Python 3.9+）
@cache
def expensive_computation(key: str) -> dict:
    ...
```

> **注意**：缓存函数的参数必须可哈希（list/dict/set 不能作为参数）。参数中含可变类型时用 `tuple` 或 `frozenset`。

---

# 八、优化原则

1. **先测量，再优化** — 用 `cProfile` / `timeit` 找到真正瓶颈，不要凭直觉
2. **二八法则** — 80% 性能问题出在 20% 代码上
3. **选择合适的数据结构** — 这是最有效的优化（set 代替 list 查找、deque 代替 list 队列）
4. **优先用内置函数** — C 实现远快于 Python 循环
5. **可读性优先** — 微优化不如清晰代码重要
