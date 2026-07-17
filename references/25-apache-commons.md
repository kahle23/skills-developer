# Apache Commons

Apache Commons 提供高质量的 Java 工具库。最常用的有 commons-lang3（字符串/对象/数组）、commons-collections4（集合）、commons-io（IO）。

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

> 另有 `commons-collections4`（4.4）和 `commons-io`（2.15.1），需要时按模块引入。

> **工具选择建议**：项目已有 Hutool 时，判空/集合/日期等优先用 Hutool。Commons 的 `StringUtils` 判空和默认值方法、`FileUtils` 文件操作、`CollectionUtils` 集合运算可作为补充。Guava 覆盖不可变集合/缓存/限流。

---

# 一、StringUtils — 字符串

```java
import org.apache.commons.lang3.StringUtils;

// 判空
isEmpty(null)         // true  — null、"" 为 true
isNotEmpty("abc")     // true
isBlank(null)         // true  — null、""、"  " 都为 true
isNotBlank("abc")     // true
isAnyEmpty("", "a")   // true  — 任一为空
isNoneEmpty("a","b")  // true  — 全部非空

// 默认值
StringUtils.defaultIfEmpty(null, "default")    // "default"
StringUtils.defaultIfBlank(null, "default")    // "default"
StringUtils.defaultIfBlank("  ", "default")    // "default"
```

**其他常用方法**：

| 分类 | 方法 | 说明 |
|------|------|------|
| 去空格 | `trim` / `trimToEmpty` / `trimToNull` / `strip` | null 安全的去空格 |
| 截取 | `substring` / `left` / `right` / `mid` / `substringBefore/After` | 各种截取方式 |
| 大小写 | `upperCase` / `lowerCase` / `capitalize` / `uncapitalize` | null 安全 |
| 填充 | `leftPad` / `rightPad` / `center` | 左/右/居中对齐填充 |
| 重复 | `repeat("ab", 3)` | → "ababab" |
| 包含 | `contains` / `containsIgnoreCase` / `containsAny` | 字符串包含判断 |
| 前后缀 | `startsWith` / `endsWith` / `startsWithIgnoreCase` | 前/后缀判断 |
| 替换 | `replace` / `replaceOnce` / `overlay` | null 安全的替换 |
| 分割 | `split("a,b,c", ",")` | → ["a","b","c"] |
| 连接 | `join(arr, ", ")` / `join(list, "\|")` | 数组/集合连接 |
| 删除 | `remove` / `removeStart` / `removeEnd` | 删除子串 |
| 其他 | `abbreviate` / `reverse` / `countMatches` / `isAlpha` / `isNumeric` | 缩写/反转/计数/判断 |

---

# 二、ObjectUtils — 对象

```java
import org.apache.commons.lang3.ObjectUtils;

ObjectUtils.defaultIfNull(null, "default")     // "default"
ObjectUtils.firstNonNull(null, null, "third")  // "third"
ObjectUtils.compare(1, 2)                      // -1
ObjectUtils.min(1, 2, 3)                       // 1
ObjectUtils.max(1, 2, 3)                       // 3
```

---

# 三、ArrayUtils — 数组

| 方法 | 说明 |
|------|------|
| `isEmpty` / `isNotEmpty` | 判空 |
| `add` / `addAll` | 添加/合并（返回新数组） |
| `remove` / `removeElement` | 删除指定索引/元素 |
| `contains` | 是否包含 |
| `indexOf` / `lastIndexOf` | 查找索引 |
| `reverse` | 反转 |
| `toObject` / `toPrimitive` | 装箱/拆箱 |
| `getLength(array)` | null 返回 0 |

---

# 四、CollectionUtils / MapUtils — 集合

```java
import org.apache.commons.collections4.CollectionUtils;
import org.apache.commons.collections4.MapUtils;

// 判空 & 默认值
CollectionUtils.isEmpty(list) / CollectionUtils.isNotEmpty(list)
CollectionUtils.emptyIfNull(list)  // null → 空集合

// 集合运算
CollectionUtils.intersection(list1, list2)   // 交集
CollectionUtils.union(list1, list2)          // 并集
CollectionUtils.subtract(list1, list2)       // 差集

// 过滤 & 转换
CollectionUtils.filter(list, u -> u.getAge() > 18);          // 原地过滤
List<User> filtered = CollectionUtils.select(list, u -> u.getAge() > 18);  // 新集合
List<String> names = CollectionUtils.collect(users, User::getName);

// MapUtils
MapUtils.isEmpty(map) / MapUtils.isNotEmpty(map)
MapUtils.emptyIfNull(map)
Long val = MapUtils.getLong(map, "key", 0L);  // 带默认值
```

---

# 五、文件工具

```java
import org.apache.commons.io.FilenameUtils;
import org.apache.commons.io.FileUtils;

// 文件名
FilenameUtils.getExtension(path)       // "txt"
FilenameUtils.getBaseName(path)        // "file"
FilenameUtils.getName(path)            // "file.txt"
FilenameUtils.normalize(path)          // 规范化路径

// 文件读写
FileUtils.readFileToString(file, "UTF-8")
FileUtils.readLines(file, "UTF-8")
FileUtils.writeStringToFile(file, "content", "UTF-8")

// 文件操作
FileUtils.copyFile(src, dest)
FileUtils.copyDirectory(srcDir, destDir)
FileUtils.deleteDirectory(dir)
FileUtils.deleteQuietly(file)          // 不抛异常
FileUtils.moveFile(src, dest)
FileUtils.forceMkdir(dir)
FileUtils.listFiles(dir, {"txt"}, true)  // 递归列出文件

// 文件大小
long size = FileUtils.sizeOfDirectory(dir);
String human = FileUtils.byteCountToDisplaySize(size);  // "1.5 MB"
```

---

# 六、其他工具

```java
import org.apache.commons.lang3.BooleanUtils;
import org.apache.commons.lang3.math.NumberUtils;
import org.apache.commons.lang3.EnumUtils;

// BooleanUtils
BooleanUtils.toBoolean(1)             // true
BooleanUtils.negate(true)             // false

// NumberUtils
NumberUtils.toInt("123")              // 123
NumberUtils.max(1, 2, 3) / NumberUtils.min(1, 2, 3)  // 3 / 1

// EnumUtils
Status s = EnumUtils.getEnum(Status.class, "ACTIVE");
boolean valid = EnumUtils.isValidEnum(Status.class, "ACTIVE");
```

---

# 七、工具选择对比

| 场景 | Hutool | Guava | Apache Commons |
|------|--------|-------|----------------|
| 字符串判空 | `StrUtil.isBlank` | — | `StringUtils.isBlank` |
| 集合判空 | `CollUtil.isEmpty` | — | `CollectionUtils.isEmpty` |
| 不可变集合 | — | `ImmutableList.of` | — |
| 集合分组 | `CollUtil.groupBy` | `Multimaps.index` | `CollectionUtils.collect` |
| 日期处理 | `DateUtil` | — | — |
| JSON 处理 | `JSONUtil` | — | — |
| Bean 拷贝 | `BeanUtil.copyProperties` | — | — |
| 本地缓存 | — | `CacheBuilder` | — |
| 文件操作 | `FileUtil` | — | `FileUtils` |
| 前置校验 | — | `Preconditions` | — |
| 限流 | — | `RateLimiter` | — |
