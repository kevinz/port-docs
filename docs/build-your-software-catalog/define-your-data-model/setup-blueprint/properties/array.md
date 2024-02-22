---
sidebar_position: 2
description: 数组数组是一种数据类型，被引用用于保存数据列表

---

import ApiRef from "../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 阵列

数组是一种用于保存数据列表的数据类型。

## 💡 常用数组 Usage

例如，数组属性类型可被引用来存储任何数据列表: 

* 被引用的软件包
* 依赖关系
* 徽章

在[live demo](https://demo.getport.io/service_catalog) 这个示例中，我们可以看到 `Monitor Tooling` 数组属性。

## 应用程序接口定义

```json showLineNumbers
{
  "myArrayProp": {
    "title": "My array",
    "icon": "My icon",
    "description": "My array property",
    // highlight-start
    "type": "array",
    // highlight-end
    "default": [1, 2, 3]
  }
}
```

<ApiRef />

## Terraform 定义

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties = {
    array_props = {
      "myArrayProp" = {
        title      = "My array"
        required   = true
      }
    }
  }
  # highlight-end
}
```

:::info 要设置数组属性的类型，需要使用 `<type>_items` 属性类型。例如，要设置字符串数组，需要使用 `string_items` 属性类型。

```
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  properties = {
    array_props = {
      "myArrayProp" = {
        title      = "My array"
        required   = true
        string_items = {} # You can also set here default values
      }
    }
  }
}
```

我们目前支持以下类型的数组项: `string_items`, `number_items`, `boolean_items`, `object_items`。

:::

## Pulumi 的定义

<Tabs groupId="basic" queryString defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertiesArgs,BlueprintPropertyArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    # highlight-start
    properties=BlueprintPropertiesArgs(
        array_props={
            "myArrayProp": BlueprintPropertyArgs(
                title="My array", required=True,
            )
        }
    ),
    # highlight-end
    relations=[]
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
  // highlight-start
  properties: {
    array_props: {
      myArrayProp: {
        title: "My array",
        required: true,
      },
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
  // highlight-start
  properties: {
    array_props: {
      myArrayProp: {
        title: "My array",
        required: true,
      },
    },
  },
  // highlight-end
  relations: [],
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
      // highlight-start
    		Properties: port.BlueprintPropertiesArgs{
    			ArrayProps: port.BlueprintPropertiesArrayPropsMap{
                    "myArrayProp": port.BlueprintPropertyArgs{
                        Title:    pulumi.String("My array"),
                        Required: pulumi.Bool(true),
                    },
                },
    		},
      // highlight-end
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

## 验证数组

数组验证支持以下操作符: 

* 最小项目
* 最大项目
* 唯一项目

:::tip 数组验证遵循 JSON 模式模型，请参阅[JSON schema docs](https://json-schema.org/understanding-json-schema/reference/array.html) 了解所有可用的验证。

:::

<Tabs groupId="validation-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Terraform", value: "tf"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myArrayProp": {
    "title": "My array",
    "icon": "My icon",
    "description": "My array property",
    "type": "array",
    // highlight-start
    "minItems": 0,
    "maxItems": 5,
    "uniqueItems": false
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  properties = {
    array_props = {
      "myArrayProp" = {
        title      = "My array"
        required   = true
        # highlight-start
        min_items  = 0
        max_items  = 5
        # highlight-end
      }
    }
  }
}
```

</TabItem>
</Tabs>