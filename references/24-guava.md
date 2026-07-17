# Guava 工具库

Google Guava 是 Java 开发最广泛使用的工具库之一，提供了不可变集合、缓存、并发工具、字符串处理等。

---

# 一、不可变集合

线程安全、无需同步、可安全共享。

```java
import com.google.common.collect.*;

// of / copyOf / builder 三种创建方式
ImmutableList<String> list = ImmutableList.of("a", "b", "c");
ImmutableList<String> list = ImmutableList.<String>builder().add("a").add("b").add("c").build();
ImmutableList<String> list = ImmutableList.copyOf(existingList);

ImmutableSet<Long> set = ImmutableSet.of(1L, 2L, 3L);

ImmutableMap<String, Integer> map = ImmutableMap.of("a", 1, "b", 2);
ImmutableMap<String, Integer> map = ImmutableMap.<String, Integer>builder()
    .put("a", 1).put("b", 2).put("c", 3).build();
```

---

# 二、可变集合工厂

```java
import com.google.common.collect.*;

// List
List<String> list = Lists.newArrayList("a", "b", "c");
List<String> list = Lists.newArrayListWithExpectedSize(100);

// Set / Map
Set<String> set = Sets.newHashSet("a", "b", "c");
Set<String> set = Sets.newLinkedHashSet();     // 保持插入顺序
Map<String, Integer> map = Maps.newHashMapWithExpectedSize(100);
Map<String, Integer> map = Maps.newLinkedHashMap();
```

---

# 三、集合操作

```java
import com.google.common.collect.*;

// 列表分页
List<List<String>> partition = Lists.partition(list, 100);

// 集合运算
Set<String> intersection = Sets.intersection(set1, set2);
Set<String> union = Sets.union(set1, set2);
Set<String> difference = Sets.difference(set1, set2);

// Map 差异
MapDifference<String, Integer> diff = Maps.difference(map1, map2);
diff.entriesOnlyOnLeft();   // 仅 map1 有
diff.entriesDiffering();    // key 相同 value 不同

// 集合转 Map / Multimap
ImmutableMap<Long, User> map = Maps.uniqueIndex(users, User::getId);  // key 必须唯一
ImmutableListMultimap<String, User> multimap = Multimaps.index(users, User::getDeptName);
```

---

# 四、Preconditions — 前置条件校验

```java
import com.google.common.base.Preconditions;

Preconditions.checkNotNull(name, "名称不能为空");
Preconditions.checkArgument(age > 0, "年龄必须大于0");
Preconditions.checkState(isInitialized(), "服务未初始化");
Preconditions.checkElementIndex(index, size, "索引越界");
```

---

# 五、Strings — 字符串

```java
import com.google.common.base.Strings;

Strings.padStart("5", 3, '0')   // "005"
Strings.padEnd("5", 3, '0')     // "500"
Strings.repeat("ab", 3)         // "ababab"
Strings.isNullOrEmpty(str)      // true
Strings.nullToEmpty(str)        // null → ""
Strings.emptyToNull(str)        // "" → null
```

---

# 六、Joiner / Splitter

```java
import com.google.common.base.Joiner;
import com.google.common.base.Splitter;

// Joiner — 连接
Joiner.on(", ").join("a", "b", "c")                    // "a, b, c"
Joiner.on(", ").skipNulls().join("a", null, "c")       // "a, c"
Joiner.on(", ").useForNull("N/A").join("a", null)      // "a, N/A"

// Splitter — 分割
Splitter.on(',').splitToList("a,b,c")                  // ["a", "b", "c"]
Splitter.on(',').trimResults().omitEmptyStrings()
    .splitToList(" a , , b , c ")                     // ["a", "b", "c"]
Splitter.on("&").withKeyValueSeparator("=").split("a=1&b=2")  // {"a":"1","b":"2"}
```

---

# 七、Optional

> Java 8+ 推荐使用 `java.util.Optional`。主要差异：Guava `Optional` 实现 `Serializable`（`java.util.Optional` 不实现），`or(Optional)` 返回 `Optional` 而非值。

```java
// Guava                                          // java.util
Optional.fromNullable(val)                         // Optional.ofNullable(val)
optional.or(default)                               // optional.orElse(default)
```

---

# 八、Cache — 本地缓存

```java
import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

Cache<String, User> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .recordStats()
    .build();

// 操作
cache.put("user:1", user);                          // 放入
User user = cache.getIfPresent("user:1");           // 获取（不存在返回 null）
User user = cache.get("user:1", () -> selectById(1L)); // 获取或加载（推荐）
cache.invalidate("user:1");                         // 删除单个
cache.invalidateAll();                              // 清空

// 统计
CacheStats stats = cache.stats();
stats.hitRate();   // 命中率
stats.missCount(); // 未命中次数
```

---

# 九、Hashing — 哈希

```java
import com.google.common.hash.Hashing;

Hashing.md5().hashString("hello", StandardCharsets.UTF_8).toString()      // MD5
Hashing.sha256().hashString("hello", StandardCharsets.UTF_8).toString()   // SHA256
```

> 另有 `sha512` / `hmacSha256` / `BloomFilter` 等方法，需要时查阅。

---

# 十、RateLimiter — 限流

```java
import com.google.common.util.concurrent.RateLimiter;

RateLimiter limiter = RateLimiter.create(10.0);       // 每秒10个请求
limiter.acquire();                                    // 阻塞获取
boolean ok = limiter.tryAcquire();                    // 非阻塞尝试
boolean ok = limiter.tryAcquire(100, TimeUnit.MILLISECONDS);  // 等待100ms
```

---

# 十一、EventBus — 事件总线

```java
import com.google.common.eventbus.EventBus;
import com.google.common.eventbus.Subscribe;

EventBus eventBus = new EventBus("myBus");

eventBus.register(new Object() {
    @Subscribe
    public void onUserCreated(UserCreatedEvent event) {
        log.info("用户创建: {}", event.getUserId());
    }
});

eventBus.post(new UserCreatedEvent(123L));
```
