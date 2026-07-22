# 03 - Playwright 脚本模板

> 基于 **Playwright Python（同步 API）+ pytest** 的 E2E 测试项目骨架。
> 所有交互操作的详细写法见 05-交互操作手册。

---

## 一、环境准备

### 1.1 依赖安装

```bash
# 创建虚拟环境
python -m venv .venv

# 激活（Windows）
.venv\Scripts\activate

# 激活（macOS/Linux）
source .venv/bin/activate

# 安装依赖
pip install pytest playwright pytest-playwright python-dotenv

# 安装浏览器内核（只需一次）
playwright install chromium

# 可选：安装所有浏览器
playwright install
```

> `pytest-playwright` 插件自动提供 `page` fixture，无需手写即可在测试中使用 `page` 参数。

### 1.2 requirements.txt

```
pytest>=7.0
playwright>=1.40
pytest-playwright>=0.4.0
python-dotenv>=1.0
```

---

## 二、项目目录结构

```
tests/
├── conftest.py              # 公共 fixture（登录态、基础 URL 等）
├── pages/                   # 页面对象（POM）
│   ├── __init__.py
│   ├── base_page.py         # 基础页类（公共操作）
│   ├── login_page.py        # 登录页
│   └── user_manage_page.py  # 用户管理页
├── test_user/               # 用户模块测试
│   ├── test_user_create.py  # 新增用户测试
│   └── test_user_list.py    # 用户列表测试
├── pytest.ini               # pytest 配置
└── .env                     # 环境变量（不提交）
```

---

## 三、配置文件

### 3.1 pytest.ini

```ini
[pytest]
python_files = test_*.py
python_classes = Test*
python_functions = test_*

# 命令行参数
addopts =
    -v
    --tb=short
    --browser chromium

# 标记
markers =
    smoke: 冒烟测试（核心流程）
    regression: 回归测试（全量）
    slow: 耗时较长的测试
```

### 3.2 .env（不提交）

```ini
BASE_URL=http://localhost:8080
LOGIN_USERNAME=admin
LOGIN_PASSWORD=admin123
```

---

## 四、conftest.py — 公共 fixture

```python
"""
conftest.py - 全局公共 fixture，所有测试文件共享，无需 import。
"""
import os
import pytest
from dotenv import load_dotenv

from pages.login_page import LoginPage

load_dotenv()

BASE_URL = os.getenv("BASE_URL", "http://localhost:8080")


@pytest.fixture(scope="session")
def base_url():
    """测试环境基础 URL"""
    return BASE_URL


@pytest.fixture(scope="function")
def logged_in_page(page, base_url):
    """
    每个测试前自动登录，返回已登录的 page 对象。
    pytest-playwright 提供的 page fixture 是 function 级别，每个测试自动隔离。
    """
    login_page = LoginPage(page, base_url)
    login_page.navigate()
    login_page.login(
        os.getenv("LOGIN_USERNAME"),
        os.getenv("LOGIN_PASSWORD"),
    )
    yield page
    # 测试后无需手动清理，page fixture 会自动关闭


@pytest.fixture
def test_data():
    """测试数据工厂，按需扩展。"""
    import uuid
    return {
        "username": f"testuser_{uuid.uuid4().hex[:8]}",
        "name": "自动化测试用户",
        "phone": "13800138000",
    }
```

---

## 五、基础页类（POM 核心）

### pages/base_page.py

