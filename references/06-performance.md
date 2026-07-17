# 性能优化

本文档收录 Java 日常开发中常见的性能优化技巧，以实用为主。

---

# 一、字符串优化

## 1.1 字符串拼接

```java
// BAD — 循环中用 +，每次创建新对象
String result = "";
for (String s : list) { result += s; }

// GOOD — StringBuilder（知道大小时指定容量）
StringBuilder sb = new StringBuilder(list.size() * 16);
for (String s : list) { sb.append(s); }
String result = sb.toString();

// GOOD — Stream / String.join
String result = list.stream().collect(Collectors.joining(", "));
String result = String.join(", ", list);
```

## 1.2 字符串比较

> 常量放前面避免 NPE：`"value".equals(str)`。null 安全比较：`Objects.equals(s1, s2)`。

---

# 二、集合优化

## 2.1 初始化容量

```java
// BAD — 默认容量，频繁扩容
Map<String, Object> map = new HashMap<>();
List<String> list = new ArrayList<>();

// GOOD — 指定容量（HashMap 负载因子 0.75）
Map<String, Object> map = new HashMap<>(expectedSize * 4 / 3 + 1);
List<String> list = new ArrayList<>(expectedSize);
```

## 2.2 避免不必要的转换

```java
// BAD — List.contains 是 O(n)
if (idList.contains(targetId)) { }

// GOOD — Set.contains 是 O(1)
if (idSet.contains(targetId)) { }
```

> 集合选择指南详见 [04-集合与 Stream](./04-collection-and-stream.md#11-集合选择指南)。

---

# 三、避免自动装箱

```java
// BAD — 自动装箱，大量临时 Long 对象
Long sum = 0L;
for (int i = 0; i < 1000000; i++) { sum += i; }

// GOOD — 使用基本类型
long sum = 0L;
for (int i = 0; i < 1000000; i++) { sum += i; }

// Long 等包装类型比较必须用 equals，不能用 ==
// BAD: if (a == b)     GOOD: if (a.equals(b)) / Objects.equals(a, b)
```

---

# 四、循环优化

```java
// BAD — 循环内创建重复对象
for (String s : list) {
    Pattern p = Pattern.compile("\\d+");  // 每次编译正则
    Matcher m = p.matcher(s);
}

// GOOD — 提到循环外
Pattern p = Pattern.compile("\\d+");
for (String s : list) {
    Matcher m = p.matcher(s);
}

// 推荐增强 for 循环，除非需要索引
for (User user : users) { process(user); }
```

---

# 五、异常优化

```java
// BAD — 异常创建开销大，不要用异常控制流程
try { user = getUser(id); } catch (UserNotFoundException e) { user = createDefault(); }

// GOOD — 先检查
User user = getUser(id);
if (user == null) { user = createDefaultUser(); }
```

> 异常处理规范（捕获具体异常、try-with-resources）详见 [03-异常与日志](./03-exception-and-logging.md)。

---

# 六、I/O 优化

> 始终使用 `BufferedReader`/`BufferedWriter` 包装流。`try-with-resources` 详见 [03-异常与日志](./03-exception-and-logging.md)。

---

# 七、数据库批量操作

```java
// BAD — 循环内逐条删除/查询，N 次数据库访问
for (BillDetail d : details) { billDetailService.deleteById(d.getId()); }

// GOOD — 收集 ID 批量操作，1 次数据库访问
List<Long> ids = details.stream().map(BillDetail::getId).distinct().collect(Collectors.toList());
billDetailService.removeByIds(ids);
```

**注意事项**：
1. 收集 ID 时务必 `.distinct()`
2. 数据量 > 1000 时分批执行，避免 SQL 过长
3. 批量操作应在同一事务内，确保原子性

> 并发优化（`ConcurrentHashMap`、`ReadWriteLock`）详见 [07-并发编程](./07-concurrent.md)。

---

# 八、优化原则

1. **先测量，再优化** — 用 Profiler 找到真正瓶颈，不要凭直觉
2. **二八法则** — 80% 性能问题出在 20% 代码上，集中优化热点
3. **可读性优先** — 微优化不如清晰代码重要；选择合适的数据结构是最有效的优化
