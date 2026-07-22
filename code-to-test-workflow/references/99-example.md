# 99 - 完整示例：用户新增功能

> 以"用户新增"功能为例，展示从代码到测试用例 + 脚本的完整产出。
> 假设这是一个 **Vue + Element UI** 的后台管理系统。

---

## 一、假设的代码

> 实际使用时，这里的代码应该是你从项目中读到的真实代码。

### 1.1 前端页面（user/index.vue）

```vue
<template>
  <div class="app-container">
    <el-button type="primary" @click="handleAdd">新增</el-button>

    <el-table :data="userList">
      <el-table-column prop="username" label="用户名" />
      <el-table-column prop="name" label="姓名" />
      <el-table-column prop="phone" label="手机号" />
      <el-table-column prop="roleName" label="角色" />
      <el-table-column label="操作">
        <template #default="{ row }">
          <el-button type="text" @click="handleEdit(row)">编辑</el-button>
          <el-button type="text" @click="handleDelete(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 新增弹窗 -->
    <el-dialog title="新增用户" :visible.sync="dialogVisible" width="500px">
      <el-form :model="form" :rules="rules" ref="formRef" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="form.username" placeholder="请输入用户名" />
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input v-model="form.name" placeholder="请输入姓名" />
        </el-form-item>
        <el-form-item label="手机号" prop="phone">
          <el-input v-model="form.phone" placeholder="请输入手机号" />
        </el-form-item>
        <el-form-item label="角色" prop="roleId">
          <el-select v-model="form.roleId" placeholder="请选择角色">
            <el-option v-for="r in roles" :key="r.id" :label="r.name" :value="r.id" />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleSubmit">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import { getUserList, createUser } from '@/api/user'

export default {
  data() {
    return {
      dialogVisible: false,
      form: { username: '', name: '', phone: '', roleId: null },
      roles: [],
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
        ],
        name: [
          { required: true, message: '请输入姓名', trigger: 'blur' }
        ],
        phone: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
        ],
        roleId: [
          { required: true, message: '请选择角色', trigger: 'change' }
        ]
      }
    }
  },
  methods: {
    handleAdd() { this.dialogVisible = true },
    async handleSubmit() {
      this.$refs.formRef.validate(async (valid) => {
        if (!valid) return
        const res = await createUser(this.form)
        if (res.code === 200) {
          this.$message.success('新增成功')
          this.dialogVisible = false
          this.loadList()
        } else {
          this.$message.error(res.msg)
        }
      })
    }
  }
}
</script>
```

### 1.2 后端 Controller + Service

```java
// UserController.java
@PostMapping("/api/user/create")
public Result create(@Validated @RequestBody UserCreateDTO dto) {
    return userService.create(dto);
}

// UserService.java
public Result create(UserCreateDTO dto) {
    if (userMapper.selectCount(new QueryWrapper<User>().eq("username", dto.getUsername())) > 0) {
        throw new BusinessException("用户名已存在");
    }
    User user = new User();
    BeanUtils.copyProperties(dto, user);
    user.setStatus(0);
    userMapper.insert(user);
    return Result.success("新增成功");
}
```

---

## 二、从代码提取的测试点

| 测试点 | 来源 | 类型 |
|--------|------|------|
| 用户名为空 | `rules.username[0].required` | 异常 - 必填 |
| 用户名 < 3 字符 | `rules.username[1].min` | 边界值 |
| 用户名 > 20 字符 | `rules.username[1].max` | 边界值 |
| 姓名为空 | `rules.name[0].required` | 异常 - 必填 |
| 手机号为空 | `rules.phone[0].required` | 异常 - 必填 |
| 手机号格式错误 | `rules.phone[1].pattern` | 异常 - 格式 |
| 角色未选 | `rules.roleId[0].required` | 异常 - 必填 |
| 用户名重复 | `Service: if exists` | 异常 - 业务 |
| 正常新增 | 主流程 | 正常路径 |

---

## 三、测试用例文档

### 用户管理 - 新增功能测试用例

**功能模块**：用户管理 - 新增
**涉及页面**：`/system/user`
**涉及接口**：`POST /api/user/create`

---

#### TC-USER-001：新增用户 - 正常新增

| 项 | 内容 |
|----|------|
| 用例编号 | TC-USER-001 |
| 用例标题 | 新增用户 - 所有字段填写正确 |
| 优先级 | P0 |
| 前置条件 | 1. 已登录管理员账号<br>2. 在用户管理页面 |
| 测试步骤 | 1. 点击"新增"按钮<br>2. 用户名输入 `testuser001`<br>3. 姓名输入 `测试用户`<br>4. 手机号输入 `13800138000`<br>5. 角色选择 `普通用户`<br>6. 点击"保存"按钮 |
| 预期结果 | 1. 弹窗关闭<br>2. 页面提示 `新增成功`<br>3. 用户列表中出现 `testuser001` |
| 测试数据 | username=testuser001, name=测试用户, phone=13800138000, role=普通用户 |

