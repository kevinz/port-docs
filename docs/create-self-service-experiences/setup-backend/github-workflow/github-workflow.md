# GitHub 工作流自助服务操作

[Port's GitHub application](../../../build-your-software-catalog/sync-data-to-catalog/git/github/installation.md) 可使用客户提供的输入和[`port_payload`](../../self-service-actions-deep-dive/self-service-actions-deep-dive.md#action-message-structure) 触发[GitHub workflow](https://docs.github.com/en/actions/using-workflows) 。

:::tip 使用 GitHub 工作流的自助操作既可通过标准的 Port[GitHub app](../../../build-your-software-catalog/sync-data-to-catalog/git/github/github.md) ，也可通过[self-hosted version](../../../build-your-software-catalog/sync-data-to-catalog/git/github/self-hosted-installation.md)

:::

![Port GitHub workflow Architecture](../../../../static/img/self-service-actions/portGithubWorkflowArchitecture.png)

上图所示步骤如下: 

1. Port将调用的 "操作 "信息发布到主题；
2. 安全主题 (`ORG_ID.github.runs`)保存所有调用的操作；
3. Port 的 GitHub 应用程序上的监听器会接收新的主题消息，并运行 DevOps 团队定义的 GitHub 工作流。

流程示例如下

1. 开发人员要求部署现有 "Microservice "的新版本；
2. 创建 "操作会发送到 "github.runs "主题；
3. Port的 GitHub 应用程序事件处理程序会被该新操作消息触发；
4. Port 的 GitHub 应用程序触发部署新版本服务的 GitHub 工作流；
5. 作为工作流的一部分，新的微服务 "部署 "将报告给 Port；
6. 工作流结束后，Port 的 GitHub 应用程序会根据工作流的 "结论 "向 Port 报告运行状态("成功 "或 "失败")。

:::info  触发工作流链 使用 `workflow_dispatch` 触发器触发的工作流是自包含的。 这意味着它在资源库上的操作和效果不能触发其他自动工作流。

1. 开发人员调用 "在 monorepo 中提供新的微服务 "工作流；
2. 工作流根据预定义模板在目标资源库中打开一个新 PR；
3. 该版本库也有一个工作流，该工作流被 "on: pull_request: types: "opened"` trigger；
4. 在这种情况下，不会触发自动 PR 工作流。

:::

## 示例

有关使用 Github 工作流程后端实现各种用例的情况，请参阅[examples](/create-self-service-experiences/setup-backend/github-workflow/examples/) 页面。

更多示例请参见我们的网站[action examples Github repository](https://github.com/port-labs/self-service-actions-examples) 。