---

sidebar_position: 18
description: 将 Falco 警报纳入您的目录

---

import AlertBlueprint from './resources/falco/_example_alert_blueprint.mdx'
import AlertWebhookConfig from './resources/falco/_example_webhook_configuration.mdx'

# Falco Sidekick

在本示例中，您将在[Falco Sidekick](https://github.com/falcosecurity/falcosidekick) 和 Port 之间创建一个 webhook 集成，用于接收警报实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Alert blueprint</summary>

<AlertBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Alert webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Falco Alert Mapper`；
    2.标识符 : `falcoo_alert_mapper`；
    3.Description : `将 Falco sidekicks 警报映射到 Port` 的 webhook 配置；
    4.图标 : `Alert`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <AlertWebhookConfig/>
3.单击页面底部的**保存**。

</details>

## 配置 Falco Sidekick 以发送 webhook

1. 如果您使用的是带有[Docker](https://github.com/falcosecurity/falcosidekick#with-docker) 的 Falcosidekick，请使用以下命令进行安装。将 `YOUR_WEBHOOK_URL` 替换为创建 webhook 配置后收到的 `url` 密钥的值；


   ```bash showLineNumbers
   docker run -d -p 2801:2801 -e WEBHOOK_ADDRESS=YOUR_WEBHOOK_URL falcosecurity/falcosidekick
   ```


2.如果您希望使用[Helm](https://github.com/falcosecurity/falcosidekick#with-helm) 安装 Falcosidekick，请按照以下步骤操作: 
    1.将 webhook 配置添加到 config.yaml 文件中，将 `YOUR_WEBHOOK_URL` 替换为 webhook 设置中的实际 URL。
   <details>
   <summary>Example configuration file</summary>


   ```yaml showLineNumbers
   webhook:
     address: YOUR_WEBHOOK_URL
   ```


   </details>

2.使用以下命令安装或升级 Helm chart: 


   ```bash showLineNumbers
   helm repo add falcosecurity https://falcosecurity.github.io/charts
   helm repo update

helm install falco --config-file=config.yaml falcosecurity/falco

```


Done! Any change that happens to your alerts in your server will trigger a webhook event to the webhook URL provided by Port. Port will parse the events according to the mapping and update the catalog entities accordingly.
```