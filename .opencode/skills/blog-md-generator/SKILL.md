---
name: blog-md-generator
description: 根据用户提供的主题、资料、文章内容或技术文档，生成符合指定规范的 Markdown 博客文件。自动根据内容生成文件名、一级标题和 tags，并确保 Markdown 文件格式统一。
---

# Blog Markdown Generator

## 作用

将用户提供的主题、技术资料、文章、笔记或其他内容，整理并生成一个标准化的 Markdown 博客文件。

生成的文件主要用于个人技术博客、知识库或 Markdown 文档系统。

---

## 核心规则

### 1. 文件格式

最终输出必须是 Markdown 文件：

- 文件后缀只能是 `.md`
- 不允许使用 `.markdown`、`.txt`、`.html` 等其他后缀
- 文件内容必须是合法 Markdown

---

### 2. Front Matter

Markdown 文件开头必须包含 YAML Front Matter。

固定格式：

```yaml
---
tags: ["tag1", "tag2", "tag3"]
---
````

注意：

* `tags` 必须是数组
* 使用双引号包裹每一个 tag
* tag 使用英文、小写、短横线 `-` 分隔
* 不要使用中文 tag
* 不要在 tag 中使用空格
* tag 数量根据文章实际内容决定
* 通常生成 3～8 个 tags
* 不要为了凑数量添加与文章无关的 tag
* tags 必须能够反映文章的核心技术、领域、框架、产品、概念等

例如：

```yaml
---
tags: ["apache-atlas", "data-governance", "metadata-management", "hadoop-ecosystem"]
---
```

---

### 3. 一级标题

Front Matter 后必须存在一个一级标题。

一级标题使用中文，描述文章主题：

```markdown
# Apache Atlas 深度解析
```

一级标题必须与文件名保持一致。

例如：

文件名：

```text
Apache_Atlas深度解析.md
```

文件内容：

```markdown
---
tags: ["apache-atlas", "data-governance", "metadata-management"]
---

# Apache_Atlas深度解析
```

二者必须完全一致。

---

### 4. 文件名生成规则

如果用户没有明确指定文件名，则必须根据文章实际内容自动生成中文文件名。

文件名规则：

* 使用中文
* 不允许出现空格，空格使用下划线 `_` 替代
* 不能包含 `/`、`\` 等会破坏 URL 的字符（文件名会成为 URL）
* 后缀必须为 `.md`
* 文件名应该简洁、明确地表达文章主题
* 不要使用无意义的文件名，例如：

  * `article.md`
  * `test.md`
  * `blog.md`
  * `new.md`
  * `document.md`

例如文章主题：

> Apache Atlas 深度解析：元数据管理与数据治理

应该生成：

```text
Apache_Atlas深度解析.md
```

而不是：

```text
Apache_Atlas.md
```

如果文章主要讨论：

> Apache Kafka 消费者组机制

可以生成：

```text
Apache_Kafka消费者组机制.md
```

如果文章主要讨论：

> Reasoning Model 的发展、原理与技术路线

可以生成：

```text
Reasoning_Model的发展_原理与技术路线.md
```

---

## 5. 文件名与一级标题一致

这是强制规则。

必须满足：

```text
文件名（去掉 .md） == 一级标题内容
```

例如：

```text
Apache_Kafka消费者组机制.md
```

对应：

```markdown
# Apache_Kafka消费者组机制
```

禁止出现：

```text
Apache_Kafka消费者组机制.md

# Kafka 消费者组机制
```

也禁止：

```text
Apache_Kafka消费者组机制.md

