# 代码规范

涵盖 Python 编码核心规范：命名、结构、文档字符串、类型提示、异常处理、日志、None 安全。

---

# 一、命名规范

| 类型 | 风格 | 示例 |
|------|------|------|
| 模块/包 | 全小写下划线 | `user_service.py` |
| 类 | 大驼峰（PascalCase），枚举不加 `Enum` 后缀 | `UserService`、`OrderStatus` |
| 函数/变量 | 小写下划线（snake_case） | `get_user`、`order_count` |
| 常量 | 全大写下划线 | `MAX_RETRY`、`DEFAULT_TIMEOUT` |
| 布尔 | is/has/can 前缀 | `is_active`、`has_children` |

函数常用动词前缀：`get/set`、`save/update/delete`、`query/find`、`count/exists`、`is_/has_`、`convert/build/validate`、`parse/format`。

```python
# BAD — 覆盖内置名称
list = [1, 2, 3]       # 覆盖了 list()
id = 100               # 覆盖了 id()
type = "user"          # 覆盖了 type()

# GOOD
data_list = [1, 2, 3]
user_id = 100
user_type = "user"
```

避免：单字母（循环 `i/j` 除外）、模糊名（`data`、`temp`）、与内置冲突的名称。

---

# 二、代码结构规范

## 2.1 模块结构顺序

```python
"""模块文档字符串。"""

# 1. 导入（标准库 → 第三方库 → 本地模块）
import os
import requests
from app.models import User

# 2. 常量
MAX_RETRY = 3

# 3. 类定义
class UserService: ...

# 4. 函数定义
def main(): ...

# 5. 主入口
if __name__ == "__main__":
    main()
```

## 2.2 类成员顺序

`__init__` → 魔术方法 → `@property` → **公共方法 → 私有方法**

```python
class UserService:
    """用户服务类。"""

    def __init__(self, db: Session) -> None:
        self._db = db
        self._cache: dict[int, User] = {}

    def __repr__(self) -> str:
        return f"<UserService(cache={len(self._cache)})>"

    @property
    def cache_size(self) -> int:
        return len(self._cache)

    # 公共方法（对外接口，优先展示）
    def get_user(self, user_id: int) -> Optional[User]: ...

    # 私有方法（实现细节）
    def _load_from_cache(self, user_id: int) -> Optional[User]: ...
```

## 2.3 函数参数

参数超过 3~4 个时封装为 dataclass：

```python
@dataclass
class OrderCreateParam:
    user_id: int
    product_id: int
    quantity: int
    address: str
    coupon: Optional[str] = None

def create_order(param: OrderCreateParam) -> Order: ...
```

## 2.4 控制流——卫语句提前返回

```python
# GOOD — 卫语句，扁平
def process(user: User) -> str:
    if user is None:
        raise ValueError("用户不能为空")
    if not user.is_active:
        raise ValueError("用户未激活")
    return do_something(user)
```

## 2.5 空行规则

- 顶层函数/类之间：空 2 行
- 类内方法之间：空 1 行
- 函数内逻辑块之间：空 1 行

---

# 三、文档字符串（Docstring）

## 3.1 模块级

```python
"""用户服务模块，提供用户增删改查和权限管理。

采用 Repository + Service 分层设计，支持缓存加速。
"""
```

第一行一句话概括，空一行后补充说明。不需要放示例。

## 3.2 函数/方法

```python
def query_users(dept_id: int, keyword: Optional[str] = None) -> list[User]:
    """查询部门下的用户列表。

    Args:
        dept_id: 部门ID。
        keyword: 搜索关键词，为 None 时不筛选。

    Returns:
        用户列表，按创建时间倒序。

    Raises:
        ValueError: dept_id 为空时抛出。
    """
    ...
```

**编写原则**：公共 API 必须写；第一行概括用途；私有方法或一目了然的函数可省略。

---

# 四、类型提示（Type Hints）

所有函数参数和返回值必须有类型提示。

```python
# 基本类型
def get_name(user_id: int) -> str: ...

# 集合类型（3.9+ 直接用 list/dict/set）
def query(ids: list[int]) -> list[User]: ...
def get_config(key: str) -> dict[str, Any]: ...

# Optional — 可能为 None
def find_user(user_id: int) -> Optional[User]: ...

# 多类型（3.10+ 用 | ）
def process(data: str | bytes) -> str: ...
```

---

# 五、异常处理

## 5.1 自定义业务异常

```python
class BusinessError(Exception):
    """业务异常基类。"""
    def __init__(self, code: str, message: str) -> None:
        self.code = code
        super().__init__(message)
```

## 5.2 处理原则

```python
# BAD — 吞掉异常
try:
    do_something()
except Exception:
    pass

# BAD — 捕获太宽
try:
    data = json.loads(text)
except Exception:
    data = {}

# GOOD — 捕获具体异常
try:
    data = json.loads(text)
except json.JSONDecodeError as e:
    logger.error("JSON 解析失败: %s", e)
    data = {}
```

- **禁止裸 `except:`** — 会捕获 `KeyboardInterrupt`、`SystemExit`
- **捕获具体异常** — 知道什么会抛出就捕获什么
- **不要用异常控制正常流程**

## 5.3 资源管理

用 `with` 自动释放资源：

```python
with open(path, "r", encoding="utf-8") as f:
    content = f.read()

from contextlib import suppress
with suppress(FileNotFoundError):     # 忽略特定异常
    os.remove(temp_file)
```

---

# 六、日志规范

```python
import logging
logger = logging.getLogger(__name__)

logger.info("订单创建成功: order_no=%s", order_no)
logger.error("保存用户失败: user_id=%s", user_id, exc_info=True)
```

- **用占位符 `%s`**，不用 f-string（级别不够时仍会计算 f-string）
- **ERROR 带 `exc_info=True`** 打印完整堆栈
- **不打印密码、Token、身份证号**等敏感信息

```python
# BAD
logger.info(f"用户ID: {user_id}")          # f-string
logger.info("密码: %s", password)           # 敏感信息

# GOOD
logger.info("用户ID: %s", user_id)
```

---

# 七、None 安全

```python
# 参数校验前置
def process_user(user_id: int, name: str) -> None:
    if user_id is None or user_id <= 0:
        raise ValueError(f"非法 user_id: {user_id}")
    if not name or not name.strip():
        raise ValueError("name 不能为空")

# 返回空集合而非 None
def get_users(dept_id: int) -> list[User]:
    if dept_id is None:
        return []
    return user_repo.query_by_dept(dept_id)

# None 判断用 is，不用 ==
if user is None: ...       # GOOD
if user == None: ...       # BAD

# 字典安全取值
name = user_dict.get("name", "")
name = user_dict.get("name") or "未知"

# 海象运算符（3.8+）赋值 + 判断
if (user := get_user(uid)) is not None:
    process(user)
```
