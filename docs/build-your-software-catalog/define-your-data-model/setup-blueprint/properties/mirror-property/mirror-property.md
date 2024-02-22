---
sidebar_position: 15
description: 镜像属性镜像属性允许您将相关实体的数据映射到您的实体中

---

import ApiRef from "../../../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 🪞 镜子属性

镜像属性允许您将相关实体的数据映射到您的实体中。 镜像属性可被用于到具有[relations defined](../../../relate-blueprints/relate-blueprints.md) 的蓝图中。

当两个蓝图通过关系连接时，"源 "蓝图中的实体就可以使用一组新的属性。

这些新属性被称为 `mirrorProperties` 。

镜像属性将作为一个名为 "mirrorProperties "的附加键出现在 "源 "蓝图上。 它代表从 "目标 "蓝图(或从连接图下方的其他实体)中查询到的附加属性。

镜像属性可让您将相关实体的属性值映射到 "源 "蓝图中的 "键"，从而在查看实体时为您提供更多上下文和数据，同时不会在 Output 中加入不必要的字段。

镜像属性同时支持[user-defined](../properties.md#available-properties) 属性和[meta-properties](../meta-properties.md) ，使用类似的语法。

## 💡 常用镜像 Usage

镜像属性可以通过映射目录中其他相关实体的附加数据和属性，丰富实体的可见数据: 

* 显示运行服务的图表版本；
* 显示运行中服务的环境类型；
* 显示 k8s 集群的云 Provider；
* 等等。

在[live demo](https://demo.getport.io/k8s-clusters) 这个示例中，我们可以看到 `Cloud Provider` 属性，它是相关的 `Cloud Account` 蓝图的镜像属性 🎬

## 应用程序接口定义

`mirrorProperties` 键是实体 JSON 中的顶级键(类似于 `identifier`, `title`, `properties` 等)。

<Tabs groupId="api-definition" defaultValue="basic" values={[
{label: "Basic", value: "basic"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "mirrorProperties": {
    "myMirrorProp": {
      "title": "My mirror property",
      "path": "myRelation.myProperty"
    }
  }
}
```

</TabItem>
</Tabs>

<ApiRef />

:::info 路径 "键接收链式关系的路径，这些链式关系导致蓝图属性或[meta-property](#meta-property-as-a-mirror-property)

:::

## Terraform 定义

<Tabs groupId="tf-definition" defaultValue="basic" values={[
{label: "Basic", value: "basic"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # highlight-start
  mirror_properties = {
    myMirrorProp = {
      title = "My mirror property"
      path  = "myRelation.myProperty"
    }
  }
  # highlight-end
}
```

</TabItem>
</Tabs>

## Pulumi 的定义

<Tabs groupId="pulumi-definition-mirror-basic" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "TypeScript", value: "typescript"},
{label: "JavaScript", value: "javascript"},
{label: "GO", value: "go"}
]}>

<TabItem value="python">

```python showLineNumbers
"""A Python Pulumi program"""

import pulumi
from port_pulumi import Blueprint,BlueprintPropertiesArgs,BlueprintMirrorPropertiesArgs

blueprint = Blueprint(
    "myBlueprint",
    identifier="myBlueprint",
    title="My Blueprint",
    properties=BlueprintPropertiesArgs(
        # blueprint properties
    ),
    # highlight-start
    mirror_properties={
        "myMirrorProp": BlueprintMirrorPropertiesArgs(
            title="My mirror property", path="myRelation.myStringProp"
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
  mirrorProperties: {
    myMirrorProp: {
      title: "My mirror property",
      path: "myRelation.myProperty",
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
  mirrorProperties: {
    myMirrorProp: {
      title: "My mirror property",
      path: "myRelation.myProperty",
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
    		MirrorProperties: port.BlueprintMirrorPropertiesMap{
              "myMirrorProp": port.BlueprintMirrorPropertiesArgs{
                    Title: pulumi.String("My mirror property"),
                    Path:  pulumi.String("myRelation.myProperty"),
    		  },
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

##`Meta-property` 作为镜像属性

这是一个镜像属性，由 Port 的[meta-properties](../meta-properties.md) 在`目标`蓝图上创建。

在下面的示例中，我们创建了一个名为 `microserviceName` 的镜像属性，它被映射到 `target` 蓝图中的 `title` 元属性(在这个示例中，关系的名称是 `deployment-to-microservice`)。 请注意 `title` 字段是如何被 `$title` 引用的，因为它是一个元属性: 

```json showLineNumbers
"microserviceName": {
    "title": "Microservice Name",
    "path": "deployment-to-microservice.$title"
}
```

## 作为镜像属性的嵌套关系

可以使用镜像属性来映射非 "源 "蓝图直系子蓝图的属性。

例如，假设我们有以下关系链: "Microservice -> System -> Domain"。

我们希望将拥有微服务的域成员直接映射到 "Microservice "实体。

域成员列于[array property](../array.md) 的用户自定义属性 `domain_members` 下。

这些关系的名称是

* `Microservice -> System`: `system`.
* 系统 -> 域":  `domain

让我们使用名为 `owningDomainMembers` 的镜像属性来映射小队成员: 

```json showLineNumbers
"owningDomainMembers": {
    "title": "Owning Domain Members",
    "path": "system.domain.domain_members"
}
```