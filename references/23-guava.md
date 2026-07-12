# Guava 工具库

Google Guava 是 Java 开发中最广泛使用的工具库之一，提供了不可变集合、缓存、并发工具、字符串处理等。

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>32.1.3-jre</version>
</dependency>
```

---

# 一、不可变集合

不可变集合线程安全、无需同步、可安全共享。

```java
import com.google.common.collect.*;

// List
ImmutableList<String> list = ImmutableList.of("a", "b", "c");
ImmutableList<String> list = ImmutableList.<String>builder()
    .add("a").add("b").add("c").build();
ImmutableList<String> list = ImmutableList.copyOf(existingList);

// Set
ImmutableSet<Long> set = ImmutableSet.of(1L, 2L, 3L);
ImmutableSet<Long> set = ImmutableSet.copyOf(existingSet);

// Map
ImmutableMap<String, Integer> map = ImmutableMap.of("a", 1, "b", 2);
// 超过 5 个 entry 用 builder
ImmutableMap<String, Integer> map = ImmutableMap.<String, Integer>builder()
    .put("a", 1).put("b", 2).put("c", 3).build();
ImmutableMap<String, Integer> map = ImmutableMap.copyOf(existingMap);

// Multimap（一个 key 对应多个 value）
ImmutableListMultimap<String, String> multimap = ImmutableListMultimap.of(
    "fruits", "apple", "fruits", "banana", "veggies", "carrot"
);
List<String> fruits = multimap.get("fruits");  // ["apple", "banana"]
```

---

# 二、可变集合工厂

```java
import com.google.common.collect.*;

// List
List<String> list = Lists.newArrayList();
List<String> list = Lists.newArrayList("a", "b", "c");
List<String> list = Lists.newArrayListWithCapacity(100);
List<String> list = Lists.newArrayListWithExpectedSize(100);
LinkedList<String> linked = Lists.newLinkedList();

// Set
Set<String> set = Sets.newHashSet();
Set<String> set = Sets.newHashSet("a", "b", "c");
Set<String> set = Sets.newLinkedHashSet();  // 保持插入顺序
TreeSet<String> sorted = Sets.newTreeSet();

// Map
Map<String, Integer> map = Maps.newHashMap();
Map<String, Integer> map = Maps.newLinkedHashMap();  // 保持插入顺序
TreeMap<String, Integer> sorted = Maps.newTreeMap();
Map<String, Integer> map = Maps.newHashMapWithExpectedSize(100);

// 双向 Map（BiMap）
BiMap<String, Integer> biMap = HashBiMap.create();
biMap.put("one", 1);
biMap.inverse().get(1);  // "one" — 反向查找

// Table（二维 Map）
Table<String, String, Integer> table = HashBasedTable.create();
table.put("row1", "col1", 1);
table.row("row1");     // {"col1": 1}
table.column("col1");  // {"row1": 1}
```

---

# 三、集合操作

```java
import com.google.common.collect.*;
import com.google.common.base.Predicates;

// 过滤
List<String> filtered = FluentIterable.from(list)
    .filter(s -> s.startsWith("a"))
    .toList();

// 转换
List<Integer> lengths = FluentIterable.from(list)
    .transform(String::length)
    .toList();

// 分割列表（每 N 个一组）
List<List<String>> partition = Lists.partition(list, 100);

// 笛卡尔积
List<List<Integer>> product = Lists.cartesianProduct(list1, list2);

// 排列
ImmutableList<ImmutableList<String>> permutations = Collections2.permutations(list);

// 交集、并集、差集（使用 Sets 工具类）
Set<String> intersection = Sets.intersection(set1, set2);
Set<String> union = Sets.union(set1, set2);
Set<String> difference = Sets.difference(set1, set2);
Set<String> symmetricDiff = Sets.symmetricDifference(set1, set2);

