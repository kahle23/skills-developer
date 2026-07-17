# Baibao & Kunlun 框架使用指南

## 一、引入依赖

```xml
<dependency>
    <groupId>io.github.kahle23</groupId>
    <artifactId>baibao</artifactId>
    <version>最新版</version>
</dependency>
```

`baibao` 依赖中已包含 `kunlun` 框架，两者配合使用。

---

## 二、Baibao 框架

Baibao 提供 Service 基类、查询模式控制、删除状态等基础设施。

### 2.1 BaseServiceImpl — Service 基类

```java
@Service
public class InvoiceServiceImpl extends BaseServiceImpl<InvoiceMapper, Invoice
        , InvoiceAddParam, InvoiceEditParam, InvoiceQuery, InvoiceResult>
        implements InvoiceService {
}
```

**6 个泛型参数**：

| 位置 | 类型 | 说明 |
|------|------|------|
| 1 | `Mapper` | MyBatis-Plus Mapper 接口 |
| 2 | `Entity` | 数据库实体类 |
| 3 | `AddParam` | 新增参数对象 |
| 4 | `EditParam` | 编辑参数对象 |
| 5 | `Query` | 查询条件对象 |
| 6 | `Result` | 查询结果对象 |

### 2.2 QueryMode — 查询模式

控制 Service 方法行为，决定是否执行 `processData` / `fillingData`：

| 模式 | 说明 | processData | fillingData |
|------|------|:-:|:-:|
| `FULL` | 完整模式 | ✅ | ✅ |
| `ONLY_PROCESS` | 仅处理数据 | ✅ | ❌ |
| `ONLY_FILL` | 仅填充数据 | ❌ | ✅ |
| 无 / null | 不处理 | ❌ | ❌ |

```java
import static baibao.common.enums.QueryMode.allowProcess;
import static baibao.common.enums.QueryMode.allowFill;

if (allowProcess(query)) { processData(query, result.getData()); }
if (allowFill(query))    { fillingData(query, result.getData()); }
```

### 2.3 DeleteStatus — 删除状态

| 值 | 含义 |
|----|------|
| `0` | 未删除 |
| `1` | 已删除 |

```java
// 过滤未删除
List<InvoiceUsage> activeList = usageList.stream()
    .filter(u -> Objects.equals(u.getDeleteStatus(), 0))
    .collect(Collectors.toList());

// 标记已删除
entity.setDeleteStatus(DeleteStatus.DELETED.getCode());
```

### 2.4 Actions

`INTERNAL_BUS` 是内部消息总线标识，用于发送服务间消息。

```java
import static baibao.common.constant.Actions.INTERNAL_BUS;
```

---

## 三、Kunlun 框架

### 3.1 核心模块一览

| 模块 | 包路径 | 功能 |
|------|--------|------|
| 分页 | `kunlun.common` | `Page` 分页对象 |
| 数据处理 | `kunlun.data` | Bean 转换、字典、数据填充 |
| 事件消息 | `kunlun.action` / `kunlun.message` | 事件驱动、消息总线 |
| 安全认证 | `kunlun.security` | 用户上下文 |
| 文件处理 | `kunlun.io.fileprocessor` | Excel 导入导出 |
| ID 生成 | `kunlun.generator.id` | 分布式 ID |
| 工具类 | `kunlun.util` | 分页辅助、集合操作 |

### 3.2 Bean 转换

```java
import kunlun.data.bean.BeanUtil;

// Param ↔ Entity
Invoice entity = BeanUtil.beanToBean(param, Invoice.class);
InvoiceEditParam param = BeanUtil.beanToBean(entity, InvoiceEditParam.class);

// List 批量转换
List<InvoiceResult> results = BeanUtil.beanToBeanInList(list, InvoiceResult.class);
```

### 3.3 数据填充

