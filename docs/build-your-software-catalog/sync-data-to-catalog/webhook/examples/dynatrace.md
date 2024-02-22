---
sidebar_position: 7
description: 将 Dynatrace 问题纳入您的目录

---

import DynatraceProblemBlueprint from "./resources/dynatrace/_example_dynatrace_problem_blueprint.mdx";
import DynatraceProblemConfiguration from "./resources/dynatrace/_example_dynatrace_problem_webhook_configuration.mdx"
import DynatraceMicroserviceBlueprint from "./resources/dynatrace/_example_dynatrace_microservice_blueprint.mdx"

# Dynatrace

在本示例中，您将在[Dynatrace](https://www.dynatrace.com/) 和 Port 之间创建一个 webhook 集成，它将把问题实体摄取到 Port，并将其映射到您的微服务实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Dynatrace microservice blueprint</summary>
<DynatraceMicroserviceBlueprint/>
</details>

<details>
<summary>Dynatrace problem blueprint</summary>
<DynatraceProblemBlueprint/>
</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>
<summary>Dynatrace problem webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Dynatrace Problem Mapper`；
    2.标识符: `dynatrace_problem_mapper`；
    3.Description : `来自 Dynatrace 的问题事件的 webhook 配置；
    4.图标 : `Dynatrace`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <DynatraceProblemConfiguration/>
3.单击页面底部的**保存**。

</details>

:::note 只有当 Port 微服务实体的标识符与 Dynatrace 中实体的名称相匹配时，webhook 配置的关系映射才能正常运行。

如果出现不匹配，可以利用[Dynatrace Tags](https://www.dynatrace.com/support/help/manage/tags-and-metadata) 对齐 Port 中的实际标识符。

为此，请创建一个关键字为 proj、值为 `microservice_identifier` 的标记。

然后，更新关系 JQ 语法，在 Dynatrace 问题和 Port 微服务之间建立连接。 以下是更新后的 JQ 映射: 

```json showLineNumbers
{
    "blueprint": "dynatraceProblem",
    "entity": {
     ...Properties mappings
      "relations": {
         "microservice": ".body.ProblemTags | split(\", \") | map(select(test(\"proj:\")) | sub(\"proj:\";\"\"))"
      }
    }
}
```

<details>
<summary>JQ expression explained</summary>
The above JQ expression will split the tags by comma and space, then filter the tags that start with `proj:` and remove the `proj:` prefix from the tag value.
</details>

:::

## 在 Dynatrace 中创建 webhook

1. 使用您的凭据登录 Dynatrace；
2. 单击页面左侧边栏的**设置**；
3. 选择**集成**，然后单击**问题通知**；
4. 选择**添加通知**；
5. 从可用的集成类型中选择**自定义集成**；
6. 输入以下详细信息: 
    1. 显示名称"- 使用有意义的名称，如 Port Webhook；
    2. Webhook URL` - 输入创建 Webhook 配置后收到的 `url` 键的值；
    3. `Overview` - 您可以在 webhook 请求中添加一个可选的 HTTP 标头；
    4. 自定义有效载荷"- 当在您的实体上检测或解决问题时，此有效载荷将被发送到 webhook URL。您可以在文本框中输入此 JSON 占位符；


      ```json showLineNumbers
      {
         "State":"{State}",
         "PID":"{PID}",
         "ProblemTitle":"{ProblemTitle}",
         "ImpactedEntity": "{ImpactedEntity}",
         "ProblemDetailsText": "{ProblemDetailsText}",
         "ProblemImpact": "{ProblemImpact}",
         "ProblemSeverity": "{ProblemSeverity}",
         "ProblemURL": "{ProblemURL}",
         "ProblemTags": "{Tags}",
         "ImpactedEntities": {ImpactedEntities}
      }
      ```

   5. `Alerting profile` - configure your preferred alerting rule or use the default one;
7. Click **Save changes** at the bottom of the page;

:::tip 要查看 Dynatrace webhooks 中可用的不同有效载荷和事件，请执行以下操作[look here](https://www.dynatrace.com/support/help/observe-and-explore/notifications-and-alerting/problem-notifications/webhook-integration)

:::

完成！在 Dynatrace 实体上检测到的任何问题都将触发 webhook 事件。 Port 将根据映射解析事件并相应更新目录实体。

## Ingest Dynatrace Entities

在此示例中，您将创建一个 `dynatrace_entity` 蓝图，用于从 Dynatrace 帐户中摄取受监控的实体。 然后，您将添加一些 python 脚本，以调用 Dynatrace REST API 并获取帐户数据。

* * [Code Repository Example](https://github.com/port-labs/example-dynatrace-entities)
