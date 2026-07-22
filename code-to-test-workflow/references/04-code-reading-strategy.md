# 04 - 代码阅读策略（测试视角）

> 从代码中提取**测试点**的策略。和 code-to-doc-workflow 的代码阅读策略方向不同：
> - 文档导向：关注"功能怎么实现的"
> - **测试导向**：关注"哪里可以测、测什么、断言依据是什么"

---

## 一、前端框架自动检测

> 在读代码之前，先确认前端框架。不同框架的元素结构、组件库、定位策略不同。

### 检测方法

| 检测项 | Vue | React | Angular | jQuery/传统 |
|--------|-----|-------|---------|-------------|
| package.json | `vue` 依赖 | `react` 依赖 | `@angular/core` | `jquery` 依赖 |
| 文件后缀 | `.vue` | `.jsx`/`.tsx` | `.ts`（component） | `.html`+`.js` |
| 根元素属性 | `id="app"` + `data-v-xxx` | `id="root"` + `data-reactroot` | `ng-version="x.x"` | 无特殊属性 |
| 模板语法 | `v-if` `v-for` `{{ }}` | `{condition && <JSX>}` | `*ngIf` `*ngFor` `{{ }}` | 无模板语法 |
| UI 组件库 | Element UI / Element Plus / Ant Design Vue | Ant Design / Material UI | NG-ZORRO | Bootstrap / 自定义 |

### 快速判断流程

```
1. 找 package.json → 看 dependencies
   ├── 有 "vue" → Vue 项目
   ├── 有 "react" → React 项目
   ├── 有 "@angular/core" → Angular 项目
   └── 有 "jquery" 且无以上 → 传统 jQuery

2. 找 UI 组件库（看 dependencies 或 import）
   ├── Vue: "element-ui" / "element-plus" / "ant-design-vue"
   ├── React: "antd" / "@mui/material"
   └── 据此判断下拉、日期等组件的实现方式（影响交互操作，详见 05）
```

### Vue 版本进一步区分

```bash
# Element UI（Vue 2）→ CSS 类名 el-select, el-date-editor
# Element Plus（Vue 3）→ CSS 类名同上，但某些行为不同
# 看 package.json：
"element-ui": "^2.x"     → Element UI (Vue 2)
"element-plus": "^1.x"   → Element Plus (Vue 3)
```

> **关键影响**：不同 UI 组件库的下拉选择、日期选择交互方式不同，直接影响 05 中的操作写法。

---

## 二、前端阅读路径（找测试点）

### 阅读顺序

```
1. 路由配置 → 找到页面 URL 和组件路径
2. 页面组件 → 找表单字段、校验规则、按钮交互
3. API 封装 → 找调用的后端接口（断言依据）
4. 组件库确认 → 确定交互操作方式（原生 vs UI 组件库）
```

### 2.1 路由 → 找到页面

```javascript
// router/index.js
{
  path: '/system/user',
  component: () => import('@/views/system/user/index'),
  meta: { title: '用户管理', permission: 'system:user:list' }
}
```

提取：
- 页面 URL：`/system/user` → 测试脚本的 `navigate` 路径
- 权限要求：`system:user:list` → 需要登录有权限的账号

### 2.2 页面组件 → 找表单字段和校验 ⭐

这是**测试用例的核心来源**。

**找表单字段：**

```vue
<el-form :model="form" :rules="rules" ref="formRef">
  <el-form-item label="用户名" prop="username">
    <el-input v-model="form.username" placeholder="请输入用户名" />
  </el-form-item>

  <el-form-item label="手机号" prop="phone">
    <el-input v-model="form.phone" />
  </el-form-item>

  <el-form-item label="角色" prop="roleId">
    <el-select v-model="form.roleId" placeholder="请选择角色">
      <el-option v-for="r in roles" :key="r.id" :label="r.name" :value="r.id" />
    </el-select>
  </el-form-item>
</el-form>
```

提取的测试点：

| 字段 | 类型 | 校验 | 测试用例 |
|------|------|------|---------|
| 用户名 | 文本输入 | 见 rules | 必填测试、长度测试 |
| 手机号 | 文本输入 | 见 rules | 格式测试 |
| 角色 | **UI 组件库下拉**（el-select） | 见 rules | 必选测试、选项联动 |

**找校验规则：**

```javascript
rules: {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '长度在 3 到 20 个字符', trigger: 'blur' }
  ],
  phone: [
    { required: true, message: '请输入手机号', trigger: 'blur' },
    { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
  ]
}
```

提取的测试点：

| 规则 | 生成的测试用例 |
|------|--------------|
| `required: true` + `message: '请输入用户名'` | TC: 用户名为空 → 预期出现"请输入用户名" |
| `min: 3, max: 20` | TC: 输入 2 字符 → 报错；输入 20 字符 → 通过 |
| `pattern: /^1[3-9]\d{9}$/` | TC: 输入 "123" → 报错"手机号格式不正确" |

> ⚠️ **message 文本是断言的关键**。测试脚本中要断言这个文本出现。

**找按钮交互：**

```vue
<el-button type="primary" @click="handleSubmit" :loading="submitting">保存</el-button>
<el-button @click="handleCancel">取消</el-button>
<el-button type="danger" @click="handleDelete" :disabled="!selectedRow">删除</el-button>
```

提取：
- 按钮文本 → 测试脚本中 `get_by_role("button", name="保存")`
- `:disabled` 条件 → 测试用例：未选中行时删除按钮应不可点

