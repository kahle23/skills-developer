# 代码规范

本规范涵盖 Java 编码中最基础也最重要的方方面面：命名、结构、注释、异常、日志、空值处理。遵循这些规范可以让代码更易读、易维护、易协作。

---

# 一、命名规范

## 1.1 包命名

```java
// 全小写，点分隔，有意义的层次结构
com.example.project.module
com.example.project.module.service
com.example.project.module.controller
com.example.project.module.pojo.entity
com.example.project.module.pojo.param
com.example.project.module.pojo.query
com.example.project.module.pojo.result
```

## 1.2 类命名

```java
// 大驼峰，名词或名词短语，自解释
public class UserService { }            // 服务类
public class OrderController { }        // 控制器
public class PaymentRecord { }          // 实体/记录
public class UserQuery { }              // 查询对象
public class OrderAddParam { }          // 新增参数
public class OrderEditParam { }         // 编辑参数
public class OrderResult { }            // 结果对象
public class OrderSmpResult { }         // 简要结果（Smp = Simple）

// 接口：不强制 I 前缀
public interface PaymentService { }
public interface DataProcessor { }

// 抽象类
public abstract class BaseHandler { }
public abstract class AbstractDataProcessor<T> { }

// 异常类
public class BusinessException extends RuntimeException { }
public class DataNotFoundException extends BusinessException { }

// 枚举
public enum OrderStatus { }             // 不用 OrderStatusEnum
public enum Gender { }                  // 不用 GenderEnum
```

## 1.3 方法命名

```java
// 小驼峰，动词或动词短语
public User getUserById(Long id) { }
public List<User> queryUsers(UserQuery query) { }
public void saveUser(User user) { }
public void updateUser(User user) { }
public void deleteUser(Long id) { }
public boolean isValid(String input) { }
public boolean hasPermission(Long userId) { }
public boolean canEdit(Long orderId) { }

// 常用动词前缀
get / set           // 获取 / 设置
save / add          // 保存 / 新增
update / edit       // 更新 / 编辑
delete / remove     // 删除
query / find / select // 查询
count               // 计数
exist               // 是否存在
is / has / can      // 判断（返回 boolean）
convert / transform // 转换
build / create      // 构建 / 创建
validate / check    // 校验 / 检查
process / handle    // 处理
init                // 初始化
```

## 1.4 变量命名

```java
// 成员变量、局部变量：小驼峰
private String userName;
private int orderCount;
private boolean isDeleted;
private Date createTime;

// 常量：全大写下划线分隔
public static final String DEFAULT_CHARSET = "UTF-8";
public static final int MAX_RETRY_COUNT = 3;
public static final long ONE_DAY_MILLIS = 86400000L;

// 集合类型：复数或集合含义
List<User> users = new ArrayList<>();
Map<String, List<Order>> orderMap = new HashMap<>();
Set<Long> idSet = new HashSet<>();

// 布尔类型：is/has/can 前缀
boolean isActive = true;
boolean hasChildren = false;
boolean canEdit = true;

// 临时变量要有意义
// BAD
String s = user.getName();
int n = list.size();

// GOOD
String displayName = user.getName();
int totalCount = list.size();
```

## 1.5 避免的命名

```java
// BAD — 单字母（循环变量除外）
String a = getName();
void process(List<String> d) { }

// BAD — 含义模糊
String str = getData();
int num = getCount();
List<Object> data = query();

// BAD — 缩写过度
String usrNm = getUserName();
int ordCnt = getOrderCount();

// GOOD
String userName = getUserName();
int orderCount = getOrderCount();
List<User> users = queryUsers();
```

---

# 二、代码结构规范

## 2.1 类结构顺序

```java
public class UserService {
    // 1. 静态常量
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    private static final String PREFIX = "user_";

    // 2. 注入的依赖
    @Resource
    private UserMapper userMapper;
    @Resource
    private RoleService roleService;

    // 3. 普通成员变量
    private String configValue;

    // 4. 构造方法
    public UserService() { }

    // 5. 公开方法（业务方法）— 放在前面，便于快速理解类的职责
    public User getUserById(Long id) { }
    public void saveUser(User user) { }

    // 6. 受保护方法（供子类重写）
    protected void afterSave(User user) { }

    // 7. 私有方法（辅助方法）
    private void validateUser(User user) { }
    private String buildKey(Long id) { }

    // 8. getter/setter（如需要，通常 Lombok 自动生成）
}
```

## 2.2 方法长度与职责

```java
// 建议单个方法不超过 50 行
// 如果过长，说明职责过多，应拆分

// BAD — 一个方法做了太多事
public void processOrder(Order order) {
    // 200 行：校验 + 计算 + 保存 + 通知...
}

// GOOD — 拆分为多个小方法
public void processOrder(Order order) {
    validateOrder(order);
    calculateAmount(order);
    saveOrder(order);
    sendNotification(order);
}

private void validateOrder(Order order) { }
private void calculateAmount(Order order) { }
private void saveOrder(Order order) { }
private void sendNotification(Order order) { }
```

## 2.3 方法参数

