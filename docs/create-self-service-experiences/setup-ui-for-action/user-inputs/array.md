---
sidebar_position: 5
description: 数组是数据列表的输入端

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 数组

数组是数据列表的输入。

## 💡 常用数组 Usage

例如，数组输入类型可被用来存储任何数据列表: 

* 配置参数；
* 有序值；
* 等等。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myArrayInput": {
    "title": "My array input",
    "icon": "My icon",
    "description": "My array input",
    // highlight-start
    "type": "array",
    // highlight-end
    "default": [1, 2, 3]
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myArraySelectInput": {
    "title": "My array select input",
    "icon": "My icon",
    "description": "My array select input",
    "type": "array",
    // highlight-next-line
    "enum": [
      [1, 2, 3],
      [1, 2]
    ]
  }
}
```

</TabItem>
</Tabs>

<ApiRef />

## Terraform 定义

<Tabs groupId="tf-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum) - coming soon", value: "enum"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties {
    identifier  = "myArrayInput"
    title       = "My array input"
    description = "My array input"
    type        = "array"
  }
  # highlight-end
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
  "myArrayInput": {
    "title": "My array input",
    "icon": "My icon",
    "description": "My array input ",
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
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    array_props = {
      "myArrayInput" = {
        title       = "My array input"
        description = "My array input"
        min_items   = 0
        max_items   = 5
        unique_items = false
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>