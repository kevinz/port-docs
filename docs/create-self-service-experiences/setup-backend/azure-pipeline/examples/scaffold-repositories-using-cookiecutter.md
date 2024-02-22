---

sidebar_position: 2

---

# 使用 Cookiecutter 创建脚手架资源库

本示例演示了如何使用[Cookiecutter Template](https://www.cookiecutter.io/templates) 通过 Port Actions 快速搭建 Azure DevOps 资源库。

此外，由于 cookiecutter 是一个开源项目，您可以制作自己的项目模板，了解更多信息请访问[here](https://cookiecutter.readthedocs.io/en/2.0.2/tutorials.html#create-your-very-own-cookiecutter-project-template) 。

## 示例 - python 模板脚手架

请按照以下步骤开始使用 Python 模板: 

1. 在 Azure Devops 组织/项目中创建名为 "python_scaffolder "的 Azure DevOps 资源库，并配置[Service Connection](/create-self-service-experiences/setup-backend/azure-pipeline#define-incoming-webhook-in-azure) 。

:::note 配置 Webhook 名称 "和 "服务连接名称 "时，请将 "port_trigger "用于 "Webhook 名称 "和 "服务连接名称"。[Service Connection](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&amp;tabs=yaml)

:::

<br/>

2.使用以下 JSON 定义创建 Port 蓝图: 

:::note 请记住，这可以是你想要的任何蓝图，这只是一个例子。

:::

<details>
  <summary>Port Microservice Blueprint</summary>

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
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>
<br/>

3.使用以下 JSON 定义创建 Port 操作: 

:::note 确保将 AZURE_DEVOPS_ORANZATION_NAME 的占位符替换为`python_scaffolder`所在的位置。 同时验证`invocationMethod.webhook`等于`port_trigger`。

:::

<details>
  <summary>Port Action</summary>

```json showLineNumbers
[
  {
    "identifier": "azure_scaffolder",
    "title": "Azure Scaffolder",
    "icon": "Azure",
    "userInputs": {
      "properties": {
        "service_name": {
          "icon": "DefaultProperty",
          "title": "Service Name",
          "type": "string",
          "description": "Name of the service to scaffold"
        },
        "azure_organization": {
          "icon": "DefaultProperty",
          "title": "Azure Organization",
          "type": "string",
          "description": "Your Azure DevOps organization name"
        },
        "azure_project": {
          "icon": "DefaultProperty",
          "title": "Azure Project",
          "type": "string",
          "description": "Your Azure DevOps project name"
        },
        "description": {
          "icon": "DefaultProperty",
          "title": "Description",
          "type": "string",
          "description": "Service description"
        }
      },
      "required": ["service_name"],
      "order": [
        "service_name",
        "azure_organization",
        "azure_project",
        "description"
      ]
    },
    "invocationMethod": {
      "type": "AZURE-DEVOPS",
      "webhook": "port_trigger",
      "org": "<AZURE_DEVOPS_ORANZATION_NAME>"
    },
    "trigger": "CREATE",
    "requiredApproval": false
  }
]
```

</details>
<br/>

4.在您的 `python_scaffolder` Azure DevOps Repository 中，在 repo 主分支根中的 `azure-pipelines.yml` 下创建一个 Azure Pipelines 文件，内容如下: 

<details>
<summary>Azure DevOps Pipeline Script</summary>

```yml showLineNumbers
trigger: none

pool:
  vmImage: "ubuntu-latest"

variables:
  RUN_ID: "${{ parameters.port_trigger.context.runId }}"
  BLUEPRINT_ID: "${{ parameters.port_trigger.context.blueprint }}"
  SERVICE_NAME: "${{ parameters.port_trigger.payload.properties.service_name }}"
  DESCRIPTION: "${{ parameters.port_trigger.payload.properties.description }}"
  AZURE_ORGANIZATION: "${{ parameters.port_trigger.payload.properties.azure_organization }}"
  AZURE_PROJECT: "${{ parameters.port_trigger.payload.properties.azure_project }}"
  # PERSONAL_ACCESS_TOKEN: $(PERSONAL_ACCESS_TOKEN) // set up PERSONAL_ACCESS_TOKEN as a sercet variable

resources:
  webhooks:
    - webhook: port_trigger
      connection: port_trigger

stages:
  - stage: fetch_port_access_token
    jobs:
      - job: fetch_port_access_token
        steps:
          - script: |
              sudo apt-get update
              sudo apt-get install -y jq
          - script: |
              accessToken=$(curl -X POST \
                    -H 'Content-Type: application/json' \
                    -d '{"clientId": "$(PORT_CLIENT_ID)", "clientSecret": "$(PORT_CLIENT_SECRET)"}' \
                    -s 'https://api.getport.io/v1/auth/access_token' | jq -r '.accessToken')
              echo "##vso[task.setvariable variable=accessToken;isOutput=true]$accessToken"
            displayName: Fetch Access Token and Run ID
            name: getToken

  - stage: scaffold
    dependsOn:
      - fetch_port_access_token
    jobs:
      - job: scaffold
        variables:
          COOKIECUTTER_TEMPLATE_URL: "https://github.com/brettcannon/python-azure-web-app-cookiecutter"
        steps:
          - script: |
              sudo apt-get update
              sudo apt-get install -y jq
              sudo pip install cookiecutter -q
          - script: |
              # Create the repository
              PROJECT_ID=$(curl -X GET "https://dev.azure.com/${{ variables.AZURE_ORGANIZATION }}/_apis/projects/${{ variables.AZURE_PROJECT }}?api-version=7.0" \
              -H "Authorization: Basic $(PERSONAL_ACCESS_TOKEN)" \
              -H "Content-Type: application/json" \
              -H "Content-Length: 0" | jq .id)

              if [[ -z "$PROJECT_ID" ]]; then
                echo "Failed to fetch Azure Devops Project ID."
                exit 1
              fi

              PAYLOAD='{"name":"${{ variables.SERVICE_NAME }}","project":{"id":'$PROJECT_ID'}}'

              CREATE_REPO_RESPONSE=$(curl -X POST "https://dev.azure.com/${{ variables.AZURE_ORGANIZATION }}/_apis/git/repositories?api-version=7.0" \
              -H "Authorization: Basic $(PERSONAL_ACCESS_TOKEN)" \
              -H "Content-Type: application/json" \
              -d $PAYLOAD)

              PROJECT_URL=$(echo $CREATE_REPO_RESPONSE | jq -r .webUrl)

              if [[ -z "$PROJECT_URL" ]]; then
                echo "Failed to create Azure Devops repository."
                exit 1
              fi

              echo "##vso[task.setvariable variable=PROJECT_URL;isOutput=true]$PROJECT_URL"

              cat <<EOF > cookiecutter.yaml
              default_context:
                site_name: "${{ variables.SERVICE_NAME }}"
                python_version: "3.6.0"
              EOF
              cookiecutter $COOKIECUTTER_TEMPLATE_URL --no-input --config-file cookiecutter.yaml --output-dir scaffold_out

              echo "Initializing new repository..."
              git config --global user.email "scaffolder@email.com"
              git config --global user.name "Mighty Scaffolder"
              git config --global init.defaultBranch "main"

              cd "scaffold_out/${{ variables.SERVICE_NAME }}"
              git init
              git add .
              git commit -m "Initial commit"
              decoded_pat=$(echo $(PERSONAL_ACCESS_TOKEN) | base64 -d)
              git remote add origin https://$decoded_pat@dev.azure.com/${{ variables.AZURE_ORGANIZATION }}/${{ variables.AZURE_PROJECT }}/_git/${{ variables.SERVICE_NAME }}
              git push -u origin --all
            name: scaffold

  - stage: upsert_entity
    dependsOn:
      - fetch_port_access_token
      - scaffold
    jobs:
      - job: upsert_entity
        variables:
          accessToken: $[ stageDependencies.fetch_port_access_token.fetch_port_access_token.outputs['getToken.accessToken'] ]
          PROJECT_URL: $[ stageDependencies.scaffold.scaffold.outputs['scaffold.PROJECT_URL'] ]
        steps:
          - script: |
              sudo apt-get update
              sudo apt-get install -y jq
          - script: |
              curl -X POST \
                -H 'Content-Type: application/json' \
                -H 'Authorization: Bearer $(accessToken)' \
                -d '{
                    "identifier": "${{ variables.SERVICE_NAME }}",
                    "title": "${{ variables.SERVICE_NAME }}",
                    "properties": {"description":"${{ variables.DESCRIPTION }}","url":"$(PROJECT_URL)" },
                    "relations": {}
                  }' \
                "https://api.getport.io/v1/blueprints/${{ variables.BLUEPRINT_ID }}/entities?upsert=true&run_id=${{ variables.RUN_ID }}&create_missing_related_entities=true"

  - stage: update_run_status
    dependsOn:
      - upsert_entity
      - fetch_port_access_token
      - scaffold
    jobs:
      - job: update_run_status
        variables:
          accessToken: $[ stageDependencies.fetch_port_access_token.fetch_port_access_token.outputs['getToken.accessToken'] ]
          PROJECT_URL: $[ stageDependencies.scaffold.scaffold.outputs['scaffold.PROJECT_URL'] ]
        steps:
          - script: |
              sudo apt-get update
              sudo apt-get install -y jq
          - script: |
              curl -X PATCH \
                -H 'Content-Type: application/json' \
                -H 'Authorization: Bearer $(accessToken)' \
                -d '{"status":"SUCCESS", "message": {"run_status": "Scaffold ${{ variables.SERVICE_NAME }} finished successfully!\\n Project URL: $(PROJECT_URL)" }}' \
                "https://api.getport.io/v1/actions/runs/${{ variables.RUN_ID }}"

  - stage: update_run_status_failed
    dependsOn:
      - upsert_entity
      - fetch_port_access_token
      - scaffold
    condition: failed()
    jobs:
      - job: update_run_status_failed
        variables:
          accessToken: $[ stageDependencies.fetch_port_access_token.fetch_port_access_token.outputs['getToken.accessToken'] ]
        steps:
          - script: |
              curl -X PATCH \
                -H 'Content-Type: application/json' \
                -H 'Authorization: Bearer $(accessToken)' \
                -d '{"status":"FAILURE", "message": {"run_status": "Scaffold ${{ variables.SERVICE_NAME }} failed" }}' \
                "https://api.getport.io/v1/actions/runs/${{ variables.RUN_ID }}"
```

</details>
<br/>

5.要在项目中配置 Pipelines，请转到 Pipelines -> Create Pipeline -> Azure Repos Git，然后选择 "python_scaffolder"，点击保存(在 "运行 "下拉菜单中)。
   <br/>
6.将以下变量创建为[Secret Variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/set-secret-variables?view=azure-devops&amp;tabs=yaml%2Cbash) : 
    1. PERSONAL_ACCESS_TOKEN` - base64 编码的[Personal Access Token](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&amp;tabs=Windows) ，其作用域如下: 
        + 代码: Full.
        + 发布: 读取、写入和执行。
        使用以下脚本对您的 PAT 进行编码: 


      ```bash
      MY_PAT=YourPAT
      B64_PAT=$(printf ":%s" "$MY_PAT" | base64)
      echo $B64_PAT
      ```


2.`PORT_CLIENT_ID` - Port客户 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token).
3.`PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token).

<br/>

7.从 Port 应用程序的[Self-service](https://app.getport.io/self-serve) 标签触发操作。
