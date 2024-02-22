import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# 高级输入配置

高级输入设置可让您为执行自助操作的用户创建更多可定制的体验。 具体方法是创建自适应输入，根据实体、用户和其他输入的相关数据进行更改。

## 常见被引用情况

* 过滤下拉输入中的可用选项。
* 在输入之间创建依赖关系，允许用户根据另一个输入的值选择一个值。
* 根据登录的用户属性(如团队、电子邮件、角色)或正在执行操作的实体(仅限第 2 天或删除操作)定义动态默认值。

:::info  Pulumi 示例的语言 除非另有说明，所有**Pulumi**配置示例均以 Python 语言提供。 有关其他语言的用法，请参阅 Pulumi Provider 文档[here](https://www.pulumi.com/registry/packages/port/api-docs/action/) 。

:::

## Usage

目前仅支持以 JSON 模式定义高级输入。

创建动作后，第二步是定义输入。 至少定义一个输入后，表单底部会出现 "高级配置 "部分。 单击 "编辑 JSON"，然后添加 JSON 格式的配置。

<img src='/img/self-service-actions/advancedInputsFormExample.png' width='60%' />

### 编写配置模式

Port 提供了一个 `jqQuery` 属性，可用于从实体、登录用户或当前操作的表单输入中提取数据。 它还可用于执行数据操作。

例如，下面的 `jqQuery` 检查另一个属性(`language`)的值，并据此确定 `SDK` 属性的可能值: 

```json showLineNumbers
{
  "properties": {
    "language": {
      "type": "string",
      "enum": ["javascript", "python"]
    },
    "SDK": {
      "type": "string",
      "enum": {
        "jqQuery": "if .form.language == \"javascript\" then [\"Node 16\", \"Node 18\"] else [\"Python 3.8\"] end"
      }
    }
  }
}
```

#### 使用 "jqQuery "对象可以访问的属性

<Tabs
groupId="jqquery-properties"
queryString
defaultValue="form"
values={[
{label: 'form', value: 'form'},
{label: 'entity', value: 'entity'},
{label: 'user', value: 'user'},
]}>

<TabItem value="form">

当前操作表单中的输入值。

Usage:

```json
{
  "jqQuery": ".form.input1"
}
```

可用的 `form` 对象(每个输入都是动作[`userInputs`](/create-self-service-experiences/setup-ui-for-action/user-inputs/) 对象中的一个键) : 

```json
{
  "input1": "...",
  "input2": "...",
  "input3": "..."
}
```

</TabItem>
<TabItem value="entity">

执行操作的 "实体 "的属性。 实体数据仅在 "第 2 天 "和 "删除 "操作中可用。

Usage:

```json
{
  "jqQuery": ".entity.properties.property1"
}
```

可用的 "实体 "对象: 

```json
{
  "identifier": "...",
  "title": "...",
  "blueprint": "...",
  "team": ["..."],
  "properties": {
    "property1": "...",
    "property2": "...",
    "property3": "..."
  },
  "relations": {
    "relation1": "...",
    "relation2": "...",
    "relationMany": ["...", "..."]
  },
  "createdAt": "...",
  "createdBy": "...",
  "updatedAt": "...",
  "updatedBy": "...",
  "scorecards": {
    "ResourceQuota": {
      "rules": [
        {
          "identifier": "...",
          "status": "...",
          "level": "..."
        },
        {
          "identifier": "...",
          "status": "...",
          "level": "..."
        }
      ],
      "level": "..."
    },
    "Ownership": {
      "rules": [
        {
          "identifier": "...",
          "status": "...",
          "level": "..."
        },
        {
          "identifier": "...",
          "status": "...",
          "level": "..."
        }
      ],
      "level": "..."
    }
  }
}
```

</TabItem>
<TabItem value="user">

执行操作的用户的属性。

Usage:

```json
{
  "jqQuery": ".user.email"
}
```

可用的 logging 用户对象: 

```json
{
  "picture": "...",
  "userId": "...",
  "email": "...",
  "name": "...",
  "mainRole": "...",
  "roles": [
    {
      "name": "..."
    }
  ],
  "teams": [
    {
      "name": "...",
      "provider": "..."
    },
    {
      "name": "...",
      "provider": "..."
    }
  ]
}
```

</TabItem>
</Tabs>

jqQuery 表达式支持的键: 


| Key      | Description                                       |
| -------- | ------------------------------------------------- |
| enum     | any enum of a property                            |
| default  | the default value of any property                 |
| required | the properties which will be required in the form |
| value    | the value inside a "dataset" rule                 |
| visible  | the condition to display any property in the form |


---

#### 其他可用房产

您可以使用这些附加属性创建更复杂的输入: 

<Tabs
defaultValue="visible"
groupId="additional-inputs"
queryString
values={[
{ label: 'visible', value: 'visible', },
{ label: 'dependsOn', value: 'dependsOn', },
{ label: 'dataset', value: 'dataset', },
]}>

<TabItem value="visible">

可见 "属性用于动态隐藏/显示表单中的输入。 可见 "值可以设置为一个布尔值("true "值总是显示，"false "值总是隐藏)，也可以设置为一个评估为布尔值的 "jqQuery"。

在本例中，"runArguments "属性配置为 "visible"，因此只有在 "language "输入框中选择匹配值时，它们才会显示在表单中: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "properties": {
    "language": {
      "type": "string",
      "enum": ["javascript", "python"]
    },
    "pythonRunArguments": {
      "type": "string",
      "visible": {
        "jqQuery": ".form.language == \"python\""
      }
    },
    "nodeRunArguments": {
      "type": "string",
      "visible": {
        "jqQuery": ".form.language == \"javascript\""
      }
    }
  }
}
```

</TabItem>
<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        language = {
          enum = ["javascript", "python"]
        }
        pythonRunArguments = {
          visible_jq_query = ".form.language == \"python\""
        }
        nodeRunArguments = {
          visible_jq_query = ".form.language == \"javascript\""
        }
      }
    }
  }
}
```

