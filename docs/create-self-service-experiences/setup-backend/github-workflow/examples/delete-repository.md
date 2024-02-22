---
sidebar_position: 4
---

import PortTooltip from "/src/components/tooltip/tooltip.jsx";

# 删除 GitHub 仓库

在以下指南中，您将在 Port 中创建一个自助服务操作，执行 GitHub 工作流，[delete a GitHub repository](https://docs.github.com/en/rest/repos/repos#delete-a-repository) 。

## 先决条件

1. 点击[here](https://github.com/apps/getport-io/installations/new) 安装 Port 的 GitHub 应用程序。
2. 本指南假定存在代表您的版本库的蓝图。如果您还没有这样做，请先参考[guide](/build-your-software-catalog/sync-data-to-catalog/git/github/examples#mapping-repositories-file-contents-and-pull-requests) 开始 GitHub 数据模型的设置。
3. 包含action资源(即 Github 工作流文件)的资源库。

## Guide

请按照以下步骤开始操作: 

1. 创建以下 GitHub 操作secret: 
    - 创建以下 Port 凭据: 
        + `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token).
        + `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    - `GH_TOKEN` - 具有以下作用域的[Classic Personal Access Token](https://github.com/settings/tokens) : `repo` 和 `delete_repo

<br />
2. Create a Port action in the [self-service page](https://app.getport.io/self-serve) or with the following JSON definition:

<details>

  <summary>Port Action: Delete GitHub Repository</summary>
   :::tip
- `<GITHUB-ORG>` - your GitHub organization or user name.
- `<GITHUB-REPO-NAME>` - your GitHub repository name.

:::

```json showLineNumbers
[
  {
    "identifier": "delete_repo",
    "title": "Delete Repo",
    "icon": "Github",
    "userInputs": {
      "properties": {
        "org_name": {
          "icon": "Github",
          "title": "Organisation Name",
          "type": "string",
          "default": "default-org"
        },
        "delete_dependents": {
          "icon": "Github",
          "title": "Delete Dependent Items",
          "type": "boolean",
          "default": false
        }
      },
      "required": [],
      "order": [
        "org_name",
        "delete_dependents"
      ]
    },
    "invocationMethod": {
      "type": "GITHUB",
      "org": "<GITHUB-ORG>",
      "repo": "<GITHUB-REPO-NAME>",
      "workflow": "delete-repo.yml",
      "omitUserInputs": false,
      "omitPayload": false,
      "reportWorkflowStatus": true
    },
    "trigger": "DELETE",
    "description": "A github action that deletes a github repo",
    "requiredApproval": false
  }
]
```

</details>
<br />

3.在`.github/workflows/delete-repo.yml`下创建一个工作流程文件，内容如下: 

<details>

<summary>GitHub workflow script</summary>

```yaml showLineNumbers title="delete-repo.yml"
name: Delete Repository

on:
  workflow_dispatch:
    inputs:
      org_name:
        required: true
        type: string
      delete_dependents:
        required: false
        type: boolean
        default: false
      port_payload:
        required: true
        type: string
jobs:
  delete-repo:
    runs-on: ubuntu-latest

    steps:
      - name: Inform starting of deletion
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: PATCH_RUN
          runId: ${{ fromJson(inputs.port_payload).context.runId }}
          logMessage: |
            Deleting a github repository... ⛴️

      - name: Delete Repository
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          REPO_NAME: ${{ fromJson(inputs.port_payload).context.entity }}
        run: |
          echo $GH_TOKEN
          HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
            -X DELETE \
            -H "Accept: application/vnd.github+json" \
            -H "Authorization: Bearer $GH_TOKEN" \
            "https://api.github.com/repos/${{ inputs.org_name }}/$REPO_NAME")

          echo "HTTP Status: $HTTP_STATUS"

          # Check if HTTP_STATUS is 204 (No Content)
          if [ $HTTP_STATUS -eq 204 ]; then
            echo "Repository deleted successfully."
            echo "delete_successful=true" >> $GITHUB_ENV
          else
            echo "Failed to delete repository. HTTP Status: $HTTP_STATUS"
            echo "delete_successful=false" >> $GITHUB_ENV
          fi

      - name: Delete record in Port
        if: ${{ env.delete_successful == 'true' }}
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: DELETE
          delete_dependents: ${{ inputs.delete_dependents }}
          identifier: ${{ fromJson(inputs.port_payload).context.entity }}
          blueprint: ${{ fromJson(inputs.port_payload).context.blueprint }}

      - name: Inform completion of deletion
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: PATCH_RUN
          runId: ${{ fromJson(inputs.port_payload).context.runId }}
          logMessage: |
            GitHub repository deleted! ✅
```

</details>
<br />
4. Trigger the action from the [self-service](https://app.getport.io/self-serve) page of your Port application.