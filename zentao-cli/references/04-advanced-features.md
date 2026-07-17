# 04 — 高级功能

## 配置管理 — `config`

通过 `zentao config set <key> <value>` 设置默认配置，`zentao config get [key]` 查看（不指定 key 返回全部）。

```bash
zentao config set defaultOutputFormat json
zentao config get defaultOutputFormat
zentao config get                  # 查看所有配置
```

支持的配置项：

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `defaultOutputFormat` | 默认输出格式：`markdown` / `json` / `raw` | `markdown` |
| `lang` | 界面语言：`zh-cn` / `zh-tw` / `en` | `zh-cn` |
| `defaultRecPerPage` | 默认分页大小 | 20 |
| `insecure` | 是否忽略 SSL/TLS 证书验证 | `false` |
| `timeout` | 请求超时时间（ms） | 10000 |
| `htmlToMarkdown` | 是否将对象属性中的 HTML 转换为 Markdown | `true` |
| `batchFailFast` | 批量操作出错时是否立即停止 | `false` |
| `silent` | 是否启用静默模式 | `false` |
| `jsonPretty` | JSON 输出是否格式化（添加空格） | `false` |
| `autoSetWorkspace` | 是否自动设置工作区 | `false` |
| `pagers` | 各模块分页配置（product/project/execution 等） | - |

## 版本信息 — `version`

```bash
zentao version
# Zentao CLI: 0.1.8
# Zentao Server: 22.1 (https://zentao.example.com)
```

已登录时会同时输出禅道服务端版本。

## 自动补全 — `autocomplete`

生成 shell 自动补全脚本（支持 zsh / bash / fish）：

```bash
zentao autocomplete zsh
zentao autocomplete bash
zentao autocomplete fish
```

## 静默模式 — `--silent`

启用后不再输出普通信息，仅在出错时输出错误信息：

```bash
zentao product create --silent
```

也可全局开启：`zentao config set silent true`。

## 批量操作与错误处理

删除/创建/更新支持逗号分隔多个 ID 批量操作：

```bash
zentao product delete 1,2 --yes
```

批量操作默认在某个对象出错后跳过并继续。如希望出错立即停止，加 `--batch-fail-fast`：

```bash
zentao product delete 1,2,3,4,5 --yes --batch-fail-fast
```

也可全局开启：`zentao config set batchFailFast true`。

JSON 输出会返回 `success` / `failed` / `skipped` 分组结果，便于程序化判断。

## 工作区

工作区用于记住用户上次访问的产品、项目、执行，设置后列表命令可省略 `--product` / `--project` / `--execution` 等范围参数。

- 通过 `autoSetWorkspace` 配置项控制是否自动设置（默认 `false`）。
- 相关错误码见 07（E4001 未找到工作区 / E4002 未设置工作区）。
- 设置工作区后，`zentao story`、`zentao bug`、`zentao task` 等会自动带上所属产品/执行。

## MCP 服务

zentao-cli 提供 MCP 服务，便于在 AI Agent 中以工具方式调用禅道。

### 手动启动

```bash
npx -y zentao-cli mcp
```

### 一键配置到 AI Agent

```bash
zentao add-mcp           # 交互式选择 Agent（Cursor/Claude Desktop/Claude Code/Windsurf/Cline/Trae/VS Code/Cherry Studio/OpenCode/Codex）
```

### 手动配置示例

无需提前安装 zentao-cli，在 Agent 的 MCP 配置中添加：

```json
{
  "mcpServers": {
    "zentao-cli": {
      "command": "npx",
      "args": ["-y", "zentao-cli", "mcp"],
      "env": {
        "ZENTAO_URL": "https://zentao.example.com",
        "ZENTAO_ACCOUNT": "admin",
        "ZENTAO_PASSWORD": "123456"
      }
    }
  }
}
```

## 安装技能到 AI Agent — `add-skill`

将 zentao-cli 作为技能一键安装到 AI Agent（Claude Code / Cursor / Cherry Studio / Codex / OpenCode / VS Code / Antigravity / Gemini）：

```bash
zentao add-skill                # 交互式选择
zentao add-skill claude-code    # 直接指定 Agent
zentao add-skill all            # 全部安装
```

一键安装、登录并配置：

```bash
pnpm install -g zentao-cli && zentao login && zentao add-skill all
```
