# Lombok 使用指南

Lombok 通过注解自动生成样板代码（getter/setter/构造方法/日志等），大幅减少重复代码量。适用于 Java 1.6+ 项目。

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
```

---

# 一、实体类注解

## 1.1 @Data

最常用的注解，组合了 `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor`。

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
// 自动生成：getId/setName/getAge/setAge/toString/equals/hashCode/无参构造
```

## 1.2 @NoArgsConstructor / @AllArgsConstructor

```java
@NoArgsConstructor   // 无参构造
@AllArgsConstructor  // 全参构造
public class User {
    private Long id;
    private String name;
}

// 部分参数构造用 @RequiredArgsConstructor（final 字段 + @NonNull 字段）
@RequiredArgsConstructor
public class UserService {
    @NonNull
    private final UserMapper userMapper;  // 这个字段会出现在构造方法中
    private String config;                // 这个不会
}
```

---

# 二、日志注解

## 2.1 @Slf4j

```java
@Slf4j
public class UserService {
    public void process(Long userId) {
        log.info("处理用户，ID: {}", userId);
        log.debug("详细信息: {}", detail);
        log.error("处理失败", e);
    }
}
// 等价于：
// private static final Logger log = LoggerFactory.getLogger(UserService.class);
```

## 2.2 其他日志注解

```java
@Log4j        // java.util.logging.Logger
@Log4j2       // Log4j2
@CommonsLog   // Apache Commons Logging
@Flogger      // Google Flogger
@JBossLog     // JBoss Logger
@XSlf4j       // SLF4J with topic (X = Extended)
```

---

# 三、工具注解

## 3.1 @ToString

```java
@ToString                              // 生成 toString()
@ToString(exclude = "password")        // 排除字段
@ToString(of = {"id", "name"})         // 只包含指定字段
@ToString(callSuper = true)            // 包含父类字段
@ToString(includeFieldNames = false)   // 不显示字段名
```

## 3.2 @EqualsAndHashCode

```java
@EqualsAndHashCode                     // 生成 equals/hashCode
@EqualsAndHashCode(exclude = "id")     // 排除字段
@EqualsAndHashCode(of = {"name"})      // 只用指定字段
@EqualsAndHashCode(callSuper = true)   // 包含父类字段

// 注意：如果用了 @Data，子类建议加上：
@EqualsAndHashCode(callSuper = true)
public class Student extends Person { }
```

## 3.3 @Getter / @Setter

```java
// 类级别
@Getter
@Setter
public class User {
    private Long id;
    private String name;
}

// 字段级别
public class User {
    @Getter private Long id;         // 只生成 getter
    @Setter private String name;     // 只生成 setter
    @Getter(AccessLevel.NONE) private String secret;  // 不生成
}
```

## 3.4 @Value

不可变对象版的 `@Data`（所有字段 private final）。

```java
@Value
public class Point {
    int x;
    int y;
}
// 等价于：
// @Getter @ToString @EqualsAndHashCode @AllArgsConstructor @FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE)
```

---

# 四、其他实用注解

## 4.1 @NonNull

参数非空校验，自动生成 null 检查。

```java
public void process(@NonNull String name, @NonNull Long id) {
    // 自动生成：
    // if (name == null) throw new NullPointerException("name is marked non-null but is null");
    // if (id == null) throw new NullPointerException("id is marked non-null but is null");
}
```

## 4.2 @Cleanup

自动关闭资源（类似 try-with-resources）。

```java
public String readFile(String path) throws IOException {
    @Cleanup InputStream is = new FileInputStream(path);
    @Cleanup BufferedReader reader = new BufferedReader(new InputStreamReader(is));
    // 方法结束时自动关闭 is 和 reader
}
```

## 4.3 @SneakyThrows

抛出受检异常而不声明。

```java
@SneakyThrows
public void process() {
    // 可以直接抛出 IOException 而不需要 throws IOException
    Files.readAllBytes(Paths.get("file.txt"));
}
// 等价于在方法上加 throws，但更隐蔽
// 谨慎使用：可能隐藏异常
```

## 4.4 @Synchronized

方法级同步（比 synchronized 更安全，使用私有锁对象）。

```java
public class Counter {
    private int count;

    @Synchronized  // 使用名为 $lock 的私有对象作为锁
    public void increment() {
        count++;
    }

    @Synchronized("myLock")  // 使用自定义锁对象
    private Object myLock = new Object();
    public void decrement() {
        count--;
    }
}
```

## 4.5 @With

生成 with 方法（返回新对象，不修改原对象）。

```java
@With
@AllArgsConstructor
public class User {
    private Long id;
    private String name;
}

User user = new User(1L, "张三");
User updated = user.withName("李四");  // 返回新对象，user 不变
```

## 4.6 @Wither（已废弃，用 @With 替代）

---

# 五、继承注意事项

```java
// 父类
@Data
public class BaseEntity {
    private Long id;
    private Date createTime;
    private Long createUser;
}

// 子类 — 必须加 callSuper = true
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class User extends BaseEntity {
    private String name;
    private Integer age;
}
```

---

# 六、常见陷阱

```java
// 陷阱1：@Data 用于有继承的类
// 不加 callSuper = true 会导致 equals/hashCode 只看子类字段

// 陷阱2：@Data 用于需要作为 Map Key 的类
// 如果 equals/hashCode 只看部分字段，可能会出问题
// 建议：Map Key 用 @Value 或手动指定字段

// 陷阱3：Lombok 生成的代码不被某些工具识别
// IDE 需要安装 Lombok 插件
// 代码覆盖率工具可能需要特殊配置
```

---

# 七、注解速查表

| 注解 | 作用 | 适用场景 |
|------|------|---------|
| `@Data` | getter/setter/toString/equals/hashCode | 实体类、DTO |
| `@Value` | 不可变对象 | 值对象、配置类 |
| `@Slf4j` | 日志声明 | 所有需要日志的类 |
| `@NoArgsConstructor` | 无参构造 | JPA 实体、反序列化 |
| `@AllArgsConstructor` | 全参构造 | 全字段初始化 |
| `@RequiredArgsConstructor` | final/@NonNull 字段构造 | 依赖注入 |
| `@ToString` | toString 方法 | 调试、日志 |
| `@EqualsAndHashCode` | equals/hashCode | 比较、集合 |
| `@Getter` / `@Setter` | 精确控制 | 只需要部分访问器 |
| `@NonNull` | 非空校验 | 方法参数 |
| `@Cleanup` | 自动关闭资源 | I/O 操作 |
| `@SneakyThrows` | 抛出受检异常 | 简化异常处理 |
| `@Synchronized` | 线程安全 | 并发场景 |
| `@With` | 返回新对象 | 不可变修改 |
