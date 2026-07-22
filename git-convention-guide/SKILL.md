---
name: "git-convention-guide"
description: "Git操作规范与约定。触发：git提交/推送/分支操作、commit message编写、代码提交规范。"
---

# Git Convention Guide

团队 Git 操作规范，重点是 Commit Message 格式与提交流程；分支/推送规范作为补充。

## 安全红线

- **禁止自动 push**：commit 后不自动推送，等用户确认
- **禁止暴力操作**：`--force` / `--hard` / `-D` 除非用户明确要求
- **禁止提交敏感信息**：.env、密钥、密码、证书文件不得进入暂存区
- **禁止擅自添加 AI 身份信息**：commit message 严禁附加 AI 生成标记、`Co-authored-by`、`Generated with ...`、署名、emoji 广告等任何 AI/工具身份信息；只写符合规范的变更描述，不改动 author/committer 身份

## 文档索引 + 加载规则

| 编号 | 主题 | 关键条目 | 加载规则（用户意图 → 编号） |
|-----|------|---------|---------------------------|
| 01 | Commit Message 规范 | 格式模板 ｜ Type 类型表 ｜ version 规则 ｜ 描述规范 ｜ 单行优先 | 编写 commit message、判断 type、查 version 写法 → 读 01 |
| 02 | 提交流程与安全 | 5 步流程 ｜ 暂存规则 ｜ 多文件 commit 策略 ｜ 安全注意事项 ｜ 快速参考 | 执行 git commit（含多文件变更、不确定怎么暂存） → 读 01 → 02 |
| 03 | 分支操作 | 常驻环境分支（prod/pre/test/dev/hotfix） ｜ 临时分支命名（v1_8_1、feat_xxxxx） ｜ rebase 合并 ｜ 用完即删 | 涉及分支创建/切换/推送/合并、查分支命名、理解分支体系 → 读 03 |

> 简单查询（git status / diff / log）：直接执行，无需读文档。

## 团队分支约定要点（AI 必读）

本团队的分支管理有几条与通用 Git Flow **不同**的做法，AI 必须了解以免给出错误建议：

1. **分支按"环境/版本"命名，不用 `feature/` `fix/` 这类前缀**
   - 常驻环境分支：`prod` / `pre` / `test` / `dev` / `hotfix`
   - 临时工作分支：用版本号 `v1_8_1` 或功能描述 `feat_xxxxx`

2. **合并用 rebase，不用 merge --no-ff**
   - 保持提交历史线性
   - rebase 会改写历史，操作由用户人工执行

3. **临时分支用完即删**
   - 发版合并到 `prod` 后删除临时分支，不长期保留
   - 开发历史由 commit message 承载，旧分支无保留价值

4. **分支操作以人工灵活处理为主**
   - 默认由用户手动操作，AI 仅在明确要求时辅助
   - 不要主动建议"开 feature 分支""发 PR"等通用流程

## Commit Type 与分支的关系

Commit Type（8 种）描述**单次提交做了什么**，分支描述**当前在哪个环境/版本上工作**，两者独立：

- 临时工作分支（如 `v1_8_1`）上可以出现任意 commit type——`[feat]` 为主，穿插 `[fix ]` `[opti]` `[refa]` `[styl]` `[test]` 都正常
- `[doc]` `[misc]` 通常直接在当前工作分支提交，不单独开分支
- 不需要为每种 commit type 对应开分支

## 条目速览（补充索引钩子）

**01 条目速览**：见文档索引行；核心是 type 4 字符对齐、version 沿用项目既有风格、单行优先。

**02 条目速览**：5 步流程（状态→分析→暂存→提交→确认） ｜ ⚠️只 add 相关文件、禁止 `git add -A` ｜ 多文件变更按"一个逻辑改动 = 一个 commit"拆分 ｜ 大 diff 先 `--stat` 看概览 ｜ ⚠️commit 前必 diff 查敏感信息 ｜ 禁止擅自署名

**03 条目速览**：常驻分支 prod/pre/test/dev/hotfix ｜ 临时分支用版本号 `v1_8_1` 或 `feat_xxxxx` ｜ ⚠️合并用 rebase 不用 merge ｜ ⚠️临时分支发版后删除 ｜ 分支操作默认人工处理