```java
// 参数不超过 3~4 个，超过则封装为对象
// BAD
public User createUser(String name, int age, String email,
                       String phone, String address, String city) { }

// GOOD
public User createUser(UserCreateParam param) { }

// 参数顺序建议：必需参数在前，可选参数在后
public void process(String name, Long id, String callback) { }
```

## 2.4 控制流

```java
// 早返回（Guard Clause）— 减少嵌套
// BAD
public String process(User user) {
    if (user != null) {
        if (user.isActive()) {
            if (user.hasPermission()) {
                return doSomething(user);
            } else {
                throw new SecurityException("无权限");
            }
        } else {
            throw new IllegalStateException("未激活");
        }
    } else {
        throw new IllegalArgumentException("用户为空");
    }
}

// GOOD — 卫语句，扁平结构
public String process(User user) {
    if (user == null) {
        throw new IllegalArgumentException("用户不能为空");
    }
    if (!user.isActive()) {
        throw new IllegalStateException("用户未激活");
    }
    if (!user.hasPermission()) {
        throw new SecurityException("用户无权限");
    }
    return doSomething(user);
}
```

## 2.5 代码块之间加空行

```java
public void process(Long userId) {
    // 查询用户
    User user = userMapper.selectById(userId);       // ← 逻辑块1
    notNull(user, "用户不存在");                       //    内部不空行

    // 查询角色
    List<Role> roles = roleService.queryByUserId(userId); // ← 逻辑块2
    notEmpty(roles, "角色为空");                           //    内部不空行

    // 组装结果
    UserResult result = new UserResult();
    result.setUser(user);
    result.setRoles(roles);
    return result;
}
```

---

# 三、注释与文档规范

## 3.1 类注释

```java
/**
 * 用户服务类，处理用户相关的业务逻辑.
 *
 * @author 张三
 * @since 2024-01-01
 */
public class UserService {
```

## 3.2 方法注释

```java
/**
 * 根据用户ID查询用户详情（包含角色信息）.
 *
 * @param userId 用户ID，不能为空
 * @return 用户详情对象，不存在返回null
 * @throws BusinessException 用户不存在时抛出
 */
public UserDetail getUserDetail(Long userId) {
```

## 3.3 常量注释

```java
/**
 * 雪花ID的名称.
 */
public static final String SNOWFLAKE = "snowflake";

/**
 * 系统用户ID的名称.
 */
public static final String SYS_USER_ID = "sys-user-id";
```

## 3.4 行内注释

```java
// 好的注释 — 解释"为什么"，而不是"是什么"
// 因为下游系统只支持32位字符串，所以需要截断
String truncated = StringUtils.truncate(name, 32);

// 批量处理，每次100条，避免单次数据量过大导致OOM
Lists.partition(allIds, 100).forEach(this::processBatch);

// BAD — 废话注释
// 设置名称
user.setName(name);

// 获取列表长度
int size = list.size();
```

## 3.5 TODO/FIXME 标记

```java
// TODO: 2024-06-01 后续需要支持批量导入
// FIXME: 偶发并发问题，需要加锁
// HACK: 临时方案，等下游修复后删除
```

---

# 四、异常处理规范

## 4.1 自定义业务异常

```java
public class BusinessException extends RuntimeException {
    private final String code;

    public BusinessException(String message) {
        super(message);
        this.code = "BUSINESS_ERROR";
    }

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}
```

## 4.2 异常处理原则

```java
// 1. 不要捕获通用异常然后吞掉
// BAD
try {
    doSomething();
} catch (Exception e) {
    // 什么都不做 — 最差实践
}

// 2. 捕获具体异常，记录日志，按需抛出
// GOOD
try {
    doSomething();
} catch (IOException e) {
    log.error("文件读取失败，路径: {}", filePath, e);
    throw new BusinessException("FILE_READ_ERROR", "文件处理失败", e);
}

// 3. 异常信息要具体，包含上下文
// BAD
throw new RuntimeException("error");

// GOOD
throw new BusinessException("USER_NOT_FOUND", "用户不存在，ID: " + userId);

// 4. 使用断言式校验（方法入口快速失败）
public void process(Long userId, String name) {
    notNull(userId, "用户ID不能为空");
    notBlank(name, "名称不能为空");
    // 业务逻辑...
}

// 5. finally 确保资源释放（Java 6）
InputStream is = null;
try {
    is = new FileInputStream(path);
    // ...
} finally {
    if (is != null) {
        try { is.close(); } catch (IOException ignored) { }
    }
}

// Java 7+ 使用 try-with-resources
try (InputStream is = new FileInputStream(path);
     BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
    // ...
}
```

## 4.3 异常分类

```
Throwable
├── Error（不捕取，JVM级错误）
│   ├── OutOfMemoryError
│   └── StackOverflowError
│
└── Exception
    ├── RuntimeException（非受检异常）
    │   ├── IllegalArgumentException    ← 参数非法
    │   ├── IllegalStateException       ← 状态非法
    │   ├── NullPointerException        ← 空指针
    │   └── BusinessException           ← 业务异常（自定义）
    │
    └── 受检异常
        ├── IOException
        ├── SQLException
        └── ...
```

---

# 五、日志规范

