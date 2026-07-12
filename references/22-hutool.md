# Hutool 工具库

Hutool 是一个 Java 工具集，提供了大量实用的工具类，覆盖字符串、集合、日期、JSON、Bean、加密、HTTP 等场景。

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.8.25</version>
</dependency>
<!-- 也可以按需引入子模块 -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-core</artifactId>
    <version>5.8.25</version>
</dependency>
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-json</artifactId>
    <version>5.8.25</version>
</dependency>
```

---

# 一、StrUtil — 字符串工具

```java
import cn.hutool.core.util.StrUtil;

// 判空
StrUtil.isBlank(null)        // true  — null、""、"  " 都为 true
StrUtil.isNotBlank("abc")    // true
StrUtil.isEmpty(null)        // true  — null、"" 为 true，"  " 为 false
StrUtil.isNotEmpty("abc")    // true
StrUtil.hasBlank("a", "")    // true  — 任一为空
StrUtil.hasEmpty("a", null)  // true  — 任一为空

// 去空格
StrUtil.trim(null)           // null
StrUtil.trim("  abc  ")      // "abc"
StrUtil.trimToEmpty(null)    // ""    — null 安全，返回空串
StrUtil.trimToNull("  ")     // null  — 空白返回 null

// 格式化
StrUtil.format("用户{}的订单{}", userId, orderId)
// "用户123的订单456"

// 分割
List<String> parts = StrUtil.split("a,b,c", ',');         // ["a", "b", "c"]
List<String> parts = StrUtil.splitTrim(" a , b , c ", ','); // ["a", "b", "c"]（去空格）

// 截取
StrUtil.sub("abcdef", 0, 3)    // "abc"
StrUtil.subBefore("a.b.c", '.', false)  // "a"
StrUtil.subAfter("a.b.c", '.', false)   // "b.c"

// 大小写
StrUtil.upperFirst("hello")   // "Hello"
StrUtil.lowerFirst("Hello")   // "hello"

// 下划线 ↔ 驼峰
StrUtil.toCamelCase("user_name")     // "userName"
StrUtil.toUnderlineCase("userName")  // "user_name"

// 包装
StrUtil.wrap("hello", "\"")   // "\"hello\""
StrUtil.unWrap("\"hello\"", "\"")  // "hello"

// 重复
StrUtil.repeat("ab", 3)       // "ababab"

// 填充
StrUtil.padBefore("5", 3, '0')  // "005"
StrUtil.padAfter("5", 3, '0')  // "500"
```

---

# 二、CollUtil — 集合工具

```java
import cn.hutool.core.collection.CollUtil;

// 判空
CollUtil.isEmpty(list)         // true
CollUtil.isNotEmpty(list)      // true

// 新建集合
List<String> list = CollUtil.newArrayList("a", "b", "c");
Set<Long> set = CollUtil.newHashSet(1L, 2L, 3L);
Map<String, Object> map = CollUtil.newHashMap();
LinkedList<String> linked = CollUtil.newLinkedList();

// 分组
Map<Integer, List<User>> groupMap = CollUtil.groupByField(userList, "age");
Map<String, List<User>> groupMap = CollUtil.groupBy(userList, User::getDeptName);

// 集合运算
List<String> intersection = CollUtil.intersection(list1, list2);  // 交集
List<String> union = CollUtil.union(list1, list2);                // 并集
List<String> disjunction = CollUtil.disjunction(list1, list2);    // 对称差集
List<String> subtract = CollUtil.subtract(list1, list2);          // 差集

// 取首尾
String first = CollUtil.getFirst(list);
String last = CollUtil.getLast(list);

// 分页
List<List<String>> partition = CollUtil.split(list, 3);  // 每3个一组

// 排序
List<User> sorted = CollUtil.sort(userList, Comparator.comparing(User::getAge));

// 过滤
List<User> filtered = CollUtil.filter(userList, user -> user.getAge() > 18);
// filter 返回新集合，filterSelf 修改原集合

// 提取字段
List<Long> ids = CollUtil.getFieldValues(userList, "id", Long.class);

