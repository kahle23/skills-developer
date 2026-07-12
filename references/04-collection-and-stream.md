# 集合与 Stream

Java 集合框架和 Stream API 是日常开发中使用最频繁的部分。本文档涵盖 List、Set、Map 的最佳用法，以及 Java 8 Stream API 和泛型的常见模式。

---

# 一、集合基础

## 1.1 集合选择指南

```
需要什么？
│
├─ 有序、可重复 → List
│   ├─ 随机访问多 → ArrayList
│   └─ 插入/删除多 → LinkedList
│
├─ 无序、不可重复 → Set
│   ├─ 通用 → HashSet
│   ├─ 需要排序 → TreeSet
│   └─ 保持插入顺序 → LinkedHashSet
│
└─ 键值对 → Map
    ├─ 通用 → HashMap
    ├─ 需要排序 → TreeMap
    ├─ 保持插入顺序 → LinkedHashMap
    └─ 并发安全 → ConcurrentHashMap
```

## 1.2 集合初始化

```java
// ====== Java 6/7 ======
List<String> list = new ArrayList<String>();
list.add("a");
list.add("b");
list.add("c");

// 使用 Arrays.asList（固定大小，不能 add/remove）
List<String> list = Arrays.asList("a", "b", "c");

// ====== Java 8+ ======
List<String> list = Arrays.asList("a", "b", "c");  // 固定大小
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));  // 可变

// ====== Java 9+ ======
List<String> list = List.of("a", "b", "c");         // 不可变
Map<String, Integer> map = Map.of("a", 1, "b", 2);  // 不可变
Set<Integer> set = Set.of(1, 2, 3);                  // 不可变

// ====== Guava ======
List<String> list = ImmutableList.of("a", "b", "c");
Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 2);
```

## 1.3 List 操作

```java
List<User> users = new ArrayList<>();

// 添加
users.add(user);
users.addAll(otherList);
users.add(0, user);  // 指定位置插入

// 查询
User user = users.get(0);
int index = users.indexOf(user);
boolean exists = users.contains(user);
int size = users.size();

// 删除
users.remove(0);            // 按索引
users.remove(user);         // 按对象
users.removeIf(u -> u.getAge() < 18);  // 条件删除（Java 8+）
users.clear();

// 排序
Collections.sort(users);                            // 自然排序
Collections.sort(users, Comparator.reverseOrder()); // 逆序
users.sort(Comparator.comparing(User::getName));    // 按字段排序（Java 8+）
users.sort(Comparator.comparing(User::getAge).reversed().thenComparing(User::getName));

// 子列表
List<User> sub = users.subList(0, 10);  // [0, 10)
```

## 1.4 Map 操作

```java
Map<String, User> userMap = new HashMap<>();

// 添加
userMap.put("u1", user1);
userMap.putAll(otherMap);

// 查询
User user = userMap.get("u1");
User user = userMap.getOrDefault("u1", defaultUser);  // Java 8+
boolean exists = userMap.containsKey("u1");
int size = userMap.size();

// 条件操作（Java 8+）
userMap.putIfAbsent("u1", user);             // 不存在才放入
userMap.computeIfAbsent("u1", k -> new User(k));  // 不存在才计算
userMap.computeIfPresent("u1", (k, v) -> {   // 存在才计算
    v.setAge(v.getAge() + 1);
    return v;
});
userMap.merge("u1", 1, Integer::sum);  // 合并值

// 遍历
for (Map.Entry<String, User> entry : userMap.entrySet()) {
    String key = entry.getKey();
    User value = entry.getValue();
}

// Java 8+ 遍历
userMap.forEach((key, value) -> {
    log.info("key: {}, value: {}", key, value);
});

// 删除
userMap.remove("u1");
userMap.remove("u1", user);  // 值匹配才删除（Java 8+）
```

## 1.5 Set 操作

