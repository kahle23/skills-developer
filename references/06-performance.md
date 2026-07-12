# 性能优化

本文档收录 Java 日常开发中常见的性能优化技巧，以实用为主。

---

# 一、字符串优化

## 1.1 字符串拼接

```java
// BAD — 循环中用 + 拼接，每次创建新对象
String result = "";
for (String s : list) {
    result += s;
}

// GOOD — StringBuilder
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);
}
String result = sb.toString();

// GOOD — StringBuilder 指定初始容量（知道大致长度时）
StringBuilder sb = new StringBuilder(list.size() * 16);
for (String s : list) {
    sb.append(s);
}

// GOOD — StringJoiner（Java 8+，带分隔符）
StringJoiner joiner = new StringJoiner(", ");
for (String s : list) {
    joiner.add(s);
}
String result = joiner.toString();

// GOOD — Stream（Java 8+）
String result = list.stream().collect(Collectors.joining(", "));
String result = String.join(", ", list);  // 更简洁
```

## 1.2 字符串比较

```java
// BAD
if (str.equals("value")) { }
if (str == "value") { }       // 引用比较，不可靠

// GOOD — 常量放前面，避免 NPE
if ("value".equals(str)) { }

// 使用工具方法
if (Objects.equals(str1, str2)) { }     // null 安全
if (StringUtils.equals(str1, str2)) { } // null 安全
```

---

# 二、集合优化

## 2.1 初始化容量

```java
// BAD — 默认容量，频繁扩容
Map<String, Object> map = new HashMap<>();
List<String> list = new ArrayList<>();

// GOOD — 指定容量，避免扩容
int expectedSize = 100;
Map<String, Object> map = new HashMap<>(expectedSize * 4 / 3 + 1);  // 负载因子 0.75
List<String> list = new ArrayList<>(expectedSize);

// Guava
Map<String, Object> map = Maps.newHashMapWithExpectedSize(100);
List<String> list = Lists.newArrayListWithExpectedSize(100);
```

## 2.2 避免不必要的转换

```java
// BAD — 先转 List 再遍历
Set<Long> idSet = new HashSet<>(idList);
for (Long id : idList) {  // 应该遍历 idSet
    process(id);
}

// GOOD — 直接遍历 Set
for (Long id : idSet) {
    process(id);
}

// BAD — List.contains 是 O(n)
if (idList.contains(targetId)) { }

// GOOD — Set.contains 是 O(1)
if (idSet.contains(targetId)) { }
```

## 2.3 集合选择

```java
// 频繁随机访问 → ArrayList（O(1)）
// 频繁插入/删除 → LinkedList（O(1)）
// 频繁查找 → HashMap（O(1)）/ TreeMap（O(log n)）
// 需要去重 → HashSet
// 需要排序 → TreeSet / TreeMap
// 并发环境 → ConcurrentHashMap / CopyOnWriteArrayList
```

---

# 三、避免自动装箱

```java
// BAD — 自动装箱，大量临时 Long 对象
Long sum = 0L;
for (int i = 0; i < 1000000; i++) {
    sum += i;  // 每次都拆箱 + 装箱
}

// GOOD — 使用基本类型
long sum = 0L;
for (int i = 0; i < 1000000; i++) {
    sum += i;
}

// BAD — 集合中使用包装类型
List<Long> ids = new ArrayList<>();  // Long 是对象，占用更多内存

// 如果数量极大且对内存敏感，考虑使用基本类型数组
long[] ids = new long[1000000];

// BAD — 频繁的 Long 比较
Long a = 128L;
Long b = 128L;
if (a == b) { }  // 可能失败！超出缓存范围

// GOOD
if (a.equals(b)) { }
if (Objects.equals(a, b)) { }
```

---

# 四、循环优化

```java
// BAD — 循环内重复计算
for (int i = 0; i < list.size(); i++) {  // list.size() 每次调用
    process(list.get(i));
}

// GOOD — 缓存大小
int size = list.size();
for (int i = 0; i < size; i++) {
    process(list.get(i));
}

// 增强 for 循环（推荐，除非需要索引）
for (User user : users) {
    process(user);
}

// BAD — 循环内创建对象
for (String s : list) {
    Pattern p = Pattern.compile("\\d+");  // 每次编译正则
    Matcher m = p.matcher(s);
}

// GOOD — 提到循环外
Pattern p = Pattern.compile("\\d+");
for (String s : list) {
    Matcher m = p.matcher(s);
}
```

---

# 五、异常优化

```java
// BAD — 用异常控制流程（异常创建开销大）
try {
    User user = getUser(id);
} catch (UserNotFoundException e) {
    user = createDefaultUser();
}

// GOOD — 先检查再操作
User user = getUser(id);
if (user == null) {
    user = createDefaultUser();
}

// BAD — 捕获通用异常
try {
    doSomething();
} catch (Exception e) {
    // 太宽泛
}

// GOOD — 捕获具体异常
try {
    doSomething();
} catch (IOException e) {
    // 具体处理
}
```

---

# 六、I/O 优化

## 6.1 缓冲流

```java
// BAD — 无缓冲
InputStream is = new FileInputStream("file.txt");
int b;
while ((b = is.read()) != -1) {  // 逐字节读取
    process((byte) b);
}

// GOOD — 使用缓冲
try (BufferedReader reader = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        process(line);
    }
}
```

## 6.2 try-with-resources

```java
// BAD — 手动关闭，容易遗漏
InputStream is = null;
try {
    is = new FileInputStream(path);
    // ...
} finally {
    if (is != null) {
        try { is.close(); } catch (IOException ignored) { }
    }
}

// GOOD — 自动关闭（Java 7+）
try (InputStream is = new FileInputStream(path);
     OutputStream os = new FileOutputStream(output)) {
    byte[] buffer = new byte[8192];
    int len;
    while ((len = is.read(buffer)) != -1) {
        os.write(buffer, 0, len);
    }
}
```

---

# 七、并发优化

```java
// BAD — 不必要的同步
public synchronized void read() {  // 读操作不需要同步
    return this.value;
}

// GOOD — 读写分离
private final ReadWriteLock lock = new ReentrantReadWriteLock();

public void read() {
    lock.readLock().lock();
    try {
        return this.value;
    } finally {
        lock.readLock().unlock();
    }
}

public void write(Object value) {
    lock.writeLock().lock();
    try {
        this.value = value;
    } finally {
        lock.writeLock().unlock();
    }
}

// 使用 ConcurrentHashMap 替代 Collections.synchronizedMap
Map<String, Object> map = new ConcurrentHashMap<>();  // 分段锁，性能更好
```

---

# 八、序列化优化

```java
// 如果对象很大，考虑使用更高效的序列化
// Jackson（通用）
ObjectMapper mapper = new ObjectMapper();
byte[] bytes = mapper.writeValueAsBytes(obj);

// Hessian（二进制，更紧凑）
Hessian2Output output = new Hessian2Output(os);
output.writeObject(obj);

// Kryo（最快，但需要注册类）
Kryo kryo = new Kryo();
kryo.register(User.class);
kryo.writeObject(output, user);
```

---

# 九、优化原则

1. **先测量，再优化** — 不要凭直觉优化，用 Profiler 找到真正的瓶颈
2. **二八法则** — 80% 的性能问题出在 20% 的代码上
3. **可读性优先** — 微优化不如清晰的代码重要
4. **选择合适的数据结构** — 这是最大的优化
5. **减少对象创建** — 复用对象、使用基本类型、避免装箱
6. **批处理** — 合并多次 I/O 为一次批量操作
