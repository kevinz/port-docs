---
sidebar_position: 16
description: 团队是一种数据类型，用于引用存在于 Port

---

import ApiRef from "../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 团队

团队是一种数据类型，用于引用存在于 Port 中的团队。

## 💡 团队常用 Usage

例如，团队属性类型可被用来引用任何存在于 Port 中的团队: 

* 服务拥有团队；
* 当前待命人员
* 主要维护者；
* 等等。

在[live demo](https://demo.getport.io/service_catalog) 这个例子中，我们可以看到 `Team` 团队属性。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myTeamProp": {
    "title": "My team",
    "icon": "My icon",
    "description": "My team property",
    // highlight-start
    "type": "string",
    "format": "team",
    // highlight-end
    "default": "my-team"
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myTeamArray": {
    "title": "My team array",
    "icon": "My icon",
    "description": "My team array",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "team"
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
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties = {
    string_props = {
      myTeamProp = {
        title    = "My team"
        required = false
        format   = "team"
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
    array_props = {
      myTeamArray = {
        title    = "My team array"
        required = false
        type     = "array"
        string_items = {
          format = "user"
        }
      }
    }
    # highlight-end
  }
}
```

</TabItem>

</Tabs>

## Pulumi 的定义

<Tabs groupId="pulumi-definition-team-basic" queryString defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertiesArgs,BlueprintPropertiesStringPropsArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    # highlight-start
    properties=BlueprintPropertiesArgs(
        string_props={
            "myTeamProp": BlueprintPropertiesStringPropsArgs(
                title="My team",
                required=False,
                format="team",
            )
        }
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
    stringProps: {
      myTeamProp: {
        title: "My team",
        required: false,
        format: "team",
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
    stringProps: {
      myTeamProp: {
        title: "My team",
        required: false,
        format: "team",
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
    			StringProps: port.BlueprintPropertiesStringPropsMap{
    				"myTeamProp": &port.BlueprintPropertyArgs{
                        Title:      pulumi.String("My team"),
                        Required:   pulumi.Bool(false),
                        Format:     pulumi.String("team"),
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