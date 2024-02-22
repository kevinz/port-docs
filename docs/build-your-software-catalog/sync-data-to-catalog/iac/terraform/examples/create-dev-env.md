---

sidebar_position: 3
title: 管理开发环境生命周期
description: 管理开发人员环境并在 Port 中反映出来

---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# 管理开发环境生命周期

在本例中，您将在 AWS 中创建一个开发人员环境，然后将其信息作为开发人员环境实体报告给 Port。

我们的开发环境将包括

* 接收消息的 SQS 队列；
* 由发送到队列的消息触发的 Lambda 函数；
* 一个 S3 存储桶，用于存储 S3 函数生成的工件。

## 先决条件

既然我们要创建一个 lambda 函数，就需要提供它的源代码。您可以点击[here](https://port-docs-assets.s3.eu-west-1.amazonaws.com/lambda_function_payload.zip) 下载一个基本的 Nodejs Lambda 模板(请确保将 `.zip` 文件保存在 `main.tf` 文件旁边) 。

您需要创建一个开发人员环境蓝图来遵循这个示例: 

<Tabs groupId="blueprint" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "identifier": "developerEnvironment",
  "description": "",
  "title": "Developer Environment",
  "icon": "Environment",
  "schema": {
    "properties": {
      "bucketUrl": {
        "type": "string",
        "title": "Bucket URL",
        "format": "url"
      },
      "lambdaUrl": {
        "type": "string",
        "title": "Lambda URL",
        "format": "url"
      },
      "queueUrl": {
        "type": "string",
        "title": "Queue URL",
        "format": "url"
      },
      "memorySize": {
        "type": "number",
        "title": "Memory Size"
      }
    },
    "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</TabItem>

<TabItem value="terraform">

```hcl showLineNumbers
terraform {
  required_providers {
    port = {
      source  = "port-labs/port-labs"
      version = "~> 1.0.0"
    }
  }
}

provider "port" {
  client_id = "YOUR_CLIENT_ID"     # or set the environment variable PORT_CLIENT_ID
  secret    = "YOUR_CLIENT_SECRET" # or set the environment variable PORT_CLIENT_SECRET
}

resource "port_blueprint" "developer_environment" {
  identifier = "developerEnvironment"
  icon       = "Environment"
  title      = "Developer Environment"

  properties = {
    string_props = {
      "bucketUrl" = {
        title = "Bucket URL"
        format = "url"
        required   = false
      }
      "queueUrl" = {
        title = "Queue URL"
        format = "url"
        required   = false
      }
      "lambdaUrl" = {
        title = "Lambda URL"
        format = "url"
        required   = false
      }
    }
    number_props = {
      "memorySize" = {
        title = "Memory Size"
        required   = false
      }
    }
  }
}
```

</TabItem>

</Tabs>

下面是完整的 `main.tf` 文件: 

<details>
<summary>Complete Terraform definition file</summary>

```hcl showLineNumbers
terraform {
  required_providers {
    port = {
      source  = "port-labs/port-labs"
      version = "~> 1.0.0"
    }
  }
}

provider "aws" {
  access_key = "YOUR_ACCESS_KEY_ID"
  secret_key = "YOUR_SECRET_ACCESS_KEY"
  region     = "eu-west-1"
}

provider "port" {
  client_id = "YOUR_CLIENT_ID"     # or set the environment variable PORT_CLIENT_ID
  secret    = "YOUR_CLIENT_SECRET" # or set the environment variable PORT_CLIENT_SECRET
}

resource "aws_s3_bucket" "port_terraform_example_dev_env_bucket" {
  bucket = "my-port-terraform-example-dev-env-bucket"
}

resource "aws_s3_bucket_acl" "port_terraform_example_dev_env_bucket-acl" {
  bucket = aws_s3_bucket.port_terraform_example_dev_env_bucket.id
  acl    = "private"
}

resource "aws_sqs_queue" "port_terraform_example_dev_env_queue" {
  name                        = "terraform-example-queue.fifo"
  fifo_queue                  = true
  content_based_deduplication = true
}

resource "aws_iam_role" "iam_for_lambda" {
  name = "iam_for_lambda"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_role_policy_attachment" "attach_sqs_policy_for_lambda_role" {
  role       = aws_iam_role.iam_for_lambda.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaSQSQueueExecutionRole"
}

resource "aws_lambda_function" "port_terraform_example_dev_env_lambda" {
  depends_on = [
    aws_sqs_queue.port_terraform_example_dev_env_queue
  ]

  filename      = "lambda_function_payload.zip"
  function_name = "dev-env-function"
  role          = aws_iam_role.iam_for_lambda.arn
  handler       = "index.test"

  runtime = "nodejs16.x"
}

resource "aws_lambda_function_url" "example_function_url" {
  function_name      = aws_lambda_function.port_terraform_example_dev_env_lambda.function_name
  authorization_type = "NONE"
}

resource "aws_lambda_event_source_mapping" "event_source_mapping" {
  event_source_arn = aws_sqs_queue.port_terraform_example_dev_env_queue.arn
  enabled          = true
  function_name    = aws_lambda_function.port_terraform_example_dev_env_lambda.arn
  batch_size       = 1
}

resource "port_entity" "dev_env" {
  depends_on = [
    aws_s3_bucket.port_terraform_example_dev_env_bucket,
    aws_lambda_function.port_terraform_example_dev_env_lambda,
    aws_sqs_queue.port_terraform_example_dev_env_queue,
    aws_lambda_function_url.example_function_url
  ]

  identifier = aws_lambda_function.port_terraform_example_dev_env_lambda.function_name
  title      = aws_lambda_function.port_terraform_example_dev_env_lambda.function_name
  blueprint  = "developerEnvironment"

  properties = {
    string_props = {
      "bucketUrl" = "https://${aws_s3_bucket.port_terraform_example_dev_env_bucket.bucket_domain_name}"
      "queueUrl"  = aws_sqs_queue.port_terraform_example_dev_env_queue.id
      "lambdaUrl" = aws_lambda_function_url.example_function_url.function_url
    }
    number_props = {
      "memorySize" = aws_lambda_function.port_terraform_example_dev_env_lambda.memory_size
    }
  }
}
```

</details>

要自己使用这个示例，只需替换 `access_key`、`secret_key`、`client_id` 和 `secret` 的占位符，然后运行以下命令设置新后端、创建新基础架构并更新软件目录: 

```shell showLineNumbers
# install modules and create an initial state
terraform init
# To view Terraform's planned changes based on your .tf definition file:
terraform plan
# To apply the changes and update the catalog
terraform apply
```

让我们来分解定义文件，了解其中的各个部分: 

## 模块导入

这部分包括导入和设置所需的 Terraform Provider 和模块: 

```hcl showLineNumbers
terraform {
  required_providers {
    port = {
      source  = "port-labs/port-labs"
      version = "~> 1.0.0"
    }
  }
}

provider "aws" {
  access_key = "YOUR_ACCESS_KEY_ID"
  secret_key = "YOUR_SECRET_ACCESS_KEY"
  region     = "eu-west-1"
}

provider "port" {
  client_id = "YOUR_CLIENT_ID"     # or set the environment variable PORT_CLIENT_ID
  secret    = "YOUR_CLIENT_SECRET" # or set the environment variable PORT_CLIENT_SECRET
}
```

## 定义 AWS 资源

这部分包括定义以下 AWS 资源: 

* 一个 S3 桶；
* 一个 SQS FIFO 队列；
* 可访问新 SQS 队列的 Lambda 函数的 IAM 策略；
* 具有指向 SQS 队列的函数 URL 和事件源映射的 Lambda 函数。

```hcl showLineNumbers
resource "aws_s3_bucket" "port_terraform_example_dev_env_bucket" {
  bucket = "my-port-terraform-example-dev-env-bucket"
}

resource "aws_s3_bucket_acl" "port_terraform_example_dev_env_bucket-acl" {
  bucket = aws_s3_bucket.port_terraform_example_dev_env_bucket.id
  acl    = "private"
}

resource "aws_sqs_queue" "port_terraform_example_dev_env_queue" {
  name                        = "terraform-example-queue.fifo"
  fifo_queue                  = true
  content_based_deduplication = true
}

resource "aws_iam_role" "iam_for_lambda" {
  name = "iam_for_lambda"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

resource "aws_iam_role_policy_attachment" "attach_sqs_policy_for_lambda_role" {
  role       = aws_iam_role.iam_for_lambda.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaSQSQueueExecutionRole"
}

resource "aws_lambda_function" "port_terraform_example_dev_env_lambda" {
  depends_on = [
    aws_sqs_queue.port_terraform_example_dev_env_queue
  ]

  filename      = "lambda_function_payload.zip"
  function_name = "dev-env-function"
  role          = aws_iam_role.iam_for_lambda.arn
  handler       = "index.test"

  runtime = "nodejs16.x"
}

resource "aws_lambda_function_url" "example_function_url" {
  function_name      = aws_lambda_function.port_terraform_example_dev_env_lambda.function_name
  authorization_type = "NONE"
}

resource "aws_lambda_event_source_mapping" "event_source_mapping" {
  event_source_arn = aws_sqs_queue.port_terraform_example_dev_env_queue.arn
  enabled          = true
  function_name    = aws_lambda_function.port_terraform_example_dev_env_lambda.arn
  batch_size       = 1
}
```

## 创建与新环境相匹配的开发环境实体

这部分包括配置 `developerEnvironment` 蓝图，并为我们的新水桶创建一个实体: 

```hcl showLineNumbers
resource "port_entity" "dev_env" {
  depends_on = [
    aws_s3_bucket.port_terraform_example_dev_env_bucket,
    aws_lambda_function.port_terraform_example_dev_env_lambda,
    aws_sqs_queue.port_terraform_example_dev_env_queue,
    aws_lambda_function_url.example_function_url
  ]

  identifier = aws_lambda_function.port_terraform_example_dev_env_lambda.function_name
  title      = aws_lambda_function.port_terraform_example_dev_env_lambda.function_name
  blueprint  = "developerEnvironment"

  properties = {
    string_props = {
      "bucketName" = aws_s3_bucket.port_terraform_example_dev_env_bucket.id
      "queueUrl"   = aws_sqs_queue.port_terraform_example_dev_env_queue.id
      "lambdaUrl" = aws_lambda_function_url.example_function_url.function_url
    }
  }
}
```

:::info  Terraform 依赖项 请注意，我们在新开发环境实体上使用了 "depends_on "块，这是必需的，因为在创建实体之前，"developerEnvironment "蓝图必须存在。 此外，实体依赖的 Values 只有在创建 AWS 资源后才可用。

:::

## 结果

运行 "terraform apply "后，您将看到 AWS 中的资源和 Port 中匹配的 "developerEnvironment "实体。

继续阅读，了解如何进行更新和清理。

## 更新开发环境和匹配实体

由于新的开发者环境实体直接从 Terraform 中定义的 AWS 资源映射动态属性，因此当其中一个开发者环境资源发生变化时，它也会更新。

例如，让我们增加 Lambda 函数的内存大小: 

```hcl showLineNumbers
resource "aws_lambda_function" "port_terraform_example_dev_env_lambda" {
  depends_on = [
    aws_sqs_queue.port_terraform_example_dev_env_queue
  ]

  filename      = "lambda_function_payload.zip"
  function_name = "dev-env-function"
  role          = aws_iam_role.iam_for_lambda.arn
  handler       = "index.handler"

  runtime = "nodejs16.x"

  // highlight-next-line
  memory_size = 256 # was 128
}
```

现在运行 `terraform apply` 后，Lambda 资源和匹配实体的 `memorySize` 属性都会发生变化。

## 清理

要清理环境，可以运行 `terraform destroy` 命令，它将删除本例中创建的所有资源(包括 AWS 基础设施和匹配的 Port 实体)。