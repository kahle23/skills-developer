# Hutool 工具库

Hutool 是 Java 工具集，覆盖字符串、集合、日期、JSON、Bean 等场景。

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.25</version>
</dependency>
```

---

# 一、StrUtil — 字符串

```java
import cn.hutool.core.util.StrUtil;

// 判空（最常用）
StrUtil.isBlank(null)        // true  — null、""、"  " 都为 true
StrUtil.isNotBlank("abc")    // true
StrUtil.isEmpty(null)        // true  — null、"" 为 true，"  " 为 false
StrUtil.isNotEmpty("abc")    // true
StrUtil.hasBlank("a", "")    // true  — 任一为空
StrUtil.hasEmpty("a", null)  // true
```

> 另有 `trim` / `split` / `sub` / `format` / `toCamelCase` / `toUnderlineCase` 等方法，需要时查阅 API。

---

# 二、CollUtil — 集合

```java
import cn.hutool.core.collection.CollUtil;

// 判空
CollUtil.isEmpty(list) / CollUtil.isNotEmpty(list)

// 分组
Map<Integer, List<User>> groupByField = CollUtil.groupByField(userList, "age");
Map<String, List<User>> groupBy = CollUtil.groupBy(userList, User::getDeptName);

// 集合运算
CollUtil.intersection(list1, list2)   // 交集
CollUtil.union(list1, list2)          // 并集
CollUtil.disjunction(list1, list2)    // 对称差集
CollUtil.subtract(list1, list2)       // 差集

// 分页
List<List<String>> partition = CollUtil.split(list, 3);  // 每3个一组
```

---

# 三、ObjectUtil — 对象

```java
import cn.hutool.core.util.ObjectUtil;

// 判空
ObjectUtil.isNull(obj) / ObjectUtil.isNotNull(obj)

// 默认值
String val = ObjectUtil.defaultIfNull(input, "默认值");
String val = ObjectUtil.defaultIfEmpty(input, "默认值");  // null/""/"  " → 默认值

// 比较
boolean eq = ObjectUtil.equal(obj1, obj2);
int cmp = ObjectUtil.compare(obj1, obj2);
```

---

# 四、DateUtil — 日期

```java
import cn.hutool.core.date.DateUtil;

// 当前时间
Date now = DateUtil.date();

// 字符串 → 日期
Date date = DateUtil.parse("2024-01-01");
Date date = DateUtil.parse("2024-01-01 12:30:00");
Date date = DateUtil.parse("20240101", "yyyyMMdd");

// 日期 → 字符串
DateUtil.format(date, "yyyy-MM-dd")            // "2024-01-01"
DateUtil.formatDateTime(date)                  // "2024-01-01 12:30:00"

// 日期计算
DateUtil.offsetDay(date, 1)     // +1天
DateUtil.offsetMonth(date, 1)   // +1月
DateUtil.offsetHour(date, 2)    // +2小时

// 一天开始/结束
Date beginOfDay = DateUtil.beginOfDay(date);   // 2024-01-01 00:00:00
Date endOfDay = DateUtil.endOfDay(date);       // 2024-01-01 23:59:59
```

> 时间差可用 `DateUtil.between(date1, date2, DateUnit.DAY)`；项目若用 Java 8+ 可考虑 `java.time` 替代。

---

# 五、JSONUtil — JSON

```java
import cn.hutool.json.JSONUtil;
import cn.hutool.json.JSONObject;

// 对象 ↔ JSON
String json = JSONUtil.toJsonStr(user);
User user = JSONUtil.toBean(json, User.class);
List<User> users = JSONUtil.toList(json, User.class);

// JSONObject 操作
JSONObject obj = JSONUtil.parseObj(json);
String name = obj.getStr("name");
Long id = obj.getLong("id");
String city = obj.getByPath("address.city", String.class);
obj.set("name", "新名称");
```

---

# 六、BeanUtil — Bean 拷贝

```java
import cn.hutool.core.bean.BeanUtil;

// 拷贝到已有对象
BeanUtil.copyProperties(user, dto);
BeanUtil.copyProperties(user, dto, "id", "createTime");  // 排除字段

// 拷贝到新对象
UserDTO dto = BeanUtil.copyProperties(user, UserDTO.class);
UserDTO dto = BeanUtil.toBean(user, UserDTO.class);

// 列表拷贝
List<UserDTO> dtos = BeanUtil.copyToList(users, UserDTO.class);
```

---

# 七、NumberUtil — 数字

```java
import cn.hutool.core.util.NumberUtil;

// 算术运算（BigDecimal 精度）
NumberUtil.add(1.1, 2.2)       // 3.3
NumberUtil.mul(2.0, 3.0)       // 6.0
NumberUtil.div(10, 3, 2)       // 3.33
NumberUtil.round(3.14159, 2)   // 3.14

// 判断
NumberUtil.isNumber("123.45")
NumberUtil.isBetween(5, 1, 10)    // true

// 格式化
NumberUtil.decimalFormat(",##0.00", 1234567.89)  // "1,234,567.89"
```

---

# 八、IdUtil — ID 生成

```java
import cn.hutool.core.util.IdUtil;

String uuid = IdUtil.fastSimpleUUID();       // UUID 无横线
long snowflakeId = IdUtil.getSnowflakeNextId();
String snowflakeId = IdUtil.getSnowflakeNextIdStr();
```

---

# 九、Convert — 类型转换

```java
import cn.hutool.core.convert.Convert;

// 基本类型
int num = Convert.toInt("123");
boolean bool = Convert.toBool("true");

// 集合
List<Long> ids = Convert.toList(Long.class, "1,2,3,4");

// 日期
Date date = Convert.toDate("2024-01-01");
```

---

# 十、ReUtil & CronUtil

```java
import cn.hutool.core.util.ReUtil;

// 正则：提取 / 匹配 / 替换
ReUtil.get("\\d+", "abc123def", 0)     // "123"
ReUtil.isMatch("\\d+", "123")          // true
ReUtil.replaceAll("123", "\\d", "*")   // "***"

// Cron 表达式
import cn.hutool.cron.CronUtil;
CronUtil.match("0 0 12 * * ?", date)                   // 是否匹配
List<DateTime> dates = CronUtil.getMatchedDates("0 0/30 * * * ?", 5);  // 最近5次执行时间
```
