---

sidebar_position: 14
description: 将 Grafana 警报纳入目录

---

import GrafanaAlertBlueprint from './resources/grafana/_example_alert_blueprint.mdx'
import GrafanaAlertWebhookConfig from './resources/grafana/_example_alert_webhook_configuration.mdx'

# Grafana

在本示例中，您将在[Grafana](https://grafana.com/) 和 Port 之间创建一个 webhook 集成，用于接收警报实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Alert blueprint</summary>

<GrafanaAlertBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Alert webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Grafana Alert Mapper`；
    2.标识符 : `grafana_alert_mapper`；
    3.Description : `将 Grafana 警报映射到 Port` 的 webhook 配置；
    4.图标 : `Grafana`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <GrafanaAlertWebhookConfig/>
3.点击页面底部的**保存**。

</details>

## 在 Grafana 中创建 webhook

1. 转到 Grafana 账户中的**警报**；
2. 在**联系点**下点击**添加联系点**；
3. 输入以下详细信息: 
    1. 名称"- 被引用一个有意义的名称，如 Port Webhook；
    2. 集成"- 从列表中选择 "Webhook"；
    3. `URL` - 输入创建 Webhook 配置后收到的 `url` 键的值；
4.单击 **保存联系人点**保存联系人；
5.转到**通知策略**，将 Port Webhook 联系点添加到您的**默认策略**，并在 Grafana 中收到任何警报通知；
6.您可以选择将联系点添加到现有的通知策略或创建新策略，具体取决于您的用例；
7.单击**保存策略**。

完成！您在 Grafana 中的警报发生的任何变化都会触发一个 Webhook 事件，发送到 Port 提供的 Webhook URL。 Port 将根据映射解析事件，并相应地更新目录实体。