```python
"""
基础页类 - 所有页面对象的父类。
封装公共操作，子类继承后只需定义元素定位和页面特有操作。
"""
from playwright.sync_api import Page, expect


class BasePage:
    def __init__(self, page: Page, base_url: str = ""):
        self.page = page
        self.base_url = base_url

    def navigate(self, path: str = ""):
        """导航到指定路径"""
        self.page.goto(f"{self.base_url}{path}")

    def get_field(self, label: str):
        """通过 label 文本定位表单字段（推荐方式）"""
        return self.page.get_by_label(label)

    def get_button(self, name: str):
        """通过按钮文本定位按钮"""
        return self.page.get_by_role("button", name=name)

    def click_button(self, name: str):
        """点击按钮"""
        self.get_button(name).click()

    def fill_field(self, label: str, value: str):
        """填写表单字段"""
        self.get_field(label).fill(value)

    def expect_toast(self, text: str):
        """
        断言提示消息出现。
        适配 Element UI / Element Plus 的 el-message。
        """
        expect(self.page.get_by_text(text).first).to_be_visible()

    def expect_field_error(self, field_label: str, error_text: str):
        """
        断言表单校验错误。
        适配 Element UI / Element Plus 的 form-item 校验提示。
        """
        form_item = self.get_field(field_label).locator(
            "xpath=ancestor::*[contains(@class,'el-form-item')]"
        )
        expect(form_item.get_by_text(error_text)).to_be_visible()

    def wait_for_loading(self):
        """等待页面加载完成（适配 Element UI 的 loading 遮罩）"""
        loading = self.page.locator(".el-loading-mask")
        if loading.count() > 0:
            loading.wait_for(state="hidden")
```

---

## 六、页面对象示例

### 6.1 pages/login_page.py

```python
from pages.base_page import BasePage


class LoginPage(BasePage):
    def navigate(self):
        self.navigate("/login")

    def login(self, username: str, password: str):
        """执行登录"""
        self.fill_field("用户名", username)
        self.fill_field("密码", password)
        self.click_button("登录")
        # 等待登录完成（URL 离开 /login）
        self.page.wait_for_url(lambda url: "/login" not in url)
```

### 6.2 pages/user_manage_page.py

```python
"""
用户管理页对象。

⚠️ 以下定位器基于 Vue + Element UI 的常见结构。
   实际项目中需根据代码调整定位策略，详见 05-交互操作手册。
"""
from playwright.sync_api import expect
from pages.base_page import BasePage


class UserManagePage(BasePage):
    def navigate(self):
        self.navigate("/system/user")

    def open_create_dialog(self):
        """打开新增用户弹窗"""
        self.click_button("新增")

    def fill_user_form(self, data: dict):
        """
        填写用户表单。
        data 示例：
        {
            "username": "testuser01",
            "name": "测试用户",
            "phone": "13800138000",
            "role": "普通用户",   # 下拉选择
        }
        """
        self.fill_field("用户名", data["username"])
        self.fill_field("姓名", data["name"])
        self.fill_field("手机号", data["phone"])

        # ⚠️ 下拉选择：Element UI 的自定义下拉不是原生 <select>
        # 详见 05-交互操作手册 → 下拉选择章节
        self.select_element_dropdown("角色", data["role"])

    def select_element_dropdown(self, field_label: str, option_text: str):
        """
        操作 Element UI / Element Plus 的自定义下拉。
        不能用 select_option（那是给原生 <select> 用的）。
        """
        trigger = self.get_field(field_label)
        trigger.click()
        # Element UI 选项面板的选项有 role="option"
        self.page.get_by_role("option", name=option_text).click()

    def submit_form(self):
        self.click_button("保存")

    def expect_user_in_list(self, username: str):
        """断言用户出现在列表中"""
        expect(self.page.get_by_text(username).first).to_be_visible()

    def expect_form_error(self, field_label: str, error_text: str):
        """断言表单校验错误"""
        self.expect_field_error(field_label, error_text)
```

---

## 七、测试文件示例

### test_user/test_user_create.py

