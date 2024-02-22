---
sidebar_position: 2
title: 相关蓝图

sidebar_label: 相关蓝图

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 相关蓝图

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/McUWOC4gcu4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

关系定义了蓝图之间的连接，并将其转化为软件目录中资产的依赖性反映。

## 什么是关系？

关系使我们能够在蓝图之间建立联系，从而将基于这些蓝图的实体连接起来。 这就为软件目录提供了逻辑上下文。

## 💡 共同关系

例如，关系可被引用来表示软件目录中资产之间的逻辑联系: 

* 一个**微服务**被引用的**packages**；
* 一个**CI作业**的**运行**历史记录；
* 一个**云账户中存在的**Kubernetes集群**；
* 等等。

在[live demo](https://demo.getport.io/dev-portal) 示例中，我们可以看到 DevPortal 生成器页面，其中包含所有蓝图及其关系。

## 关系模式结构

关系对象的基本结构: 

```json showLineNumbers
{
  "myRelation": {
    "title": "My title",
    "target": "My target blueprint",
    "required": true,
    "many": false
  }
}
```

:::info 关系中的 "relations "键下存在一个关系。[Blueprint JSON schema](../setup-blueprint/setup-blueprint.md#blueprint-schema-structure)

:::

## 结构表


| Field        | Description                                                                                            | Notes                                                                                                                                                                                                                                                                              |
| ------------ | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `identifier` | Unique identifier                                                                                      | The identifier is used for API calls, programmatic access and distinguishing between different relations. <br></br> <br></br> The identifier is the key of the relation schema object, in the [schema structure](#relation-schema-structure) above, the identifier is `myRelation` |
| `title`      | Relation name that will be shown in the UI                                                             | Human-readable name for the relation                                                                                                                                                                                                                                               |
| `target`     | Target blueprint identifier                                                                            | The target blueprint has to exist when defining the relation                                                                                                                                                                                                                       |
| `required`   | Boolean flag to define whether the target must be provided when creating a new entity of the blueprint |
| `many`       | Boolean flag to define whether multiple target entities can be mapped to the Relation                  | For more information refer to [many relation](#many)                                                                                                                                                                                                                               |


## 关系类型

### :bust_in_silhouette: Single

单一类型关系被用来将单一目标实体映射到源实体。

#### 💡 共同单一关系

* 将**部署**映射到所部署的**运行服务**；
* 将**包版本**映射到**包**；
* 将**k8s 集群**映射到它所配置的**云账户**；
* 等等。

在[live demo](https://demo.getport.io/packageVersionEntity?identifier=AnalyticsTracker_1_2_9) 这个例子中，我们可以看到一个特定的软件包版本及其相关的核心软件包。 🎬

#### 单一关系结构

单一类型关系由 `many: false` 配置区分: 

<Tabs groupId="definition" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myRelation": {
    "title": "My title",
    "target": "myTargetBlueprint",
    "required": false,
    "many": false
  }
}
```

<ApiRef />

</TabItem>
<TabItem value="tf">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # ...user-defined properties
  # highlight-start
  relations {
    identifier = "myRelation"
    title      = "My relation"
    target     = "myTargetBlueprint"
    required   = false
    many       = false
  }
  # highlight-end
}
```

</TabItem>
</Tabs>

### 👥 许多

多类型关系被用来将多个目标实体映射到源实体。

#### 💡 常见的多种关系

* 映射服务之间的依赖关系；
* 映射**服务被引用的**packages**；
* 映射**服务被引用的**云资源；
* 映射在**开发环境中部署的**服务；
* 等等。

在这个[live demo](https://demo.getport.io/developerEnvEntity?identifier=test-shizuko) 示例中，我们可以看到一个特定的开发人员环境及其被引用的运行服务。 🎬

#### 多关系结构

多类型关系由 `many: true` 配置区分: 

<Tabs groupId="definition" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "myRelation": {
    "title": "My title",
    "target": "myTargetBlueprint",
    "required": false,
    "many": true
  }
}
```

<ApiRef />

</TabItem>
<TabItem value="tf">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # ...user-defined properties
  # highlight-start
  relations {
    identifier = "myRelation"
    title      = "My relation"
    target     = "myTargetBlueprint"
    required   = false
    many       = true
  }
  # highlight-end
}
```

</TabItem>
</Tabs>

:::note 关系 "的 "many "和 "required "均设为 "true "时无法配置

:::

## 在 Port 中配置关系

关系是[blueprint](../setup-blueprint/setup-blueprint.md#blueprint-structure) 结构的一部分。

<Tabs groupId="definition" queryString defaultValue="api" values={[
{label: "API", value: "api"},
{label: "UI", value: "ui"},
{label: "Terraform", value: "tf"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "identifier": "myIdentifier",
  "title": "My title",
  "description": "My description",
  "icon": "My icon",
  "calculationProperties": {},
  "schema": {
    "properties": {},
    "required": []
  },
  // highlight-start
  "relations": {
    "myRelation": {
      "title": "My title",
      "target": "My target blueprint",
      "required": true,
      "many": false
    }
  }
  // highlight-end
}
```

<ApiRef />

</TabItem>

<TabItem value="ui">

1. 请访问[DevPortal Builder page](https://app.getport.io/dev-portal) ；
2. 单击关系的 "源 "蓝图上的铅笔图标: 

![Blueprints page with Create Relation Marked](../../../../static/img/build-your-software-catalog/define-your-data-model/relate-blueprints/editBlueprintMarked.png)

</TabItem>

<TabItem value="tf">

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  # ...blueprint properties
  # ...user-defined properties
  # highlight-start
  relations = {
    "myRelation" = {
      title    = "My title"
      target   = "My target blueprint"
      required = true
      many     = false
    }
  }
  # highlight-end
}
```

</TabItem>
</Tabs>

一旦添加到蓝图定义中，您就可以[apply the blueprint](../setup-blueprint/setup-blueprint.md#apply-blueprints-to-port) 到 Port