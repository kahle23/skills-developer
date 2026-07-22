# Skills Developer — 开发者技能集

一套面向 AI 编程助手的**开发者技能合集**。把表设计、功能设计、代码规范、工具库用法、项目协作工具等工程经验沉淀为结构化文档，让 AI 在协助开发时输出风格统一、质量稳定、贴合团队约定的代码与方案。

> 每个技能都是一个独立的目录，包含 `SKILL.md`（技能元数据与加载规则）和 `references/`（分主题的参考文档）。AI 按需查阅，不会一次性读取全部内容。

---

## 为什么需要这些技能

AI 写代码常见的问题：命名风格漂移、重复造轮子、忽略已有工具方法、异常处理与日志风格不一致、API 设计随意。这套技能通过把团队约定固化成文档，让 AI：

- **先复用再新建** — 编码前先搜索项目已有代码，避免重复造轮子
- **风格统一** — 命名、结构、注释、异常、日志遵循一致约定
- **善用工具库** — 优先使用 Hutool、Guava、Apache Commons 等成熟组件
- **规范输出** — API 遵循 RESTful、统一返回值、分页与错误码体系
- **设计先行** — 表结构、功能方案在动手前就有章可循

---

## 技能清单

| 技能 | 说明 | 状态 |
|------|------|------|
| [java-coding-guide](./java-coding-guide) | Java 编码规范与工具库指南（不绑定特定框架） | ✅ 已完成 |
| [python-coding-guide](./python-coding-guide) | Python 编码规范与最佳实践（不绑定特定框架） | ✅ 已完成 |
| [db-design-guide](./db-design-guide) | 数据库表结构设计规范（命名、标准字段、索引、SQL模板、禁止项） | ✅ 已完成 |
| [feature-design-workflow](./feature-design-workflow) | 功能设计文档工作流（需求概述→功能梳理→表结构→接口设计→业务逻辑，含阶段门禁） | ✅ 已完成 |
| [zentao-cli](./zentao-cli) | 禅道 CLI 使用指南（查询/操作禅道数据：产品、项目、执行、需求、Bug、任务等） | ✅ 已完成 |
| [git-convention-guide](./git-convention-guide) | Git 操作规范与约定（commit message 格式、提交流程、分支操作） | ✅ 已完成 |
| [code-to-doc-guide](./code-to-doc-guide) | 代码转文档指南（从前后端代码逆向生成技术文档 + 用户使用文档） | ✅ 已完成 |
| [code-to-test-guide](./code-to-test-guide) | 代码转测试指南（从前后端代码生成测试用例 + Playwright E2E 自动化脚本） | ✅ 已完成 |

> 技能之间相互独立，可按需选用；也鼓励把它们组合使用，覆盖从设计到编码到测试的完整链路。

---

## 目录结构

```
skills-developer/
│
├── README.md              # 本文件
├── java-coding-guide/     # Java 编码指南
├── python-coding-guide/   # Python 编码指南
├── db-design-guide/       # 数据库表设计规范
├── feature-design-workflow/  # 功能设计文档工作流（含阶段门禁）
├── zentao-cli/            # 禅道 CLI 使用指南
├── git-convention-guide/  # Git 操作规范与约定
├── code-to-doc-guide/     # 代码转文档指南（技术文档 + 用户文档）
└── code-to-test-guide/    # 代码转测试指南（测试用例 + Playwright 脚本）
```

每个技能目录内部结构一致：`SKILL.md`（元数据与加载规则）+ `references/`（分主题参考文档，按编号区间组织）。具体文档清单见各技能的 `SKILL.md` 文档索引。

文档编号区间约定（各技能通用）：

- **01~02**：工作流与模板（AI 编码前必读）
- **03~08**：核心规范（通用知识）
- **21~40**：第三方框架/工具库
- **41~60**：公司内部包/框架（按需创建）
- **99**：其他（兜底收录，含完整示例）

---

## 如何使用

### 1. 安装技能

将技能目录（如 `java-coding-guide/`）放入你的项目中，AI 会自动识别 `SKILL.md` 并按场景查阅对应文档。

### 2. 典型场景

| 你对 AI 说的话 | AI 会参考的文档 |
|---------------|----------------|
| "帮我写一个用户服务类" | 01 → 03 → 07 → 22 |
| "设计一个导出数据的接口" | 01 → 08 → 05 |
| "优化这段代码的性能" | 06 → 04 |
| "用 Hutool 处理日期" | 22 |
| "帮我写并发任务处理" | 07 → 06 |
| "帮我设计一张业务表" | db: 01 → 03 → 02 |
| "写个 Python 脚本处理 CSV" | python: 01 → 04 → 06 |
| "这个功能怎么设计文档" | feature-workflow: 01 → 02 |
| "帮我写个规范的 commit" | git: 01 |
| "帮我梳理这个功能的文档" | code-to-doc: 01 → 02/03 |
| "这段代码怎么用，写个用户说明" | code-to-doc: 01 → 03 |
| "帮我给这个功能写测试" | code-to-test: 01 → 02 → 03 → 05 |
| "基于代码生成自动化测试脚本" | code-to-test: 01 → 03 → 05 |

### 3. 定制技能

所有技能文档都是纯 Markdown，直接编辑即可调整风格偏好：

- **改命名风格** → 编辑 `03-code-standards.md`
- **停用某个工具库** → 删除或注释对应文档
- **加入公司内部包** → 按 `41-xxx.md` 命名新建文档
- **喂代码示例自学** → 发送代码片段，AI 按 `02` 模板提炼规则

详细定制方式见各技能目录下的 `README.md`（java-coding-guide、python-coding-guide）或 `SKILL.md`。

---

## 新增技能

每个技能是一个独立目录，建议结构：

```
{skill-name}/
├── README.md        # 技能说明：能做什么、如何使用、如何定制
├── SKILL.md         # 元数据：name、description、加载规则、文档索引
└── references/      # 分主题参考文档（按编号区间组织）
```

`SKILL.md` 头部需包含 frontmatter：

```markdown
---
name: {skill-name}
description: 技能描述与触发词，用于 AI 判断何时加载
---
```

命名建议：技能目录与 `name` 一致，用小写英文加连字符，如 `db-design-guide`、`feature-design-workflow`。

---

## 路线图

- [x] **java-coding-guide** — Java 编码规范与工具库
- [x] **python-coding-guide** — Python 编码规范与最佳实践
- [x] **db-design-guide** — 数据库表结构设计规范
- [x] **feature-design-workflow** — 功能设计工作流（含阶段门禁）
- [x] **zentao-cli** — 禅道 CLI 使用指南
- [x] **git-convention-guide** — Git 操作规范与约定
- [x] **code-to-doc-guide** — 代码转文档指南（技术文档 + 用户文档）
- [x] **code-to-test-guide** — 代码转测试指南（测试用例 + Playwright 脚本）
- [ ] **code-review-guide** — 代码审查清单与规范

---

## 许可

内部使用，按需自定义。
