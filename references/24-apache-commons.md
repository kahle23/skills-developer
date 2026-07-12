# Apache Commons

Apache Commons 提供了一系列高质量的 Java 工具库。最常用的有 commons-lang3（字符串/对象/数组）、commons-collections4（集合）、commons-io（IO）。

```xml
<!-- 字符串、对象、数组、枚举等 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>

<!-- 集合扩展 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.4</version>
</dependency>

<!-- IO 工具 -->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.15.1</version>
</dependency>
```

---

# 一、StringUtils — 字符串工具

```java
import org.apache.commons.lang3.StringUtils;

// 判空
StringUtils.isEmpty(null)        // true — null、"" 为 true
StringUtils.isNotEmpty("abc")   // true
StringUtils.isBlank(null)        // true — null、""、"  " 都为 true
StringUtils.isNotBlank("abc")   // true
StringUtils.isAnyEmpty("", "a") // true — 任一为空
StringUtils.isAnyBlank(" ", "a")// true — 任一为空白
StringUtils.isNoneEmpty("a","b") // true — 全部非空

// 默认值
StringUtils.defaultIfEmpty(null, "default")    // "default"
StringUtils.defaultIfEmpty("", "default")      // "default"
StringUtils.defaultIfEmpty("  ", "default")    // "  "（空白不算空）
StringUtils.defaultIfBlank(null, "default")    // "default"
StringUtils.defaultIfBlank("  ", "default")    // "default"

// 去空格
StringUtils.trim(null)           // null
StringUtils.trimToEmpty(null)    // "" — null 安全
StringUtils.trimToNull("  ")     // null — 空白返回 null
StringUtils.strip(null)          // null（类似 trim，但支持 Unicode 空白）
StringUtils.stripToEmpty(null)   // ""

// 截取
StringUtils.substring("abcdef", 0, 3)    // "abc"
StringUtils.left("abcdef", 3)            // "abc"
StringUtils.right("abcdef", 3)           // "def"
StringUtils.mid("abcdef", 2, 3)          // "cde"
StringUtils.substringBefore("a.b.c", ".")  // "a"
StringUtils.substringAfter("a.b.c", ".")   // "b.c"
StringUtils.substringBeforeLast("a.b.c", ".") // "a.b"
StringUtils.substringAfterLast("a.b.c", ".")  // "c"

// 大小写
StringUtils.upperCase("hello")     // "HELLO"
StringUtils.lowerCase("HELLO")     // "hello"
StringUtils.capitalize("hello")    // "Hello"（首字母大写）
StringUtils.uncapitalize("Hello")  // "hello"（首字母小写）
StringUtils.swapCase("Hello")      // "hELLO"

// 填充
StringUtils.leftPad("5", 3, '0')   // "005"
StringUtils.rightPad("5", 3, '0')  // "500"
StringUtils.center("hi", 6, '*')   // "**hi**"

// 重复
StringUtils.repeat("ab", 3)        // "ababab"

// 包含
StringUtils.contains("abc", "b")   // true
StringUtils.containsIgnoreCase("ABC", "b")  // true
StringUtils.containsAny("abc", "x", "b")    // true — 任一包含

// 前后缀
StringUtils.startsWith("abc", "a")     // true
StringUtils.startsWithIgnoreCase("ABC", "a")  // true
StringUtils.endsWith("abc", "c")       // true

// 替换
StringUtils.replace("aabbcc", "b", "x")  // "aaxxcc"
StringUtils.replaceOnce("aabbcc", "b", "x")  // "aaxbcc"
StringUtils.overlay("abcdef", "xx", 2, 4)  // "abxxef"（覆盖 2~4 位）

// 分割
StringUtils.split("a,b,c", ",")      // ["a", "b", "c"]
StringUtils.split("a , b , c", ", ")  // ["a", "b", "c"]（自动去空格）

// 连接
StringUtils.join(new String[]{"a","b","c"}, ", ")  // "a, b, "c"
StringUtils.join(Arrays.asList("a","b","c"), "|")  // "a|b|c"

// 删除
StringUtils.remove("abccb", "b")     // "acc"
StringUtils.removeStart("foobar", "foo")  // "bar"
StringUtils.removeEnd("foobar", "bar")    // "foo"

// 缩写
StringUtils.abbreviate("abcdefghij", 7)     // "abcd..."
StringUtils.abbreviate("abcdefghij", 5, 7)  // "...ef..."

// 反转
StringUtils.reverse("abc")         // "cba"

// 随机
StringUtils.randomAlphabetic(5)    // "xKmQp"（5个字母）
StringUtils.randomAlphanumeric(8)  // "a3bK9mP2"（8个字母数字）
StringUtils.randomNumeric(4)       // "3847"（4个数字）

// 计数
StringUtils.countMatches("aabbcc", "b")  // 2

// 首次/末次出现位置
StringUtils.indexOf("abcabc", "b")       // 1
StringUtils.lastIndexOf("abcabc", "b")   // 4

// 判断是否纯字母/纯数字
StringUtils.isAlpha("abc")        // true
StringUtils.isNumeric("123")      // true
StringUtils.isAlphanumeric("a1b") // true
```