```python
"""
用户新增功能 - E2E 测试
对应测试用例：TC-USER-001 ~ TC-USER-005

运行：pytest tests/test_user/test_user_create.py -v
"""
import pytest
from pages.user_manage_page import UserManagePage


class TestUserCreate:
    """用户新增功能测试"""

    # ---- TC-USER-001: 正常新增 ----
    @pytest.mark.smoke
    def test_create_user_success(self, logged_in_page, base_url, test_data):
        """
        TC-USER-001：新增用户 - 正常新增
        """
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_toast("新增成功")
        page.expect_user_in_list(test_data["username"])

    # ---- TC-USER-002: 用户名为空 ----
    @pytest.mark.smoke
    def test_create_user_empty_username(self, logged_in_page, base_url, test_data):
        """
        TC-USER-002：新增用户 - 用户名为空
        """
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["username"] = ""
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_form_error("用户名", "请输入用户名")

    # ---- TC-USER-003: 手机号格式错误 ----
    @pytest.mark.regression
    def test_create_user_invalid_phone(self, logged_in_page, base_url, test_data):
        """TC-USER-003：新增用户 - 手机号格式错误"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["phone"] = "123"
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_form_error("手机号", "手机号格式不正确")

    # ---- TC-USER-004: 用户名重复 ----
    @pytest.mark.regression
    def test_create_user_duplicate_username(self, logged_in_page, base_url, test_data):
        """TC-USER-004：新增用户 - 用户名重复"""
        # TODO: 确认测试环境已存在的用户名
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["username"] = "admin"  # 已存在的用户名
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_toast("用户名已存在")

    # ---- TC-USER-005: 边界值 - 用户名超长 ----
    @pytest.mark.slow
    def test_create_user_username_too_long(self, logged_in_page, base_url, test_data):
        """TC-USER-005：新增用户 - 用户名超过最大长度"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["username"] = "a" * 256
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_form_error("用户名", "用户名长度不能超过")
```

---

## 八、运行测试

```bash
# 运行所有测试
pytest

# 运行指定文件
pytest tests/test_user/test_user_create.py -v

# 只跑冒烟测试
pytest -m smoke

# 只跑回归测试
pytest -m regression

# 显示浏览器窗口（调试用）
pytest --headed

# 慢动作模式（每个操作之间暂停 500ms）
pytest --headed --slowmo 500

# 生成 HTML 报告
pytest --html=report.html

# 失败时自动截图
pytest --screenshot=only-on-failure

# 失败时录制视频
pytest --video=retain-on-failure

# 查看详细 trace（失败时）
pytest --trace=on
```

---

## 九、关键注意事项

### 9.1 同步 API vs 异步 API

本技能所有示例使用**同步 API**（`sync_playwright`）：
- 学习成本低，代码直观
- 配合 pytest 同步执行，调试方便
- 对于 E2E 测试（非高并发场景）性能足够

如果项目已有异步框架（asyncio），可用 `async_playwright`，但需搭配 `pytest-asyncio`。

### 9.2 元素定位优先级

```
get_by_label()       ← 表单字段首选（有 label 关联时）
get_by_role()        ← 按钮、链接、标题等交互元素首选
get_by_placeholder() ← 没有 label 但有 placeholder 的字段
get_by_text()        ← 非交互元素（提示文字、列表数据）
get_by_test_id()     ← 最后手段（需前端配合加 data-testid）
```

> ⚠️ **不推荐** CSS 选择器和 XPath，DOM 结构变化会导致测试脆弱。
> 但 Element UI 的自定义组件（select、date-picker）可能需要辅助定位，详见 05。

### 9.3 ⚠️ 等待策略

Playwright 内置**自动等待**，大部分场景不需要手动 `wait`：

```python
# ✅ 正确：Playwright 会自动等元素可操作
page.get_by_label("用户名").fill("admin")

# ❌ 错误：不需要手动 sleep
import time; time.sleep(2)

# ✅ 需要等待特定条件时用 expect 断言（自带等待重试）
expect(page.get_by_text("新增成功")).to_be_visible()
```

什么时候需要手动等待：
- 等待网络请求完成：`page.wait_for_response("**/api/user/create")`
- 等待 URL 变化：`page.wait_for_url("**/system/user")`
- 等待 loading 遮罩消失：见 base_page.wait_for_loading()

### 9.4 测试数据管理

| 策略 | 适用场景 | 示例 |
|------|---------|------|
| 随机数据 | 避免重复冲突 | `f"test_{uuid.uuid4().hex[:8]}"` |
| 固定数据 | 需要断言特定值 | `username = "testuser_fixed"` |
| 前置创建 | 测试依赖已有数据 | 测试前先调 API 创建 |
| 测试后清理 | 保持环境干净 | fixture 中 yield 后删除 |
