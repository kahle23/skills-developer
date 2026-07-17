# 06 — 常用操作示例

## 查看进行中的项目和执行

```bash
zentao project --filter='status:doing' --pick=id,name,status
zentao execution --project=5 --pick=id,name,status
```

## 搜索已关闭执行中的任务（重要）

**问题**：`zentao execution --all` 及 `--params='{"browseType":"all"}'`、`--filter='status=closed'` 等方式**都只能返回 doing/wait 状态的执行**，已关闭的执行完全不出现在列表中。

**解决方案**：通过执行 ID 断层逐个探测已关闭的执行，再在其下搜索任务。

### 步骤 1：探测已关闭的执行

已关闭的执行虽然不在列表中，但通过 `zentao execution <id>` 仍可获取详情。利用已知执行 ID 之间的断层（如已知 ID 10 和 23，中间 11-22 可能存在已关闭的执行），逐个探测：

```bash
zentao execution 11 --format=json | grep '"name"\|"status"'   # 确认是否存在及其状态
zentao execution 12 --format=json | grep '"name"\|"status"'
# ... 依次探测
```

返回 `{"error":{"code":"2008",...}}` 表示该 ID 不存在；返回正常 JSON 则为已关闭的执行。

### 步骤 2：在已关闭执行中搜索任务

```bash
# 关键：必须加 --status=all，否则默认只返回未关闭的任务
zentao task --executionID=19 --status=all --search="扣款" --search-fields=name,desc --all --pick=id,name,status --format=json
```

> **注意**：任务列表的 `--status` 参数默认为 `unclosed`（未关闭），即已完成（done）、已关闭（closed）的任务默认不返回。搜索时务必加 `--status=all` 确保覆盖全部状态的任务。

## 创建需求并关联计划

```bash
zentao story create --product=1 --title="需求标题" --assignedTo=admin --pri=3
zentao story update 11 --title="需求标题" --plan=1
```

## 创建并解决 Bug

```bash
zentao bug create --product=1 --title="Bug标题" --severity=2 --pri=2 --type=codeerror --openedBuild=trunk
zentao bug resolve 42
zentao bug resolve 42 --comment "已解决" --resolution=fixed
```

## 创建、启动并完成任务

```bash
zentao task create --execution=1 --name="任务名" --type=devel --assignedTo=admin --estimate=4
zentao task start 100
zentao task finish 100 --consumed=4
```

## 创建产品

```bash
zentao product create --name=新产品
zentao product create --data='{"name": "新产品", "code": "new"}'
```

## 批量删除

```bash
zentao product delete 1,2 --yes                  # 批量删除，跳过确认
zentao product delete 1,2,3,4,5 --yes --batch-fail-fast   # 出错即停
```

## 通过管道创建

```bash
cat products.json | zentao product create --data @-
echo '{"name": "新产品"}' | zentao product create
```

## 查看帮助

```bash
zentao bug help          # 查看 Bug 模块的参数和操作
zentao story update help # 查看需求更新操作的参数和操作
zentao help              # 查看所有命令
```

## 查看版本与配置

```bash
zentao version                       # 查看 CLI 及禅道服务端版本
zentao config get                    # 查看所有默认配置
zentao config set defaultOutputFormat json   # 设置默认输出格式
```