// 转 Map
Map<Long, User> map = CollUtil.toMap(userList, User::getId, true);

// 包含
boolean has = CollUtil.contains(list, "a");
boolean any = CollUtil.containsAny(list1, list2);

// 转不可变集合
List<String> immutable = CollUtil.unmodifiable(list);
```

---

# 三、ObjectUtil — 对象工具

```java
import cn.hutool.core.util.ObjectUtil;

// 判空
ObjectUtil.isNull(obj)           // true
ObjectUtil.isNotNull(obj)        // true
ObjectUtil.isEmpty(obj)          // true — 对于集合、数组、字符串、Map 等
ObjectUtil.isNotEmpty(obj)       // true

// 默认值
String val = ObjectUtil.defaultIfNull(input, "默认值");
String val = ObjectUtil.defaultIfEmpty(input, "默认值");  // null/""/"  " 都返回默认值

// 比较
boolean eq = ObjectUtil.equal(obj1, obj2);
int compare = ObjectUtil.compare(obj1, obj2);

// 克隆
User clone = ObjectUtil.clone(user);
User clone = ObjectUtil.cloneByStream(user);  // 深克隆（序列化方式）

// 序列化
byte[] bytes = ObjectUtil.serialize(user);
User user = ObjectUtil.deserialize(bytes);

// 转换
Long val = ObjectUtil.toLong("123");
Integer val = ObjectUtil.toInt("123");
String str = ObjectUtil.toStr(obj);
```

---

# 四、DateUtil — 日期工具

```java
import cn.hutool.core.date.DateUtil;
import cn.hutool.core.date.DateUnit;

// 当前时间
Date now = DateUtil.date();
DateTime now = DateUtil.date();  // DateTime 继承自 Date
long millis = DateUtil.current(false);  // 当前毫秒
long seconds = DateUtil.currentSeconds();  // 当前秒

// 字符串 → 日期
Date date = DateUtil.parse("2024-01-01");
Date date = DateUtil.parse("2024-01-01 12:30:00");
Date date = DateUtil.parse("20240101", "yyyyMMdd");
Date date = DateUtil.parseDate("2024-01-01");

// 日期 → 字符串
String str = DateUtil.format(date, "yyyy-MM-dd");          // "2024-01-01"
String str = DateUtil.formatDateTime(date);                // "2024-01-01 12:30:00"
String str = DateUtil.formatDate(date);                    // "2024-01-01"
String str = DateUtil.formatTime(date);                    // "12:30:00"
String str = DateUtil.format(date, "yyyy年MM月dd日");       // "2024年01月01日"

// 日期计算
Date tomorrow = DateUtil.offsetDay(date, 1);               // +1天
Date yesterday = DateUtil.offsetDay(date, -1);             // -1天
Date nextMonth = DateUtil.offsetMonth(date, 1);            // +1月
Date nextYear = DateUtil.offsetYear(date, 1);              // +1年
Date nextHour = DateUtil.offsetHour(date, 2);              // +2小时

// 时间差
long days = DateUtil.between(date1, date2, DateUnit.DAY);        // 天数差
long hours = DateUtil.between(date1, date2, DateUnit.HOUR);      // 小时差
long millis = DateUtil.between(date1, date2, DateUnit.MS);       // 毫秒差
String between = DateUtil.formatBetween(date1, date2, false);    // "3天2小时5分钟"

// 日期信息
int year = DateUtil.year(date);
int month = DateUtil.month(date);          // 0-11（Calendar 规范）或使用 month+1
int day = DateUtil.dayOfMonth(date);
int dayOfWeek = DateUtil.dayOfWeek(date);  // 1=周日, 2=周一...
String week = DateUtil.dayOfWeekEnum(date).toChinese();  // "星期一"
boolean isWeekend = DateUtil.isWeekend(date);

