---
title: 'Elasticsearch stats 里all节点和indices的区别'
date: '2024-06-24'
tags: ['blog', 'Elasticsearch', 'stats']
draft: false
summary: Elasticsearch Dynamic Templates
---

# Elasticsearch stats 命令

### `_stats` 命令中 `_all` 和 `indices` 部分的 `primaries` 区别

在Elasticsearch中，`_stats`命令提供多个层次的统计信息，包括集群级别和索引级别的数据。`_all`和`indices`部分的`primaries`有特定的意义和区别。

#### `_all` 和 `indices`

1. **`_all`**:
- `"_all"`部分提供的是整个集群的统计信息，汇总了集群中所有索引的统计数据。
- 包括所有主分片和副本分片的数据的总和。

2. **`indices`**:
- `indices`部分提供的是每个具体索引的统计信息。
- 每个索引的数据单独列出，允许对单个索引的性能和使用情况进行详细分析。

#### `primaries` 在 `_all` 和 `indices` 中的区别

1. **`_all.primaries`**:
- 汇总了整个集群中所有主分片（primary shards）的统计数据。
- 例如，`_all.primaries.docs.count`表示整个集群中所有主分片的文档总数。
- 这些数据代表整个集群中所有主分片的合计值。

2. **`indices.[index_name].primaries`**:
- 提供某个具体索引的主分片的统计数据。
- 例如，`indices.XXX.primaries.docs.count`表示名为`XXX`的索引的所有主分片的文档总数。
- 这些数据只反映特定索引的主分片情况，不包括其他索引的数据。

#### 举例说明

假设集群中有两个索引 `index1` 和 `index2`，每个索引都有若干主分片。

- **`_all.primaries` 的统计信息是集群中所有主分片的汇总数据**:
```json
"_all": {
  "primaries": {
    "docs": {
      "count": 5000000,  // 所有索引的主分片文档总数
      "deleted": 500000
    },
    ...
  },
  ...
}
  ```

- **`indices.index1.primaries` 的统计信息仅包括 `index1` 的主分片数据**:
```json
"indices": {
  "index1": {
    "primaries": {
      "docs": {
        "count": 3000000,  // index1 索引的主分片文档总数
        "deleted": 300000
      },
      ...
    },
    ...
  },
  "index2": {
    "primaries": {
      "docs": {
        "count": 2000000,  // index2 索引的主分片文档总数
        "deleted": 200000
      },
      ...
    },
    ...
  }
}
  ```

因此，`_all.primaries` 是集群层面所有主分片的汇总数据，而 `indices.[index_name].primaries` 是具体某个索引的主分片数据。