```java
Set<Long> idSet = new HashSet<>();

// 添加
idSet.add(1L);
idSet.addAll(otherSet);

// 查询
boolean exists = idSet.contains(1L);
int size = idSet.size();

// 删除
idSet.remove(1L);
idSet.removeIf(id -> id < 100);  // Java 8+

// 集合运算
Set<String> a = new HashSet<>(Arrays.asList("a", "b", "c"));
Set<String> b = new HashSet<>(Arrays.asList("b", "c", "d"));

// 交集
Set<String> intersection = new HashSet<>(a);
intersection.retainAll(b);  // [b, c]

// 并集
Set<String> union = new HashSet<>(a);
union.addAll(b);  // [a, b, c, d]

// 差集
Set<String> diff = new HashSet<>(a);
diff.removeAll(b);  // [a]
```

---

# 二、Stream API（Java 8+）

## 2.1 创建 Stream

```java
// 从集合
Stream<User> stream = list.stream();
Stream<User> parallelStream = list.parallelStream();

// 从数组
Stream<String> stream = Arrays.stream(array);
Stream<String> stream = Arrays.stream(array, 0, 5);  // 指定范围

// 直接创建
Stream<String> stream = Stream.of("a", "b", "c");
Stream<Integer> stream = Stream.iterate(0, n -> n + 1).limit(10);  // 0~9
Stream<Double> stream = Stream.generate(Math::random).limit(5);    // 5个随机数
```

## 2.2 中间操作（惰性求值）

```java
list.stream()
    // 过滤
    .filter(user -> user.getAge() > 18)

    // 映射
    .map(User::getName)                    // 提取字段
    .flatMap(user -> user.getRoles().stream())  // 扁平化嵌套集合

    // 去重
    .distinct()

    // 排序
    .sorted()                              // 自然排序
    .sorted(Comparator.comparing(User::getAge).reversed())  // 自定义排序

    // 截取
    .limit(10)                             // 取前10个
    .skip(5)                               // 跳过前5个

    // 调试
    .peek(user -> log.debug("处理: {}", user))

    // 执行终端操作...
```

## 2.3 终端操作

```java
// 收集为集合
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());

Set<Long> ids = users.stream()
    .map(User::getId)
    .collect(Collectors.toSet());

// 收集为 Map
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity()));
// 有重复 key 时需要合并函数
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity(), (u1, u2) -> u1));

// 分组
Map<Integer, List<User>> groupByAge = users.stream()
    .collect(Collectors.groupingBy(User::getAge));

// 分组 + 下游收集
Map<Integer, Long> countByAge = users.stream()
    .collect(Collectors.groupingBy(User::getAge, Collectors.counting()));

Map<Integer, List<String>> nameByAge = users.stream()
    .collect(Collectors.groupingBy(User::getAge,
        Collectors.mapping(User::getName, Collectors.toList())));

// 分区（按 boolean 分两组）
Map<Boolean, List<User>> partitioned = users.stream()
    .collect(Collectors.partitioningBy(user -> user.getAge() > 18));

// 连接字符串
String joined = names.stream()
    .collect(Collectors.joining(", "));
String joined = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));  // 带前后缀

// 聚合
long count = users.stream().count();
int sum = users.stream().mapToInt(User::getAge).sum();
double avg = users.stream().mapToInt(User::getAge).average().orElse(0);
int max = users.stream().mapToInt(User::getAge).max().orElse(0);
int min = users.stream().mapToInt(User::getAge).min().orElse(0);

// 归约
int total = users.stream()
    .map(User::getAge)
    .reduce(0, Integer::sum);

// 遍历
users.stream().forEach(System.out::println);

// 匹配
boolean anyMatch = users.stream().anyMatch(user -> user.getAge() > 18);
boolean allMatch = users.stream().allMatch(user -> user.getAge() > 0);
boolean noneMatch = users.stream().noneMatch(user -> user.getAge() < 0);

// 查找
Optional<User> first = users.stream().filter(user -> user.getAge() > 18).findFirst();
Optional<User> any = users.stream().filter(user -> user.getAge() > 18).findAny();
```

## 2.4 常见 Stream 模式

