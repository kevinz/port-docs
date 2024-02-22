---
sidebar_position: 13
description: 计算属性允许您从实体的现有属性中构建新数据

---

import ApiRef from "../../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# ➕ 计算属性

计算属性允许您直接或通过使用关系和镜像属性使用蓝图上定义的现有属性，以便通过使用[`jq`](https://github.com/stedolan/jq) 处理器为 `JSON` 创建新属性。

* 过滤/选择/切片/合并现有属性中的数据；
* 创建数学公式或修改。例如，通过指定页面大小和所需页面数计算所需磁盘存储空间。
* 合并复杂属性，包括深度合并和覆盖。

## 💡 常用计算 Usage

计算属性可以更方便地定义基于其他属性值的属性，并增加了转换数据等功能: 

* 根据模板构建自定义 URL，例如
    - `https://slack.com/ + {my_parameter}`；
    - `https://datadog.com/ + {my_parameter}`；
    - `https://launchdarkly.com/ + {my_parameter}`；
* 合并服务配置模板，创建真正的服务配置；
* 计算代码 Owners 的数量；
* 等等。

在[live demo](https://demo.getport.io/service_catalog) 这个示例中，我们可以看到 "Slack Notifications "计算属性。

## 定义

<Tabs groupId="api-definition" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"},
{label: "Pulumi", value: "pulumi"}
]}>

<TabItem value="api">

calculationProperties "键是实体 JSON 中的顶级键(类似于 "identifier"、"title"、"properties "等)。

您可以使用 `.properties` 作为计算的一部分访问属性

```json showLineNumbers
{
  "calculationProperties": {
    "myCalculationProp": {
      "title": "My calculation property",
      "type": "string",
      "calculation": ".properties.myStringProp + .properties.myStringProp"
    }
  }
}
```

<ApiRef />

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  calculation_properties {
    identifier  = "myCalculationProp"
    title       = "My calculation property"
    type        = "string"
    calculation = ".properties.myStringProp + .properties.myStringProp"
  }
  # highlight-end
}
```

</TabItem>

<TabItem value="pulumi">

<Tabs groupId="pulumi-definition-calculation" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertiesArgs,BlueprintCalculationPropertiesArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    properties=BlueprintPropertiesArgs(
    # blueprint properties
    ),
    # highlight-start
    calculation_properties={
      "myCalculation": BlueprintCalculationPropertiesArgs(
        title="My calculation property", calculation=".properties.myStringProp + .properties.myStringProp", type="string",
      )
    },
    # highlight-end
)
```

</TabItem>

<TabItem value="typescript">

```typescript showLineNumbers
import * as pulumi from "@pulumi/pulumi";
import * as port from "@port-labs/port";

export const blueprint = new port.Blueprint("myBlueprint", {
  identifier: "myBlueprint",
  title: "My Blueprint",
  properties: {
    // blueprint properties
  },
  // highlight-start
  calculationProperties: {
    myCalculation: {
      title: "My calculation property",
      calculation: ".properties.myStringProp + .properties.myStringProp",
      type: "string",
    },
  },
  // highlight-end
});
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
"use strict";
const pulumi = require("@pulumi/pulumi");
const port = require("@port-labs/port");

const entity = new port.Blueprint("myBlueprint", {
  title: "My Blueprint",
  identifier: "myBlueprint",
  properties: {
    // blueprint properties
  },
  // highlight-start
  calculationProperties: {
    myCalculation: {
      title: "My calculation property",
      calculation: ".properties.myStringProp + .properties.myStringProp",
      type: "string",
    },
  },
  // highlight-end
});

exports.title = entity.title;
```

</TabItem>
<TabItem value="go">

```go showLineNumbers
package main

import (
    "github.com/port-labs/pulumi-port/sdk/go/port"
    "github.com/pulumi/pulumi/sdk/v3/go/pulumi"
)

func main() {
    pulumi.Run(func(ctx *pulumi.Context) error {
    	blueprint, err := port.NewBlueprint(ctx, "myBlueprint", &port.BlueprintArgs{
    		Identifier: pulumi.String("myBlueprint"),
    		Title:      pulumi.String("My Blueprint"),
    		// blueprint properties..
      # highlight-start
            CalculationProperties: port.BlueprintCalculationPropertiesMap{
              "myCalculation": port.BlueprintCalculationPropertiesArgs{
                  Title:       pulumi.String("My calculation property"),
                  Calculation: pulumi.String(".properties.myStringProp + .properties.myStringProp"),
                  Type:        pulumi.String("string"),
              }
            },
      # highlight-end
    	})
    	ctx.Export("blueprint", blueprint.Title)
    	if err != nil {
    		return err
    	}
    	return nil
    })
}
```

