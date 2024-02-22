---
sidebar_position: 4
---

# Terraform: 无代码资源调配

Port 的 Terraform Provider 与 Port 的[self-service actions](../../../create-self-service-experiences.md) 结合使用，可向用户公开 IaC 无代码资源调配操作。这些调配操作可自动执行不同云资源的生命周期。

使用 IaC 无代码资源调配，开发人员无需了解相应的系统、工具和实践(如 Terraform)，就能轻松管理资源。

让我们回顾一个例子，看看如何利用 Port 的自助操作来引用 IaC 无代码资源调配。

## 示例

下面的示例将演示如何设置自助操作，以创建 S3 存储桶、更改其 ACL(Day-2 操作)和删除它们。

该示例被引用了一个用 Go 编写的后端，它利用 Terraform 模板来执行上述操作。

:::info 完整示例见[**HERE**](https://github.com/port-labs/terraform-connector) 。

另一个用于管理 "SNS 主题 "的模板也可在链接的资源库中找到。

:::

:::note  先决条件

* 具有创建、更改 ACL 和删除 S3 存储桶权限的 AWS 凭据。
* 用于运行后端镜像的 Docker 守护进程。

:::

### 运行后端

首先，用以下命令运行后端镜像(确保替换占位符): 

```shell
docker run \
  -e PORT_CLIENT_ID=<PORT_CLIENT_ID> \
  -e PORT_CLIENT_SECRET=<PORT_CLIENT_SECRET> \
  -e AWS_ACCESS_KEY_ID=<AWS_ACCESS_KEY_ID> \
  -e AWS_SECRET_ACCESS_KEY=<AWS_SECRET_ACCESS_KEY> \
  -e AWS_SESSION_TOKEN=<AWS_SESSION_TOKEN> \
  -e AWS_REGION=<AWS_REGION> \
  -e DEBUG=true \
  -p 8080:8080 \
  -it ghcr.io/port-labs/webhook-terraform:v0.2
```

现在，您有了一个正在运行的服务器，它可以接收来自 Port 的 webhook 请求，并将它们应用到 terraform 文件中！

要将本地计算机公开给 Port，可以引用[ngrok](https://ngrok.com/download) 或[smee](https://smee.io/) 。

Port's[local webhook debugging guide](../local-debugging-webhook.md#creating-the-vm-create-action) 包含如何使用 `smee`的参考资料。

否则，如果选择 "ngrok"，则运行: 

```shell
ngrok http 8080
```

您将看到类似下面的 Output: 

```shell
ngrok

Session Status online
...
Forwarding https://1234-5678-9101-112-1314-1516-abcd-efgh-ijkl.eu.ngrok.io -> http://localhost:8080
...
```

保留 "转发 URL "以备后用。

#### 设置Port资源

首先，在 Port 中为 S3 桶设置[blueprint](../../../../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md) 。

您可以为水桶创建任意数量的[properties](../../../../build-your-software-catalog/define-your-data-model/setup-blueprint/properties/properties.md) ，但本示例只使用了 4 个属性--`URL`、`水桶名称`、`水桶 ACL` 和`标签`。

<details>
<summary> AWS bucket blueprint </summary>

```json showLineNumbers
{
  "identifier": "s3_bucket",
  "title": "AWS Bucket",
  "icon": "Bucket",
  "schema": {
    "properties": {
      "url": {
        "type": "string",
        "title": "URL",
        "format": "url"
      },
      "bucket_name": {
        "type": "string",
        "title": "Bucket Name"
      },
      "bucket_acl": {
        "type": "string",
        "title": "Bucket ACL",
        "default": "private"
      },
      "tags": {
        "type": "object",
        "title": "Tags"
      }
    },
    "required": ["url", "bucket_name"]
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>

接下来，我们要创建自助服务操作，以支持 "创建"、"更改 ACL "和 "删除 "S3 存储桶。

<details>
<summary> Self-service actions for AWS bucket blueprint </summary>

将 `<YOUR_WEBHOOK_URL>` 替换为之前从 Ngrok 或 Smee 处获得的 URL。

```json showLineNumbers
[
  {
    "identifier": "create_bucket",
    "title": "Create",
    "icon": "Bucket",
    "userInputs": {
      "properties": {
        "bucket_name": {
          "type": "string",
          "title": "Name for the S3 bucket"
        },
        "tags": {
          "type": "object",
          "title": "Tags",
          "default": {}
        }
      },
      "required": ["bucket_name"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<YOUR_WEBHOOK_URL>"
    },
    "trigger": "CREATE",
    "description": "Create a new S3 Bucket in AWS"
  },
  {
    "identifier": "change_acl",
    "title": "Change ACL",
    "icon": "Bucket",
    "userInputs": {
      "properties": {
        "bucket_acl": {
          "type": "string",
          "enum": ["private", "public-read"],
          "title": "ACL"
        }
      },
      "required": ["bucket_acl"]
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<YOUR_WEBHOOK_URL>"
    },
    "trigger": "DAY-2",
    "description": "Change S3 Bucket ACL"
  },
  {
    "identifier": "delete_bucket",
    "title": "Delete",
    "icon": "Bucket",
    "userInputs": {
      "properties": {},
      "required": []
    },
    "invocationMethod": {
      "type": "WEBHOOK",
      "url": "<YOUR_WEBHOOK_URL>"
    },
    "trigger": "DELETE",
    "description": "Delete an S3 Bucket from AWS"
  }
]
```

</details>

### 运行自助服务action

#### 创建

一切准备就绪，可以运行已配置的自助服务操作。

转到 "AWS Bucket "蓝图页面，运行 "创建 AWS Bucket":  ！[create-bucket-button.png](../../../../../static/img/complete-use-cases/iac-templates/create-bucket-button.png)

填写 S3 存储桶的名称(必须是全局唯一的！)，然后点击 "创建": 

![create-bucket-form.png](../../../../../static/img/complete-use-cases/iac-templates/create-bucket-form.png)

好耶！马上就会有一个新的 S3 存储桶，它也会被添加为 Port 实体。

![created-bucket.png](../../../../../static/img/complete-use-cases/iac-templates/created-bucket.png)

#### 更改 ACL(第 2 天操作)

创建水桶后，随着时间的推移，您可能需要更改其配置。

例如，一个有效的用例是将水桶的可见性从 "private "改为 "public-read"。

转到水桶实体，选择 "更改 ACL "第 2 天操作: 

![change-acl-button.png](../../../../../static/img/complete-use-cases/iac-templates/change-acl-button.png)

为 "ACL "选择 "public-read "选项，然后选择 "Execute": 

![change-acl-form.png](../../../../../static/img/complete-use-cases/iac-templates/change-acl-form.png)

`Terraform apply` 将在幕后触发，完成后，你会看到实体的 `Bucket ACL` 属性更新为 `public-read` 。

#### 删除

最后，您可以清除环境并删除水桶。

转到水桶实体，选择 "删除": 

![delete-bucket-button.png](../../../../../static/img/complete-use-cases/iac-templates/delete-bucket-button.png)

点击 "删除": 

![delete-bucket-modal.png](../../../../../static/img/complete-use-cases/iac-templates/delete-bucket-modal.png)

完成！您的水桶将从 AWS 和 Port 中删除。

## 摘要

IaC 无代码资源调配可帮助您的团队有效控制和配置所拥有的任何云资源。

Port 自助操作可让您快速升级基于事件的基础架构，从而充分利用 IaC 无代码资源调配工作流。