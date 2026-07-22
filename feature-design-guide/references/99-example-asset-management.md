# 99 — 完整示例文档：IT 资产管理模块

> 本文档是一份完整的设计文档示例，供编写新功能设计文档时对照参考。
> 涵盖模板五章节的完整写法、表格格式、SQL 风格、接口梳理方式。
> 实际编写时，以此为骨架，替换为真实业务内容。

---

## 一、需求概述

### 功能背景与目标

公司 IT 设备数量增长，目前通过 Excel 管理设备台账，存在以下问题：

- 设备状态（领用/归还/维修）更新不及时，数据滞后
- 多人维护同一份 Excel，版本混乱，责任不清
- 无法按部门、人员、时间维度统计设备使用情况

**目标**：建立统一的 IT 资产管理系统，实现设备全生命周期管理（入库 → 领用 → 归还 → 报废），提供实时的设备台账和统计报表。

### 适用范围

- 资产类型：仅限 IT 设备（笔记本、台式机、显示器、手机、外设）
- 使用范围：全公司正式员工

### 核心名词定义

| 术语 | 说明 |
|------|------|
| 资产编码 | IT 设备的唯一标识，规则见第五章 |
| 设备状态 | 在库 / 已领用 / 维修中 / 已报废 |
| 责任人 | 当前持有该设备的人员，设备流转时同步变更 |

---

## 二、功能模块梳理

本模块共 4 个子模块、11 个功能点，其中核心 7 个、扩展 4 个。

### 设备台账（核心）

| 功能点 | 说明 |
|--------|------|
| 设备录入 | 录入新设备信息，自动生成资产编码 |
| 设备列表 | 分页查询，支持按状态/类型/责任人筛选 |
| 设备详情 | 查看设备完整信息和流转记录 |
| 设备编辑 | 修改设备基础信息（仅管理员） |

### 设备流转（核心）

| 功能点 | 说明 |
|--------|------|
| 领用申请 | 员工申请领用在库设备 |
| 归还登记 | 设备归还入库，更新状态 |
| 流转记录 | 记录每次领用/归还历史 |

### 设备维护（扩展）

| 功能点 | 说明 |
|--------|------|
| 维修登记 | 登记设备维修信息 |
| 报废处理 | 标记设备报废，不可再领用 |

### 统计报表（扩展）

| 功能点 | 说明 |
|--------|------|
| 设备统计 | 按部门/类型/状态统计设备数量 |
| 使用率报表 | 统计设备领用率、闲置率 |

---

## 三、表结构设计

### 3.1 新建表

#### it_device（IT 设备表）

```sql
CREATE TABLE `it_device` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `device_code` varchar(32) NOT NULL COMMENT '资产编码',
  `device_name` varchar(64) NOT NULL COMMENT '设备名称',
  `device_type` varchar(16) NOT NULL COMMENT '设备类型（notebook/desktop/monitor/phone/peripheral）',
  `brand` varchar(32) DEFAULT NULL COMMENT '品牌',
  `model` varchar(64) DEFAULT NULL COMMENT '型号',
  `serial_number` varchar(64) DEFAULT NULL COMMENT '序列号/SN码',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态（0=在库 1=已领用 2=维修中 3=已报废）',
  `current_user_id` bigint(20) DEFAULT NULL COMMENT '当前责任人ID',
  `current_user_name` varchar(32) DEFAULT NULL COMMENT '当前责任人姓名',
  `current_dept_id` bigint(20) DEFAULT NULL COMMENT '当前所属部门ID',
  `purchase_date` date DEFAULT NULL COMMENT '采购日期',
  `purchase_price` decimal(10,2) DEFAULT NULL COMMENT '采购价格',
  `warranty_expire_date` date DEFAULT NULL COMMENT '保修到期日',
  `remark` varchar(500) DEFAULT NULL COMMENT '备注',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_by` varchar(32) NOT NULL COMMENT '创建人',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `update_by` varchar(32) NOT NULL COMMENT '更新人',
  `deleted` tinyint(1) NOT NULL DEFAULT '0' COMMENT '逻辑删除标记（0=未删除 1=已删除）',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_device_code` (`device_code`),
  KEY `idx_status` (`status`),
  KEY `idx_current_user` (`current_user_id`),
  KEY `idx_type_status` (`device_type`, `status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='IT设备表';
