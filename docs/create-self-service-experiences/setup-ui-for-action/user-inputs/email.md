---
sidebar_position: 7
description: 电子邮件是用于保存电子邮件地址的输入法

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 电子邮件

电子邮件是用于保存电子邮件地址的输入。

## 💡 常用电子邮件 Usage

电子邮件输入类型可被用来存储任何合法的电子邮件地址。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Select (Enum)", value: "enum"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myEmailInput": {
    "title": "My email input",
    "icon": "My icon",
    "description": "My email input",
    // highlight-start
    "type": "string",
    "format": "email",
    // highlight-end
    "default": "me@example.com"
  }
}
```

</TabItem>
<TabItem value="enum">

```json showLineNumbers
{
  "myEmailSelectInput": {
    "title": "My email select input",
    "icon": "My icon",
    "description": "My email select input",
    "type": "string",
    "format": "email",
    // highlight-next-line
    "enum": ["me@example.com", "example@example.com"]
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myEmailArrayInput": {
    "title": "My email array input",
    "icon": "My icon",
    "description": "My email array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "email"
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
{label: "Select (Enum)", value: "enum"},
{label: "Array - coming soon", value: "array"}
]}>

<TabItem value="basic">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myEmailInput" = {
        title       = "My email input"
        description = "My email input"
        format      = "email"
        default     = "me@example.com"
      }
    }
  }
  # highlight-end
}
```

</TabItem>

<TabItem value="enum">

```hcl showLineNumbers
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myEmailInput" = {
        title       = "My email input"
        description = "My email input"
        format      = "email"
        default     = "me@example.com"
        enum = ["me@example.com", "example@example.com"]
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
      "myEmailInput" = {
        title       = "My email input"
        description = "My email input"
        default     = "me@example.com"
        string_items = {
          format = "email"
        }
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>