---
sidebar_position: 10
description: Datetime 是用于引用日期和时间的输入法

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 日期时间

Datetime 是用于引用日期和时间的输入法。

## 💡 常用日期时间 Usage

例如，datetime 输入类型可被用来存储任何日期和时间: 

* 部署时间；
* 发布时间
* 创建时间戳；
* 等等。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myDatetimeInput": {
    "title": "My datetime input",
    "icon": "My icon",
    "description": "My datetime input",
    // highlight-start
    "type": "string",
    "format": "date-time",
    // highlight-end
    "default": "2022-04-18T11:44:15.345Z"
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myDatetimeArrayInput": {
    "title": "My datetime array",
    "icon": "My icon",
    "description": "My datetime array",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "date-time"
    }
    // highlight-end
  }
}
```

</TabItem>
</Tabs>

<ApiRef />

## Terraform 定义

<Tabs groupId="tf-definition" defaultValue="basic" values={[
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
      myDatetimeProp = {
        title    = "My datetime"
        format   = "date-time"
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
      myArrayDatetimeProp = {
        title    = "My array datetime"
        string_items = {
          format = "date-time"
        }
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>