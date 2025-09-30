---
title: 'Elasticsearch Dynamic Templates 配置'
date: '2024-06-21'
tags: ['blog', 'Elasticsearch', 'DynamicTemplates']
draft: false
summary: Elasticsearch Dynamic Templates
---

# Elasticsearch Dynamic Templates

Dynamic Templates 是 Elasticsearch 中的一种机制，用于动态映射新字段。当索引中出现新的、未定义的字段时，Dynamic Templates 可以自动应用预定义的规则来确定这些字段的映射。这样可以避免手动更新映射，确保新字段得到适当的处理。

如下的配置案例中：

```json
"dynamic_templates": [
  {
    "all_fields": {
      "path_match": "*",
      "mapping": {
        "index": "false",
        "store": "false"
      }
    }
  }
]
```

这个动态模板匹配所有字段（`path_match: "*"`），并将其映射为不被索引和不被存储（`index: false` 和 `store: false`）。

### Mapping 中的 `index` 和 `store`

在 Elasticsearch 的映射（Mapping）中，`index` 和 `store` 是两个重要的属性，用于控制字段的索引和存储行为：

#### `index`

- **`index: true`**: 表示该字段会被索引，索引后的字段可以用于搜索和查询。这是默认设置。被索引的字段会占用额外的存储空间，但能提供快速的搜索功能。
- **`index: false`**: 表示该字段不会被索引。这样的字段无法通过搜索查询找到，只能通过文档ID直接检索。这个设置可以减少索引的大小和提升索引速度，但会失去搜索该字段的能力。

#### `store`

- **`store: true`**: 表示该字段的原始值会被单独存储，这样可以在搜索结果中直接返回字段值，而不需要从 `_source`（整个文档）中提取。单独存储的字段会增加存储需求和索引时间。
- **`store: false`**: 表示该字段的原始值不会被单独存储，只能从 `_source` 中提取。这是默认设置。一般情况下，不需要将字段单独存储，因为从 `_source` 中提取值已经足够。

### 示例和作用

假设有一个包含用户信息的文档结构：

```json
{
  "name": "John Doe",
  "age": 30,
  "address": "123 Main St"
}
```

1. **`index: true` 和 `store: false`**（默认设置）：
- `name` 字段被索引，可以搜索 `name: "John Doe"` 找到这个文档。
- `name` 字段不单独存储，搜索结果中通过从 `_source` 中提取 `name` 的值来返回。

2. **`index: false` 和 `store: false`**：
- `address` 字段不被索引，不能搜索 `address: "123 Main St"` 找到这个文档。
- `address` 字段不单独存储，只能从 `_source` 中提取 `address` 的值来返回。

3. **`index: true` 和 `store: true`**：
- `age` 字段被索引，可以搜索 `age: 30` 找到这个文档。
- `age` 字段单独存储，搜索结果中直接返回 `age` 的值，而不需要从 `_source` 中提取。

通过配置 `index` 和 `store`，可以优化 Elasticsearch 索引的存储和搜索性能，根据具体需求决定哪些字段需要搜索和存储。

### `_source` 提取

在 Elasticsearch 中，`_source` 字段存储了文档的原始 JSON 数据。当你向 Elasticsearch 索引一个文档时，这个文档的完整 JSON 结构会被存储在 `_source` 字段中。 `_source` 提取指的是从这个存储的原始 JSON 数据中检索字段值。

#### `_source` 提取的工作机制

1. **存储**：
当你将一个文档索引到 Elasticsearch 中，整个文档的 JSON 结构会被存储在 `_source` 字段中。这意味着，即使你设置某些字段不被索引或单独存储，整个文档的原始数据仍然会保存在 `_source` 中。

示例文档：

```json
{
  "name": "John Doe",
  "age": 30,
  "address": "123 Main St"
}
   ```

2. **检索**：
当你查询一个文档时，默认情况下，Elasticsearch 会返回 `_source` 字段的内容。你可以从 `_source` 中提取任何字段的值，而无需单独存储这些字段。

例如，当你查询上述文档时，Elasticsearch 会返回如下结果：

```json
{
  "_index": "your_index",
  "_type": "_doc",
  "_id": "1",
  "_source": {
    "name": "John Doe",
    "age": 30,
    "address": "123 Main St"
  }
}
   ```

### `store` 属性的作用

在映射中设置 `store` 属性为 `true`，则会将字段的值单独存储在索引中，而不仅仅是存储在 `_source` 字段中。这样做的好处是在需要返回特定字段时，可以直接从存储中获取这些字段的值，而不需要提取整个 `_source`。

#### 对比 `_source` 提取和单独存储

- **默认（`store: false`）**：
- 字段值存储在 `_source` 中。
- 查询时，从 `_source` 中提取字段值。
- 优点：节省存储空间，适用于大多数情况。
- 缺点：提取特定字段值时，可能需要处理整个文档的 `_source`。

- **单独存储（`store: true`）**：
- 字段值单独存储。
- 查询时，直接从存储中获取字段值，而不需要处理整个 `_source`。
- 优点：查询性能更高，适合需要频繁访问特定字段的情况。
- 缺点：增加存储需求。

### 示例

假设你有如下映射：

```json
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "store": true
      },
      "age": {
        "type": "integer",
        "index": false
      },
      "address": {
        "type": "text",
        "index": false
      }
    }
  }
}
```

- `name` 字段：被单独存储（`store: true`），可以直接获取。
- `age` 字段：不被索引，不能通过查询找到，但可以从 `_source` 中提取。
- `address` 字段：不被索引，不能通过查询找到，但可以从 `_source` 中提取。

查询示例：

1. **获取整个文档**：

```json
{
  "_index": "your_index",
  "_id": "1",
  "_source": {
    "name": "John Doe",
    "age": 30,
    "address": "123 Main St"
  }
}
   ```

2. **获取单独存储的字段**：
查询时指定返回存储的字段值：

```json
{
  "stored_fields": ["name"]
}
   ```

返回结果：

```json
{
  "_index": "your_index",
  "_id": "1",
  "fields": {
    "name": ["John Doe"]
  }
}
   ```

通过理解 `_source` 提取和 `store` 属性，可以优化索引和查询性能，满足不同的需求。

---