# 05 - 交互操作手册

> **核心参考文件**。生成 Playwright 脚本时，所有具体的浏览器交互操作都在这里查。
> 按 UI 组件类型分节，每节给出可直接复制的代码模板。

---

## 一、元素定位优先级

> 选对定位方式，测试稳定性提升 50%。

```
get_by_label()       ← 表单字段首选（有 label 关联时）
get_by_role()        ← 按钮/链接/标题等交互元素首选
get_by_placeholder() ← 没有 label 但有 placeholder
get_by_text()        ← 非交互元素（提示文字、列表数据）
get_by_test_id()     ← 最后手段
```

```python
# ✅ 推荐
page.get_by_label("用户名").fill("admin")
page.get_by_role("button", name="保存").click()

# ⚠️ UI 组件库的特殊情况（详见下文各组件章节）
# Element UI 的 el-select 不是原生 <select>，不能用 select_option
# Element UI 的 el-date-picker 不是原生 <input type="date">，需要点击日历面板
```

---

## 二、文本输入

适用：input、textarea、number 类型的输入框。

### 2.1 基本输入

```python
# 通过 label 定位（推荐）
page.get_by_label("用户名").fill("admin")

# 通过 placeholder 定位（无 label 时）
page.get_by_placeholder("请输入用户名").fill("admin")

# 清空再输入
page.get_by_label("用户名").fill("")     # 先清空
page.get_by_label("用户名").fill("admin")
```

### 2.2 密码输入

```python
page.get_by_label("密码").fill("secret123")
```

### 2.3 数字输入（Element UI 的 el-input-number）

```python
# el-input-number 本质还是 input，用 fill
page.get_by_label("数量").fill("100")

# 或通过按钮加减
page.get_by_label("数量").locator(".el-input-number__increase").click()
```

### 2.4 文本域（textarea）

```python
page.get_by_label("备注").fill("这是一段备注内容")
```

### 2.5 断言输入值

```python
from playwright.sync_api import expect

expect(page.get_by_label("用户名")).to_have_value("admin")
```

---

## 三、⚠️ 下拉选择

> **最容易踩坑的交互**。原生 `<select>` 和 UI 组件库下拉操作方式完全不同。

### 3.1 原生 `<select>`（直接用 select_option）

```html
<!-- 原生 HTML -->
<select>
  <option value="1">管理员</option>
  <option value="2">普通用户</option>
</select>
```

```python
# 按 value 选择
page.get_by_label("角色").select_option("1")

# 按显示文本（label）选择
page.get_by_label("角色").select_option(label="普通用户")

# 按索引选择
page.get_by_label("角色").select_option(index=1)

# 多选
page.get_by_label("标签").select_option(["tag1", "tag2"])
```

### 3.2 ⚠️ Element UI / Element Plus 的 el-select

> **不能用 select_option**！el-select 是用 div 模拟的，需要点击触发 + 点击选项。

```html
<!-- Element UI 结构 -->
<div class="el-select">
  <div class="el-input">  <!-- 触发器 -->
    <input type="text" /> <!-- 显示选中值 -->
  </div>
</div>
<!-- 选项面板（可能渲染在 body 底部） -->
<ul class="el-select-dropdown__list">
  <li class="el-select-dropdown__item">管理员</li>
  <li class="el-select-dropdown__item">普通用户</li>
</ul>
```

```python
# ✅ 正确：点击触发器 → 点击选项
def select_element_dropdown(page, field_label, option_text):
    """操作 Element UI 下拉"""
    # 1. 点击下拉触发器
    page.get_by_label(field_label).click()
    # 2. 点击选项（选项面板渲染在 body 底部，有 role="option" 或 class 标识）
    page.get_by_role("option", name=option_text).click()
    # 如果 role="option" 定位不到，用文本定位
    # page.get_by_text(option_text, exact=True).click()
```

**封装为页对象方法：**

```python
# pages/base_page.py 中添加
def select_element_dropdown(self, field_label: str, option_text: str):
    """
    操作 Element UI / Element Plus 的自定义下拉。
    """
    self.get_field(field_label).click()
    self.page.get_by_role("option", name=option_text).click()
```

### 3.3 ⚠️ 可搜索的下拉（filterable）

Element UI 的 `el-select` 带 `filterable` 时，点击后是输入框，需要输入再选：