---

# 二、ObjectUtils — 对象工具

```java
import org.apache.commons.lang3.ObjectUtils;

// 默认值
ObjectUtils.defaultIfNull(null, "default")  // "default"
ObjectUtils.firstNonNull(null, null, "third")  // "third"

// 比较
ObjectUtils.compare(1, 2)          // -1
ObjectUtils.compare(2, 1)          // 1
ObjectUtils.compare(1, 1)          // 0
ObjectUtils.min(1, 2, 3)           // 1
ObjectUtils.max(1, 2, 3)           // 3

// 克隆
User clone = ObjectUtils.clone(user);  // 实现 Cloneable 的对象

// 转字符串
ObjectUtils.toString(null)         // ""
ObjectUtils.toString(obj)          // obj.toString()
ObjectUtils.toString(obj, "N/A")   // "N/A" if null

// 判断
ObjectUtils.notEqual(obj1, obj2)   // true
ObjectUtils.isNotEmpty(obj)        // true
ObjectUtils.isEmpty(obj)           // true
```

---

# 三、ArrayUtils — 数组工具

```java
import org.apache.commons.lang3.ArrayUtils;

// 判空
ArrayUtils.isEmpty(array)          // true
ArrayUtils.isNotEmpty(array)       // true

// 添加
ArrayUtils.add(array, element)     // 返回新数组
ArrayUtils.addAll(array1, array2)  // 合并数组

// 删除
ArrayUtils.remove(array, index)    // 删除指定索引
ArrayUtils.removeElement(array, element)  // 删除指定元素

// 包含
ArrayUtils.contains(array, element)  // true

// 查找
int index = ArrayUtils.indexOf(array, element);
int index = ArrayUtils.lastIndexOf(array, element);

// 反转
ArrayUtils.reverse(array);

// 转换
Integer[] boxed = ArrayUtils.toObject(intArray);
int[] primitive = ArrayUtils.toPrimitive(IntegerArray);

// null 安全
int length = ArrayUtils.getLength(array);  // null 返回 0
```

---

# 四、CollectionUtils — 集合工具（commons-collections4）

```java
import org.apache.commons.collections4.CollectionUtils;
import org.apache.commons.collections4.MapUtils;

// 判空
CollectionUtils.isEmpty(list)           // true
CollectionUtils.isNotEmpty(list)        // true

// 默认值
CollectionUtils.emptyIfNull(list)       // null → 空集合

// 集合运算
Collection<String> intersection = CollectionUtils.intersection(list1, list2);  // 交集
Collection<String> union = CollectionUtils.union(list1, list2);                // 并集
Collection<String> subtract = CollectionUtils.subtract(list1, list2);          // 差集
Collection<String> disjunction = CollectionUtils.disjunction(list1, list2);    // 对称差集

// 包含
boolean contains = CollectionUtils.containsAny(list1, list2);  // 任一包含

// 过滤
CollectionUtils.filter(list, user -> user.getAge() > 18);  // 原地过滤
List<User> filtered = CollectionUtils.select(list, user -> user.getAge() > 18);  // 返回新集合

// 转换
List<String> names = CollectionUtils.collect(users, User::getName);

// 统计
int count = CollectionUtils.countMatches(list, user -> user.getAge() > 18);

// 判断
boolean equal = CollectionUtils.isEqualCollection(list1, list2);

// Map 工具
MapUtils.isEmpty(map)             // true
MapUtils.isNotEmpty(map)          // true
MapUtils.emptyIfNull(map)         // null → 空 Map
Long val = MapUtils.getLong(map, "key", 0L);  // 带默认值
```

