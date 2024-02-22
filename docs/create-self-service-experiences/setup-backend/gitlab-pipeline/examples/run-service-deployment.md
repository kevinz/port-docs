# 为服务运行部署

在下面的指南中，您将在 Port 中创建一个自助操作，在幕后执行 GitLab 管道。

本例中的 GitLab 管道将运行一个新的部署，并向 Port 报告操作运行状态和部署实体。

## 先决条件

* Port API `CLIENT_ID`和`CLIENT_SECRET`。

## 创建 GitLab Pipelines

首先，我们需要为 CI/CD 流程设置一个 GitLab 管道。

您可以使用 GitLab CI/CD 中的[Port's API](../../../../build-your-software-catalog/sync-data-to-catalog/api/api.md) 这个被引用的示例: 

<details>
<summary>Click here to see the code</summary>

```yaml showLineNumbers
stages: # List of stages for jobs, and their order of execution
  - save-port-data
  - deploy
  - report-deployment
  - send-logs
  - update-status

save-port-data: # Example - get the Port API access token and RunId
  stage: save-port-data
  before_script:
    - apt-get -qq update
    - apt-get install -y jq
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

deploy-job:
  stage: deploy
  script:
    - echo "Deploying application..."
    ## Enter your deploy logic here

report-deployment: # Example - create a deployment entity
  stage: report-deployment
  script:
    - |
      curl --location --request POST "https://api.getport.io/v1/blueprints/deployment/entities?upsert=true" \
        --header "Authorization: Bearer $ACCESS_TOKEN" \
        --header "Content-Type: application/json" \
        --data-raw '{
          "identifier": "'"$service-$environment"'",
          "properties": {"jobUrl":"'"$CI_JOB_URL"'","imageTag":"latest"},
          "relations": {}
        }'

send-logs: # Example - send Logs of the action run to Port
  stage: send-logs
  script:
    - |
      curl -X POST \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"message": "this is a log test message example"}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID/logs"

update-status: # Example - update the Action run status as success
  stage: update-status
  image: curlimages/curl:latest
  script:
    - |
      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"status":"SUCCESS", "message": {"run_status": "GitLab CI/CD Run completed successfully!"}}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID"
```

</details>

:::important  重要事项 为了通过 Port 代理触发 GitLab Pipelines，您需要[create a GitLab Pipeline trigger token](https://docs.gitlab.com/ee/ci/triggers/).

只要拥有一个具有所需权限的触发令牌，就可以在多个 GitLab 项目中触发 Pipelines。

触发令牌作为环境变量加载到 Port 代理。

:::

## 创建蓝图

让我们配置一个新的蓝图，命名为 "部署"，其基本结构是

```json showLineNumbers
{
  "identifier": "deployment",
  "title": "Deployment",
  "icon": "Deployment",
  "schema": {
    "properties": {
      "jobUrl": {
        "type": "string",
        "format": "url",
        "title": "Job URL"
      },
      "deployingUser": {
        "type": "string",
        "title": "Deploying User"
      },
      "imageTag": {
        "type": "string",
        "title": "Image Tag"
      },
      "commitSha": {
        "type": "string",
        "title": "Commit SHA"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

## 创建 Port 操作

现在让我们配置一个自助服务操作。 添加一个 `CREATE` 操作，开发人员每次要为服务启动新的部署时都会触发该操作。

以下是该操作的 JSON 文件: 

```json showLineNumbers
{
  "identifier": "runGitLabPipline",
  "title": "Trigger Gitlab Pipeline",
  "icon": "DeployedAt",
  "userInputs": {
    "properties": {
      "ref": {
        "type": "string",
        "title": "Ref"
      },
      "pipelineVariable1": {
        "type": "string",
        "title": "First Pipeline Variable "
      },
      "pipelineVariable2": {
        "type": "string",
        "title": "Second Pipeline Variable"
      }
    }
  },
  "invocationMethod": {
    "type": "GITLAB",
    "projectName": "project",
    "groupName": "group",
    "defaultRef": "main",
    "agent": true,
    "omitPayload": false,
    "omitUserInputs": false
  },
  "trigger": "CREATE",
  "description": "Trigger a GitLab Pipeline through the Port Agent",
  "requiredApproval": false
}
```

### 调用方法属性


| Field            | Type      | Description                                                                                                                                                                                                                                                                                                                                                 | Example values                       |
| ---------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `type`           | `string`  | Defines the self-service action destination type                                                                                                                                                                                                                                                                                                            | `GITLAB`                             |
| `agent`          | `boolean` | Defines whether to use [Port Agent](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/port-execution-agent.md) for execution or not.                                                                                                                                                                                                                                               | must be `true` in GitLab action type |
| `projectName`    | `string`  | Defines the GitLab project name                                                                                                                                                                                                                                                                                                                             | `port`                               |
| `groupName`      | `string`  | Defines the GitLab group name                                                                                                                                                                                                                                                                                                                               | `port-labs`                          |
| `defaultRef`     | `string`  | The default ref (branch / tag name) we want the action to use. <br></br> `defaultRef` can be overriden dynamically,<br></br> by adding `ref` as user input. <br></br> If not set, the agent triggers `main` branch                                                                                                                                          | `main`                               |
| `omitPayload`    | `boolean` | Flag to control whether to add [`port_payload`](../../../self-service-actions-deep-dive/self-service-actions-deep-dive.md#action-message-structure) JSON string to the GitLab pipeline trigger payload (default: `false`). If set to true, the payload will not be sent as part of the body.                                                                | `true` or `false`                    |
| `omitUserInputs` | `boolean` | Flag to control whether to send the user inputs of the Port action as [GitLab CI/CD variables](https://docs.gitlab.com/ee/ci/variables/) to the GitLab pipeline. <br></br> By default, the user inputs are passed as variables (default: `false`). <br></br> When disabled, you can still get the user inputs from the `port_payload` (unless omitted too). | `true` or `false`                    |