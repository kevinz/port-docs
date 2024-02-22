---
sidebar_position: 2
---

# 示例

### 所有权记分卡

下面的示例展示了所有权记分卡。

它定义了一个过滤器: 

* 只评估与生产相关的实体(通过检查 `is_production` 属性是否设置为 `true`)。

它有两条规则: 

* 检查是否存在已定义的待命人员，以及 "open_incidents "的数量是否少于 5；
* 检查是否存在团队。

```json showLineNumbers
[
  {
    "title": "Ownership",
    "identifier": "ownership",
    "filter": {
      "combinator": "and",
      "conditions": [
        {
          "property": "is_production",
          "operator": "=",
          "value": true
        }
      ]
    },
    "rules": [
      {
        "title": "Has on call?",
        "identifier": "has_on_call",
        "level": "Gold",
        "query": {
          "combinator": "and",
          "conditions": [
            {
              "operator": "isNotEmpty",
              "property": "on_call"
            },
            {
              "operator": "<",
              "property": "open_incidents",
              "value": 5
            }
          ]
        }
      },
      {
        "title": "Has a team?",
        "identifier": "has_team",
        "level": "Silver",
        "query": {
          "combinator": "and",
          "conditions": [
            {
              "operator": "isNotEmpty",
              "property": "$team"
            }
          ]
        }
      }
    ]
  }
]
```

#### 确保关系存在

假设我们有一个与另一个名为 "域 "的蓝图有关系的 "服务 "蓝图。 我们可以定义一个记分卡，检查我们的所有服务是否都有相关的域。 如果 "域 "关系为空，则检查失败: 

```json showLineNumbers
{
  "title": "Domain definition",
  "identifier": "domain_definition",
  "rules": [
    {
      "identifier": "hasDomain",
      "title": "Has domain",
      "level": "Bronze",
      "query": {
        "combinator": "and",
        "conditions": [
          {
            "operator": "isNotEmpty",
            "relation": "domain"
          }
        ]
      }
    }
  ]
}
```