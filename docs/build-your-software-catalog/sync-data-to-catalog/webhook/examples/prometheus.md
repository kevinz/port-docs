---
sidebar_position: 16
description: 将 Prometheus 警报纳入目录

---

import AlertBlueprint from './resources/prometheus/_example_alert_blueprint.mdx'
import AlertWebhookConfig from './resources/prometheus/_example_alert_webhook_config.mdx'

# 普罗米修斯

在本示例中，您将在[Prometheus Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) 和 Port 之间创建一个 webhook 集成，用于接收警报实体。

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
    1.title: `Prometheus Alert Mapper`；
    2.标识符 : `prometheus_alert_mapper`；
    3.Description : `将 Prometheus 警报映射到 Port` 的 webhook 配置；
    4.图标 : `Prometheus`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <AlertWebhookConfig/>
3.点击页面底部的**保存**。

</details>

## 配置 Alertmanager 以发送 webhook

1. 确保已按照[prometheus/alertmanager](https://github.com/prometheus/alertmanager#installation) 中的说明安装 Prometheus Alertmanager；
2. 配置 Alertmanager，以便从服务器发送警报信息到 Port。编辑 Alertmanager 配置文件 (`alertmanager.yaml`)，将从 Port 生成的 webhook 添加为**接收器**；
    1.创建名为 `port_webhook` 的新**接收器**对象。将 webhook 的 `URL` 粘贴到 `url` 字段，并将 `send_resolved` 值设置为 `true`。
    2.将 `port_webhook` **receivers** 添加到 **route** 对象中；
   <details>
   <summary>Example configuration file.</summary>

   ```yaml showLineNumbers
   global:
     resolve_timeout: 20s

   route:
     group_wait: 30s
     group_interval: 5m
     repeat_interval: 3h
     receiver: port_webhook

   receivers:
    - name: port_webhook
    webhook_configs:
    - url: https://port-webhook-url
       send_resolved: true
   ```

   </details>

3. Save the `alertmanager.yaml` file and restart the alertmanager to apply the changes.

Done! Any change that happens to your alerts in your server will trigger a webhook event to the webhook URL provided by Port. Port will parse the events according to the mapping and update the catalog entities accordingly.
```