```python
def select_filterable_dropdown(page, field_label, search_text, option_text):
    """可搜索下拉"""
    trigger = page.get_by_label(field_label)
    trigger.click()
    # 输入搜索
    trigger.fill(search_text)
    # 等待选项加载后点击
    page.get_by_role("option", name=option_text).click()
```

### 3.4 ⚠️ 级联选择器（el-cascader）

```python
def select_cascader(page, field_label, path: list):
    """
    级联选择，path 是各级选项的文本。
    例: select_cascader(page, "地区", ["江苏省", "南京市", "鼓楼区"])
    """
    page.get_by_label(field_label).click()
    for level, option in enumerate(path):
        panels = page.locator(".el-cascader-menu")
        panels.nth(level).get_by_text(option).click()
```

### 3.5 多选下拉（el-select multiple）

```python
def select_multiple_dropdown(page, field_label, options: list):
    """多选下拉"""
    page.get_by_label(field_label).click()
    for option in options:
        page.get_by_role("option", name=option).click()
    # 点击空白处关闭
    page.locator("body").click()
```

---

## 四、⚠️ 日期选择

> Element UI 的 `el-date-picker` 需要操作日历面板。

### 4.1 原生日期输入

```html
<input type="date" />
```

```python
page.get_by_label("日期").fill("2024-03-15")
```

### 4.2 ⚠️ Element UI 单日期选择

```python
from datetime import datetime

def pick_date(page, field_label, date_str):
    """
    Element UI 日期选择。
    date_str 格式: "2024-03-15"
    """
    # 1. 点击触发器打开日历
    page.get_by_label(field_label).click()

    # 2. 在日历面板中点击日期
    date = datetime.strptime(date_str, "%Y-%m-%d")
    date_text = str(date.day)  # 日历中显示的就是数字

    # 定位日历面板
    date_panel = page.locator(".el-date-picker")
    # 需要先确认年和月是否正确，如果不匹配需要翻页
    # 简单做法：直接点击匹配的日期文本
    date_panel.get_by_text(date_text, exact=True).click()
```

### 4.3 ⚠️ 日期范围选择

```python
def pick_date_range(page, field_label, start_date, end_date):
    """日期范围"""
    page.get_by_label(field_label).click()
    # 点击开始日期
    pick_date(page, None, start_date)
    # 点击结束日期
    pick_date(page, None, end_date)
```

### 4.4 替代方案：直接通过 JS 设值（兜底）

> 当日历面板定位困难时，可以直接通过 JS 设置值并触发事件。

```python
def set_date_via_js(page, selector, value):
    """通过 JS 直接设值（兜底方案）"""
    page.evaluate(f"""
        const el = document.querySelector('{selector}');
        el.value = '{value}';
        el.dispatchEvent(new Event('input', {{ bubbles: true }}));
        el.dispatchEvent(new Event('change', {{ bubbles: true }}));
    """)
```

---

## 五、文件上传

### 5.1 原生文件上传（input[type=file]）

```python
# 单文件
page.get_by_label("上传文件").set_input_files("/path/to/file.pdf")

# 多文件
page.locator("input[type=file]").set_input_files([
    "/path/to/file1.pdf",
    "/path/to/file2.pdf",
])
```

### 5.2 Element UI 的 el-upload

> el-upload 通常隐藏了真实的 input[type=file]，用按钮触发。

```python
def upload_file_element_ui(page, button_text, file_path):
    """
    Element UI 上传。
    button_text: 上传按钮文本，如 "点击上传"
    """
    # 方式1：点击上传按钮，监听文件选择器
    with page.expect_file_chooser() as fc_info:
        page.get_by_role("button", name=button_text).click()
    file_chooser = fc_info.value
    file_chooser.set_files(file_path)

    # 方式2：直接定位隐藏的 input[type=file]
    # page.locator("input[type=file]").set_input_files(file_path)
```

### 5.3 拖拽上传

```python
def upload_by_drag(page, drop_area_locator, file_path):
    """拖拽上传"""
    page.set_input_files(drop_area_locator, file_path)
```

---

## 六、复选框 / 单选 / 开关

### 6.1 复选框（checkbox）

