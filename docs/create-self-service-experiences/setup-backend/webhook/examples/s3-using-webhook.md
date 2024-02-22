---
sidebar_position: 1
---

# 使用自助服务操作创建 S3 存储桶

在本指南中，我们将介绍使用 Port 自助操作创建 S3 存储桶(例如)的不同方法。

:::note 在整个教程中，我们将使用[webhook-actions](../webhook.md) 和一个简单的后端来监听 webhook 事件。

您也可以选择被用于[kafka-actions](/create-self-service-experiences/setup-backend/webhook/kafka/kafka.md) 。

本例中显示的每个操作都会在 Port 中创建一个新实体，并更新操作运行信息，以便跟踪操作状态及其结果。

:::

## 使用 Git 操作

Git 操作，如 "git commit"，可被用来引用工作流。

在工作流程中，我们可以利用基础设施即代码工具(Terraform、Pulumi、AWS CloudFormation......)来配置新资源。

在本[example](https://github.com/port-labs/port-action-runner-examples/tree/main/python/s3_bucket_creation/terraform_github_workflow/webhook) 中，您将学习如何使用 GitHub pull-request、GitHub 工作流和 Terraform 创建 S3 bucket。

## 利用 CI 工作

现有/新的 CI 作业可用于处理自助服务操作。

一些 CI 工具(GitHub、Jenkins 等)有内置集成，可以轻松配置新资源。

请查看[example](https://github.com/port-labs/port-action-runner-examples/tree/main/python/s3_bucket_creation/github_action/webhook) ，了解如何使用公共市场中的 GitHub 操作来实现 S3 bucket 的创建。

## Provision your backend

一个有效的选择是配置自己的后端来处理自助服务操作的 webhook。

使用这种方法，您可以选择任何代码语言和库来实现您的操作。

下面是一个[example](https://github.com/port-labs/port-action-runner-examples/tree/main/python/s3_bucket_creation/aws_sdk/webhook) ，用 FastAPI(python)提供了一个简单的后端 ，并用[AWS SDK](https://aws.amazon.com/sdk-for-python/) 创建了一个新的 S3 bucket。

## 摘要

在本例中，我们向你介绍了创建 S3 存储桶的几种替代方法。

毋庸赘言，您可以使用 webhook 执行任何操作，创建任何所需的资源。 例如: 

* 创建 EC2；
* 配置 K8s 集群；
* 部署新的微服务版本；
* 使 Cloudfront 缓存失效；
* 等等。
