# 设计模式实践

本文档收录 Java 日常开发中最常用的设计模式，以实用为主，避免过度设计。

---

# 一、建造者模式（Builder）

适用场景：对象字段多、构造参数复杂、需要灵活组合。

```java
public class UserQuery {
    private String name;
    private Integer minAge;
    private Integer maxAge;
    private String deptName;
    private List<Long> idList;
    private Integer pageNum = 1;
    private Integer pageSize = 20;

    // 私有构造
    private UserQuery() {}

    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private final UserQuery q = new UserQuery();

        public Builder name(String name)       { q.name = name; return this; }
        public Builder minAge(Integer minAge)   { q.minAge = minAge; return this; }
        public Builder maxAge(Integer maxAge)   { q.maxAge = maxAge; return this; }
        public Builder deptName(String deptName) { q.deptName = deptName; return this; }
        public Builder idList(List<Long> idList) { q.idList = idList; return this; }
        public Builder pageNum(Integer pageNum) { q.pageNum = pageNum; return this; }
        public Builder pageSize(Integer pageSize) { q.pageSize = pageSize; return this; }

        public UserQuery build() { return q; }
    }
}

// 使用
UserQuery query = UserQuery.builder()
    .name("张三")
    .minAge(18)
    .maxAge(60)
    .pageNum(1)
    .pageSize(20)
    .build();
```

---

# 二、策略模式（Strategy）

适用场景：同一操作有多种实现，需要根据条件切换。

```java
// 1. 定义策略接口
public interface ExportStrategy {
    String getFileExtension();
    byte[] export(List<?> data);
}

// 2. 具体策略
public class ExcelExportStrategy implements ExportStrategy {
    @Override
    public String getFileExtension() { return ".xlsx"; }

    @Override
    public byte[] export(List<?> data) {
        // EasyExcel 导出逻辑
        return excelBytes;
    }
}

public class CsvExportStrategy implements ExportStrategy {
    @Override
    public String getFileExtension() { return ".csv"; }

    @Override
    public byte[] export(List<?> data) {
        // CSV 导出逻辑
        return csvBytes;
    }
}

public class PdfExportStrategy implements ExportStrategy {
    @Override
    public String getFileExtension() { return ".pdf"; }

    @Override
    public byte[] export(List<?> data) {
        // PDF 导出逻辑
        return pdfBytes;
    }
}

// 3. 策略工厂
public class ExportStrategyFactory {
    private static final Map<String, ExportStrategy> STRATEGY_MAP = new HashMap<>();

    static {
        STRATEGY_MAP.put("excel", new ExcelExportStrategy());
        STRATEGY_MAP.put("csv", new CsvExportStrategy());
        STRATEGY_MAP.put("pdf", new PdfExportStrategy());
    }

    public static ExportStrategy getStrategy(String type) {
        ExportStrategy strategy = STRATEGY_MAP.get(type.toLowerCase());
        if (strategy == null) {
            throw new IllegalArgumentException("不支持的导出类型: " + type);
        }
        return strategy;
    }
}

// 4. 使用
public class ExportService {
    public byte[] export(String type, List<?> data) {
        ExportStrategy strategy = ExportStrategyFactory.getStrategy(type);
        return strategy.export(data);
    }
}
```

---

# 三、模板方法模式（Template Method）

适用场景：流程固定，但某些步骤由子类自定义。

```java
// 1. 抽象模板类
public abstract class DataImportTemplate<T> {

    // 模板方法 — 定义流程骨架
    public final ImportResult execute(InputStream input) {
        // 1. 读取数据
        List<String> rawData = readData(input);

        // 2. 解析数据
        List<T> dataList = parseData(rawData);

        // 3. 校验数据
        validate(dataList);

        // 4. 保存数据（子类实现）
        int saved = saveData(dataList);

        // 5. 返回结果
        ImportResult result = new ImportResult();
        result.setTotal(dataList.size());
        result.setSuccess(saved);
        return result;
    }

    // 抽象方法 — 子类必须实现
    protected abstract List<T> parseData(List<String> rawData);
    protected abstract int saveData(List<T> dataList);

    // 钩子方法 — 子类可选重写
    protected void validate(List<T> dataList) {
        // 默认不做校验
    }

    // 通用方法 — 子类直接使用
    protected List<String> readData(InputStream input) {
        // 通用的 Excel/CSV 读取逻辑
    }
}

// 2. 具体实现
public class UserImportTemplate extends DataImportTemplate<User> {

    @Override
    protected List<User> parseData(List<String> rawData) {
        // 将原始数据解析为 User 对象
    }

    @Override
    protected int saveData(List<User> dataList) {
        return userMapper.batchInsert(dataList);
    }

    @Override
    protected void validate(List<User> dataList) {
        // 校验用户数据：名称不为空、邮箱格式正确等
    }
}
```

---

# 四、观察者模式（Observer / Event）

适用场景：一个操作完成后需要触发多个后续动作，且这些动作可能变化。

