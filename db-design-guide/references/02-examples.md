# 02 — 完整示例表

本文档提供一份**完整的、可直接参考**的建表示例，覆盖标准字段 + 业务字段 + 全套索引 + 关联关系。设计新表时对照此示例检查格式。

## 正确示例：oa_asset_record（资产领用记录）

> 业务场景：记录员工领用资产的流水。关联资产表、用户表、机构表。多租户场景。

```sql
-- ----------------------------
-- Table structure for oa_asset_record
-- ----------------------------
DROP TABLE IF EXISTS `oa_asset_record`;
CREATE TABLE `oa_asset_record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `asset_no` varchar(64) NOT NULL DEFAULT '' COMMENT '资产编号（关联 oa_asset.asset_no）',
  `asset_name` varchar(200) NOT NULL DEFAULT '' COMMENT '资产名称（冗余便于查询）',
  `holder_user_id` bigint(20) NOT NULL COMMENT '领用人ID（关联 user 表主键）',
  `holder_org_id` bigint(20) NOT NULL COMMENT '领用人所属机构ID（关联 org 表主键）',
  `receive_date` date NOT NULL COMMENT '领用日期',
  `return_date` date DEFAULT NULL COMMENT '归还日期，未归还为 NULL',
  `status` tinyint(4) NOT NULL DEFAULT 0 COMMENT '状态：0 领用中，1 已归还，2 已报废',
  `remark` varchar(500) NOT NULL DEFAULT '' COMMENT '备注',

  `platform` varchar(50) NOT NULL DEFAULT '' COMMENT '来源平台：web/app/miniapp/h5/pc，空=未标识',
  `tenant_id` varchar(50) NOT NULL DEFAULT '' COMMENT '租户ID（数字编码或业务编码，按租户体系而定）',
  `owner_id` bigint(20) NOT NULL COMMENT '数据所属人ID',
  `own_org_id` bigint(20) NOT NULL COMMENT '数据所属机构ID',
  `create_user` bigint(20) NOT NULL COMMENT '创建者',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `modify_user` bigint(20) NOT NULL COMMENT '修改者',
  `modify_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `delete_status` tinyint(4) NOT NULL DEFAULT 0 COMMENT '删除状态：0 未删除，1 已删除',

  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `uk_asset_no_tenant` (`asset_no`, `tenant_id`) COMMENT '同租户下资产编号唯一',
  KEY `idx_tenant_status` (`tenant_id`, `status`) COMMENT '多租户按状态筛选',
  KEY `idx_holder_user` (`holder_user_id`) COMMENT '按领用人查询',
  KEY `idx_receive_date` (`receive_date`) COMMENT '按领用日期范围查询'
) ENGINE = InnoDB AUTO_INCREMENT = 1 COMMENT = '资产领用记录' ROW_FORMAT = Dynamic;
```

### 关联关系说明

| 本表字段 | 关联到 | 关系 | 说明 |
|---------|-------|------|------|
| `asset_no` | `oa_asset.asset_no` | 多对一 | 一条领用记录对应一个资产 |
| `holder_user_id` | `user.id` | 多对一 | 一条记录对应一个领用人 |
| `holder_org_id` | `org.id` | 多对一 | 冗余机构ID，避免join |

### 索引设计理由

| 索引 | 支撑的查询场景 |
|------|--------------|
| `uk_asset_no_tenant` | 防止同租户下资产重复领用；按资产编号精确查询 |
| `idx_tenant_status` | "查某租户下所有领用中/已归还的记录"——最高频查询，tenant_id 放最左 |
| `idx_holder_user` | "查某人领用了哪些资产"——按人维度查询 |
| `idx_receive_date` | 按日期范围统计/导出报表 |

---

## 常见错误对照

### ❌ 错误 1：表名用复数 + `_info` 后缀

```sql
-- 错误
CREATE TABLE `oa_asset_records` (...);          -- 复数形式
CREATE TABLE `oa_asset_info` (...);             -- _info 语义模糊
CREATE TABLE `oa_asset_record_infos` (...);     -- 两个都犯

-- ✅ 正确
CREATE TABLE `oa_asset_record` (...);           -- 单数 + 具体含义
```

### ❌ 错误 2：遗漏标准字段

```sql
-- 错误：缺 tenant_id、owner_id 等
CREATE TABLE `oa_asset_record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `asset_no` varchar(64) NOT NULL,
  ...
  PRIMARY KEY (`id`)
) ENGINE = InnoDB;

-- ✅ 正确：完整包含 10 个标准字段（见 03 第 2 节）
```

### ❌ 错误 3：使用物理外键

```sql
-- 错误
CONSTRAINT `fk_asset_no` FOREIGN KEY (`asset_no`) REFERENCES `oa_asset` (`asset_no`)

-- ✅ 正确：逻辑外键（字段命名 + 索引），不加 CONSTRAINT FOREIGN KEY
```

### ❌ 错误 4：显式声明字符集

```sql
-- 错误：与数据库默认配置重复，换库时维护成本高
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_general_ci COMMENT = 'xxx';

-- ✅ 正确：不指定，由数据库默认字符集控制
) ENGINE = InnoDB COMMENT = 'xxx' ROW_FORMAT = Dynamic;
```

### ❌ 错误 5：多租户表索引不加 tenant_id

```sql
-- 错误：跨租户查询会全表扫描
KEY `idx_status` (`status`)

-- ✅ 正确：tenant_id 放最左前缀
KEY `idx_tenant_status` (`tenant_id`, `status`)
```

### ❌ 错误 6：索引命名不规范

```sql
-- 错误
KEY `asset_no` (`asset_no`)                      -- 没有前缀
KEY `index_status` (`status`)                    -- 前缀太长
UNIQUE KEY `unique_asset` (`asset_no`)           -- 前缀太长

-- ✅ 正确
KEY `idx_asset_no` (`asset_no`)                  -- 普通索引 idx_
UNIQUE KEY `uk_asset_no_tenant` (`asset_no`, `tenant_id`)  -- 唯一索引 uk_
```