// Map 操作
MapDifference<String, Integer> diff = Maps.difference(map1, map2);
diff.entriesOnlyOnLeft();   // 只在 map1 中
diff.entriesOnlyOnRight();  // 只在 map2 中
diff.entriesInCommon();     // 两者共有
diff.entriesDiffering();    // key 相同但 value 不同

// 集合转 Map
ImmutableMap<Long, User> map = Maps.uniqueIndex(users, User::getId);  // key 必须唯一
ImmutableListMultimap<String, User> multimap = Multimaps.index(users, User::getDeptName);

// 频率统计
Multiset<String> multiset = HashMultiset.create();
multiset.add("a");
multiset.add("a");
multiset.count("a");  // 2
```

---

# 四、Preconditions — 前置条件校验

```java
import com.google.common.base.Preconditions;

public void process(String name, int age, List<Long> ids) {
    // checkNotNull — 校验非空
    Preconditions.checkNotNull(name, "名称不能为空");
    Preconditions.checkNotNull(name, "名称不能为空，传入值: %s", name);  // 支持格式化

    // checkArgument — 校验条件
    Preconditions.checkArgument(age > 0, "年龄必须大于0，实际: %s", age);
    Preconditions.checkArgument(ids != null && !ids.isEmpty(), "ID列表不能为空");

    // checkState — 校验状态
    Preconditions.checkState(isInitialized(), "服务未初始化");
    Preconditions.checkState(queue.size() < MAX_SIZE, "队列已满");

    // checkElementIndex — 校验索引
    Preconditions.checkElementIndex(index, size, "索引越界");

    // checkPositionIndex — 校验位置
    Preconditions.checkPositionIndex(position, size, "位置越界");
}
```

---

# 五、Strings — 字符串工具

```java
import com.google.common.base.Strings;

// 填充
Strings.padStart("5", 3, '0');     // "005"
Strings.padEnd("5", 3, '0');       // "500"

// 重复
Strings.repeat("ab", 3);           // "ababab"

// 判断
Strings.isNullOrEmpty(str);        // true
Strings.nullToEmpty(str);          // null → ""
Strings.emptyToNull(str);          // "" → null

// 截断
Strings.shortenMiddle("abcdefghij", 7);  // "abcd...ij"（自定义实现）
```

---

# 六、Joiner / Splitter — 连接与分割

```java
import com.google.common.base.Joiner;
import com.google.common.base.Splitter;

// Joiner — 连接
String result = Joiner.on(", ").join("a", "b", "c");         // "a, b, c"
String result = Joiner.on(", ").join(list);                    // "a, b, c"
String result = Joiner.on(", ").skipNulls().join("a", null, "c");  // "a, c"
String result = Joiner.on(", ").useForNull("N/A").join("a", null);  // "a, N/A"

// Joiner 转 Map
String result = Joiner.on("&").withKeyValueSeparator("=").join(map);  // "a=1&b=2"

// Splitter — 分割
List<String> parts = Splitter.on(',').splitToList("a,b,c");        // ["a", "b", "c"]
List<String> parts = Splitter.on(',')
    .trimResults()                      // 去空格
    .omitEmptyStrings()                 // 去空串
    .splitToList(" a , , b , c ");      // ["a", "b", "c"]

// 按长度分割
List<String> parts = Splitter.fixedLength(3).splitToList("abcdefgh");  // ["abc", "def", "gh"]

// 按正则分割
List<String> parts = Splitter.onPattern("\\d+").splitToList("a1b2c");  // ["a", "b", "c"]

// Splitter 转 Map
Map<String, String> map = Splitter.on("&")
    .withKeyValueSeparator("=")
    .split("a=1&b=2");  // {"a": "1", "b": "2"}

// limit 限制分割数量
List<String> parts = Splitter.on(',').limit(2).splitToList("a,b,c");  // ["a", "b,c"]
```

---

# 七、Optional — Guava 版（Java 8+ 推荐用 java.util.Optional）

```java
import com.google.common.base.Optional;

