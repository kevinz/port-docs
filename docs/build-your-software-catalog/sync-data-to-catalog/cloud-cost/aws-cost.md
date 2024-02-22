import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# AWS 成本

通过 AWS 成本集成，您可以将 "AWS 成本 "报告导入 Port。

Port 中的每个实体代表每张账单中单个资源的成本详情。

根据您的配置，实体将在 Port 中保存几个月(默认为 3 个月)。

## 安装

:::info 这是一个[open-source](https://github.com/port-labs/port-aws-cost-exporter) 集成。

:::

1. [AWS Setup](#aws) - 需要。
2. [Port Setup](#port) - 需要。
3. [Exporter Setup](#exporter) - 请选择其中一个选项: 
    -[Local](#local)
    -[Docker](#docker)
    -[GitHub Workflow](#github-workflow)
    -[GitLab Pipeline](#gitlab-pipeline)

![Catalog Architecture](/img/sync-data-to-catalog/aws_cost.png)

#### AWS

1. [Create an AWS S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html) 用于托管成本报告(将 `<AWS_BUCKET_NAME>`, `<AWS_REGION>` 替换为您想要的存储桶名称和 AWS 区域).

```bash showLineNumbers
aws s3api create-bucket --bucket <AWS_BUCKET_NAME> --region <AWS_REGION>
```

2.在本地创建一个名为 `policy.json` 的文件。将以下[content](https://docs.aws.amazon.com/cur/latest/userguide/cur-s3.html) 复制并粘贴到该文件中并保存，确保用第一步中创建的水桶名称和 AWS 账户 ID 更新 `<AWS_BUCKET_NAME>` 和 `<AWS_ACCOUNT_ID>`。

<details>
  <summary> policy.json </summary>

```json showLineNumbers
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "billingreports.amazonaws.com"
      },
      "Action": ["s3:GetBucketAcl", "s3:GetBucketPolicy"],
      # highlight-next-line
      "Resource": "arn:aws:s3:::<AWS_BUCKET_NAME>",
      "Condition": {
        "StringEquals": {
          # highlight-next-line
          "aws:SourceArn": "arn:aws:cur:us-east-1:<AWS_ACCOUNT_ID>:definition/*",
          # highlight-next-line
          "aws:SourceAccount": "<AWS_ACCOUNT_ID>"
        }
      }
    },
    {
      "Sid": "Stmt1335892526596",
      "Effect": "Allow",
      "Principal": {
        "Service": "billingreports.amazonaws.com"
      },
      "Action": "s3:PutObject",
      # highlight-next-line
      "Resource": "arn:aws:s3:::<AWS_BUCKET_NAME>/*",
      "Condition": {
        "StringEquals": {
          # highlight-next-line
          "aws:SourceArn": "arn:aws:cur:us-east-1:<AWS_ACCOUNT_ID>:definition/*",
          # highlight-next-line
          "aws:SourceAccount": "<AWS_ACCOUNT_ID>"
        }
      }
    }
  ]
}
```

</details>

3.将您在第二步中创建的[bucket policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/add-bucket-policy.html) 添加到您在第一步中创建的存储桶中(以下命令假定 `policy.json` 位于您当前的工作目录中，如果保存在其他地方，则应更新文件的正确位置) 。此策略将允许 AWS 将成本和使用报告 (CUR) (将 `<AWS_BUCKET_NAME>` 替换为您在第一步中创建的存储桶名称) 写入您的存储桶: 

```bash
aws s3api put-bucket-policy --bucket <AWS_BUCKET_NAME> --policy file://policy.json
```

4.在本地创建一个名为 `report-definition.json`的文件。将以下推荐内容复制并粘贴到该文件中并保存，确保用第一步中创建的水桶名称和目标 AWS 区域更新 `<AWS_BUCKET_NAME>` 和 `<AWS_REGION>`。

<details>
  <summary> report-definition.json </summary>

```json showLineNumbers
{
  "ReportName": "aws-monthly-cost-report-for-port",
  "TimeUnit": "MONTHLY",
  "Format": "textORcsv",
  "Compression": "GZIP",
  "AdditionalSchemaElements": ["RESOURCES"],
  # highlight-next-line
  "S3Bucket": "<AWS_BUCKET_NAME>",
  "S3Prefix": "cost-reports",
  # highlight-next-line
  "S3Region": "<AWS_REGION>",
  "RefreshClosedReports": true,
  "ReportVersioning": "OVERWRITE_REPORT"
}
```

</details>

5. [Create an AWS Cost and Usage Report](https://docs.aws.amazon.com/cur/latest/userguide/cur-create.html) 用于每日生成成本报告，并保存在报告桶中(以下命令假定 `report-definition.json` 位于当前工作目录中，如果保存在其他目录中，则应更新文件的正确位置).

```bash
aws cur put-report-definition --report-definition file://report-definition.json
```

6.最多等待 24 小时，直到生成第一份报告。运行以下 AWS CLI 命令，检查是否已创建 CUR 并将其添加到您的存储桶，确保用您在第一步中创建的存储桶名称更新以下命令中的 `AWS_BUCKET_NAME`: 

```bash
aws s3 ls s3://AWS_BUCKET_NAME/cost-reports/aws-monthly-cost-report-for-port/
```

如果上述命令至少返回一个以 CUR 创建次日的日期范围命名的目录，则报告已准备就绪，可以输入 Port。

### Port

1. 创建 `awsCost` 蓝图(下面的蓝图是一个示例，可根据需要进行修改): 

<details>
  <summary> AWS Cost Blueprint </summary>

```json showLineNumbers
{
  "identifier": "awsCost",
  "title": "AWS Cost",
  "icon": "AWS",
  "schema": {
    "properties": {
      "unblendedCost": {
        "title": "Unblended Cost",
        "type": "number",
        "description": "Represent your usage costs on the day they are charged to you. It’s the default option for analyzing costs."
      },
      "blendedCost": {
        "title": "Blended Cost",
        "type": "number",
        "description": "Calculated by multiplying each account’s service usage against a blended rate. This cost is not used frequently due to the way that it calculated."
      },
      "amortizedCost": {
        "title": "Amortized Cost",
        "type": "number",
        "description": "View recurring and upfront costs distributed evenly across the months, and not when they were charged. Especially useful when using AWS Reservations or Savings Plans."
      },
      "ondemandCost": {
        "title": "On-Demand Cost",
        "type": "number",
        "description": "The total cost for the line item based on public On-Demand Instance rates."
      },
      "payingAccount": {
        "title": "Paying Account",
        "type": "string"
      },
      "usageAccount": {
        "title": "Usage Account",
        "type": "string"
      },
      "product": {
        "title": "Product",
        "type": "string"
      },
      "billStartDate": {
        "title": "Bill Start Date",
        "type": "string",
        "format": "date-time"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {
    "link": {
      "title": "Link",
      "calculation": "if (.identifier | startswith(\"arn:\")) then \"https://console.aws.amazon.com/go/view?arn=\" + (.identifier | split(\"@\")[0]) else null end",
      "type": "string",
      "format": "url"
    }
  },
  "relations": {}
}
```

</details>

### 出口商

所有设置选项的输出器环境变量: 


| Env Var                        | Description                                                                  | Required | Default                                         |
| ------------------------------ | ---------------------------------------------------------------------------- | -------- | ----------------------------------------------- |
| PORT_CLIENT_ID                 | Your Port client id                                                          | **true** |                                                 |
| PORT_CLIENT_SECRET             | Your Port client secret                                                      | **true** |                                                 |
| PORT_BASE_URL                  | Port API base url                                                            | false    | `https://api.getport.io/v1`                     |
| PORT_BLUEPRINT                 | Port blueprint identifier to use                                             | false    | `awsCost`                                       |
| PORT_MAX_WORKERS               | Max concurrent threads that interacts with Port API (upsert/delete entities) | false    | `5`                                             |
| PORT_MONTHS_TO_KEEP            | Amount of months to keep the cost reports in Port                            | false    | `3`                                             |
| AWS_BUCKET_NAME                | Your AWS bucket name to store cost reports                                   | **true** |                                                 |
| AWS_COST_REPORT_S3_PATH_PREFIX | Your AWS cost report S3 path prefix                                          | false    | `cost-reports/aws-monthly-cost-report-for-port` |


#### 本地

1. 确保已安装 Python，并确保 Python 版本至少为 Python 3.11: 

```bash
python3 --version
```

2.根据你首选的克隆方法(下面的示例被用于了 SSH 克隆方法)克隆[port-aws-cost-exporter](https://github.com/port-labs/port-aws-cost-exporter) 版本库 ，然后将你的工作目录切换到该克隆版本库。

```bash showLineNumbers
git clone git@github.com:port-labs/port-aws-cost-exporter.git
cd port-aws-cost-exporter
```

3.创建新的虚拟环境并安装需求

```bash showLineNumbers
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

4.设置所需的环境变量并运行输出程序

```bash showLineNumbers
export PORT_CLIENT_ID=<PORT_CLIENT_ID>
export PORT_CLIENT_SECRET=<PORT_CLIENT_SECRET>
export AWS_BUCKET_NAME=<AWS_BUCKET_NAME>
export AWS_ACCESS_KEY_ID=<AWS_ACCESS_KEY_ID>
export AWS_SECRET_ACCESS_KEY=<AWS_SECRET_ACCESS_KEY>
python3 main.py
```

#### docker

1. 创建包含所需环境变量的 `.env` 文件

```
PORT_CLIENT_ID=<PORT_CLIENT_ID>
PORT_CLIENT_SECRET=<PORT_CLIENT_SECRET>
AWS_BUCKET_NAME=<AWS_BUCKET_NAME>
AWS_ACCESS_KEY_ID=<AWS_ACCESS_KEY_ID>
AWS_SECRET_ACCESS_KEY=<AWS_SECRET_ACCESS_KEY>
```

2.使用 `.env` 运行 docker 的 Docker 镜像

```bash
docker run -d --name getport.io-port-aws-cost-exporter --env-file .env ghcr.io/port-labs/port-aws-cost-exporter:latest
```

3.查看容器的 logging，观察进度: 

```bash
docker logs -f getport.io-port-aws-cost-exporter
```

#### GitHub 工作流程

1. 创建以下 GitHub 仓库 secrets: 

需要: 

* aws_access_key_id`
* 访问密钥
* Port客户 ID
* Port客户机密钥

排期所需: 

* aws_bucket_name`

2.被用于的 GitHub 工作流定义允许按计划或手动运行导出器: 

<details>
  <summary> GitHub Workflow run.yml </summary>

```yaml showLineNumbers
name: portAwsCostExporter

on:
  schedule:
    - cron: "0 0 * * *" # At 00:00 on every day
  workflow_dispatch:
    inputs:
      AWS_BUCKET_NAME:
        description: "The AWS Bucket name of the cost reports"
        type: string
        required: true

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - name: run
        uses: docker://ghcr.io/port-labs/port-aws-cost-exporter:latest
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_BUCKET_NAME: ${{ inputs.AWS_BUCKET_NAME || secrets.AWS_BUCKET_NAME }}
          PORT_CLIENT_ID: ${{ secrets.PORT_CLIENT_ID }}
          PORT_CLIENT_SECRET: ${{ secrets.PORT_CLIENT_SECRET }}
```

</details>

#### GitLab Pipeline

1. 创建以下 GitLab CI/CD 变量: 

:::tip 请参阅本指南，了解如何[setup a GitLab CI variable](https://docs.gitlab.com/ee/ci/variables/index.html#define-a-cicd-variable-in-the-ui)

:::

需要: 

* aws_access_key_id`
* aws_access_key_id`
* `aws_bucket_name`(服务器名称
* Port客户 ID
* `port_client_secret` Port客户密钥

2.被用于的 GitLab CI Pipeline 定义允许按计划或手动运行输出程序: 

<details>
  <summary> GitLab Pipeline gitlab-ci.yml </summary>

```yaml showLineNumbers
image: docker:latest

services:
  - docker:dind

variables:
  PORT_CLIENT_ID: $PORT_CLIENT_ID
  PORT_CLIENT_SECRET: $PORT_CLIENT_SECRET
  AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
  AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
  AWS_BUCKET_NAME: $AWS_BUCKET_NAME

stages:
  - run

run_job:
  stage: run
  script:
    - docker run -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY -e AWS_BUCKET_NAME=$AWS_BUCKET_NAME -e PORT_CLIENT_ID=$PORT_CLIENT_ID -e PORT_CLIENT_SECRET=$PORT_CLIENT_SECRET ghcr.io/port-labs/port-aws-cost-exporter:latest
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
      when: always
```

</details>

3.安排脚本时间: 
    1.进入 GitLab 仓库，从侧边栏菜单中选择**构建**。
    2.点击**Pipelines schedules**，然后点击**New Schedule**。
    3.在表单中填写计划详情: 描述、间隔模式、时区、目标分支。
    4.确保选中**已激活**复选框。
    5.单击 ** 创建 Pipelines 计划表** 创建计划表。
    提示
    建议将 Pipelines 计划为每天最多运行一次，因为 AWS 每天刷新一次数据。
    :::