// 一天的开始/结束
Date beginOfDay = DateUtil.beginOfDay(date);   // 2024-01-01 00:00:00
Date endOfDay = DateUtil.endOfDay(date);       // 2024-01-01 23:59:59
Date beginOfMonth = DateUtil.beginOfMonth(date);
Date endOfMonth = DateUtil.endOfMonth(date);
Date beginOfYear = DateUtil.beginOfYear(date);
Date endOfYear = DateUtil.endOfYear(date);

// 判断
boolean sameDay = DateUtil.isSameDay(date1, date2);
boolean inRange = DateUtil.isIn(date, start, end);  // 是否在范围内

// 年龄
int age = DateUtil.age(date, new Date());
```

---

# 五、JSONUtil — JSON 工具

```java
import cn.hutool.json.JSONUtil;

// 对象 → JSON
String json = JSONUtil.toJsonStr(user);
String pretty = JSONUtil.toJsonPrettyStr(user);  // 格式化

// JSON → 对象
User user = JSONUtil.toBean(json, User.class);
Map<String, Object> map = JSONUtil.toBean(json, Map.class);

// JSON → List
List<User> users = JSONUtil.toList(json, User.class);

// JSON → JSONObject / JSONArray
JSONObject obj = JSONUtil.parseObj(json);
JSONArray arr = JSONUtil.parseArray(jsonStr);

// JSONObject 操作
JSONObject obj = JSONUtil.parseObj(json);
String name = obj.getStr("name");
Long id = obj.getLong("id");
Integer age = obj.getInt("age");
JSONObject nested = obj.getJSONObject("address");
String city = obj.getByPath("address.city", String.class);  // 路径提取
obj.set("name", "新名称");  // 设置值
obj.remove("name");          // 删除

// JSONArray 操作
JSONArray arr = JSONUtil.parseArray(jsonStr);
JSONObject first = arr.getJSONObject(0);
String first = arr.getStr(0);
int size = arr.size();
arr.add(new JSONObject().set("key", "value"));
arr.remove(0);

// 判断
boolean isJson = JSONUtil.isJson(jsonStr);
boolean isType = JSONUtil.isJsonObj(jsonStr);  // 是否是 JSON 对象
boolean isArray = JSONUtil.isJsonArray(jsonStr); // 是否是 JSON 数组

// XML ↔ JSON
String xml = JSONUtil.toXml(jsonObj);
JSONObject obj = JSONUtil.parseXml(xmlStr);

// Bean ↔ JSONObject
JSONObject obj = JSONUtil.parseObj(user);
User user = obj.toBean(User.class);
```

---

# 六、BeanUtil — Bean 工具

```java
import cn.hutool.core.bean.BeanUtil;
import cn.hutool.core.bean.copier.CopyOptions;

// Bean → Map
Map<String, Object> map = BeanUtil.beanToMap(user);
Map<String, Object> map = BeanUtil.beanToMap(user, false, true);  // ignoreNulls, toCamelCase

// Map → Bean
User user = BeanUtil.mapToBean(map, User.class, true);  // ignoreError

// Bean 拷贝
UserDTO dto = new UserDTO();
BeanUtil.copyProperties(user, dto);
BeanUtil.copyProperties(user, dto, "id", "createTime");  // 排除字段

// 拷贝到新对象
UserDTO dto = BeanUtil.copyProperties(user, UserDTO.class);
UserDTO dto = BeanUtil.toBean(user, UserDTO.class);

// 列表拷贝
List<UserDTO> dtos = BeanUtil.copyToList(users, UserDTO.class);

// 自定义拷贝选项
BeanUtil.copyProperties(user, dto, CopyOptions.create()
    .setIgnoreNullValue(true)        // 忽略 null
    .setIgnoreError(true)            // 忽略转换错误
    .setIgnoreCase(true)             // 忽略大小写
    .setFieldNameEditor(name -> "prefix_" + name)  // 字段名转换
);

// 属性获取
Object val = BeanUtil.getProperty(user, "name");

// 判断 Bean 是否为空
boolean empty = BeanUtil.isEmpty(user);  // 所有字段都为 null
```

---

# 七、NumberUtil — 数字工具

```java
import cn.hutool.core.util.NumberUtil;

