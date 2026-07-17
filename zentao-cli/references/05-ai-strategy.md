# 05 — AI 使用策略与意图识别

## 输出格式选择

| 场景 | 格式 | 说明 |
|------|------|------|
| 展示给用户看 | 不加 `--format`（默认 markdown） | 列表输出 Markdown 表格，单对象输出列表 |
| AI 需要程序化解析（提取 ID、字段判断、后续操作） | `--format=json` | 返回结构化 `data` + `pager` |
| 需要服务端原始字段 | `--format=raw` | 字段名可能与服务端一致（如 `products`、`recTotal`） |

> 默认输出已将 HTML 转换为 Markdown（受 `htmlToMarkdown` 配置控制）。

## 交互确认

AI 场景下执行删除等需要确认的操作时，加 `--yes` 跳过确认提示：

```bash
zentao bug delete 1 --yes
zentao product delete 1,2 --yes     # 批量删除
```

## 不知道 ID 时

先查列表获取 ID，再操作具体对象：

```bash
zentao product --pick=id,name           # 查看产品列表
zentao bug --product=1 --pick=id,title  # 查看 Bug 列表
zentao bug 42                           # 查看具体 Bug
```

## 写操作前确认

执行创建、更新、删除等写操作前，先向用户确认操作内容（要操作的对象、字段、值）。用户明确要求不确认时可跳过。

> 更新操作无需手动先查再传完整参数：CLI 会先 GET 当前对象，把未显式传入的字段用现值填充后再 PUT，避免禅道 PUT 覆盖未提交字段导致清空。因此只需传想改的字段。

## 意图识别表

| 用户意图 | CLI 命令 |
|---------|---------|
| 所有产品/项目/项目集 | `zentao product` / `zentao project` / `zentao program` |
| 进行中的项目 | `zentao project --filter='status:doing'` |
| 某产品的 Bug | `zentao bug --product=<id>` |
| 某执行的任务 | `zentao task --execution=<id>` |
| 某执行的全部任务（含已完成/已关闭） | `zentao task --execution=<id> --status=all --all` |
| 已关闭执行中的任务 | 先 `zentao execution <id>` 探测确认存在，再 `zentao task --executionID=<id> --status=all --search="关键词" --search-fields=name,desc --all`（详见 06、08） |
| 创建/新增 Bug | `zentao bug create ...` |
| 解决 Bug | `zentao bug resolve <id>` |
| 关闭 Bug | `zentao bug close <id>` |
| 激活 Bug | `zentao bug activate <id>` |
| 创建需求 | `zentao story create ...` |
| 变更/关闭/激活需求 | `zentao story change/close/activate <id>` |
| 业务需求 | `zentao epic ...`（同 story） |
| 用户需求 | `zentao requirement ...`（同 story） |
| 创建/启动/完成/关闭任务 | `zentao task create/start/finish/close ...` |
| 测试用例 | `zentao testcase ...` |
| 测试单 | `zentao testtask ...` |
| 产品计划 | `zentao productplan ...` |
| 版本/Build | `zentao build ...` |
| 发布 | `zentao release ...` |
| 反馈 | `zentao feedback ...` |
| 工单 | `zentao ticket ...` |
| 用户列表 | `zentao user` |
| 当前用户信息 / 切换账号 | `zentao profile` |
