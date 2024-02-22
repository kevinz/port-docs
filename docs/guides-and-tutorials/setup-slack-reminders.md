---
sidebar_position: 7
title: 为记分卡发送 Slack 提醒

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# Slack 记分卡提醒

本指南只需 7 分钟即可完成，旨在展示以下内容: 

* 如何使用记分卡启动组织变革
* 如何使用 Port 的自助服务操作自动执行 Slack 提醒

:::tip  先决条件

* 本指南假定您已拥有 Port 账户，并已完成[onboarding process](/quickstart) 。我们将使用入职过程中创建的 "服务 "蓝图。
* 您需要一个 Git 仓库，在其中放置本指南将使用的工作流/Pipelines。如果没有，建议创建一个名为 `Port-actions` 的新仓库。

:::

<br/>

### 本指南的目标

在本指南中，我们将创建一个自助操作，为未完成的[scorecard rules](https://docs.getport.io/promote-scorecards/#what-is-a-scorecard) 发送 Slack 提醒。实际上，这样的操作可被研发经理/平台工程师用来提醒开发人员未达到的标准。

完成这项工作后，你就会了解它如何使你的组织中的不同角色受益: 

* 开发人员将收到平台工程师设定的需要修复的政策通知。
* 研发经理和平台工程师将能提醒开发人员服务中未满足的要求。

<br/>

#### 设置动作的前端

让我们从创建动作的前端开始。 这部分将被开发人员用来触发动作: 

<Tabs groupId="git-provider" queryString values={[
{label: "Github", value: "github"},
{label: "GitLab", value: "gitlab"}
]}>

<TabItem value="github" label="Github">

:::tip  入职

作为入职流程的一部分，您的[self-service tab](https://app.getport.io/self-serve) 中应该已经有一个名为 "发送记分卡提醒 "的操作。在这种情况下，您可以跳到[Define action type](#define-backend-type) 步骤。

如果您跳过了***登录，或者您想从头开始创建操作，请完成以下 1-4 步骤。

:::

<details>
<summary><b>Create the action's frontend (steps 1-4)</b></summary>

1. 要开始操作，请访问 Port 应用程序中的[Self-service tab](https://app.getport.io/self-serve) ，然后点击 "新建操作": 

<img src='/img/guides/actionsCreateNew.png' width='50%' />

2.Port 中的每个操作都与一个<PortTooltip id="blueprint">蓝图</PortTooltip>直接相关。由于我们要发送的是服务提醒，因此将从下拉菜单中引用我们在[quickstart guide](/quickstart) 中创建的 "服务 "蓝图。
3.像这样填写操作的基本细节，然后单击 "下一步": 

<img src='/img/guides/actionReminderBasicDetails.png' width='60%' />

4.再次单击 "下一步"，因为在此操作中我们不需要用户输入信息。

</details>

#### 定义后端类型

现在我们来定义动作的后端: 

* 选择 "Github 工作流 "作为调用类型。
* 用您的 Values 替换 `Organization` 和 `Repository` 值(这是工作流将驻留和运行的位置)。
* 将工作流命名为 `portSlackReminder.yaml`。
* 像这样填写表单的其余部分，然后单击 `下一步`: 

<img src='/img/guides/slackReminderBackend.png' width='70%' />

<br/><br/>

最后一步是自定义动作的权限。 为简单起见，我们将使用默认设置。 欲了解更多信息，请参阅[permissions](/create-self-service-experiences/set-self-service-actions-rbac/) 页面。 点击 "创建"。

</TabItem>

<TabItem value="gitlab" label="GitLab">

:::tip  入职

作为入职流程的一部分，您的[self-service tab](https://app.getport.io/self-serve) 中应该已经有一个名为 "发送记分卡提醒 "的操作。在这种情况下，您可以跳到[Define action type](#define-backend-type) 步骤。

如果您跳过了***登录，或者您想从头开始创建操作，请完成以下 1-4 步骤。

:::

<details>
<summary><b>Create the action's frontend (steps 1-4)</b></summary>

1. 要开始操作，请访问 Port 应用程序中的[Self-service tab](https://app.getport.io/self-serve) ，然后点击 "新建操作": 

<img src='/img/guides/actionsCreateNew.png' width='50%' />

2.Port 中的每个操作都与一个<PortTooltip id="blueprint">蓝图</PortTooltip>直接相关。由于我们要发送的是服务提醒，因此将从下拉菜单中引用我们在[quickstart guide](/quickstart) 中创建的 "服务 "蓝图。
3.像这样填写操作的基本细节，然后单击 "下一步": 

<img src='/img/guides/actionReminderBasicDetails.png' width='60%' />

4.再次单击 "下一步"，因为在此操作中我们不需要用户输入信息。

</details>

现在我们来定义动作的后端: 

* 选择 "触发 Webhook URL "作为 "调用类型"。
* 端点 URL "暂时留空，我们将在下一节创建它，然后回来更新。
* 像这样填写表单的其余部分，然后点击`下一步`: 

<img src='/img/guides/slackReminderBackendGitLab.png' width='70%' />

<br/><br/>

最后一步是自定义动作的权限。 为简单起见，我们将使用默认设置。 欲了解更多信息，请参阅[permissions](/create-self-service-experiences/set-self-service-actions-rbac/) 页面。 点击 "创建"。

</TabItem>

</Tabs>

action的前端已准备就绪 🥳

<br/>

#### 设置action的后端

现在，我们要编写我们的操作将触发的逻辑: 

<Tabs groupId="git-provider" queryString values={[
{label: "Github", value: "github"},
{label: "GitLab", value: "gitlab"}
]}>

<TabItem value="github" label="Github">

1. 首先，让我们创建必要的 token 和 secrets: 
    - 转到所需的 Slack 频道，然后[setup incoming webhooks](https://api.slack.com/messaging/webhooks) 。确保复制 webhook URL，我们将在 Github 工作流程中使用它。
    - 进入[Port application](https://app.getport.io/) ，点击右上角的"..."，然后点击 "Credentials"。复制 "客户端 ID "和 "客户端 secret"。
2.在工作流程所在的版本库中，在 "设置 -> secret和变量 -> 操作 "下创建 3 个新secret: 

* `SLACK_WEBHOOK_URL` - 目标频道的 Slack Webhook URL。
* `PORT_CLIENT_ID` - 从 Port 应用程序复制的客户端 ID。
* `PORT_CLIENT_SECRET` - 从 Port 应用程序复制的客户端secret。

<img src='/img/guides/repositorySecretSlack.png' width='80%' />

<br/><br/>

3.现在，让我们创建包含逻辑的工作流文件。在`.github/workflows`下，创建一个名为`portSlackReminder.yaml`的新文件，并使用以下代码段作为其内容: 

<details>
<summary><b>Github workflow (click to expand)</b></summary>

```yaml showLineNumbers
# portSlackReminder.yaml

name: Generate Scorecards Reminders
on:
  workflow_dispatch:
    inputs:
      port_payload:
        required: true
        type: string
jobs:
    generate-scorecards-reminders:
        runs-on: ubuntu-latest
        steps:
            - name: Generate Scorecards Reminders
              uses: port-labs/port-sender@v0.2.3
              with:
                operation_kind: scorecard_reminder
                port_client_id: ${{ secrets.PORT_CLIENT_ID }}
                port_client_secret: ${{ secrets.PORT_CLIENT_SECRET }}
                slack_webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
                blueprint: service
                scorecard: ProductionReadiness
                target_kind: slack
            - name: Report status to Port
              uses: port-labs/port-github-action@v1
              with:
                clientId: ${{ secrets.PORT_CLIENT_ID }}
                clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
                operation: PATCH_RUN
                runId: ${{ fromJson(inputs.port_payload).context.runId }}
                logMessage: |
                    Slack reminder sent successfully 🚀
```

</details>

:::tip  Port Initiatives sender Github action 此工作流程被引用到 Port 的[Initiatives Sender GitHub Action](https://github.com/marketplace/actions/port-sender) 中，用于发送 Slack 消息。

:::

</TabItem>

<TabItem value="gitlab" label="GitLab">

1. 首先，让我们创建所需的 webhook 和变量: 

* 转到所需的 Slack 频道，然后[setup incoming webhooks](https://api.slack.com/messaging/webhooks) 。确保复制 webhook URL，我们将在 Github 工作流程中使用它。
* 进入[Port application](https://app.getport.io/) ，点击右上角的"..."，然后点击 "凭据"。复制 "客户端 ID "和 "客户端 secret"。

2.在管道所在的 GitLab 项目中，在 "设置->CI/CD->变量 "下创建 3 个新变量: 

* `SLACK_WEBHOOK_URL` - 目标频道的 Slack Webhook URL。
* `PORT_CLIENT_ID` - 从 Port 应用程序复制的客户端 ID。
* `PORT_CLIENT_SECRET` - 从 Port 应用程序复制的客户端secret。

<img src='/img/guides/repositorySecretSlackGitLab.png' width='75%' />

3.在 GitLab 中创建用于触发 GitLab 的 webhook: 

* 创建[pipeline trigger token](https://docs.gitlab.com/ee/ci/triggers) ；
* 使用项目详细信息构建[pipeline trigger webhook URL](https://docs.gitlab.com/ee/ci/triggers/#use-a-webhook) 。
* 返回 Port，编辑您的操作，并在其 "后端 "步骤的 "端点 URL "字段中粘贴 webhook URL。

4.现在让我们创建包含逻辑的 Pipelines 文件。在 GitLab 项目中新建一个名为 `gitlab-ci.yaml` 的文件，并将以下代码段作为其内容: 

<details>
<summary><b>GitLab pipeline (click to expand)</b></summary>

```yaml showLineNumbers
image: python:3.10.0-alpine

stages:
  - fetch-port-access-token
  - send_reminders
  - post-run-logs
  - update-run-status

fetch-port-access-token: # Example - get the Port API access token and RunId
  stage: fetch-port-access-token
  except:
    - pushes
  before_script:
    - apk update
    - apk add jq curl -q
  script:
    - |
      accessToken=$(curl -X POST \
        -H 'Content-Type: application/json' \
        -d '{"clientId": "'"$PORT_CLIENT_ID"'", "clientSecret": "'"$PORT_CLIENT_SECRET"'"}' \
        -s 'https://api.getport.io/v1/auth/access_token' | jq -r '.accessToken')
      echo "ACCESS_TOKEN=$accessToken" >> data.env
      runId=$(cat $TRIGGER_PAYLOAD | jq -r '.context.runId')
      echo "RUN_ID=$runId" >> data.env
  artifacts:
    reports:
      dotenv: data.env

generate-scorecards-reminders:
  stage: send_reminders
  image: docker:24.0.7
  services:
    - docker:24.0.7-dind
  script:
    - image_name="ghcr.io/port-labs/port-sender:$VERSION"
    - echo "Generate Scorecards Reminders"
    - |
      docker run -i --rm --platform="linux/arm64/v8" \
      -e INPUT_PORT_CLIENT_ID=$PORT_CLIENT_ID \
      -e INPUT_PORT_CLIENT_SECRET=$PORT_CLIENT_SECRET \
      -e INPUT_SLACK_WEBHOOK_URL=$SLACK_WEBHOOK_URL \
      -e INPUT_OPERATION_KIND="scorecard_reminder" \
      -e INPUT_BLUEPRINT="service" \
      -e INPUT_SCORECARD="ProductionReadiness" \
      -e INPUT_TARGET_KIND="slack" \
      $image_name
    - echo "Report status to Port"

post-run-logs:
  stage: post-run-logs
  except:
    - pushes
  image: curlimages/curl:latest
  script:
    - |
      curl -X POST \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"message": "Slack reminder sent successfully 🚀"}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID/logs"
update-run-status:
  stage: update-run-status
  except:
    - pushes
  image: curlimages/curl:latest
  script:
    - |
      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H "Authorization: Bearer $ACCESS_TOKEN" \
        -d '{"status":"SUCCESS", "message": {"run_status": "Created Merge Request for '"$bucket_name"' successfully! Merge Request URL: '"$MR_URL"'"}}' \
        "https://api.getport.io/v1/actions/runs/$RUN_ID"

variables:
  PORT_CLIENT_ID: $PORT_CLIENT_ID
  PORT_CLIENT_SECRET: $PORT_CLIENT_SECRET
  SLACK_WEBHOOK_URL: $SLACK_WEBHOOK_URL
  VERSION: "0.2.3"
```

</details>

</TabItem>

</Tabs>

<br/>

完成！动作已准备就绪，可以开始使用 🚀

<br/>

### 执行操作

创建操作后，该操作将出现在 Port 应用程序的 "自助服务 "选项卡下: 

<img src='/img/guides/selfServiceAfterReminderCreation.png' width='75%' />

1. 点击 "创建 "开始执行操作。
2. 点击 "执行"。弹出一个小窗口，点击`查看详情`: 

<img src='/img/guides/executionDetails.png' width='45%' />

<br/><br/>

3.该页面提供了有关操作运行的详细信息。可以看到，后端返回了 `Success`， repo 已成功创建: 

<img src='/img/guides/runStatusReminder.png' width='90%' />

:::tip  记录操作进度 💡 注意底部的 "日志流"，它可用于报告进度、结果和错误。 点击[here](https://docs.getport.io/create-self-service-experiences/reflect-action-progress/) 了解更多信息。

:::

<br/>

4.现在您可以进入 Slack 频道并查看记分卡提醒: 

<img src='/img/guides/slackReminderExample.png' width='50%' />

恭喜！您现在可以从 Port 💪🏽 轻松发送 Slack 提醒信息了。

#### 结论

创建记分卡是我们在开发生命周期中设定标准的第一步。 然而，为了确保达到这些标准，我们需要将违反规则的行为转化为action项目。 通过自动化 Slack 提醒和创建 Jira 任务，我们可以使用熟悉的工具在整个组织内推动变革，将其原生结合到我们的交付生命周期中。

### 更多例子

* * [Open/Close JIRA issues based on scorecards](/promote-scorecards/manage-using-3rd-party-apps/jira)
* [Send a scorecard report on Slack](/promote-scorecards/manage-using-3rd-party-apps/slack#slack-scorecard-report-example)
