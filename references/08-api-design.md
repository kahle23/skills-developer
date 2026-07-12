# API 设计规范

本文档定义 RESTful API 的设计规范，包括 URL 命名、HTTP 方法、请求参数、返回值结构、分页约定、错误码体系等。

---

# 一、URL 命名规范

## 1.1 基本规则

```
/api/{版本}/{模块}/{资源}
```

```java
// 使用小写字母和连字符（kebab-case），不用驼峰和下划线
// BAD
GET /api/v1/getUserInfo
GET /api/v1/user_info
GET /api/v1/UserInfo

// GOOD
GET /api/v1/users
GET /api/v1/users/{id}
GET /api/v1/work-orders
GET /api/v1/work-orders/{id}
```

## 1.2 资源用复数名词

```java
// BAD — 动词、单数
GET /api/v1/user/getAll
GET /api/v1/user/{id}
POST /api/v1/user/create

// GOOD — 复数名词，HTTP 方法表达动作
GET    /api/v1/users           // 查询列表
GET    /api/v1/users/{id}      // 查询单个
POST   /api/v1/users           // 新增
PUT    /api/v1/users/{id}      // 全量更新
PATCH  /api/v1/users/{id}      // 部分更新
DELETE /api/v1/users/{id}      // 删除
```

## 1.3 子资源关系

```java
// 用户的订单
GET /api/v1/users/{userId}/orders

// 订单的明细
GET /api/v1/orders/{orderId}/items
POST /api/v1/orders/{orderId}/items

// 嵌套不超过两层，超过则用查询参数
// BAD — 嵌套过深
GET /api/v1/users/{userId}/orders/{orderId}/items/{itemId}/reviews

// GOOD — 扁平化
GET /api/v1/reviews?userId=1&orderId=2&itemId=3
```

## 1.4 非 CRUD 操作的 URL 设计

```java
// 用动词表达非标准操作
POST /api/v1/work-orders/{id}/submit        // 提交工单
POST /api/v1/work-orders/{id}/approve       // 审批工单
POST /api/v1/work-orders/{id}/cancel        // 取消工单
POST /api/v1/users/{id}/reset-password      // 重置密码
POST /api/v1/orders/{id}/refund             // 退款

// 或者用状态变更
PATCH /api/v1/work-orders/{id}
Body: { "status": "SUBMITTED" }
```

---

# 二、HTTP 方法使用

| 方法 | 语义 | 幂等 | 安全 | 典型场景 |
|------|------|------|------|---------|
| GET | 查询 | ✅ | ✅ | 获取资源列表/详情 |
| POST | 创建 | ❌ | ❌ | 新增资源、触发操作 |
| PUT | 全量替换 | ✅ | ❌ | 更新整个资源 |
| PATCH | 部分更新 | ✅ | ❌ | 更新资源的某些字段 |
| DELETE | 删除 | ✅ | ❌ | 删除资源 |

```java
// 幂等性说明
// PUT 更新多次，结果一样 → 幂等
// POST 创建多次，会创建多条记录 → 不幂等
// DELETE 删除多次，第一次之后都是空操作 → 幂等
```

---

# 三、请求参数规范

## 3.1 查询参数

```java
// 列表查询使用 Query String
GET /api/v1/users?name=张三&age=25&page=1&pageSize=20&sort=createTime,desc

// 参数命名：小驼峰（与 Java 字段风格一致）
// 分页参数：page（页码，从1开始）、pageSize（每页条数）
// 排序参数：sort=字段名,排序方向（asc/desc）
// 过滤参数：直接用字段名
```

## 3.2 请求体

```java
// 使用 JSON 格式
POST /api/v1/users
Content-Type: application/json

{
    "name": "张三",
    "age": 25,
    "email": "zhangsan@example.com"
}

// 字段命名：小驼峰（与 Java 字段风格一致）
// 不要使用下划线风格
// BAD:  { "user_name": "张三" }
// GOOD: { "userName": "张三" }
```

## 3.3 路径参数

