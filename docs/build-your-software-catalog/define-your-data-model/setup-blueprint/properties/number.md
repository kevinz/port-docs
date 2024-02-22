---

sidebar_position: 11
description: 数字Number 是一种原语数据类型，用于保存数值数据

---

import ApiRef from "../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 编号

Number 是一种原语数据类型，用于保存数值数据。

## 💡 常用数字用法

例如，数字属性类型可被引用来存储任何数值数据: 

* 关键漏洞数量；
* 内存/存储分配；
* 副本数量；
* 未决问题的数量；
* 等等。

在[live demo](https://demo.getport.io/service_catalog) 示例中，我们可以看到 "JIRA 问题 "编号属性。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Enum", value: "enum"},
{label: "Array", value: "array"},
{label: "Enum Array", value: "enumArray"},
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myNumberProp": {
    "title": "My number",
    "icon": "My icon",
    "description": "My number property",
    // highlight-start
    "type": "number",
    // highlight-end
    "default": 7
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myNumberEnum": {
    "title": "My number enum",
    "icon": "My icon",
    "description": "My number enum",
    "type": "number",
    // highlight-next-line
    "enum": [1, 2, 3, 4]
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myNumberArray": {
    "title": "My number array",
    "icon": "My icon",
    "description": "My number array",
    // highlight-start
    "type": "array",
    "items": {
      "type": "number"
    }
    // highlight-end
  }
}
```

</TabItem>
<TabItem value="enumArray">

```json showLineNumbers
{
  "myNumberArray": {
    "title": "My number enum array",
    "icon": "My icon",
    "description": "My number enum array",
    // highlight-start
    "type": "array",
    "items": {
      "type": "number",
      "enum": [1, 2, 3, 4],
      "enumColors": {
        "1": "red",
        "2": "green",
        "3": "blue"
      }
    }
    // highlight-end
  }
}
```

</TabItem>
</Tabs>

<ApiRef />

## Terraform 定义

<Tabs groupId="tf-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Enum", value: "enum"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties = {
    number_props = {
      "myNumberProp" = {
        title       = "My number"
        description = "My number property"
        default     = 7
      }
    }
  }
  # highlight-end
}
```

</TabItem>
<TabItem value="enum">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties = {
    number_props = {
      "myNumberProp" = {
        title       = "My number"
        description = "My number property"
        enum        = [1, 2, 3, 4]
      }
    }
  }
  # highlight-end
}
```

</TabItem>

<TabItem value="array">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties = {
    "myNumberArray" = {
      title        = "My number array"
      description  = "My number array"
      number_items = {}
    }
    "myNumberArrayWithDefault" = {
      title       = "My number array with default"
      description = "My number array"
      number_items = {
        default = [1, 2, 3, 4]
      }
    }
  }
}
```

</TabItem>

</Tabs>

## Pulumi 的定义

<Tabs groupId="pulumi-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Enum - coming soon", value: "enum"},
{label: "Array - coming soon", value: "array"}
]}>

<TabItem value="basic">

<Tabs groupId="pulumi-definition-number-basic" queryString defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertiesArgs,BlueprintPropertiesNumberPropsArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    # highlight-start
    properties=BlueprintPropertiesArgs(
        number_props={
            "myNumberProp": BlueprintPropertiesNumberPropsArgs(
                title="My number", required=False,
            )
        },
    ),
    # highlight-end
    relations={}
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
    numberProps: {
      myNumberProp: {
        title: "My number",
        required: false,
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
    numberProps: {
      myNumberProp: {
        title: "My number",
        required: false,
      },
    },
  },
  // highlight-end
  relations: {},
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
    			NumberProps: port.BlueprintPropertiesNumberPropsMap{
    				"myNumberProp": port.BlueprintPropertiesNumberPropsArgs{
    					Title:    pulumi.String("My number"),
    					Required: pulumi.Bool(false),
    				},
    			},
    		},
    	})
    // highlight-end
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

## 验证号码

数字验证支持以下操作符: 

* 范围

使用 "最小值 "和 "最大值 "关键字(或 "独占最小值 "和 "独占最大值 "表达独占范围)的组合指定数字范围。

如果 _x_ 是要验证的值，则以下条件必须成立: 

* _x_ ≥ `最小值
* _x_ > `专属最小值
* _x_ ≤ `最大值
* _x_ < `独占最大值

<Tabs groupId="validation-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"},
{label: "Terraform", value: "tf"},
{label: "Pulumi - comfing soon", value: "pulumi"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myNumberProp": {
    "title": "My number",
    "icon": "My icon",
    "description": "My number property",
    "type": "number",
    // highlight-start
    "minimum": 0,
    "maximum": 50
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="array">

```json showLineNumbers
{
  "myNumberArray": {
    "title": "My number array",
    "icon": "My icon",
    "description": "My number array",
    // highlight-start
    "type": "array",
    "items": {
      "type": "number",
      "exclusiveMinimum": 0,
      "exclusiveMaximum": 50
    }
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  properties = {
    "number_props" = {
      "myNumberProp" = {
        title       = "My number"
        icon        = "My icon"
        description = "My number property"
        # highlight-start
        minimum     = 0
        maximum     = 50
        # highlight-end
      }
    }
  }
}
```

</TabItem>

</Tabs>