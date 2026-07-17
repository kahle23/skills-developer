# 设计模式实践

本文档收录 Java 日常开发中最常用的设计模式，以实用为主，避免过度设计。

---

# 一、建造者（Builder）

> 对象字段多、构造参数复杂。推荐 Lombok `@Builder`，手写参考：

```java
public class UserQuery {
    private String name;    private Integer minAge;
    private Integer maxAge; private Integer pageNum = 1;
    private Integer pageSize = 20;

    private UserQuery() {}
    public static Builder builder() { return new Builder(); }

    public static class Builder {
        private final UserQuery q = new UserQuery();
        public Builder name(String v)      { q.name = v;     return this; }
        public Builder minAge(Integer v)   { q.minAge = v;   return this; }
        public Builder maxAge(Integer v)   { q.maxAge = v;   return this; }
        public Builder pageNum(Integer v)  { q.pageNum = v;  return this; }
        public Builder pageSize(Integer v) { q.pageSize = v; return this; }
        public UserQuery build() { return q; }
    }
}
// UserQuery.builder().name("张三").minAge(18).pageSize(20).build();
```

---

# 二、策略（Strategy）

> 同一操作多种实现，按类型切换。

```java
// 接口
public interface ExportStrategy {
    String getFileExtension();
    byte[] export(List<?> data);
}
// Excel/Csv/PdfExportStrategy 各自实现 export()，注册到工厂

// 工厂
public class ExportStrategyFactory {
    private static final Map<String, ExportStrategy> MAP = new HashMap<>();
    static { MAP.put("excel", new ExcelExportStrategy()); /* ... */ }

    public static ExportStrategy get(String type) {
        ExportStrategy s = MAP.get(type.toLowerCase());
        if (s == null) throw new IllegalArgumentException("不支持: " + type);
        return s;
    }
}
// 使用：ExportStrategyFactory.get(type).export(data);
```

---

# 三、模板方法（Template Method）

> 流程固定，某些步骤子类自定义。

```java
public abstract class DataImportTemplate<T> {
    public final ImportResult execute(InputStream input) {
        List<T> list = parseData(readData(input));
        validate(list);
        return buildResult(list.size(), saveData(list));
    }

    protected abstract List<T> parseData(List<String> raw);
    protected abstract int saveData(List<T> list);
    protected void validate(List<T> list) { }                // 钩子——可选重写
    protected List<String> readData(InputStream in) { /* */ } // 通用方法
}

public class UserImport extends DataImportTemplate<User> {
    @Override protected List<User> parseData(List<String> raw) { /* 解析 */ }
    @Override protected int saveData(List<User> list) { return mapper.batchInsert(list); }
    @Override protected void validate(List<User> list) { /* 名称非空、邮箱格式 */ }
}
```

---

# 四、观察者（Event）

> 操作完成后触发多个后续动作。Spring 事件方案：

```java
// 事件
public class OrderCreatedEvent {
    private final Long orderId, userId;
    public OrderCreatedEvent(Long oid, Long uid) { orderId = oid; userId = uid; }
    // getter...
}

// 监听器（多个监听器处理不同逻辑：通知、扣库存、日志）
@Component
public class OrderEventListener {
    @EventListener
    public void handle(OrderCreatedEvent e) { sendNotification(e.getUserId()); }
}

// 发布
@Service
public class OrderService {
    @Autowired private ApplicationEventPublisher publisher;
    public Long create(Order o) {
        mapper.insert(o);
        publisher.publishEvent(new OrderCreatedEvent(o.getId(), o.getUserId()));
        return o.getId();
    }
}
```

---

# 五、责任链（Chain of Responsibility）

> 请求经多个处理者，每个决定处理或传递。

```java
public abstract class Handler {
    private Handler next;
    public Handler setNext(Handler h) { this.next = h; return h; }

    public void handle(Request req) {
        if (canHandle(req)) doHandle(req);
        else if (next != null) next.handle(req);
        else throw new BusinessException("无处理器");
    }
    protected abstract boolean canHandle(Request req);
    protected abstract void doHandle(Request req);
}
// AuthHandler/RateLimitHandler/BusinessHandler 各自实现 canHandle+doHandle
// new AuthHandler().setNext(new RateLimitHandler()).setNext(new BusinessHandler());
```

---

# 六、工厂（Factory）

> 推荐 Spring 自动注入，避免 switch-case（违反开闭原则）。

```java
@Component
public class PaymentFactory {
    @Autowired private List<PaymentService> services;  // 自动注入所有实现
    private final Map<String, PaymentService> map = new HashMap<>();

    @PostConstruct
    void init() { services.forEach(s -> map.put(s.getType(), s)); }

    public PaymentService get(String type) {
        PaymentService s = map.get(type);
        if (s == null) throw new IllegalArgumentException("不支持: " + type);
        return s;
    }
}
```

---

# 七、单例（Singleton）

```java
// 推荐：枚举（最简单、线程安全、防反射）
public enum Singleton { INSTANCE; public void doSomething() { } }
// 次选：静态内部类 / DCL
```

---

# 八、适配器（Adapter）

> 将不兼容的接口包装为兼容接口。

```java
// 旧接口: getUserById(String) / queryUsers(Map)
// 新接口: getUserById(Long) / queryUsers(UserQuery)

public class UserServiceAdapter implements UserService {
    @Autowired private OldUserService old;

    @Override public UserDTO getUserById(Long id) {
        return convert(old.getUserById(String.valueOf(id)));
    }
    @Override public PageResult<UserDTO> queryUsers(UserQuery q) {
        return PageResult.of(old.queryUsers(convertToParams(q))
            .stream().map(this::convert).collect(Collectors.toList()));
    }
}
```

---

# 九、速查表

| 问题 | 模式 | 关键特征 |
|------|------|---------|
| 对象构造参数太多 | 建造者 | Builder / Lombok `@Builder` |
| 同一操作多种实现 | 策略 | 接口 + Map 分发 |
| 流程固定步骤可变 | 模板方法 | 抽象类 + 钩子 |
| 操作后触发多动作 | 观察者 | Spring `@EventListener` |
| 请求需多层处理 | 责任链 | 链式传递 |
| 按条件创建对象 | 工厂 | Spring 注入所有实现 |
| 全局唯一实例 | 单例 | 枚举最优先 |
| 接口不兼容 | 适配器 | 包装旧接口 |
