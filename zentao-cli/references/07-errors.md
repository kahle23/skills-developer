# 07 — 错误代码与排查手册

命令出错时会输出 `Error(Exxxx)` 信息。下表列出全部错误码、原因与处理方式。

## 认证与配置（10xx）

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| E1001 | 未提供有效的禅道服务地址、用户名和密码或 TOKEN | 执行 `zentao login -s <url> -u <user> -p <pwd>` |
| E1002 | 禅道服务地址无法访问 | 检查 URL 是否正确、网络是否通畅 |
| E1003 | 用户名和密码不正确 | 核对账号密码后重新 `zentao login` |
| E1004 | Token 已失效 | 重新 `zentao login`，或提供新的 `ZENTAO_TOKEN` |
| E1005 | 配置文件损坏或无法读取 | 检查配置文件路径（默认 `~/.config/zentao/zentao.json`，可能被 `--config` 或 `ZENTAO_CONFIG_FILE` 覆盖） |
| E1006 | 未找到指定的用户配置 | 先 `zentao login -s <url> -u <user> -p <pwd> -t <token>` 登录 |
| E1007 | 指定的用户配置不存在 | 通过 `zentao profile` 查看可用配置 |
| E1008 | 当前禅道版本不受支持 | 升级禅道服务端至 22.0 及以上 |

## API 调用（20xx）

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| E2001 | 未找到指定的模块 | 执行 `zentao help` 查看支持的模块 |
| E2002 | 未找到指定的对象 | 检查对象 ID 是否正确 |
| E2003 | 缺少必要参数 | 执行 `zentao <module> help` 或 `zentao <module> <action> help` 查看必要参数 |
| E2004 | 参数值无效 | 检查参数格式是否正确 |
| E2005 | 不支持的操作 | 执行 `zentao <module> help` 查看支持的操作 |
| E2006 | 当前用户没有权限 | 提示用户检查权限 |
| E2007 | `--data` 参数中的 JSON 格式无效 | 检查 JSON 字符串格式 |
| E2008 | 禅道服务端返回错误 | 查看详细错误信息中的 Url / Status / serverResponse |
| E2009 | 选项值无效 | 按提示 reason 检查选项值 |

## 数据处理（30xx）

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| E3001 | `--pick` 指定的字段不存在 | 检查字段名，先用 `--format=json` 看可用字段 |
| E3002 | `--filter` 表达式格式无效 | 检查过滤条件格式（见 03） |
| E3003 | `--sort` 表达式格式无效 | 检查排序条件格式（见 03） |

## 工作区（40xx）

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| E4001 | 未找到指定的工作区 | 通过 `zentao workspace ls` 查看可用工作区 |
| E4002 | 当前未设置工作区 | 通过 `zentao workspace set` 设置工作区 |

## 网络通信（50xx）

| 错误码 | 含义 | 处理方式 |
|--------|------|---------|
| E5001 | 请求超时 | 检查网络连接或禅道服务是否正常；可调大 `timeout` 配置 |
| E5002 | SSL/TLS 证书验证失败 | 检查禅道服务地址；自签证书可临时 `zentao config set insecure true` |

## 常见错误示例

```bash
# 未登录访问数据
$ zentao product
Error(E1001): 必须提供有效的禅道服务地址、用户名和密码

# JSON 格式输出错误
$ zentao product --format=json
{
    "error": {
        "code": "1001",
        "message": "必须提供有效的禅道服务地址、用户名和密码"
    }
}
```

## 批量操作错误结果

批量操作加 `--format=json` 会返回分组结果，便于程序化判断成功/失败/跳过：

```json
{
    "status": "success",
    "result": {
        "success": [1, 2],
        "failed": [3],
        "skipped": [4, 5],
        "errors": [
            {"objectID": 3, "error": {"code": "2006", "message": "当前用户没有权限执行此操作"}}
        ]
    }
}
```
