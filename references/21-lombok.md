# Lombok 使用指南

Lombok 通过注解自动生成样板代码（getter/setter/构造方法/日志等），大幅减少重复代码量。

---

# 一、实体类注解

## 1.1 @Data

最常用注解，组合了 `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor`。

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
}
// 自动生成：所有 getter/setter/toString/equals/hashCode/无参构造
```

## 1.2 构造方法注解

- `@NoArgsConstructor` — 生成无参构造
- `@AllArgsConstructor` — 生成全参构造
- `@RequiredArgsConstructor` — 生成包含 `final` 字段 + `@NonNull` 字段的构造

```java
@RequiredArgsConstructor
public class UserService {
    @NonNull
    private final UserMapper userMapper;  // 出现在构造方法中
    private String config;                // 不会出现
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
        log.error("处理失败", e);
    }
}
// 等价于：private static final Logger log = LoggerFactory.getLogger(UserService.class);
```

## 2.2 其他日志注解

另有 `@Log4j2` / `@CommonsLog` / `@Flogger` / `@XSlf4j` 等，按项目日志框架选择即可。

---

# 三、工具注解

## 3.1 @ToString

| 参数 | 说明 |
|------|------|
| `exclude = "password"` | 排除指定字段 |
| `of = {"id", "name"}` | 仅包含指定字段 |
| `callSuper = true` | 包含父类字段 |
| `includeFieldNames = false` | 不显示字段名 |

## 3.2 @EqualsAndHashCode

| 参数 | 说明 |
|------|------|
| `exclude = "id"` | 排除指定字段 |
| `of = {"name"}` | 仅使用指定字段 |
| `callSuper = true` | 包含父类字段 |

> 注意：子类使用 `@Data` 时建议加 `@EqualsAndHashCode(callSuper = true)`。

## 3.3 @Getter / @Setter

```java
@Getter @Setter           // 类级别 — 为所有字段生成
public class User {
    @Getter private Long id;                        // 字段级别 — 只生成 getter
    @Getter(AccessLevel.NONE) private String secret; // 不生成
}
```

## 3.4 @Value

不可变对象（所有字段 `private final`），等价于 `@Getter + @ToString + @EqualsAndHashCode + @AllArgsConstructor + @FieldDefaults(makeFinal = true, level = PRIVATE)`。适用于值对象、配置类。

---

# 四、其他实用注解

**@NonNull** — 参数非空校验，为标注的参数自动生成 null 检查并抛出 `NullPointerException`。

```java
@Cleanup InputStream is = new FileInputStream(path);      // 方法结束自动关闭（类似 try-with-resources）
@Cleanup BufferedReader reader = new BufferedReader(...); // 按声明逆序关闭

@SneakyThrows                                              // 抛出受检异常而不声明 throws
public void process() { Files.readAllBytes(Paths.get("file.txt")); }

@Synchronized                                              // 方法级同步，使用私有锁对象
public void increment() { count++; }

@With                                                      // 生成 withXxx() 返回新对象
@AllArgsConstructor
public class User { private Long id; private String name; }
User updated = user.withName("李四");  // user 不变
```

> `@Wither` 已废弃，使用 `@With` 替代。

---

# 五、继承注意事项

```java
// 父类
@Data
public class BaseEntity {
    private Long id;
    private Date createTime;
}

// 子类 — 必须加 callSuper = true
@Data
@EqualsAndHashCode(callSuper = true)
@ToString(callSuper = true)
public class User extends BaseEntity {
    private String name;
}
```

---

# 六、常见陷阱

1. 有继承关系的类使用 `@Data` 必须加 `callSuper = true`，否则 equals/hashCode 只看子类字段
2. 作为 Map Key 的类建议用 `@Value` 或手动指定字段，避免 equals/hashCode 范围不一致
3. IDE 需安装 Lombok 插件；代码覆盖率工具可能需特殊配置

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
| `@Getter` / `@Setter` | 精确控制访问器 | 只需要部分字段 |
| `@NonNull` | 非空校验 | 方法参数 |
| `@Cleanup` | 自动关闭资源 | I/O 操作 |
| `@SneakyThrows` | 抛出受检异常 | 简化异常处理 |
| `@Synchronized` | 线程安全 | 并发场景 |
| `@With` | 返回新对象 | 不可变修改 |
