---

sidebar_position: 9
description: 团队是用于引用 Port 中存在的团队的输入信息。

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 团队

团队是一个输入项，被用来引用存在于 Port 中的团队。

## 💡 团队常用 Usage

例如，团队输入类型可被用来引用任何存在于 Port 中的团队: 

* 服务拥有团队；
* 当前待命人员
* 主要维护者；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助中心页面，我们可以看到**脚手架新服务**操作，其 "Owning Team "输入是用户输入。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"},
{label: "Array", value: "array"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myTeamInput": {
    "title": "My team input",
    "icon": "My icon",
    "description": "My team input",
    // highlight-start
    "type": "string",
    "format": "team",
    // highlight-end
    "default": "my-team"
  }
}
```

</TabItem>
<TabItem value="array">

```json showLineNumbers
{
  "myTeamArrayInput": {
    "title": "My team array input",
    "icon": "My icon",
    "description": "My team array input",
    // highlight-start
    "type": "array",
    "items": {
      "type": "string",
      "format": "team"
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
      myTeamInput = {
        title       = "My team input"
        description = "My team input"
        format      = "team"
        default     = "my-team"
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
      myTeamArrayInput = {
        title       = "My team array input"
        description = "My team array input"
        format      = "team"
      }
    }
  }
  # highlight-end
}
```

</TabItem>

</Tabs>
```