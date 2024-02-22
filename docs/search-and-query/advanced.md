---
sidebar_position: 3
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 高级

### 搜索路径查询参数

搜索路径还支持几个影响返回输出的查询参数: 


| Parameter                       | Description                                                                                                                                                                                                                                                                                                                             | Available values | Default value |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ------------- |
| `attach_title_to_relation`      | `true`: Both the identifier and the title of the related Entity will appear under the Relation key <br></br><br></br> `false`: Only the identifier of the related Entity will appear under the Relation key                                                                                                                             | `true`/`false`   | `false`       |
| `exclude_calculated_properties` | Should [mirror properties](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/mirror-property/mirror-property.md) and [calculation properties](../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/calculation-property/calculation-property.md) be returned with the result | `true`/`false`   | `false`       |


#### `attach_title_to_relation`示例

下面是根据参数 `attach_title_to_relation` 的值输出的示例: 

<Tabs groupId="attach-title" defaultValue="true" values={[
{label: "True", value: "true"},
{label: "False", value: "false"},
]}>

<TabItem value="true">

下面是使用 `attach_title_to_relation=true`时的搜索响应: 

```json showLineNumbers
{
  "ok": true,
  "matchingBlueprints": [
    "region",
    "deployment",
    "vm",
    "microservice",
    "k8sCluster",
    "permission",
    "runningService"
  ],
  "entities": [
    {
      "identifier": "e_vb9EPyW1zOamcbT1",
      "title": "cart-deployment",
      "blueprint": "deployment",
      "team": ["Team BE"],
      "properties": {
        "version": "1.4",
        "user": "yonatan",
        "status": "failed",
        "github-action-url": "https://a.com",
        "Region": "AWS"
      },
      // highlight-start
      "relations": {
        "microservice": {
          "identifier": "e_47MwTvQj03MpVyBx",
          "title": "admin-test"
        }
      },
      // highlight-end
      "createdAt": "2022-07-27T17:11:04.344Z",
      "createdBy": "auth0|6278b02000955c006f9132d3",
      "updatedAt": "2022-07-27T17:11:04.344Z",
      "updatedBy": "auth0|6278b02000955c006f9132d3"
    }
  ]
}
```

</TabItem>

<TabItem value="false">

以下是使用 `attach_title_to_relation=false` 时的相同搜索响应: 

```json showLineNumbers
{
  "ok": true,
  "matchingBlueprints": [
    "region",
    "deployment",
    "vm",
    "microservice",
    "k8sCluster",
    "permission",
    "runningService"
  ],
  "entities": [
    {
      "identifier": "e_vb9EPyW1zOamcbT1",
      "title": "cart-deployment",
      "blueprint": "deployment",
      "team": ["Team BE"],
      "properties": {
        "version": "1.4",
        "user": "yonatan",
        "status": "failed",
        "github-action-url": "https://a.com",
        "Region": "AWS"
      },
      // highlight-start
      "relations": {
        "microservice": "e_47MwTvQj03MpVyBx"
      },
      // highlight-end
      "createdAt": "2022-07-27T17:11:04.344Z",
      "createdBy": "auth0|6278b02000955c006f9132d3",
      "updatedAt": "2022-07-27T17:11:04.344Z",
      "updatedBy": "auth0|6278b02000955c006f9132d3"
    }
  ]
}
```

</TabItem>

</Tabs>