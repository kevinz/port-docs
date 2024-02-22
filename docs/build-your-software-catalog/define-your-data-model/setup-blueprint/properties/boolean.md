---
sidebar_position: 3
description: 布尔值布尔是一种原语数据类型，它有两种可能的值--true 和 false 其中之一

---

import ApiRef from "../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 布尔型

布尔是一种原语数据类型，有两个可能的值--"true "和 "false"。

## 💡 常用布尔用法

例如，布尔属性类型可被引用来存储任何真/假门: 

* 环境是否为部署锁定
* 环境是否应每晚关闭
* 服务是否处理 PII
* 环境是否公开

## 应用程序接口定义

```json showLineNumbers
{
  "myBooleanProp": {
    "title": "My boolean",
    "icon": "My icon",
    "description": "My boolean property",
    // highlight-start
    "type": "boolean",
    // highlight-end
    "default": true
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
    boolean_props = {
      "myBooleanProp" = {
        title      = "My boolean"
        required   = true
      }
    }
  }
  # highlight-end
}
```

## Pulumi 的定义

<Tabs groupId="pulumi-definition-boolean-basic" queryString defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertiesArgs,BlueprintPropertiesBooleanPropsArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    # highlight-start
    properties=BlueprintPropertiesArgs(
        boolean_props={
            "myBooleanProp": BlueprintPropertiesBooleanPropsArgs(
                title="My boolean",
                required=True,
            )
        }
    ),
    # highlight-end
    relations={},
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
    booleanProps: {
      myBooleanProp: {
        title: "My boolean",
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
    booleanProps: {
      myBooleanProp: {
        title: "My boolean",
        required: true,
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
    			BooleanProps: port.BlueprintPropertiesBooleanPropsMap{
    				"myBooleanProp": port.BlueprintPropertiesBooleanPropsArgs{
    					Title:    pulumi.String("My boolean"),
    					Required: pulumi.Bool(false),
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