```python
# 勾选
page.get_by_role("checkbox", name="记住我").check()

# 取消勾选
page.get_by_role("checkbox", name="记住我").uncheck()

# 断言已勾选
from playwright.sync_api import expect
expect(page.get_by_role("checkbox", name="记住我")).to_be_checked()
```

### 6.2 单选按钮（radio）

```python
# 点击选中
page.get_by_role("radio", name="男").check()

# 断言选中
expect(page.get_by_role("radio", name="男")).to_be_checked()
```

### 6.3 Element UI 的 el-switch

```python
def toggle_switch(page, field_label, turn_on=True):
    """Element UI 开关"""
    switch = page.get_by_label(field_label)
    is_checked = switch.get_attribute("class")
    if "is-checked" in is_checked and not turn_on:
        switch.click()
    elif "is-checked" not in is_checked and turn_on:
        switch.click()
```

---

## 七、表格操作

### 7.1 表格数据断言

```python
from playwright.sync_api import expect

# 断言表格中存在某行数据
expect(page.get_by_role("cell", name="testuser01")).to_be_visible()

# 断言行数
rows = page.locator(".el-table__row")
expect(rows).to_have_count(5)

# 断言特定行的某列值
row = page.locator(".el-table__row").filter(has_text="testuser01")
expect(row.get_by_role("cell").nth(2)).to_have_text("测试用户")
```

### 7.2 表格分页

```python
# 跳转到指定页
page.get_by_role("button", name="2").click()  # 页码按钮

# 修改每页条数（Element UI 分页下拉）
page.locator(".el-pagination .el-select").click()
page.get_by_role("option", name="50 条/页").click()
```

### 7.3 行内操作按钮

```python
# 找到特定行，点击其中的操作按钮
row = page.locator(".el-table__row").filter(has_text="testuser01")
row.get_by_role("button", name="编辑").click()

# 或者直接用文本定位（如果按钮文本唯一）
page.get_by_role("button", name="编辑").click()
```

### 7.4 表格排序

```python
# 点击表头排序
page.get_by_role("columnheader", name="创建时间").click()
```

---

## 八、弹窗 / 对话框

### 8.1 确认弹窗（alert/confirm）

```python
# 原生 confirm 弹窗
page.on("dialog", lambda dialog: dialog.accept())

# 或者用 page.expect_dialog
with page.expect_dialog() as dialog_info:
    page.get_by_role("button", name="删除").click()
dialog = dialog_info.value
dialog.accept()
```

### 8.2 Element UI 确认弹窗（MessageBox）

```python
def confirm_element_messagebox(page, button_text="确定"):
    """
    Element UI 的 $confirm 弹窗（MessageBox）。
    弹窗是异步出现的，需要等待。
    """
    msgbox = page.locator(".el-message-box")
    expect(msgbox).to_be_visible()
    msgbox.get_by_role("button", name=button_text).click()
```

### 8.3 关闭弹窗/抽屉

```python
# 关闭 Dialog（点右上角 X）
page.locator(".el-dialog__close").click()

# 关闭抽屉（Drawer）
page.locator(".el-drawer__close-btn").click()

# 按 ESC 关闭
page.keyboard.press("Escape")
```

### 8.4 断言提示消息（Toast/Message）

```python
# Element UI 的 el-message（成功/失败提示）
# 自动消失，需要快速断言
def expect_message(page, text, msg_type=None):
    """
    断言 Element UI 消息提示。
    msg_type: "success" / "error" / "warning" / None(不限)
    """
    selector = ".el-message"
    if msg_type:
        selector += f".el-message--{msg_type}"
    expect(page.locator(selector).filter(has_text=text)).to_be_visible()
```

---

## 九、⚠️ 表单校验断言

> Element UI / Element Plus 表单校验：校验通过的字段 form-item 会加 `is-success`，
> 校验失败加 `is-error`，错误提示在 `.el-form-item__error`。

### 9.1 断言校验错误

```python
def expect_form_error(page, field_label, error_text):
    """
    断言某字段的校验错误提示。
    """
    # 找到字段所在的 form-item
    form_item = page.get_by_label(field_label).locator(
        "xpath=ancestor::*[contains(@class,'el-form-item')]"
    )
    # 断言 form-item 有错误类
    expect(form_item).to_have_class(".*is-error.*")  # 正则匹配
    # 断言错误文本
    expect(form_item.locator(".el-form-item__error")).to_have_text(error_text)
```

