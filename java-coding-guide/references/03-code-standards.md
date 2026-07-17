# 代码规范

涵盖 Java 编码核心规范：命名、结构、注释、异常、日志、空值处理。

---

# 一、命名规范

## 1.1 包命名

全小写，点分隔，有意义。如：`com.example.project.module.service`、`com.example.project.module.pojo.param`

## 1.2 类命名

大驼峰，名词或名词短语。接口不加 `I` 前缀，枚举不加 `Enum` 后缀。

```java
public class UserService { }
public class OrderController { }
public class OrderAddParam { }        // 参数类：操作+类型+Param
public class OrderResult { }          // 结果类：类型+Result
public interface PaymentService { }   // 接口不加 I 前缀
public enum OrderStatus { }           // 枚举不加 Enum 后缀
public class BusinessException extends RuntimeException { }
```

## 1.3 方法命名

小驼峰，动词前缀。常用前缀：

| 前缀 | 用途 |
|------|------|
| get / set | 获取 / 设置 |
| save / update / delete | 增改删 |
| query / find | 查询 |
| count / exist | 计数 / 存在判断 |
| is / has / can | boolean 判断 |
| convert / build / validate | 转换 / 构建 / 校验 |

## 1.4 变量命名

- 成员变量、局部变量：小驼峰（`userName`、`orderCount`）
- 常量：全大写下划线分隔（`DEFAULT_CHARSET`、`MAX_RETRY_COUNT`）
- 集合类型用复数或集合含义（`users`、`orderMap`、`idSet`）
- 布尔类型用 is/has/can 前缀（`isActive`、`hasChildren`）

## 1.5 避免的命名

- 不用单字母（循环变量 `i/j` 除外）
- 不用含义模糊的名称（`str`、`num`、`data`）
- 不用过度缩写（`usrNm` → `userName`）

---

# 二、代码结构规范

## 2.1 类结构顺序

```java
public class UserService {
    // 1. 静态常量
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    // 2. 注入的依赖
    @Resource
    private UserMapper userMapper;
    // 3. 成员变量
    // 4. 构造方法
    // 5. 公开方法（业务方法）
    // 6. 受保护方法（供子类重写）
    // 7. 私有方法（辅助方法）
    // 8. getter/setter（Lombok 自动生成）
}
```

## 2.2 方法长度与职责

单个方法建议不超过 50 行，职责过多时拆分为多个小方法。

```java
// GOOD — 拆分为职责单一的小方法
public void processOrder(Order order) {
    validateOrder(order);
    calculateAmount(order);
    saveOrder(order);
    sendNotification(order);
}
```

## 2.3 方法参数

参数不超过 3~4 个，超过则封装为对象。参数顺序：必需参数在前，可选参数在后。

## 2.4 控制流

使用卫语句（Guard Clause）提前返回，减少嵌套。

```java
public String process(User user) {
    if (user == null) { throw new IllegalArgumentException("用户不能为空"); }
    if (!user.isActive()) { throw new IllegalStateException("用户未激活"); }
    if (!user.hasPermission()) { throw new SecurityException("用户无权限"); }
    return doSomething(user);
}
```

## 2.5 空行规则

逻辑块之间加空行，逻辑块内部不空行。

```java
public void process(Long userId) {
    User user = userMapper.selectById(userId);
    notNull(user, "用户不存在");

    List<Role> roles = roleService.queryByUserId(userId);
    notEmpty(roles, "角色为空");

    UserResult result = new UserResult();
    result.setUser(user);
    result.setRoles(roles);
}
```

---

# 三、注释规范

- **类注释**：说明类的职责，标注 `@author`、`@since`
- **方法注释**：说明用途，标注 `@param`、`@return`、`@throws`
- **常量注释**：特殊常量需说明含义
- **行内注释**：解释"为什么"，不解释"是什么"
- **TODO/FIXME**：标注日期和原因

```java
// 因为下游系统只支持32位字符串，所以需要截断
String truncated = StringUtils.truncate(name, 32);

// BAD — 废话注释
// 设置名称
user.setName(name);
```

---

# 四、异常处理规范

## 4.1 自定义业务异常

```java
public class BusinessException extends RuntimeException {
    private final String code;

    public BusinessException(String code, String message) {
        super(message);
        this.code = code;
    }

    public BusinessException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    public String getCode() { return code; }
}
```

## 4.2 异常处理原则

- 不捕获通用异常后吞掉（`catch (Exception e) {}` 禁止）
- 捕获具体异常，记录日志，按需包装抛出
- 异常信息要具体，包含上下文
- 使用断言式校验（`notNull(userId, "用户ID不能为空")`）
- 资源释放使用 try-with-resources

```java
try (InputStream is = new FileInputStream(path);
     BufferedReader reader = new BufferedReader(new InputStreamReader(is))) {
    // ...
} catch (IOException e) {
    log.error("文件读取失败，路径: {}", filePath, e);
    throw new BusinessException("FILE_READ_ERROR", "文件处理失败", e);
}
```

## 4.3 异常分类

- **Error**：JVM 级错误（OOM、StackOverflow），不捕获
- **RuntimeException**（非受检）：参数非法、状态非法、空指针、业务异常（自定义 `BusinessException`）
- **受检异常**：IOException、SQLException 等，必须处理或声明

---

# 五、日志规范

- 声明：优先用 `@Slf4j`（Lombok）
- 级别：DEBUG 调试信息 / INFO 关键业务节点 / WARN 非主流程异常 / ERROR 功能错误
- 使用占位符 `{}`，不字符串拼接
- ERROR 级别必须带异常对象作为最后一个参数
- 不打印密码、Token、身份证号等敏感信息

```java
log.info("订单创建成功，订单号: {}", orderNo);
log.error("保存用户失败，ID: {}", userId, e);  // 异常对象作为最后参数

// BAD
log.info("用户ID:" + userId);                    // 字符串拼接
log.info("账号: {}，密码: {}", account, password); // 敏感信息
```

---

# 六、空值安全处理

## 6.1 参数校验前置

方法入口处用工具方法快速失败：

```java
public void processUser(Long userId, String name) {
    notNull(userId, "用户ID不能为空");
    notBlank(name, "名称不能为空");
    // 业务逻辑...
}
```

## 6.2 返回空集合而非 null

```java
public List<User> getUsers(Long deptId) {
    if (deptId == null) {
        return Collections.emptyList();
    }
    return userMapper.selectByDeptId(deptId);
}
```

## 6.3 Optional 使用

```java
// 返回值可能为空
public Optional<User> findUser(Long id) {
    return Optional.ofNullable(userMapper.selectById(id));
}

// 取值
findUser(userId).map(User::getName).orElse("未知用户");
findUser(userId).orElseThrow(() -> new BusinessException("用户不存在"));
```

## 6.4 集合/数组判空

使用工具类（`CollUtil.isNotEmpty`/`isEmpty`），详见 `04-collection-and-stream.md`。

## 6.5 链式空值保护

```java
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .map(City::getName)
    .orElse(null);
```
