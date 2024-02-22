---
title: 设置操作

---

import ApiRef from "../../api-reference/_learn_more_reference.mdx";

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 设置操作

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/DhDQ_lucdgM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

选择动作的名称、描述: 和图标，使其易于识别。

选择希望用户填写的用户输入信息，以使用该操作。

Port 支持各种输入类型，包括构建带有条件和步骤的向导，以最佳方式满足用户的体验需求。

设置行动包括以下步骤: 

1. **Define[action information](#structure-table)** - title, icon, description 和相关的[blueprint](../../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) ；
2. **选择[user inputs](#userinputs---form--wizard-ui)** - 通过指定用户需要填写的输入类型来创建类似向导的体验，同时还包括输入验证；
3. **配置[action type](#trigger--action-type)** - 创建/第 2 天/删除；
4. **将操作连接到[backend](#invocationmethod---connect-to-a-backend)** - 对于您在 Port 中定义的每个操作，您都要告诉 Port 由哪个组件负责处理该操作的调用。这就是所谓的**调用方法(**invocation method)，Port 支持针对不同用例和环境的各种调用方法；

<!-- 5. **Reflect the action's progress** by making requests to Port's API from your action backend. You can provide additional information such as status, logs and links to job runners and pipelines that the action triggers; -->

5. **配置 RBAC 和防护栏**--这一可选步骤可让你选择谁可以触发操作，操作是否需要管理员手动批准，以及谁有批准或驳回请求的权限。

## 💡 共同行动

例如，操作可被引用来执行您选择的任何逻辑: 

* 为新的微服务搭建脚手架；
* 部署新版本；
* 锁定部署；
* 添加secret；
* 启动临时开发环境；
* 扩展环境的 TTL；
* 供应云资源；
* 更新服务的 pod 数量；
* 更新自动扩展组；
* 更改客户配置；
* 训练机器学习模型；
* 预处理数据集；
* 等等。

在[live demo](https://demo.getport.io/self-serve) 示例中，我们可以看到带有示例操作的自助服务枢纽页面。

## 行动结构

每个操作都由[Json schema](https://json-schema.org/) 表示，如下节所示: 

```json showLineNumbers
{
  "identifier": "myIdentifier",
  "title": "My title",
  "description": "My description",
  "icon": "My icon",
  "userInputs": {
    "properties": {
      "myInput1": {
        "type": "my_type",
        "title": "My input"
      },
      "myInput2": {
        "type": "my_special_type",
        "title": "My special input"
      }
    },
    "required": []
  },
  "invocationMethod": {
    "type": "myInvocationType"
  },
  "trigger": "myActionTrigger",
  "requiredApproval": false
}
```

:::note  动作数组 为蓝图配置的动作以数组形式保存，在上面的 JSON 示例中可以看到单个动作的模式，要在蓝图中保存该动作定义，请记住用方括号(`[]`)将其包起来，使其成为数组的一部分: 

```json showLineNumbers
[
  {
  "identifier": "myIdentifier",
  "title": "My title",
  "description": "My description",
  "icon": "My icon",
  "userInputs": {
    "properties": {
  ... action definition
    }
  }
]
```

:::

#### 结构表

一个动作由多个属性组成: 


| Field              | Description                                                                                                                       | Notes                                                                                                                                                                                                                                  |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`               | Internal Action ID                                                                                                                | An internal ID Port assigns to each action. Immutable. The `id` field will be added to the action JSON automatically after creating the action                                                                                         |
| `identifier`       | Unique identifier                                                                                                                 | **Required**. The identifier is used for API calls, programmatic access and distinguishing between different actions                                                                                                                   |
| `title`            | Name                                                                                                                              | Human-readable name for the action                                                                                                                                                                                                     |
| `description`      | Description                                                                                                                       | The value is visible on the action card in the Self-Service Hub and also as a tooltip to users when hovering over the action in the UI                                                                                                 |
| `icon`             | Action icon                                                                                                                       | See the full icon list [here](../../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md#full-icon-list)                                                                                              |
| `trigger`          | The type of the action: `CREATE`, `DAY-2` or `DELETE`                                                                             | The action type is sent by Port to the action backend as part of the action metadata. In addition, Port automatically places actions in specific places in the developer portal UI to make them accessible and intuitive for the user. |
| `userInputs`       | An object containing `properties` ,`required` and `order` as seen in [Action user inputs](./user-inputs/user-inputs.md#structure) |
| `requiredApproval` | Whether the action requires approval or not                                                                                       |
| `invocationMethod` | The invocation method for the action                                                                                              | Defines the destination where invocations of the action will be delivered, see [invocation method](#invocation-method) for details                                                                                                     |


:::tip  可用的用户输入法[user inputs](./user-inputs/user-inputs.md) 页面列出了所有可用的用户输入法。

:::

### `userInputs` - 表单和向导用户界面

Port 操作支持各种用户输入，可轻松创建用户界面表单和向导。

通过使用用户输入，您可以向用户明确说明您的后端需要哪些信息来处理操作请求。 此外，用户输入还提供开箱即用的验证支持，让您可以轻松设置防护栏，确保输入值标准化。

要了解有关用户输入的更多信息，请参阅[user inputs](./user-inputs/user-inputs.md) 页面。

### `trigger` - 动作类型

Port 操作支持 3 种操作类型: 

* `CREATE` - 创建操作用于创建新的新资产和资源，并且不与软件目录中的现有实体绑定；
* `DAY-2` - 日-2 操作操作用于修改、更改、增强或添加到软件目录中的现有资源和实体；
* `DELETE` - 删除操作用于触发有组织地删除软件目录中的现有资源及其相应实体。

不同的操作类型在软件目录用户界面中也有特定的表现形式: 

* 在[Self-Service Hub](https://app.getport.io/self-serve) 页面中，每种操作类型都有单独的部分；
* 在蓝图表中，`创建`操作显示为全局按钮，不与任何特定实体绑定；
* 在蓝图表和[entity page](../../customize-pages-dashboards-and-plugins/page/entity-page.md#entity-page) 中，"DAY-2 "和 "DELETE "操作专门显示在您希望触发操作的实体上。

### `invocationMethod` - 连接到后端

Port操作支持各种目标后端，在调用操作时可以触发这些后端。

不同的后端通过 `invocationMethod` 键来表示和配置，可用的方法有

* **Webhook** - 设置 webhook URL 以接收和处理表单和向导提交；
* **Kafka** - 订阅 Kafka 主题，以监听表单和向导提交信息；
* **CI Native (Github Workflow)** - 设置 Port 的[GitHub app](../../build-your-software-catalog/sync-data-to-catalog/git/github/github.md) ，以处理通过 Github 工作流提交的表单和向导；
* **CI Native(Azure Pipelines)**- 设置 Webhook 类型的服务连接，以触发 Azure 管道并通过 Github 工作流处理表单和向导提交；
* **Port 代理**-设置 Port 代理，接收表单和向导提交，并将其转发到内部网络上的后端。

要进一步了解不同的可用调用方法和后端，请参阅[setup backend](../setup-backend/setup-backend.md) 页面。

### `requireApproval` - 要求人工批准(可选)

Port 操作支持手动审批流程。 手动审批可以控制谁可以审批操作调用请求，还可以在操作请求等待相关角色审核时通知他们。

请参阅[self-service actions RBAC](../set-self-service-actions-rbac/set-self-service-actions-rbac.md) 页面了解更多信息。

## 在 Port 中配置操作

<Tabs groupId="configure" queryString>

<TabItem value="api" label="API">

```json showLineNumbers
{
  "identifier": "myIdentifier",
  "title": "My title",
  "description": "My description",
  "icon": "My icon",
  "userInputs": {
    "properties": {},
    "required": []
  },
  "invocationMethod": {
    "type": "myInvocationType"
  },
  "trigger": "myActionTrigger",
  "requiredApproval": false
}
```

:::note 上面显示的 JSON 是针对单个蓝图操作的，蓝图的操作存储在一个数组(`[]`)中

:::

<ApiRef />

</TabItem>

<TabItem value="terraform" label="Terraform">

```hcl showLineNumbers
resource "port_action" "myAction" {
  blueprint = "myBlueprint"
  identifier           = "myAction"
  description          = "My self-service action"
  user_properties = {}
  trigger = "myActionTrigger"
  myInvocationType_method = {}
}
```

</TabItem>

<TabItem value="ui" label="UI">

1. 请访问[DevPortal Builder page](https://app.getport.io/dev-portal) ；
2. 展开要添加操作的蓝图；
3. 从 3 点菜单中选择动作按钮；
4. 输入所需操作的 JSON 规范。

</TabItem>

</Tabs>