```java
// 模式1：提取 ID 列表
List<Long> userIds = users.stream()
    .map(User::getId)
    .distinct()
    .collect(Collectors.toList());

// 模式2：List 转 Map（ID -> 对象）
Map<Long, User> userMap = users.stream()
    .collect(Collectors.toMap(User::getId, Function.identity()));

// 模式3：List 转 Map（ID -> 名称）
Map<Long, String> nameMap = users.stream()
    .collect(Collectors.toMap(User::getId, User::getName));

// 模式4：按字段分组
Map<String, List<User>> deptUserMap = users.stream()
    .collect(Collectors.groupingBy(User::getDeptName));

// 模式5：过滤 + 提取
List<String> activeNames = users.stream()
    .filter(User::isActive)
    .map(User::getName)
    .collect(Collectors.toList());

// 模式6：排序 + 取前N
List<User> top10 = users.stream()
    .sorted(Comparator.comparing(User::getAge).reversed())
    .limit(10)
    .collect(Collectors.toList());

// 模式7：扁平化嵌套集合
List<Role> allRoles = users.stream()
    .map(User::getRoles)
    .flatMap(Collection::stream)
    .distinct()
    .collect(Collectors.toList());

// 模式8：统计
IntSummaryStatistics stats = users.stream()
    .mapToInt(User::getAge)
    .summaryStatistics();
// stats.getCount(), stats.getSum(), stats.getAverage(), stats.getMax(), stats.getMin()
```

## 2.5 并行 Stream

```java
// 注意：并非所有场景都适合并行
// 适合：数据量大 + 操作无状态 + 无副作用
// 不适合：数据量小、需要顺序、有共享状态

List<Result> results = bigList.parallelStream()
    .map(this::heavyProcess)
    .collect(Collectors.toList());

// 并行流中的线程安全问题
// BAD — HashMap 不是线程安全的
Map<Long, User> map = users.parallelStream()
    .collect(Collectors.toMap(User::getId, Function.identity()));

// GOOD — 使用 ConcurrentHashMap
Map<Long, User> map = users.parallelStream()
    .collect(Collectors.toConcurrentMap(User::getId, Function.identity()));
```

---

# 三、泛型

## 3.1 泛型类

```java
// 基本泛型类
public class Result<T> {
    private int code;
    private String message;
    private T data;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setData(data);
        return result;
    }
}

// 多泛型参数
public class Pair<K, V> {
    private K key;
    private V value;
}
```

## 3.2 泛型方法

```java
// 泛型方法
public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    return list.stream().filter(predicate).collect(Collectors.toList());
}

// 使用
List<User> activeUsers = filter(users, User::isActive);
List<Order> unpaidOrders = filter(orders, order -> !order.isPaid());
```

## 3.3 泛型通配符

```java
// ? extends T — 上界通配符（只读）
// 只能读，不能写（因为不知道具体类型）
public void printAll(List<? extends Number> list) {
    for (Number n : list) {
        System.out.println(n);
    }
    // list.add(1);  // 编译错误

    Number first = list.get(0);  // OK
}

// ? super T — 下界通配符（只写）
// 可以写 T 及其子类，读出来是 Object
public void addAll(List<? super Integer> list) {
    list.add(1);       // OK
    list.add(2);       // OK
    // Integer n = list.get(0);  // 编译错误，只能读出 Object
    Object obj = list.get(0);   // OK
}

// PECS 原则：Producer Extends, Consumer Super
// 如果是生产者（提供数据），用 extends
// 如果是消费者（接收数据），用 super

// 典型应用：Collections.copy
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));
    }
}
```

## 3.4 类型擦除注意事项

```java
// Java 泛型是类型擦除的，运行时无法获取泛型类型
public class Container<T> {
    private T value;

    public void check() {
        // if (value instanceof String) {}  // OK
        // if (T instanceof String) {}      // 编译错误
        // new T();                          // 编译错误
    }
}

// 获取泛型类型的变通方式
// 1. 传入 Class 参数
public <T> T create(Class<T> clazz) {
    return clazz.newInstance();
}

// 2. 使用 TypeReference（如 Guava、Jackson 的做法）
public abstract class TypeReference<T> {
    private final Type type;

    protected TypeReference() {
        Type superClass = getClass().getGenericSuperclass();
        this.type = ((ParameterizedType) superClass).getActualTypeArguments()[0];
    }

    public Type getType() { return type; }
}

// 使用
Type type = new TypeReference<List<String>>() {}.getType();
```
