# 并发编程

本文档涵盖 Java 并发编程中的核心工具和最佳实践，包括线程池、锁、原子类、CompletableFuture、并发集合等。

---

# 一、线程池

## 1.1 禁止使用 Executors 创建线程池

```java
// BAD — 阿里规约明确禁止，可能造成 OOM
ExecutorService pool = Executors.newFixedThreadPool(10);
ExecutorService pool = Executors.newCachedThreadPool();
ExecutorService pool = Executors.newSingleThreadExecutor();
ExecutorService pool = Executors.newScheduledThreadPool(5);

// GOOD — 使用 ThreadPoolExecutor 显式指定参数
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    5,                              // corePoolSize — 核心线程数
    10,                             // maximumPoolSize — 最大线程数
    60L,                            // keepAliveTime — 空闲线程存活时间
    TimeUnit.SECONDS,               // 时间单位
    new LinkedBlockingQueue<>(200), // workQueue — 任务队列（必须指定容量）
    new ThreadFactory() {           // threadFactory — 线程工厂（自定义线程名）
        private final AtomicInteger counter = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "business-pool-" + counter.getAndIncrement());
            t.setDaemon(false);
            return t;
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy() // handler — 拒绝策略
);
```

## 1.2 四种拒绝策略

```java
// 1. AbortPolicy（默认）— 抛出 RejectedExecutionException
new ThreadPoolExecutor.AbortPolicy();

// 2. CallerRunsPolicy — 由调用线程执行任务（推荐，不丢任务）
new ThreadPoolExecutor.CallerRunsPolicy();

// 3. DiscardPolicy — 静默丢弃新任务（注意任务丢失风险）
new ThreadPoolExecutor.DiscardPolicy();

// 4. DiscardOldestPolicy — 丢弃队列最旧的任务
new ThreadPoolExecutor.DiscardOldestPolicy();

// 自定义拒绝策略（如记录日志后丢弃）
new RejectedExecutionHandler() {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        log.warn("任务被拒绝，队列已满，活跃线程: {}", executor.getActiveCount());
    }
}
```

## 1.3 线程池参数调优参考

```
CPU 密集型任务：
  核心线程数 = CPU 核心数 + 1
  例如：8 核机器 → corePoolSize = 9

IO 密集型任务：
  核心线程数 = CPU 核心数 × 2（或更多）
  例如：8 核机器 → corePoolSize = 16

混合型任务：
  拆分为 CPU 密集和 IO 密集两个线程池，分别配置

队列容量：
  不要用无界队列（默认 Integer.MAX_VALUE），建议 100~1000
```

## 1.4 线程池优雅关闭

```java
// 1. shutdown — 不再接受新任务，等待已提交任务完成
pool.shutdown();

// 2. 等待所有任务完成（带超时）
try {
    if (!pool.awaitTermination(30, TimeUnit.SECONDS)) {
        pool.shutdownNow();  // 超时强制关闭
    }
} catch (InterruptedException e) {
    pool.shutdownNow();
    Thread.currentThread().interrupt();
}

// 3. shutdownNow — 立即停止，尝试中断正在执行的任务
List<Runnable> unfinished = pool.shutdownNow();

// Spring 中使用 @PreDestroy
@PreDestroy
public void destroy() {
    pool.shutdown();
}
```

## 1.5 常见线程池使用模式

```java
// 模式1：批量并行处理，收集结果
List<Future<Result>> futures = new ArrayList<>();
for (Task task : tasks) {
    futures.add(pool.submit(() -> process(task)));
}
List<Result> results = new ArrayList<>();
for (Future<Result> future : futures) {
    results.add(future.get(30, TimeUnit.SECONDS));  // 带超时
}

// 模式2：异步执行，不关心结果
pool.execute(() -> {
    try {
        sendNotification(userId, message);
    } catch (Exception e) {
        log.error("发送通知失败", e);
    }
});

// 模式3：定时/周期执行（ScheduledThreadPool）
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
scheduler.scheduleAtFixedRate(() -> {
    cleanExpiredData();
}, 0, 1, TimeUnit.HOURS);  // 每小时执行一次，首次立即执行
```

---

# 二、锁

## 2.1 synchronized

```java
// 方法级锁
public synchronized void increment() {
    count++;
}

// 代码块锁（推荐，锁粒度更小）
public void process() {
    // 非临界区代码...
    synchronized (lockObject) {
        // 临界区代码
        count++;
    }
    // 非临界区代码...
}

// 类锁
public static synchronized void globalMethod() { }

// 注意：synchronized 锁的是对象，不是代码
```

## 2.2 ReentrantLock

```java
private final ReentrantLock lock = new ReentrantLock();

public void process() {
    lock.lock();
    try {
        // 临界区代码
    } finally {
        lock.unlock();  // 必须在 finally 中释放
    }
}

// 尝试获取锁（非阻塞）
if (lock.tryLock()) {
    try {
        // 获取到锁
    } finally {
        lock.unlock();
    }
} else {
    // 未获取到锁，执行其他逻辑
}

// 尝试获取锁（带超时）
if (lock.tryLock(3, TimeUnit.SECONDS)) {
    try {
        // 3秒内获取到锁
    } finally {
        lock.unlock();
    }
} else {
    log.warn("获取锁超时");
}

// 公平锁（按等待顺序获取，性能略低）
ReentrantLock fairLock = new ReentrantLock(true);
```

