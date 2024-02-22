---

sidebar_position: 8
description: 用户User 是一个输入，用于引用存在于 Port 中的用户。

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 用户

User 是用于引用 Port 中存在的用户的输入。

## 💡 用户常用 Usage

例如，用户输入类型可被用来引用 Port 中存在的任何用户: 

* Owners；
* 当前值班人员
* 主要维护者；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助服务中枢页面，我们可以看到**更改待命**操作，其 "待命 "输入是用户输入。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myUserInput": {
    "title": "My user input",
    "icon": "My icon",
    "description": "My user input",
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
  "myUserArrayInput": {
    "title": "My user array input",
    "icon": "My icon",
    "description": "My user array input",
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
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    string_props = {
      "myUserInput" = {
        title       = "My user input"
        description = "My user input"
        format      = "user"
        default     = "me@example.com"
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
      "myUserArrayInput" = {
        title       = "My user array input"
        description = "My user array input"
        format      = "user"
      }
    }
  }
  # highlight-end
}
```

</TabItem>
</Tabs>