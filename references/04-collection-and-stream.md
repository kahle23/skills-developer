# 集合与 Stream

Java 集合框架和 Stream API 是日常开发中使用最频繁的部分。

---

# 一、集合基础

## 1.1 集合选择指南

| 需求 | 首选 |
|------|------|
| 有序可重复、随机访问多 | `ArrayList` |
| 有序可重复、插入/删除多 | `LinkedList` |
| 无序不可重复 | `HashSet` |
| 不可重复、需排序 | `TreeSet` |
| 不可重复、保持插入顺序 | `LinkedHashSet` |
| 键值对通用 | `HashMap` |
| 键值对需排序 | `TreeMap` |
| 键值对保持插入顺序 | `LinkedHashMap` |
| 并发安全键值对 | `ConcurrentHashMap` |

## 1.2 集合初始化

```java
// Java 8+
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));  // 可变

// Java 9+（不可变）
List<String> list = List.of("a", "b", "c");
Map<String, Integer> map = Map.of("a", 1, "b", 2);
Set<Integer> set = Set.of(1, 2, 3);

// Guava（不可变）
List<String> list = ImmutableList.of("a", "b", "c");
Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 2);
```

## 1.3 List 操作

```java
// 条件删除
users.removeIf(u -> u.getAge() < 18);

// 排序
users.sort(Comparator.comparing(User::getName));
users.sort(Comparator.comparing(User::getAge).reversed().thenComparing(User::getName));

// 子列表（修改子列表会影响原列表）
List<User> sub = users.subList(0, 10);
```

## 1.4 Map 高级操作

```java
userMap.putIfAbsent("u1", user);                      // 不存在才放入
userMap.computeIfAbsent("u1", k -> loadUser(k));       // 不存在才计算——适合懒加载缓存
userMap.computeIfPresent("u1", (k, v) -> modify(v));   // 存在才计算
userMap.merge("u1", 1, Integer::sum);                  // 合并值

// 遍历
userMap.forEach((k, v) -> log.info("{}: {}", k, v));
```

## 1.5 Set 集合运算

```java
Set<String> a = new HashSet<>(Arrays.asList("a", "b", "c"));
Set<String> b = new HashSet<>(Arrays.asList("b", "c", "d"));

Set<String> inter = new HashSet<>(a); inter.retainAll(b);  // 交集 [b, c]
Set<String> union = new HashSet<>(a); union.addAll(b);     // 并集 [a, b, c, d]
Set<String> diff  = new HashSet<>(a); diff.removeAll(b);   // 差集 [a]
```

---

# 二、Stream API

## 2.1 创建 Stream

```java
Stream<User> s1 = list.stream();                                  // 从集合
Stream<User> s2 = list.parallelStream();                          // 并行流
Stream<String> s3 = Arrays.stream(array);                         // 从数组
Stream<String> s4 = Stream.of("a", "b");                          // 直接创建
Stream<Integer> s5 = Stream.iterate(0, n -> n + 1).limit(10);     // 0~9
```

## 2.2 中间操作（惰性求值）

| 操作 | 说明 |
|------|------|
| `filter(pred)` | 过滤——保留满足条件的元素 |
| `map(Function)` | 映射——转换类型 |
| `flatMap(Function)` | 扁平化嵌套集合 |
| `distinct()` | 去重——基于 `equals` |
| `sorted()` / `sorted(Comparator)` | 排序 |
| `limit(n)` | 截取前 n 个 |
| `skip(n)` | 跳过前 n 个 |
| `peek(Consumer)` | 调试——查看中间结果 |

## 2.3 终端操作

```java
// 收集为 List / Map / 分组 / 分区
List<String> names = users.stream().map(User::getName).collect(Collectors.toList());

Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity(), (u1, u2) -> u1));

Map<Integer, List<User>> byAge = users.stream()
    .collect(Collectors.groupingBy(User::getAge));

Map<Integer, Long> countByAge = users.stream()
    .collect(Collectors.groupingBy(User::getAge, Collectors.counting()));

Map<Boolean, List<User>> p = users.stream()
    .collect(Collectors.partitioningBy(u -> u.getAge() > 18));

// 连接
String joined = names.stream().collect(Collectors.joining(", "));
String joined = names.stream().collect(Collectors.joining(", ", "[", "]"));

// 聚合
long count = users.stream().count();
int sum = users.stream().mapToInt(User::getAge).sum();
double avg = users.stream().mapToInt(User::getAge).average().orElse(0);
```

## 2.4 Stream 格式规范

**两行格式**：第一行 `.stream()`，第二行放所有链式操作。

**ID 去重**：收集 ID 时务必 `.distinct()`：

```java
List<Long> ids = list.stream()
        .map(Item::getId).distinct().collect(Collectors.toList());
```

**只取去重 key 不需要 groupingBy**——直接用 distinct 即可，分组有额外开销。

## 2.5 常见 Stream 模式

```java
// 1. 提取 ID   2. List → Map(ID→对象)   3. List → Map(ID→字段)
List<Long> ids = users.stream().map(User::getId).distinct().collect(Collectors.toList());
Map<Long, User> uMap = users.stream().collect(Collectors.toMap(User::getId, Function.identity()));
Map<Long, String> nMap = users.stream().collect(Collectors.toMap(User::getId, User::getName));

// 4. 按字段分组   5. 过滤+提取
Map<String, List<User>> gMap = users.stream().collect(Collectors.groupingBy(User::getDeptName));
List<String> names = users.stream().filter(User::isActive).map(User::getName).collect(Collectors.toList());

// 6. 排序TopN   7. 扁平化嵌套
List<User> top10 = users.stream()
    .sorted(Comparator.comparing(User::getAge).reversed()).limit(10).collect(Collectors.toList());
List<Role> roles = users.stream()
    .map(User::getRoles).flatMap(Collection::stream).distinct().collect(Collectors.toList());

// 8. 统计
IntSummaryStatistics s = users.stream().mapToInt(User::getAge).summaryStatistics();
```

## 2.6 并行 Stream

> **谨慎使用**：仅数据量大（> 10w）+ 无状态 + 无副作用时使用。并行收集到 Map 须用 `Collectors.toConcurrentMap()`。

---

# 三、泛型速查

```java
// 泛型类
public class Result<T> {
    private T data;
    public static <T> Result<T> success(T data) { ... }
}

// PECS 原则：Producer Extends（只读），Consumer Super（只写）
public void printAll(List<? extends Number> list) { }   // ? extends T
public void addAll(List<? super Integer> list) { }      // ? super T

// 类型擦除：运行时无法获取泛型。解决：传入 Class<T> 参数
```
