# 08 — 注意事项

> 本文件记录实战中踩过的坑与反直觉行为，**持续补充**。新增条目请保持「问题 → 解决方案」结构，并标注重要程度。

## 操作习惯

- **不确定就查帮助**：不确定模块参数时先 `zentao <module> help`；不确定操作参数时先 `zentao <module> <action> help`。
- **写操作前先确认**：创建/更新/删除前向用户确认对象与字段；用户要求不确认时可跳过。
- **更新只需传改动字段**：CLI 会先 GET 当前对象补全未传字段再 PUT，无需手动先查再传完整参数。
- **AI 删除加 `--yes`**：AI 场景跳过交互确认提示。
- **程序化处理用 `--format=json`**：展示给用户用默认 Markdown；提取 ID/字段判断用 JSON。

## 列表与状态相关（重要）

### 已关闭的执行不出现在执行列表中

`zentao execution --all`、`--params='{"browseType":"all"}'`、`--filter='status=closed'` **均无法返回已关闭的执行**。需通过 `zentao execution <id>` 逐个探测 ID 断层来发现已关闭的执行（详见 06「搜索已关闭执行中的任务」）。

### 任务列表默认隐藏已关闭任务

`--status` 参数默认为 `unclosed`，已完成（done）和已关闭（closed）的任务默认不返回。搜索任务时务必加 `--status=all` 才能覆盖全部状态。

### 全局搜索不存在

zentao CLI 没有跨执行/跨项目的全局任务搜索功能。需要先获取所有执行 ID（含已关闭的），再逐个执行搜索。

### browseType 常用值

`all`（全部）、`doing`（进行中）、`closed`（已关闭）、`wait`（未开始）等。具体可用值以 `zentao <module> help` 为准。

## 账户与配置

- **多账号切换**：`zentao profile` 查看所有已登录账户并切换当前用户；`zentao logout` 退出。
- **环境变量不生效**：若已手动登录过，身份信息可能优先于环境变量，用 `zentao login --useEnv` 强制使用环境变量。
- **自定义配置文件**：`--config <path>` 或 `ZENTAO_CONFIG_FILE` 环境变量指定，`--config` 优先。

## 数据处理

- **`--filter` 逗号语义**：同一 `--filter` 内逗号分隔 = AND；多个 `--filter` 参数 = OR。值含逗号时用引号包裹。
- **`--search` 多个 = OR**：多个 `--search` 参数为 OR 逻辑。
- **`--sort` 后缀**：`_asc` 升序、`_desc` 降序，必须带后缀。
- **`--pick` 字段名**：报 E3001 时先用 `--format=json` 查看可用字段名；支持 `.` 访问子字段。

## 网络与证书

- **自签证书报 E5002**：临时 `zentao config set insecure true` 跳过证书校验（仅限内网可信环境）。
- **请求超时 E5001**：调大 `timeout` 配置，如 `zentao config set timeout 30000`。

## 凭证安全（红线）

- 不在对话里收集账号密码，引导用户在终端执行 `zentao login`。
- 严禁读取 `ZENTAO_PASSWORD` / `ZENTAO_TOKEN` 环境变量和 `~/.config/zentao/zentao.json` 配置文件。

---

<!-- 新增注意事项模板：
### <标题>（重要 / 一般）

**问题**：<描述反直觉行为或坑点>

**解决方案**：<具体命令或步骤>
-->
