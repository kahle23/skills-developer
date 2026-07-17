---
name: "git-convention-guide"
description: "Git操作规范与约定。触发：git提交/推送/分支操作、commit message编写、代码提交规范。"
---

# Git Convention Guide

团队 Git 操作规范，重点是 Commit Message 格式与提流程。

## 安全红线

- **禁止自动 push**：commit 后不自动推送，等用户确认
- **禁止暴力操作**：`--force` / `--hard` / `-D` 除非用户明确要求
- **禁止提交敏感信息**：.env、密钥、密码、证书文件不得进入暂存区
- **禁止擅自添加 AI 身份信息**：commit message 严禁附加 AI 生成标记、`Co-authored-by`、`Generated with ...`、署名、emoji 广告等任何 AI/工具身份信息；只写符合规范的变更描述，不改动 author/committer 身份

## 文档索引

| 编号 | 主题 | 何时读取 |
|-----|------|---------|
| 01 | Commit Message 规范（格式/类型表/version/描述规则/示例） | 编写 commit message 时必读 |
| 03 | 提流程与安全（5步流程/暂存规则/安全注意事项/快速参考） | 执行 git commit 时必读 |
| 04 | 分支操作（命名规范/推送/禁止操作） | 涉及分支创建、推送、切换时读取 |

## 01 条目速览

- **Type 类型**：feat ｜ fix(补空格对齐) ｜ opti ｜ refa ｜ doc ｜ styl ｜ test ｜ misc
- ⚠️ type 标记统一占 4 字符位，`fix` 需补尾部空格写为 `[fix ]`，其余无需补空格
- **Version**：从最近 commit 获取，保持一致；⚠️无版本号时直接省略 version 段
- **描述**：中文、≤72字符、说明做了什么、不以句号结尾
- ⚠️ **单行优先**：commit message 默认只写一行，除非变更确实重要且一行说不清才写多行正文

## 加载规则

- **编写 commit message**：读 01
- **执行 git commit**：读 01 → 03
- **分支操作**：读 04
- **简单查询**（git status / diff / log）：直接执行，无需读文档
