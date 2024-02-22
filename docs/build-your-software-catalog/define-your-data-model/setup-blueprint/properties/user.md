---
sidebar_position: 19
description: 用户User 是一种数据类型，用于引用存在于 Port

---

import ApiRef from "../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 用户

User 是一种数据类型，用于引用存在于 Port 中的用户。

## 💡 用户常用 Usage

例如，用户属性类型可被用来引用 Port 中存在的任何用户: 

* Owners；
* 当前值班人员
* 主要维护者；
* 等等。

在[live demo](https://demo.getport.io/service_catalog) 示例中，我们可以看到 "On Call "用户属性。

:::note 尽管 "电子邮件 "和 "用户 "格式的输入相同，但它们的表现形式却不同: 

* `email` 格式显示原始电子邮件字符串；
* `user` 格式从 Port 的已知用户列表中显示用户名和头像。

此外，"用户 "格式还可根据用户的状态对其进行区分: 


| User Status  | Example                                                                                 |
| ------------ | --------------------------------------------------------------------------------------- |
| Active       | ![Active user](../../../../../static/img/software-catalog/blueprint/activeUser.png)     |
| Invited      | ![Invited user](../../../../../static/img/software-catalog/blueprint/invitedUser.png)   |
| Unregistered | ![External user](../../../../../static/img/software-catalog/blueprint/externalUser.png) |


:::

## API 定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myUserProp": {
    "title": "My user",
    "icon": "My icon",
    "description": "My user property",
    // highlight-start
    "type": "string",
    "format": "user",
    // highlight-end
    "default": "me@example.com"
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myUserArray": {
    "title": "My user array",
    "icon": "My icon",
    "description": "My user array",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "user"
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
      myUserProp = {
        title      = "My user"
        required   = false
        format     = "user"
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
      "myUserArray" = {
        title      = "My user array"
        required   = false
        string_items = {
          format = "user"
        }
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

## Pulumi 的定义

<Tabs groupId="pulumi-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Enum - coming soon", value: "enum"}
]}>

<TabItem value="basic">

<Tabs groupId="pulumi-definition-user-basic" queryString defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertyArgs,BlueprintPropertiesArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    # highlight-start
    properties=BlueprintPropertiesArgs(
        string_props={
            "myUserProp": BlueprintPropertyArgs(
                title="My user",
                required=False,
                format="user",
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
      myUserProp: {
        title: "My user",
        required: false,
        format: "user",
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
      myUserProp: {
        title: "My user",
        required: false,
        format: "user",
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
    			StringProps: port.BlueprintPropertiesStringPropsArgs{
                    "myUserProp": port.BlueprintPropertiesStringPropsArgs{
                        Title:    pulumi.String("My user"),
                        Required: pulumi.Bool(false),
                        Format:   pulumi.String("user"),
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

</TabItem>
</Tabs>