</TabItem>
<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "language": {
        "enums": ["python", "javascript"],
      },
      "pythonRunArguments": {"visible_jq_query": '.form.language == "python"'},
      "nodeRunArguments": {"visible_jq_query": '.form.language == "javascript"'},
    },
  }
)
```

</TabItem>

</Tabs>

</TabItem>

<TabItem value="dependsOn">

依赖于 "属性被用来创建输入之间的依赖关系。 如果输入 X 依赖于输入 Y，则输入 X 将被**禁用**，直到输入 Y 填满为止。 在下面的示例中，"SDK "输入依赖于 "语言 "输入: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "properties": {
    "language": {
      "type": "string",
      "enum": ["javascript", "python"]
    },
    "SDK": {
      "type": "string",
      "dependsOn": ["language"]
    }
  }
}
```

</TabItem>
<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        language = {
          enum = ["javascript", "python"]
        }
        SDK = {
          depends_on: ["language"]
        }
      }
    }
  }
}
```

</TabItem>
<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "language": {
        "enums": ["python", "javascript"],
      },
      "SDK": {
        "depends_ons": ["language"]
      },
    },
  }
)
```

</TabItem>
</Tabs>

</TabItem>
<TabItem value="dataset">

数据集 "属性用于过滤[entity](/create-self-service-experiences/setup-ui-for-action/user-inputs/entity) 输入中显示的选项。它由两个属性组成: 

