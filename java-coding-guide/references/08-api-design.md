# API 设计规范

RESTful API 设计规范：URL 命名、HTTP 方法、请求参数、返回值结构、分页约定、错误码体系等。

---

# 一、URL 命名规范

**核心规则：**
- 路径格式：`/api/{版本}/{模块}/{资源}`，使用小写字母和连字符（kebab-case）
- 资源用复数名词，HTTP 方法表达动作
- 嵌套不超过两层，超过则用查询参数扁平化

```java
// GOOD
GET    /api/v1/users              // 查询列表
GET    /api/v1/users/{id}         // 查询单个
POST   /api/v1/users              // 新增
PUT    /api/v1/users/{id}         // 全量更新
PATCH  /api/v1/users/{id}         // 部分更新
DELETE /api/v1/users/{id}         // 删除
GET    /api/v1/work-orders/{id}   // 多词用连字符

// 子资源（最多两层）
GET /api/v1/users/{userId}/orders
// 超过两层 → 扁平化
GET /api/v1/reviews?userId=1&orderId=2&itemId=3

// 非 CRUD 操作：用动词表达状态变更
POST /api/v1/work-orders/{id}/submit
POST /api/v1/orders/{id}/refund
POST /api/v1/users/{id}/reset-password
// 或用 PATCH 变更状态
PATCH /api/v1/work-orders/{id}  Body: { "status": "SUBMITTED" }
```

---

# 二、HTTP 方法

| 方法 | 语义 | 幂等 | 典型场景 |
|------|------|------|---------|
| GET | 查询 | ✅ | 获取资源列表/详情 |
| POST | 创建 | ❌ | 新增资源、触发操作 |
| PUT | 全量替换 | ✅ | 更新整个资源 |
| PATCH | 部分更新 | ✅ | 更新某些字段 |
| DELETE | 删除 | ✅ | 删除资源 |

---

# 三、请求参数规范

```java
// 查询参数：小驼峰命名
GET /api/v1/users?name=张三&page=1&pageSize=20&sort=createTime,desc

// 请求体：JSON 格式，字段用小驼峰
POST /api/v1/users
Content-Type: application/json
{ "name": "张三", "userName": "张三" }  // 不用 user_name

// 路径参数：资源标识符
GET /api/v1/users/123
```

---

# 四、返回值结构

## 4.1 统一响应体

```json
// 成功
{ "code": 200, "message": "success", "data": { ... } }

// 列表
{ "code": 200, "message": "success", "data": { "list": [...], "total": 100, "page": 1, "pageSize": 20 } }

// 错误
{ "code": 40001, "message": "用户不存在", "data": null }
```

## 4.2 统一响应类

```java
@Data
public class R<T> implements Serializable {
    private int code;
    private String message;
    private T data;

    public static <T> R<T> ok(T data) { ... }     // code=200, message="success"
    public static <T> R<T> fail(int code, String message) { ... }
}

// PageResult 字段：list, total, page, pageSize
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
    public R<Void> edit(@PathVariable Long id, @RequestBody @Valid UserEditParam param) {
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
格式：AABBBB（AA=模块编号，BBBB=错误编号）
  00=通用，01=用户，02=订单，03=仓库...

通用错误码：
  000001 — 参数校验失败
  000002 — 未登录/Token过期
  000003 — 无权限
  000004 — 资源不存在
  000005 — 业务异常
  000006 — 系统繁忙
```

## 5.2 全局异常处理

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public R<Void> handleBusinessException(BusinessException e) {
        return R.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public R<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.joining("; "));
        return R.fail(400, message);
    }

    @ExceptionHandler(Exception.class)
    public R<Void> handleException(Exception e) {
        return R.fail(500, "系统繁忙，请稍后重试");
    }
}
```

---

# 六、分页约定

```java
// 请求
GET /api/v1/users?page=1&pageSize=20&sort=createTime,desc
// 页码从 1 开始，默认 pageSize=20，最大 pageSize=100

// 响应
{ "code": 200, "message": "success", "data": { "list": [...], "total": 156, "page": 1, "pageSize": 20 } }
```

Service 层分页实现参考 `23-baibao` 中的 `PageUtil`。

---

# 七、接口版本控制

```
方式1：URL 路径（推荐，最清晰）          GET /api/v1/users
方式2：请求头（干净但不易发现）            Header: Accept-Version: v1
方式3：请求参数（不推荐，容易遗漏）         GET /api/users?version=1
```

升级原则：新增字段不需升级（向后兼容）；修改字段含义或删除字段必须新版本。

---

# 八、接口文档

```java
@Tag(name = "用户管理", description = "用户的增删改查")
@RestController
@RequestMapping("/api/v1/users")
public class UserController {

    @Operation(summary = "查询用户列表", description = "支持按姓名、部门筛选")
    @GetMapping
    public R<PageResult<UserResult>> query(UserQuery query) { }

    @Operation(summary = "获取用户详情")
    @GetMapping("/{id}")
    public R<UserResult> detail(@Parameter(description = "用户ID") @PathVariable Long id) { }
}

// 参数注解
@Schema(description = "用户姓名", required = true, example = "张三")
@NotBlank(message = "姓名不能为空")
private String name;
```

---

# 九、安全相关

```java
// 参数校验 — @Valid + Bean Validation
@PostMapping
public R<Long> add(@RequestBody @Valid UserAddParam param) { }

@Data
public class UserAddParam {
    @NotBlank @Size(max = 50)        private String name;
    @NotNull @Min(0) @Max(150)       private Integer age;
    @Email                            private String email;
    @Pattern(regexp = "^1[3-9]\\d{9}$") private String phone;
}
```

- 敏感数据脱敏：手机号、身份证号、银行卡号等返回时脱敏（如 `138****1234`）
- 接口限流：使用 `@RateLimiter` 或 Sentinel
- 防重复提交：使用 Token 机制或幂等性设计

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
