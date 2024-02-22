# GitLab Pipelines action

Port 的 GitLab Pipeline Action 可使用客户提供的输入和[`port_payload`](../../self-service-actions-deep-dive/self-service-actions-deep-dive.md#action-message-structure) 触发[GitLab Pipeline](https://docs.gitlab.com/ee/ci/pipelines/) 。

![Port Kafka Architecture](../../../../static/img/self-service-actions/setup-backend/gitlab-pipeline/gitlab-pipeline-agent-architecture.jpg)

上图所示步骤如下: 

1. Port 向主题发布调用的包含管道详细信息的 "Action "消息；
2. 安全主题("ORG_ID.runs")保存所有的操作调用；
3. Port 的执行代理从 Kafka 主题中提取新的触发事件，并触发 GitLab 管道。

## 进一步的步骤

* 请参阅 GitLab Pipelines 的[deployment example](./examples/run-service-deployment.md) 。
* 通过 Intercom 联系我们，为您的组织设置 Kafka 主题。
* [Install the Port execution agent to triggering the GitLab pipelines](./Installation.md).
* [Learn how to customize the payload sent to gitlab api](./Installation.md#control-the-payload).
