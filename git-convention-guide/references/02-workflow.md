# 提交流程与安全规范

## 标准 5 步提流程

### 步骤 1：查看状态（并行执行）

```bash
git status
git diff
git log --oneline -10
```

- `git status`：查看有哪些文件变更
- `git diff`：查看具体改了什么
- `git log --oneline -10`：查看最近 commit 风格和当前版本号

**变更文件较多时**：先用 `git diff --stat` 看概览，再针对关键文件单独 `git diff <file>` 查看细节，避免一次性刷屏。

### 步骤 2：分析变更，拟定 commit message

根据 diff 内容判断变更类型：

- 新增功能/接口 → `[feat]`
- 修复问题/Bug → `[fix ]`
- 小幅优化 → `[opti]`；大幅结构调整 → `[refa]`（判定见 01 文档）
- 配置/依赖/构建 → `[misc]`

从 `git log --oneline -10` 获取版本号；无版本号则省略。

### 步骤 3：暂存文件

```bash
git add <具体文件路径>
```

- **只暂存本次相关的文件**，不要 `git add -A` 或 `git add .`
- 不要暂存敏感文件（.env、credentials 等）
- 不要暂存无关文件（IDE 配置、临时文件等）

### 步骤 4：提交

有版本号时：

```bash
git commit -m "[type][version] 描述信息"
```

无版本号时：

```bash
git commit -m "[type] 描述信息"
```

### 步骤 5：确认结果

```bash
git status
git log --oneline -3
```

## 多文件 commit 策略

当一次工作涉及多个文件时，按"**一个逻辑改动 = 一个 commit**"原则拆分：

| 情况 | 处理方式 |
|------|---------|
| 多文件属于同一逻辑改动（如新增功能 + 相关配置） | **合并为一个 commit**，描述概括这次改动 |
| 多文件属于多个独立逻辑（如改 Bug A 又顺手优化了 B） | **拆成多个 commit**，每个 commit 只含对应逻辑的文件 |

**操作示例**（拆分提交）：

```bash
# 先提交逻辑 A 的文件
git add fileA1 fileA2
git commit -m "[fix ][v1.8.1] 修复资产导入空指针问题"

# 再提交逻辑 B 的文件
git add fileB1
git commit -m "[opti][v1.8.1] 优化导入结果汇总查询"
```

> 实在拿不准时，**合并优于拆分**——避免把一个连贯的改动拆得过于细碎。

## 安全注意事项

1. **提交前**：必须先 `git diff` 查看变更内容，确认无敏感信息
2. **不要提交**：.env 文件、API 密钥、数据库密码、证书文件
3. **冲突处理**：不要强行覆盖，先 `git pull --rebase` 再处理
4. **确认再推**：commit 后不自动 push，等用户确认
5. **禁止擅自署名**：commit message 只写变更描述，严禁附加任何 AI/工具身份信息，包括：
   - `Co-authored-by: xxx` 之类的协作署名
   - `🤖 Generated with ...`、`Created by AI` 等生成标记
   - 尾部签名、广告、emoji 宣传语
   - 也不得通过 `--author` / `user.name` / `user.email` 篡改提交者身份
   如用户明确要求添加署名，才可按用户指定内容添加

## 快速参考

| 操作 | 命令 |
|------|------|
| 查看状态 | `git status` |
| 查看变更 | `git diff` |
| 变更概览 | `git diff --stat` |
| 查看历史 | `git log --oneline -N` |
| 暂存文件 | `git add <file1> <file2>` |
| 提交 | `git commit -m "[type][ver] msg"` |
| 推送 | `git push` |
| 拉取 | `git pull --rebase` |
| 撤销暂存 | `git restore --staged <file>` |
| 撤销修改 | `git restore <file>` |
