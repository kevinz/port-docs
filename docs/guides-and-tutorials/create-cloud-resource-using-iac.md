---
sidebar_position: 3
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 用 IaC 创建云资源

本指南只需 8 分钟即可完成，旨在展示以下内容: 

* 使用 IaC 创建资源的完整流程。
* 从自助操作后端与 Port 进行通信的简便性。

:::tip  先决条件

* 本指南假定您已拥有 Port 账户，并已完成[onboarding process](/quickstart) 。我们将使用Onboarding过程中创建的 "服务 "蓝图。
* 您需要一个 Git 仓库(Github、GitLab 或 Bitbucket)，您可以在其中放置我们将在本指南中使用的工作流/Pipelines。如果没有，建议创建一个名为 "Port-actions "的新仓库。

:::

<br/>

### 本指南的目标

在本指南中，我们将在 Port 内部的 Git 仓库中打开一个拉取请求，使用 gitops 创建一个新的云资源。

完成这项工作后，你就会了解它如何使你的组织中的不同角色受益: 

* 平台工程师将能够定义强大的操作，开发人员可在受控的权限范围内使用这些操作。
* 开发人员将能从 Port 轻松创建和跟踪云资源。

### 在新资源定义中添加 URL

在本指南中，我们将在 `service`<PortTooltip id="blueprint">蓝图</PortTooltip>中添加一个新属性，我们可以用它来访问我们的云资源定义。