Optional<String> optional = Optional.of("value");
Optional<String> optional = Optional.fromNullable(nullableValue);
Optional<String> optional = Optional.absent();

optional.isPresent();                      // true
optional.get();                            // "value"
optional.or("default");                    // "default" if absent
optional.orNull();                         // null if absent
optional.or(Optional.of("default"));       // Optional("default")
```

---

# 八、Cache — 本地缓存

```java
import com.google.common.cache.Cache;
import com.google.common.cache.CacheBuilder;

// 构建缓存
Cache<String, User> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)                     // 最大条目数
    .expireAfterWrite(10, TimeUnit.MINUTES) // 写入后 10 分钟过期
    .expireAfterAccess(5, TimeUnit.MINUTES) // 访问后 5 分钟过期
    .concurrencyLevel(8)                   // 并发级别
    .recordStats()                         // 记录统计信息
    .build();

// 放入
cache.put("user:1", user);
cache.putAll(map);

// 获取
User user = cache.getIfPresent("user:1");  // 返回 null 如果不存在

// 获取或加载（推荐）
User user = cache.get("user:1", () -> {
    return userMapper.selectById(1L);      // 缓存未命中时加载
});

// 删除
cache.invalidate("user:1");               // 删除单个
cache.invalidateAll();                     // 删除所有
cache.invalidateAll(keys);                 // 批量删除

// 统计
CacheStats stats = cache.stats();
stats.hitRate();                           // 命中率
stats.hitCount();                          // 命中次数
stats.missCount();                         // 未命中次数
stats.loadCount();                         // 加载次数
stats.averageLoadPenalty();                // 平均加载耗时

// 手动刷新
cache.refresh("user:1");
```

---

# 九、Hashing — 哈希工具

```java
import com.google.common.hash.Hashing;
import com.google.common.hash.HashCode;

// MD5
HashCode md5 = Hashing.md5().hashString("hello", StandardCharsets.UTF_8);
String hex = md5.toString();  // "5d41402abc4b2a76b9719d911017c592"

// SHA256
HashCode sha256 = Hashing.sha256().hashString("hello", StandardCharsets.UTF_8);

// SHA512
HashCode sha512 = Hashing.sha512().hashString("hello", StandardCharsets.UTF_8);

// HMAC
HashCode hmac = Hashing.hmacSha256(secretKey.getBytes())
    .hashString("hello", StandardCharsets.UTF_8);

// Bloom Filter
BloomFilter<String> filter = BloomFilter.create(
    Funnels.stringFunnel(StandardCharsets.UTF_8),
    1000,    // 预期插入数量
    0.01     // 误判率
);
filter.put("hello");
filter.mightContain("hello");  // true（可能）
filter.mightContain("world");  // 大概率 false
```

---

# 十、RateLimiter — 限流器

```java
import com.google.common.util.concurrent.RateLimiter;

// 创建限流器（每秒 10 个请求）
RateLimiter limiter = RateLimiter.create(10.0);

// 阻塞等待获取令牌
limiter.acquire();           // 阻塞直到获取到令牌
limiter.acquire(5);          // 获取 5 个令牌

// 尝试获取（非阻塞）
boolean success = limiter.tryAcquire();            // 立即返回
boolean success = limiter.tryAcquire(100, TimeUnit.MILLISECONDS);  // 等待 100ms
```

---

# 十一、EventBus — 事件总线

```java
import com.google.common.eventbus.EventBus;
import com.google.common.eventbus.Subscribe;

// 创建 EventBus
EventBus eventBus = new EventBus("myEventBus");

// 注册监听器
eventBus.register(new Object() {
    @Subscribe
    public void onUserCreated(UserCreatedEvent event) {
        log.info("用户创建: {}", event.getUserId());
    }
});

// 发布事件
eventBus.post(new UserCreatedEvent(123L));

// 异步 EventBus
AsyncEventBus asyncEventBus = new AsyncEventBus(Executors.newFixedThreadPool(4));
```
