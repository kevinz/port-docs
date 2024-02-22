---

sidebar_position: 2

---

# 使用 Cookiecutter 创建脚手架资源库

[This GitHub action](https://github.com/port-labs/cookiecutter-gha) 可让您通过 Port Actions 使用任何选定的[Cookiecutter Template](https://www.cookiecutter.io/templates) 快速构建软件源。

此外，由于 cookiecutter 是一个开源项目，您可以制作自己的项目模板，了解更多信息请访问[here](https://cookiecutter.readthedocs.io/en/2.0.2/tutorials.html#create-your-very-own-cookiecutter-project-template) 。

## 示例 - golang 模板脚手架

请按照以下步骤开始使用 Golang 模板: 

1. 创建以下 GitHub 操作secret: 
    1. `ORG_TOKEN` - 具有创建版本库权限的[fine-grained PAT](https://github.com/settings/tokens?type=beta) 。
    2. `PORT_CLIENT_ID` - Port客户端 ID[learn more](../../../../build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    3. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](../../../../build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
2.点击[here](https://github.com/apps/getport-io/installations/new) 安装 Port 的 GitHub 应用程序。
3.创建具有以下属性的 Port 蓝图: 

:::note 请记住，这可以是你想要的任何蓝图，这只是一个例子。

:::

```json showLineNumbers
{
  "identifier": "microservice",
  "title": "Microservice",
  "icon": "Microservice",
  "schema": {
    "properties": {
      "description": {
        "title": "Description",
        "type": "string"
      },
      "url": {
        "title": "URL",
        "format": "url",
        "type": "string"
      },
      "readme": {
        "title": "README",
        "type": "string",
        "format": "markdown",
        "icon": "Book"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

4.使用以下 JSON 定义创建 Port 操作: 

:::note 请记住，任何以 `cookiecutter_` 开头的输入都会自动作为变量注入 cookiecutter 操作中。我们正在使用[Golang Template](https://github.com/lacion/cookiecutter-golang) 的 `cookiecutter_app_name` 输入。

:::

```json showLineNumbers
[
  {
    "identifier": "scaffold",
    "title": "Scaffold Golang Microservice",
    "icon": "Go",
    "userInputs": {
      "properties": {
        "name": {
          "title": "Repo Name",
          "type": "string"
        },
        "cookiecutter_app_name": {
          "type": "string",
          "title": "Application Name"
        }
      },
      "required": ["name"]
    },
    "invocationMethod": {
      "type": "GITHUB",
      "org": "port-cookiecutter-example",
      "repo": "gha-templater",
      "workflow": "scaffold-golang.yml",
      "omitUserInputs": true
    },
    "trigger": "CREATE",
    "description": "Scaffold a new Microservice from a Cookiecutter teplate"
  }
]
```

5.在`.github/workflows/scaffold-golang.yml`下创建一个工作流程文件，内容如下: 

```yml showLineNumbers
on:
  workflow_dispatch:
    inputs:
      port_payload:
        required: true
        description: "Port's payload, including details for who triggered the action and general context (blueprint, run id, etc...)"
        type: string
    secrets:
      ORG_TOKEN:
        required: true
      PORT_CLIENT_ID:
        required: true
      PORT_CLIENT_SECRET:
        required: true
jobs:
  scaffold:
    runs-on: ubuntu-latest
    steps:
      - uses: port-labs/cookiecutter-gha@v1.1
        with:
          portClientId: ${{ secrets.PORT_CLIENT_ID }}
          portClientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          token: ${{ secrets.ORG_TOKEN }}
          portRunId: ${{ fromJson(inputs.port_payload).context.runId }}
          repositoryName: ${{ fromJson(inputs.port_payload).payload.properties.name }}
          portUserInputs: ${{ fromJson(inputs.port_payload).payload.properties }}
          cookiecutterTemplate: https://github.com/lacion/cookiecutter-golang
          blueprintIdentifier: "microservice"
          organizationName: INSERT_ORG_NAME
```

6.从 Port 的用户界面触发操作。

![gif](../../../../../static/img/self-service-actions/ScaffoldGolang.gif)

## 下一步

* [Connect Port's GitHub exporter](../../../../build-your-software-catalog/sync-data-to-catalog/git/github/github.md) 以确保从 GitHub 自动获取所有属性(如 URL、readme 等)。
    - 您可以了解如何设置 Port 的 GitHub 输出程序[here](../../../../build-your-software-catalog/sync-data-to-catalog/git/github/github.md#ingesting-git-objects) 。
    - 您可以查看示例配置和用例[here](../../../../build-your-software-catalog/sync-data-to-catalog/git/github/examples.md) 。