* `Combinator` - 在数据集的规则之间应用的逻辑运算。[Read more](/search-and-query/#combinator).
* `Rules` -[rules](/search-and-query/#rules) 的数组，只有通过这些规则的实体才会显示在表单中。请注意，数据集中的 `value` 键可以是常量(字符串、数字等)或 "jqQuery "对象。

<Tabs defaultValue="api" Values={[ {label: 'API', value: 'api'}, {label: 'Terraform', value: 'terraform'}, {label: 'Pulumi', value: 'pulumi'}

]}>

<TabItem value="api">

```json showLineNumbers
{
  "namespace": {
    "type": "string",
    "format": "entity",
    "blueprint": "namespace",
    "dataset": {
      "combinator": "and",
      "rules": [
        {
          "property": "$team",
          "operator": "containsAny",
          "value": "value here. this can also be a 'jqQuery' object"
        }
      ]
    }
  }
}
```

</TabItem>
<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  user_properties = {
    string_props = {
      "namespace" = {
        format      = "entity"
        blueprint   = "namespace"
        dataset = {
          combinator = "and"
          rules = [
            {
              property = "$team"
              operator = "containsAny"
              value = "value here. this can also be a 'jqQuery' object"
            }
          ]
        }
      }
    }
  }
}
```

</TabItem>
<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "namespace": {
        "format": "entity",
        "blueprint": "namespace",
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "property": "$team",
              "operator": "containsAny",
              "value": "value here. this can also be a 'jqQuery' object"
            }
          ]
        }
      }
    }
  }
)
```

</TabItem>

</Tabs>

</TabItem>

</Tabs>

---

## 模式示例

### 在两个表单输入之间创建依赖关系

此示例中，"language "输入和 "SDK "输入之间存在依赖关系，"SDK "输入的可用选项是根据所选语言定义的(参见 "jqQuery "键)。

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "properties": {
    "language": {
      "type": "string",
      "enum": ["javascript", "python"]
    },
    "SDK": {
      "type": "string",
      "enum": {
        "jqQuery": "if .form.language == \"javascript\" then [\"Node 16\", \"Node 18\"] else [\"Python 3.8\"] end"
      },
      "dependsOn": ["language"]
    }
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        language = {
          enum = ["javascript", "python"]
        }
        SDK = {
          enum_jq_query = "if .form.language == \"javascript\" then [\"Node 16\", \"Node 18\"] else [\"Python 3.8\"] end"
          depends_on: ["language"]
        }
      }
    }
  }
}
```

</TabItem>
<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "language": {
        "enums": ["python", "javascript"],
      },
      "SDK": {
        "enum_jq_query": "if .form.language == \"javascript\" then [\"Node 16\", \"Node 18\"] else [\"Python 3.8\"] end"
        "depends_ons": ["language"]
      },
    },
  }
)
```

</TabItem>
</Tabs>

![Cluster And Namespace Action](../../../static/img/software-catalog/blueprint/javascriptSDK.png)

###根据执行用户的角色隐藏属性