# apache-kafka-consumer-groups
```

---

## 6. Tags 生成规则

tags 必须根据文章实际内容动态生成。

优先从以下维度选择：

1. 核心技术
2. 核心框架
3. 核心产品
4. 技术领域
5. 关键概念
6. 所属生态
7. 文章主要应用场景

例如文章：

> Apache Atlas 深度解析，介绍 Atlas 的元数据模型、血缘关系、分类体系以及它在 Hadoop 数据治理体系中的作用。

可以生成：

```yaml
---
tags: ["apache-atlas", "metadata-management", "data-lineage", "data-governance", "hadoop-ecosystem"]
---
```

不要生成：

```yaml
---
tags: ["technology", "blog", "computer", "article", "data"]
---
```

因为这些 tags 过于宽泛，没有实际检索价值。

---

## 7. 内容整理规则

如果用户只提供一个主题：

例如：

> 写一篇 Apache Atlas 深度解析

需要根据主题生成完整的技术博客。

如果用户提供已有资料：

* 不要改变原始技术事实
* 可以重新组织文章结构
* 可以补充必要的解释
* 可以增加示例
* 可以增加代码
* 可以增加架构说明
* 可以增加优缺点分析
* 可以增加使用场景
* 可以增加总结

但不能凭空编造不存在的事实。

如果用户提供的是已经完整的文章：

* 主要进行 Markdown 格式化
* 不要无意义地重写文章
* 保留原始信息
* 根据实际内容生成 tags
* 根据内容生成合适的中文文件名
* 添加符合规则的、与文件名一致的中文一级标题

---

## 8. 推荐的博客结构

如果用户没有指定文章结构，可以根据主题自动选择结构。

技术类文章通常优先考虑：

```markdown
---
tags: ["xxx", "xxx", "xxx"]
---

# xxx

## 什么是 xxx

## 为什么需要 xxx

## 核心概念

## 整体架构

## 核心原理

## 关键组件

## 工作流程

## 实际使用

## 优缺点

## 与其他方案对比

## 常见问题

## 总结
```

不要求所有文章都必须包含这些章节。

应该根据实际主题调整结构。

---

## 9. Markdown 规范

生成内容必须使用标准 Markdown。

代码必须使用 fenced code block：

````markdown
```java
public class Example {
}
````

````

列表使用：

```markdown
- item 1
- item 2
- item 3
````

表格使用标准 Markdown table。

不要使用 HTML 替代普通 Markdown，除非用户明确要求。

---

## 10. 输出要求

当用户要求生成博客文件时，最终应该生成一个真实的 `.md` 文件，而不是只在聊天窗口输出 Markdown。

例如：

```text
Apache_Atlas深度解析.md
```

文件内容：

```markdown
---
tags: ["apache-atlas", "data-governance", "metadata-management", "hadoop-ecosystem"]
---

# Apache_Atlas深度解析

## 什么是 Apache Atlas

...
```

---

## 11. 生成前检查

生成 `.md` 文件前必须执行以下检查：

### 文件检查

* [ ] 后缀是否为 `.md`
* [ ] 文件名是否使用中文
* [ ] 文件名中是否没有空格，空格是否已用 `_` 替代
* [ ] 文件名中是否不包含 `/`、`\` 等特殊字符
* [ ] 是否存在 Front Matter
* [ ] Front Matter 是否包含 `tags`
* [ ] tags 是否使用双引号
* [ ] tags 是否与文章内容相关
* [ ] 是否存在一级标题
* [ ] 一级标题是否使用中文
* [ ] 一级标题是否与文件名完全一致

### 内容检查

* [ ] Markdown 是否结构完整
* [ ] 代码块是否正确闭合
* [ ] 标题层级是否合理
* [ ] 是否存在明显重复内容
* [ ] 是否存在明显无关内容
* [ ] 技术事实是否与用户提供的资料一致

---

## 12. 最终格式示例

对于主题：

> Apache Atlas 深度解析，介绍它的元数据管理、数据血缘、分类以及 Hadoop 生态中的应用。

应该生成：

文件：

```text
Apache_Atlas深度解析.md
```

内容：

```markdown
---
tags: ["apache-atlas", "metadata-management", "data-lineage", "data-governance", "hadoop-ecosystem"]
---

# Apache_Atlas深度解析

## 什么是 Apache Atlas

...

## Apache Atlas 解决什么问题

...

## 元数据管理

...

## 数据血缘

...

## 分类与标签

...

## Hadoop 生态中的应用

...

## 总结

...
```

---

## 最重要的约束

始终遵循：

```text
文章内容
   ↓
分析主题
   ↓
生成 tags
   ↓
生成中文文件名
   ↓
文件名去掉 .md
   ↓
作为一级标题
   ↓
生成 Markdown
   ↓
检查格式
   ↓
输出 .md 文件
```

最终必须满足：

```text
中文文件名.md
    ↓
中文文件名
    ↓
# 中文文件名
```

即：

```text
文件名 == 一级标题
```

这是最高优先级的格式约束。