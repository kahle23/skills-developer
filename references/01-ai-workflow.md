# AI 编码工作流

在编写任何业务代码之前，AI 必须遵循以下工作流，确保复用已有代码、遵循已有文档、避免重复造轮子。

---

# 一、核心原则

**先找，再读，最后写。**

```
用户提出编码需求
│
├─ Step 1: 搜 — 搜索项目中是否已有类似功能
├─ Step 2: 读 — 阅读相关文档和已有代码
├─ Step 3: 判 — 判断能否复用/扩展已有代码
├─ Step 4: 写 — 只有确实不存在时才编写新代码
└─ Step 5: 录 — 将新增的通用方法/模式记录到文档中
```

---

# 二、Step 1：搜索已有代码

在写任何代码之前，先在项目中搜索：

## 2.1 搜索关键词

| 需求类型 | 搜索方式 |
|---------|---------|
| 新增功能 | 搜索功能相关的类名、方法名、关键词 |
| 工具方法 | 搜索工具类（如 `*Util.java`、`*Helper.java`、`*Common*.java`） |
| 数据处理 | 搜索现有的 Mapper、Service 中是否有类似查询 |
| 枚举/常量 | 搜索 `*Enum.java`、`*Const*.java`、`*Constant*.java` |
| 接口定义 | 搜索 API 文档、Swagger 注解、Controller 方法 |

## 2.2 搜索范围

```
优先级从高到低：
1. 当前模块（如 wms、sale 等）
2. 相邻模块（如 wms 搜不到就搜 biz、base）
3. 公共模块（common、util、utils）
4. 第三方工具类（hutool、guava、apache-commons）
```

## 2.3 搜索示例

```
需求：需要一个根据日期范围查询订单的方法

搜索步骤：
1. grep "date" "*.java" in OrderService / OrderMapper — 是否已有日期查询
2. grep "DateUtil" "*.java" — 是否有日期处理工具方法
3. grep "orderDate" "*.xml" — 是否有相关 SQL
4. 检查 Query 类是否已有日期字段
```

---

# 三、Step 2：阅读相关文档和代码

## 3.1 项目文档优先级

```
1. README / 项目说明文档 — 了解项目整体架构
2. API 文档（Swagger/OpenAPI）— 了解已有接口
3. 数据库设计文档 — 了解表结构和字段含义
4. 模块说明文档 — 了解各模块职责边界
5. 本技能文档（references/）— 了解编码规范
```

## 3.2 阅读已有代码时关注

| 关注点 | 说明 |
|-------|------|
| 命名风格 | 已有类/方法的命名模式，新代码要保持一致 |
| 返回类型 | 已有方法返回 `Result`/`R`/`PageResult` 等，新代码要统一 |
| 异常处理 | 已有代码如何抛异常，新代码要一致 |
| 工具类使用 | 已有代码用 hutool 还是 apache-commons，新代码要统一 |
| 注解风格 | 已有代码用 `@Resource` 还是 `@Autowired`，新代码要一致 |
| 分页方式 | 已有代码用哪种分页方式，新代码要统一 |
| 日志格式 | 已有代码的日志风格，新代码要统一 |

---

# 四、Step 3：判断复用策略

```
搜索结果分析
│
├─ 找到完全匹配的方法 → 直接调用，不重复编写
│
├─ 找到相似但不完全匹配的方法 →
│   ├─ 差异小 → 扩展已有方法（增加参数/重载）
│   └─ 差异大 → 新建方法，但引用已有方法处理公共逻辑
│
├─ 找到可组合的多个方法 → 组合调用，不重写
│
└─ 完全没有 → 编写新代码，遵循已有风格
```

## 4.1 复用判断示例

```
需求：根据仓库ID和日期范围查询库存变动

搜索结果：
- StockMapper.selectByWarehouseId(Long warehouseId) — 只按仓库查
- StockMapper.selectByDateRange(Date start, Date end) — 只按日期查

判断：
- 两个方法都不完全满足需求
- 但可以参考它们的 SQL 结构和返回类型
→ 新建 selectByWarehouseIdAndDateRange 方法，复用已有的字段映射和结果类
```

## 4.2 不要重复造轮子的常见场景

```java
// BAD — 自己写日期格式化
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
String dateStr = sdf.format(date);

// GOOD — 项目已有 Hutool，直接用
String dateStr = DateUtil.format(date, "yyyy-MM-dd");

// BAD — 自己写集合判空工具
if (list != null && list.size() > 0) { }

// GOOD — 项目已有工具类
if (CollUtil.isNotEmpty(list)) { }

// BAD — 自己写分页逻辑
int start = (pageNum - 1) * pageSize;
List<T> subList = list.subList(start, Math.min(start + pageSize, list.size()));

// GOOD — 使用已有的分页工具或框架分页
PageHelper.startPage(pageNum, pageSize);
```

---

# 五、Step 4：编写新代码

只有确认没有可复用的代码后，才编写新代码。编写时：

1. **遵循已有风格** — 参考 Step 2 中观察到的命名、注解、异常处理风格
2. **遵循本技能规范** — 参考 `03-code-standards.md` 等文档
3. **使用项目已有工具类** — 优先用项目中已引入的工具库
4. **保持接口一致性** — 返回类型、参数风格与已有代码统一

---

# 六、Step 5：记录与文档化

如果新增了通用方法或新模式：

1. 在代码注释中说明用途
2. 如果是通用工具方法，考虑是否应该放到公共模块
3. 如果是新的编码模式，用户可以通过 `02-code-examples-template.md` 将其沉淀到本技能中

---

# 七、快速检查清单

在开始写代码前，逐项确认：

```
□ 搜索了项目中是否已有类似功能
□ 搜索了公共模块（acommon/common）中的工具类
□ 确认了项目使用的工具库（hutool/guava/apache-commons）
□ 阅读了相关模块的已有代码风格
□ 确认了返回类型和异常处理方式
□ 确认了分页方式
□ 确认了日志风格
□ 确认无法复用，才开始编写新代码
```