---

# 五、FilenameUtils / FileUtils — 文件工具（commons-io）

```java
import org.apache.commons.io.FilenameUtils;
import org.apache.commons.io.FileUtils;
import org.apache.commons.io.IOUtils;

// 文件名操作
String ext = FilenameUtils.getExtension("/path/file.txt");       // "txt"
String name = FilenameUtils.getBaseName("/path/file.txt");       // "file"
String name = FilenameUtils.getName("/path/file.txt");           // "file.txt"
String path = FilenameUtils.getFullPath("/path/file.txt");       // "/path/"
String path = FilenameUtils.normalize("/path/../path/file.txt"); // "/path/file.txt"
String path = FilenameUtils.concat("/path", "file.txt");         // "/path/file.txt"
boolean equals = FilenameUtils.equals("file.txt", "FILE.TXT", true, IOCase.INSENSITIVE);

// 文件读写
String content = FileUtils.readFileToString(file, "UTF-8");
List<String> lines = FileUtils.readLines(file, "UTF-8");
FileUtils.write(file, "content", "UTF-8");
FileUtils.writeLines(file, "UTF-8", lines);
FileUtils.writeStringToFile(file, "content", "UTF-8", false);  // append=false

// 文件操作
FileUtils.copyFile(srcFile, destFile);
FileUtils.copyDirectory(srcDir, destDir);
FileUtils.copyInputStreamToFile(inputStream, destFile);
FileUtils.deleteDirectory(dir);
FileUtils.deleteQuietly(file);  // 不抛异常
FileUtils.forceDelete(file);
FileUtils.moveFile(srcFile, destFile);
FileUtils.moveDirectory(srcDir, destDir);

// 目录操作
FileUtils.forceMkdir(dir);
Collection<File> files = FileUtils.listFiles(dir, {"txt", "csv"}, true);  // 递归
Collection<File> files = FileUtils.listFilesAndDirs(dir, TrueFileFilter.INSTANCE, null);

// 文件大小
long size = FileUtils.sizeOfDirectory(dir);
String humanSize = FileUtils.byteCountToDisplaySize(size);  // "1.5 MB"

// IO 流操作
String content = IOUtils.toString(inputStream, "UTF-8");
byte[] bytes = IOUtils.toByteArray(inputStream);
IOUtils.copy(inputStream, outputStream);
IOUtils.closeQuietly(inputStream);  // 静默关闭

// 临时文件
File tempFile = File.createTempFile("prefix", ".tmp");
tempFile.deleteOnExit();  // JVM 退出时删除
```

---

# 六、BooleanUtils / NumberUtils / EnumUtils

```java
import org.apache.commons.lang3.BooleanUtils;
import org.apache.commons.lang3.math.NumberUtils;
import org.apache.commons.lang3.EnumUtils;

// BooleanUtils
BooleanUtils.toBoolean(1)           // true
BooleanUtils.toBoolean("yes")       // true
BooleanUtils.toInteger(true)        // 1
BooleanUtils.toString(true, "Y", "N")  // "Y"
BooleanUtils.negate(true)           // false

// NumberUtils
NumberUtils.toInt("123")            // 123
NumberUtils.toLong("123")           // 123L
NumberUtils.toDouble("123.45")      // 123.45
NumberUtils.max(1, 2, 3)            // 3
NumberUtils.min(1, 2, 3)            // 1
NumberUtils.isCreatable("123")      // true
NumberUtils.isParsable("123.45")    // true
BigDecimal bd = NumberUtils.createBigDecimal("123.45");
long[] range = NumberUtils.createLongArray("1,2,3");

// EnumUtils（需要 @EnumValue 或标准枚举）
enum Status { ACTIVE, INACTIVE }
Status status = EnumUtils.getEnum(Status.class, "ACTIVE");
List<String> names = EnumUtils.getEnumList(Status.class);  // ["ACTIVE", "INACTIVE"]
Map<String, Status> map = EnumUtils.getEnumMap(Status.class);
boolean valid = EnumUtils.isValidEnum(Status.class, "ACTIVE");  // true
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
| 前置校验 | — | `Preconditions` | `Validate` |
