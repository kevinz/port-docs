---

sidebar_position: 1
title: 塑造 C4 模式的后台
description: 在 Port 中创建 C4 模型的核心布局

---

# Modeling The Backstage C4 Model

<!-- TODO: complete and reveal this example when the Terraform provider supports creating entities with a many relation -->

在本例中，您将创建[Backstage C4 model](https://www.getport.io/blog/using-backstages-c4-model-adaptation-to-visualize-software-creating-a-software-catalog-in-port) 的基本布局。

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

provider "port" {
  #   client_id = "YOUR_CLIENT_ID"     # or set the environment variable PORT_CLIENT_ID
  #   secret    = "YOUR_CLIENT_SECRET" # or set the environment variable PORT_CLIENT_SECRET
}

resource "port_blueprint" "component" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api
  ]

  identifier = "component"
  icon       = "Cloud"
  title      = "Component"

  properties {
    string_props = {
      "type" = {
        title      = "Type"
        required   = false
        type       = "string"
        enum       = ["service", "library"]
        enum_colors = {
          "service" = "blue",
          "library" = "green"
        }
      }
    }
  }

  relations = {
  "system" = {
    target     = "system"
    required   = false
    many       = false
    title      = "System"
    }
  }
  relations {
    identifier = "resource"
    target     = "resource"
    required   = false
    many       = true
    title      = "Resources"
  }
  relations {
    identifier = "cosnumesApi"
    target     = "api"
    required   = false
    many       = true
    title      = "Consumes API"
  }
  relations {
    identifier = "component"
    target     = "component"
    required   = false
    many       = true
    title      = "Components"
  }
  relations {
    identifier = "providesApi"
    target     = "api"
    required   = false
    many       = false
    title      = "Provides API"
  }
}
resource "port_blueprint" "resource" {
  identifier = "resource"
  icon       = "DevopsTool"
  title      = "Resource"

  properties {
    identifier = "type"
    title      = "Type"
    required   = false
    type       = "string"
    enum       = ["postgres", "kafka-topic", "rabbit-queue", "s3-bucket"]
  }
}

resource "port_blueprint" "api" {
  identifier = "api"
  icon       = "Link"
  title      = "API"

  properties {
    identifier = "type"
    title      = "Type"
    required   = false
    type       = "string"
    enum       = ["Open API", "gRPC"]
  }
}

resource "port_blueprint" "domain" {
  identifier = "domain"
  icon       = "Server"
  title      = "Domain"

  properties {
    identifier = "active"
    title      = "Active?"
    required   = false
    type       = "boolean"
  }
}

resource "port_blueprint" "system" {
  depends_on = [
    port_blueprint.domain
  ]

  identifier = "system"
  icon       = "DevopsTool"
  title      = "System"

  properties {
    identifier = "active"
    title      = "Active?"
    required   = false
    type       = "boolean"
  }

  relations {
    identifier = "domain"
    target     = "domain"
    required   = false
    many       = false
    title      = "Domain"
  }
}

resource "port_entity" "orderDomain" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
  ]

  identifier = "orders"
  title      = "Orders"
  blueprint  = port_blueprint.domain.identifier

  properties {
    name  = "active"
    value = true
  }
}

resource "port_entity" "cartSystem" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.orderDomain,
  ]

  identifier = "cart"
  title      = "Cart"
  blueprint  = port_blueprint.system.identifier

  properties {
    name  = "active"
    value = true
  }

  relations {
    name       = "domain"
    identifier = port_entity.orderDomain.identifier
  }
}

resource "port_entity" "productsSystem" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.orderDomain,
  ]

  identifier = "product"
  title      = "Products"
  blueprint  = port_blueprint.system.identifier

  properties {
    name  = "active"
    value = true
  }

  relations {
    name       = "domain"
    identifier = port_entity.orderDomain.identifier
  }
}

resource "port_entity" "cartResource" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.orderDomain,
  ]

  identifier = "cartSqlDb"
  title      = "Cart SQL Database"
  blueprint  = port_blueprint.resource.identifier

  properties {
    name  = "type"
    value = "postgres"
  }
}

resource "port_entity" "cartApi" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.orderDomain,
  ]

  identifier = "cartApi"
  title      = "Cart API"
  blueprint  = port_blueprint.api.identifier

  properties {
    name  = "type"
    value = "Open API"
  }
}

resource "port_entity" "coreKafkaLibraryComponent" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.cartSystem,
  ]

  identifier = "coreKafkaLibrary"
  title      = "Core Kafka Library"
  blueprint  = port_blueprint.component.identifier

  properties {
    name  = "type"
    value = "library"
  }
}

resource "port_entity" "corePaymentLibraryComponent" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.cartSystem,
  ]

  identifier = "coreKafkaLibrary"
  title      = "Core Kafka Library"
  blueprint  = port_blueprint.component.identifier

  properties {
    name  = "type"
    value = "library"
  }

  relations {
    name       = "system"
    identifier = port_entity.cartSystem.identifier
  }
}

resource "port_entity" "cartService" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.cartSystem,
  ]

  identifier = "cartService"
  title      = "Cart Service"
  blueprint  = port_blueprint.component.identifier

  properties {
    name  = "type"
    value = "service"
  }

  relations {
    name       = "system"
    identifier = port_entity.cartSystem.identifier
  }
  relations {
    name = "resource"
    identifier = [
      port_entity.cartResource.identifier
    ]
  }
  relations {
    name = "component"
    identifier = [
      port_entity.coreKafkaLibraryComponent.identifier,
      port_entity.corePaymentLibraryComponent.identifier
    ]
  }
}

resource "port_entity" "productService" {
  depends_on = [
    port_blueprint.system,
    port_blueprint.resource,
    port_blueprint.api,
    port_blueprint.domain,
    port_blueprint.component,
    port_entity.cartSystem,
  ]

  identifier = "productService"
  title      = "Product Service"
  blueprint  = port_blueprint.component.identifier

  properties {
    name  = "type"
    value = "service"
  }

  relations {
    name       = "system"
    identifier = port_entity.productsSystem.identifier
  }
  relations {
    name       = "consumesApi"
    identifier = port_entity.cartApi.identifier
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

## 创建 S3 存储桶蓝图和与新存储桶匹配的实体

这部分包括配置 `s3Bucket` 蓝图并为我们的新水桶创建一个实体: 

```hcl showLineNumbers
resource "port_blueprint" "s3_bucket" {
  identifier = "s3Bucket"
  icon       = "Bucket"
  title      = "S3 Bucket"

  properties {
    identifier = "isPrivate"
    title      = "Is private?"
    required   = false
    type       = "boolean"
  }
}

resource "port_entity" "s3_bucket" {
# highlight-start
  depends_on = [
    aws_s3_bucket.port-terraform-example-bucket,
    port_blueprint.s3_bucket
  ]
# highlight-end

  identifier = aws_s3_bucket.port-terraform-example-bucket.bucket
  title      = aws_s3_bucket.port-terraform-example-bucket.bucket
  blueprint  = port_blueprint.s3_bucket.identifier

  properties {
    name  = "isPrivate"
    value = aws_s3_bucket_acl.port-terraform-example-bucket-acl.acl == "private" ? true : false
  }
}
```

:::info  Terraform 依赖项 请注意我们如何在新的 s3 实体上引用了 "depends_on "块，这是必需的，因为 "s3Bucket "蓝图必须在实体创建前就存在。 此外，实体依赖的 Values 只有在 S3 bucket 创建后才可用。

:::

## 结果

运行 "terraform apply "后，您将在 AWS 中看到新的 S3 bucket，并在 Port 中看到匹配的 "s3Bucket "实体。