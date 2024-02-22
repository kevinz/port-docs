---
title: 搜索和查询

sidebar_label: 🔍 搜索和查询

---

# 🔍 搜索和查询

import CombinatorIntro from "./_combinator_intro.md"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

Port 的应用程序接口(API)提供了轻松查询、搜索和过滤软件目录数据的工具。

## 💡 常用查询 Usage

要有效跟踪软件目录中的资产，高质量的搜索是必不可少的，而使用 Port 的搜索功能则可以做到这一点: 

* 查找所有不健康的运行服务；
* 列出存在已知漏洞的所有库；
* 获取特定集群中运行的所有服务；
* 等等。

## 搜索请求

基本搜索路由是 "https://api.getport.io/v1/entities/search"，它接收 HTTP POST 请求。

搜索请求定义了不同搜索规则之间的逻辑关系，并包含用于查找合适实体的过滤器和规则。 每个搜索请求都由一个 JSON 对象表示，如下例所示: 

```json showLineNumbers
{
  "combinator": "and",
  "rules": [
    {
      "property": "$blueprint",
      "operator": "=",
      "value": "myBlueprint"
    },
    {
      "property": "$identifier",
      "operator": "contains",
      "value": "myIdentifierPart"
    }
  ]
}
```

上述查询会从 `myBlueprint` 蓝图中搜索其 `identifier` 包含字符串 `myIdentifierPart` 的所有实体。

## 搜索请求要素


| Field        | Description                                               |
| ------------ | --------------------------------------------------------- |
| `combinator` | Defines the logical operation to apply to the query rules |
| `rules`      | An array of search rules to filter results with           |


## Combinator

<CombinatorIntro />

<Tabs groupId="combinators" defaultValue="and" values={[
{label: "And", value: "and"},
{label: "Or", value: "or"}
]}>

<TabItem value="and">

```json showLineNumbers
{
  // highlight-next-line
  "combinator": "and",
  "rules": [
    {
      "property": "$blueprint",
      "operator": "=",
      "value": "myBlueprint"
    },
    {
      "property": "$identifier",
      "operator": "contains",
      "value": "myIdentifierPart"
    }
  ]
}
```

</TabItem>

<TabItem value="or">

```json showLineNumbers
{
  // highlight-next-line
  "combinator": "or",
  "rules": [
    {
      "property": "$blueprint",
      "operator": "=",
      "value": "myBlueprint"
    },
    {
      "property": "$identifier",
      "operator": "contains",
      "value": "myIdentifierPart"
    }
  ]
}
```

</TabItem>

</Tabs>

## 规则

搜索规则是一个小型过滤单元，用于控制搜索输出。

下面是一个搜索规则示例: 

```json showLineNumbers
{
  "property": "$blueprint",
  "operator": "=",
  "value": "microservice"
}
```

Port 有 2 种搜索规则操作符: 

1. 比较运算符(`=` `>`等)；
2. 关系运算符(`relatedTo` 等)。

#### 比较和运算符

#### 结构