在此示例中，"visible "会检查执行用户是否具有 "admin "角色，如果不具有该角色，则会隐藏高级选项。 默认值仍会被填写并发送到后端: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "properties": {
    "simpleOption": {
      "type": "string",
      "enum": ["option1", "option2"]
    },
    "advancedOption": {
      "type": "string",
      "default": "default advanced value",
      "visible": {
        "jqQuery": ".user.roles | any(.name == \"Admin\")"
      }
    }
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        simpleOption = {
          enum = ["option1", "option2"]
        }
        advancedOption = {
          visible_jq_query = ".user.roles | any(.name == \"Admin\")"
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  "pulumi-resource-name",
  identifier="action-identifier",
  title="Action Title",
  blueprint="myBlueprint",
  user_properties={
    "string_props": {
      "simpleOption": {
          "enums": ["option1", "option2"]
      },
      "advancedOption": {"visible_jq_query": ".user.roles | any(.name == \"Admin\")"}
    },
  },
  trigger="DAY-2",
  webhook_method={"url": "https://myserver.com"},
)
```

</TabItem>
</Tabs>

非管理员用户的运行表单显示如下:  ！[What Non-Admins See](../../../static/img/software-catalog/blueprint/hiddenPropertyExample.png)

管理员用户的表单显示如下:  ！[What Admins See](../../../static/img/software-catalog/blueprint/hiddenPropertyShownExample.png)

###根据属性过滤下拉菜单中的可用选项

此示例包含一个过滤器，只显示[related to](/search-and-query/#operators-1) 在 `Cluster` 输入中所选集群的 namespace: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "env": {
    "type": "string",
    "format": "entity",
    "blueprint": "environment",
    "dataset": {
      "combinator": "and",
      "rules": [
        {
          "property": "type",
          "operator": "!=",
          "value": "production"
        }
      ]
    }
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        env = {
          format : "entity",
          blueprint : "environment"
          dataset = {
            combinator = "and"
            rules = [
              {
                property = "type"
                operator = "!="
                value = {
                  jq_query = "'production'"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "env": {
        "format": "entity",
        "blueprint": "environment",
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "property": "type",
              "operator": "!=",
              "value": "production"
            }
          ]
        }
      }
    }
  }
)
```

</TabItem>
</Tabs>

![Only Production Envs](../../../static/img/software-catalog/blueprint/onlyNotProductionEnvs.png)

:point_up: 只有类型不是 "生产 "的环境才会出现在下拉菜单中。 :point_up: 

###根据之前的输入过滤下拉菜单中的可用选项

此示例包含一个过滤器，只显示[related to](/search-and-query/#operators-1) 在 `Cluster` 输入中所选集群的 namespace: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "Cluster": {
    "type": "string",
    "format": "entity",
    "blueprint": "Cluster",
    "title": "Cluster",
    "description": "The cluster to create the namespace in"
  },
  "namespace": {
    "type": "string",
    "format": "entity",
    "blueprint": "namespace",
    "dependsOn": ["Cluster"],
    "dataset": {
      "combinator": "and",
      "rules": [
        {
          "blueprint": "Cluster",
          "operator": "relatedTo",
          "value": {
            "jqQuery": ".form.Cluster.identifier"
          }
        }
      ]
    },
    "title": "namespace",
    "description": "The namespace to create the cluster in"
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        cluster = {
          format      = "entity",
          blueprint   = "Cluster",
          title       = "Cluster",
          description = "The cluster to create the namespace in"
        }
        namespace = {
          title : "namespace",
          description : "The namespace to create the cluster in"
          format     = "entity",
          blueprint  = "namespace",
          depends_on = ["Cluster"],
          dataset = {
            combinator = "and"
            rules = [
              {
                blueprint = "Cluster"
                operator  = "relatedTo"
                value = {
                  jq_query = ".form.Cluster.identifier"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "Cluster": {
        "format": "entity",
        "blueprint": "Cluster",
        "title": "Cluster",
        "description": "The cluster to create the namespace in"
      },
      "namespace": {
        "format": "entity",
        "blueprint": "namespace",
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "blueprint": "Cluster",
              "operator": "relatedTo",
              "value": {
                "jq_query": ".form.Cluster.identifier"
              }
            }
          ]
        }
      }
    }
  }
)
```

</TabItem>

</Tabs>

![Cluster And Namespace Action](../../../static/img/software-catalog/blueprint/clusterNamespaceActionSmallerExample.png)

point_up: 用户需要选择一个集群，然后`namespace'输入列表将填充与所选集群相关的 namespace 实体: 

###根据执行操作的用户属性过滤下拉菜单的可用选项

此示例包含一个过滤器，只显示属于用户团队的 namespace(注意规则对象中的值键): 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "namespace": {
    "type": "string",
    "format": "entity",
    "blueprint": "namespace",
    "dataset": {
      "combinator": "and",
      "rules": [
        {
          "property": "$team",
          "operator": "containsAny",
          "value": {
            "jqQuery": "[.user.teams[].name]"
          }
        }
      ]
    }
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        namespace = {
          format : "entity",
          blueprint : "namespace"
          dataset = {
            combinator = "and"
            rules = [
              {
                property = "$team",
                operator = "containsAny",
                value = {
                  jq_query = "[.user.teams[].name]"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "string_props": {
      "namespace": {
        "format": "entity",
        "blueprint": "namespace",
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "property": "$team",
              "operator": "containsAny",
              "value": {
                "jq_query": "[.user.teams[].name]"
              }
            }
          ]
        }
      }
    }
  }
)
```

</TabItem>
</Tabs>

![Cluster And Namespace Action](../../../static/img/software-catalog/blueprint/userPropertiesModal.png)

:point_up: 这些是唯一与登录用户的团队相关联的 namespace: 

###根据执行操作的实体属性过滤下拉菜单的可用选项

此示例包含一个过滤器，只显示与执行操作的实体具有相似标记的 namespace: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "namespace": {
    "type": "string",
    "format": "entity",
    "blueprint": "namespace",
    "dataset": {
      "combinator": "and",
      "rules": [
        {
          "property": "tags",
          "operator": "containsAny",
          "value": {
            "jqQuery": "[.entity.properties.tags[]]"
          }
        }
      ]
    }
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        namespace = {
          format     = "entity",
          blueprint  = "namespace",
          dataset = {
            combinator = "and"
            rules = [
              {
                property = "tags"
                operator = "containsAny"
                value = {
                  jq_query = "[.entity.properties.tags[]]"
                }
              }
            ]
          }
        }
      }
    }
  }
}
```

</TabItem>

<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties = {
    "string_props": {
      "namespace": {
        "format": "entity",
        "blueprint": "namespace",
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "property": "tags",
              "operator": "containsAny",
              "value": {
                "jq_query": "[.entity.properties.tags[]]"
              }
            }
          ]
        }
      }
    }
  }
)
```

</TabItem>
</Tabs>

### 使用 jqQuery 设置默认值

此示例包含一个数组输入，其默认值将等于执行操作的实体的标记: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "some_input": {
    "type": "array",
    "default": {
      "jqQuery": ".entity.properties.tags"
    }
  }
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      array_props = {
        some_input = {
          default_jq_query = ".entity.properties.tags"
        }
      }
    }
  }
}
```

