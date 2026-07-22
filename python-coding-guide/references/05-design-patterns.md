# 设计模式

本文档收录 Python 日常开发中最常用的设计模式，以 Pythonic 实用为主，避免过度设计。

> Python 有自己的惯用法，不要照搬 Java/C++ 的类体系。能用函数和字典解决的，就不要建类。

---

# 一、数据类（dataclass）

> 代替手写 `__init__`、`__repr__`、`__eq__`，字段多时首选。

```python
from dataclasses import dataclass, field

@dataclass
class UserCreateParam:
    name: str
    age: int
    email: str = ""
    tags: list[str] = field(default_factory=list)  # 可变默认值必须用 field

# 不可变版本
@dataclass(frozen=True)
class Point:
    x: float
    y: float
```

---

# 二、策略模式——函数字典

> 同一操作多种实现，按类型切换。Python 不需要接口+工厂，**函数字典**最直接。

```python
from typing import Callable

# 策略就是函数，注册到字典
PAYMENT_HANDLERS: dict[str, Callable[[Order], None]] = {
    "alipay": pay_alipay,
    "wechat": pay_wechat,
    "bank":   pay_bank,
}

def pay(order: Order, method: str) -> None:
    handler = PAYMENT_HANDLERS.get(method)
    if handler is None:
        raise ValueError(f"不支持的支付方式: {method}")
    handler(order)
```

策略较多、需要共享状态时才用类，此时用 `typing.Protocol` 做鸭子类型接口（3.8+）：

```python
from typing import Protocol

class Exporter(Protocol):
    def export(self, data: list[dict]) -> bytes: ...

class ExcelExporter:
    def export(self, data: list[dict]) -> bytes: ...

class CsvExporter:
    def export(self, data: list[dict]) -> bytes: ...

EXPORTERS: dict[str, Exporter] = {
    "excel": ExcelExporter(),
    "csv":   CsvExporter(),
}
```

> 选择：能用函数就用函数字典；策略内部逻辑复杂、有状态才用类 + Protocol。

---

# 三、装饰器（Decorator）

> Python 最 Pythonic 的模式。用于日志、缓存、重试、权限校验等横切关注点。

## 3.1 基本装饰器

```python
import functools
import logging

logger = logging.getLogger(__name__)

def log_call(func):
    @functools.wraps(func)  # 保留原函数的 __name__、__doc__
    def wrapper(*args, **kwargs):
        logger.debug("调用 %s", func.__name__)
        return func(*args, **kwargs)
    return wrapper

@log_call
def get_user(user_id: int) -> User:
    ...
```

## 3.2 带参数的装饰器

```python
def retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay * (attempt + 1))
        return wrapper
    return decorator

@retry(max_attempts=5, delay=2.0)
def call_external_api(url: str) -> dict:
    ...
```

> **优先用标准库**：`functools.lru_cache` 做缓存、`contextlib.suppress` 忽略异常，不要自己造轮子。

---

# 四、上下文管理器（Context Manager）

> 管理资源的获取和释放：文件、数据库连接、锁。

## 4.1 contextmanager 装饰器（最常用）

```python
from contextlib import contextmanager

@contextmanager
def db_transaction(session):
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise

# 使用
with db_transaction(session) as tx:
    tx.add(user)
```

## 4.2 类实现（需要复杂初始化时）

```python
class Timer:
    def __enter__(self):
        self.start = time.time()
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.time() - self.start
        return False  # 不吞异常

with Timer() as t:
    do_something()
```

---

# 五、生成器（Generator）

> Python 最有特色的模式——惰性计算，节省内存，简化迭代逻辑。

## 5.1 数据管道

```python
def read_large_file(path):
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            yield line.strip()

# 链式处理，全程只保留当前元素在内存中
lines = read_large_file(path)              # 生成器
valid = (l for l in lines if l)            # 生成器
results = (parse(l) for l in valid)        # 生成器
for r in results:
    save(r)
```

## 5.2 无限序列

```python
def natural_numbers(start=0):
    n = start
    while True:
        yield n
        n += 1

nums = natural_numbers()
first_10 = [next(nums) for _ in range(10)]
```

## 5.3 生成器 vs 列表

| 场景 | 选择 |
|------|------|
| 数据量大 / 流式处理 | 生成器 `(x for x in ...)` |
| 需要多次遍历、随机访问、len() | 列表 `[x for x in ...]` |

---

# 六、观察者模式——回调列表

> 事件触发多个后续动作。Python 用回调列表足够，不需要 EventBus 类。

```python
class OrderService:
    def __init__(self):
        self._on_created: list[Callable] = []

    def on_created(self, callback: Callable) -> None:
        self._on_created.append(callback)

    def create(self, order: Order) -> None:
        save_order(order)
        for callback in self._on_created:
            callback(order)

# 注册
service.on_created(send_notification)
service.on_created(deduct_stock)
```

事件类型较多需要按 key 分发时，用 `dict[str, list[Callable]]` 即可。

---

# 七、单例——模块即单例

> Python 模块天然是单例（import 只执行一次）。优先用模块级变量。

```python
# config.py — 模块即单例，import 到处用
class _Config:
    def __init__(self):
        self.settings = {}

config = _Config()
```

```python
# 其他文件
from config import config  # 全局唯一实例
```

> 不需要 `__new__` 或装饰器单例。如果必须延迟初始化，用函数 + 模块变量缓存即可。

---

# 八、速查表

| 问题 | Pythonic 方案 | 不要这么做 |
|------|--------------|-----------|
| 数据容器/参数对象 | `@dataclass` | 手写 `__init__`/`__repr__` |
| 同一操作多种实现 | 函数字典 | 接口 + 工厂类 |
| 横切逻辑（日志/缓存/重试） | 装饰器 `@functools.wraps` | 修改每个函数 |
| 资源管理（文件/连接/锁） | `with` / `@contextmanager` | try-finally 手动释放 |
| 大数据/流式处理 | 生成器 | 一次性读进列表 |
| 事件通知 | 回调列表 | EventBus 类体系 |
| 全局唯一实例 | 模块级变量 | `__new__` / 装饰器单例 |
| 忽略指定异常 | `contextlib.suppress` | `except: pass` |
| 函数缓存 | `functools.lru_cache` | 手写字典缓存 |
