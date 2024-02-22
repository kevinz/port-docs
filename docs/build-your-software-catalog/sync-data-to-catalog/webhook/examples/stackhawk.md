---
sidebar_position: 12
description: 将 StackHawk 应用程序漏洞纳入 Port

---

import StackHawkVulnerabilityBlueprint from "./resources/stackhawk/_example_vulnerability_blueprint.mdx";
import StackHawkWebhookConfig from "./resources/stackhawk/_example_webhook_configuration.mdx";

# StackHawk

在本示例中，您将在[StackHawk](https://www.stackhawk.com/) 和 Port 之间创建一个 webhook 集成，将应用程序漏洞摄取到 Port 中。该集成将包括设置一个 webhook，以便在扫描应用程序漏洞时接收来自 StackHawk 的通知，从而使 Port 能够摄取和处理相应的漏洞实体。

## 先决条件

创建以下蓝图定义和 webhook 配置: 

<details>
<summary>StackHawk vulnerability blueprint</summary>

<StackHawkVulnerabilityBlueprint/>

</details>

<details>
<summary>StackHawk webhook configuration</summary>

切记用在 StackHawk 中订阅 webhook 时提供的真实 secret 和 header 值更新 `WEBHOOK_SECRET` 和 `AUTH_SIGNATURE_HEADER` 。

<StackHawkWebhookConfig/>

</details>

#### 创建 StackHawk webhook

1. 转到[StackHawk](https://app.stackhawk.com) ，选择要配置 webhook 的账户。
2. 导航至左侧导航栏中的**Integrations**，然后点击**Generic Webhook**。
3. 点击**添加网络钩子**，并提供以下信息: 
    1. 名称"- 输入 Webhook 的名称。
    2. `Description` - 为您的 Webhook 提供详细描述。
    3. 扫描数据"- 选择要接收 webhook 事件的应用程序，如果要为所有应用程序配置全局 webhook，则选择 "全选"。
    4. 扫描事件"- 选择 "扫描完成 "事件类型。
    5.Webhook Endpoint URL` - 输入[creating the webhook configuration](../webhook.md#configuring-webhook-endpoints) 后收到的 `url` 键的值。
    6.授权头名称"- 输入包含授权令牌/密钥的 HTTP 头的名称。例如，可以输入 `X-StackHawk-Port-Webhook`。
    7.Auth Header Value` - 输入将添加到 webhook 有效负载的secret身份验证令牌。
4.单击**创建和测试**创建您的 webhook。

:::tip 为了查看 StackHawk webhooks 中可用的不同事件，请点击以下链接、[look here](https://docs.stackhawk.com/workflow-integrations/webhook.html)

:::

完成！StackHawk 中应用程序发生的任何更改都将触发指向 Provider 提供的 Webhook URL 的 Webhook 事件。 Port 将根据映射解析事件，并相应地更新目录实体。

## 测试 webhook

本节包括一个当应用程序被扫描时从 StackHawk 发送的 webhook 事件示例。 此外，本节还包括根据上一节提供的 webhook 配置从该事件创建的实体。

### 有效载荷

下面是扫描应用程序时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "service": "StackHawk",
  "scanCompleted": {
    "scan": {
      "id": "c30e848d-35a7-4357-a113-35ce3392e967",
      "hawkscanVersion": "3.1.0",
      "env": "Development",
      "status": "COMPLETED",
      "application": "Python App",
      "startedTimestamp": "2023-06-23T11:01:18.273Z",
      "scanURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967",
      "tags": ["test-application"]
    },
    "scanDuration": "72",
    "spiderDuration": "49",
    "completedScanStats": {
      "urlsCount": "2",
      "duration": "121",
      "scanResultsStats": {
        "totalCount": "5",
        "lowCount": "7",
        "mediumCount": "4",
        "highCount": "0",
        "lowTriagedCount": "0",
        "mediumTriagedCount": "0",
        "highTriagedCount": "0"
      }
    },
    "findings": [
      {
        "pluginId": "10021",
        "pluginName": "X-Content-Type-Options Header Missing",
        "severity": "Low",
        "host": "https://example.com",
        "paths": [
          {
            "path": "",
            "method": "GET",
            "status": "NEW",
            "pathURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10021/path/769898/message/4"
          }
        ],
        "pathStats": [
          {
            "status": "NEW",
            "count": 1
          }
        ],
        "totalCount": "1",
        "category": "Information Leakage",
        "findingURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10021"
      },
      {
        "pluginId": "10035",
        "pluginName": "Strict-Transport-Security Header Not Set",
        "severity": "Low",
        "host": "https://example.com",
        "paths": [
          {
            "path": "/robots.txt",
            "method": "GET",
            "status": "NEW",
            "pathURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10035/path/769897/message/7"
          },
          {
            "path": "/sitemap.xml",
            "method": "GET",
            "status": "NEW",
            "pathURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10035/path/769896/message/6"
          },
          {
            "path": "",
            "method": "GET",
            "status": "NEW",
            "pathURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10035/path/769898/message/4"
          }
        ],
        "pathStats": [
          {
            "status": "NEW",
            "count": 3
          }
        ],
        "totalCount": "3",
        "category": "Information Leakage",
        "findingURL": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10035"
      }
    ]
  }
}
```

</details>

### 绘制结果

结合示例有效载荷和 webhook 配置会生成以下 Port 实体(在上述示例中，由于 `findings` 数组包含多个对象，因此会生成多个实体): 

```json showLineNumbers
{
  "identifier": "Python-App-10020-1",
  "title": "Missing Anti-clickjacking Header",
  "blueprint": "stackHawkVulnerability",
  "properties": {
    "severity": "Medium",
    "host": "https://example.com",
    "totalCount": 1,
    "category": "Information Leakage",
    "url": "https://app.stackhawk.com/scans/c30e848d-35a7-4357-a113-35ce3392e967/finding/10020-1",
    "tags": ["test-application"]
  },
  "relations": {}
}
```