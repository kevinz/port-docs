---
sidebar_position: 2
title: 快速入门

sidebar_label: ⏱️ 快速入门

---

# ⏱️ 快速入门

## Port 是什么？

Port 是一个开发者平台，通过创建一个单一的平台，作为企业技术栈中所有基础设施资产和操作的单一真实来源，让企业中的开发人员和 DevOps 的生活变得更轻松。

然后，Port 允许工程师以自助方式对这些资产执行操作，包括调配开发环境、了解谁是微服务的所有者，或 DevOps 希望自助和自动化的任何独特用例。

#### Port 可以帮助您

* 通过在一个地方映射所有软件和基础架构组件，创建一个全面的**软件目录**: 微服务、单体、部署、资源库、数据库等。
* 让您的开发人员在您设定的策略和防护栏内，对目录中暴露的任何资产(无论是否为微服务)进行调配、终止和执行第 2 天操作，确保组织内部流程的统一标准和治理。

![Developer Platform complete vision](../static/img/quickstart/platform-vision.png)

Port 的三个核心构建模块是_Blueprint_、_Entities_和_Relations_。 本教程将指导您完成平台的第一步，开始您的开发人员门户之旅！ 🚢

## 本教程的目标

本教程的目标是

* 向您介绍 Port 的核心组件；
* 让您熟悉 Port 的 Web UI；
* 查看与 Port API 交互的功能代码片段；
* 体验 Port 作为内部开发平台的强大功能。

本指南将为您在 Port 中开始构建软件目录和查看软件基础架构的完整镜像奠定基础。

您的开发人员将能看到特定环境中的所有服务及其状态: 

![Developer Portal Environment View for running services](../static/img/quickstart/EndResultEnvironmentPage.png)

此外，您的开发人员还能看到特定微服务部署的所有环境，以及哪个版本部署在哪里: 

![Developer Portal Service View for environments](../static/img/quickstart/EndResultServicePage.png)

让我们开始吧！ 🚢

## 定义蓝图

蓝图用于在 Port 中建立数据模型。 蓝图允许我们定义 _Entity_ 包含哪些属性和字段。

架构和部署千差万别，对数据表示和资产结构的偏好和标准也不尽相同。 因此，在 Port 中，您可以完全控制使用任何您希望的数据格式呈现数据的方式，从而使软件目录真正代表您对开发人员门户的所有需求。

但现在，让我们从一个简单的例子开始: 

您的组织使用微服务架构；单个**微服务**可部署到多个**环境**(生产、暂存、质量保证等)。

要创建软件目录，您需要摄取和跟踪您的微服务，跟踪您的现有环境，并绘制出哪个微服务部署在哪个环境中。

在本教程中，您将创建 3 个蓝图: 

* 服务；
* 环境；
* 运行服务。

请注意**运行中的服务**蓝图--它旨在表示您的**环境**中正在运行的**服务**部署。

:::tip 在本教程中，我们将演示如何使用 Port 的 Web UI 和 Port 的 REST API 执行每个步骤。

本教程包括与 Port 的 API 进行交互的各种示例。 如需了解更多信息，欢迎访问蓝图基础知识中的 API 部分。

Port 还有一个 GitHub 应用程序和一个 Terraform Providers，您可以用它们来引用数据并与 Port 的组件进行交互。

为便于阅读，用于复制和粘贴的片段以及示例将放在折叠框内: 

<details>
<summary>Example JSON block</summary>

```json showLineNumbers
{
  "foo": "bar"
}
```

</details>

<details>
<summary>Example code block</summary>

```python showLineNumbers
print("hello world!")
```

</details>

:::

#### 服务蓝图

我们的服务蓝图将包括以下属性(以及其他属性): 

* **Github URL** - 微服务 GitHub 资源库的链接；
* **On Call** - 当前待命开发人员；
* **上次事件** - 微服务上次发生事件的时间；
* **语言** - 微服务被引用的主要编程语言；
* **产品** - 微服务的业务单位类别；
* **最新版本** - 微服务的最新版本；
* **JIRA 问题数** - 当前打开的 JIRA 问题数；
* **Slack通知** - 微服务负责团队 Slack 频道的 URL。

:::tip 如果您想添加/删除属性，不用担心，您可以随时返回并重新编辑它们。

:::

此外，"待命 "字段被标记为 "必需"，这样我们就能随时知道谁是服务的当前待命人员。

#### 从用户界面

