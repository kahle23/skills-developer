# 03 — 表设计规范

> **适用数据库**：MySQL 5.7（保留整数显示宽度如 `bigint(20)`），对 8.0 前向兼容。
> 完整示例见 `02-examples.md`。

## 1. 表名规范

- 采用 `小写下划线` 命名法
- 格式：`模块前缀_业务中段_表意后缀`，如 `oa_asset_record`
- 前缀需与现有项目模块保持一致
- 禁止使用复数形式
- 禁止使用 `_info` 等语义模糊的后缀（如 `oa_asset_info` → `oa_asset_record`）

## 2. 必含标准字段

所有业务表必须包含以下 10 个标准字段：

| 字段名 | 类型 | NOT NULL | 默认值 | 说明 |
|--------|------|----------|--------|------|
| `id` | bigint(20) | ✓ | AUTO_INCREMENT | 主键 |
| `platform` | varchar(50) | ✓ | '' | 来源平台标识（见下方说明） |
| `tenant_id` | varchar(50) | ✓ | '' | 租户ID（见下方说明） |
| `owner_id` | bigint(20) | ✓ | — | 数据所属人ID |
| `own_org_id` | bigint(20) | ✓ | — | 数据所属机构ID |
| `create_user` | bigint(20) | ✓ | — | 创建者 |
| `create_time` | datetime | ✓ | — | 创建时间 |
| `modify_user` | bigint(20) | ✓ | — | 修改者 |
| `modify_time` | datetime | ✓ | CURRENT_TIMESTAMP | 修改时间 |
| `delete_status` | tinyint(4) | ✓ | 0 | 删除状态：0 未删除，1 已删除 |

### 字段语义说明

**`platform`**：标识数据来源平台。
- 取值范围：`web` / `app` / `miniapp` / `h5` / `pc` 等客户端端类型，也包含服务器场景如 `server_a` / `server_b`
- 空字符串 `''` 表示"未标识来源"，属于合法默认值，不等于错误
- 是否扩展更多取值由业务决定，无强制字典约束

**`tenant_id`**：租户ID，用 varchar(50) 而非 bigint 的原因：
- 大部分场景租户ID 是数字，但存在把租户的用户名/业务编码当作租户标识的情况
- 字符串类型兼容两种场景，避免类型不一致导致的问题
- 设计新表时统一用 varchar(50)，不要因为"我这个是纯数字"就改成 bigint

**`create_time` / `modify_time`**：
- 统一用 `datetime` 类型（等价于 `datetime(0)`，秒级精度）
- `create_time` 无默认值，由应用层写入；`modify_time` 默认 `CURRENT_TIMESTAMP`

## 3. 字段命名规范

- 小写下划线命名，如 `purchase_date`、`holder_user_id`
- 外键字段格式：`[业务名]_[关联表主键名]`，如 `holder_user_id`（关联用户表）、`asset_no`（关联资产表编号）
- 状态类字段用 `status`，类型 tinyint，配合字典表或 COMMENT 注明取值
- 日期类字段用 `date` 类型（如 `purchase_date`、`operate_date`）
- 时间戳类字段用 `datetime` 类型（如 `create_time`、`operate_time`）

## 4. 索引规范

### 命名规则

- 主键：`PRIMARY KEY (id) USING BTREE`
- 唯一索引：`uk_[字段名]`，如 `uk_asset_no`
- 普通索引：`idx_[字段名]`，如 `idx_status`、`idx_holder_user_id`
- 复合索引按需设计，命名体现关键字段，如 `idx_tenant_status`

### 多租户索引策略

SaaS / 多租户场景下，几乎所有业务查询都带 `tenant_id` 作为过滤条件。索引设计要点：

- **复合索引把 `tenant_id` 放最左前缀**：如 `idx_tenant_status (tenant_id, status)`，而不是单独建 `idx_status`
- **唯一约束加 `tenant_id`**：如 `uk_asset_no_tenant (asset_no, tenant_id)`，保证"同租户内唯一"而非"全局唯一"
- 避免只对 `tenant_id` 单独建索引——区分度太低，单字段索引意义不大

## 5. SQL 输出格式

```sql
-- ----------------------------
-- Table structure for [表名]
-- ----------------------------
DROP TABLE IF EXISTS `[表名]`;
CREATE TABLE `[表名]` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  -- 业务字段（每个都要有 COMMENT）...

  -- 标准字段（10 个，顺序固定）
  `platform` varchar(50) NOT NULL DEFAULT '' COMMENT '来源平台：web/app/miniapp/h5/pc，空=未标识',
  `tenant_id` varchar(50) NOT NULL DEFAULT '' COMMENT '租户ID',
  `owner_id` bigint(20) NOT NULL COMMENT '数据的所属人ID',
  `own_org_id` bigint(20) NOT NULL COMMENT '数据的所属机构ID',
  `create_user` bigint(20) NOT NULL COMMENT '创建者',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `modify_user` bigint(20) NOT NULL COMMENT '修改者',
  `modify_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `delete_status` tinyint(4) NOT NULL DEFAULT 0 COMMENT '删除状态：0 未删除，1 已删除',

  PRIMARY KEY (`id`) USING BTREE,
  -- 其他索引...
) ENGINE = InnoDB AUTO_INCREMENT = 1 COMMENT = '[表中文说明]' ROW_FORMAT = Dynamic;
```

> 完整带业务字段的示例见 `02-examples.md`。

## 6. 禁止项

| 禁止项 | 原因 |
|-------|------|
| ⚠️ 显式指定 `CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci` | 与数据库默认配置重复，换库/改字符集时维护成本高，由数据库默认字符集统一控制 |
| ⚠️ 遗漏任何标准字段 | 破坏多租户、数据归属、审计追踪能力，10 个标准字段必须完整 |
| ⚠️ 表名使用复数形式 | 与项目惯例不一致，且单数更贴合"一行=一个实体"的语义 |
| ⚠️ 使用 `_info` 等语义模糊的表名后缀 | 语义不清，用具体含义的词替代（如 `_record`、`_config`、`_log`） |
| ⚠️ 使用 `FOREIGN KEY` 物理外键约束 | ① 影响写入性能 ② 分库分表时外键失效 ③ 数据迁移/导入困难 ④ 死锁风险。关联关系通过逻辑外键（字段命名 + 索引）表达 |
| ⚠️ 多租户表索引不加 `tenant_id` | 跨租户查询会全表扫描，tenant_id 必须纳入组合索引最左前缀 |

## 7. 输出要求

每次输出表结构时，必须同时提供：
1. 完整的 CREATE TABLE SQL 语句
2. 索引设计说明（每个索引支撑哪些查询场景）
3. 字段填写说明（必填项、默认值、特殊业务含义）
4. 与现有表的关联关系（逻辑外键，非物理外键）
