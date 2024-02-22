---

sidebar_position: 13
description: 实体实体是一种输入，用于在触发操作时引用软件目录中的现有实体

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 实体

实体是一种输入类型，用于在触发操作时引用软件目录中的现有[entities](../../../build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#creating-entities) 。

## 💡 常见实体 Usage

例如，实体输入类型可被用来引用软件目录中的任何现有实体: 

* 云区域；
* 集群；
* 配置；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助中心页面，我们可以看到**scaffold new service** 操作，其 `Domain` 输入是实体输入。

## 实体输入结构

实体由唯一的 "entity"_format_和随附的 "blueprint "键表示，如下节所示: 

```json showLineNumbers
{
  "myEntityInput": {
    "title": "My entity input",
    "icon": "My icon",
    "description": "My entity input",
    // highlight-start
    "type": "string",
    "format": "entity",
    "blueprint": "myBlueprint"
    // highlight-end
  }
}
```

#### 结构表


| Field                        | Description                                                                               | Notes                                                       |
| ---------------------------- | ----------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `"format": "entity"`         | Used to specify that this is an entity input                                              | **Required**                                                |
| `"blueprint": "myBlueprint"` | Used to specify the identifier of the target blueprint that entities will be queried from | **Required**. Must specify an existing blueprint identifier |


## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myEntityInput": {
    "title": "My entity input",
    "icon": "My icon",
    "description": "My entity input",
    // highlight-start
    "type": "string",
    "format": "entity",
    "blueprint": "myBlueprint"
    // highlight-end
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "EntityArrayInput": {
    "title": "My entity array input",
    "icon": "My icon",
    "description": "My entity array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "entity",
      "blueprint": "myBlueprint"
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
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myEntityInput" = {
        title       = "My entity input"
        description = "My entity input"
        format      = "entity"
        blueprint   = "myBlueprint"
      }
    }
  }
  # highlight-end
}
```

</TabItem>

<TabItem value="array">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    array_props = {
      "EntityArrayInput" = {
        title       = "My entity array input"
        description = "My entity array input"
        string_items = {
          format = "entity"
        }
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>