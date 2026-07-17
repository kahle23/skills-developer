# 03 — 数据处理

`zentao-cli` 支持对列表数据进行摘取、过滤、搜索、排序、分页等处理。所有选项均支持通过 `.` 访问子字段（如 `assignedTo.realname`）。

## 摘取字段 — `--pick`

只显示指定字段，多个字段用逗号分隔：

```bash
zentao product --pick=id,name,status
zentao bug --product=1 --pick=id,title,severity
```

## 过滤 — `--filter`

```bash
zentao bug --product=1 --filter='status:active'
zentao bug --product=1 --filter='severity<=2,pri<=2'             # 同一 --filter 内逗号分隔 = AND
zentao bug --product=1 --filter='status:active' --filter='status:resolved'  # 多个 --filter = OR
zentao product --filter='name~产品'
zentao product --filter='name~"产品1,产品2"'   # 值含逗号时用引号包裹
```

支持的运算符：

| 运算符 | 含义 |
|--------|------|
| `:` | 等于 |
| `!=` | 不等于 |
| `>` | 大于 |
| `<` | 小于 |
| `>=` | 大于等于 |
| `<=` | 小于等于 |
| `~` | 包含 |
| `!~` | 不包含 |

- 同一个 `--filter` 内逗号分隔多个条件 → **AND** 逻辑。
- 多个 `--filter` 参数 → **OR** 逻辑。
- 参数值包含逗号时，用引号包裹整个值。

## 模糊搜索 — `--search` / `--search-fields`

```bash
zentao bug --product=1 --search=登录 --search-fields=title,steps
zentao product --search=产品1 --search=产品2           # 多个 --search = OR
```

- 多个关键词用逗号分隔（同一 `--search` 内）。
- 多个 `--search` 参数 → **OR** 逻辑。
- `--search-fields` 指定搜索字段，多个用逗号分隔，支持 `.` 子字段。

## 排序 — `--sort`

```bash
zentao bug --product=1 --sort=pri_asc,severity_asc
zentao product --sort=name_asc
zentao bug --sort=id_desc
```

- 后缀 `_asc` 升序、`_desc` 降序。
- 多个排序条件用逗号分隔。

## 分页

| 选项 | 说明 |
|------|------|
| `--page=<n>` | 页码，默认 1 |
| `--recPerPage=<n>` | 每页条数，默认 20 |
| `--all` | 获取全部数据，不分页 |
| `--limit=<n>` | 只取前 n 条 |

```bash
zentao bug --product=1 --page=1 --recPerPage=50
zentao bug --product=1 --all
zentao bug --product=1 --limit=10
```

## 输出格式 — `--format`

| 值 | 说明 |
|----|------|
| `markdown`（默认） | 列表输出 Markdown 表格，单对象输出列表 |
| `json` | 结构化 JSON（CLI 规范化后的 `data` + `pager`），适合程序处理 |
| `raw` | 禅道服务端原始 JSON（字段名可能不同，如 `products`、`recTotal`） |

```bash
zentao product                          # 默认 Markdown
zentao product --format=json            # 规范化 JSON
zentao product --format=raw             # 服务端原始 JSON
```

## 管道与标准输入（Stdin）

通过 `--data @-` 从管道读取 JSON，或直接管道传入：

```bash
cat products.json | zentao product create --data @-
echo '{"name": "新产品"}' | zentao product create
```

## `--data` 传入 JSON

创建/更新时可用 `--data='JSON_STRING'` 传完整数据，适合字段较多或含嵌套结构：

```bash
zentao product create --data='{"name": "产品1"}'
zentao product update 1 --data='{"name": "产品1"}'
```
