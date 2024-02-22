---

sidebar_position: 1

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 示例

在本节中，我们将举例说明在企业中使用自助操作权限的方法，以及如何应用这些方法。

## 用例 💡

在使用自助操作权限管理时，除其他外，还可进行以下配置: 

1. 自助服务操作只能为特定用户启用；
2. 只允许特定用户/角色执行特定的自助服务操作；
3. 指定允许批准自助服务操作请求的用户；

## 设置蓝图权限

要为自助服务操作设置权限，请单击 DevPortal 生成器页面中带有所需自助服务操作的蓝图的权限按钮。 这将打开一个包含权限 JSON 的模式窗，允许您控制可在蓝图或其实体上执行的所有操作。

## 设置行动权限

### 角色示例

<Tabs groupId="action-permissions" defaultValue="action-only-admin-moderator" values={[
{label: "Only let admins/moderators run action", value: "action-only-admin-moderator"}
]}>

<TabItem value="action-only-admin-moderator">

默认情况下，**Member** 用户可以执行蓝图上定义的所有操作。 在本例中，我们只允许**Moderators**(和**Admins**)执行 "clone_env "操作(同时禁用 Member 的执行权限): 

```json showLineNumbers
{
  "actions": {
    "clone_env": {
      "execute": {
        // highlight-next-line
        "roles": ["Admin", "Env-moderator"], // changed from ["Admin", "Env-moderator", "Member"]
        "users": [],
        "teams": [],
        "ownedByTeam": false
      }
    }
  }
}
```

</TabItem>

</Tabs>

## 设置 Slack 通知

要启用 Slack 通知，您需要创建一个 Slack 应用程序，并按照[Slack API Documentation](https://api.slack.com/messaging/webhooks) 中概述的步骤 1-3 将其安装到工作区中。

完成安装过程后，您将获得如下所示的 webhook URL: 

```text
https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX
```

现在，您可以使用 webhook URL 将手动审批通知发送到与 webhook URL 绑定的 Slack 频道。

为此，请修改操作配置中的 "approvalNotification "字段，如下所示: 

```json showLineNumbers
{
  ...
  "requiredApproval": true,
  // highlight-start
  "approvalNotification": {
    "type": "webhook",
    "format": "slack",
    "url": "https://my-slack-webhook.com"
  },
  // highlight-end
  ...
}
```