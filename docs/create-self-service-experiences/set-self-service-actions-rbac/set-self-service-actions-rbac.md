---
title: 设置自助操作 RBAC

sidebar_label: 设置 RBAC 操作

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 设置 RBAC 操作

Provider 提供细粒度控制，确保每个用户只能执行和调用与其相关的操作。

:::tip 本节将介绍 Port RBAC 功能中的自助操作部分，虽然这不是先决条件，但强烈建议您同时浏览 Port 的[permission controls](../../sso-rbac/rbac/rbac.md) 。

要管理谁可以查看 Port 中的哪些页面，请查看[page permissions](../../customize-pages-dashboards-and-plugins/page/page-permissions.md) 。

:::

## 💡 常见自助服务操作 RBAC Usage

自助服务操作 RBAC 允许管理员精细控制哪些用户可以执行哪些自助服务操作等: 

* 让开发人员只为其微服务或开发人员环境提供数据库；
* 指定新的集群供应请求需要 DevOps 团队手动批准；
* 等等。

## 配置action权限

创建/编辑自助操作时，可以使用以下方法之一设置权限: 

<Tabs groupId="config-method" queryString values={[
{label: "UI", value: "ui"},
{label: "Terraform", value: "terraform"},
]}>

<TabItem value="ui">

创建操作的最后一步是配置权限。 默认情况下，"允许组织内所有人访问 "切换按钮已启用。 若要限制对选定用户/团队的执行访问权限，请关闭切换按钮，然后使用下拉菜单选择用户/团队。

<img src='/img/self-service-actions/rbac/actionFormPermissions.png' width='70%' />

</TabItem>

<TabItem value="terraform">

Port 的 Terraform Provider 允许您通过 Terraform 控制权限。单击[here](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_action_permissions) 获取更多信息和示例。

</TabItem>

</Tabs>

## 配置手动审批操作并赋予审批权限

您可以为您的操作设置手动审批步骤。

在某项action可能具有危险性、破坏性、代价高昂的情况下，或者在组织政策规定在继续执行前必须进行额外审查的情况下，该功能尤为有用。

当用户点击需要审批的操作的 "执行 "按钮时，将在 Port 中创建一个新的 "运行 "对象，该 "运行 "对象的状态为 "WAITING_FOR_APPROVAL"，并在操作的 "运行 "选项卡中可见。

当新申请需要审批时，Port 会通过电子邮件向有审批权限的用户发送通知，或通过网络请求向配置的网址发送通知。

要配置手动审批步骤，请在操作中添加 `requiredApproval` 字段: 

```json showLineNumbers
[
  {
    ...
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "https://example.com"
    },
    "trigger": "CREATE",
    // highlight-next-line
    "requiredApproval": true,
    ...
  }
]
```

要配置哪些用户可以批准操作，请参阅[Managing permissions](/docs/create-self-service-experiences/set-self-service-actions-rbac/examples.md#setting-action-permissions) 。

## 配置批准通知

默认情况下，手动审批通知会通过电子邮件发送给有审批权限的用户。

还可以配置一个 Webhook URL，将批准通知发送到该 URL。

这样，您就能以自己选择的格式接收通知，既可以是纯 JSON 对象，也可以是 Slack 消息。

要向 URL 发送批准通知，请在操作配置中添加 `approvalNotification` 字段: 

```json showLineNumbers
{
    ...
    "requiredApproval": true,
    // highlight-start
    "approvalNotification": {
      "type": "webhook",
      "format": "json / slack",
      "url": "https://my-slack-webhook.com"
    },
    // highlight-end
    ...
}
```

单击[here](/docs/create-self-service-experiences/set-self-service-actions-rbac/examples.md#setting-up-a-slack-notification) 了解如何向 Slack 发送手动审批请求。

### 自助action RBAC 示例

有关 Port RBAC 的实际示例，请参阅[examples](./examples.md) 页面。
