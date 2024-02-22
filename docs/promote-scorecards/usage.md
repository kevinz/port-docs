---
sidebar_position: 1
sidebar_label: Usage

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 如何使用记分卡

## 创建记分卡

记分卡可以通过三种方法创建: 

* 用户界面
* 应用程序接口
* Terraform

<!-- TODO: fix this back to some actual blueprint -->

:::info 记分卡是根据蓝图创建的。因此，如果您还没有在[Quickstart](../quickstart.md#service-blueprint) 中创建 "microservice "蓝图，请务必先创建该蓝图。

:::

#### 从用户界面

要从用户界面创建记分卡，请转到 DevPortal 创建器页面，然后单击 "microservice "蓝图上的 3 个圆点图标。

将打开一个编辑器窗口，其中包含已定义记分卡的当前 JSON 数组。 由于蓝图上目前未配置记分卡，因此 "记分卡 "数组将为空。 在编辑器中粘贴以下内容，即可创建本例中的记分卡: 

```json showLineNumbers
[
  {
    "identifier": "Ownership",
    "title": "Ownership",
    "rules": [
      {
        "identifier": "hasSlackChannel",
        "title": "Has Slack Channel",
        "level": "Silver",
        "query": {
          "combinator": "and",
          "conditions": [
            {
              "operator": "isNotEmpty",
              "property": "slackChannel"
            }
          ]
        }
      },
      {
        "identifier": "hasTeam",
        "title": "Has Team",
        "level": "Bronze",
        "query": {
          "combinator": "and",
          "conditions": [
            {
              "operator": "isNotEmpty",
              "property": "$team"
            }
          ]
        }
      }
    ]
  }
]
```

### 从应用程序接口

:::note 请记住，必须有访问令牌才能进行 API 请求。 如果需要生成新令牌，请参阅[Getting an API token](../build-your-software-catalog/sync-data-to-catalog/api/api.md#get-api-token) 。

:::

要通过 API 创建记分卡，您需要向 URL `https://api.getport.io/v1/blueprints/{blueprint_identifier}/scorecards` 发送 PUT 请求。

下面是一些将在 "微服务 "蓝图上创建 "所有权记分卡 "的请求示例: 

<Tabs groupId="code-examples" defaultValue="python" values={[
{label: "Python", value: "python"},
{label: "Javascript", value: "javascript"}
]}>

<TabItem value="python">

```python showLineNumbers
# Dependencies to install:
# $ python -m pip install requests

# the access_token variable should already have the token from previous examples

import requests

API_URL = 'https://api.getport.io/v1'

blueprint_name = 'microservice'

scorecards = [
  {
    'identifier': 'Ownership',
    'title': 'Ownership',
    'rules': [
      {
        'identifier': 'hasSlackChannel',
        'title': 'Has Slack Channel',
        'level': 'Silver',
        'query': {
          'combinator': 'and',
          'conditions': [{'operator': 'isNotEmpty', 'property': 'slackChannel'}]
        }
      },
      {
        'identifier': 'hasTeam',
        'title': 'Has Team',
        'level': 'Bronze',
        'query': {
          'combinator': 'and',
          'conditions': [{'operator': 'isNotEmpty', 'property': '$team'}]
        }
      }
    ]
  }
]

headers = {
    'Authorization': f'Bearer {access_token}'
}

response = requests.put(f'{API_URL}/blueprints/{blueprint_name}/scorecards', json=scorecards, headers=headers)
```

</TabItem>

<TabItem value="javascript">

```javascript showLineNumbers
// Dependencies to install:
// $ npm install axios --save

// the accessToken variable should already have the token from previous examples

const axios = require("axios").default;

const API_URL = "https://api.getport.io/v1";

const blueprintName = "microservice";

const scorecards = [
  {
    identifier: "Ownership",
    title: "Ownership",
    rules: [
      {
        identifier: "hasSlackChannel",
        title: "Has Slack Channel",
        level: "Bronze",
        query: {
          combinator: "and",
          conditions: [
            {
              operator: "isNotEmpty",
              property: "slackChannel",
            },
          ],
        },
      },
      {
        identifier: "hasTeam",
        title: "Has Team",
        level: "Silver",
        query: {
          combinator: "and",
          conditions: [
            {
              operator: "isNotEmpty",
              property: "$team",
            },
          ],
        },
      },
    ],
  },
];

const config = {
  headers: {
    Authorization: `Bearer ${accessToken}`,
  },
};

const response = await axios.put(
  `${API_URL}/blueprints/${blueprintName}/scorecards`,
  scorecards,
  config
);
```

</TabItem>

</Tabs>

创建记分卡后，您将在每个蓝图实体的 "概况实体 "页面中看到一个新标签，显示各种记分卡级别。

例如，我们可以[create the entity below](../build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#creating-entities)

```json showLineNumbers
{
  "identifier": "cart-service",
  "title": "Cart Service",
  "icon": "Microservice",
  "properties": {
    "slackChannel": "https://slack.com",
    "repoUrl": "https://github.com"
  },
  "relations": {}
}
```

然后在记分卡选项卡上查看该实体的[specific page](https://app.getport.io/MicroserviceEntity?identifier=cart-service&amp;activeTab=3) 

![Developer Portal Scorecards Tab](../../static/img/software-catalog/scorecard/tutorial/ScorecardsTab.png)

我们可以看到，"hasSlackChannel "规则通过了，因为我们为该实体提供了一个，而 "hasTeam "规则失败了，因为我们没有提供任何团队。

因此，该实体的级别为 "青铜"，因为它通过了 "青铜 "级别的所有规则(具有松弛通道)

### 来自 Terraform

要从[Terraform provider](../../build-your-software-catalog/sync-data-to-catalog/iac/terraform/) 创建记分卡，需要使用 `port_scorecard` 资源。

下面是如何使用 Terraform Provider 创建 "所有权记分卡 "的示例: 

```hcl showLineNumbers
resource "port_scorecard" "ownership" {
  blueprint = "microservice"
  identifier = "Ownership"
  title = "Ownership"
  rules = [
    {
      identifier = "hasSlackChannel"
      title = "Has Slack Channel"
      level = "Silver"
      query = {
        combinator = "and"
        conditions = [
          jsonencode({
            operator = "isNotEmpty"
            property = "slackChannel"
          })
        ]
      }
    },
    {
      identifier = "hasTeam"
      title = "Has Team"
      level = "Bronze"
      query = {
        combinator = "and"
        conditions = [
          jsonencode({
            operator = "isNotEmpty"
            property = "$team"
          })
        ]
      }
    }
  ]
}
```

## 更新记分卡

要更新记分卡，我们可以使用之前用过的 URL 和有效载荷，并加上后端为该记分卡生成的 `id` 。

正如我们在教程前面所展示的，你可以通过用户界面或应用程序接口更新记分卡。

#### 从用户界面

要从用户界面更新记分卡，请转到 DevPortal 创建器页面，展开所需的蓝图，然后切换到记分卡选项卡。

编辑器窗口将打开，显示蓝图的当前记分卡。 要更新记分卡，只需在记分卡数组中更改想要的记分卡，点击编辑器右下角的 "保存"，即可查看更新后的记分卡。

### 从应用程序接口

要更新记分卡，可以使用 2 个不同的 URL: 

1. 使用 URL `https://api.getport.io/v1/blueprints/{blueprint_identifier}/scorecards/{scorecard_identifier}` 更新单张记分卡。请求正文将是完整的记分卡和希望更改的 Values
2. 向 URL `https://api.getport.io/v1/blueprints/{blueprint_identifier}/scorecards` 发送 PUT 请求。同时更新多个记分卡

在对现有记分卡应用所需的更新后，请求正文将包括现有的记分卡正文。

### 来自 Terraform

要使用 Terraform Provider 更新记分卡，需要使用更新后的记分卡资源运行 `terraform apply -target=port_scorecard.<resourceId>` 命令。

## 删除记分卡

:::danger 记分卡删除后无法恢复！

:::

您可以通过用户界面或应用程序接口删除记分卡: 

#### 从用户界面

要通过用户界面删除记分卡，请转到 "创建器 "页面，展开蓝图并切换到 "记分卡 "选项卡。 将所需的记分卡悬停，然后从三点菜单中选择 "删除记分卡"。

### 从应用程序接口

* 发送 HTTP PUT 请求，并通过 URL `https://api.getport.io/v1/blueprints/{blueprint_identifier}/scorecards` 从记分卡数组中删除它
* 向 URL `https://api.getport.io/v1/blueprints/{blueprint_identifier}/{scorecard_identifier}` 发送 HTTP DELETE 请求，其中 `scorecard_identifier` 是我们要删除的记分卡的标识符

:::note 在使用多重更新记分卡 `https://api.getport.io/v1/blueprints/{blueprint_identifier}/scorecards` PUT 请求时，请注意您将看到一个新的 `id` 属性。该属性通过 Port 用于识别记分卡，以便更新其属性。

:::

### 来自 Terraform

要使用 Terraform Provider 删除记分卡，请使用 `terraform destroy -target=port_scorecard.<resourceId>` 命令并输入要删除的记分卡资源。(请记住，也可以从 `.tf` 文件中删除 `port_scorecard` 资源的定义并运行 `terraform apply`)
