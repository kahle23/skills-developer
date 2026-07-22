# 04 - 代码阅读策略

> 代码量大时怎么读、从哪开始、怎么快速建立全局认知。

## 通用方法论（语言无关）

### 三步阅读法

```
第 1 步：找入口 → 第 2 步：顺调用链 → 第 3 步：看命名反推业务
```

**第 1 步：找入口**

入口是"用户操作触发代码执行"的起点。找入口的方法：

| 维度 | 怎么找 |
|------|--------|
| 按模块名 | 用户说"资产领用"，就找 asset / borrow /领用 相关的目录和文件 |
| 按路由/Controller | 前端找路由配置，后端找 Controller 类 |
| 按表名 | 知道涉及哪张表，反向找操作这张表的代码 |

**第 2 步：顺调用链**

找到入口后，顺着调用关系往下读：

```
入口（路由/Controller）
  → 业务编排（页面组件/Service）
    → 数据操作（API 调用/Mapper/数据库）
```

> 只读主链路，不要一开始就深入每个方法的实现细节。先建立全局，再回头补细节。

**第 3 步：看命名反推业务**

代码命名往往直接暴露业务语义：

| 代码命名 | 反推业务 |
|---------|---------|
| `borrowAsset()` | 资产借用的业务 |
| `checkAvailable()` | 有可用性校验 |
| `notifyApprover()` | 涉及审批通知 |
| `status = APPROVED` | 有审批状态 |
| 表名 `oa_asset_record` | OA 系统的资产记录 |

> ⚠️ 当命名和你的理解冲突时，相信代码。命名可能有历史包袱，但代码行为是确定的。

---

## Vue 前端阅读路径

> Vue 项目是当前团队的前端技术栈。以下策略以 Vue 为主，但思路适用于其他前端框架。

### 阅读顺序

```
1. 路由配置（router）→ 找到页面组件路径
2. 页面组件（views）→ 理解页面结构和交互
3. API 封装（api 目录）→ 找到调用的后端接口
4. 组件（components）→ 理解复用的 UI 组件
```

### 各层关键看点

**1. 路由（router/index.js 或 router/modules/）**

```javascript
// 看这个，快速定位功能对应的页面
{
  path: '/asset/borrow',
  component: () => import('@/views/asset/borrow.vue'),
  meta: { title: '资产领用', permission: 'asset:borrow' }
}
```

看点：
- `path` → 前端 URL，用户文档里可能要提到
- `component` → 页面组件在哪
- `meta.title` → 菜单显示名称（**用户文档里用的就是这个**）
- `meta.permission` → 需要什么权限

**2. 页面组件（views/）**

重点看：
- `<template>` → 页面有哪些元素（表格、表单、按钮），**对应用户文档的操作步骤**
- `data()` / `setup()` → 页面有哪些状态
- `methods` → 用户操作触发了什么（点击按钮调哪个 API）
- 表单校验规则 `rules` → 用户文档里要提醒用户"必填项"

**3. API 封装（api/）**

```javascript
// api/asset.js
export function applyBorrow(data) {
  return request({
    url: '/asset/borrow/apply',
    method: 'post',
    data
  })
}
```

看点：
- `url` → 对应后端接口路径（**技术文档的接口列表来源**）
- 方法名 → 前端怎么理解这个接口的（业务语义）
- 参数 → 用户在界面上填了什么

**4. 组件（components/）**

一般不用深读，除非：
- 页面里有复杂的自定义组件（如自定义的审批流程组件）
- 组件名暗示了重要业务逻辑

### Vue 特有的提示信息来源

| 来源 | 能提取什么 | 文档用途 |
|------|-----------|---------|
| `this.$message.success('提交成功')` | 操作成功提示语 | 用户文档原文引用 |
| `this.$message.error('资产已被领用')` | 报错提示 | 用户文档"常见问题" |
| 表单 `rules` 中的 `required: true` | 必填字段 | 用户文档"怎么填"表格 |
| 按钮的 `disabled` 条件 | 什么情况不能操作 | 用户文档"前置条件" |

---

## 后端阅读路径（以 Java + Spring 系为例）

### 阅读顺序

```
1. Controller → 找到所有接口入口
2. Service → 理解核心业务逻辑
3. Mapper/Repository → 数据操作和涉及的表
4. Entity/DTO → 数据结构
```

### 各层关键看点

**1. Controller**

- `@RequestMapping` / `@GetMapping` / `@PostMapping` → 接口路径和方法
- 方法参数 → 接口入参
- 返回值 → 接口出参
- `@PreAuthorize` / 权限注解 → 接口权限要求

**2. Service**

- 这是业务逻辑的核心，**技术文档的"关键业务逻辑"章节主要来源**
- 重点找：条件判断（if/switch）、状态变更、事务操作、异常抛出
- 状态流转：找所有修改 `status` 字段的地方

**3. Mapper/Repository**

- SQL 语句 / QueryWrapper → 涉及哪些表、查询/修改了什么
- 复杂查询 → 可能暗示隐藏的业务规则

**4. Entity/DTO**

- 字段 → 数据存储章节的来源
- 字段注解（`@TableField`、`@NotNull`）→ 字段约束

### 后端框架特别说明

如果项目用了公司内部框架（如 Baibao/Kunlun），参考 `java-coding-standard` 的 23 号文档理解：
- `BaseServiceImpl` 的泛型 → 实体类型、主键类型
- `QueryMode` → 查询模式约定
- `DeleteStatus` → 软删除机制

---

## ⚠️ 前后端接口对应关系

> 梳理文档时，最关键的是把"前端调了哪个接口"和"后端哪个 Controller 处理"对应起来。

| 前端 API（api/*.js） | 后端 Controller | 业务动作 |
|---------------------|-----------------|---------|
| `applyBorrow()` → POST `/asset/borrow/apply` | `BorrowController.apply()` | 提交领用申请 |
| `approveBorrow()` → POST `/asset/borrow/approve` | `BorrowController.approve()` | 审批 |

> 当只有前端或只有后端代码时，这张表能帮你定位缺失的部分。

---

## 多语言扩展（预留）

本技能的代码阅读方法论是通用的。如果将来项目有其他语言/框架：

| 技术栈 | 入口在哪 | 阅读路径 |
|--------|---------|---------|
| Vue | router | router → views → api → components |
| React | 路由配置 | routes → pages → services → components |
| Angular | 路由 | routing → components → services |
| 原生/多页 | HTML/模板 | 入口页 → JS → 后端接口 |
| 小程序 | app.json 的 pages | pages → 组件 → api |

后端同理（Java/Spring、Python/Django、Node/Express 等），核心都是**找入口 → 顺调用链 → 看命名**。

需要新增某种技术栈的详细策略时，在本文件追加小节即可，不影响其他内容。