## 2.3 读写锁 ReentrantReadWriteLock

```java
private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
private final ReentrantReadWriteLock.ReadLock readLock = rwLock.readLock();
private final ReentrantReadWriteLock.WriteLock writeLock = rwLock.writeLock();

// 读操作 — 多线程可同时读
public String getData(String key) {
    readLock.lock();
    try {
        return cache.get(key);
    } finally {
        readLock.unlock();
    }
}

// 写操作 — 独占
public void putData(String key, String value) {
    writeLock.lock();
    try {
        cache.put(key, value);
    } finally {
        writeLock.unlock();
    }
}
```

## 2.4 锁选择建议

```
synchronized vs ReentrantLock：
├─ 简单场景 → synchronized（JVM 自动管理，不易遗漏释放）
├─ 需要 tryLock / 超时 / 公平锁 → ReentrantLock
├─ 需要多个条件变量 → ReentrantLock + Condition
└─ 读多写少 → ReentrantReadWriteLock

注意：优先考虑无锁方案（如 ConcurrentHashMap、原子类）
```

---

# 三、原子类

```java
import java.util.concurrent.atomic.*;

// AtomicInteger
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();    // ++i，返回新值
counter.getAndIncrement();    // i++，返回旧值
counter.decrementAndGet();    // --i
counter.addAndGet(5);         // i += 5
counter.compareAndSet(10, 20); // CAS，当前值是10则设为20
counter.get();                // 获取当前值

// AtomicLong
AtomicLong idGenerator = new AtomicLong(0);
long id = idGenerator.incrementAndGet();

// AtomicReference
AtomicReference<User> currentUser = new AtomicReference<>(null);
currentUser.compareAndSet(null, user);  // 仅当为null时设置

// LongAdder（Java 8+，高并发计数推荐，比 AtomicLong 更高效）
LongAdder adder = new LongAdder();
adder.increment();
adder.add(5);
long total = adder.sum();  // 注意：sum 不是原子操作，适合统计场景

// AtomicBoolean（常用于状态标记）
AtomicBoolean processed = new AtomicBoolean(false);
if (processed.compareAndSet(false, true)) {
    // 只有一个线程能进入这里
    doProcess();
}
```

---

# 四、CompletableFuture（Java 8+）

## 4.1 创建

```java
// 1. 已知结果
CompletableFuture<String> cf = CompletableFuture.completedFuture("hello");

// 2. 异步执行（使用 ForkJoinPool.commonPool）
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    return queryFromDB();
});

// 3. 异步执行（使用自定义线程池，推荐）
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    return queryFromDB();
}, businessPool);
```

## 4.2 链式调用

```java
CompletableFuture<User> future = CompletableFuture
    .supplyAsync(() -> userService.getById(userId), pool)       // 异步查询用户
    .thenApply(user -> orderService.getByUserId(user.getId()))  // 转换：查询订单
    .thenApply(order -> detailService.buildDetail(order))       // 转换：构建详情
    .exceptionally(ex -> {                                      // 异常处理
        log.error("处理失败", ex);
        return defaultDetail;
    });
```

## 4.3 组合多个异步任务

```java
// 场景：同时查询用户信息和订单列表，然后合并

CompletableFuture<User> userFuture = CompletableFuture
    .supplyAsync(() -> userService.getById(userId), pool);

CompletableFuture<List<Order>> orderFuture = CompletableFuture
    .supplyAsync(() -> orderService.queryByUserId(userId), pool);

// 等待两个任务都完成
CompletableFuture.allOf(userFuture, orderFuture).join();

// 获取结果
User user = userFuture.get();
List<Order> orders = orderFuture.get();

// 或者合并结果
CompletableFuture<UserDetail> detailFuture = userFuture.thenCombine(
    orderFuture,
    (user, orders) -> new UserDetail(user, orders)
);
```

## 4.4 超时控制（Java 9+）

```java
// 带超时
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> slowQuery(), pool)
    .orTimeout(5, TimeUnit.SECONDS)        // 5秒超时抛 TimeoutException
    .completeOnTimeout("default", 5, TimeUnit.SECONDS);  // 5秒超时返回默认值
```

## 4.5 批量并行处理

```java
// 批量查询，取所有结果
List<CompletableFuture<Result>> futures = ids.stream()
    .map(id -> CompletableFuture.supplyAsync(
        () -> queryById(id), pool))
    .collect(Collectors.toList());

// 等待全部完成
List<Result> results = futures.stream()
    .map(CompletableFuture::join)  // join 不抛受检异常
    .collect(Collectors.toList());
```

## 4.6 注意事项

