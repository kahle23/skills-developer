# 并发编程

Java 并发编程核心工具和最佳实践。

---

# 一、线程池

## 1.1 禁止使用 Executors 创建线程池

```java
// BAD — 阿里规约明确禁止，可能造成 OOM
ExecutorService pool = Executors.newFixedThreadPool(10);
ExecutorService pool = Executors.newCachedThreadPool();

// GOOD — 使用 ThreadPoolExecutor 显式指定参数
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    5,                                // corePoolSize
    10,                               // maximumPoolSize
    60L, TimeUnit.SECONDS,            // keepAliveTime, unit
    new LinkedBlockingQueue<>(200),   // workQueue（必须指定容量）
    new ThreadFactory() {             // threadFactory（自定义线程名）
        private final AtomicInteger counter = new AtomicInteger(1);
        @Override
        public Thread newThread(Runnable r) {
            return new Thread(r, "business-pool-" + counter.getAndIncrement());
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy()  // handler — 拒绝策略
);
```

| 参数 | 说明 |
|------|------|
| corePoolSize | 核心线程数 |
| maximumPoolSize | 最大线程数 |
| keepAliveTime | 空闲线程存活时间 |
| workQueue | 任务队列（必须指定容量，禁止无界） |
| threadFactory | 线程工厂（自定义线程名，便于排查） |
| handler | 拒绝策略 |

## 1.2 四种拒绝策略

| 策略 | 行为 | 适用场景 |
|------|------|---------|
| AbortPolicy（默认） | 抛 RejectedExecutionException | 需要感知队列溢出 |
| CallerRunsPolicy | 由调用线程执行任务 | **推荐**，不丢任务，天然限流 |
| DiscardPolicy | 静默丢弃新任务 | 允许丢任务且不关心 |
| DiscardOldestPolicy | 丢弃队列最旧的任务 | 新任务优先级更高 |

## 1.3 线程池参数调优参考

```
CPU 密集型：corePoolSize = CPU 核心数 + 1
IO 密集型：corePoolSize = CPU 核心数 × 2（或更多）
混合型：拆分为 CPU 密集和 IO 密集两个线程池
队列容量：不要用无界队列，建议 100~1000
```

## 1.4 线程池优雅关闭

```java
pool.shutdown();  // 不再接受新任务，等待已提交任务完成
try {
    if (!pool.awaitTermination(30, TimeUnit.SECONDS)) {
        pool.shutdownNow();  // 超时强制关闭
    }
} catch (InterruptedException e) {
    pool.shutdownNow();
    Thread.currentThread().interrupt();
}
```

## 1.5 常见线程池使用模式

```java
// 模式1：批量并行处理，收集结果
List<Future<Result>> futures = new ArrayList<>();
for (Task task : tasks) {
    futures.add(pool.submit(() -> process(task)));
}
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
```

---

# 二、锁

## 2.1 synchronized

```java
public synchronized void increment() { count++; }  // 方法锁

public void process() {
    synchronized (lockObject) {   // 代码块锁（推荐，粒度更小）
        count++;
    }
}
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
    try { /* 获取到锁 */ }
    finally { lock.unlock(); }
} else {
    // 未获取到锁，执行其他逻辑
}
```

## 2.3 读写锁 ReentrantReadWriteLock

```java
private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();

public String getData(String key) {        // 读操作 — 多线程可同时读
    rwLock.readLock().lock();
    try { return cache.get(key); }
    finally { rwLock.readLock().unlock(); }
}

public void putData(String key, String value) {  // 写操作 — 独占
    rwLock.writeLock().lock();
    try { cache.put(key, value); }
    finally { rwLock.writeLock().unlock(); }
}
```

## 2.4 锁选择建议

```
简单场景 → synchronized（JVM 自动管理，不易遗漏释放）
需要 tryLock / 超时 / 公平锁 → ReentrantLock
读多写少 → ReentrantReadWriteLock
优先考虑无锁方案（ConcurrentHashMap、原子类）
```

---

# 三、原子类