---

#### TC-USER-002：新增用户 - 用户名为空

| 项 | 内容 |
|----|------|
| 用例编号 | TC-USER-002 |
| 用例标题 | 用户名不填写，其他字段正常 |
| 优先级 | P0 |
| 前置条件 | 1. 已登录管理员账号<br>2. 打开了新增弹窗 |
| 测试步骤 | 1. 用户名留空<br>2. 其他字段正常填写<br>3. 点击"保存" |
| 预期结果 | 1. 用户名输入框下方显示 `请输入用户名`<br>2. 弹窗不关闭<br>3. 不发送保存请求 |
| 测试数据 | username=(空), 其他字段正常 |

---

#### TC-USER-003：新增用户 - 用户名过短

| 项 | 内容 |
|----|------|
| 用例编号 | TC-USER-003 |
| 用例标题 | 用户名输入 2 个字符 |
| 优先级 | P1 |
| 前置条件 | 已打开新增弹窗 |
| 测试步骤 | 1. 用户名输入 `ab`<br>2. 其他正常<br>3. 点击保存 |
| 预期结果 | 用户名下方显示 `长度在 3 到 20 个字符` |
| 测试数据 | username=ab |

---

#### TC-USER-004：新增用户 - 手机号格式错误

| 项 | 内容 |
|----|------|
| 用例编号 | TC-USER-004 |
| 用例标题 | 手机号输入非手机号格式 |
| 优先级 | P0 |
| 前置条件 | 已打开新增弹窗 |
| 测试步骤 | 1. 手机号输入 `12345`<br>2. 其他正常<br>3. 点击保存 |
| 预期结果 | 手机号下方显示 `手机号格式不正确` |
| 测试数据 | phone=12345 |

---

#### TC-USER-005：新增用户 - 角色未选

| 项 | 内容 |
|----|------|
| 用例编号 | TC-USER-005 |
| 用例标题 | 角色下拉不选择 |
| 优先级 | P1 |
| 前置条件 | 已打开新增弹窗 |
| 测试步骤 | 1. 角色不选择<br>2. 其他正常<br>3. 点击保存 |
| 预期结果 | 角色下方显示 `请选择角色` |
| 测试数据 | roleId=null |

---

#### TC-USER-006：新增用户 - 用户名重复

| 项 | 内容 |
|----|------|
| 用例编号 | TC-USER-006 |
| 用例标题 | 使用已存在的用户名 |
| 优先级 | P0 |
| 前置条件 | 系统中已存在用户名 `admin` |
| 测试步骤 | 1. 用户名输入 `admin`<br>2. 其他正常<br>3. 点击保存 |
| 预期结果 | 页面提示 `用户名已存在` |
| 测试数据 | username=admin |

---

## 四、自动化测试脚本

### 4.1 目录结构

```
tests/
├── conftest.py
├── pages/
│   ├── __init__.py
│   ├── base_page.py
│   ├── login_page.py
│   └── user_manage_page.py
├── test_user/
│   └── test_user_create.py
├── pytest.ini
└── .env
```

### 4.2 conftest.py

```python
import os
import pytest
from dotenv import load_dotenv
from pages.login_page import LoginPage

load_dotenv()
BASE_URL = os.getenv("BASE_URL", "http://localhost:8080")


@pytest.fixture(scope="session")
def base_url():
    return BASE_URL


@pytest.fixture(scope="function")
def logged_in_page(page, base_url):
    """每个测试前自动登录"""
    login_page = LoginPage(page, base_url)
    login_page.navigate()
    login_page.login(os.getenv("LOGIN_USERNAME"), os.getenv("LOGIN_PASSWORD"))
    yield page


@pytest.fixture
def test_data():
    import uuid
    return {
        "username": f"testuser_{uuid.uuid4().hex[:8]}",
        "name": "自动化测试用户",
        "phone": "13800138000",
        "role": "普通用户",
    }
```

### 4.3 pages/base_page.py

```python
from playwright.sync_api import Page, expect


class BasePage:
    def __init__(self, page: Page, base_url: str = ""):
        self.page = page
        self.base_url = base_url

    def navigate(self, path: str = ""):
        self.page.goto(f"{self.base_url}{path}")

    def get_field(self, label: str):
        return self.page.get_by_label(label)

    def get_button(self, name: str):
        return self.page.get_by_role("button", name=name)

    def click_button(self, name: str):
        self.get_button(name).click()

    def fill_field(self, label: str, value: str):
        self.get_field(label).fill(value)

    def expect_toast(self, text: str):
        """断言 Element UI el-message 提示"""
        expect(self.page.locator(".el-message").filter(has_text=text)).to_be_visible()

    def expect_form_error(self, field_label: str, error_text: str):
        """断言 Element UI 表单校验错误"""
        form_item = self.get_field(field_label).locator(
            "xpath=ancestor::*[contains(@class,'el-form-item')]"
        )
        expect(form_item.locator(".el-form-item__error")).to_have_text(error_text)

    def select_element_dropdown(self, field_label: str, option_text: str):
        """操作 Element UI 下拉"""
        self.get_field(field_label).click()
        self.page.get_by_role("option", name=option_text).click()
```

