---

sidebar_position: 2
title: 管理 S3 Bucket 生命周期
description: 管理 S3 桶生命周期并在 Port 中反映出来

---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# 管理 S3 存储桶生命周期

在本例中，您将创建一个 AWS S3 存储桶，然后将其信息作为 S3 存储桶实体报告给 Port。

## 先决条件

您需要创建一个开发人员环境蓝图来遵循这个示例: 

<Tabs groupId="blueprint" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "terraform"}
]}>

<TabItem value="api">

```json showLineNumbers
{
  "identifier": "s3Bucket",
  "description": "",
  "title": "S3 Bucket",
  "icon": "Bucket",
  "schema": {
    "properties": {
      "isPrivate": {
        "type": "boolean",
        "title": "Is private?"
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

resource "port_blueprint" "s3_bucket" {
  identifier = "s3Bucket"
  icon       = "Bucket"
  title      = "S3 Bucket"

  properties = {
    boolean_props = {
      isPrivate = {
        title      = "Is private?"
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

resource "aws_s3_bucket" "port-terraform-example-bucket" {
  bucket = "my-port-terraform-example-bucket"
}

resource "aws_s3_bucket_acl" "port-terraform-example-bucket-acl" {
  bucket = aws_s3_bucket.port-terraform-example-bucket.id
  acl    = "private"
}

resource "port_entity" "s3_bucket" {
  depends_on = [
    aws_s3_bucket.port-terraform-example-bucket
  ]

  identifier = aws_s3_bucket.port-terraform-example-bucket.bucket
  title      = aws_s3_bucket.port-terraform-example-bucket.bucket
  blueprint  = "s3Bucket"

  properties = {
    string_props = {
      "isPrivate" = aws_s3_bucket_acl.port-terraform-example-bucket-acl.acl == "private" ? true : false
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

## 定义 S3 存储桶和存储桶 ACL

这部分包括定义 S3 存储桶和附加 ACL 策略: 

```hcl showLineNumbers
resource "aws_s3_bucket" "port-terraform-example-bucket" {
  bucket = "my-port-terraform-example-bucket"
}

resource "aws_s3_bucket_acl" "port-terraform-example-bucket-acl" {
  bucket = aws_s3_bucket.port-terraform-example-bucket.id
  acl    = "public-read"
}
```

## 创建与新桶匹配的 S3 桶实体

这部分包括配置 `s3Bucket` 蓝图并为我们的新水桶创建一个实体: 

```hcl showLineNumbers
resource "port_entity" "s3_bucket" {
# highlight-start
  depends_on = [
    aws_s3_bucket.port-terraform-example-bucket
  ]
# highlight-end

  identifier = aws_s3_bucket.port-terraform-example-bucket.bucket
  title      = aws_s3_bucket.port-terraform-example-bucket.bucket
  blueprint  = "s3Bucket"

  properties = {
    string_props = {
      "isPrivate" = aws_s3_bucket_acl.port-terraform-example-bucket-acl.acl == "private" ? true : false
    }
  }
}
```

:::info  Terraform 依赖项 请注意，我们在新的 s3 实体上使用了 "depends_on "块，因为该实体依赖的值只有在创建 S3 bucket 后才可用。

:::

## 结果

运行 "terraform apply "后，您将在 AWS 中看到新的 S3 bucket，并在 Port 中看到匹配的 "s3Bucket "实体。

继续阅读，了解如何进行更新和清理。

## 更新 S3 存储桶和匹配实体

请注意我们是如何定义水桶实体的 `isPrivate` 属性的: 

```hcl showLineNumbers
properties = {
    string_props = {
      "isPrivate" = aws_s3_bucket_acl.port-terraform-example-bucket-acl.acl == "private" ? true : false
    }
}
```

由于我们创建的初始水桶被配置为 `private`，因此该属性的值为 `true`。

让我们来修改水桶政策: 

```hcl showLineNumbers
resource "aws_s3_bucket_acl" "port-terraform-example-bucket-acl" {
  bucket = aws_s3_bucket.port-terraform-example-bucket.id
  // highlight-next-line
  acl    = "public-read" # Changed from "private"
}
```

现在运行 `terraform apply` 后，水桶策略和匹配实体的 `isPrivate` 属性都会改变。

## 清理

要清理环境，可以运行 `terraform destroy` 命令，它将删除本例中创建的所有资源(包括 S3 存储桶和匹配的 Port 实体)。