</TabItem>
<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  # ...action properties
  user_properties={
    "array_props": {
      "some_input": {
        "default_jq_query": ".entity.properties.tags"
      }
    },
  },
  trigger="DAY-2", # CREATE, DAY-2, DELETE
)
```

</TabItem>
</Tabs>

![entity tags action](../../../static/img/software-catalog/blueprint/defaultEntityTags.png)

:point_up: 命名空间标记已插入表单中。 :point_up: 

### 使用 jqQuery 设置所需输入内容

此示例包含两个用户输入: 一个始终为必填项，另一个根据实体属性为必填项。

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "properties": {
    "alwaysRequiredInput": {
      "type": "string"
    },
    "inputRequiredBasedOnData": {
      "type": "string"
    }
  },
  "required": {
    "jqQuery": "if .entity.properties.conditionBooleanProperty then [\"alwaysRequiredInput\", \"inputRequiredBasedOnData\"] else [\"alwaysRequiredInput\"] end"
  }
}
```

</TabItem>
<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" myAction {
  # ...action configuration
  {
    user_properties = {
      string_props = {
        alwaysRequiredInput = {}
        inputRequiredBasedOnData = {}
      }
    }
    required_jq_query = "if .entity.properties.conditionBooleanProperty then [\"alwaysRequiredInput\", \"inputRequiredBasedOnData\"] else [\"alwaysRequiredInput\"] end"
  }
}
```

</TabItem>
<TabItem value="pulumi">

```python showLineNumbers title="pulumi.py"
action = Action(
  "budding-action",
  identifier="budding-action",
  title="A Budding Act",
  # ...more action properties
  user_properties={
    "string_props": {
      "alwaysRequiredInput": {},
      "inputRequiredBasedOnData": {}
    },
  },
  required_jq_query='if .entity.properties.conditionBooleanProperty then ["alwaysRequiredInput", "inputRequiredBasedOnData"] else ["alwaysRequiredInput"] end',
  trigger="DAY-2", # CREATE, DAY-2, DELETE
)

