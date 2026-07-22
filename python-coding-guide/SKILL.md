---
name: "python-coding-guide"
description: "Python 编码规范与最佳实践。触发：Python 代码编写/审查/优化/重构、编码规范、类型提示、异常处理、数据结构选型、装饰器/上下文管理器、设计模式、性能优化、并发编程（threading/asyncio/multiprocessing）、pytest 测试、虚拟环境、依赖管理、代码格式化。"
---

# Python Coding Guide

面向 Python 3.9+，不绑定特定框架（Flask / FastAPI / Django 均适用）。本文档为 AI 编写 Python 代码的统一规范入口。

> **版本注意**：示例使用 `list[int]` / `dict[str, Any]` 等泛型语法（3.9+）、`X | Y` 联合类型（3.10+）、`dataclasses`（3.7+）。低版本项目用 `typing.List` / `typing.Union` 替代。

## 工作流入口

所有编码任务遵循「先搜再读最后写」，详见 01。收到用户代码示例时按 02 沉淀规范。

## 文档索引与触发钩子

| 编号 | 主题 | 关键条目（按需读取触发条件） |
|-----|------|------|
| 01 | 工作流：先搜再读最后写 | 编码前搜索已有实现｜复用策略判断｜新代码遵循已有风格 |
| 02 | 代码示例学习模板 | 收到用户代码示例时｜将示例沉淀为规范条目｜去重与冲突处理 |
| 03 | 命名/结构/文档字符串/异常/日志/类型提示/None安全 | snake_case命名｜⚠️禁止裸except｜⚠️不吞异常｜with管理资源｜类型提示必加｜返回空集合非None｜用is判断None |
| 04 | 列表/字典/集合/元组/推导式/切片/枚举 | list/dict/set选型｜⚠️列表乘法嵌套陷阱｜推导式vs生成器｜Counter/deque/defaultdict｜字典3.7+有序 |
| 05 | 设计模式 | dataclass｜策略(函数字典优先于Protocol)｜装饰器(functools.wraps)｜上下文管理器(contextmanager)｜生成器｜观察者(回调列表)｜单例(模块级) |
| 06 | 性能优化 | ⚠️字符串拼接用join｜set代替list查找｜推导式vs生成器｜⚠️循环内不重复编译正则｜内置函数优于手写循环｜lru_cache缓存 |
| 07 | 并发编程 | I/O密集用threading/asyncio｜CPU密集用multiprocessing｜ThreadPoolExecutor｜asyncio.gather｜⚠️GIL限制｜⚠️multiprocessing要main保护 |
| 08 | 测试与工具链 | pytest(fixture/parametrize/mock)｜mypy类型检查｜ruff格式化｜venv虚拟环境｜pyproject.toml｜.gitignore |
| 99 | 其他（兜底） | 零散知识收录 |

## 加载规则

简单任务（改名/抽取函数/小工具/项目已有成熟模式）跳过文档。其余按场景触发：

| 用户意图 | 读取文档 |
|---------|---------|
| 编写/修改任意 Python 代码（先搜项目已有实现） | 01 |
| 用户发来代码示例，希望沉淀风格 | 02 |
| 涉及命名、模块/类结构、文档字符串、异常处理、类型提示、日志、None 安全 | 03 |
| 列表/字典/集合/元组选型、推导式、切片、Counter/deque | 04 |
| dataclass、装饰器、上下文管理器、策略/生成器/单例模式 | 05 |
| 性能优化、字符串拼接、循环优化、查找性能、缓存 | 06 |
| 多线程/多进程/异步、ThreadPoolExecutor、asyncio | 07 |
| 编写测试、类型检查、代码格式化、虚拟环境、依赖管理 | 08 |

不确定时先读 01 判断。