```java
// AtomicInteger
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();       // ++i，返回新值
counter.getAndIncrement();      // i++，返回旧值
counter.addAndGet(5);            // i += 5
counter.compareAndSet(10, 20);   // CAS，当前值是10则设为20

// LongAdder（Java 8+，高并发计数推荐，比 AtomicLong 更高效）
LongAdder adder = new LongAdder();
adder.increment();
adder.add(5);
long total = adder.sum();  // 注意：sum 不是原子操作，适合统计场景

// AtomicBoolean（常用于状态标记）
AtomicBoolean processed = new AtomicBoolean(false);
if (processed.compareAndSet(false, true)) {  // 只有一个线程能进入
    doProcess();
}
```

---

# 四、CompletableFuture（Java 8+）

```java
// 创建（推荐使用自定义线程池）
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> queryFromDB(), businessPool);

// 链式调用
CompletableFuture<User> future = CompletableFuture
    .supplyAsync(() -> userService.getById(userId), pool)
    .thenApply(user -> orderService.getByUserId(user.getId()))
    .thenApply(order -> detailService.buildDetail(order))
    .exceptionally(ex -> { log.error("处理失败", ex); return defaultDetail; });

// 组合多个异步任务
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> userService.getById(userId), pool);
CompletableFuture<List<Order>> orderFuture = CompletableFuture.supplyAsync(() -> orderService.queryByUserId(userId), pool);
CompletableFuture.allOf(userFuture, orderFuture).join();
User user = userFuture.get();
List<Order> orders = orderFuture.get();

// 合并结果
UserDetail detail = userFuture.thenCombine(orderFuture, (u, o) -> new UserDetail(u, o)).join();
```

**关键提醒：**
- 必须指定自定义线程池，避免使用默认 ForkJoinPool.commonPool（全局共享，容易阻塞）
- 链式调用末尾必须加 `.exceptionally()` 处理异常，否则异常被吞掉
- `get()` 必须带超时：`future.get(10, TimeUnit.SECONDS)`

---

# 五、并发集合

```java
// ConcurrentHashMap — 线程安全的 HashMap
ConcurrentHashMap<String, User> map = new ConcurrentHashMap<>();
map.putIfAbsent("key", value);               // 不存在才放入
map.computeIfAbsent("key", k -> new User(k)); // 不存在才计算
map.merge("key", 1, Integer::sum);           // 原子合并

// BlockingQueue — 生产者-消费者模型
BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>(100);
taskQueue.put(task);                          // 阻塞等待直到有空间
Task task = taskQueue.poll(5, TimeUnit.SECONDS); // 阻塞等待直到有数据
```

| 类型 | 场景 | 注意点 |
|------|------|--------|
| CopyOnWriteArrayList | 读多写少 | 写时复制整个数组，开销大 |
| ConcurrentLinkedQueue | 无界线程安全队列 | CAS 实现，高性能，无阻塞 |

---

# 六、ThreadLocal

```java
private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

public void login(User user) { currentUser.set(user); }
public User getCurrentUser() { return currentUser.get(); }
public void logout() { currentUser.remove(); }  // 必须清除，防止内存泄漏
```

**关键提醒：** 必须在 `finally` 或拦截器中调用 `remove()`，否则线程池复用时会导致内存泄漏和数据串号。

---

# 七、CountDownLatch / CyclicBarrier / Semaphore

```java
// CountDownLatch — 一次性等待 N 个任务完成
CountDownLatch latch = new CountDownLatch(5);
for (int i = 0; i < 5; i++) {
    pool.execute(() -> { try { doSomething(); } finally { latch.countDown(); } });
}
latch.await(30, TimeUnit.SECONDS);

// CyclicBarrier — 所有线程到齐后一起执行
CyclicBarrier barrier = new CyclicBarrier(3, () -> log.info("全部就绪"));
for (int i = 0; i < 3; i++) {
    pool.execute(() -> { prepare(); barrier.await(); execute(); });
}

// Semaphore — 控制并发数
Semaphore semaphore = new Semaphore(10);
pool.execute(() -> {
    try { semaphore.acquire(); doSomething(); }
    finally { semaphore.release(); }
});
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