// 算术运算（BigDecimal 精度）
BigDecimal result = NumberUtil.add(1.1, 2.2);         // 3.3
BigDecimal result = NumberUtil.sub(3.3, 1.1);         // 2.2
BigDecimal result = NumberUtil.mul(2.0, 3.0);         // 6.0
BigDecimal result = NumberUtil.div(10, 3);             // 3.333333...
BigDecimal result = NumberUtil.div(10, 3, 2);          // 3.33（保留2位小数）
BigDecimal result = NumberUtil.round(3.14159, 2);      // 3.14

// 判断
boolean isNum = NumberUtil.isNumber("123.45");
boolean isInt = NumberUtil.isInteger("123");
boolean isBetween = NumberUtil.isBetween(5, 1, 10);    // true

// 格式化
String str = NumberUtil.decimalFormat(",##0.00", 1234567.89);  // "1,234,567.89"
String str = NumberUtil.decimalFormat("0.00", 3.14159);        // "3.14"

// 进制转换
String hex = NumberUtil.toHex(255);           // "ff"
int dec = NumberUtil.hexToInt("ff");          // 255

// 生成随机数
int random = NumberUtil.generateRandomNumber(100, 999);  // 100~999
```

---

# 八、IdUtil — ID 生成工具

```java
import cn.hutool.core.util.IdUtil;

// UUID（去横线）
String uuid = IdUtil.fastSimpleUUID();   // "5a6b7c8d9e0f..."

// 雪花 ID
long snowflakeId = IdUtil.getSnowflakeNextId();
String snowflakeId = IdUtil.getSnowflakeNextIdStr();

// 自定义雪花（指定 workerId 和 datacenterId）
IdWorker idWorker = new IdWorker(1, 1);
long id = idWorker.nextId();

// 短 UUID
String shortId = IdUtil.fastUUID();      // 带横线
String nanoId = IdUtil.nanoId(32);       // NanoID，自定义长度
```

---

# 九、Convert — 类型转换工具

```java
import cn.hutool.core.convert.Convert;

// 基本类型转换
int num = Convert.toInt("123");
long num = Convert.toLong("123");
double num = Convert.toDouble("123.45");
boolean bool = Convert.toBool("true");

// 集合转换
List<Long> ids = Convert.toList(Long.class, "1,2,3,4");
List<String> list = Convert.toList(String.class, listObj);

// 数组转换
Long[] ids = Convert.toArray(Long.class, "1,2,3");

// 日期转换
Date date = Convert.toDate("2024-01-01");
Date date = Convert.toDate("20240101", "yyyyMMdd");

// 枚举转换
MyEnum e = Convert.toEnum(MyEnum.class, 1);
MyEnum e = Convert.toEnumIgnoreException(MyEnum.class, 999);  // 不抛异常

// 字符集转换
String gbk = Convert.convertStr(charset, "GBK", str);
byte[] bytes = Convert.convert(Charset.forName("GBK"), byte[].class, str);
```

---

# 十、ReUtil — 正则工具

```java
import cn.hutool.core.util.ReUtil;

// 提取
String num = ReUtil.get("\\d+", "abc123def", 0);     // "123"
List<String> all = ReUtil.findAll("\\d+", "a1b2c3");  // ["1", "2", "3"]

// 匹配
boolean match = ReUtil.isMatch("\\d+", "123");        // true

// 替换
String result = ReUtil.replaceAll("123", "\\d", "*");  // "***"
String result = ReUtil.delFirst("\\d+", "abc123def456");  // "abcdef456"

// 分组提取
String name = ReUtil.getGroup1("(\\w+)@(\\w+\\.\\w+)", "test@example.com");  // "test"
```

---

# 十一、CronUtil — Cron 表达式工具

```java
import cn.hutool.cron.CronUtil;

// 解析 cron 表达式
boolean match = CronUtil.match("0 0 12 * * ?", date);  // 是否匹配

// 获取最近 N 次执行时间
List<DateTime> dates = CronUtil.getMatchedDates("0 0/30 * * * ?", 5);
```
