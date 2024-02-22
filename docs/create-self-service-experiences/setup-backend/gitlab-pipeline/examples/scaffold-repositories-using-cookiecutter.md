---
sidebar_position: 2
---

# 使用 Cookiecutter 创建脚手架资源库

本示例演示了如何使用[Cookiecutter Template](https://www.cookiecutter.io/templates) 通过 Port Actions 快速搭建 Gitlab 仓库。

此外，由于 cookiecutter 是一个开源项目，您可以制作自己的项目模板，了解更多信息请访问[here](https://cookiecutter.readthedocs.io/en/2.0.2/tutorials.html#create-your-very-own-cookiecutter-project-template) 。

## 示例 - python 模板脚手架

请按照以下步骤开始使用 Python 模板: 

1. 创建以下变量作为[Gitlab Variables](https://docs.gitlab.com/ee/ci/variables/index.html) : 
    1. GITLAB_ACCESS_TOKEN` -[Personal Access Token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) ，作用域如下:   
    `api`, `read_api`, `read_repository`, `write_repository`.
    2. `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    3. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token).
       <br/>
2.在 Gitlab 组中创建名为 `python_scaffolder` 的 Gitlab 项目，并配置[Pipeline Trigger Token](https://docs.gitlab.com/ee/ci/triggers/index.html) 。

:::note 您可以被用于任何您喜欢的名称，只要确保在 Port Action 中正确配置即可。

:::

<br/>

3.按照我们的指南[here](../Installation) 安装 Port 的 Gitlab 代理。

:::note 确保在安装 Port 的 Gitlab 代理时被用于 Pipeline 触发令牌。

:::

<br/>

4.使用以下 JSON 定义创建 Port 蓝图: 

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

5.使用以下 JSON 定义创建 Port 操作: 

:::note 确保替换 `python_scaffolder` 中 PROJECT_NAME 和 GROUP_NAME 的占位符。

:::

<details>
  <summary>Port Action</summary>

```json showLineNumbers
[
  {
    "identifier": "scaffold_gitlab",
    "title": "Scaffold Gitlab Microservice",
    "userInputs": {
      "properties": {
        "description": {
          "title": "Description",
          "type": "string",
          "description": "Short description of your service"
        },
        "service_name": {
          "icon": "DefaultProperty",
          "title": "Service Name",
          "type": "string",
          "description": "Gitlab Project Name"
        },
        "gitlab_username": {
          "title": "Gitlab Username",
          "type": "string",
          "description": "Gitlab username which should match your GITLAB_ACCESS_TOKEN configured in the Gitlab Project Variables"
        },
        "gitlab_group": {
          "title": "Gitlab Group",
          "description": "Gitlab Group to create the project in",
          "type": "string"
        }
      },
      "required": ["service_name", "gitlab_username", "gitlab_group"],
      "order": [
        "service_name",
        "description",
        "gitlab_username",
        "gitlab_group"
      ]
    },
    "invocationMethod": {
      "type": "GITLAB",
      "omitPayload": false,
      "omitUserInputs": false,
      "projectName": "<PROJECT_NAME>",
      "groupName": "<GROUP_NAME>",
      "defaultRef": "main",
      "agent": true
    },
    "trigger": "CREATE",
    "requiredApproval": false
  }
]
```

</details>
<br/>

6.在你的 `python_scaffolder` Gitlab 项目中，在主分支的 `.gitlab-ci.yml` 下创建一个 Gitlab CI 文件，内容如下: 

<details>
<summary>Gitlab CI Script</summary>

```yml showLineNumbers
image: python:3.10.0-alpine

variables:
  COOKIECUTTER_TEMPLATE_URL: "https://gitlab.com/AdriaanRol/cookiecutter-pypackage-gitlab"
  GITLAB_API_URL: "https://gitlab.com/api/v4"
  # GITLAB_ACCESS_TOKEN: """  # This should be set in GitLab's CI/CD environment variables for security

stages: # List of stages for jobs, and their order of execution
  - fetch-port-access-token
  - scaffold
  - create-entity
  - update-run-status

fetch-port-access-token: # Example - get the Port API access token and RunId
  stage: fetch-port-access-token
  except:
    - pushes
  before_script:
    - apk update
    - apk add jq curl -q
  script:
    - |
      accessToken=$(curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"clientId": "'"$PORT_CLIENT_ID"'", "clientSecret": "'"$PORT_CLIENT_SECRET"'"}' \
        -s 'https://api.getport.io/v1/auth/access_token' | jq -r '.accessToken')
      echo "ACCESS_TOKEN=$accessToken" >> data.env
      runId=$(cat $TRIGGER_PAYLOAD | jq -r '.port_payload.context.runId')
      echo "RUN_ID=$runId" >> data.env
  artifacts:
    reports:
      dotenv: data.env

scaffold:
  before_script: |
    apk update
    apk add jq curl git -q
    pip3 install cookiecutter==2.3.0 -q
  stage: scaffold
  except:
    - pushes
  script:
    - |
      if [[ -z "$gitlab_group" ]]; then
          echo "Gitlab group name must be provided"
          exit 1
      fi

      # Create new repository on GitLab
      NAMESPACE_ID=$(curl -s --header "Private-Token: $GITLAB_ACCESS_TOKEN" "$GITLAB_API_URL/groups/$gitlab_group" | jq -r .id)
      CREATE_REPO_RESPONSE=$(curl -X POST -s "$GITLAB_API_URL/projects" --header "Private-Token: $GITLAB_ACCESS_TOKEN" --form "name=$service_name" --form "namespace_id=$NAMESPACE_ID")
      PROJECT_URL=$(echo $CREATE_REPO_RESPONSE | jq -r .http_url_to_repo)

      # Check if the repository creation was successful
      if [[ -z "$PROJECT_URL" ]]; then
          echo "Failed to create GitLab repository."
          exit 1
      fi

      FIRST_NAME=$(cat $TRIGGER_PAYLOAD | jq -r '.trigger.by.user.firstName')
      LAST_NAME=$(cat $TRIGGER_PAYLOAD | jq -r '.trigger.by.user.lastName')
      EMAIL=$(cat $TRIGGER_PAYLOAD | jq -r '.trigger.by.user.email')
      BLUEPRINT_ID=$(cat $TRIGGER_PAYLOAD | jq -r '.port_payload.context.blueprint')

      echo "PROJECT_URL=$PROJECT_URL" >> data.env
      echo "BLUEPRINT_ID=$BLUEPRINT_ID" >> data.env

      # Generate cookiecutter.yaml file
      cat <<EOF > cookiecutter.yaml
      default_context:
        full_name: "${FIRST_NAME} ${LAST_NAME}"
        email: "${EMAIL}"
        project_short_description: "${description}"
        gitlab_username: "${gitlab_username}"
        project_name: "${service_name}"
      EOF
      cookiecutter $COOKIECUTTER_TEMPLATE_URL --no-input --config-file cookiecutter.yaml --output-dir scaffold_out

      echo "Initializing new repository..."
      git config --global user.email "scaffolder@email.com"
      git config --global user.name "Mighty Scaffolder"
      git config --global init.defaultBranch "main"

      cd scaffold_out/$service_name
      git init
      git add .
      git commit -m "Initial commit"
      GITLAB_HOSTNAME=$(echo "$GITLAB_API_URL" | cut -d'/' -f3)
      git remote add origin https://$gitlab_username:$GITLAB_ACCESS_TOKEN@$GITLAB_HOSTNAME/${gitlab_group}/${service_name}.git
      git push -u origin main
  artifacts:
    reports:
      dotenv: data.env

create-entity:
  stage: create-entity
  except:
    - pushes
  before_script:
    - apk update
    - apk add jq curl -q
  script:
    - |
      curl --location --request POST "https://api.getport.io/v1/blueprints/$BLUEPRINT_ID/entities?upsert=true&run_id=$RUN_ID&create_missing_related_entities=true" \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "Content-Type: application/json" \
        -d '{"identifier": "'"$service_name"'","title": "'"$service_name"'","properties": {"description": "'"$description"'","url": "'"$PROJECT_URL"'"}, "relations": {}}'

update-run-status:
  stage: update-run-status
  except:
    - pushes
  image: curlimages/curl:latest
  script:
    - |
      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"status":"SUCCESS", "message": {"run_status": "Scaffold '"$service_name"' finished successfully!\n Project URL: '"$PROJECT_URL"'"}}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID"
```

</details>
<br/>

7.从 Port 应用程序的[Self-service](https://app.getport.io/self-serve) 标签触发操作。
