import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 动态权限

Port 允许根据操作相应蓝图的任何属性/关系设置执行和/或批准执行自助操作的动态权限。

## 潜在的被引用情况

动态权限的有用应用实例: 

* 确保团队成员要求执行的action只能由其直接经理批准。
* 对依赖于相关实体数据的输入进行验证/处理。

## 配置权限

要引用动态权限: 

* 进入[`builder`](https://app.getport.io/dev-portal/data-model) 页面。
* 选择与要设置权限的 "操作 "相关联的蓝图。
* 单击蓝图的"... "图标，然后单击 "权限"。

![blueprintEditPermissions](../../../static/img/self-service-actions/rbac/blueprintEditPermissions.png)

以 JSON 格式显示的蓝图权限配置将在新窗口中打开，您可以在此为属于蓝图的实体和操作定义权限。

:::info  注 目前，动态权限仅支持操作，对实体的支持即将推出 😎

:::

查找`"actions"`键，并在该键下找到要设置权限的操作名称。 每个操作下都有以下两个键: 

* `"执行"` - 这里定义的任何逻辑都与动作的执行有关。这里可以定义谁可以执行操作。
* `"approve"` - 这里定义的任何逻辑都与批准操作有关。如果此操作未启用 "手动审批"，则此键无关，因为不需要审批。

在这两个键下，可以分别添加一个 `policy` 键，这样就可以使用两个键来使用更复杂的逻辑: 

1. ["queries"](/search-and-query/) -[rules](/search-and-query/#rules) 的集合，您可以用它来获取/过滤您需要的数据。
2."conditions" - 字符串数组，其中每个字符串都是一个 jq 查询，可以访问 `"queries"` 数据。

<details>
<summary>Example snippet (click to expand)</summary>

```json showLineNumbers
"actions": {
  "action_name": {
    "execute": {
      "policy": {
        "queries": {
          "query_name": {
            "rules": [
                # Your rule/s logic here
              ],
              "combinator": "and"
          }
        },
        "conditions": [
          # A jq query resulting in a boolean value
        ]
      }
    },
    "approve": {
      "roles": [
        "Admin"
      ],
      "users": [],
      "teams": [],
      "policy": {
        "queries": {
          "query_name": {
            "rules": [
                # Your rule/s logic here
              ],
              "combinator": "and"
          }
        },
        "conditions": [
          # A jq query resulting in an array of strings
        ]
      }
    }
  }
}
```

</details>

### 指导方针

* 您可以为执行/批准策略定义任意数量的查询。
* 对于 "执行 "策略，条件必须返回一个 "布尔 "值(决定是否允许请求者执行操作)。
* 对于 "批准 "策略，条件必须返回一个字符串数组(可以批准执行操作的用户)。
* 在 `rules` 和 `conditions` 值中，您都可以访问以下元数据: 
    - `blueprint` - 与操作绑定的蓝图。
    - `action` - 动作对象。
    - `inputs` - 执行该操作的用户提供给操作输入的 Values。
    - `user` - 执行/希望批准该操作的用户(根据策略类型)。
    - `entity` - 对于第 2 天的操作，这将包含执行操作的实体。
    - `trigger` - 触发操作的相关信息: 
        + `at` - 执行操作的日期。
        + `user` - 执行操作的用户。
* 任何无法 evaluated 的查询都将被忽略。
* 每个查询最多可返回 1000 个实体，因此请确保尽可能精确。

## 完整示例

下面是属于简单 "服务 "蓝图的权限 JSON 示例。 在这个示例中，"scaffold_new_microservice "操作只能由要求执行该操作的用户的团队领导批准。

请注意 "scaffold_new_microservice "操作(**行 49**)。 其下的 "approve "键包含一个 "policy "键，表明这里定义了一些附加逻辑，以确定谁可以批准执行该操作。

<details>
<summary>Service permissions JSON (click to expand)</summary>

```json showLineNumbers
{
  "entities": {
    "register": {
      "roles": ["microservice-moderator", "Admin"],
      "users": ["admin@dyn-permissions-demo.com"],
      "teams": [],
      "ownedByTeam": false
    },
    "update": {
      "roles": ["microservice-moderator", "Admin"],
      "users": ["admin@dyn-permissions-demo.com"],
      "teams": [],
      "ownedByTeam": false
    },
    "unregister": {
      "roles": ["microservice-moderator", "Admin"],
      "users": ["admin@dyn-permissions-demo.com"],
      "teams": [],
      "ownedByTeam": false
    },
    "updateProperties": {
      "$identifier": {
        "roles": ["microservice-moderator", "Admin"],
        "users": ["admin@dyn-permissions-demo.com"],
        "teams": [],
        "ownedByTeam": false
      },
      "$title": {
        "roles": ["microservice-moderator", "Admin"],
        "users": ["admin@dyn-permissions-demo.com"],
        "teams": [],
        "ownedByTeam": false
      },
      "$team": {
        "roles": ["microservice-moderator", "Admin"],
        "users": ["admin@dyn-permissions-demo.com"],
        "teams": [],
        "ownedByTeam": false
      },
      "$icon": {
        "roles": ["microservice-moderator", "Admin"],
        "users": ["admin@dyn-permissions-demo.com"],
        "teams": [],
        "ownedByTeam": false
      }
    }
  },
  "actions": {
    "scaffold_new_microservice": {
      "execute": {
        "roles": ["Member", "Admin"],
        "users": [],
        "teams": [],
        "ownedByTeam": false
      },
      "approve": {
        "roles": ["Admin"],
        "users": [],
        "teams": [],
        "policy": {
          "queries": {
            "executingUser": {
              "rules": [
                {
                  "value": "user",
                  "operator": "=",
                  "property": "$blueprint"
                },
                {
                  "value": "{{.trigger.user.email}}",
                  "operator": "=",
                  "property": "$identifier"
                }
              ],
              "combinator": "and"
            },
            "approvingUsers": {
              "rules": [
                {
                  "value": "user",
                  "operator": "=",
                  "property": "$blueprint"
                },
                {
                  "value": "Approver",
                  "operator": "=",
                  "property": "role"
                }
              ],
              "combinator": "and"
            }
          },
          "conditions": [
            "(.results.executingUser.entities | first | .relations.team) as $executerTeam | [.results.approvingUsers.entities[] | select(.relations.team == $executerTeam) | .identifier]"
          ]
        }
      }
    }
  }
}
```

</details>