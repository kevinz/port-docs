---
sidebar_position: 5
---

import FindCredentials from "../../api/_template_docs/_find_credentials.mdx"

# 自主安装

:::note  先决条件

* 已在 Port 注册的机构；
* 您的 Port 用户角色设置为 "Admin"。

:::

对于自行安装 GitHub 的组织，无法访问我们的官方公共应用程序，因此需要采取一些额外步骤来安装 GitHub 应用程序: 

1. [Register](#register-ports-github-app) Port 在 GitHub 组织中的 GitHub 应用程序；
2. [Deploy](#deployment) Port 的 GitHub 应用程序在您的 VPC 中的 Docker 镜像；
3. [Install](#installing-ports-github-application) Port's GitHub 应用程序在您的 GitHub 组织和特定软件源中。

## 注册 Port 的 GitHub 应用程序

1. 在自我托管的 GitHub 中导航到你的组织，点击设置: 

![Org view](../../../../../static/img/integrations/github-app/SelfHostedOrganizaionView.png)

2.在设置视图中，点击开发者设置 -> 然后选择 GitHub 应用程序: 

![Settings view](../../../../../static/img/integrations/github-app/SelfHostedOrganizationSettings.png)

3.点击 "新建 GitHub 应用程序": 

![New GitHub App](../../../../../static/img/integrations/github-app/SelfHostedNewGitHubApp.png)

4.插入以下属性

* **GitHub 应用程序名称: ** Port.io
* **主页 URL:** https://getport.io
* **设置 URL:** https://app.getport.io
* **Webhook URL: ** HTTP 服务器 URL，如果您还不知道这一步的值，请留空，直到您部署了 GitHub 后端为止
* **Webhook secret: ** Webhook secret(任何你想要的字符串)
* **存储库权限: **
    - 操作: 读取和写入(用于使用 GitHub 工作流执行自助操作)
    - 检查: 读取和写入(用于验证 Port.yml)
    - 内容: 只读(用于读取Port配置文件和版本库文件)
    - 元数据: 只读
    - 问题: 只读
    - 拉取请求: 读写
    - Dependabot 警报只读
    - 管理: 只读(用于同步 github 团队)
* **组织权限: **
    - 成员: 只读(用于同步 Github 团队)
* **Repository Events**(接收来自 GitHub 的 webhook 变动所需的权限): 
    - 问题
    - 拉取请求
    - 推送
    - 工作流运行
    - 团队
    - Dependabot 警报

然后选择 "创建 GitHub 应用程序

5.进入已创建的 GitHub App 的设置，生成私钥并保存下载的文件: 

![Generate Private key](../../../../../static/img/integrations/github-app/SelfHosetdGeneratePrivayKey.png)

保留该文件，部署步骤将需要它。

## 部署

:::note  先决条件

您需要输入 Port `CLIENT_ID`和`CLIENT_SECRET`。

<FindCredentials/>

:::

如需使用[Self-Service Actions using GitHub Workflow](../../../../create-self-service-experiences/setup-backend/github-workflow/github-workflow.md) ，请通过 support@getport.io 与我们联系。

## Docker

要使用我们的 GitHub 应用程序，您需要在 VPC 上部署我们的 GitHub 应用程序官方 docker 镜像。

它可以部署在任何允许将镜像作为容器部署的平台上，如: K8s、ECS、AWS App Runner 等。

您可以通过运行以下命令拉取 Docker 镜像: 

```bash showLineNumbers
docker pull ghcr.io/port-labs/port-self-hosted-github-app:0.12.0
```

运行以下命令启动应用程序: 

```bash showLineNumbers
docker run \
  -e APP_ID=<APP_ID from register step> \
  -e WEBHOOK_SECRET=<WEBHOOK_SECRET from previous step> \
  -e GHE_HOST=<GITHUB BASE HOST, ie github.compay.com> \
  -e PORT=<Any PORT> \
  -e PORT_URL=https://api.getport.io \
  -e PORT_CLIENT_ID=<CLIENT_ID> \
  -e PORT_CLIENT_SECRET=<CLIENT_SECRET> \
  -e PRIVATE_KEY=<BASE 64 PRIVATEKEY> \
  -p <PORT>:<PORT> \
  ghcr.io/port-labs/port-self-hosted-github-app
```


| Env variable         | Description                                                                         |
| -------------------- | ----------------------------------------------------------------------------------- |
| `APP_ID`             | Application ID, you can find it in the edit GitHub App page                         |
| `WEBHOOK_SECRET`     | The same string that was used to register the application in the previous step      |
| `GHE_HOST`           | Your organization's self-hosted GitHub hostname                                     |
| `PORT`               | The port that the GitHub App will listen to                                         |
| `PORT_URL`           | Port's API Base URL                                                                 |
| `PORT_CLIENT_ID`     | Port client id for interacting with the API                                         |
| `PORT_CLIENT_SECRET` | Port client secret for interacting with the API                                     |
| `PRIVATE_KEY`        | A base64 encoded private key. You can use a tool like https://www.base64encode.org/ |


## 健康检查路线

健康检查是一种被用于来检查服务健康状况的途径，是确保服务正常运行并能执行其预期功能的一种手段。

我们的 GitHub 应用镜像在 `https://host:port/health` 公开了一个健康检查路由，以监控其状态。

## 安装 Port 的 GitHub 应用程序

在您的组织中注册了应用程序，Docker 也已启动并运行后，您就可以安装应用程序，并选择要与之集成的软件源: 

1. 首先，在自我托管的 GitHub 中导航到你的组织，然后点击设置: 

![Org view](../../../../../static/img/integrations/github-app/SelfHostedOrganizaionView.png)

2.在设置视图中，点击开发者设置 -> 然后选择 GitHub 应用程序: 

![Settings view](../../../../../static/img/integrations/github-app/SelfHostedOrganizationSettings.png)

3.在前一步创建的 GitHub 应用程序上点击 "编辑": 

![GitHub app installation page](../../../../../static/img/integrations/github-app/SelfHostedEditGitHubApp.png)

4.转到 "安装应用程序"->，选择您想要的组织的安装按钮；
5.选择要安装应用程序的软件源: 

![GitHub app installation chooses repositories](../../../../../static/img/integrations/github-app/SelfHostedInstallationRepoSelection.png)

## 限制

由于这是一个自托管版本，出于安全考虑会有一些限制，而且我们无法访问您的 GitHub 实例。

* 您必须将配置作为版本库的一部分，而且不能通过 Port 的用户界面/API 进行配置；
* 要使用自助服务操作，您需要为组织配置[Kafka Credentials](/create-self-service-experiences/setup-backend/webhook/kafka/kafka.md) ；
