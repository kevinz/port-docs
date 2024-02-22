---
sidebar_position: 9
description: 将 Datadog 警报/监视器纳入目录

---

import DatadogBlueprint from "./resources/datadog/_example_datadog_alert_blueprint.mdx";
import DatadogConfiguration from "./resources/datadog/_example_datadog_webhook_configuration.mdx"
import DatadogMicroserviceBlueprint from "./resources/datadog/_example_datadog_microservice.mdx"

# Datadog

在本示例中，您将在[Datadog](https://www.datadoghq.com/) 和 Port 之间创建 webhook 集成，将警报和监控实体摄取到 Port，并将其映射到您的微服务实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Datadog microservice blueprint</summary>
<DatadogMicroserviceBlueprint/>
</details>

<details>
<summary>Datadog alert/monitor blueprint</summary>
<DatadogBlueprint/>
</details>

创建以下 webhook 配置[using Port UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Datadog webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Datadog Alert Mapper`；
    2.标识符 : `datadog_alert_mapper`；
    3.Description : `来自 Datadog 的警报/监控事件的 webhook 配置；
    4.图标 : `Datadog`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <DatadogConfiguration/>
3.单击页面底部的**保存**。

</details>

:::note 只有当 Port 微服务实体的标识符与 Datadog 中服务或主机的名称相匹配时，webhook 配置的关系映射才能正常运行。

:::

## 在 Datadog 中创建 webhook

1. 使用您的凭据登录 Datadog；
2. 点击页面左侧边栏的**集成**；
3. 在搜索框中搜索**Webhooks**，然后选择它；
4. 进入 "**配置**"选项卡，并按照安装说明进行操作；
5. 点击**新**；
6. 输入以下详细信息: 
    1. 名称 `Name` - 被用于一个有意义的名称，如 Port_Webhook；
    2. `URL` - 输入[creating the webhook configuration](../webhook.md#configuring-webhook-endpoints) 后收到的 `url` 键的值；
    3. `Payload` - 当监控器触发警报时，此有效载荷将被发送到 webhook URL。您可以在文本框中输入此 JSON 占位符；


      ```json showLineNumbers
      {
        "id": "$ID",
        "message": "$TEXT_ONLY_MSG",
        "priority": "$PRIORITY",
        "last_updated": "$LAST_UPDATED",
        "event_type": "$EVENT_TYPE",
        "event_url": "$LINK",
        "service": "$HOSTNAME",
        "creator": "$USER",
        "title": "$EVENT_TITLE",
        "date": "$DATE",
        "org_id": "$ORG_ID",
        "org_name": "$ORG_NAME",
        "alert_id": "$ALERT_ID",
        "alert_metric": "$ALERT_METRIC",
        "alert_status": "$ALERT_STATUS",
        "alert_title": "$ALERT_TITLE",
        "alert_type": "$ALERT_TYPE",
        "tags": "$TAGS"
      }
      ```

   4. `Custom Headers` - configure any custom HTTP header to be added to the webhook event. The format for the header should be in JSON;
7. Click **Save** at the bottom of the page;

:::tip 为了查看 Datadog webhook 中事件的不同有效载荷和结构、[look here](https://docs.datadoghq.com/integrations/webhooks/#variables)

:::

完成！在 Datadog 实例上检测到的任何问题都将触发 webhook 事件。 Port 将根据映射解析事件，并相应地更新目录实体。

## 摄取服务级别目标(SLO)

本指南将指导您完成将 Datadog SLO 导入 Port 的步骤。 按照这些步骤，您将能够在 Port 中创建一个 `microservice` 实体蓝图，代表 Datadog 帐户中的一个服务。 此外，您还将在此服务和 `datadogSLO` 蓝图之间建立关系，从而能够从 Datadog 帐户中导入所有已定义的 SLO。

Provider 提供的示例演示了如何使用 GitLab Pipelines 按预定时间间隔从 Datadog 的 REST API 提取数据并将数据报告给 Port。

* * [GitLab CI Pipeline Example](https://github.com/port-labs/datadog-slo-example)

## 从 APM 获取服务依赖性

在本示例中，您将创建一个 `service` 蓝图，使用 REST API 在 Datadog APM 中引用所有服务及其相关依赖关系。 然后，您将添加一些 shell 脚本，以便每次 GitLab CI 被日程触发时在 Port 中创建新实体。

* * [GitLab CI Pipeline Example](https://github.com/port-labs/datadog-service-dependency-example)

## 接收服务目录

在本示例中，您将创建一个 `datadogServiceCatalog` 蓝图，用于从 Datadog 帐户中摄取所有服务目录。 然后，您将添加一些 python 脚本，以便向 Datadog REST API 进行 API 调用，并为您的帐户获取数据。

* * [Code Repository Example](https://github.com/port-labs/datadog-service-catalog)