让我们访问[Port](https://app.getport.io/dev-portal) 并查看 DevPortal Builder 页面，在右上角点击**添加蓝图**并配置我们的第一个蓝图--**服务**，如下镜像所示: 

![Developer PortalCreate New Blueprint](../static/img/quickstart/newBlueprintButton.png)

点击按钮后，您将看到如下所示的创建表单: 

![Developer Portal New Blueprint Text](../static/img/quickstart/newBlueprintDefaultText.png)

为了创建服务蓝图，请被引用以下 JSON 主体: 

<details>
<summary>Service Blueprint JSON</summary>

```json showLineNumbers
{
  "identifier": "microservice",
  "description": "This blueprint represents service in our software catalog",
  "title": "Service",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "on-call": {
        "type": "string",
        "icon": "Okta",
        "title": "On Call",
        "format": "email",
        "default": "developer@getport.io"
      },
      "language": {
        "type": "string",
        "icon": "Git",
        "title": "Language",
        "default": "Node",
        "enum": ["GO", "Python", "Node"],
        "enumColors": {
          "GO": "red",
          "Python": "green",
          "Node": "blue"
        }
      },
      "number-of-open-jira-issues": {
        "type": "number",
        "icon": "DevopsTool",
        "title": "Number of JIRA Issues",
        "default": 42
      },
      "product": {
        "title": "Product",
        "type": "string",
        "icon": "Deployment",
        "default": "Analytics",
        "enum": ["SaaS", "Control Panel", "Analytics"],
        "description": "Choose product unit related to the service"
      },
      "url": {
        "type": "string",
        "title": "Github URL",
        "icon": "Github",
        "format": "url",
        "default": "https://git.com",
        "description": "the link to the repo in our github"
      },
      "config": {
        "title": "Service Config",
        "type": "object",
        "icon": "Argo",
        "default": {
          "foo": "bar"
        }
      },
      "monitor-links": {
        "title": "Monitor Tooling",
        "type": "array",
        "icon": "Datadog",
        "items": {
          "type": "string",
          "format": "url"
        },
        "default": [
          "https://grafana.com",
          "https://prometheus.com",
          "https://datadog.com"
        ]
      },
      "last-incident": {
        "icon": "CPU",
        "type": "string",
        "title": "Last Incident",
        "format": "date-time",
        "default": "2022-04-18T11:44:15.345Z"
      },
      "version": {
        "type": "string",
        "icon": "Package",
        "title": "Latest Version",
        "pattern": "[a-zA-Z0-9.]",
        "description": "A property that supports values specified by a regex pattern",
        "default": "Port1337"
      },
      "ip": {
        "title": "IPv4 Property",
        "icon": "Cluster",
        "type": "string",
        "format": "ipv4",
        "description": "An IPv4 property",
        "default": "127.0.0.1"
      }
    },
    "required": ["on-call"]
  },
  "mirrorProperties": {},
  "calculationProperties": {
    "slack-notifications": {
      "title": "Slack Notifications",
      "icon": "Link",
      "calculation": "'https://slack.com/' + .identifier",
      "type": "string",
      "format": "url"
    },
    "latest-build-output": {
      "title": "Latest Build Output",
      "icon": "Jenkins",
      "calculation": ".properties.url + '/' + .properties.version",
      "type": "string",
      "format": "url"
    },
    "launch-darkly": {
      "title": "Launch Darkly",
      "calculation": "'https://launchdarkly.com/' + .title",
      "type": "string",
      "format": "url"
    }
  },
  "relations": {}
}
```

</details>

单击 "保存 "按钮，然后在 DevPortal Builder 页面[you will see](#the-results) 新蓝图。

#### 来自应用程序接口

为了与 Port 的应用程序接口交互，我们将使用 python，唯一需要的软件包是[requests](https://pypi.org/project/requests/) 库，运行该库即可安装: 

```bash showLineNumbers
python -m pip install requests
```

:::note 接下来，您需要输入 Port `CLIENT_ID`和`CLIENT_SECRET`。

要查找您的 Port API 凭据，请访问[Port](https://app.getport.io) ，将鼠标悬停在右上角的 "3 点按钮 "上，选择 "凭据"，然后您就可以查看并复制您的 "CLIENT_ID "和 "CLIENT_SECRET": 

<center>

![Port Developer Portal Credentials Modal](../static/img/software-catalog/credentials-modal.png)

</center>

:::

要使用 Port 的应用程序接口执行任何操作，您首先需要一个**访问令牌: 

<details>
<summary>Get an API access token</summary>

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests

import requests

CLIENT_ID = 'YOUR_CLIENT_ID'
CLIENT_SECRET = 'YOUR_CLIENT_SECRET'

API_URL = 'https://api.getport.io/v1'

credentials = {'clientId': CLIENT_ID, 'clientSecret': CLIENT_SECRET}

token_response = requests.post(f'{API_URL}/auth/access_token', json=credentials)

access_token = token_response.json()['accessToken']

# You can now use the value in access_token when making further requests
```

:::tip 有关其他语言的示例，请访问 Blueprint basics 中的 API 部分。

:::

</details>

现在您已经有了访问令牌，可以在与 Port 的 API 进行的每次交互中使用它。 在下面创建 "服务 "蓝图的部分也将使用它: 

<details>
<summary>Create the service Blueprint</summary>

请注意，本示例假定令牌保存在 `access_token` 变量中。

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
import json
import requests

# the access_token variable should already have the token from the previous example

API_URL = 'https://api.getport.io/v1'

headers = {
    'Authorization': f'Bearer {access_token}'
}

blueprint = {
    "identifier": "microservice",
    "description": "This blueprint represents service in our software catalog",
    "title": "Service",
    "icon": "Microservice",
    "schema": {
        "properties": {
            "on-call": {
                "type": "string",
                "icon": "Okta",
                "title": "On Call",
                "format": "email",
                "default": "developer@getport.io"
            },
            "language": {
                "type": "string",
                "icon": "Git",
                "title": "Language",
                "default": "Node",
                "enum": ["GO", "Python", "Node"],
                "enumColors": {
                    "GO": "red",
                    "Python": "green",
                    "Node": "blue"
                }
            },
            "number-of-open-jira-issues": {
                "type": "number",
                "icon": "DevopsTool",
                "title": "Number of JIRA Issues",
                "default": 42
            },
            "product": {
                "title": "Product",
                "type": "string",
                "icon": "ApiDoc",
                "default": "Analytics",
                "enum": ["SaaS", "Control Panel", "Analytics"],
                "description": "Choose product unit related to the service"
            },
            "url": {
                "type": "string",
                "title": "Github URL",
                "icon": "Github",
                "format": "url",
                "default": "https://git.com",
                "description": "the link to the repo in our github"
            },
            "config": {
                "title": "Service Config",
                "type": "object",
                "icon": "Argo",
                "default": {
                    "foo": "bar"
                }
            },
            "monitor-links": {
                "title": "Monitor Tooling",
                "type": "array",
                "icon": "Datadog",
                "items": {
                    "type": "string",
                    "format": "url"
                },
                "default": ["https://grafana.com", "https://prometheus.com", "https://datadog.com"]
            },
            "last-incident": {
                "icon": "CPU",
                "type": "string",
                "title": "Last Incident",
                "format": "date-time",
                "default": "2022-04-18T11:44:15.345Z"
            },
            "version": {
                "type": "string",
                "icon": "Package",
                "title": "Latest Version",
                "pattern": "[a-zA-Z0-9.]",
                "description": "A property that supports values specified by a regex pattern",
                "default": "Port1337"
            },
            "ip": {
                "title": "IPv4 Property",
                "icon": "Cluster",
                "type": "string",
                "format": "ipv4",
                "description": "An IPv4 property",
                "default": "127.0.0.1"
            }
        },
        "required": ["on-call"]
    },
    "mirrorProperties": {},
    "calculationProperties": {
      "slack-notifications": {
        "title": "Slack Notifications",
        "icon": "Link",
        "calculation": "'https://slack.com/' + .identifier",
        "type": "string",
        "format": "url"
      },
      "latest-build-output": {
        "title": "Latest Build Output",
        "icon": "Jenkins",
        "calculation": ".properties.url + '/' + .properties.version",
        "type": "string",
        "format": "url"
      },
      "launch-darkly": {
        "title": "Launch Darkly",
        "calculation": "'https://launchdarkly.com/' + .title",
        "type": "string",
        "format": "url"
      }
  },
    "relations": {}
}

response = requests.post(f'{API_URL}/blueprints', json=blueprint, headers=headers)
# response.json() contains the content of the resulting blueprint

print(json.dumps(response.json(), indent=2))
```

</details>

#### 结果

![Developer Portal Blueprints graph with new Service Blueprint](../static/img/quickstart/blueprintGraphWithServiceClosed.png)

点击 "展开 "按钮，如下镜像所示: 

![Developer Portal Blueprints graph with new Service Blueprint And Expand Marked](../static/img/quickstart/blueprintGraphWithServiceClosedAndExpandMarked.png)

您将看到刚刚创建的蓝图的展开视图，其所有属性都与您提供的类型一并列出: 

![Developer Portal Blueprints graph with new Service open](../static/img/quickstart/blueprintGraphWithServiceOpen.png)

恭喜您！您刚刚创建了第一个蓝图！ 🎉

#### 环境蓝图

我们的环境蓝图将包括以下属性: 

* **环境类型** - 环境类型(生产、暂存、QA 等)；
* **Cloud Provider** - 部署集群的云提供商；
* **Region** - 部署集群的云区域。

此外，蓝图还将包括以下计算属性: 

* **Grafana URL** - 环境的 Grafana 面板链接。

:::tip 有关计算属性的更多信息，请点击此处。

:::

此外，"环境类型 "字段将被标记为 "必需"，这样我们就能确保我们的环境被正确标记。

要创建环境蓝图，请被引用以下 JSON 主体: 

<details>
<summary>Environment Blueprint JSON</summary>

```json showLineNumbers
{
  "identifier": "environment",
  "description": "This blueprint represents an environment in our software catalog",
  "title": "Environment",
  "icon": "Environment",
  "schema": {
    "properties": {
      "type": {
        "type": "string",
        "title": "Environment type",
        "enum": ["Integration", "Production", "Staging", "Security", "QA"]
      },
      "cloud-provider": {
        "type": "string",
        "title": "Cloud Provider",
        "enum": ["AWS", "GCP", "Azure", "Oracle"],
        "enumColors": {
          "AWS": "orange",
          "GCP": "green",
          "Azure": "blue",
          "Oracle": "red"
        }
      },
      "region": {
        "type": "string",
        "title": "Region",
        "enum": [
          "eu-west-1",
          "eu-west-2",
          "us-west-1",
          "us-east-1",
          "us-east-2"
        ]
      }
    },
    "required": ["type"]
  },
  "mirrorProperties": {},
  "calculationProperties": {
    "grafanaUrl": {
      "title": "Grafana URL",
      "calculation": "'https://grafana.com/' + .identifier",
      "type": "string"
    }
  },
  "relations": {}
}
```

</details>

#### 从用户界面

要从用户界面创建环境蓝图，请使用环境蓝图 JSON 重复[creating a service Blueprint from the UI](#from-the-ui) 中的步骤。

#### 来自应用程序接口

要从应用程序接口创建环境蓝图，请使用以下代码片段(切记需要访问令牌): 

<details>
<summary>Create the environment Blueprint</summary>

请注意，本示例假定令牌保存在 `access_token` 变量中。

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
import json
import requests

# the access_token variable should already have the token from the previous example

API_URL = 'https://api.getport.io/v1'

headers = {
    'Authorization': f'Bearer {access_token}'
}

blueprint = {
    "identifier": "environment",
    "description": "This blueprint represents an environment in our software catalog",
    "title": "Environment",
    "icon": "Environment",
    "schema": {
        "properties": {
            "type": {
                "type": "string",
                "title": "Environment type",
                "enum": [
                    "Integration",
                    "Production",
                    "Staging",
                    "Security",
                    "QA"
                ]
            },
            "cloud-provider": {
                "type": "string",
                "title": "Cloud Provider",
                "enum": [
                    "AWS",
                    "GCP",
                    "Azure",
                    "Oracle"
                ],
                "enumColors": {
                    "AWS": "orange",
                    "GCP": "green",
                    "Azure": "blue",
                    "Oracle": "red"
                }
            },
            "region": {
                "type": "string",
                "title": "Region",
                "enum": [
                    "eu-west-1",
                    "eu-west-2",
                    "us-west-1",
                    "us-east-1",
                    "us-east-2"
                ]
            }
        },
        "required": [
            "type"
        ]
    },
    "mirrorProperties": {},
    "calculationProperties": {
      "grafanaUrl": {
            "title": "Grafana URL",
            "calculation": "'https://grafana.com/' + .identifier",
            "type": "string"
        }
    },
    "relations": {}
}

response = requests.post(f'{API_URL}/blueprints', json=blueprint, headers=headers)
# response.json() contains the content of the resulting blueprint

print(json.dumps(response.json(), indent=2))
```

</details>

#### 结果

![Developer Portal Blueprints graph with new Environment Blueprint](../static/img/quickstart/blueprintGraphWithEnvironmentClosed.png)

点击 "展开 "按钮，如下镜像所示: 

![Developer Portal Blueprints graph with new Environment Blueprint And Expand Marked](../static/img/quickstart/blueprintGraphWithEnvironmentClosedAndExpandMarked.png)

您将看到刚刚创建的蓝图的展开视图，其所有属性都与您提供的类型一并列出: 

![Developer Portal Blueprints graph with new Environment open](../static/img/quickstart/blueprintGraphWithEnvironmentOpen.png)

在下一部分，我们将开始创建与我们创建的新蓝图相匹配的实体，组装软件目录！

## 创建第一个实体

现在我们已经有了 `environment` 和 `service` 的蓝图，可以添加一些 _Entities_ 了。

实体**是与某个蓝图的类型相匹配的对象。 在我们的例子中，我们在服务蓝图下创建的每个实体都是我们组织中的一个微服务。 而我们在环境蓝图下创建的每个环境都是我们代码运行的不同环境。

让我们慢慢来，先创建一些初始实体。

#### 服务和环境实体

#### 从用户界面

点击左侧边栏上的服务页面: 

![Developer Portal Blueprints graph with new Service and Services page marked](../static/img/quickstart/blueprintGraphWithServicesPageMarked.png)

在服务页面，单击 "+ 服务 "按钮创建新实体: 

![Developer Portal Service Entity page with create entity button marked](../static/img/quickstart/serviceEntityPageWithCreateEntityMarked.png)

点击按钮后，会出现一个新的服务表格。 让我们填写以下详细信息: 

<details>
<summary>Notification service Entity JSON</summary>

```json showLineNumbers
{
  "properties": {
    "on-call": "mor@getport.io",
    "language": "Python",
    "number-of-open-jira-issues": 21,
    "product": "Analytics",
    "url": "https://github.com/port/notification-service",
    "config": {
      "enable_analytics": true
    },
    "monitor-links": ["https://grafana.com", "https://datadog.com"],
    "last-incident": "2022-04-18T11:44:15.345Z",
    "version": "1.0.0",
    "ip": "8.8.8.8"
  },
  "relations": {},
  "title": "Notification Service",
  "identifier": "notification-service"
}
```

:::tip 您可以使用切换开关将创建表单切换为 Json 模式，也可以直接在字段中手动输入值。

:::

</details>

填写完以上所有内容后，您的创建页面应该是这样的: 

![Developer Portal Service Entity filled with create entity button marked](../static/img/quickstart/serviceEntityCreateFilledAndCreateMarked.png)

您可以继续按右下角的 "创建 "按钮(如上图所示)。

现在，要创建环境实体，请重复同样的步骤，但这次要转到环境页面: 

![Developer Portal Blueprints graph with new Environment and Environments page marked](../static/img/quickstart/blueprintGraphWithEnvironmentsPageMarked.png)

并为环境实体引用以下数据: 

<details>
<summary>Production environment Entity JSON</summary>

```json showLineNumbers
{
  "properties": {
    "type": "Production",
    "cloud-provider": "AWS",
    "region": "eu-west-1"
  },
  "relations": {},
  "title": "Production",
  "identifier": "production"
}
```

</details>

现在，您可以[witness your new service and environment](#the-results-2) 实体分别出现在服务页面和环境页面。

#### 来自应用程序接口

要从应用程序接口创建服务实体和环境实体，请使用以下代码片段(切记需要访问令牌): 

<details>
<summary>Create the service Entity and the environment Entity</summary>

请注意，本示例假定令牌保存在 `access_token` 变量中。

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
import json
import requests

# the access_token variable should already have the token from the previous example

API_URL = 'https://api.getport.io/v1'

headers = {
    'Authorization': f'Bearer {access_token}'
}

service_blueprint_identifier = 'microservice'

env_blueprint_identifier = 'environment'

service_entity = {
    "properties": {
        "on-call": "mor@getport.io",
        "language": "Python",
        "number-of-open-jira-issues": 21,
        "product": "Analytics",
        "url": "https://github.com/port/notification-service",
        "config": {
            "enable_analytics": True
        },
        "monitor-links": [
            "https://grafana.com",
            "https://datadog.com"
        ],
        "last-incident": "2022-04-18T11:44:15.345Z",
        "version": "1.0.0",
        "ip": "8.8.8.8"
    },
    "relations": {},
    "title": "Notification Service",
    "identifier": "notification-service"
}

env_entity = {
    "properties": {
        "type": "Production",
        "cloud-provider": "AWS",
        "region": "eu-west-1"
    },
    "relations": {},
    "title": "Production",
    "identifier": "production"
}

service_response = requests.post(f'{API_URL}/blueprints/{service_blueprint_identifier}/entities', json=service_entity, headers=headers)

print(json.dumps(service_response.json(), indent=2))

env_response = requests.post(f'{API_URL}/blueprints/{env_blueprint_identifier}/entities', json=env_entity, headers=headers)

print(json.dumps(env_response.json(), indent=2))
```

</details>

#### 结果

现在，每个蓝图的相应页面都会显示我们创建的实体: 

![Developer Portal Service Entity page with first entity](../static/img/quickstart/serviceEntityPageWithFirstEntity.png)

![Developer Portal Environment Entity page with first entity](../static/img/quickstart/environmentEntityPageWithFirstEntity.png)

了不起！你刚刚创造了两个了不起的实体🎉。

在结束使用 Port 的第一步时，我们使用 Blueprints 来定义数据模型，并使用 Entities 来存储与 Blueprints 类型相匹配的数据对象。

下一部分，我们将学习最后一个构件--"关系"。 让我们开始吧！

## 创建关系

关系(**Relation**)是两个蓝图和基于蓝图的实体之间的连接。 使用关系，您可以在多个实体之间创建连接图，连接图可以帮助您了解基础架构的结构，并更轻松地访问相关实体的数据。

目前，我们的软件目录包含服务和环境，但实际上，一个服务会同时部署到多个环境中。 为了跟踪所有不同的服务及其活动部署，我们现在要创建另一个蓝图 "运行中的服务"。 我们的运行中的服务蓝图将包含以下字段: 

* **健康状况** - 运行服务的健康状况；
* **Deployed branch** - 运行服务的代码所来自的分支；
* **Locked**-运行中的服务当前是否已锁定(这意味着不允许进行新的部署)；
* **版本** - 运行服务的当前版本。

此外，运行服务蓝图将包括两个**关系**: 

* 与服务的关系 - 表示运行服务所属的微服务；
* 与环境的关系--表示运行服务所部署的环境。

要创建运行中的服务蓝图，请被引用以下 JSON 主体: 

<details>
<summary>Running service Blueprint JSON</summary>

```json showLineNumbers
{
  "identifier": "runningService",
  "description": "This blueprint represents a running version of a service in an environment",
  "title": "Running Service",
  "icon": "DeployedAt",
  "schema": {
    "properties": {
      "health-status": {
        "type": "string",
        "title": "Health Status",
        "enum": ["Healthy", "Degraded", "Crashed"],
        "enumColors": {
          "Healthy": "green",
          "Degraded": "orange",
          "Crashed": "red"
        }
      },
      "locked": {
        "type": "boolean",
        "title": "Locked",
        "icon": "Lock",
        "default": false
      },
      "deployedBranch": {
        "type": "string",
        "title": "Deployed Branch"
      },
      "version": {
        "type": "string",
        "icon": "Package",
        "title": "Version",
        "pattern": "[a-zA-Z0-9.]",
        "default": "Port1337"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {
    "sentryUrl": {
      "title": "Sentry URL",
      "type": "string",
      "format": "url",
      "icon": "Link",
      "calculation": "'https://sentry.io/' + .identifier"
    },
    "newRelicUrl": {
      "title": "NewRelic URL",
      "type": "string",
      "format": "url",
      "icon": "Link",
      "calculation": "'https://newrelic.com/' + .identifier"
    }
  },
  "relations": {
    "microservice": {
      "target": "microservice",
      "description": "The service this running deployment belongs to",
      "many": false,
      "required": true,
      "title": "Service"
    },
    "environment": {
      "target": "environment",
      "description": "The environment this running deployment is deployed at",
      "many": false,
      "required": true,
      "title": "Environment"
    }
  }
}
```

</details>

:::tip 在正在运行的服务蓝图的蓝图 JSON 中，你会注意到一个新添加的内容: "关系 "对象现在填入了两个键--"服务 "和 "环境"，这两个键代表我们关系的目标蓝图。

这里还提供了与两个 "关系 "匹配的 JSON: 

<details>
<summary>Service Relation JSON</summary>

```json showLineNumbers
{
  "microservice": {
    "title": "Service",
    "description": "The service this running deployment belongs to",
    "target": "microservice",
    "required": true,
    "many": false
  }
}
```

</details>

<details>
<summary>Environment Relation JSON</summary>

```json showLineNumbers
{
  "environment": {
    "title": "Environment",
    "description": "The environment this running deployment is deployed at",
    "target": "environment",
    "required": true,
    "many": false
  }
}
```

</details>

:::

让我们继续创建**运行服务蓝图**: 

#### 从用户界面

* 返回蓝图页面；
* 单击 "添加蓝图 "按钮；
* 粘贴正在运行的服务蓝图 JSON 主体，然后单击 "保存

:::tip 
**请记住**，如果您在任何时候遇到问题，您都可以在[Define a Blueprint section](#define-a-blueprint) 的**服务**蓝图中执行完全相同的步骤，因此请随时回去参考。

:::

### 从应用程序接口

要从应用程序接口创建运行中的服务蓝图，请使用以下代码片段(切记需要访问令牌): 

<details>
<summary>Create the running service Blueprint</summary>

请注意，本示例假定令牌保存在 `access_token` 变量中。

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
import json
import requests

# the access_token variable should already have the token from the previous example

API_URL = 'https://api.getport.io/v1'

headers = {
    'Authorization': f'Bearer {access_token}'
}

blueprint = {
    "identifier": "runningService",
    "description": "This blueprint represents a running version of a service in an environment",
    "title": "Running Service",
    "icon": "DeployedAt",
    "schema": {
        "properties": {
            "health-status": {
                "type": "string",
                "title": "Health Status",
                "enum": [
                    "Healthy",
                    "Degraded",
                    "Crashed"
                ],
                "enumColors": {
                    "Healthy": "green",
                    "Degraded": "orange",
                    "Crashed": "red"
                }
            },
            "locked": {
                "type": "boolean",
                "title": "Locked",
                "icon": "Lock",
                "default": False
            },
            "deployedBranch": {
                "type": "string",
                "title": "Deployed Branch"
            },
            "version": {
                "type": "string",
                "icon": "Package",
                "title": "Version",
                "pattern": "[a-zA-Z0-9.]",
                "default": "Port1337"
            }
        },
        "required": []
    },
    "mirrorProperties": {},
    "calculationProperties": {
      "sentryUrl": {
        "title": "Sentry URL",
        "type": "string",
        "format": "url",
        "icon": "Link",
        "calculation": "'https://sentry.io/' + .identifier"
    },
    "newRelicUrl": {
        "title": "NewRelic URL",
        "type": "string",
        "format": "url",
        "icon": "Link",
        "calculation": "'https://newrelic.com/' + .identifier"
    }
    },
    "relations": {
        "microservice": {
            "target": "microservice",
            "description": "The service this running deployment belongs to",
            "many": False,
            "required": True,
            "title": "Service"
        },
        "environment": {
            "target": "environment",
            "description": "The environment this running deployment is deployed at",
            "many": False,
            "required": True,
            "title": "Environment"
        }
    }
}

response = requests.post(f'{API_URL}/blueprints', json=blueprint, headers=headers)
# response.json() contains the content of the resulting blueprint

print(json.dumps(response.json(), indent=2))
```

</details>

### 结果

完成后，您的 DevPortal Builder 页面将如下所示: 

![Developer Portal Blueprints Page with service, environment and running service](../static/img/quickstart/blueprintsGraphWithRunningServiceEnvironmentServiceRelation.png)

:::note 看一下你刚刚创建的连接图，你用一种能显示哪个蓝图依赖于另一个蓝图的方式模拟了蓝图之间的关系。

:::

现在，我们将创建一个正在运行的服务实体。

## 运行服务实体

我们的目标是跟踪微服务的运行版本及其各自的环境。

您已经创建了 2 个实体，它们是 "生产 "环境和 "通知服务 "服务。

现在，您要创建一个正在运行的服务实体，并使用您创建的关系将环境和服务映射到该实体。

为了创建运行中的服务实体，请被引用以下 JSON 主体: 

<details>
<summary>Notification service prod Running Service Entity JSON</summary>

```json showLineNumbers
{
  "properties": {
    "locked": true,
    "health-status": "Healthy",
    "deployedBranch": "main",
    "version": "1.0.0"
  },
  "relations": {
    "microservice": "notification-service",
    "environment": "production"
  },
  "title": "Notification Service Prod",
  "identifier": "notification-service-prod"
}
```

</details>

#### 从用户界面

要从用户界面创建实体，请重复[creating service and environment Entities from the UI](#from-the-ui-2) 中的步骤，使用 `notification-service-prod` 实体 JSON。

### 从应用程序接口

要从应用程序接口创建 "通知-服务-产品 "实体，请使用以下代码片段(切记需要访问令牌): 

<details>
<summary>Create the running service Entity</summary>

请注意，本示例假定令牌保存在 `access_token` 变量中。

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
import json
import requests

# the access_token variable should already have the token from the previous example

API_URL = 'https://api.getport.io/v1'

headers = {
    'Authorization': f'Bearer {access_token}'
}

running_service_blueprint_identifier = 'runningService'

running_service_entity = {
    "properties": {
        "locked": True,
        "health-status": "Healthy",
        "deployedBranch": "main",
        "version": "1.0.0"
    },
    "relations": {
        "microservice": "notification-service",
        "environment": "production"
    },
    "title": "Notification Service Prod",
    "identifier": "notification-service-prod"
}

running_service_response = requests.post(f'{API_URL}/blueprints/{running_service_blueprint_identifier}/entities',
                                         json=running_service_entity, headers=headers)

print(json.dumps(running_service_response.json(), indent=2))
```

</details>

### 结果

现在，您将看到新运行的服务实体，如果查看 "服务 "栏和 "环境 "栏，您将看到之前创建的服务和环境: 

![Developer Portal Running Service with service and environment marked](../static/img/quickstart/RunningServiceWithServiceAndEnvironment.png)

点击 "title "栏中的 "Notification Service Prod "链接，您将看到我们所说的 "**特定实体 "页面**。 该页面允许您查看特定实体的完整详细信息和依赖关系图。

![Developer Portal Running Service specific entity page after relation](../static/img/quickstart/runningServiceSpecificEntityPageAfterRelation.png)

您还可以点击 "图表 "按钮，查看所创建实体的可视化视图以及它们之间的联系: 

![Developer Portal Running Service specific entity page graph view](../static/img/quickstart/runningServiceSpecificEntityPageGraphView.png)

:::info 在我们的例子中，运行中服务的特定实体页面也会显示一个标签页，其中包含运行中服务所属的**微服务**，以及另一个包含运行中服务的**环境**的标签页，因为这是我们映射的关系。

:::

请继续浏览特定实体页面以及环境、服务和正在运行的服务页面。 请注意 Port 表格部件右上方的 "过滤"、"隐藏"、"排序 "和 "分组 "控件。

您还可以使用 Port 的 API 对蓝图和实体发出 GET 请求，下面是一个获取我们在本教程中创建的正在运行的服务实体的代码示例: 

<details>
<summary>Get the running service Entity</summary>

请注意，本示例假定令牌保存在 `access_token` 变量中。

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests
import json
import requests

# the access_token variable should already have the token from the previous example

API_URL = 'https://api.getport.io/v1'

headers = {
    'Authorization': f'Bearer {access_token}'
}

running_service_blueprint_identifier = 'runningService'

running_service_entity_identifier = 'notification-service-prod'

running_service_response = requests.get(
    f'{API_URL}/blueprints/{running_service_blueprint_identifier}/entities/{running_service_entity_identifier}', headers=headers)

print(json.dumps(running_service_response.json(), indent=2))
```

</details>

此外，您还可以使用 Port 的 API 搜索蓝图和实体。

#What now?

恭喜！您刚刚在 Port 中建立了第一个环境模型！🎉🚢

本快速入门手册被用来向您传授 Provider 提供的基本构件。 现在，您已经掌握了开始编目和跟踪环境所需的所有工具！

您可以开始创建描述 "服务"、"应用程序"、"集群 "和 "基础设施资源 "的蓝图。

:::tip  重用还是重启？ 请记住，您在这里创建的蓝图、实体和关系被用作基本示例，但 Port 总是允许您返回并编辑它们，直到它们与您要编目的基础架构相匹配。

如果你想做一些完全不同的事情，你可以直接删除在这里创建的内容，然后开始按照你想要的方式绘制实体。

:::

### 建议采取的下一步措施

:::tip 这些建议展示了创建自己的开发人员门户网站的基本步骤，如果您想在开始开发人员门户网站之旅之前了解更多有关 Port 的信息，请访问[Diving deeper](#diving-deeper) 或[Using the API](#using-the-api) 。

:::

1. 为软件和基础设施组件创建蓝图；
2. 绘制蓝图之间的关系图；
3. 通过 Port 的用户界面或使用我们的 API，根据您的蓝图创建实体，从而将数据引用到您的目录中；
4. 定义可被您和您的开发人员引用的自助服务操作；
5. 使用我们的完整用例之一，全面设置您的软件目录。

### 深入下潜

如果您想进一步了解 Port 在特定领域的能力，可以查看这些资源: 

* * [Build your software catalog](./build-your-software-catalog/build-your-software-catalog.md)
* [Define your data model](./build-your-software-catalog/define-your-data-model/define-your-data-model.md)
* [Setup blueprints](./build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md)
* [Relate blueprints](./build-your-software-catalog/define-your-data-model/relate-blueprints/relate-blueprints.md)
* [Ingest data to catalog](./build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md)
* [Create self-service experiences](./create-self-service-experiences/create-self-service-experiences.md)

### 使用应用程序接口

如果您想继续使用 Port 的 REST API，请查看这些资源: 

* * [API guide](./build-your-software-catalog/sync-data-to-catalog/api/api.md)
* [API Reference](./api-reference/api-reference.mdx)