```java
import kunlun.data.fill.classic.FillCfg;
import kunlun.data.fill.classic.DataCfg;
import kunlun.data.fill.classic.support.EnumSupplier;
import kunlun.data.fill.classic.support.MpServIdsSupplier;

// 枚举值填充 — addFieldConfig(源字段, 目标字段, 枚举属性)
FillCfg.of(result)
    .addDataConfig(DataCfg.of(EnumSupplier.of(InvoiceDirection.class))
        .addFieldConfig("invoiceDirection", "directionDesc", "description"))
    .fill();

// Service ID 填充 — addFieldConfig(ID字段, 结果字段, 提取属性)
FillCfg.of(data)
    .addDataConfig(DataCfg.of(MpServIdsSupplier.of(UserService.class, User::getId))
        .addFieldConfig("contractBizUserId", "contractBizUserTxt", "name"))
    .addDataConfig(DataCfg.of(MpServIdsSupplier.of(OrganizationService.class, Organization::getId))
        .addFieldConfig("contractOwningOrgId", "contractOwningOrgTxt", "abbr"))
    .fill();
```

### 3.4 字典管理

```java
// 获取字典组
List<DataDict> dicts = DictUtil.listByGroup("INVOICE_TYPE");

// 批量获取并构建 name→value 映射
Map<String, Map<String, String>> dictMaps = Stream.of(INVOICE_TYPE, INVOICE_RISK_LEVEL)
    .collect(Collectors.toMap(Function.identity(),
        group -> DictUtil.listByGroup(group).stream()
            .collect(Collectors.toMap(DataDict::getName, DataDict::getValue))));

// 值转换
invoice.setInvoiceType(Integer.parseInt(invoiceTypeMap.get(txt)));
```

### 3.5 分页工具

```java
import kunlun.util.PageUtil;
import kunlun.common.Page;

// 开启分页
PageUtil.startPage(query.getPageNum(), query.getPageSize());

// 处理结果
Page<InvoiceResult> result = PageUtil.handleResult(list, InvoiceResult.class);

// 添加序号
PageUtil.fillSerialNumber(result.getData(), result.getPageNum(), result.getPageSize());

// 空分页
Page<InvoiceResult> empty = Page.of();
```

### 3.6 分布式 ID 生成

```java
import kunlun.generator.id.IdUtil;

entity.setId(IdUtil.nextLong("invoice-id"));
```

### 3.7 安全与用户上下文

```java
import kunlun.security.SecurityUtil;
import kunlun.security.UserDetail;

UserDetail userDetail = SecurityUtil.getUserDetail();
String userName = userDetail != null ? userDetail.getDisplayName() : null;
```

### 3.8 事件与消息

```java
import kunlun.action.ActionUtil;
import kunlun.data.Event;
import kunlun.message.model.Message;

// 发送内部消息
ActionUtil.execute(INTERNAL_BUS, new Message(TOPIC, payload));

// 记录变更日志 — preProcess 中可对字段差异做枚举值翻译等预处理
ActionUtil.execute(Event.ofChangeLog()
    .setBusinessType(业务类型)
    .setBusinessId(bizId)
    .appendMessage(new FieldDifferenceBuilder(oldData, newData, targetClz) {
        @Override
        protected void preProcess(List<FieldCompareResult> results) {
            // 对需要枚举翻译的字段使用 FillCfg 填充
        }
    })
);
```

### 3.9 文件导入导出

```java
import kunlun.io.fileprocessor.ProcConfig;
import kunlun.io.fileprocessor.support.EasyExcelOneTimeImportProcessor;
import kunlun.io.fileprocessor.support.EasyExcelByteArrayBasedExportProcessor;

// 导入
ProcResult result = new ProcConfig<MultipartFile, InvoiceImportResult>()
    .setFileProcessor(new EasyExcelOneTimeImportProcessor(
            InvoiceImportResult.class, "发票导入.xlsx"))
    .setParam(file)
    .setDataConsumer((context, page) -> {
        // 处理每页数据，更新 statistic.setSuccessCount / setFailureCount
    })
    .execute();

// 导出
ProcResult result = new ProcConfig<InvoiceQuery, InvoiceResult>()
    .setFileProcessor(new EasyExcelByteArrayBasedExportProcessor(
            InvoiceResult.class, "文件导出.xlsx"))
    .setParam(query)
    .setDataSupplier((context, pageId) -> {
        query.setPageNum((Integer) pageId);
        query.setPageSize(200);
        return queryPage(query);
    })
    .execute();
```

### 3.10 验证工具

