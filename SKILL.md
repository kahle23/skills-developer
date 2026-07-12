---
name: "java-coding-guide"
description: "Java编码指南，包含AI编码工作流、编码规范、并发编程、API设计、设计模式、性能优化、常用工具库(hutool/guava/apache-commons)及公司内部包使用规范。适用于Java 1.6+项目，帮助编写高质量、风格统一的Java代码。当用户要求编写、审查或优化Java代码时触发此技能。"
---

# Java 编码指南

本技能为 AI 提供 Java 编码的最佳实践和风格规范，适用于 **Java 1.6 及以上版本**的项目，不绑定特定框架或技术栈。

---

## 文档编号规则

| 区间 | 用途 |
|------|------|
| 01~02 | 工作流与模板（AI 编码前必读） |
| 03~08 | 核心编码规范（通用 Java 知识） |
| 21~40 | 第三方框架/工具库（hutool、guava 等） |
| 41~60 | 公司内部包/框架（按项目扩展） |
| 99 | 其他（兜底收录） |

---

## 使用指南

在编写 Java 代码前，请根据场景查阅对应文档：

### 工作流与模板

| 场景 | 文档 |
|------|------|
| **编码前必读**：先搜已有代码、再复用、最后写新的 | [01-ai-workflow.md](references/01-ai-workflow.md) |
| 收到代码示例时，按模板提炼并写入技能 | [02-code-examples-template.md](references/02-code-examples-template.md) |

### 核心编码规范

| 场景 | 文档 |
|------|------|
| 命名、类结构、注释、异常、日志、空值处理 | [03-code-standards.md](references/03-code-standards.md) |
| List/Set/Map 操作、Stream API、泛型 | [04-collection-and-stream.md](references/04-collection-and-stream.md) |
| 建造者、策略、模板方法、观察者等模式 | [05-design-patterns.md](references/05-design-patterns.md) |
| 字符串拼接、集合容量、装箱、缓存 | [06-performance.md](references/06-performance.md) |
| 线程池、锁、原子类、CompletableFuture | [07-concurrency.md](references/07-concurrency.md) |
| RESTful URL、请求参数、返回值、分页、错误码 | [08-api-design.md](references/08-api-design.md) |

### 第三方框架/工具库

| 场景 | 文档 |
|------|------|
| @Data/@Slf4j/@NoArgsConstructor 等注解 | [21-lombok.md](references/21-lombok.md) |
| StrUtil/CollUtil/DateUtil/JSONUtil/BeanUtil | [22-hutool.md](references/22-hutool.md) |
| ImmutableList/Preconditions/Cache/Hashing | [23-guava.md](references/23-guava.md) |
| StringUtils/CollectionUtils/BeanUtils | [24-apache-commons.md](references/24-apache-commons.md) |

### 公司内部包

| 场景 | 文档 |
|------|------|
| 内部自研包/框架的使用规范（41~60 区间，按需扩展） | [41-internal-packages.md](references/41-internal-packages.md) |

### 其他

| 场景 | 文档 |
|------|------|
| 无法归入以上类别的规范 | [99-others.md](references/99-others.md) |

---

## 快速决策树

```
编码前必读 → 01-ai-workflow（先搜再读最后写）
│
├─ 写新类/方法 → 03-code-standards（命名、结构、注释）
├─ 处理集合数据 → 04-collection-and-stream
├─ 设计类/接口架构 → 05-design-patterns
├─ 优化运行效率 → 06-performance
├─ 多线程/并发/异步 → 07-concurrency
├─ 设计 API 接口 → 08-api-design
│
├─ 减少样板代码 → 21-lombok
├─ 字符串/日期/JSON/Bean 操作 → 22-hutool
├─ 不可变集合/缓存/哈希 → 23-guava
├─ 字符串/集合通用工具 → 24-apache-commons
│
├─ 公司内部包 → 41~60（查看 41-internal-packages 了解规则）
│
├─ 收到代码示例 → 02-code-examples-template（提炼规则并写入技能）
└─ 无法归类的内容 → 99-others（兜底收录）
```

---

## 核心原则

1. **先找，再读，最后写** — 编码前先搜索项目中是否已有类似功能，能复用就不重写。详见 [01-ai-workflow.md](references/01-ai-workflow.md)
2. **命名即注释** — 名字要自解释，减少不必要的注释
3. **方法短小** — 一个方法只做一件事，不超过 50 行
4. **善用工具库** — 不要重复造轮子，优先使用成熟的工具类
5. **防御式编程** — 参数校验前置，空值处理明确
6. **保持一致性** — 团队统一风格比"最佳"风格更重要

---

*本技能适用于 Java 1.6+ 项目，涵盖 Java SE 标准库、主流第三方工具库及公司内部包。*
