---
sidebar_position: 3
description: 布尔值布尔值是一种基本输入法，它有两种可能的值--true 和 false 其中之一

---

import ApiRef from "../../../api-reference/_learn_more_reference.mdx"

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 布尔型

布尔值是一种基本输入法，它有两种可能的值--`true`和`false`。

## 💡 常用布尔用法

例如，布尔输入类型可被用来存储任何真/假门: 

* 部署环境是否锁定；
* 环境是否应每晚关闭；
* 服务是否处理 PII；
* 环境是否公开；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 自助中心页面，我们可以看到**删除回购**操作，其 `Confirm` 输入是布尔输入。

## 应用程序接口定义

<Tabs groupId="api-definition" queryString defaultValue="basic" values={[
{label: "Basic", value: "basic"}
]}>

<TabItem value="basic">

```json showLineNumbers
{
  "myBooleanInput": {
    "title": "My boolean input",
    "icon": "My icon",
    "description": "My boolean input",
    // highlight-start
    "type": "boolean",
    // highlight-end
    "default": true
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
resource "port_action" "myAction" {
  # ...action properties
  # highlight-start
  user_properties = {
    boolean_props = {
      myBooleanInput = {
        title       = "My boolean input"
        description = "My boolean input"
        default     = true
      }
    }
  }
  # highlight-end
}
```

</TabItem>
</Tabs>