**找提交逻辑：**

```javascript
methods: {
  async handleSubmit() {
    this.$refs.formRef.validate(async (valid) => {
      if (!valid) return           // ← 校验不过不发请求
      const res = await createUser(this.form)  // ← 调的哪个接口
      if (res.code === 200) {
        this.$message.success('新增成功')     // ← 成功提示文本
      } else {
        this.$message.error(res.msg)          // ← 失败提示
      }
    })
  }
}
```

提取：
- 调用接口：`createUser` → POST `/api/user/create`（从 api/ 目录确认）
- 成功提示：`新增成功` → **断言依据**
- 失败提示：`res.msg` → 后端返回的消息

### 2.3 API 封装 → 找后端接口

```javascript
// api/user.js
export function createUser(data) {
  return request({ url: '/api/user/create', method: 'post', data })
}
```

提取：
- 接口路径：`/api/user/create`
- 请求方法：POST
- 请求参数：`data`（即表单数据）

> 这个接口路径在测试中可用于：`page.wait_for_response("**/api/user/create")`

### 2.4 确认组件库 → 决定交互方式

| 组件 | 原生 HTML | UI 组件库 |
|------|----------|----------|
| 下拉 | `<select><option>` | `<el-select><el-option>` |
| 日期 | `<input type="date">` | `<el-date-picker>` |
| 上传 | `<input type="file">` | `<el-upload>` |

> ⚠️ **这直接决定 05 中的操作方式**。原生用 Playwright 原生方法，UI 组件库需要模拟点击交互。

---

## 三、后端阅读路径（找校验逻辑）

### 阅读顺序

```
1. Controller → 找入参校验注解
2. Service → 找业务校验逻辑
3. Entity/DTO → 找字段约束
```

### 3.1 Controller → 入参校验

```java
@PostMapping("/create")
public Result create(@Validated @RequestBody UserCreateDTO dto) {
    return userService.create(dto);
}
```

```java
// UserCreateDTO.java
public class UserCreateDTO {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20个字符")
    private String username;

    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
}
```

> 如果前端有同样的校验，前端校验会先触发。后端校验是兜底，适合接口测试覆盖。

### 3.2 Service → 业务校验 ⭐

```java
public Result create(UserCreateDTO dto) {
    // 1. 唯一性校验
    if (userMapper.selectCount(new QueryWrapper<User>()
            .eq("username", dto.getUsername())) > 0) {
        throw new BusinessException("用户名已存在");    // ← 测试点！
    }

    // 2. 角色校验
    if (dto.getRoleId() != null && !roleExists(dto.getRoleId())) {
        throw new BusinessException("角色不存在");      // ← 测试点！
    }

    // 3. 创建用户
    User user = new User();
    BeanUtils.copyProperties(dto, user);
    user.setStatus(0);  // 待激活
    userMapper.insert(user);

    return Result.success("新增成功");
}
```

提取的业务校验测试点：

| 校验逻辑 | 测试用例 |
|---------|---------|
| 用户名唯一性 | TC: 用已存在的用户名 → 报"用户名已存在" |
| 角色存在性 | TC: 传不存在的角色 ID → 报"角色不存在" |
| 初始状态 | TC: 新增后状态应为"待激活"（status=0） |

### 3.3 状态流转 → 测试状态变更

```java
// Service 中所有改 status 的地方
user.setStatus(1);  // 激活
user.setStatus(2);  // 禁用
```

> 状态流转是重要的测试场景，每个状态变更都要有用例覆盖。

---

## 四、⚠️ 前后端接口对应（断言依据）

> 最关键的一步：把前端表单字段、后端接口、数据库字段对应起来。这张表是测试断言的基础。

| 前端表单字段 | 前端校验 | 后端接口参数 | 后端校验 | 数据库字段 | 断言依据 |
|------------|---------|------------|---------|-----------|---------|
| 用户名 (username) | required + 3-20 字符 | username | @NotBlank + @Size | sys_user.username | 列表中出现 / 报错提示 |
| 手机号 (phone) | pattern | phone | @Pattern | sys_user.phone | 报错提示"格式不正确" |
| 角色 (roleId) | required | roleId | 业务校验存在性 | sys_user.role_id | 下拉选中成功 |

> 当只有前端或只有后端代码时，这张表能帮你定位缺失的校验信息。

---

## 五、测试点提取清单

读完代码后，用以下清单确保没有遗漏：

### 表单测试点

- [ ] 每个必填字段为空 → 报错
- [ ] 每个格式校验 → 不匹配格式报错
- [ ] 每个长度限制 → 超长/过短报错
- [ ] 每个下拉/选择 → 不选报错、选了成功
- [ ] 正常填写所有字段 → 成功

### 业务测试点

- [ ] 唯一性校验 → 重复值报错
- [ ] 关联数据存在性 → 不存在的关联 ID 报错
- [ ] 状态流转 → 每个状态变更有对应用例
- [ ] 权限控制 → 无权限操作被拒绝

### 交互测试点

- [ ] 按钮 disabled 条件 → 不满足条件时不可点
- [ ] 级联/联动 → 父级变化后子级刷新
- [ ] 提交时 loading → 防重复提交
- [ ] 成功/失败提示 → 断言提示文本

### 数据测试点

- [ ] 提交后数据正确 → 列表/数据库验证
- [ ] 提交后字段值正确 → 特别是默认值、计算值