## 5.1 日志声明

```java
// 推荐使用 @Slf4j（Lombok），等价于下面的手动声明
@Slf4j
public class UserService { }

// 手动声明方式
private static final Logger log = LoggerFactory.getLogger(UserService.class);
```

## 5.2 日志级别使用

```java
// TRACE — 最细粒度，通常只在开发时使用
log.trace("进入方法，参数: {}", param);

// DEBUG — 调试信息，生产环境通常关闭
log.debug("查询条件: {}", query);
log.debug("SQL执行耗时: {}ms", elapsed);

// INFO — 关键业务节点，生产环境需要
log.info("用户登录成功，ID: {}，IP: {}", userId, ip);
log.info("订单创建成功，订单号: {}", orderNo);
log.info("批量导入完成，总数: {}，成功: {}，失败: {}", total, success, fail);

// WARN — 不影响主流程但需要关注的异常情况
log.warn("配置缺失，使用默认值: {}", defaultValue);
log.warn("接口调用超时，已重试第{}次", retryCount);

// ERROR — 影响功能的错误，必须记录异常堆栈
log.error("保存用户失败，ID: {}", userId, e);
log.error("数据库连接失败", e);
```

## 5.3 日志格式

```java
// 使用占位符，不要字符串拼接
// BAD
log.info("用户ID:" + userId + "，名称:" + userName);
log.debug("查询结果：" + JSONUtil.toJsonStr(result));

// GOOD
log.info("用户ID: {}，名称: {}", userId, userName);
log.debug("查询结果: {}", result);

// ERROR 级别必须带异常对象作为最后一个参数
// BAD
log.error("保存失败: " + e.getMessage());

// GOOD
log.error("保存失败，用户ID: {}", userId, e);
```

## 5.4 日志中的敏感信息

```java
// 不要打印密码、Token、身份证号等敏感信息
// BAD
log.info("用户登录，账号: {}，密码: {}", account, password);

// GOOD
log.info("用户登录，账号: {}", account);
```

---

# 六、空值安全处理

## 6.1 参数校验前置

```java
// 方法入口处校验，快速失败
public void processUser(Long userId, String name) {
    if (userId == null) {
        throw new IllegalArgumentException("用户ID不能为空");
    }
    if (name == null || name.trim().isEmpty()) {
        throw new IllegalArgumentException("名称不能为空");
    }
    // 业务逻辑...
}

// 使用工具方法简化
public void processUser(Long userId, String name) {
    notNull(userId, "用户ID不能为空");
    notBlank(name, "名称不能为空");
    // 业务逻辑...
}
```

## 6.2 返回空集合而非 null

```java
// BAD
public List<User> getUsers(Long deptId) {
    if (deptId == null) {
        return null;  // 调用方必须判空
    }
    return userMapper.selectByDeptId(deptId);
}

// GOOD
public List<User> getUsers(Long deptId) {
    if (deptId == null) {
        return Collections.emptyList();  // 调用方可以直接遍历
    }
    return userMapper.selectByDeptId(deptId);
}
```

## 6.3 Optional 使用（Java 8+）

```java
// 返回值可能为空时使用 Optional
public Optional<User> findUser(Long id) {
    return Optional.ofNullable(userMapper.selectById(id));
}

// 使用 Optional
findUser(userId)
    .map(User::getName)
    .orElse("未知用户");

findUser(userId)
    .ifPresent(user -> sendEmail(user.getEmail()));

findUser(userId)
    .orElseThrow(() -> new BusinessException("用户不存在"));
```

## 6.4 集合/数组判空

```java
// 访问集合前判空
if (CollUtil.isNotEmpty(list)) {
    list.forEach(item -> process(item));
}

// Map 取值判空
String value = map.get(key);
if (value != null) {
    // 使用 value
}

// 或使用 getOrDefault
String value = map.getOrDefault(key, "默认值");

// 数组判空
if (array != null && array.length > 0) {
    // 使用 array
}
```

## 6.5 链式调用中的空值保护

```java
// BAD — 每层都可能 NPE
String city = user.getAddress().getCity().getName();

// GOOD — 逐层判空
String city = null;
if (user != null && user.getAddress() != null
        && user.getAddress().getCity() != null) {
    city = user.getAddress().getCity().getName();
}

// BETTER — 使用 Optional（Java 8+）
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .map(City::getName)
    .orElse(null);
```

---

# 七、编码风格速查表

| 规则 | BAD | GOOD |
|------|-----|------|
| 命名 | `String s`, `int n` | `String userName`, `int count` |
| 常量 | `public int max = 100` | `public static final int MAX = 100` |
| 方法长度 | 200行一个方法 | 拆分为多个小方法 |
| 参数数量 | 7个参数 | 封装为 Param 对象 |
| 返回空值 | `return null` | `return Collections.emptyList()` |
| 日志拼接 | `"id:" + id` | `"id: {}", id` |
| 异常吞掉 | `catch (Exception e) {}` | `log.error("msg", e)` |
| 注释废话 | `// 设置名称` | （不写，命名已自解释） |
| 嵌套层数 | 4层 if 嵌套 | 卫语句提前返回 |
