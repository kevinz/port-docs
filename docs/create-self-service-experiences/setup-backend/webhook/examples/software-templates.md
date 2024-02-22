---
sidebar_position: 4
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 软件模板

软件模板允许您生成新资源(如服务)的定制骨架，通常以社区最佳实践为基础。

有一些开源项目可以让你根据软件模板创建项目，如[Cookiecutter](https://github.com/cookiecutter/cookiecutter) 。

下一节我们将举例说明。

:::tip 本指南的所有相关文件和资源可查阅[**Here for GitHub**](https://github.com/port-labs/port-cookiecutter-example) ，以及[**Here for GitLab**](https://github.com/port-labs/port-cookiecutter-gitlab-example) 。

:::

## 示例 - 创建新的服务存储库

下面的示例利用 Port[webhook-actions](../webhook.md) 从软件模板创建新的服务资源库。

首先，您需要创建一个简单的 `Service` 蓝图。

<details>
<summary>Service blueprint JSON</summary>

```json showLineNumbers
{
  "identifier": "service",
  "title": "Service",
  "icon": "Service",
  "schema": {
    "properties": {
      "description": {
        "type": "string",
        "title": "Description"
      },
      "url": {
        "type": "string",
        "format": "url",
        "title": "URL"
      }
    },
    "required": []
  },
  "mirrorProperties": {}
}
```

</details>

然后，在蓝图中添加 "创建 "自助服务操作，以支持从不同框架创建多个服务。

在本例中，我们添加了提供[Django](https://github.com/cookiecutter/cookiecutter-django) 、[C++](https://github.com/DerThorsten/cpp_cookiecutter) 和[Go](https://github.com/lacion/cookiecutter-golang) 服务的操作。

该操作将接收以下用户输入: 

<Tabs groupId="actions" defaultValue="github" values={[
{label: "GitHub", value: "github"},
{label: "GitLab", value: "gitlab"}
]}>

<TabItem value="github">

* 托管已创建服务项目的 GitHub 组织和存储库；
* 模板特定参数，如 `project_name` 和 `description`。

:::note 在下面的 JSON 中，您需要用您的 URL 替换 `<WEBHOOK_URL>` 占位符。

有关本地设置，请参阅[example](../local-debugging-webhook.md#creating-the-vm-create-action) 。

:::

<details>
<summary>Self-Service Actions JSON</summary>

```json showLineNumbers
[
  {
    "identifier": "CreateDjangoService",
    "title": "Create Django",
    "icon": "Service",
    "userInputs": {
      "properties": {
        "github_organization": {
          "type": "string"
        },
        "github_repository": {
          "type": "string"
        },
        "project_name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        }
      },
      "required": ["github_organization", "github_repository"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Creates a new Django service"
  },
  {
    "identifier": "CreateCPPService",
    "title": "Create C++",
    "icon": "Service",
    "userInputs": {
      "properties": {
        "github_organization": {
          "type": "string"
        },
        "github_repository": {
          "type": "string"
        },
        "project_name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        }
      },
      "required": ["github_organization", "github_repository"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Creates a new C++ service"
  },
  {
    "identifier": "CreateGoService",
    "title": "Create Go",
    "icon": "Service",
    "userInputs": {
      "properties": {
        "github_organization": {
          "type": "string"
        },
        "github_repository": {
          "type": "string"
        },
        "app_name": {
          "type": "string"
        },
        "project_short_description": {
          "type": "string"
        }
      },
      "required": ["github_organization", "github_repository"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Creates a new Go service"
  }
]
```

</details>

</TabItem>

<TabItem value="gitlab">

* 用于托管已创建服务项目的存储库；
* 模板特定参数，如 `project_name` 和 `description`。

:::note 在下面的 JSON 中，您需要用您的 URL 替换 `<WEBHOOK_URL>` 占位符。

有关本地设置，请参阅[example](../local-debugging-webhook.md#creating-the-vm-create-action) 。

:::

<details>
<summary>Self-Service Actions JSON</summary>

```json showLineNumbers
[
  {
    "identifier": "CreateDjangoService",
    "title": "Create Django",
    "icon": "Service",
    "userInputs": {
      "properties": {
        "repository_name": {
          "type": "string"
        },
        "project_name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        }
      },
      "required": ["repository_name"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Creates a new Django service"
  },
  {
    "identifier": "CreateCPPService",
    "title": "Create C++",
    "icon": "Service",
    "userInputs": {
      "properties": {
        "repository_name": {
          "type": "string"
        },
        "project_name": {
          "type": "string"
        },
        "description": {
          "type": "string"
        }
      },
      "required": ["repository_name"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Creates a new C++ service"
  },
  {
    "identifier": "CreateGoService",
    "title": "Create Go",
    "icon": "Service",
    "userInputs": {
      "properties": {
        "repository_name": {
          "type": "string"
        },
        "app_name": {
          "type": "string"
        },
        "project_short_description": {
          "type": "string"
        }
      },
      "required": ["repository_name"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Creates a new Go service"
  }
]
```

</details>

</TabItem>

</Tabs>

接下来，为了监听 webhook 事件，需要设置一个简单的后端。

在后端，您将根据 Cookiecutter 模板(使用提供的用户参数)生成项目，并将其推送到您指定的 GitHub 仓库。

完整的后端示例可在[here for GitHub](https://github.com/port-labs/port-cookiecutter-example) 或[here for GitLab](https://github.com/port-labs/port-cookiecutter-gitlab-example) 上找到。

:::info 上述示例还在 Port 中创建了一个新的服务实体，并更新了操作运行详情。

强烈建议采取这些步骤，以便长期跟踪运行的操作、其日志和产生的资源。

:::

就这样！现在就可以使用被引用的操作了，如图所示: 

![create-service.png](../../../../../static/img/complete-use-cases/software-templates/create-service.png)

## 摘要

为了在保证高质量的同时保持开发的高速度，软件模板是非常有用的。

使用 Port 自助操作，您可以方便地从公共或私人模板中创建和记录新项目。