### 9.2 断言校验通过

```python
def expect_form_valid(page, field_label):
    """断言某字段校验通过（无错误提示）"""
    form_item = page.get_by_label(field_label).locator(
        "xpath=ancestor::*[contains(@class,'el-form-item')]"
    )
    expect(form_item.locator(".el-form-item__error")).to_have_count(0)
```

---

## 十、等待策略

### 10.1 自动等待（默认行为）

Playwright 的 `click()`、`fill()` 等操作**自动等待**元素出现、可见、可交互：

```python
# ✅ 不需要手动等待，Playwright 会自动等
page.get_by_role("button", name="保存").click()
```

### 10.2 需要手动等待的场景

| 场景 | 方法 |
|------|------|
| 等待网络请求完成 | `page.wait_for_response("**/api/user/list")` |
| 等待 URL 变化 | `page.wait_for_url("**/system/user")` |
| 等待 loading 遮罩消失 | `page.locator(".el-loading-mask").wait_for(state="hidden")` |
| 等待元素出现 | `page.locator(".result").wait_for(state="visible")` |
| 等待元素消失 | `page.locator(".loading").wait_for(state="hidden")` |

### 10.3 断言自带等待

```python
# expect 断言会自动重试，直到超时或条件满足
expect(page.get_by_text("新增成功")).to_be_visible()  # 最多等 5 秒

# 设置超时
expect(page.get_by_text("新增成功")).to_be_visible(timeout=10000)  # 10 秒
```

### 10.4 ⚠️ 禁止的做法

```python
# ❌ 绝对不要用固定等待
import time
time.sleep(3)  # 这是反模式！

# ❌ 也不要用 page.wait_for_timeout
page.wait_for_timeout(3000)  # 只有调试时才用
```

---

## 十一、登录态保持

> 每次测试都重新登录很慢。可以保存登录状态复用。

### 11.1 保存登录状态

```python
# 登录后保存 storage state
page.goto("http://localhost:8080/login")
page.get_by_label("用户名").fill("admin")
page.get_by_label("密码").fill("admin123")
page.get_by_role("button", name="登录").click()
page.wait_for_url("**/dashboard")

# 保存状态
page.context.storage_state(path="auth_state.json")
```

### 11.2 复用登录状态

```python
# conftest.py
@pytest.fixture(scope="session")
def browser_context_args(browser_context_args):
    return {
        **browser_context_args,
        "storage_state": "auth_state.json",
    }
```

> pytest-playwright 会自动使用 `browser_context_args` fixture 创建 context。

---

## 十二、多标签页 / iframe

### 12.1 弹出窗口（新标签页）

```python
# 点击会打开新窗口的链接
with page.context.expect_page() as new_page_info:
    page.get_by_role("link", name="查看详情").click()
new_page = new_page_info.value
new_page.wait_for_load_state()

# 在新页面操作
new_page.get_by_role("button", name="确认").click()
```

### 12.2 iframe

```python
# 定位 iframe
iframe = page.frame_locator("iframe[src='/editor']")
iframe.get_by_role("button", name="保存").click()
```

---

## 十三、键盘操作

```python
# 逐字符输入
page.get_by_label("搜索").press_sequentially("hello")

# 按特殊键
page.keyboard.press("Enter")
page.keyboard.press("Escape")
page.keyboard.press("Tab")

# 组合键
page.keyboard.press("Control+A")  # 全选
page.keyboard.press("Control+C")  # 复制

# 在某个元素上按键
page.get_by_label("搜索").press("Enter")
```

---

## 十四、调试技巧

### 14.1 录制操作（Codegen）

```bash
# 启动录制器，操作浏览器自动生成代码
playwright codegen http://localhost:8080
```

> 在不知道怎么定位时，先用 codegen 录制一遍，看生成的代码。

### 14.2 慢动作运行

```bash
pytest --headed --slowmo 1000  # 每个操作间隔 1 秒
```

### 14.3 调试模式（Inspector）

```bash
PWDEBUG=1 pytest tests/test_user/test_user_create.py
```

会打开 Playwright Inspector，可以暂停、单步、查看定位器。

### 14.4 失败截图和视频

```bash
# pytest.ini 中配置
pytest --screenshot=only-on-failure --video=retain-on-failure
```
