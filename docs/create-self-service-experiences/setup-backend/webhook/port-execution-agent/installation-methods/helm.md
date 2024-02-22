---
sidebar_position: 1
---

# Helm

本页面将指导您使用 Helm 在 Kubernetes 集群中安装 Port 执行代理。

:::info 您可以观察舵图和可用参数[here](https://github.com/port-labs/helm-charts/tree/main/charts/port-agent) 。

:::

:::note  先决条件

* [Helm](https://helm.sh) 必须安装才能使用海图。有关安装的更多详情，请参阅 Helm[documentation](https://helm.sh/docs/intro/install/) 。
* 与 Kafka 的连接凭证由 Port 提供给您。
* 如果要触发 GitLab Pipelines，需要有一个[GitLab trigger token](https://docs.gitlab.com/ee/ci/triggers/)

:::

## 安装

1. 使用以下命令添加 Port 的 Helm 软件源: 

```bash
helm repo add port-labs https://port-labs.github.io/helm-charts
helm repo update
```

然后，您可以运行 ` helm search repo port-labs` 查看图表。

2.使用以下命令安装 `port-agent` 图表: 

:::note 记住要替换 `YOUR_ORG_ID`、`YOUR_KAFKA_COMNSUMER_GROUP`、`YOUR_PORT_CLIENT_ID` 和 `YOUR_PORT_CLIENT_SECRET`的占位符。

:::

```bash showLineNumbers
helm install my-port-agent port-labs/port-agent \
    --create-namespace --namespace port-agent \
    --set env.normal.PORT_ORG_ID=YOUR_ORG_ID \
    --set env.normal.KAFKA_CONSUMER_GROUP_ID=YOUR_KAFKA_CONSUMER_GROUP \
    --set env.secret.PORT_CLIENT_ID=YOUR_PORT_CLIENT_ID \
    --set env.secret.PORT_CLIENT_SECRET=YOUR_PORT_CLIENT_SECRET
```

## 接下来的步骤

* 请参考[usage guide](/create-self-service-experiences/setup-backend/webhook/port-execution-agent/usage.md) 设置 webhook。
