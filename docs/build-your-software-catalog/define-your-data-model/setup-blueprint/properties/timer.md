---
sidebar_position: 17
description: 定时器计时器是一种数据类型，用于定义特定实体的过期日期/生命周期

---

import ApiRef from "../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 定时器

定时器是一种数据类型，用于定义特定实体的过期日期/寿命。

## 💡 常用计时器 Usage

例如，计时器属性类型可被用来存储目录实体和属性的未来到期日期: 

* 临时开发环境；
* 下一次健康检查倒计时；
* 临时云资源；
* 为资源添加临时权限；
* 等等。

在[live demo](https://demo.getport.io/developerEnvs) 这个示例中，我们可以看到 `TTL` 定时器属性。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myTimerProp": {
    "title": "My timer",
    "icon": "My icon",
    "description": "My timer property",
    // highlight-start
    "type": "string",
    "format": "timer",
    // highlight-end
    "default": "2022-04-18T11:44:15.345Z"
  }
}
```

</TabItem>
</Tabs>

<ApiRef />

## Terraform 定义

<Tabs groupId="tf-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  properties = {
    string_props = {
      "myTimerProp" = {
        title       = "My timer"
        icon        = "My icon"
        description = "My timer property"
        format      = "timer"
        default     = "2022-04-18T11:44:15.345Z"
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
]}>

<TabItem value="basic">

<Tabs groupId="pulumi-definition-timer-basic" queryString defaultValue="python" values={[
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
            "myTimerProp": BlueprintPropertiesStringPropsArgs(
                title="My timer",
                format="timer",
                required=True,
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
      myTimerProp: {
        title: "My timer",
        format: "timer",
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
    stringProps: {
      myTimerProp: {
        title: "My timer",
        format: "timer",
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
    			StringProps: port.BlueprintPropertiesStringPropsMap{
    				"myTimerProp": &port.BlueprintPropertiesStringPropsArgs{
                        Title:    pulumi.String("My timer"),
                        Format:   pulumi.String("timer"),
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

</TabItem>
</Tabs>

## 示例

下面是一个 `timerExample` 蓝图的实体，它有一个标识符为 `timer` 的定时器属性。

在示例实体中，指定了过期日期: 

```json showLineNumbers
"identifier": "entityIdentifier",
  "title": "Timer Example",
  "icon": "Microservice",
  "blueprint": "timerExample",
  "properties": {
    // highlight-next-line
    "timer": "2022-12-01T16:50:00+02:00"
  },
  "relations": {}
```

查看 Port 的用户界面，可以看到我们创建的计时器将在 2 小时后过期: 

![Timer entity](../../../../../static/img/software-catalog/entity/TTLCreateEntity.png)

2 小时后，属性状态将变为 "已过期"，"定时器已过期 "事件将被发送到更改日志: 

![Timer entity expired](../../../../../static/img/software-catalog/entity/TTLExpiredEntity.png)

计时器过期事件也会出现在 Port 的审核日志中: 

![Timer Audit log](../../../../../static/img/software-catalog/entity/AuditLogTTL.png)

<!-- TODO: add a link to the docs about changelog destination and event listener -->

为了通知定时器到期，以下通知正文将被发送到在蓝图的 "changelogDestination "中配置的 Webhook/Kafka 主题: 

```json showLineNumbers
{
  "context": {
    "blueprint": "timerExample",
    "entity": "entityIdentifier"
  },
  "action": "TIMER_EXPIRED",
  "trigger": {
    "at": "2022-12-01T16:50:00+02:00",
    "by": {
      "byPort": true,
      "orgId": "org_example"
    },
    "origin": "API"
  },
  "resourceType": "entity",
  "status": "SUCCESS"
}
```