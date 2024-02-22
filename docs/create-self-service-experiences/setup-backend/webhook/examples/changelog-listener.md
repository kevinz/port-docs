---

sidebar_position: 2

---

# 更新日志监听器

自助服务行动的一个常见被引用案例是监听软件目录中的变化并对其做出反应。

例如

* 当数据库的 CPU 利用率超过 80% 时，您可能希望触发一个作业来关闭陈旧的连接、杀死停滞的查询或确保一切运行正常。
* 当微服务部署的健康检查状态从 "健康 "变为 "降级 "时，您可能需要执行一些扩展或缩放操作，或者向负责微服务团队的 slack 频道发送警报消息，以确保值班人员了解该问题。

## 目标

在下面的示例中，您将创建一个更新日志监听器，该监听器能以自定义方式对 Port 中的更改做出反应。

要实现监听器，请被引用以下内容: 

* Python 与[FastAPI](https://fastapi.tiangolo.com/) - 设置 API 以处理 webhook 请求；
* [Smee.io](https://smee.io) 和[pysmee](https://pypi.org/project/pysmee/) - 将 webhook 请求重定向到本地 API；
* Port 的更新日志功能--每当软件目录中的内容发生变化时，向您的 API 发送事件；
* Slack webhooks - 向您的 slack 服务器发送消息，提醒您的用户发生变化。

**每当您的 "deploymentConfig "蓝图或其中一个实体发生变化时，都会触发您的 API。

每次实体的 "healthStatus "字段发生变化时，我们都会向 Slack 发送一条消息。

## 使用 Smee 创建 webhook URL

访问[smee.io](https://smee.io) 并点击 "Start a new channel"(启动一个新频道)，您应该会在页面顶部看到 "webhook proxy URL"(webhook 代理 URL)。您将在下一节中使用它来指定 webhook 目的地。

## 创建部署配置蓝图

部署配置用于在基础架构的特定环境中表示服务部署。 部署配置有多个与之绑定的 "部署"，每个 "部署 "代表匹配环境中已部署代码的匹配服务的新版本。

部署配置 "听起来就像 "配置"，这意味着它是存储运行时变量和 Values、日志、跟踪或仪表盘工具链接以及更多在部署之间不会改变的静态数据的好地方。

在这个有限的示例中，"部署配置 "蓝图将主要包括可能频繁更改的状态属性，从而显示 Port 的更新日志监听器功能: 

<details>
<summary>Deployment Config Blueprint JSON</summary>

```json showLineNumbers
{
  "identifier": "deploymentConfig",
  "title": "Deployment Config",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "healthStatus": {
        "type": "string",
        "title": "Health Status",
        "enum": ["Healthy", "Degraded", "Crashed", "Restarting"],
        "enumColors": {
          "Healthy": "green",
          "Degraded": "orange",
          "Crashed": "red",
          "Restarting": "yellow"
        }
      },
      "cpuUtil": {
        "type": "number",
        "title": "CPU Utilization"
      },
      "memoryUtil": {
        "type": "number",
        "title": "Memory Utilization"
      },
      "newRelicUrl": {
        "type": "string",
        "format": "url",
        "title": "New Relic",
        "description": "Link to the new relic dashboard of the service"
      },
      "sentryUrl": {
        "type": "string",
        "format": "url",
        "title": "Sentry URL",
        "description": "Link to the new sentry dashboard of the service"
      },
      "prometheusUrl": {
        "type": "string",
        "format": "url",
        "title": "Prometheus URL"
      },
      "locked": {
        "type": "boolean",
        "title": "Locked",
        "default": false,
        "description": "Are deployments currently allowed for this configuration",
        "icon": "Lock"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {},
  "changelogDestination": {
    "type": "WEBHOOK",
    "url": "YOUR_WEBHOOK_URL"
  }
}
```

</details>

:::info 请记住，要向 webhook 报告更新日志事件，您需要在 Blueprint 定义中提供 changelogDestination 关键字。

:::

此外，您还可以在下面找到与蓝图模式相匹配的 "部署配置 "实体: 

<details>
<summary>Deployment Config Entity JSON</summary>

```json showLineNumbers
{
  "identifier": "notification-service-prod",
  "title": "Notification Service Production",
  "properties": {
    "healthStatus": "Healthy",
    "cpuUtil": 25,
    "memoryUtil": 30,
    "newRelicUrl": "https://newrelic.com",
    "sentryUrl": "https://sentry.io/",
    "prometheusUrl": "https://prometheus.io",
    "locked": false
  },
  "relations": {}
}
```

</details>

下面是创建部署配置蓝图和实体的 "python "代码片段: 

:::note 切记用您的 Port 客户端 ID、secret 和 webhook URL(或 Smee 代理 URL)替换`YOUR_CLIENT_ID`、`YOUR_CLIENT_SECRET`和`YOUR_WEBHOOK_URL`的占位符。

:::

<details>
<summary>Click here to see the code</summary>

```python showLineNumbers
import requests

CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'
WEBHOOK_URL = 'YOUR_WEBHOOK_URL'

API_URL = 'https://api.getport.io/v1'

target_blueprint = 'deploymentConfig'

credentials = {'clientId': CLIENT_ID, 'clientSecret': CLIENT_SECRET}

token_response = requests.post(f'{API_URL}/auth/access_token', json=credentials)

access_token = token_response.json()['accessToken']

headers = {
    'Authorization': f'Bearer {access_token}'
}

blueprint = {
    "identifier": target_blueprint,
    "title": "Deployment Config",
    "icon": "Microservice",
    "schema": {
        "properties": {
            "healthStatus": {
                "type": "string",
                "title": "Health Status",
                "enum": ["Healthy", "Degraded", "Crashed", "Restarting"],
                "enumColors": {
                    "Healthy": "green",
                    "Degraded": "orange",
                    "Crashed": "red",
                    "Restarting": "yellow"
                }
            },
            "cpuUtil": {
                "type": "number",
                "title": "CPU Utilization"
            },
            "memoryUtil": {
                "type": "number",
                "title": "Memory Utilization"
            },
            "newRelicUrl": {
                "type": "string",
                "format": "url",
                "title": "New Relic",
                "description": "Link to the new relic dashboard of the service"
            },
            "sentryUrl": {
                "type": "string",
                "format": "url",
                "title": "Sentry URL",
                "description": "Link to the new sentry dashboard of the service"
            },
            "prometheusUrl": {
                "type": "string",
                "format": "url",
                "title": "Prometheus URL"
            },
            "locked": {
                "type": "boolean",
                "title": "Locked",
                "default": False,
                "description": "Are deployments currently allowed for this configuration",
                "icon": "Lock"
            }
        },
        "required": []
    },
    "mirrorProperties": {},
    "calculationProperties": {},
    "relations": {},
    "changelogDestination": {
        "type": "WEBHOOK",
        "url": WEBHOOK_URL
    }
}

entity = {
    "identifier": "notification-service-prod",
    "title": "Notification Service Production",
    "properties": {
        "healthStatus": "Healthy",
        "cpuUtil": 25,
        "memoryUtil": 30,
        "newRelicUrl": "https://newrelic.com",
        "sentryUrl": "https://sentry.io/",
        "prometheusUrl": "https://prometheus.io",
        "locked": False
    },
    "relations": {}
}

blueprint_response = requests.post(f'{API_URL}/blueprints', headers=headers, json=blueprint)
print(blueprint_response.json())

entity_response = requests.post(f'{API_URL}/blueprints/{target_blueprint}/entities', json=entity, headers=headers)

print(entity_response.json())
```

</details>

## 设置松弛网络钩子

前往[slack apps](https://api.slack.com/apps) 页面，创建一个新应用程序(或从现有应用程序中选择一个) 。 然后，前往 "Incoming Webhooks "页面，创建一个新的 webhook，指定服务器上的目标频道，发送到 slack webhook 的消息将在该频道传输。

复制 webhook URL，下一步将用它来设置 python FastAPI。

## 设置目标 webhook

现在，您要设置一个完整的基本 API，以便从 Port 中的软件目录接收更新日志事件。

### 先决条件

请使用 `pip` 为 API 安装 `python` 依赖项: 

```bash showLineNumbers
pip install fastapi pysmee pydantic uvicorn slack_sdk
```

### 设置应用程序接口

API 的代码可在[**changelog-listener-example-api**](https://github.com/port-labs/port-changelog-listener-example-api) 代码库中找到，您可以克隆它，然后用您的 Port 客户端 ID、secret 和在[setting up a slack webhook](#setting-up-a-slack-webhook) 中生成的 Slack webhook URL 替换 `YOUR_CLIENT_ID`、`YOUR_CLIENT_SECRET` 和 `SLACK_WEBHOOK_URL` 占位符。

克隆版本库后，您将拥有以下目录结构: 

```
.
├── actions
│   ├── __init__.py
│   └── send_message.py
├── api
│   ├── endpoints
│   │   ├── __init__.py
│   │   └── slack.py
│   ├── __init__.py
│   └── deps.py
├── core
│   ├── __init__.py
│   └── config.py
├── schemas
│   ├── __init__.py
│   └── webhook.py
└── main.py
```

### 运行应用程序接口

要运行 API，请打开一个新终端，在克隆版本库的根目录下运行以下命令: 

```bash showLineNumbers
python main.py
```

您将看到类似这样的 Output: 

```bash showLineNumbers
INFO:     Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)
INFO:uvicorn.error:Uvicorn running on http://0.0.0.0:80 (Press CTRL+C to quit)
INFO:     Started reloader process [32362] using StatReload
INFO:uvicorn.error:Started reloader process [32362] using StatReload
INFO:     Started server process [32364]
INFO:uvicorn.error:Started server process [32364]
INFO:     Waiting for application startup.
INFO:uvicorn.error:Waiting for application startup.
INFO:     Application startup complete.
INFO:uvicorn.error:Application startup complete.
```

### 将 webhook 指向应用程序接口

关于这一步，您可以按照[debugging webhooks locally](../local-debugging-webhook.md) 页面上[forwarding events to localhost](../local-debugging-webhook.md#forwarding-events-to-localhost) 中的步骤进行操作。

:::note 记住要引用您的 `Smee 代理 URL`，并将 `http://localhost:3000/webhooks` 替换为 `http://localhost:80/api/slack`。

:::

## 在你的松弛频道中观察变化事件

要查看 API 的结果，请更新部署配置实体的 `healthStatus` 字段。

当 "部署配置 "实体的**健康状态**发生变化时，您应该会在创建松弛 webhook 时选择的松弛频道中看到一条新消息: 

![Software catalog changelog slack message](../../../../../static/img/self-service-actions/changelog-slack-message.png)

## 摘要

本例向您展示了 Port 更新日志功能的强大。

本指南所采取的行动仅供参考，不会对您的基础设施造成任何改变，但可以加以调整: 

* 提供更多云计算资源；
* 执行扩展操作；
* 提醒值班人员；
* 等等。
