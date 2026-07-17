# Skills Developer — 开发者技能集

一套面向 AI 编程助手的**开发者技能合集**。把表设计、功能设计、代码规范、工具库用法等工程经验沉淀为结构化文档，让 AI 在协助开发时输出风格统一、质量稳定、贴合团队约定的代码与方案。

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
| [java-coding-guide](./java-coding-guide) | Java 编码规范与工具库指南（Java 1.6+，不绑定框架） | ✅ 已完成 |
| db-design-guide | 数据库表设计规范（命名、字段、索引、主键策略、分库分表） | 🚧 规划中 |
| feature-design-guide | 功能设计指南（需求拆解、模块划分、流程图、接口契约） | 🚧 规划中 |

> 技能之间相互独立，可按需选用；也鼓励把它们组合使用，覆盖从设计到编码的完整链路。

---

## 目录结构

```
skills-developer/
│
├── README.md                       # 本文件
│
└── java-coding-guide/              # Java 编码指南技能
    ├── README.md                   # 技能说明与使用示例
    ├── SKILL.md                    # 技能元数据（名称/描述/加载规则/文档索引）
    └── references/                 # 分主题参考文档
        ├── 01-ai-workflow.md           # AI 编码工作流（先搜再读最后写）
        ├── 02-code-examples-template.md# 代码示例学习模板
        ├── 03-code-standards.md        # 代码规范
        ├── 04-collection-and-stream.md # 集合与 Stream
        ├── 05-design-patterns.md       # 设计模式
        ├── 06-performance.md           # 性能优化
        ├── 07-concurrency.md           # 并发编程
        ├── 08-api-design.md            # API 设计规范
        ├── 21-lombok.md                # Lombok
        ├── 22-hutool.md                # Hutool
        ├── 23-baibao.md                # Baibao
        ├── 24-guava.md                 # Guava
        └── 25-apache-commons.md        # Apache Commons
```

文档编号区间约定（各技能通用）：

- **01~02**：工作流与模板（AI 编码前必读）
- **03~08**：核心规范（通用知识）
- **21~40**：第三方框架/工具库
- **41~60**：公司内部包/框架（按需创建）
- **99**：其他（兜底收录）

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

### 3. 定制技能

所有技能文档都是纯 Markdown，直接编辑即可调整风格偏好：

- **改命名风格** → 编辑 `03-code-standards.md`
- **停用某个工具库** → 删除或注释对应文档
- **加入公司内部包** → 按 `41-xxx.md` 命名新建文档
- **喂代码示例自学** → 发送代码片段，AI 按 `02` 模板提炼规则

详细定制方式见各技能目录下的 `README.md`。

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

命名建议：技能目录与 `name` 一致，用小写英文加连字符，如 `db-design-guide`、`feature-design-guide`。

---

## 路线图

- [x] **java-coding-guide** — Java 编码规范与工具库
- [ ] **db-design-guide** — 数据库表设计规范
- [ ] **feature-design-guide** — 功能设计指南
- [ ] **code-review-guide** — 代码审查清单与规范
- [ ] **git-workflow-guide** — 分支策略与提交规范

---

## 许可

内部使用，按需自定义。