```java
// 资源标识符使用路径参数
GET /api/v1/users/123
DELETE /api/v1/users/123

// 路径参数类型统一用字符串（如需特殊类型在后端转换）
```

---

# 四、返回值结构

## 4.1 统一响应体

```java
// 成功响应
{
    "code": 200,
    "message": "success",
    "data": { ... }
}

// 列表响应
{
    "code": 200,
    "message": "success",
    "data": {
        "list": [ ... ],
        "total": 100,
        "page": 1,
        "pageSize": 20
    }
}

// 错误响应
{
    "code": 40001,
    "message": "用户不存在",
    "data": null
}
```

## 4.2 统一响应类（Java 实现）

```java
@Data
public class R<T> implements Serializable {
    private int code;
    private String message;
    private T data;

    public static <T> R<T> ok() {
        return ok(null);
    }

    public static <T> R<T> ok(T data) {
        R<T> r = new R<>();
        r.setCode(200);
        r.setMessage("success");
        r.setData(data);
        return r;
    }

    public static <T> R<T> fail(String message) {
        return fail(500, message);
    }

    public static <T> R<T> fail(int code, String message) {
        R<T> r = new R<>();
        r.setCode(code);
        r.setMessage(message);
        return r;
    }
}

// 分页结果封装
@Data
public class PageResult<T> implements Serializable {
    private List<T> list;
    private long total;
    private int page;
    private int pageSize;

    public static <T> PageResult<T> of(List<T> list, long total, int page, int pageSize) {
        PageResult<T> result = new PageResult<>();
        result.setList(list);
        result.setTotal(total);
        result.setPage(page);
        result.setPageSize(pageSize);
        return result;
    }
}
```

## 4.3 Controller 示例

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Resource
    private UserService userService;

    @GetMapping
    public R<PageResult<UserResult>> query(UserQuery query) {
        return R.ok(userService.query(query));
    }

    @GetMapping("/{id}")
    public R<UserResult> detail(@PathVariable Long id) {
        return R.ok(userService.getDetail(id));
    }

    @PostMapping
    public R<Long> add(@RequestBody @Valid UserAddParam param) {
        return R.ok(userService.add(param));
    }

    @PutMapping("/{id}")
    public R<Void> edit(@PathVariable Long id,
                        @RequestBody @Valid UserEditParam param) {
        userService.edit(id, param);
        return R.ok();
    }

    @DeleteMapping("/{id}")
    public R<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return R.ok();
    }

    // 非 CRUD 操作
    @PostMapping("/{id}/reset-password")
    public R<Void> resetPassword(@PathVariable Long id) {
        userService.resetPassword(id);
        return R.ok();
    }
}
```

---

# 五、错误码体系

## 5.1 错误码分层

```
格式：AABBBB

AA   — 模块编号（00=通用，01=用户，02=订单，03=仓库...）
BBBB — 错误编号

通用错误码：
  000001 — 参数校验失败
  000002 — 未登录/Token过期
  000003 — 无权限
  000004 — 资源不存在
  000005 — 业务异常
  000006 — 系统繁忙

或者简化为 HTTP 语义码：
  200 — 成功
  400 — 参数错误
  401 — 未认证
  403 — 无权限
  404 — 资源不存在
  500 — 服务端错误
```

## 5.2 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    // 业务异常
    @ExceptionHandler(BusinessException.class)
    public R<Void> handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return R.fail(e.getCode(), e.getMessage());
    }

    // 参数校验异常（@Valid）
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public R<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.joining("; "));
        log.warn("参数校验失败: {}", message);
        return R.fail(400, message);
    }

    // 未知异常
    @ExceptionHandler(Exception.class)
    public R<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return R.fail(500, "系统繁忙，请稍后重试");
    }
}
```

---

# 六、分页约定

