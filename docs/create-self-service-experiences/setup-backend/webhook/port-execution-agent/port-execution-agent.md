---
sidebar_position: 1
---

# Port执行代理

我们的执行代理为您提供了一种安全、便捷的方式，用于监听和执行自助服务操作的调用以及软件目录中的更改。

被用于执行代理后，就不需要公开一个公共端点供 Port 联系。

:::tip  公共存储库 我们的 Port 代理是开源的 - 请参见[**here**](https://github.com/port-labs/port-agent)

:::

:::note 要使用执行代理，请通过 Intercom 联系我们，以获得专用的 Kafka 主题。

:::

被用于 Port 执行代理时的数据流如下: 

* 调用新的自助服务操作或更改软件目录；
* Port 将调用事件发送到专用的 Kafka 主题；
* 执行代理从您的 Kafka 主题中提取新的调用事件，并将其发送到您指定的 URL。

![Port Agent Architecture](/img/self-service-actions/portExecutionAgentArchitecture.png)

## 下一步

* * [Explore How to install and use the agent](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/installation-methods/helm.md)
* [Control the payload sent to your endpoint](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/control-the-payload.md)