</TabItem>

</Tabs>

</TabItem>

</Tabs>

## 支持的类型

<Tabs groupId="calculation-definition" defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Format", value: "format"},
{label: "Spec", value: "spec"}
]}>

<TabItem value="basic">

计算属性支持以下输出类型: "字符串"、"数字"、"对象"、"数组 "和 "布尔"。 例如: 

```json showLineNumbers
{
  "calculationProperties": {
    "myCalculationProp": {
      "title": "My calculation property",
      // highlight-next-line
      "type": "my_output_type",
      "calculation": ".properties.myStringProp + .properties.myStringProp"
    }
  }
}
```

</TabItem>

<TabItem value = "format">

计算属性支持以下输出格式: "yaml"、"team"、"user"、"ipv6 "和 "url"。 例如: 

```json showLineNumbers
{
  "calculationProperties": {
    "myCalculationProp": {
      "title": "My calculation property",
      // highlight-next-line
      "type": "string",
      // highlight-next-line
      "format": "user",
      "calculation": ".properties.user"
    }
  }
}
```

</TabItem>

<TabItem value = "spec">

计算属性支持以下输出规格: "markdown"、"open-api "和 "async-api"。 例如: 

```json showLineNumbers
{
  "calculationProperties": {
    "myCalculationProp": {
      "title": "My calculation property",
      // highlight-next-line
      "type": "string",
      // highlight-next-line
      "format": "url",
      // highlight-next-line
      "spec": "embedded-url"
      "calculation": ".properties.text"
    }
  }
}
```

</TabItem>

</Tabs>

## 在计算属性中被引用`元属性

可以将[meta properties](../meta-properties.md) 作为计算属性的模板值。

例如，如果要将模板 URL(如 `https://datadog.com`)与 `identifier` 元属性连接起来: 

给定以下 "通知服务 "实体: 

```json showLineNumbers
{
  "identifier": "notification-service",
   "title": "Notification Service",
  "properties": {
   ...
  },
}
```

蓝图的计算属性定义如下: 

```json showLineNumbers
{
  "calculationProperties": {
    "monitorUrl": {
      "title": "Monitor url",
      "type": "string",
      "format": "url",
      "calculation": "'https://datadog.com/' + .identifier"
    }
  }
}
```

属性 `monitorUrl` 的值将是 `https://datadog.com/notification-service`

## 在计算属性中被引用`镜像属性

可以将[mirror properties](../mirror-property/mirror-property.md) 作为计算属性的模板值。

例如，如果一个实体有一个名为 "owningSquad "的镜像属性: 

```json showLineNumbers
"mirrorProperties": {
    "owningSquad": {
        "path": "microservice-to-squad.$title"
    }
}
```

与小队松弛频道链接的计算属性可以是

```json showLineNumbers
"owning_squad_slack": {
    "title": "Owning Squad Channel",
    "calculation": "'https://slack.com/' + .properties.owningSquad",
}
```

## 着色计算属性

通过在计算属性对象中添加一个值为 `true` 的 `colorized` 键，可以根据计算属性的值为其着色。 您还可以添加一个 `colors` 键来指定不同值的颜色，否则，颜色将自动为您选择。

例如，如果要为名为 "status-calculation "的计算属性着色，使其具有 "OK"、"WARNING "和 "CRITICAL "三个 Values 值: 

```json showLineNumbers
"properties":{
    "status":{
        "type": "string"
    },
},
"calculationProperties": {
    "status-calculation": {
        "title": "Status",
        "type": "string",
        "calculation": ".properties.status",
        "colorized": true,
        "colors": {
            "OK": "green",
            "WARNING": "yellow",
            "CRITICAL": "red"
        }
    }
}
```

---

:::warning  包含特殊字符的参数 参数包含特殊字符(例如: `-`)或以数字开头(例如: `@/#/$/1/2/3`)时，应用单引号括起来。

```json showLineNumbers
"properties":{
    "prop-with-special-char":{
        "type": "string"
    },
},
"calculationProperties": {
    "myCalculatedProp": {
        "title": "My Calculated Property",
        "type": "string",
        "calculation": ".properties.'prop-with-special-char'",
    }
}
```

:::

## 示例

请参阅计算属性[examples](./examples.md) 页面。