```java
import static kunlun.data.validation.support.javax.ValidationUtil.validateToThrow;

validateToThrow(query);  // JSR-303 校验，失败抛异常
```

### 3.11 断言工具

```java
import static kunlun.exception.util.VerifyUtil.isFalse;
import static kunlun.exception.util.VerifyUtil.isTrue;

isFalse(condition, "错误提示");   // false 时抛异常
isTrue(!condition, "错误提示");   // true 时抛异常（取非后判断）
```

---

## 四、Service 层完整示例

### 4.1 参数转换

```java
@Override
protected Invoice fromAddParam(InvoiceAddParam param) {
    Invoice entity = BeanUtil.beanToBean(param, Invoice.class);
    entity.setId(IdUtil.nextLong("invoice-id"));
    entity.setPriceCurrency(SettleCurrency.CNY.getCode());
    entity.setBizStatus(InvoiceBizStatus.UNVERIFIED.getCode());
    return entity;
}
```

### 4.2 分页查询

```java
@Override
public Page<InvoiceResult> queryPage(InvoiceQuery query) {
    validateToThrow(query);

    if (query.isPaged()) {
        PageUtil.startPage(query.getPageNum(), query.getPageSize());
    }

    List<Invoice> list = list(buildQueryWrapper(query));
    if (CollUtil.isEmpty(list)) { return Page.of(); }

    Page<InvoiceResult> result = PageUtil.handleResult(list, InvoiceResult.class);

    if (query.isPaged()) {
        PageUtil.fillSerialNumber(result.getData(), result.getPageNum(), result.getPageSize());
    }

    if (allowProcess(query)) { processData(query, result.getData()); }
    if (allowFill(query))    { fillingData(query, result.getData()); }

    return result;
}
```

### 4.3 查询条件构建

```java
@Override
protected MPJLambdaWrapper<Invoice> buildQueryWrapper(InvoiceQuery query) {
    return JoinWrappers.lambda(Invoice.class)
        .in(isNotEmpty(query.getIdList()), Invoice::getId, query.getIdList())
        .eq(nonNull(query.getInvoiceBizType()), Invoice::getInvoiceBizType, query.getInvoiceBizType())
        .like(isNotBlank(query.getSellerNameLike()), Invoice::getSellerName, query.getSellerNameLike())
        .ge(nonNull(query.getBeginIssueTime()), Invoice::getIssueTime, query.getBeginIssueTime())
        .le(nonNull(query.getEndIssueTime()), Invoice::getIssueTime, query.getEndIssueTime())
        .orderByDesc(Invoice::getIssueTime);
}
```

### 4.4 processData — 数据处理骨架

```java
@Override
protected void processData(InvoiceQuery query, List<InvoiceResult> data) {
    List<Long> idList = data.stream()
        .map(InvoiceResult::getId)
        .distinct()
        .collect(Collectors.toList());
    // 批量查询关联数据，填充到结果对象
}
```

> `fillingData` 详见 [3.3 数据填充](#33-数据填充)，`changeLog` 详见 [3.8 事件与消息](#38-事件与消息)，Excel 导入/导出详见 [3.9 文件导入导出](#39-文件导入导出)。

---

## 五、典型调用链

```
Controller
  └── Service.queryPage(query)
        ├── validateToThrow(query)              // 参数校验
        ├── PageUtil.startPage(...)             // 分页
        ├── list(buildQueryWrapper(query))      // 查询
        ├── PageUtil.handleResult(...)          // 结果转换
        ├── allowProcess → processData()        // QueryMode 控制 — 数据处理
        └── allowFill → fillingData()           // QueryMode 控制 — 字段填充
```

---

## 六、Service 生命周期钩子

| 方法 | 调用时机 | 用途 |
|------|---------|------|
| `fromAddParam()` | 新增前 | Param → Entity，设置默认值、ID |
| `fromEditParam()` | 编辑前 | Param → Entity |
| `buildQueryWrapper()` | 查询前 | 构建查询条件 |
| `processData()` | 查询后 | 业务数据处理，受 QueryMode 控制 |
| `fillingData()` | 查询后 | 字段填充，受 QueryMode 控制 |
| `changeLog()` | 变更后 | 记录数据变更日志 |
