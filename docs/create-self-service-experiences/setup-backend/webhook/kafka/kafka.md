# Kafka 自助行动

Port 为每个客户管理一个发布执行运行请求的 Kafka Topic。

您可以使用任何代码平台监听 Kafka Topic，也可以将其用作无服务器功能的触发器。 例如，AWS Lambda。

![Port Kafka Architecture](/img/self-service-actions/portKafkaArchitecture.jpg)

上图所示步骤如下: 

1. Port 向 Kafka 发布调用的 "Action "或 "Change "消息；
2. 一个安全的 Kafka 主题会保存所有的操作调用和更改；

:::note  关于主题 行动和更改主题是分开的，它们的格式是

* 操作主题 - `ORG_ID.runs`
* 更改主题 - `ORG_ID.change.logging

:::

:::note  关于消费者组 作为设置的一部分，您需要创建一个消费者组来监听主题。 消费者组 id 可以是以下之一

* 任何以您的机构 ID 为前缀的组名，例如 `ORG_ID.my-group-name
* 与 Port 提供的用户名相匹配的组名

:::

3.客户端的监听器会接收新主题信息，并运行 DevOps 团队定义的代码。

:::info 例如，监听器可以是任何能够读取 Kafka 主题并根据接收到的消息运行代码的设备: 

* AWS Lambda；
* 从主题中读取的 Python 代码；
* 运行代码的 docker 容器。

您可以控制如何以最适合您的组织和基础设施的方式与这些主题互动。

:::

流程示例如下

1. 开发人员要求部署现有 "Microservice "的新版本；
2. 创建 "操作会被发送到 "runs "Kafka 主题；
3. 此新操作消息将触发 AWS Lambda 函数；
4. Lambda 函数会部署服务的新版本；
5. Lambda 完成后，向 Port 报告新的微服务 "部署 "情况。

## 下一步

要开始使用 webhook 自助服务操作，请查看下面的资料来源: 

* * [Setting up a basic execution runner using AWS Lambda](./examples/execution-basic-runner-using-aws-lambda.md)
* [Setting up a basic change log listener using AWS Lambda](./examples/changelog-basic-change-listener-using-aws-lambda.md)
