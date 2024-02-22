---
sidebar_position: 13
description: 将拆分 IO 印象摄入您的目录

---

import SplitioBlueprint from './resources/splitio/_example_splitio_impression_blueprint.mdx'
import SplitioWebhookConfig from './resources/splitio/_example_splitio_webhook_configuration.mdx'

# 分割

在本示例中，您将在[Split](https://www.split.io/) 和 Port 之间创建一个 webhook 集成，用于接收印象信息。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Split impression blueprint</summary>

<SplitioBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Split webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Split Mapper`；
    2.标识符 : `split_mapper`；
    3.Description : `将 Splitio 印象映射到 Port` 的 webhook 配置；
    4.图标 : `Jenkins`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <SplitioWebhookConfig/>
3.点击页面底部的**保存**。

</details>

## 在 Split 中创建 webhook

1. 转到管理设置。
2. 单击 "集成"。
3. 选择您的 Workspace。
4. 单击外发 Webhook (Impressions) 旁边的添加。
5. 选中您希望发送数据的环境。
6. 输入创建 Webhook 配置后收到的 `url` 键的值。
7. 单击保存。

完成！任何时候触发印象，webhook 都会将数据发送到 Port 并创建一个新的 "分割印象 "实体

:::info 要查看某个印象的所有可用数据，请访问[Split's documentation](https://help.split.io/hc/en-us/articles/360020700232-Webhook-impressions)

:::