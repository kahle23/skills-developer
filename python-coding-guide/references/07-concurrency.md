# 并发编程

Python 并发编程核心工具：线程、进程、异步、线程池。

> **选型前提**：先确定瓶颈类型再选方案——
> **I/O 密集**（网络请求、文件读写、数据库）→ `threading` / `asyncio`
> **CPU 密集**（数值计算、图像处理、压缩）→ `multiprocessing`
>

---

# 一、concurrent.futures（推荐入门）

> 线程池/进程池统一接口，日常使用最简单。

## 1.1 线程池

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def fetch_user(user_id: int) -> User:
    return api.get_user(user_id)

# 批量并行
user_ids = [1, 2, 3, 4, 5]
with ThreadPoolExecutor(max_workers=10) as executor:
    # 方式1：map — 保持顺序，异常会延迟到取结果时抛出
    users = list(executor.map(fetch_user, user_ids))

    # 方式2：submit + as_completed — 谁先完成谁先返回
    futures = {executor.submit(fetch_user, uid): uid for uid in user_ids}
    for future in as_completed(futures):
        uid = futures[future]
        try:
            user = future.result(timeout=30)  # 带超时
            process(uid, user)
        except Exception as e:
            logger.error("获取用户失败 uid=%s: %s", uid, e)
```

## 1.2 进程池

```python
from concurrent.futures import ProcessPoolExecutor

# CPU 密集任务用进程池
def heavy_compute(data: bytes) -> Result:
    ...

with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(heavy_compute, data_chunks))
```

| 场景 | 选择 |
|------|------|
| I/O 密集（网络/文件/数据库） | `ThreadPoolExecutor` |
| CPU 密集（计算/压缩/图像） | `ProcessPoolExecutor` |
| 不确定 | 先用线程池，CPU 跑不满再换进程池 |

---

# 二、threading（细粒度控制）

## 2.1 基本用法

```python
import threading

def worker(name: str, count: int) -> None:
    for i in range(count):
        logger.info("%s: %d", name, i)

t1 = threading.Thread(target=worker, args=("A", 5))
t2 = threading.Thread(target=worker, args=("B", 5))
t1.start()
t2.start()
t1.join()  # 等待结束
t2.join()
```

## 2.2 线程锁

```python
# threading.Lock — 互斥锁
lock = threading.Lock()
shared_data = []

def append_safely(item):
    with lock:  # 推荐用 with，自动释放
        shared_data.append(item)

# BAD — 容易忘记 unlock
lock.acquire()
try:
    shared_data.append(item)
finally:
    lock.release()
```

## 2.3 线程安全队列

```python
import queue
from threading import Thread

# 生产者-消费者模型
task_queue: queue.Queue = queue.Queue(maxsize=100)

def producer():
    for item in generate_items():
        task_queue.put(item)  # 队列满时阻塞
    task_queue.put(None)      # 哨兵值

def consumer():
    while True:
        item = task_queue.get()  # 队列空时阻塞
        if item is None:
            break
        process(item)
        task_queue.task_done()

Thread(target=producer).start()
Thread(target=consumer).start()
```

---

# 三、asyncio（异步 I/O）

> 高并发 I/O 场景（如爬虫、WebSocket 服务）性能最优，但代码是异步风格。

## 3.1 基本用法

```python
import asyncio

async def fetch_user(user_id: int) -> User:
    await asyncio.sleep(0.1)  # 模拟异步 I/O
    return User(id=user_id)

async def main():
    # 串行
    user1 = await fetch_user(1)
    user2 = await fetch_user(2)

    # 并发（推荐）
    users = await asyncio.gather(
        fetch_user(1),
        fetch_user(2),
        fetch_user(3),
    )

asyncio.run(main())
```

## 3.2 异步上下文管理器

```python
import aiohttp

async def fetch(url: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()
```

## 3.3 信号量控制并发数

```python
async def main():
    semaphore = asyncio.Semaphore(10)  # 最多 10 个并发

    async def limited_fetch(url):
        async with semaphore:
            return await fetch(url)

    results = await asyncio.gather(
        *[limited_fetch(url) for url in urls]
    )
```

> **注意**：asyncio 和 threading 不要随意混用。asyncio 是单线程协程，在协程内部不能调用阻塞的同步函数（会阻塞整个事件循环）。

---

# 四、multiprocessing（CPU 密集）

## 4.1 基本用法

```python
from multiprocessing import Process, Queue

def worker(task_queue: Queue, result_queue: Queue):
    while True:
        task = task_queue.get()
        if task is None:
            break
        result_queue.put(heavy_compute(task))

if __name__ == "__main__":  # Windows 必须!
    task_queue = Queue()
    result_queue = Queue()
    for item in tasks:
        task_queue.put(item)

    processes = [
        Process(target=worker, args=(task_queue, result_queue))
        for _ in range(4)
    ]
    for p in processes:
        task_queue.put(None)  # 哨兵
        p.start()
    for p in processes:
        p.join()
```

> **重要**：multiprocessing 代码必须放在 `if __name__ == "__main__":` 保护块内，否则 Windows/macOS 会无限递归创建子进程。

## 4.2 进程间共享数据

```python
from multiprocessing import Manager, Value

with Manager() as manager:
    shared_list = manager.list()
    shared_dict = manager.dict()
    counter = Value("i", 0)  # 共享整数
```

---

# 五、线程池使用模式

## 5.1 批量并行 + 收集结果

```python
from concurrent.futures import ThreadPoolExecutor

def process_user(user_id: int) -> ProcessResult:
    ...

with ThreadPoolExecutor(max_workers=10) as executor:
    # 批量提交
    futures = [executor.submit(process_user, uid) for uid in user_ids]
    # 等待全部完成，收集结果
    results = [f.result() for f in futures]
```

## 5.2 超时控制

```python
from concurrent.futures import ThreadPoolExecutor, TimeoutError as FutureTimeout

with ThreadPoolExecutor(max_workers=1) as executor:
    future = executor.submit(slow_task)
    try:
        result = future.result(timeout=10)  # 最多等 10 秒
    except FutureTimeout:
        logger.warning("任务超时")
        future.cancel()
```

---

# 六、常见陷阱

## 6.1 GIL 的真相

```python
# GIL（全局解释器锁）使得同一时刻只有一个线程执行 Python 字节码
# → 纯 Python 计算的多线程无法利用多核
# → 但 I/O 操作（网络/文件/数据库）会释放 GIL，所以 I/O 多线程仍然有效
```

## 6.2 线程安全

```python
# BAD — 非原子操作竞态条件
counter = 0
def increment():
    global counter
    counter += 1  # 不是原子操作！

# GOOD — 使用锁或原子类
import threading
lock = threading.Lock()
counter = 0
def increment():
    global counter
    with lock:
        counter += 1
```

## 6.3 资源清理

```python
# 线程池用 with 确保关闭
with ThreadPoolExecutor(max_workers=10) as executor:
    ...

# 不要这样 — 可能导致资源泄漏
executor = ThreadPoolExecutor(max_workers=10)
# ... 忘记 shutdown
```

---

# 七、并发模型速查

| 场景 | 推荐方案 |
|------|---------|
| I/O 密集，简单并行 | `ThreadPoolExecutor` |
| I/O 密集，超高并发 | `asyncio` + `aiohttp` |
| CPU 密集 | `ProcessPoolExecutor` / `multiprocessing` |
| 生产者-消费者 | `queue.Queue` + `threading` |
| 定时任务 | `threading.Timer` / `schedule` 库 |
| 需要取消任务 | `concurrent.futures`（submit + cancel） |
| 共享状态更新 | `threading.Lock` / `Manager` |
