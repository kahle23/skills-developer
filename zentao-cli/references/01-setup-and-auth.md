# 01 — 安装与认证

## 安装

```bash
npm install -g zentao-cli
# 或 bun install -g zentao-cli
# 或 pnpm install -g zentao-cli
# 或免安装运行：npx zentao-cli
```

如果用户没有安装，引导用户进行全局安装使用。如果系统存在 bun 或 pnpm，则优先使用 bun 或 pnpm 进行全局安装。

> 禅道服务端需 22.0 及以上版本，否则会报 E1008。

## 登录

首次执行任意 `zentao` 命令会自动提示输入禅道 URL、用户名和密码完成登录。也可显式登录：

```bash
zentao login -s https://zentao.example.com -u admin -p 123456
```

| 参数 | 说明 |
|------|------|
| `-s` | 禅道服务器 URL |
| `-u` | 用户名 |
| `-p` | 密码 |

登录成功后凭证（禅道 URL、账号、Token，不含密码）缓存在 `~/.config/zentao/zentao.json`，后续无需重复登录。

### 环境变量

CLI 支持从环境变量读取连接信息。**显式命令行参数优先级高于环境变量**：

| 变量 | 说明 |
|------|------|
| `ZENTAO_URL` | 禅道服务地址 |
| `ZENTAO_ACCOUNT` | 用户账号 |
| `ZENTAO_PASSWORD` | 密码 |
| `ZENTAO_TOKEN` | 直接指定 Token（有此变量可省略密码） |

> 即使设置了环境变量，之前手动登录过的身份信息可能仍被优先使用。若希望忽略已有身份、强制使用环境变量重新验证，执行 `zentao login --useEnv`。

## 账户管理

支持登录多个禅道服务，或同一服务下登录多个账号。最后一次登录的账号默认设为当前账户。

### 查看与切换账户

```bash
zentao profile                       # 查看当前用户信息及所有已登录账户（* 标记当前）
zentao profile dev1@https://zentao.example.com   # 切换当前用户
```

### 退出登录

```bash
zentao logout                        # 退出当前用户
zentao logout dev1@https://zentao.example.com    # 退出指定用户
```

退出后会从 `~/.config/zentao/zentao.json` 中移除对应用户信息。

## 自定义配置文件

默认配置保存在 `~/.config/zentao/zentao.json`。可通过全局选项或环境变量指定自定义路径：

```bash
zentao --config /path/to/zt.json profile           # 通过 --config 选项
ZENTAO_CONFIG_FILE=~/work/zt.json zentao product   # 通过环境变量
```

- 路径支持 `~` 展开与相对路径（相对当前工作目录）。
- `--config` 与 `ZENTAO_CONFIG_FILE` 同时存在时，`--config` 优先。
- 指定后所有读写均作用于该文件，不再使用默认路径。

## 凭证安全（红线）

- **不要在对话里收集账号密码**。用户尚未登录时，引导其在终端执行 `zentao login`，或执行任意 `zentao` 命令触发首次自动登录提示，由用户自行输入凭证。
- **严禁读取本地凭证**：`ZENTAO_PASSWORD` / `ZENTAO_TOKEN` 环境变量、`~/.config/zentao/zentao.json` 配置文件均不可读取。所有禅道数据通过 `zentao` 命令获取，凭证由 CLI 内部处理。