```java
// 请求
GET /api/v1/users?page=1&pageSize=20&sort=createTime,desc

// 页码从 1 开始
// 默认 pageSize = 20
// 最大 pageSize = 100（防止一次查询过多）

// 响应
{
    "code": 200,
    "message": "success",
    "data": {
        "list": [...],
        "total": 156,
        "page": 1,
        "pageSize": 20
    }
}

// Service 层分页实现
public PageResult<UserResult> query(UserQuery query) {
    // 设置默认值
    int page = query.getPage() == null ? 1 : query.getPage();
    int pageSize = query.getPageSize() == null ? 20 : query.getPageSize();
    pageSize = Math.min(pageSize, 100);  // 上限保护

    // 分页查询
    PageHelper.startPage(page, pageSize);
    List<User> list = userMapper.selectByQuery(query);
    PageInfo<User> pageInfo = new PageInfo<>(list);

    // 转换返回
    List<UserResult> results = BeanUtil.copyToList(list, UserResult.class);
    return PageResult.of(results, pageInfo.getTotal(), page, pageSize);
}
```

---

# 七、接口版本控制

```java
// 方式1：URL 路径（推荐，最清晰）
GET /api/v1/users
GET /api/v2/users

// 方式2：请求头
GET /api/users
Header: Accept-Version: v1

// 方式3：请求参数（不推荐，容易遗漏）
GET /api/users?version=1

// 版本升级原则
// 1. 新增字段 → 不需要升级版本（向后兼容）
// 2. 修改字段含义 → 必须新版本
// 3. 删除字段 → 逐步废弃，先标记 deprecated
```

---

# 八、接口文档

```java
// 使用 Swagger/Knife4j 注解生成文档
@Tag(name = "用户管理", description = "用户的增删改查")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Operation(summary = "查询用户列表", description = "支持按姓名、年龄、部门筛选")
    @GetMapping
    public R<PageResult<UserResult>> query(UserQuery query) { }

    @Operation(summary = "获取用户详情")
    @GetMapping("/{id}")
    public R<UserResult> detail(@Parameter(description = "用户ID") @PathVariable Long id) { }

    @Operation(summary = "新增用户")
    @PostMapping
    public R<Long> add(@RequestBody @Valid UserAddParam param) { }
}

// 请求参数注解
@Data
public class UserAddParam {
    @Schema(description = "用户姓名", required = true, example = "张三")
    @NotBlank(message = "姓名不能为空")
    private String name;

    @Schema(description = "年龄", example = "25")
    @Min(value = 0, message = "年龄不能为负数")
    private Integer age;
}
```

---

# 九、安全相关

```java
// 1. 参数校验 — 使用 @Valid + Bean Validation
@PostMapping
public R<Long> add(@RequestBody @Valid UserAddParam param) { }

@Data
public class UserAddParam {
    @NotBlank(message = "姓名不能为空")
    @Size(max = 50, message = "姓名不超过50个字符")
    private String name;

    @NotNull(message = "年龄不能为空")
    @Min(value = 0) @Max(value = 150)
    private Integer age;

    @Email(message = "邮箱格式不正确")
    private String email;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
}

// 2. 敏感数据脱敏
// 手机号、身份证号、银行卡号等返回时需要脱敏
// "138****1234"

// 3. 接口限流
// 使用 @RateLimiter 或 Sentinel 等限流工具

// 4. 防重复提交
// 使用 Token 机制或幂等性设计
```

---

# 十、API 设计速查表

| 规则 | BAD | GOOD |
|------|-----|------|
| URL 风格 | `/getUserInfo` | `/api/v1/users/{id}` |
| 资源命名 | `/api/v1/user` | `/api/v1/users`（复数） |
| 查询用户 | `POST /getUser` | `GET /api/v1/users?name=张三` |
| 新增用户 | `POST /addUser` | `POST /api/v1/users` |
| 删除用户 | `POST /deleteUser?id=1` | `DELETE /api/v1/users/1` |
| 响应格式 | 返回裸 List | `{ "code":200, "data": {...} }` |
| 错误信息 | 返回 200 + 错误描述 | 返回对应 HTTP 状态码 + 错误码 |
| 分页参数 | `startRow/limit` | `page/pageSize` |
| 排序 | `orderBy=1` | `sort=createTime,desc` |
