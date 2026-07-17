# 02 — 命令与模块

## 命令格式

### 简写方式（推荐）

```
zentao <模块名> [操作] [参数]
```

| 操作 | 命令 |
|------|------|
| 列表 | `zentao <module>` |
| 详情 | `zentao <module> <id>` |
| 创建 | `zentao <module> create --field=value` |
| 更新 | `zentao <module> update <id> --field=value` |
| 删除 | `zentao <module> delete <id>` |
| 动作 | `zentao <module> <action> <id>` |
| 帮助 | `zentao <module> help` |

也支持 `--data='JSON'` 传入完整 JSON 数据（见 03 数据处理）。

### 原始方式

当简写方式的模块名与一级命令冲突时，必须改用原始方式：

| 操作 | 命令 |
|------|------|
| 列表 | `zentao ls <module>` |
| 详情 | `zentao get <module> <id>` |
| 创建 | `zentao create <module> [params]` |
| 更新 | `zentao update <module> <id> [params]` |
| 删除 | `zentao delete <module> <id>` |
| 动作 | `zentao do <module> <action> <id> [params]` |
| 帮助 | `zentao help <module>` |

## 模块与操作速查

| 模块名 | 中文 | 支持的操作 |
|--------|------|-----------|
| program | 项目集 | CRUD |
| product | 产品 | CRUD |
| project | 项目 | CRUD |
| execution | 执行/迭代 | CRUD |
| story | 需求 | CRUD + activate / change / close |
| epic | 业务需求 | CRUD + activate / change / close |
| requirement | 用户需求 | CRUD + activate / change / close |
| bug | Bug | CRUD + activate / close / resolve |
| task | 任务 | CRUD + activate / close / finish / start |
| testcase | 测试用例 | CRUD |
| testtask | 测试单 | CUD（按产品/项目/执行查列表） |
| productplan | 产品计划 | CUD（按产品查列表） |
| build | 版本 | CUD（按项目/执行查列表） |
| release | 发布 | CUD（按产品查列表） |
| feedback | 反馈 | CRUD + activate / close |
| ticket | 工单 | CRUD + activate / close |
| system | 应用 | CU（按产品查列表） |
| user | 用户 | CRUD |
| file | 附件 | 编辑名称 + 删除 |

> CRUD = 列表 + 详情 + 创建 + 更新 + 删除；CUD = 无独立列表接口，需指定所属范围

## 列表范围参数

部分模块的列表需要指定所属范围：

```bash
zentao story --product=1                # 产品 #1 的需求
zentao bug --product=1                  # 产品 #1 的 Bug
zentao task --execution=1               # 执行 #1 的任务
zentao execution --project=5            # 项目 #5 的执行
zentao build --project=5                # 项目 #5 的版本
zentao testtask --product=1             # 产品 #1 的测试单
zentao release --product=1              # 产品 #1 的发布
zentao productplan --product=1          # 产品 #1 的计划
zentao feedback --product=1             # 产品 #1 的反馈
zentao ticket --product=1               # 产品 #1 的工单
```

设置工作区后可省略这些参数（见 04 高级功能「工作区」）。

## 帮助

```bash
zentao help              # 查看所有命令
zentao                   # 直接执行，同 help
zentao bug help          # 查看 Bug 模块的参数和操作
zentao story update help # 查看需求更新操作的参数
zentao bug resolve help  # 查看解决 Bug 操作的参数
```

> 不确定模块参数时，先 `zentao <module> help`；不确定操作参数时，先 `zentao <module> <action> help`。