```java
// 1. 事件定义
public class OrderCreatedEvent {
    private final Long orderId;
    private final Long userId;

    public OrderCreatedEvent(Long orderId, Long userId) {
        this.orderId = orderId;
        this.userId = userId;
    }
    // getter...
}

// 2. 事件监听器（Spring 风格）
@Component
public class OrderEventListener {

    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // 发送通知
        sendNotification(event.getUserId());
    }
}

@Component
public class InventoryEventListener {

    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // 扣减库存
        reduceStock(event.getOrderId());
    }
}

// 3. 发布事件
@Service
public class OrderService {

    @Autowired
    private ApplicationEventPublisher eventPublisher;

    public Long createOrder(Order order) {
        orderMapper.insert(order);
        // 发布事件，所有监听器自动执行
        eventPublisher.publishEvent(new OrderCreatedEvent(order.getId(), order.getUserId()));
        return order.getId();
    }
}
```

---

# 五、责任链模式（Chain of Responsibility）

适用场景：请求需要经过多个处理者，每个处理者决定是否处理或传递给下一个。

```java
// 1. 处理者接口
public abstract class Handler {
    private Handler next;

    public Handler setNext(Handler next) {
        this.next = next;
        return next;
    }

    public void handle(Request request) {
        if (canHandle(request)) {
            doHandle(request);
        } else if (next != null) {
            next.handle(request);
        } else {
            throw new BusinessException("无处理器可处理该请求");
        }
    }

    protected abstract boolean canHandle(Request request);
    protected abstract void doHandle(Request request);
}

// 2. 具体处理者
public class AuthHandler extends Handler {
    @Override
    protected boolean canHandle(Request request) { return true; }

    @Override
    protected void doHandle(Request request) {
        // 认证逻辑
    }
}

public class RateLimitHandler extends Handler {
    @Override
    protected boolean canHandle(Request request) { return true; }

    @Override
    protected void doHandle(Request request) {
        // 限流逻辑
    }
}

public class BusinessHandler extends Handler {
    @Override
    protected boolean canHandle(Request request) { return true; }

    @Override
    protected void doHandle(Request request) {
        // 业务逻辑
    }
}

// 3. 构建链条
Handler chain = new AuthHandler();
chain.setNext(new RateLimitHandler())
     .setNext(new BusinessHandler());

// 4. 执行
chain.handle(request);
```

---

# 六、工厂模式（Factory）

```java
// 简单工厂
public class PaymentFactory {
    public static PaymentService create(String type) {
        switch (type.toLowerCase()) {
            case "alipay":  return new AlipayService();
            case "wechat":  return new WechatPayService();
            case "bank":    return new BankPayService();
            default: throw new IllegalArgumentException("不支持的支付方式: " + type);
        }
    }
}

// 使用 Spring 的工厂模式（推荐）
@Component
public class PaymentFactory {
    @Autowired
    private List<PaymentService> paymentServices;  // Spring 自动注入所有实现

    private final Map<String, PaymentService> serviceMap = new HashMap<>();

    @PostConstruct
    public void init() {
        for (PaymentService service : paymentServices) {
            serviceMap.put(service.getType(), service);
        }
    }

    public PaymentService getService(String type) {
        PaymentService service = serviceMap.get(type);
        if (service == null) {
            throw new IllegalArgumentException("不支持的支付方式: " + type);
        }
        return service;
    }
}
```

---

# 七、单例模式（Singleton）

```java
// 方式1：枚举（最推荐）
public enum Singleton {
    INSTANCE;

    public void doSomething() { }
}

// 方式2：静态内部类
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

// 方式3：双重检查锁（DCL）
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

---

# 八、适配器模式（Adapter）

适用场景：将不兼容的接口转换为期望的接口。

```java
// 旧接口
public interface OldUserService {
    UserInfo getUserById(String id);
    List<UserInfo> queryUsers(Map<String, Object> params);
}

// 新接口
public interface UserService {
    UserDTO getUserById(Long id);
    PageResult<UserDTO> queryUsers(UserQuery query);
}

// 适配器
public class UserServiceAdapter implements UserService {
    @Autowired
    private OldUserService oldUserService;

    @Override
    public UserDTO getUserById(Long id) {
        UserInfo info = oldUserService.getUserById(String.valueOf(id));
        return convertToDTO(info);
    }

    @Override
    public PageResult<UserDTO> queryUsers(UserQuery query) {
        Map<String, Object> params = convertToParams(query);
        List<UserInfo> list = oldUserService.queryUsers(params);
        return PageResult.of(list.stream().map(this::convertToDTO).collect(Collectors.toList()));
    }

    private UserDTO convertToDTO(UserInfo info) { }
    private Map<String, Object> convertToParams(UserQuery query) { }
}
```

---

# 九、模式选择速查

| 问题 | 模式 | 关键特征 |
|------|------|---------|
| 对象构造参数太多 | 建造者 | `Builder` 内部类 |
| 同一操作多种实现 | 策略 | 接口 + Map 分发 |
| 流程固定，步骤可变 | 模板方法 | 抽象类 + 钩子方法 |
| 操作完成后触发多个后续动作 | 观察者/事件 | `Event` + `Listener` |
| 请求需要经过多层处理 | 责任链 | 链式传递 |
| 根据条件创建不同对象 | 工厂 | `create(type)` |
| 全局唯一实例 | 单例 | 枚举 / 静态内部类 |
| 接口不兼容 | 适配器 | 包装旧接口为新接口 |
