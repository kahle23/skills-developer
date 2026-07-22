# 测试与工具链

测试规范、类型检查、虚拟环境、依赖管理、代码格式化。

---

# 一、pytest 测试

## 1.1 基本用法

```python
# test_user_service.py
import pytest
from app.services import UserService

class TestUserService:
    def setup_method(self):
        self.service = UserService(test_db)

    def test_get_user_returns_user(self):
        user = self.service.get_user(1)
        assert user is not None
        assert user.name == "张三"

    def test_get_user_returns_none_for_missing(self):
        user = self.service.get_user(999)
        assert user is None
```

## 1.2 fixture（测试夹具）

```python
import pytest

@pytest.fixture
def db_session():
    session = create_test_session()
    yield session          # yield 提供资源，测试后清理
    session.close()

@pytest.fixture
def sample_user(db_session):  # fixture 可以依赖其他 fixture
    user = User(name="测试用户", age=20)
    db_session.add(user)
    return user

def test_user_age(sample_user):
    assert sample_user.age == 20
```

## 1.3 参数化测试

```python
@pytest.mark.parametrize("input,expected", [
    ("", False),
    ("a" * 256, False),
    ("test@example.com", True),
    ("invalid", False),
])
def test_validate_email(input, expected):
    assert validate_email(input) == expected
```

## 1.4 异常测试

```python
def test_create_user_raises_for_empty_name():
    with pytest.raises(ValueError, match="name 不能为空"):
        service.create_user(name="", age=18)
```

## 1.5 mock

```python
from unittest.mock import patch, MagicMock

@patch("app.services.user_service.external_api")
def test_get_user_with_mock(mock_api):
    mock_api.get_user.return_value = {"id": 1, "name": "张三"}

    user = service.get_user(1)

    assert user.name == "张三"
    mock_api.get_user.assert_called_once_with(1)
```

---

# 二、类型检查（mypy）

## 2.1 基本配置

```ini
# mypy.ini 或 pyproject.toml [tool.mypy]
[mypy]
python_version = 3.9
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
ignore_missing_imports = True
```

## 2.2 常见类型问题

```python
# mypy 能检测出的问题：
# 1. 参数类型不匹配
def add(a: int, b: int) -> int:
    return a + b
add("1", 2)  # mypy 报错

# 2. 返回类型不匹配
def get_name(user: User) -> str:
    return user.age  # mypy 报错：int 不是 str

# 3. None 不安全
def get_user(uid: int) -> User:
    user = db.find(uid)
    return user  # mypy 报错：可能返回 None
```

## 2.3 类型忽略与 Any

```python
# 临时忽略类型检查
result = untyped_function()  # type: ignore

# 明确标记 Any
from typing import Any
def parse(data: dict[str, Any]) -> Any:
    ...
```

---

# 三、虚拟环境

## 3.1 venv（标准库）

```bash
# 创建虚拟环境
python -m venv .venv

# 激活（Windows）
.venv\Scripts\activate

# 激活（macOS/Linux）
source .venv/bin/activate

# 退出
deactivate
```

## 3.2 依赖管理

```bash
# 导出当前依赖
pip freeze > requirements.txt

# 安装依赖
pip install -r requirements.txt

# 推荐使用 pip-tools 精确管理
pip install pip-tools
pip-compile requirements.in          # 生成锁定的 requirements.txt
pip-sync requirements.txt            # 同步安装（卸载多余的包）
```

## 3.3 pyproject.toml（现代项目标准）

```toml
[project]
name = "my-package"
version = "1.0.0"
requires-python = ">=3.9"
dependencies = [
    "requests>=2.28,<3",
    "pydantic>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "mypy>=1.0",
    "ruff>=0.1",
]
```

---

# 四、代码格式化与检查

## 4.1 Ruff（推荐，速度快）

> 集成 linter + formatter，替代 flake8 + isort + black。

```bash
# 安装
pip install ruff

# 检查
ruff check .

# 自动修复
ruff check --fix .

# 格式化
ruff format .
```

```toml
# pyproject.toml [tool.ruff]
[tool.ruff]
line-length = 120
target-version = "py39"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "UP", "B"]
ignore = ["E501"]  # 行长度由 formatter 管
```

## 4.2 Black（格式化）

```bash
pip install black
black .
```

```toml
# pyproject.toml [tool.black]
[tool.black]
line-length = 120
target-version = ["py39"]
```

## 4.3 isort（import 排序）

```bash
pip install isort
isort .
```

---

# 五、项目目录结构

```
my_project/
├── pyproject.toml          # 项目配置（推荐）
├── requirements.txt        # 依赖锁定（兼容旧项目）
├── README.md
├── src/                    # 源码（推荐 src layout）
│   └── my_package/
│       ├── __init__.py
│       ├── models/
│       ├── services/
│       └── utils/
├── tests/                  # 测试
│   ├── conftest.py         # pytest 公共 fixture
│   ├── test_models.py
│   └── test_services.py
└── .gitignore
```

---

# 六、.gitignore 要点

```gitignore
# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/

# 虚拟环境
.venv/
venv/
env/

# IDE
.idea/
.vscode/
*.swp

# 测试 / 类型检查
.pytest_cache/
.mypy_cache/
.ruff_cache/
htmlcov/
.coverage

# 环境变量（绝不提交）
.env
*.env.local
```

---

# 七、日志配置

```python
import logging

# 基本配置
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s - %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)

# 按模块获取 logger
logger = logging.getLogger(__name__)
```

> 详见 `03-code-standards.md` 的日志规范章节。

---

# 八、工具链速查

| 需求 | 工具 |
|------|------|
| 测试框架 | `pytest` |
| 类型检查 | `mypy` |
| Lint + 格式化 | `ruff`（推荐） / `flake8` + `black` + `isort` |
| 依赖管理 | `pip` + `requirements.txt` / `poetry` / `uv` |
| 虚拟环境 | `venv` / `uv venv` |
| 文档生成 | `mkdocs` / `sphinx` |
| 安全扫描 | `pip-audit` / `safety` |