| Field      | Description                                                                                                                                                                                                                                                                                                                                                    |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `operator` | Search operator to use when evaluating this rule, see a list of available operators below                                                                                                                                                                                                                                                                      |
| `property` | Property to filter by according to its value. It can be a [meta-property](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/meta-properties.md) such as `$identifier`, or one of the [standard properties](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/properties.md#available-properties) |
| `value`    | The value to filter by                                                                                                                                                                                                                                                                                                                                         |


#### 操作员

<Tabs className="operators-tabs" groupId="comparison" defaultValue="=" values={[
{label: "=", value: "="},
{label: "!=", value: "!="},
{label: ">", value: ">"},
{label: ">=", value: ">="},
{label: "<", value: "<"},
{label: "<=", value: "<="},
{label: "isEmpty", value: "isEmpty"},
{label: "isNotEmpty", value: "isNotEmpty"},
{label: "Property schema", value: "property-schema"},
{label: "Between", value: "between"},
{label: "notBetween", value: "notBetween"},
{label: "Contains", value: "contains"},
{label: "ContainsAny", value: "containsAny"},
{label: "In", value: "in"}
]}>

<TabItem value="=">

操作符 `=` 检查指定值是否完全匹配: 

```json showLineNumbers
{
  "operator": "=",
  "property": "myProperty",
  "value": "myExactValue"
}
```

</TabItem>

<TabItem value="!=">

操作符 `!=` 检查指定值是否完全匹配，并返回所有不满足检查的结果: 

```json showLineNumbers
{
  "operator": "!=",
  "property": "myProperty",
  "value": "myExactValue"
}
```

</TabItem>

<TabItem value=">">

操作符 `>` 检查大于指定值的值: 

```json showLineNumbers
{
  "operator": ">",
  "property": "myNumericProperty",
  "value": 7
}
```

</TabItem>

<TabItem value=">=">

操作符 `>=` 检查大于或等于指定值的值: 

```json showLineNumbers
{
  "operator": ">=",
  "property": "myNumericProperty",
  "value": 7
}
```

</TabItem>

<TabItem value="<">

操作符 `<` 检查小于指定值的值: 

```json showLineNumbers
{
  "operator": "<",
  "property": "myNumericProperty",
  "value": 7
}
```

</TabItem>

<TabItem value="<=">

操作符 `<=` 检查小于或等于指定值的值: 

```json showLineNumbers
{
  "operator": "<=",
  "property": "myNumericProperty",
  "value": 7
}
```

</TabItem>

<TabItem value="isEmpty">

isEmpty "操作符会检查指定属性的值是否为 "空": 

```json showLineNumbers
{
  "operator": "isEmpty",
  "property": "myProperty"
}
```

</TabItem>

<TabItem value="isNotEmpty">

isNotEmpty` 操作符会检查指定属性的值是否为 `null`: 

```json showLineNumbers
{
  "operator": "isNotEmpty",
  "property": "myProperty"
}
```

</TabItem>

<TabItem value="property-schema">

`propertySchema` 过滤器可与任何标准操作符一起使用。 它允许您根据与特定类型相匹配的属性过滤实体(例如，查找具有给定值的所有字符串属性): 

<Tabs values={[
{label: "String", value: "string"},
{label: "URL", value: "url"}
]}>

<TabItem value="string">

```json showLineNumbers
{
  // highlight-start
  "propertySchema": {
    "type": "string"
  },
  // highlight-end
  "operator": "=",
  "value": "My value"
}
```

</TabItem>

<TabItem value="url">

```json showLineNumbers
{
  // highlight-start
  "propertySchema": {
    "type": "string",
    "format": "url"
  },
  // highlight-end
  "operator": "=",
  "value": "https://example.com"
}
```

</TabItem>

</Tabs>

:::tip 

* `propertySchema` 可被引用到任何 Port[property](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/properties.md#supported-properties) ；
* 在执行属性模式搜索时，`propertySchema` 可取代`property` 过滤器。

:::

</TabItem>

<TabItem value="between">

between "操作符会检查日期时间值，并返回相关日期时间属性符合给定范围的实体: 

```json showLineNumbers
{
  "operator": "between",
  "property": "$createdAt",
  "value": {
    "preset": "lastWeek"
  }
}
```

**可用预设: **

* 明天
* 今天
* 昨天
* 上周
* 最近两周
* 上个月
* 最近 3 个月
* 最近 6 个月
* 最近 12 个月

between "操作符还支持标准日期范围: 

```json showLineNumbers
{
  "combinator": "and",
  "rules": [
    {
      "operator": "between",
      "property": "$createdAt",
      "value": {
        "from": "2022-07-26T16:38:06.839Z",
        "to": "2022-07-29T17:00:28.006Z"
      }
    }
  ]
}
```

</TabItem>

<TabItem value="notBetween">

notBetween" 操作符会检查日期时间值，并返回相关日期时间属性与给定范围不匹配的实体: 

```json showLineNumbers
{
  "operator": "notBetween",
  "property": "$createdAt",
  "value": {
    "preset": "lastWeek"
  }
}
```

</TabItem>

<TabItem value="contains">

包含 "操作符检查指定的子串是否存在于指定的属性中: 

```json showLineNumbers
{
  "operator": "contains",
  "property": "myStringProperty",
  "value": "mySubString"
}
```

</TabItem>

<TabItem value="containsAny">

containsAny "操作符检查目标数组中是否存在***指定的字符串: 

```json showLineNumbers
{
  "operator": "containsAny",
  "property": "myArrayProperty",
  "value": ["Value1", "Value2", ...]
}
```

</TabItem>

<TabItem value="in">

in` 操作符检查一个 `string` 属性是否等于一个或多个指定的 `string` 值: 

<Tabs values={[
{label: "Standard", value: "array"},
{label: "Dynamic Filter", value: "myTeamsDynamicFilter"}
]}>

<TabItem value="array">

```json showLineNumbers
{
  "property": "myStringProperty",
  "operator": "in",
  "value": ["Value1", "Value2"]
}
```

</TabItem>

<TabItem value="myTeamsDynamicFilter">

```json showLineNumbers
{
  "property": "$team",
  "operator": "in",
  "value": ["myTeamsDynamicFilter"]
}
```

:::note 

* 为了过滤**属于你的团队**的实体，你可以使用特殊的 "myTeamsDynamicFilter "过滤器。

:::

**用户界面: **

* 选择类型为 `string` 格式的字段 `team` 或元数据 `Team` 字段；
* 选择 `has any of` 操作符: 

![My Teams Filter](../../static/img/software-catalog/pages/MyTeamsFilter.png)

</TabItem>

</Tabs>

</TabItem>

</Tabs>

###关系结构和操作符

#### 结构


| Field       | Description                                                                               |
| ----------- | ----------------------------------------------------------------------------------------- |
| `operator`  | Search operator to use when evaluating this rule, see a list of available operators below |
| `blueprint` | Blueprint of the entity identifier specified in the `value` field                         |
| `value`     | Value to filter by                                                                        |


#### 操作员

<Tabs groupId="relation" defaultValue="relatedTo" values={[
{label: "Related To", value: "relatedTo"},
{label: "Required", value: "required"},
{label: "Direction", value: "direction"},
]}>

<TabItem value="relatedTo">

relatedTo` 操作符将返回与指定实体有关系的所有实体: 

```json showLineNumbers
{
  "operator": "relatedTo",
  "blueprint": "myBlueprint",
  "value": "myEntity"
}
```

</TabItem>

<TabItem value="required">

`relatedTo` 操作符还支持 `required` 属性--允许您搜索: 

* 来自所有关系的相关实体(具有必填 "true "或 "false "的关系)；
* 仅来自必填关系(带有必填 `true`的关系)的相关实体；
* 仅来自非必填关系(带有必填 `false` 的关系)的相关实体。

例如，要只搜索 _require_ `myBlueprint` 蓝图中的 `myEntity` 实体的相关实体，请使用以下搜索规则: 

```json showLineNumbers
{
  "operator": "relatedTo",
  "required": true,
  "value": "myEntity",
  "blueprint": "myBlueprint"
}
```

</TabItem>

<TabItem value="direction">

`relatedTo` 操作符还支持 `direction` 属性--它允许您在依赖关系图上按特定方向搜索依赖实体。 为了更好地理解该属性的功能，让我们来看看下面的示例: 

假设我们有两个蓝图 `deploymentConfig` 和 `microservice`，它们的关系定义如下(在 `deploymentConfig` 蓝图上声明式): 

```json showLineNumbers
"relations": {
  "microservice": {
    "description": "The service this Deployment Config belongs to",
    "many": false,
    "required": false,
    "target": "microservice",
    "title": "Microservice"
  }
}
```

此外，我们还有以下实体: 

```text showLineNumbers
Deployment Configs:
- Order-Service-Production
- Cart-Service-Production

Microservices:
- Order Service
- Cart Service

Environments:
- Production
```

以及以下关系

```text showLineNumbers
Order-Service-Production -> Order-Service
Order-Service-Production -> Production

Cart-Service-Production -> Cart-Service
Cart-Service-Production -> Production
```

通过查看图表布局，我们还可以绘制出方向图: 

![Dependency graph upstream downstream diagram](../../static/img/software-catalog/search-in-port/search-direction-diagram.png)

* 要搜索源依赖的实体，请使用 `"方向": "上游"；
* 要搜索依赖于源的实体 - 使用 `"direction": "下游"。

在上面的示例中，如果我们要获取 _Order-Service-Production_ 所依赖的 `Microservice` 和 `Environment`，搜索规则将是

```json showLineNumbers
{
  "operator": "relatedTo",
  "blueprint": "deploymentConfig",
  "value": "Order-Service-Production",
  "direction": "upstream"
}
```

结果将是

<details>
<summary>Order-Service-Production upstream related entities</summary>

```json showLineNumbers
{
  "ok": true,
  "matchingBlueprints": ["microservice", "environment"],
  "entities": [
    {
      "identifier": "Order-Service",
      "title": "Order-Service",
      "blueprint": "microservice",
      "properties": {
        "on-call": "mor@getport.io",
        "language": "Python",
        "slack-notifications": "https://slack.com/Order-Service",
        "launch-darkly": "https://launchdarkly.com/Order-Service"
      },
      "relations": {},
      "createdAt": "2022-11-17T15:54:20.432Z",
      "createdBy": "auth0|62ab380295b34240aa511cdb",
      "updatedAt": "2022-11-17T15:54:20.432Z",
      "updatedBy": "auth0|62ab380295b34240aa511cdb"
    },
    {
      "identifier": "Production",
      "title": "Production",
      "blueprint": "environment",
      "properties": {
        "awsRegion": "eu-west-1",
        "configUrl": "https://github.com/config-labs/kube/config.yml",
        "slackChannel": "https://yourslack.slack.com/archives/CHANNEL-ID",
        "onCall": "Mor P",
        "namespace": "Production"
      },
      "relations": {},
      "createdAt": "2022-09-19T08:54:23.025Z",
      "createdBy": "Cnc3SiO7T0Ld1y1u0BsBZFJn0SCiPeLS",
      "updatedAt": "2022-10-16T09:28:32.960Z",
      "updatedBy": "auth0|62ab380295b34240aa511cdb"
    }
  ]
}
```

</details>

如果我们要获取部署在 _Production_ `Environment` 中的所有 `deploymentConfigs` ，搜索规则将是

```json showLineNumbers
{
  "operator": "relatedTo",
  "blueprint": "environment",
  "value": "Production",
  "direction": "downstream"
}
```

结果将是

<details>
<summary>Production downstream related entities</summary>

```json showLineNumbers
{
  "ok": true,
  "matchingBlueprints": ["deploymentConfig"],
  "entities": [
    {
      "identifier": "Order-Service-Production",
      "title": "Order-Service-Production",
      "blueprint": "deploymentConfig",
      "properties": {
        "url": "https://github.com/port-labs/order-service",
        "config": {
          "encryption": "SHA256"
        },
        "monitor-links": [
          "https://grafana.com",
          "https://prometheus.com",
          "https://datadog.com"
        ]
      },
      "relations": {
        "microservice": "Order-Service",
        "environment": "Production"
      },
      "createdAt": "2022-11-17T15:55:55.591Z",
      "createdBy": "auth0|62ab380295b34240aa511cdb",
      "updatedAt": "2022-11-17T15:55:55.591Z",
      "updatedBy": "auth0|62ab380295b34240aa511cdb"
    },
    {
      "identifier": "Cart-Service-Production",
      "title": "Cart-Service-Production",
      "blueprint": "deploymentConfig",
      "properties": {
        "url": "https://github.com/port-labs/cart-service",
        "config": {
          "foo": "bar"
        },
        "monitor-links": [
          "https://grafana.com",
          "https://prometheus.com",
          "https://datadog.com"
        ]
      },
      "relations": {
        "microservice": "Cart-Service",
        "environment": "Production"
      },
      "createdAt": "2022-11-17T15:55:10.714Z",
      "createdBy": "auth0|62ab380295b34240aa511cdb",
      "updatedAt": "2022-11-17T15:55:20.253Z",
      "updatedBy": "auth0|62ab380295b34240aa511cdb"
    }
  ]
}
```

</details>

</TabItem>

</Tabs>

#### 动态属性

使用 Port 的用户界面时，可以通过以下函数在编写规则时使用已登录用户的属性: 

* `getUserTeams` - 用户所属团队的列表。
* `getUserEmail` - 用户的电子邮件。
* `getUserFullName` - 用户的全名。
* `blueprint` - 当前页面的蓝图标识符。

:::info  仅限用户界面 由于使用 API 时我们没有登录用户的上下文，因此这些函数仅在使用用户界面时可用。这在创建[chart/table widgets](/customize-pages-dashboards-and-plugins/dashboards/#chart-filters) 和[catalog pages](/customize-pages-dashboards-and-plugins/page/catalog-page#page-creation) 时非常有用。

:::

#### Usage 示例

```json showLineNumbers
[
  {
    "property": "$team",
    "operator": "containsAny",
    "value": ["{{getUserTeams()}}"]
  }
]
```

```json showLineNumbers
[
  {
    "property": "emails",
    "operator": "contains",
    "value": "{{getUserEmail()}}"
  }
]
```

```json showLineNumbers
[
  {
    "property": "name",
    "operator": "=",
    "value": "{{getUserFullName()}}"
  }
]
```

```json showLineNumbers
[
  {
    "property": "$blueprint",
    "operator": "=",
    "value": "{{blueprint}}"
  }
]
```

## 示例

有关搜索的实用代码片段，请参阅[examples](./examples.md) 页面。

## 高级

有关高级搜索用例和 Output，请参阅[advanced](./advanced.md) 页面。
