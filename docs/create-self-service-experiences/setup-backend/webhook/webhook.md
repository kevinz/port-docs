# Webhook 自助行动

## 概览

Port 可根据客户提供的 "URL "触发 webhook，既可触发 "Action "事件，也可触发 "Changelog "事件。

![Port Kafka Architecture](../../../../static/img/self-service-actions/portWebhookArchitecture.jpg)

上图所示步骤如下: 

1. Port 会调用 `Action` 或 `Changelog` 事件。
2. Port 使用 `HMAC-SHA-256` 对有效负载 + 时间戳进行签名，并将其放入请求头。
::info Webhook 安全性
使用请求头验证 webhook 请求有以下好处: 
    - 确保请求有效载荷未被篡改
    - 确保信息发送方是 Port
    - 确保收到的信息不是旧信息的重放
    要了解如何验证 webhook 请求，请参阅[Verifying Webhook Signature](../webhook/signature-verification) 页面。::: 
3.Port 通过 `POST` 请求将调用的 `Action` 或 `Changelog` 发布到客户定义的 `URL` 上。
4.客户端的监听器会接收新的 "POST "请求，并运行 DevOps 团队定义的代码。

:::info 例如，监听器可以是任何能够读取 Kafka 主题并根据接收到的消息运行代码的设备: 

* AWS Lambda；
* 从主题中读取的 Python 代码；
* 运行代码的 docker 容器。

您可以控制如何以最适合您的组织和基础设施的方式与 webhook 交互。

:::

流程示例如下

1. 开发人员要求部署现有 "Microservice "的新版本；
2. 创建 "操作会发送到定义的 "URL"；
3. 此新操作消息将触发 AWS Lambda 函数；
4. Lambda 函数会部署服务的新版本；
5. Lambda 完成后，向 Port 报告新的微服务 "部署 "情况。

## 配置

创建操作时，"后端 "步骤包括多个配置，您可以对其进行自定义: 

### HTTP 请求类型

默认情况下，"POST "请求将发送到指定的端点 URL。 您可以将请求更改为任何受支持的类型: 

![httpRequestType](/img/self-service-actions/setup-backend/httpRequestType.png)

### 同步执行与异步执行

默认情况下，该操作将**异步**执行，这意味着您的后端需要通过 API 明确发送 Port 的结果。

或者，您也可以将执行类型设置为 **同步**，这将使操作通过返回的 HTTP 状态代码和有效负载自动向 Port 报告结果。

## 下一步

要开始使用 webhook 自助服务操作，请查看下面的资料来源: 

### 示例

* * [Create an S3 bucket using Self-Service Actions](./examples/s3-using-webhook.md)
* [Webhook changelog listener](./examples/changelog-listener.md)
* [Provisioning software templates using Cookiecutter](./examples/software-templates.md)

###本地设置、调试和安全验证

* * [Debugging webhooks locally](./local-debugging-webhook.md)
* [Validating webhook signatures](./signature-verification.md)