```java
// BAD — 不指定线程池，使用 ForkJoinPool.commonPool（全局共享，容易阻塞）
CompletableFuture.supplyAsync(() -> heavyIO());

// GOOD — 使用自定义线程池
CompletableFuture.supplyAsync(() -> heavyIO(), ioPool);

// BAD — 异常被吞掉
future.thenApply(result -> {
    process(result);  // 如果这里抛异常，后续链路会出问题
    return result;
});

// GOOD — 异常处理
future.thenApply(result -> {
    process(result);
    return result;
}).exceptionally(ex -> {
    log.error("处理失败", ex);
    return defaultValue;
});

// BAD — 无限等待
Result result = future.get();

// GOOD — 带超时
Result result = future.get(10, TimeUnit.SECONDS);
```

---

# 五、并发集合

```java
import java.util.concurrent.*;

// ConcurrentHashMap — 线程安全的 HashMap
ConcurrentHashMap<String, User> map = new ConcurrentHashMap<>();
map.put("key", value);
map.putIfAbsent("key", value);            // 原子操作：不存在才放入
map.computeIfAbsent("key", k -> new User(k)); // 原子操作：不存在才计算
map.merge("key", 1, Integer::sum);        // 原子操作：合并

// CopyOnWriteArrayList — 读多写少场景
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("item");  // 写时复制整个数组，开销大
list.get(0);       // 读无锁，高效

// ConcurrentLinkedQueue — 无界线程安全队列（CAS 实现，高性能）
ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
queue.offer("item");  // 入队
queue.poll();          // 出队（null 如果空）

// BlockingQueue — 阻塞队列（生产者-消费者模型）
// LinkedBlockingQueue — 有界/无界阻塞队列
BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>(100);
taskQueue.put(task);     // 阻塞等待直到有空间
Task task = taskQueue.poll(5, TimeUnit.SECONDS);  // 阻塞等待直到有数据

// ArrayBlockingQueue — 有界数组阻塞队列（更节省内存）
BlockingQueue<Task> queue = new ArrayBlockingQueue<>(100);

// LinkedTransferQueue — 高性能无界队列
TransferQueue<Task> tq = new LinkedTransferQueue<>();
tq.transfer(task);    // 阻塞直到有消费者取走
tq.tryTransfer(task, 5, TimeUnit.SECONDS);  // 带超时
```

---

# 六、ThreadLocal

```java
// 线程本地变量，每个线程独立副本
private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

// 使用
public void login(User user) {
    currentUser.set(user);
}
public User getCurrentUser() {
    return currentUser.get();
}
public void logout() {
    currentUser.remove();  // 必须在 finally 或拦截器中清除，防止内存泄漏
}

// 带初始值
private static final ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// 典型使用场景
// 1. 用户上下文（登录信息）
// 2. 数据库连接（同一线程复用连接）
// 3. 请求链路追踪 ID

// Spring 中的 RequestContextHolder 就是基于 ThreadLocal
```

---

# 七、CountDownLatch / CyclicBarrier / Semaphore

```java
// CountDownLatch — 一次性等待（等 N 个任务完成后再继续）
int taskCount = 5;
CountDownLatch latch = new CountDownLatch(taskCount);

for (int i = 0; i < taskCount; i++) {
    pool.execute(() -> {
        try {
            doSomething();
        } finally {
            latch.countDown();  // 每完成一个任务减 1
        }
    });
}

latch.await(30, TimeUnit.SECONDS);  // 等待所有任务完成（或超时）
log.info("所有任务已完成");

// CyclicBarrier — 循环屏障（所有线程到齐后再一起执行）
int threadCount = 3;
CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> {
    log.info("所有线程已就绪，开始执行");  // 到齐后执行的回调
});

for (int i = 0; i < threadCount; i++) {
    pool.execute(() -> {
        prepare();
        barrier.await();  // 等待其他线程
        execute();         // 所有线程一起执行
    });
}

// Semaphore — 信号量（控制并发数，如限流）
Semaphore semaphore = new Semaphore(10);  // 最多 10 个并发

pool.execute(() -> {
    try {
        semaphore.acquire();     // 获取许可（阻塞）
        // 最多同时 10 个线程执行这里
        doSomething();
    } finally {
        semaphore.release();     // 释放许可
    }
});

// 非阻塞尝试
if (semaphore.tryAcquire(5, TimeUnit.SECONDS)) {
    try { doSomething(); }
    finally { semaphore.release(); }
}
```

---

# 八、并发编程速查

| 场景 | 推荐方案 |
|------|---------|
| 共享计数器 | `AtomicLong` / `LongAdder` |
| 线程安全 Map | `ConcurrentHashMap` |
| 读多写少的 List | `CopyOnWriteArrayList` |
| 生产者-消费者 | `BlockingQueue` |
| 并行执行 + 收集结果 | `CompletableFuture.allOf` |
| 批量并行 + 合并 | `CompletableFuture.thenCombine` |
| 控制并发数 | `Semaphore` / 线程池 |
| 等待多个任务完成 | `CountDownLatch` |
| 多线程同步开始 | `CyclicBarrier` |
| 线程上下文传递 | `ThreadLocal` |
