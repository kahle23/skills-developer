# 代码示例学习模板

当用户提供代码示例时，按照以下流程将其总结并整合到技能文档中。

---

# 一、处理流程

```
收到用户代码示例
│
├─ Step 1: 分析示例 — 提取命名、结构、风格特征
├─ Step 2: 归类 — 判断属于哪个已有文档的范畴
├─ Step 3: 去重 — 检查是否与已有规则冲突或重复
├─ Step 4: 提炼 — 从具体代码中抽象出通用规范
└─ Step 5: 写入 — 按下方模板格式追加到对应文档
```

---

# 二、分析维度

从每个代码示例中提取以下维度：

| 维度 | 关注点 |
|------|--------|
| **命名** | 类名、方法名、变量名、常量名的风格 |
| **结构** | 类的组织顺序、方法长度、嵌套层级 |
| **注解** | 使用了哪些注解，组合方式 |
| **异常** | 如何抛出、捕获、包装异常 |
| **日志** | 日志级别选择、格式化方式 |
| **工具库** | 使用了哪些工具类和方法 |
| **设计模式** | 是否体现了某种设计模式 |
| **注释** | 注释风格、是否有必要 |
| **空值处理** | 如何防御 null |
| **其他** | 任何值得推广的编码习惯 |

---

# 三、归类规则

根据示例内容，将其归入对应文档：

| 示例特征 | 归入文档 |
|---------|---------|
| 命名、类结构、方法规范 | `03-code-standards.md` |
| List/Map/Set/Stream 操作 | `04-collection-and-stream.md` |
| 接口抽象、模式运用 | `05-design-patterns.md` |
| 性能相关写法 | `06-performance.md` |
| Lombok 注解用法 | `21-lombok.md` |
| Hutool 工具类调用 | `22-hutool.md` |
| Guava 工具类调用 | `23-guava.md` |
| Apache Commons 调用 | `24-apache-commons.md` |
| 无法归入以上类别 | `99-others.md` |
| 跨多个类别 | 追加到最相关的文档，必要时在多个文档中引用 |

---

# 四、写入模板

将提炼出的规范按以下格式追加到对应文档的合适位置：

```markdown
### X.Y [从示例中提炼的规则名称]

```java
// 示例代码（精简后，只保留能说明规则的部分）
```

[一句话说明这个规则的要点]
```

**示例：**

假设用户提供了如下代码：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class WarehouseAddParam {
    private static final long serialVersionUID = 1L;
    private String warehouseCode;
    private String warehouseName;
    private String warehouseType;
}
```

提炼后写入 `21-lombok.md`：

```markdown
### 1.3 @Data + 构造方法组合（推荐写法）

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class WarehouseAddParam {
    private String warehouseCode;
    private String warehouseName;
}
```

三注解组合是实体类/参数类的标准写法：@Data 提供基础方法，@NoArgsConstructor/@AllArgsConstructor 兼容序列化框架和手动创建。
```

---

# 五、去重与冲突处理

| 情况 | 处理方式 |
|------|---------|
| 与已有规则完全一致 | 跳过，不重复写入 |
| 与已有规则冲突 | 标注差异，询问用户以哪个为准 |
| 已有规则的补充 | 在已有规则下方追加"补充"段落 |
| 全新规则 | 按模板新增 |

---

# 六、快速判断指令

当用户提供代码示例时，AI 应执行：

```
1. 阅读代码示例
2. 按"分析维度"逐项提取特征
3. 按"归类规则"确定目标文档
4. 按"写入模板"格式生成内容
5. 检查"去重与冲突"
6. 追加到对应文档，告知用户已更新
```

---

# 七、批量示例处理

当用户提供多个代码片段时：

```
1. 逐个分析，提取共性规范
2. 将共性规范合并为一条通用规则
3. 用最有代表性的代码作为示例
4. 其他片段作为补充说明（如有差异）
```
