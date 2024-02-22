---
sidebar_position: 2
---

# 使用 crossplane 部署 S3 存储桶

本示例演示了如何通过 Port Actions 在 Kubernetes 集群中部署[Crossplane](https://github.com/crossplane/crossplane) 资源。

我们将使用 Gitlab Pipeline 创建一个合并请求，将文件提交到项目中。

预部署的 GitOps 机制将负责在我们的集群中创建资源。

## 先决条件

* Kubernetes 集群。
* 集群中安装了 Crossplane: 
    - Crossplane[Installation Guide](https://docs.crossplane.io/v1.14/software/install/) 。
    - Crossplane[AWS Quickstart Guide](https://docs.crossplane.io/v1.14/getting-started/provider-aws/) : 
        + 部署一个 Crossplane[Provider](https://docs.crossplane.io/v1.14/getting-started/provider-aws/#install-the-aws-provider) 。
        + 部署一个 Crossplane[ProviderConfig](https://docs.crossplane.io/v1.14/getting-started/provider-aws/#create-a-providerconfig) 。
* 您选择的 GitOps 机制，可将文件从 Gitlab 项目同步到 Kubernetes 集群。

## 示例 - 在 AWS 中部署 S3 存储桶

在本示例中，我们将向您展示如何利用 Port Actions 在 Gitlab 项目中打开一个合并请求，该合并请求会提交一个描述 AWS 中 S3 Bucket 的 crossplane 配置清单。

:::note  免责声明 本示例没有实现 GitOps 工具来确保配置清单将在你的 Kubernetes 集群上创建，因此请注意，如果你只执行以下指南中的步骤，你将只能在 Gitlab 项目中创建合并请求。

:::

请按照以下步骤开始操作: 

1. 创建以下变量作为[Gitlab Variables](https://docs.gitlab.com/ee/ci/variables/index.html) : 
    1. `ACCESS_TOKEN` -[Personal Access Token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) ，其作用域如下:   
    api`、`read_api`、`read_user`、`read_repository`、`write_repository`。
    2. `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    3. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token).
       <br/>
2.在 Gitlab 组中创建名为 `crossplane_deployer` 的 Gitlab 项目，并配置[Pipeline Trigger Token](https://docs.gitlab.com/ee/ci/triggers/index.html) 。

:::note 您可以被引用任何您喜欢的名称，只要确保在 Port Action 中正确配置即可。

:::

<br/>

3.按照我们的指南[here](/create-self-service-experiences/setup-backend/gitlab-pipeline/Installation) 安装 Port 的 Gitlab 代理。

:::note 确保在安装 Port 的 Gitlab 代理时被引用 Pipeline 触发令牌。

:::

<br/>

4.使用以下 JSON 定义创建 Port 蓝图: 

:::note 请记住，这可以是你想要的任何蓝图，这只是一个例子。

:::

<details>
  <summary>Port S3Bucket Blueprint</summary>

```json showLineNumbers
{
  "identifier": "s3bucket",
  "title": "S3Bucket",
  "icon": "Crossplane",
  "schema": {
    "properties": {
      "aws_region": {
        "title": "AWS Region",
        "icon": "AWS",
        "type": "string"
      }
    },
    "required": ["aws_region"]
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>
<br/>

5.使用以下 JSON 定义创建 Port 操作: 

:::note 确保替换 `crossplane_deployer` 中 PROJECT_NAME 和 GROUP_NAME 的占位符。

:::

<details>
  <summary>Port Action</summary>

```json showLineNumbers
[
  {
    "identifier": "crossplane_s3_bucket",
    "title": "Crossplane S3 Bucket",
    "icon": "Crossplane",
    "userInputs": {
      "properties": {
        "aws_region": {
          "icon": "AWS",
          "title": "AWS Region",
          "type": "string",
          "default": "us-east-1",
          "enum": ["us-east-1", "eu-west-1"],
          "enumColors": {
            "us-east-1": "lightGray",
            "eu-west-1": "lightGray"
          }
        },
        "bucket_name": {
          "title": "Bucket Name",
          "type": "string",
          "description": "Has to be globally unique as per AWS limitations"
        }
      },
      "required": ["aws_region", "bucket_name"],
      "order": ["bucket_name", "aws_region"]
    },
    "invocationMethod": {
      "type": "GITLAB",
      "omitPayload": false,
      "omitUserInputs": false,
      "projectName": "<PROJECT_NAME>",
      "groupName": "<GROUP_NAME>",
      "agent": true
    },
    "trigger": "CREATE",
    "description": "Creates a crossplane file for a new S3 Bucket",
    "requiredApproval": false
  }
]
```

</details>
<br/>

6.在你的 `crossplane_deployer` Gitlab 项目中，在 `main` 分支的 `crossplane-templates` 目录中创建一个名为 `s3bucket-crossplane.yaml` 的模板文件，内容如下: 

<details>
<summary>s3bucket-crossplane.yaml</summary>

```yml
# s3bucket-crossplane.yaml

apiVersion: s3.aws.upbound.io/v1beta1
kind: Bucket
metadata:
  name: {{ bucket_name }}
spec:
  forProvider:
    region: {{ aws_region }}
  providerConfigRef:
    name: default
```

</details>
<br/>

7.在你的 `crossplane_deployer` Gitlab 项目中，在 `main` 分支的 `.gitlab-ci.yml` 下创建一个 Gitlab CI 文件，内容如下: 

<details>
<summary>Gitlab CI Script</summary>

```yml showLineNumbers
image: python:3.10.0-alpine

stages: # List of stages for jobs, and their order of execution
  - fetch-port-access-token
  - generate-crossplane-bucket-yaml
  - create-entity
  - update-run-status

fetch-port-access-token: # Example - get the Port API access token and RunId
  stage: fetch-port-access-token
  except:
    - pushes
  before_script:
    - |
      apk update -q
      apk add jq curl -q
  script:
    - |
      accessToken=$(curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"clientId": "'"$PORT_CLIENT_ID"'", "clientSecret": "'"$PORT_CLIENT_SECRET"'"}' \
        -s 'https://api.getport.io/v1/auth/access_token' | jq -r '.accessToken')
      echo "PORT_ACCESS_TOKEN=$accessToken" >> data.env
      runId=$(cat $TRIGGER_PAYLOAD | jq -r '.port_payload.context.runId')
      blueprintId=$(cat $TRIGGER_PAYLOAD | jq -r '.port_payload.context.blueprint')
      echo "RUN_ID=$runId" >> data.env
      echo "BLUEPRINT_ID=$blueprintId" >> data.env
  artifacts:
    reports:
      dotenv: data.env

generate-crossplane-bucket-yaml:
  variables:
    BUCKET_FILE_PATH: "manifests"
    CROSSPLANE_TEMPLATE_PATH: "crossplane-templates/s3bucket-crossplane.yaml"
    BRANCH_NAME: "add-bucket-$bucket_name-$CI_JOB_ID"
  before_script:
    - |
      apk update -q
      apk add jq curl git -q
  stage: generate-crossplane-bucket-yaml
  except:
    - pushes
  script:
    - |
      BUCKET_FILE_NAME="$BUCKET_FILE_PATH/s3bucket-crossplane-$bucket_name.yaml"
      COMMIT_MESSAGE="Added $bucket_name s3 bucket crossplane manifest"
      mkdir -p $BUCKET_FILE_PATH

      cp $CROSSPLANE_TEMPLATE_PATH $BUCKET_FILE_NAME
      sed -i "s/{{ bucket_name }}/$bucket_name/g" $BUCKET_FILE_NAME
      sed -i "s/{{ aws_region }}/$aws_region/g" $BUCKET_FILE_NAME

      git config --global user.email "gitlab-pipeline[bot]@gitlab.com"
      git config --global user.name "Gitlab Pipeline Bot"

      git add $BUCKET_FILE_NAME
      git commit -m "$COMMIT_MESSAGE"

      git checkout -b $BRANCH_NAME
      git push -o ci-skip https://:${ACCESS_TOKEN}@$CI_SERVER_HOST/$CI_PROJECT_PATH.git $BRANCH_NAME

      # Create Merge Request
      res=$(curl --request POST \
        --header "PRIVATE-TOKEN: ${ACCESS_TOKEN}" \
        --data "source_branch=$BRANCH_NAME" \
        --data "target_branch=main" \
        --data "title=$COMMIT_MESSAGE" \
        --data "remove_source_branch=true" \
        "$CI_API_V4_URL/projects/$CI_PROJECT_ID/merge_requests")

      MR_URL=$(echo $res | jq -r '.web_url')
      echo "MR_URL=$MR_URL" >> data.env
  artifacts:
    reports:
      dotenv: data.env

update-run-status:
  stage: update-run-status
  except:
    - pushes
  image: curlimages/curl:latest
  script:
    - |
      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $PORT_ACCESS_TOKEN" \
        -d '{"status":"SUCCESS", "message": {"run_status": "Created Merge Request for '"$bucket_name"' successfully! Merge Request URL: '"$MR_URL"'"}}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID"
```

</details>
<br/>

8.从 Port 应用程序的[Self-service](https://app.getport.io/self-serve) 标签触发该操作。<br/> 您会发现创建了一个新的合并请求，提交了 S3 Bucket 配置清单。

## 下一步

在本例中，我们没有为 S3 存储桶创建 Port 实体。

* 您可以通过[Connect Port's AWS exporter](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/aws.md) 确保从 AWS 自动摄取所有属性和实体。
    - 您可以了解如何设置 Port 的 AWS 输出程序[here](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/Installation.md) 。
    - 您可以查看示例配置和用例[here](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/examples.md) 。