pulumi.export("name", action.title)
```

</TabItem>
</Tabs>

## 完整示例

在这个示例中，我们将创建一个操作，让用户选择一个集群和该集群中的一个命名空间。 用户还可以选择一个已在集群中运行的服务。 然后，该操作将把选定的服务部署到集群中选定的命名空间。 用户只能选择与他的团队相关联的服务。

#### Port 中的现有模型: 

![Developer PortalCreate New Blueprint](../../../static/img/software-catalog/blueprint/clusterNamespaceBlueprint.png)

#### 动作配置: 

<Tabs
defaultValue="api"
values={[
{label: 'API', value: 'api'},
{label: 'Terraform', value: 'terraform'},
{label: 'Pulumi', value: 'pulumi'},
]}>

<TabItem value="api">

```json showLineNumbers
{
  "identifier": "createRunningService",
  "title": "Deploy running service to a cluster",
  "icon": "Cluster",
  "userInputs": {
    "properties": {
      "Cluster": {
        "type": "string",
        "format": "entity",
        "blueprint": "Cluster",
        "title": "Cluster",
        "description": "The cluster to create the namespace in"
      },
      "namespace": {
        "type": "string",
        "format": "entity",
        "blueprint": "namespace",
        "dependsOn": ["Cluster"],
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "blueprint": "Cluster",
              "operator": "relatedTo",
              "value": {
                "jqQuery": ".form.Cluster.identifier"
              }
            }
          ]
        },
        "title": "namespace",
        "description": "The namespace to create the cluster in"
      },
      "service": {
        "type": "string",
        "format": "entity",
        "blueprint": "Service",
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "blueprint": "$team",
              "operator": "containsAny",
              "value": {
                "jqQuery": "[.user.teams[].name]"
              }
            }
          ]
        },
        "title": "Service"
      }
    },
    "required": ["Cluster", "namespace", "service"]
  },
  "invocationMethod": {
    "type": "WEBHOOK",
    "url": "https://example.com"
  },
  "trigger": "CREATE",
  "description": "This will deploy a running service to a cluster"
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
resource "port_action" "createRunningService" {
  title       = "Create Running Service"
  blueprint   = "abc"
  identifier  = "createRunningService"
  trigger     = "CREATE"
  description = "This will deploy a running service to a cluster"
  webhook_method = {
    url = "https://example.com"
  }
  user_properties = {
    string_props = {
      cluster = {
        format      = "entity",
        blueprint   = "Cluster",
        title       = "Cluster"
        description = "The cluster to create the namespace in"
        required    = true
      }
      namespace = {
        title       = "Namespace"
        format      = "entity",
        blueprint   = "namespace",
        description = "The namespace to create the cluster in"
        required    = true
        depends_on  = ["cluster"]
        dataset = {
          combinator = "and"
          rules = [
            {
              blueprint = "Cluster"
              operator  = "relatedTo"
              value = {
                jq_query = ".form.Cluster.identifier"
              }
            }
          ]
        }
      }
      service = {
        title     = "Service"
        blueprint = "Service",
        required  = true
        dataset = {
          combinator = "and"
          rules = [
            {
              blueprint = "$team"
              operator  = "containsAny"
              value = {
                jq_query = "[.user.teams[].name]"
              }
            }
          ]
        }
      }
    }
  }
}
```

</TabItem>
<TabItem value="pulumi">

<Tabs
defaultValue="python"
values={[
{label: 'Python', value: 'python'},
{label: 'Javascript', value: 'javascript'}
]}>

<TabItem value="python">

```python showLineNumbers
action = Action(
  "create-running-service",
  identifier="createRunningService",
  title="Deploy running service to a cluster",
  icon="Cluster",
  user_properties={
    "string_props": {
      "Cluster": {
        "format": "entity",
        "blueprint": "Cluster",
        "required": True,
        "title": "Cluster",
        "description": "The cluster to create the namespace in"
      },
      "namespace": {
        "format": "entity",
        "blueprint": "namespace",
        "required": True,
        "depends_ons": ["Cluster"],
        "dataset": {
            "combinator": "and",
            "rules": [
              {
                "blueprint": "Cluster",
                "operator": "relatedTo",
                "value": {
                  "jq_query": ".form.Cluster.identifier"
                }
              }
            ],
        },
        "title": "namespace",
        "description": "The namespace to create the cluster in"
      },
      "service": {
        "format": "entity",
        "blueprint": "Service",
        "required": True,
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "blueprint": "$team",
              "operator": "containsAny",
              "value": {
                "jq_query": "[.user.teams[].name]"
              }
            }
          ]
        },
        "title": "Service"
      }
    },
  },
  trigger="CREATE",
  description="This will deploy a running service to a cluster"
  webhook_method={"url": "https://example.com"},
)

pulumi.export("name", action.title)
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new Action("create-running-service", {
  identifier: "createRunningService",
  title: "Deploy running service to a cluster",
  icon: "Cluster",
  userProperties: {
    stringProps: {
      "Cluster": {
        "format": "entity",
        "blueprint": "Cluster",
        "required": true,
        "title": "Cluster",
        "description": "The cluster to create the namespace in"
      },
      "namespace": {
        "format": "entity",
        "blueprint": "namespace",
        "required": true,
        "dependsOns": ["Cluster"],
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "blueprint": "Cluster",
              "operator": "relatedTo",
              "value": {
                "jqQuery": ".form.Cluster.identifier"
              }
            }
          ],
        },
      },
      "service": {
        "format": "entity",
        "blueprint": "Service",
        "required": true,
        "dataset": {
          "combinator": "and",
          "rules": [
            {
              "blueprint": "$team",
              "operator": "containsAny",
              "value": {
                "jqQuery": "[.user.teams[].name]"
              }
            }
          ]
        },
        "title": "Service"
      }
    }
  },
  trigger: "CREATE",
  description: "This will deploy a running service to a cluster"
  webhookMethod: {
    "url": "https://example.com"
  },
});

exports.title = entity.title;
```

</TabItem>

</Tabs>

</TabItem>
</Tabs>

#### 开发人员门户网站中的操作: 

![Cluster And Namespace Action](../../../static/img/software-catalog/blueprint/clusterNamespaceAction.png)

point_up: 用户需要选择一个集群，然后在名称空间输入列表中填入与所选集群相关的名称空间实体。 用户只能部署与其团队相关的服务。请注意 "team "前的"$"，这表示这是一个[metadata property](/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/meta-properties) 。