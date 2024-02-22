---
sidebar_position: 2
---

# 创建 GitHub secret

此示例演示了如何通过 Port Actions 在 GitHub 仓库中创建[GitHub Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) 。

在本例中，我们从 GitHub Marketplace 引用了一个预定义的 GitHub 动作，名为[Create GitHub Secret Action](https://github.com/marketplace/actions/create-github-secret-action) 。

## 示例 - 创建 GitHub secret

请按照以下步骤开始操作: 

1. 创建以下 GitHub Action secrets: 
    1. PERSONAL_ACCESS_TOKEN` -[Classic Personal Access Token](https://github.com/settings/tokens) ，其作用域如下: 
        ![Token Scopes](../../../../../static/img/self-service-actions/setup-backend/github-workflow/pat-scopes.png)
    2. `PORT_CLIENT_ID` - Port 客户 ID[learn more](../../../../build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    3. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](../../../../build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
2.点击[here](https://github.com/apps/getport-io/installations/new) 安装 Port 的 GitHub 应用程序。
3.创建具有以下属性的 Port 蓝图: 

:::note 请记住，这可以是你想要的任何蓝图，这只是一个例子。

:::

```json showLineNumbers
{
  "identifier": "githubsecret",
  "title": "GitHubSecret",
  "icon": "Github",
  "schema": {
    "properties": {
      "secret_key": {
        "icon": "DefaultProperty",
        "title": "Secret Key",
        "type": "string",
        "description": "All Uppercase",
        "pattern": "^[^a-z]*$"
      },
      "secret_value": {
        "icon": "DefaultProperty",
        "title": "Secret Value",
        "type": "string"
      }
    },
    "required": ["secret_key", "secret_value"]
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

4.使用以下 JSON 定义创建 Port 操作: 

:::note 确保替换 Port Action 中 GITHUB_ORG_NAME 和 GITHUB_REPO_NAME 的占位符，以匹配 GitHub 环境。

:::

```json showLineNumbers
[
  {
    "identifier": "create_github_secret",
    "title": "Create GitHub Secret",
    "icon": "Github",
    "userInputs": {
      "properties": {
        "secret_key": {
          "icon": "DefaultProperty",
          "title": "Secret Key",
          "type": "string",
          "pattern": "^[^a-z]*$"
        },
        "secret_value": {
          "icon": "DefaultProperty",
          "title": "Secret Value",
          "type": "string",
          "encryption": "aes256-gcm"
        }
      },
      "required": ["secret_key", "secret_value"],
      "order": ["secret_key", "secret_value"]
    },
    "invocationMethod": {
      "type": "GITHUB",
      "omitPayload": false,
      "omitUserInputs": false,
      "reportWorkflowStatus": true,
      "org": "<GITHUB_ORG_NAME>",
      "repo": "<GITHUB_REPO_NAME>",
      "workflow": "create-repo-secret.yml"
    },
    "trigger": "CREATE",
    "description": "Creates a GitHub secret in my repository",
    "requiredApproval": false
  }
]
```

5.在`.github/workflows/create-repo-secret.yml`下创建一个工作流文件，内容如下: 

```yml showLineNumbers
name: Create Repository Secret

on:
  workflow_dispatch:
    inputs:
      secret_key:
        type: string
        description: Name of the secret's key
      secret_value:
        type: string
        description: value of the secret
      port_payload:
        required: false
        description:
          Port's payload, including details for who triggered the action and
          general context (blueprint, run id, etc...)
        type: string

jobs:
  create_secret:
    runs-on: ubuntu-latest
    steps:
      - uses: gliech/create-github-secret-action@v1
        with:
          name: ${{ inputs.secret_key }}
          value: ${{ inputs.secret_value }}
          pa_token: ${{ secrets.PERSONAL_ACCESS_TOKEN }}

      - name: UPSERT Entity
        uses: port-labs/port-github-action@v1
        with:
          identifier: ${{ inputs.secret_key }}
          title: ${{ inputs.secret_key }}
          team: "[]"
          icon: DefaultBlueprint
          blueprint: ${{ fromJson(inputs.port_payload).context.blueprint }}
          properties: |-
            {
              "secret_key": "${{ inputs.secret_key }}",
              "secret_value": "${{ inputs.secret_value }}"
            }
          relations: "{}"
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: UPSERT
          runId: ${{ fromJson(inputs.port_payload).context.runId }}
```

6.从 Port 应用程序的[Self-service](https://app.getport.io/self-serve) 标签触发操作。
