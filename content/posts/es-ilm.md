---
title: 'Elasticsearch ILM 自动轮转'
date: '2024-06-25'
tags: ['blog', 'Elasticsearch', 'ILM']
draft: false
summary: Elasticsearch Index Lifecycle Management
---

# Elasticsearch ILM 命令

Elasticsearch的自动轮转（Index Lifecycle Management，ILM）是一种用于管理索引生命周期的功能。通过ILM，用户可以根据索引的年龄或其他条件自动执行一些操作，如滚动（rollover）、关闭、删除等，以确保索引在生命周期的不同阶段得到有效管理。以下是如何配置和使用Elasticsearch的自动轮转功能的概述。

### 配置自动轮转（ILM）

1. **创建ILM策略**：
ILM策略定义了索引在不同阶段（热、温、冷、删除）的操作。一个常见的ILM策略可能包括以下阶段：
- **热（Hot）阶段**：用于活跃写入和查询。
- **温（Warm）阶段**：用于较少查询但仍需快速访问的数据。
- **冷（Cold）阶段**：用于极少访问的数据，通常存储在成本较低的硬件上。
- **删除（Delete）阶段**：删除已不再需要的数据。

示例策略：
```json
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "30d"
          }
        }
      },
      "warm": {
        "min_age": "30d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          },
          "shrink": {
            "number_of_shards": 1
          }
        }
      },
      "cold": {
        "min_age": "60d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
   ```

2. **将ILM策略应用于模板**：
配置索引模板，使其使用定义好的ILM策略。模板会自动应用到匹配的新索引。
```json
PUT _template/my_template
{
  "index_patterns": ["my_index-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index.lifecycle.name": "my_policy",
    "index.lifecycle.rollover_alias": "my_index_alias"
  }
}
   ```

3. **创建和管理索引**：
使用ILM策略的索引创建时，需要通过别名进行滚动。确保初始索引设置正确的别名和滚动条件。
```json
PUT my_index-000001
{
  "aliases": {
    "my_index_alias": {
      "is_write_index": true
    }
  }
}
   ```

### 监控和管理ILM

Elasticsearch提供了丰富的API和工具来监控和管理ILM策略和索引状态。

- **查看ILM策略**：
```json
GET _ilm/policy/my_policy
  ```

- **查看索引生命周期状态**：
```json
GET my_index-000001/_ilm/explain
  ```

- **手动触发操作**：
如果需要手动干预，可以使用以下API强制执行某些操作。
```json
POST my_index-000001/_rollover
  ```

### 轮转（Rollover）的工作原理

轮转操作通过检查索引是否达到指定条件（如最大大小或最大年龄）来决定是否创建新索引。例如，定义了一个最大大小为50GB的轮转条件，当索引大小达到50GB时，Elasticsearch会自动创建一个新索引，并将写入操作转向新索引。

### 优化和注意事项

- **合理规划阶段**：
根据业务需求合理规划各阶段的操作和时长，以确保性能和存储成本的平衡。

- **监控和调整**：
定期监控ILM策略的执行情况，并根据实际情况进行调整。

- **硬件和性能考虑**：
确保冷阶段和删除阶段的硬件资源和性能符合预期，避免影响业务连续性。

通过合理配置和使用ILM，Elasticsearch用户可以有效地管理索引生命周期，优化存储和查询性能，降低运维复杂度。