### 4.4 pages/login_page.py

```python
from pages.base_page import BasePage


class LoginPage(BasePage):
    def navigate(self):
        self.navigate("/login")

    def login(self, username: str, password: str):
        self.fill_field("用户名", username)
        self.fill_field("密码", password)
        self.click_button("登录")
        self.page.wait_for_url(lambda url: "/login" not in url)
```

### 4.5 pages/user_manage_page.py

```python
from playwright.sync_api import expect
from pages.base_page import BasePage


class UserManagePage(BasePage):
    def navigate(self):
        self.navigate("/system/user")

    def open_create_dialog(self):
        self.click_button("新增")

    def fill_user_form(self, data: dict):
        self.fill_field("用户名", data["username"])
        self.fill_field("姓名", data["name"])
        self.fill_field("手机号", data["phone"])
        self.select_element_dropdown("角色", data["role"])

    def submit_form(self):
        self.click_button("保存")

    def expect_dialog_closed(self):
        """断言弹窗已关闭"""
        expect(self.page.locator(".el-dialog")).to_have_count(0)

    def expect_user_in_list(self, username: str):
        """断言用户出现在列表"""
        expect(self.page.locator(".el-table").get_by_text(username).first).to_be_visible()
```

### 4.6 test_user/test_user_create.py

```python
"""
用户新增功能 - E2E 测试
对应测试用例：TC-USER-001 ~ TC-USER-006
"""
import pytest
from pages.user_manage_page import UserManagePage


class TestUserCreate:
    """用户新增功能测试"""

    @pytest.mark.smoke
    def test_001_create_success(self, logged_in_page, base_url, test_data):
        """TC-USER-001：正常新增"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_toast("新增成功")
        page.expect_dialog_closed()
        page.expect_user_in_list(test_data["username"])

    @pytest.mark.smoke
    def test_002_empty_username(self, logged_in_page, base_url, test_data):
        """TC-USER-002：用户名为空"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["username"] = ""
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_form_error("用户名", "请输入用户名")

    @pytest.mark.regression
    def test_003_username_too_short(self, logged_in_page, base_url, test_data):
        """TC-USER-003：用户名过短（2 字符）"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["username"] = "ab"
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_form_error("用户名", "长度在 3 到 20 个字符")

    @pytest.mark.smoke
    def test_004_invalid_phone(self, logged_in_page, base_url, test_data):
        """TC-USER-004：手机号格式错误"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["phone"] = "12345"
        page.fill_user_form(test_data)
        page.submit_form()

        page.expect_form_error("手机号", "手机号格式不正确")

    @pytest.mark.regression
    def test_005_role_not_selected(self, logged_in_page, base_url, test_data):
        """TC-USER-005：角色未选"""
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        # 填写除角色外的所有字段
        page.fill_field("用户名", test_data["username"])
        page.fill_field("姓名", test_data["name"])
        page.fill_field("手机号", test_data["phone"])
        # 不选角色
        page.submit_form()

        page.expect_form_error("角色", "请选择角色")

    @pytest.mark.regression
    def test_006_duplicate_username(self, logged_in_page, base_url, test_data):
        """TC-USER-006：用户名重复"""
        # TODO: 确认测试环境已存在的用户名
        page = UserManagePage(logged_in_page, base_url)
        page.navigate()
        page.open_create_dialog()

        test_data["username"] = "admin"  # 已存在的用户名
        page.fill_user_form(test_data)
        page.submit_form()

        # 后端返回的错误提示
        page.expect_toast("用户名已存在")
```

---

## 五、待补充清单

| 序号 | 涉及用例 | 待补充内容 | 说明 |
|------|---------|-----------|------|
| 1 | 全部 | 测试环境地址 | .env 中的 BASE_URL 需替换为实际地址 |
| 2 | 全部 | 登录账号密码 | .env 中的 LOGIN_USERNAME/PASSWORD |
| 3 | TC-USER-006 | 已存在的用户名 | 需确认测试环境中 `admin` 是否存在 |
| 4 | 全部 | 角色选项 | 确认测试环境有哪些角色可选 |

---

## 六、运行测试

```bash
# 安装依赖
pip install pytest playwright pytest-playwright python-dotenv
playwright install chromium

# 运行所有用户测试
pytest tests/test_user/ -v

# 只跑冒烟测试（核心流程）
pytest tests/test_user/ -v -m smoke

# 带浏览器界面调试
pytest tests/test_user/ -v --headed --slowmo 500

# 失败时截图和视频
pytest tests/test_user/ -v --screenshot=only-on-failure --video=retain-on-failure
```
