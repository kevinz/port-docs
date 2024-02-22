---
sidebar_position: 4
description: 对象是 JSON 数据的基本输入

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 对象

对象是 JSON 数据的基本输入。

## 💡 常用对象 Usage

对象输入类型可被引用来存储任何基于键/值的数据，例如: 

* 配置；
* 标签
* HTTP 响应；
* 字典/哈希映射；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助服务集线器页面，我们可以看到**打开 terraform PR 以添加 S3 存储桶**操作，其 `policy` 输入是对象输入。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myObjectInput": {
    "title": "My object input",
    "icon": "My icon",
    "description": "My object input",
    // highlight-start
    "type": "object",
    // highlight-end
    "default": {
      "myKey": "myValue"
    }
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myObjectSelectInput": {
    "title": "My object select input",
    "icon": "My icon",
    "description": "My object select input",
    "type": "object",
    // highlight-next-line
    "enum": [
      {
        "myKey": 1
      },
      {
        "myKey": 2
      }
    ]
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myObjectArrayInput": {
    "title": "My object array input",
    "icon": "My icon",
    "description": "My object array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "object"
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
    object_props = {
      "myObjectInput" = {
        title       = "My object input"
        description = "My object input"
        default     = jsonencode({ "myKey" = "myValue" })
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
      "myObjectArrayInput" = {
        title       = "My object array input"
        description = "My object array input"
        object_items = {}
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>

## 验证对象

对象验证支持以下操作符: 

* `properties` - 必须出现的键及其类型；
* `additionalProperties` - 是否允许使用 `properties` 中未定义的键，以及它们的类型；
* `patternProperties` - 属性应遵循哪种 regex 模式

:::tip 对象验证遵循 JSON 模式模型，请参阅[JSON schema docs](https://json-schema.org/understanding-json-schema/reference/object.html) 了解所有可用验证

:::

<Tabs groupId="validation-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"},
{label: "Terraform - coming soon", value: "tf"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myObjectInput": {
    "title": "My object input",
    "icon": "My icon",
    "description": "My object input",
    "type": "object",
    // highlight-start
    "properties": {
      "myRequiredProp": { "type": "number" }
    },
    "patternProperties": {
      "^S_": { "type": "string" },
      "^I_": { "type": "number" }
    },
    "additionalProperties": true
    // highlight-end
  }
}
```

</TabItem>

<TabItem value="array">

```json showLineNumbers
{
  "myObjectArrayInput": {
    "title": "My object array input",
    "icon": "My icon",
    "description": "My object array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "myRequiredProp": { "type": "number" }
      },
      "patternProperties": {
        "^S_": { "type": "string" },
        "^I_": { "type": "number" }
      },
      "additionalProperties": true
    }
    // highlight-end
  }
}
```

</TabItem>
</Tabs>