1. 请访问[Builder](https://app.getport.io/dev-portal/data-model) 。
2. 单击 "服务 "<PortTooltip id="blueprint">蓝图</PortTooltip>，然后单击 "新建属性"。
3. 选择 `URL` 作为类型，填写如下内容，然后点击 `保存`: 

<img src='/img/guides/iacPropertyForm.png' width='40%' />

在所有服务中，该属性暂时为空，我们将在接下来创建的操作中填充该属性 😎

#### 设置动作的前端

1. 前往 Port 应用程序中的[Self-service tab](https://app.getport.io/self-serve) ，点击 "+ 新操作"。
2. Port 中的每个操作都与<PortTooltip id="blueprint">蓝图</PortTooltip>直接相关。我们的操作将创建一个与服务相关联的资源，并作为服务 CD 流程的一部分进行供应。  
从下拉列表中选择 "服务"。
3.此操作不会创建/删除实体，而是对现有<PortTooltip id="entity">实体</PortTooltip>执行操作。因此，我们将选择 `Day-2` 作为操作类型。  
像这样填写表格，然后单击 "下一步": 

<img src='/img/guides/iacActionDetails.png' width='50%' />

<br/><br/>

4.我们希望使用此操作的开发人员能指定简单的输入，而不是被 S3 存储桶的所有可用配置弄得不知所措。对于此操作，我们将定义一个名称和公共/私有可见性。  
点击 "+ 新输入"，像这样填写表格，然后点击 "创建": 

<img src='/img/guides/iacActionInputName.png' width='50%' />

<br/><br/>

5.现在让我们创建可见性输入，它稍后将作为我们资源的 `acl`。  
点击 "+ 新输入法"，像这样填写表格，然后点击 "创建": 

<img src='/img/guides/iacActionInputVisibility.png' width='50%' />

<br/><br/>

6.现在我们来定义动作的后端。Port 支持多种调用类型，根据您在入门流程开始时选择的 Git Providers，我们会为您选择其中一种。

<Tabs groupId="git-provider" queryString defaultValue="github" values={[
{label: "GitHub", value: "github"},
{label: "GitLab", value: "gitlab"},
{label: "Bitbucket (Jenkins)", value: "bitbucket"}
]}>

<TabItem value="github">

在表格中填写您的 Values: 

* 用您的 Values 替换 `Organization` 和 `Repository` 值(这是工作流将驻留和运行的位置)。
* 将工作流命名为 `portCreateBucket.yaml`。
* 将 "忽略用户输入 "设置为 "是"。
* 像这样填写表单的其余部分，然后单击`下一步`: 

:::tip  重要

在我们的工作流程中，有效载荷被用于为输入。 为了避免向工作流程发送额外的输入，我们省略了用户输入。

:::

<img src='/img/guides/createBucketGHBackend.png' width='75%' />

</TabItem>

<TabItem value="gitlab">

:::tip 该部分需要一些参数，这些参数在[setup the action's backend](#setup-the-actions-backend) 部分生成，建议先完成该部分的步骤，然后在掌握所有所需信息的情况下按照此处的说明进行操作。

:::

在表格中填写您的 Values: 

* 端点 URL "需要添加以下格式的 URL: 


  ```text showLineNumbers
  https://gitlab.com/api/v4/projects/{GITLAB_PROJECT_ID}/ref/main/trigger/pipeline?token={GITLAB_TRIGGER_TOKEN}
  ```

    - The value for `{GITLAB_PROJECT_ID}` is the ID of the GitLab group that you create in the [setup the action's backend](#setup-the-actions-backend) section which stores the `.gitlab-ci.yml` pipeline file.
      - To find the project ID, browse to the GitLab page of the group you created, at the top right corner of the page, click on the vertical 3 dots button (next to `Fork`) and select `Copy project ID`
    - The value for `{GITLAB_TRIGGER_TOKEN}` is the trigger token you create in the [setup the action's backend](#setup-the-actions-backend) section.
- Set `HTTP method` to `POST`.
- Set `Request type` to `Async`.
- Set `Use self-hosted agent` to `No`.

<img src='/img/guides/gitlabActionBackendForm.png' width='75%' />

</TabItem>

<TabItem value="bitbucket">

Bitbucket 要求在操作中定义另一个输入。 创建以下输入: 

<img src='/img/guides/bitbucketWorkspaceActionInputConfig.png' width='50%' />

:::tip 该部分需要一些参数，这些参数在[setup the action's backend](#setup-the-actions-backend) 部分生成，建议先完成该部分的步骤，然后在掌握所有所需信息的情况下按照此处的说明进行操作。

:::

在表格中填写您的 Values: 

* 端点 URL "需要添加以下格式的 URL: 


  ```text showLineNumbers
  https://{JENKINS_URL}/generic-webhook-trigger/invoke?token={JOB_TOKEN}
  ```

    - The value for `{JENKINS_URL}` is the URL of your Jenkins server.
    - The value for `{JOB_TOKEN}` is the unique token used to trigger the pipeline you create in the [setup the action's backend](#setup-the-actions-backend) section.
- Set `HTTP method` to `POST`.
- Set `Request type` to `Async`.
- Set `Use self-hosted agent` to `No`.

<img src='/img/guides/bitbucketActionBackendForm.png' width='75%' />

</TabItem>

</Tabs>

<br/>

7.最后一步是自定义操作权限。为简单起见，我们将被用于默认设置。更多信息，请参阅[permissions](/create-self-service-experiences/set-self-service-actions-rbac/) 页面。单击 "创建"。

action的前端已准备就绪 🥳

#### 设置action的后端

现在，我们要编写我们的操作将触发的逻辑。

<Tabs groupId="git-provider" queryString defaultValue="github" values={[
{label: "GitHub", value: "github"},
{label: "GitLab", value: "gitlab"},
{label: "Bitbucket (Jenkins)", value: "bitbucket"}
]}>

<TabItem value="github">
1. First, let's create the necessary token and secrets. If you've already completed the [scaffold a new service guide](/guides-and-tutorials/scaffold-a-new-service), you should already have these configured and you can skip this step.

* 访问[Github tokens page](https://github.com/settings/tokens) ，创建一个包含`repo`和`admin:org`范围的个人访问令牌，并将其复制(从我们的工作流中创建拉取请求需要此令牌) 。
   <img src='/img/guides/personalAccessToken.png' width='80%' />- 访问[Port application](https://app.getport.io/) ，点击右上角的"..."，然后点击 "凭据"。复制您的 `客户 ID` 和 `客户 secret`。

2.在工作流程所在的版本库中，在 "设置->secret和变量->操作 "下创建 3 个新secret: 

* ORG_ADMIN_TOKEN` - 您在上一步中创建的个人访问令牌。
* `PORT_CLIENT_ID` - 从 Port 应用程序复制的客户端 ID。
* PORT_CLIENT_SECRET` - 从 Port 应用程序复制的客户机secret。

<img src='/img/guides/repositorySecret.png' width='60%' />

<br/><br/>

3.现在，让我们创建包含逻辑的工作流程文件。我们的工作流程将包括 3 个步骤: 

* 在所选服务的资源库中创建模板文件副本，并用操作输入的数据替换其中的变量。
* 在选定服务的资源库中创建拉取请求，添加新资源。
* 向 Port 报告和记录操作结果，并使用服务资源目录的 URL 更新相关服务的 "资源定义 "属性。

在`.github/workflows/`下创建一个名为`portCreateBucket.yaml`的新文件，并使用以下代码段作为其内容: 

<details>
<summary><b>Github workflow (click to expand)</b></summary>

```yaml showLineNumbers
name: Create cloud resource
on:
  workflow_dispatch:
    inputs:
      name:
        type: string
      visibility:
        type: string
      port_payload:
        required: true
        description: Port's payload, including details for who triggered the action and general context
        type: string
jobs:
  createResource:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/checkout@v3
        with:
          repository: "${{ github.repository_owner }}/${{fromJson(inputs.port_payload).context.entity}}"
          path: ./targetRepo
          token: ${{ secrets.ORG_ADMIN_TOKEN }}
      - name: Copy template file
        run: |
          mkdir -p ./targetRepo/resources
          cp templates/cloudResource.tf ./targetRepo/resources/${{ inputs.name }}.tf
      - name: Update new file data
        run: |
          sed -i 's/{{ bucket_name }}/${{ inputs.name }}/' ./targetRepo/resources/${{ inputs.name }}.tf
          sed -i 's/{{ bucket_acl }}/${{ inputs.visibility }}/' ./targetRepo/resources/${{ inputs.name }}.tf
      - name: Open a pull request
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.ORG_ADMIN_TOKEN }}
          path: ./targetRepo
          commit-message: Create new resource - ${{ inputs.name }}
          committer: GitHub <noreply@github.com>
          author: ${{ github.actor }} <${{ github.actor }}@users.noreply.github.com>
          signoff: false
          branch: new-resource-${{ inputs.name }}
          delete-branch: true
          title: Create new resource - ${{ inputs.name }}
          body: |
            Create new ${{ inputs.visibility }} resource - ${{ inputs.name }}
          draft: false
  create-entity-in-port-and-update-run:
    runs-on: ubuntu-latest
    needs: createResource
    steps:
      - name: UPSERT Entity
        uses: port-labs/port-github-action@v1
        with:
          identifier: ${{fromJson(inputs.port_payload).context.entity}}
          blueprint: service
          properties: |-
            {
              "resource_definitions": "${{ github.server_url }}/${{ github.repository_owner }}/${{fromJson(inputs.port_payload).context.entity}}/blob/main/resources/"
            }
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: UPSERT
          runId: ${{fromJson(inputs.port_payload).context.runId}}
      - name: Create a log message
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: PATCH_RUN
          runId: ${{fromJson(inputs.port_payload).context.runId}}
          logMessage: Pull request created successfully for "${{ inputs.name }}" 🚀
```

</details>

</TabItem>

<TabItem value="gitlab">

1. 首先，让我们创建一个 GitLab 项目，存储我们新的水桶创建管道--进入 GitLab 账户，创建一个新项目。
2. 接下来，创建必要的 token 和 secrets: 

* 进入[Port application](https://app.getport.io/) ，点击右上角的"..."，然后点击 "凭据"。复制 "客户 ID "和 "客户 secret"。
* 访问[project](https://gitlab.com/) ，按照[here](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html#create-a-project-access-token) 的步骤创建一个新的项目访问令牌，其权限范围如下: `write_repository`，然后保存其值，因为下一步将需要它。
   <img src='/img/guides/gitlabProjectAccessTokenPerms.png' width='80%' />
* 转到步骤 1 中创建的新 GitLab 项目，在左侧边栏的 "设置 "菜单中选择 "CI/CD"。
* 展开 "变量 "部分，保存以下secret: 
    - `PORT_CLIENT_ID` - 您的 Port 客户端 ID。
    - `PORT_CLIENT_SECRET` - 您的 Port 客户端secret。
    - `GITLAB_ACCESS_TOKEN` - 在上一步中创建的 GitLab 组访问令牌。
   <br/>
    <img src='/img/guides/gitlabPipelineVariables.png' width='80%' />
* 展开 "Pipelines 触发令牌 "部分并添加一个新令牌，给它一个有意义的描述，如 "Bucket 创建者令牌"，并保存其值
    - 这就是定义 Action 后端所需的 `{GITLAB_TRIGGER_TOKEN}`。

<br/>

  <img src='/img/guides/gitlabPipelineTriggerToken.png' width='80%' />

<br/><br/>

3.现在让我们创建包含逻辑的 Pipelines 文件。在步骤 1 创建的新 GitLab 项目中，在项目根目录下创建一个名为 `.gitlab-ci.yml`的新文件，并将以下代码段作为其内容: 

<details>
<summary><b>GitLab pipeline (click to expand)</b></summary>

```yaml showLineNumbers title=".gitlab-ci.yml"
image: python:3.10.0-alpine

stages: # List of stages for jobs, and their order of execution
  - fetch-port-access-token
  - create-tf-resource-pr
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
      echo "Getting access token from Port API"
      accessToken=$(curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"clientId": "'"$PORT_CLIENT_ID"'", "clientSecret": "'"$PORT_CLIENT_SECRET"'"}' \
        -s 'https://api.getport.io/v1/auth/access_token' | jq -r '.accessToken')
      echo "ACCESS_TOKEN=$accessToken" >> data.env
      runId=$(cat $TRIGGER_PAYLOAD | jq -r '.context.runId')
      echo "RUN_ID=$runId" >> data.env
      curl -X POST \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $accessToken" \
        -d '{"message":"🏃‍♂️ Starting S3 bucket creation process..."}' \
        "https://api.getport.io/v1/actions/runs/$runId/logs"
      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $accessToken" \
        -d '{"link":"'"$CI_PIPELINE_URL"'"}' \
        "https://api.getport.io/v1/actions/runs/$runId"
  artifacts:
    reports:
      dotenv: data.env

create-tf-resource-pr:
  before_script: |
    apk update
    apk add jq curl git -q
  stage: create-tf-resource-pr
  except:
    - pushes
  script:
    - | 
      git config --global user.email "bucketCreator@email.com"
      git config --global user.name "Bucket Creator"
      git config --global init.defaultBranch "main"
      git clone https://:${GITLAB_ACCESS_TOKEN}@gitlab.com/${CI_PROJECT_NAMESPACE}/${CI_PROJECT_NAME}.git sourceRepo
      cat $TRIGGER_PAYLOAD
      git clone https://:${GITLAB_ACCESS_TOKEN}@gitlab.com/${CI_PROJECT_NAMESPACE}/$(cat $TRIGGER_PAYLOAD | jq -r '.context.entity').git targetRepo
    - |
      bucket_name=$(cat $TRIGGER_PAYLOAD | jq -r '.payload.properties.name')
      visibility=$(cat $TRIGGER_PAYLOAD | jq -r '.payload.properties.visibility')
      echo "BUCKET_NAME=${bucket_name}" >> data.env
      echo "Creating a new S3 bucket Terraform resource file"
      mkdir -p targetRepo/resources/
      cp sourceRepo/templates/cloudResource.tf targetRepo/resources/${bucket_name}.tf
      sed -i "s/{{ bucket_name }}/${bucket_name}/" ./targetRepo/resources/${bucket_name}.tf
      sed -i "s/{{ bucket_acl }}/${visibility}/" ./targetRepo/resources/${bucket_name}.tf
    - |
      cd ./targetRepo
      git add resources/${bucket_name}.tf
      git commit -m "Added ${bucket_name} resource file"
      git checkout -b new-bucket-branch-${bucket_name}
      git push origin new-bucket-branch-${bucket_name}
      PROJECT_NAME=$(cat $TRIGGER_PAYLOAD | jq -r '.context.entity | @uri')
      PROJECTS=$(curl --header "PRIVATE-TOKEN: $GITLAB_ACCESS_TOKEN" "https://gitlab.com/api/v4/groups/$CI_PROJECT_NAMESPACE_ID/projects?search=$(cat $TRIGGER_PAYLOAD | jq -r '.context.entity')")
      PROJECT_ID=$(echo ${PROJECTS} | jq '.[] | select(.name=="'$PROJECT_NAME'") | .id' | head -n1)

      PR_RESPONSE=$(curl --request POST --header "PRIVATE-TOKEN: ${GITLAB_ACCESS_TOKEN}" "https://gitlab.com/api/v4/projects/${PROJECT_ID}/merge_requests?source_branch=new-bucket-branch-${bucket_name}&target_branch=main&title=New-Bucket-Request")
      PR_URL=$(echo ${PR_RESPONSE} | jq -r '.web_url')
      curl -X POST \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d "{\"message\":\"📡 Opened pull request with new bucket resource!\nPR Url: ${PR_URL}\"}" \
        "https://api.getport.io/v1/actions/runs/$RUN_ID/logs"

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
      echo "Creating Port entity to match new S3 bucket"
      SERVICE_ID=$(cat $TRIGGER_PAYLOAD | jq -r '.context.entity')
      PROJECT_URL="https://gitlab.com/${CI_PROJECT_NAMESPACE_ID}/${SERVICE_ID}/-/blob/main/resources/"
      echo "SERVICE_ID=${SERVICE_ID}" >> data.env
      echo "PROJECT_URL=${PROJECT_URL}" >> data.env
      curl -X POST \
          -H 'Content-Type: application/json' \
          -H "Authorization: Bearer $ACCESS_TOKEN" \
          -d '{"message":"🚀 Updating the service with the new resource definition!"}' \
          "https://api.getport.io/v1/actions/runs/$RUN_ID/logs"
      curl --location --request POST "https://api.getport.io/v1/blueprints/service/entities?upsert=true&run_id=$RUN_ID&create_missing_related_entities=true" \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "Content-Type: application/json" \
        -d '{"identifier": "'"$SERVICE_ID"'","title": "'"$SERVICE_ID"'","properties": {"resource_definitions": "'"$PROJECT_URL"'"}, "relations": {}}'

update-run-status:
  stage: update-run-status
  except:
    - pushes
  image: curlimages/curl:latest
  script:
    - |
      echo "Updating Port action run status and final logs"
      curl -X POST \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"message":"✅ PR Opened for bucket '"$BUCKET_NAME"'!"}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID/logs"
      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"status":"SUCCESS",  "message": {"run_status": "Run completed successfully!"}}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID"
```

</details>

</TabItem>

<TabItem value="bitbucket">

1. 首先，在 Jenkins 中安装[generic webhook trigger](https://plugins.jenkins.io/generic-webhook-trigger/) 插件。
2. 接下来，让我们创建必要的令牌和 Secret
    - 进入[Port application](https://app.getport.io/) ，点击右上角的"..."，然后点击 "Credentials"。复制你的 `客户 ID` 和 `客户 secret`.
    - 将以下内容配置为 Jenkins 凭据: 
        + `BITBUCKET_USERNAME` - 可以访问 Bitbucket Workspace 和项目的用户。
        + `BITBUCKET_APP_PASSWORD` - 具有 `Repositories:Read` 和 `Repositories:Write` 权限的[App Password](https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/) 。
        + `PORT_CLIENT_ID` - 您的 Port 客户端 ID。
        + `PORT_CLIENT_SECRET` - 您的 Port 客户端secret。
       <br/>
        <img src='/img/guides/bitbucketJenkinsCredentials.png' width='80%' />

<br/>

3.用以下配置创建一个 Jenkins 管道: 
    -[Enable the webhook trigger for the pipeline](/create-self-service-experiences/setup-backend/jenkins-pipeline/jenkins-pipeline.md#enabling-webhook-trigger-for-a-pipeline)
    - 定义[`token`](/create-self-service-experiences/setup-backend/jenkins-pipeline/jenkins-pipeline.md#token-setup) 字段的值，您指定的令牌将被用于专门触发脚手架管道。例如，你可以被用于 `bucket-creator-token`。返回[frontend setup](#setup-the-actions-frontend) 至步骤 #6，并为触发 URL 设置`{JOB_TOKEN}`。
    -[Define variables for the pipeline](/create-self-service-experiences/setup-backend/jenkins-pipeline/jenkins-pipeline.md#defining-variables) : 定义 `SERVICE_NAME`、`BITBUCKET_WORKSPACE_NAME`、`BITBUCKET_PROJECT_KEY`、`BUCKET_NAME`、`VISIBILITY` 和 `RUN_ID` 变量。向下滚动到 "发布内容参数"，并**为每个变量**添加配置，如下所示(完整的变量列表请参见下表) : 
   <img src='/img/guides/jenkinsGenericVariable.png' width='100%' />创建以下变量及其相关 JSONPath 表达式: | 变量名 | JSONPath 表达式 |  |  | JSONPath 表达式 |  |  | JSONPath 表达式。
     | ------------------------ | ----------------------------------------------- |
     | SERVICE_NAME | `$.context.entity` | BITBUCKET
     | BITBUCKET_WORKSPACE_NAME | `$.payload.properties.bitbucket_workspace_name` | | RUN_ID | `$.payload.properties.bitbucket_workspace_name` | RUN_ID
     | RUN_ID | `$.context.runId` | `$.payload.properties.bitbucket_workspace_name
     BUCKET_NAME | `$.payload.properties.bucket_name` | | $.payload.properties.bitbucket_workspace_name` | | $.context.runId
     VISIBILITY | `$.payload.properties.visibility` | `$.payload.properties.target` | `$.payload.properties.target

在新的 Jenkins Pipelines 中添加以下内容: 

<details>
<summary><b>Jenkins pipeline (click to expand)</b></summary>

```groovy showLineNumbers
import groovy.json.JsonSlurper

pipeline {
    agent any

    environment {
        REPO_NAME = "${SERVICE_NAME}"
        BITBUCKET_WORKSPACE_NAME = "${BITBUCKET_WORKSPACE_NAME}"
        PORT_ACCESS_TOKEN = ""
        PORT_BLUEPRINT_ID = "service"
        PORT_RUN_ID = "${RUN_ID}"
        VISIBILITY="${VISIBILITY}"
        PR_URL=""
        SOURCE_REPO="port-actions" // UPDATE WITH YOUR SOURCE REPO NAME
    }

    stages {
        stage('Get access token') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'PORT_CLIENT_ID', variable: 'PORT_CLIENT_ID'),
                        string(credentialsId: 'PORT_CLIENT_SECRET', variable: 'PORT_CLIENT_SECRET')
                    ]) {
                        // Execute the curl command and capture the output
                        def result = sh(returnStdout: true, script: """
                            accessTokenPayload=\$(curl -X POST \
                                -H "Content-Type: application/json" \
                                -d '{"clientId": "${PORT_CLIENT_ID}", "clientSecret": "${PORT_CLIENT_SECRET}"}' \
                                -s "https://api.getport.io/v1/auth/access_token")
                            echo \$accessTokenPayload
                        """)

                        // Parse the JSON response using JsonSlurper
                        def jsonSlurper = new JsonSlurper()
                        def payloadJson = jsonSlurper.parseText(result.trim())

                        // Access the desired data from the payload
                        PORT_ACCESS_TOKEN = payloadJson.accessToken
                    }

                }
            }
        } // end of stage Get access token

        stage('Create Terraform resource Pull request') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'BITBUCKET_USERNAME', variable: 'BITBUCKET_USERNAME'),
                        string(credentialsId: 'BITBUCKET_APP_PASSWORD', variable: 'BITBUCKET_APP_PASSWORD')
                    ]) {
                    // Set Git configuration
                    sh "git config --global user.email 'bucketCreator@email.com'"
                    sh "git config --global user.name 'Bucket Creator'"
                    sh "git config --global init.defaultBranch 'main'"

                    // Clone source repository
                    sh "git clone https://${BITBUCKET_USERNAME}:${BITBUCKET_APP_PASSWORD}@bitbucket.org/${BITBUCKET_WORKSPACE_NAME}/${SOURCE_REPO}.git sourceRepo"
                    // Clone source repository
                    sh "git clone https://${BITBUCKET_USERNAME}:${BITBUCKET_APP_PASSWORD}@bitbucket.org/${BITBUCKET_WORKSPACE_NAME}/${REPO_NAME}.git targetRepo"

                    def logs_report_response = sh(script: """
                        curl -X POST \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"message": "Creating a new S3 bucket Terraform resource file: ${REPO_NAME} in Workspace: ${BITBUCKET_WORKSPACE_NAME}"}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }}
                script {
                    withCredentials([
                        string(credentialsId: 'BITBUCKET_USERNAME', variable: 'BITBUCKET_USERNAME'),
                        string(credentialsId: 'BITBUCKET_APP_PASSWORD', variable: 'BITBUCKET_APP_PASSWORD')
                    ]) {
                        sh """
                        bucket_name=${BUCKET_NAME}
                        visibility=${VISIBILITY}
                        echo 'Creating a new S3 bucket Terraform resource file'
                        mkdir -p targetRepo/resources/
                        cp sourceRepo/templates/cloudResource.tf targetRepo/resources/${BUCKET_NAME}.tf
                        sed -i 's/{{ bucket_name }}/${BUCKET_NAME}/' ./targetRepo/resources/${BUCKET_NAME}.tf
                        sed -i 's/{{ bucket_acl }}/${VISIBILITY}/' ./targetRepo/resources/${BUCKET_NAME}.tf
                        cd ./targetRepo
                        git add resources/${bucket_name}.tf
                        git commit -m "Added ${bucket_name} resource file"
                        git checkout -b new-bucket-branch-${bucket_name}
                        git push origin new-bucket-branch-${bucket_name}
                    """
                    def pr_response = sh(script:"""
                        curl -u ${BITBUCKET_USERNAME}:${BITBUCKET_APP_PASSWORD} --header 'Content-Type: application/json' \\
                            -d '{"title": "New Bucket request for ${BUCKET_NAME}", "source": {"branch": {"name": "new-bucket-branch-${BUCKET_NAME}"}}}' \\
                            https://api.bitbucket.org/2.0/repositories/${BITBUCKET_WORKSPACE_NAME}/${SERVICE_NAME}/pullrequests
                    """, returnStdout: true)
                    def jsonSlurper = new JsonSlurper()
                    def payloadJson = jsonSlurper.parseText(pr_response.trim())

                    // Access the desired data from the payload
                    PR_URL = payloadJson.links.html.href
                    println("${PR_URL}")
                    }
                }
            }
        } // end of Create Terraform resource Pull request stage

        stage('Update service entity') {
            steps {
                script {
                    def logs_report_response = sh(script: """
                        curl -X POST \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"message": "🚀 Updating the service with the new resource definition!"}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }
                script {
                    def status_report_response = sh(script: """
    					curl --location --request POST "https://api.getport.io/v1/blueprints/$PORT_BLUEPRINT_ID/entities?upsert=true&run_id=$PORT_RUN_ID&create_missing_related_entities=true" \
        --header "Authorization: Bearer $PORT_ACCESS_TOKEN" \
        --header "Content-Type: application/json" \
        --data-raw '{
    			"identifier": "${REPO_NAME}",
    			"title": "${REPO_NAME}",
    			"properties": {"resource_definitions":"https://bitbucket.org/${BITBUCKET_WORKSPACE_NAME}/${REPO_NAME}/src/main/resources/"},
    			"relations": {}
    		}'

                    """, returnStdout: true)

                    println(status_report_response)
                }
            }
        } // end of stage CREATE Microservice entity

        stage('Update Port Run Status') {
            steps {
                script {
                    def status_report_response = sh(script: """
                        curl -X POST \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"message":"✅ PR Opened for bucket '"${BUCKET_NAME}"'!"}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}/logs"
                        curl -X PATCH \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"link":"${PR_URL}","status":"SUCCESS", "message": {"run_status": "Run completed successfully!"}}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}"
                        rm -rf ./sourceRepo ./targetRepo
                    """, returnStdout: true)

                    println(status_report_response)
                }
            }
        } // end of stage Update Port Run Status
    }

    post {

        failure {
            // Update Port Run failed.
            script {
                def status_report_response = sh(script: """
                    curl -X PATCH \
                        -H "Content-Type: application/json" \
                        -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                        -d '{"status":"FAILURE", "message": {"run_status": "Run failed!❌"}}' \
                            "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}"
                """, returnStdout: true)
                sh "rm -rf ./sourceRepo ./targetRepo"
                println(status_report_response)
            }
        }

        // Clean after build
        always {
            cleanWs(cleanWhenNotBuilt: true,
                    deleteDirs: true,
                    disableDeferredWipeout: false,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
    }
}
```

</details>

</TabItem>

</Tabs>

4.现在，我们将创建一个简单的 `.tf` 文件，作为新资源的模板: 

* 在源代码库(例如 `port-actions`)中的 `/templates/`(路径应为 `/templates/cloudResource.tf`)下创建一个名为 `cloudResource.tf` 的文件。
* 复制以下代码段并粘贴到文件内容中: 

<details>
<summary><b>cloudResource.tf (click to expand)</b></summary>

```hcl showLineNumbers title="cloudResource.tf"
resource "aws_s3_bucket" "example" {
provider = aws.bucket_region
name = "{{ bucket_name }}"
acl = "{{ bucket_acl }}"
}
```

</details>

完成！操作已准备就绪 🚀

<br/>

### 执行操作

创建操作后，该操作将出现在 Port 应用程序的 "自助服务 "选项卡下: 

<img src='/img/guides/iacActionAfterCreation.png' width='35%' />

1. 点击 "执行"。
2. 输入 s3 存储桶的名称并选择可见性，从列表中选择任何服务并点击 "执行"。弹出一个小窗口，点击 "查看详情": 

<img src='/img/guides/iacActionExecutePopup.png' width='40%' />

<br/><br/>

3.该页面提供了有关操作运行的详细信息。我们可以看到，后端返回了 "成功"，拉取请求已成功创建: 

<img src='/img/guides/iacActionRunAfterExecution.png' width='90%' />

#### 从 Port 访问水桶的定义

您可能已经注意到，即使我们更新了服务的 "资源定义 "URL，它仍然指向一个不存在的页面。 这是因为我们的资源库中还没有任何资源，让我们来解决这个问题: 

1. 合并拉动请求。
2. 转到为其执行操作的服务的<PortTooltip id="entity">实体</PortTooltip>页面: 

<img src='/img/guides/iacEntityAfterAction.png' width='50%' />

<br/><br/>

3.单击 "资源定义 "链接，访问服务资源。

全部完成！现在您可以直接从 Port 💪🏽 为您的服务创建资源了

### 可能的日常整合

* 向组织中的相关人员发送松弛消息，通知新资源。
* 向经理/开发人员发送周报/月报，显示该时间段内创建的新资源及其 Owner。

#### 结论

开发人员门户需要支持并与 git-ops 实践无缝集成。 开发人员应能独立执行常规任务，而不必在组织内部造成瓶颈。 借助 Port，平台工程师可以为开发人员设计精确灵活的自助操作，同时与多种不同的后端集成，以满足您的特定需求。

更多相关指南和示例: 

* [使用 AWS CloudFormation 部署 AWS 资源
](https://docs.getport.io/create-self-service-experiences/setup-backend/github-workflow/examples/deploy-cloudformation-template)
* [Create an S3 bucket using Self-Service Actions](https://docs.getport.io/create-self-service-experiences/setup-backend/webhook/examples/s3-using-webhook/)