```

#### it_device_flow（设备流转记录表）

```sql
CREATE TABLE `it_device_flow` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `device_id` bigint(20) NOT NULL COMMENT '设备ID',
  `device_code` varchar(32) NOT NULL COMMENT '资产编码（冗余便于查询）',
  `flow_type` tinyint(4) NOT NULL COMMENT '流转类型（1=领用 2=归还 3=维修 4=报废）',
  `from_user_id` bigint(20) DEFAULT NULL COMMENT '原责任人ID',
  `from_user_name` varchar(32) DEFAULT NULL COMMENT '原责任人姓名',
  `to_user_id` bigint(20) DEFAULT NULL COMMENT '新责任人ID',
  `to_user_name` varchar(32) DEFAULT NULL COMMENT '新责任人姓名',
  `operate_time` datetime NOT NULL COMMENT '操作时间',
  `operate_reason` varchar(200) DEFAULT NULL COMMENT '操作原因/说明',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `create_by` varchar(32) NOT NULL COMMENT '操作人',
  PRIMARY KEY (`id`),
  KEY `idx_device_id` (`device_id`),
  KEY `idx_flow_type_time` (`flow_type`, `operate_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='设备流转记录表';
```

### 3.2 复用现有表

| 表名 | 复用方式 | 说明 |
|------|----------|------|
| `sys_user` | 关联查询 | 获取责任人姓名、所属部门 |
| `sys_dept` | 关联查询 | 获取部门信息，用于按部门统计 |

### 3.3 数据字典预设

```sql
-- 设备类型字典
INSERT INTO `sys_dict_group` (`group_code`, `group_name`, `remark`) VALUES
('it_device_type', 'IT设备类型', 'IT资产设备分类');

INSERT INTO `sys_dict_item` (`group_code`, `item_code`, `item_label`, `sort`) VALUES
('it_device_type', 'notebook', '笔记本电脑', 1),
('it_device_type', 'desktop', '台式机', 2),
('it_device_type', 'monitor', '显示器', 3),
('it_device_type', 'phone', '手机', 4),
('it_device_type', 'peripheral', '外设', 5);

-- 设备状态字典
INSERT INTO `sys_dict_group` (`group_code`, `group_name`, `remark`) VALUES
('it_device_status', 'IT设备状态', '设备当前状态');

INSERT INTO `sys_dict_item` (`group_code`, `item_code`, `item_label`, `sort`) VALUES
('it_device_status', 'in_stock', '在库', 1),
('it_device_status', 'in_use', '已领用', 2),
('it_device_status', 'repairing', '维修中', 3),
('it_device_status', 'scrapped', '已报废', 4);
```

---

## 四、接口梳理

> 接口的请求方式、返回结构、异常码等遵循团队接口规范，本章节只梳理需要哪些接口及其核心字段。

### 设备台账

| 序号 | 接口地址 | 接口说明 | 核心入参 |
|------|----------|----------|----------|
| 1 | /it/device/add | 新建设备 | device_name, device_type, brand, model |
| 2 | /it/device/page | 分页查询设备 | status, device_type, current_user_id, keyword |
| 3 | /it/device/detail | 设备详情 | id |
| 4 | /it/device/update | 修改设备 | id, 可修改字段 |
| 5 | /it/device/delete | 删除设备（逻辑删除） | id |

### 设备流转

| 序号 | 接口地址 | 接口说明 | 核心入参 |
|------|----------|----------|----------|
| 6 | /it/device/claim | 领用设备 | device_id, to_user_id, reason |
| 7 | /it/device/return | 归还设备 | device_id, reason |
| 8 | /it/device/flow/page | 流转记录分页 | device_id, flow_type, date_range |

### 统计报表

| 序号 | 接口地址 | 接口说明 | 核心入参 |
|------|----------|----------|----------|
| 9 | /it/stat/device_summary | 设备数量统计 | group_by（type/status/dept） |
| 10 | /it/stat/usage_rate | 设备使用率 | date_range |

---

## 五、关键业务逻辑说明

### 资产编码自动生成规则

**格式**：`IT-{类型缩写}-{YYYYMMDD}-{4位流水号}`

- 类型缩写：NB（笔记本）/ DT（台式机）/ MN（显示器）/ PH（手机）/ PR（外设）
- 示例：`IT-NB-20260101-0001`

**生成策略**：

- 流水号按"日期 + 类型"维度递增，每日重置
- 通过 Redis incr 获取流水号，key 格式 `it:device:seq:{type}:{yyyyMMdd}`，设置当日过期
- 编码生成后写入数据库，不可修改

### 设备状态流转规则

```
┌─────────┐  领用   ┌─────────┐  归还   ┌─────────┐
│  在库   │ ──────→ │ 已领用  │ ──────→ │  在库   │
│ (0)     │         │ (1)     │         │ (0)     │
└─────────┘         └─────────┘         └─────────┘
     │                   │                   │
     │ 维修登记           │ 报修               │ 维修登记
     ↓                   ↓                   ↓
┌─────────┐         ┌─────────┐
│ 维修中  │ ────→   │ 已报废  │
│ (2)     │ 维修完成 │ (3)     │
└─────────┘ 返回在库 └─────────┘
```

**规则约束**：

- 在库设备才能被领用
- 已领用设备才能归还
- 维修完成后自动回到在库状态
- 已报废状态为终态，不可再流转

### 数据权限控制

- **普通员工**：仅能查看自己当前领用的设备，可发起领用申请
- **部门管理员**：可查看本部门所有设备的流转记录
- **IT 管理员**：可操作所有设备（录入、编辑、报废），查看全量统计报表

### 字段填写说明

| 场景 | 必填字段 | 可选字段 |
|------|----------|----------|
| 新建设备 | device_name, device_type | brand, model, serial_number, purchase_date, purchase_price |
| 领用设备 | device_id, to_user_id | reason |
| 归还设备 | device_id | reason |
| 报废设备 | device_id | remark（报废原因说明） |
