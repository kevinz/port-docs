---
sidebar_position: 2
title: 确保生产准备就绪

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 确保生产准备就绪

本指南只需 10 分钟即可完成，内容包括

* 可添加到<PortTooltip id="blueprint">蓝图</PortTooltip>中的一些高级类型的属性，以及使用这些属性可以实现的功能。
* 记分卡在 Port 中的价值和灵活性。

:::tip  先决条件

* 本指南假定您已拥有 Port 账户，并已完成[onboarding process](/quickstart) 。我们将使用入职过程中创建的 "服务 "蓝图。
* 您需要一个可访问的 k8s 集群。如果没有，以下是如何快速设置[minikube cluster](https://minikube.sigs.k8s.io/docs/start/) 。
* [Helm](https://helm.sh/docs/intro/install/) - 需要安装相关集成。

:::

<br/>

### 本指南的目标

在本指南中，我们将为服务的生产就绪状态设定各种标准，并了解如何将其作为 CI 的一部分加以引用。

完成这项工作后，你就会了解它如何使你的组织中的不同角色受益: 

* 平台工程师可以为任何服务定义策略，并据此自动通过/失败发布。
* 开发人员将能轻松查看平台工程师设定的哪些策略未得到满足，以及他们需要修正哪些策略。
* 研发经理可以鸟瞰组织内所有服务的状态。

<br/>

## 扩展你的服务蓝图

在本指南中，我们将在 `service`<PortTooltip id="blueprint">蓝图</PortTooltip>中添加两个新属性，然后用它们来设置生产就绪标准: 

1. 从 Pagerduty 获取的 "待命 "服务。
2. 服务的 "代码所有者"，从 Github 获取。

### 在您的服务中加入随叫随到功能

在本指南中，我们将使用 Pagerduty 来获取服务的待命状态。 请注意，Port 还集成了其他事件响应平台。

#### 创建必要的 Pagerduty 资源

如果您已经有一个可以随意使用的 Pagerduty 账户，请跳过这一步。

1. 创建[Pagerduty account](https://www.pagerduty.com/sign-up/) (免费试用 14 天) 。
2. 创建新服务: 

![pagerdutyServiceCreation](/img/guides/pagerdutyServiceCreation.png)

* 将服务命名为 `DemoPdService`。
* 选择现有的 `Default` 升级策略。
* 在 "减少噪音 "下使用被引用的设置。
* 在 "集成 "下向下滚动并单击 "创建无集成服务"。

#### 将 Pagerduty 整合到 Port 中

现在，让我们将 Pagerduty 数据导入 Port。 Port 的 Pagerduty 集成会自动获取 "服务 "和 "事件"，并为它们创建<PortTooltip id="blueprint">蓝图</PortTooltip>和<PortTooltip id="entity">实体</PortTooltip>。

:::info  注 要进行此安装，您需要 Helm 和一个正在运行的 k8s 集群(请参阅[prerequisites](/guides-and-tutorials/ensure-production-readiness)) 。

:::

1. 在终端运行以下命令，使用 Helm 引用安装 Port 的 Pagerduty 集成。

* 将 `CLIENT_ID`和 `CLIENT_SECRET`替换为您的凭证(获取它们[here](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials)) 。
* 将 `token` 替换为您的 Pagerduty 令牌。获取令牌
    - 将鼠标悬停在 Pagerduty 应用程序右上角您的头像上，然后单击 "我的个人资料"。
    - 点击 "用户设置 "选项卡并滚动到底部。
    - 点击 "创建 API 用户令牌 "并 Provider 一个名称。
    - 复制新的令牌值。

![pagerdutyUserSettings](/img/guides/pagerdutyUserSettings.png)

<details>
<summary>Installation command (click to expand)</summary>

```bash showLineNumbers
helm repo add port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-pagerduty-integration port-labs/port-ocean \
    --set port.clientId="CLIENT_ID" \     # REPLACE VALUE
    --set port.clientSecret="CLIENT_SECRET"  \     # REPLACE VALUE
    --set initializePortResources=true  \
    --set integration.identifier="my-pagerduty-integration"  \
    --set integration.type="pagerduty"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.secrets.token="token"  \     # REPLACE VALUE
    --set integration.config.apiUrl="https://api.pagerduty.com"
```

</details>

太好了！现在集成已安装完毕，我们应该可以在 Port 中看到一些新组件了: 

* 访问[Builder](https://app.getport.io/dev-portal/data-model) ，现在应该可以看到集成创建的两个新<PortTooltip id="blueprint">蓝图</PortTooltip>--"PagerDuty 服务 "和 "PagerDuty 事件"。
* 访问[Software catalog](https://app.getport.io/services) ，单击侧边栏中的 "PagerDuty Services"，现在应该可以看到为我们的 "DemoPdService "创建的新<PortTooltip id="entity">实体</PortTooltip>，并填充了 "On-call "属性。

#### 在服务蓝图中添加待命属性

现在 Port 已与 Pagerduty 资源同步，让我们在我们的服务中反映 Pagerduty 服务的待命状态。 首先，我们需要在我们的服务和相应的 Pagerduty 服务之间创建一个[relation](/build-your-software-catalog/define-your-data-model/relate-blueprints/#what-is-a-relation) 。

1. 返回[Builder](https://app.getport.io/dev-portal/data-model) ，选择 "服务 "<PortTooltip id="blueprint">蓝图</PortTooltip>，点击 "新建关系": 

<img src='/img/guides/serviceCreateRelation.png' width='40%' />

<br/><br/>

2.像这样填写表格，然后点击 "创建": 

<img src='/img/guides/prodReadinessRelationCreation.png' width='50%' />

<br/><br/>

既然<PortTooltip id="blueprint">蓝图</PortTooltip>是相关的，那就让我们在服务中创建一个[mirror property](https://docs.getport.io/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/mirror-property/) 来显示它的待命状态。

1. 再次选择 "服务 "<PortTooltip id="blueprint">蓝图</PortTooltip>，在 "PagerDuty 服务 "关系下点击 "新建镜像属性"。  
像这样填写表格，然后点击 "创建": 

<img src='/img/guides/mirrorPropertyCreation.png' width='40%' />

<br/><br/>

2.既然镜像属性已经设置，我们就需要为每个服务分配相关的 Pagerduty 服务。这可以通过添加一些映射逻辑来完成。访问[data sources page](https://app.getport.io/dev-portal/data-sources) ，点击 Pagerduty 集成: 

<img src='/img/guides/pdDataSources.png' width='60%' />

<br/><br/>

在 "资源 "键下的映射中添加以下 YAML 块，然后点击 "保存并重新同步": 

<details>
<summary>Relation mapping (click to expand)</summary>

```yaml showLineNumbers
- kind: services
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .name | gsub("[^a-zA-Z0-9@_.:/=-]"; "-") | tostring
        title: .name
        blueprint: '"service"'
        properties: {}
        relations:
          pagerduty_service: .id
```

</details>

现在，如果我们的 `service` 标识符等于 Pagerduty 服务的名称，则该 `service` 的 `on-call` 属性将自动填充:  🎉

![entitiesAfterOnCallMapping](/img/guides/entitiesAfterOnCallMapping.png)

**注意**，如果您愿意，可以随时手动执行这项分配: 

1. 进入[Software catalog](https://app.getport.io/services) ，在 "服务 "下的表格中选择任何服务，点击"..."，然后点击 "编辑": 

![editServiceEntity](/img/guides/editServiceEntity.png)

2.现在你会在表单中看到一个名为 "PagerDuty Service "的属性，从下拉菜单中选择我们创建的 "DemoPdService"，然后点击 "更新": 

<img src='/img/guides/editServiceChoosePdService.png' width='40%' />

<br/><br/>

#### 显示每个服务的代码 Owner

Git Provider 允许你在一个版本库中添加 "CODEOWNERS "文件，指定其 Owners。 详情和示例请参阅相关文档: 

<Tabs groupId="git-provider" queryString defaultValue="github" values={[
{label: "GitHub", value: "github"},
{label: "GitLab", value: "gitlab"},
{label: "Bitbucket", value: "bitbucket"}
]}>

<TabItem value="github" label="Github">

[Github codeowners documentation](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)

</TabItem>

<TabItem value="gitlab" label="GitLab">

[GitLab codeowners documentation](https://docs.gitlab.com/ee/user/project/codeowners/)

</TabItem>

<TabItem value="bitbucket" label="BitBucket">

[BitBucket codeowners documentation](https://confluence.atlassian.com/bitbucketserver/code-owners-1296171116.html)

</TabItem>

</Tabs>

<br/>
Let's see how we can easily ingest a CODEOWNERS file into our existing services:

#### 在服务蓝图中添加代码所有者属性

1. 再次进入[Builder](https://app.getport.io/dev-portal/data-model) ，选择 "服务 "<PortTooltip id="blueprint">蓝图</PortTooltip>，然后点击 "新建属性"。
2. 像这样填写表格  
注意 `identifier` 字段的值，下一步我们将需要它。

<img src='/img/guides/addCodeownersForm.png' width='40%' />

3.接下来，我们将更新 Github 输出程序映射，并添加新属性。请访问[data sources page](https://app.getport.io/dev-portal/data-sources) 。
4.在 "导出器 "下，点击带有组织名称的 Github 导出器。
5.在映射 YAML 中，添加一行 `code_owners: file://CODEOWNERS`，如图所示，然后点击 `Resync`: 

![mappingAddCodeOwners](/img/guides/mappingAddCodeOwners.png)

这将告诉 Port 如何填充新属性_ 😎

回到我们的目录，现在可以看到我们的<PortTooltip id="entity">实体</PortTooltip>已经显示了它们的代码 Owners: 

![entityAfterCodeowners](/img/guides/entityAfterCodeowners.png)

<br/>

### 更新服务记分卡

现在，让我们使用我们创建的属性来为我们的服务设定标准。

#### 为现有记分卡添加规则

假设我们要确保每项服务都能满足我们的新要求，并有不同的重要程度。 我们的 "服务 "蓝图已经有了一个名为 "生产准备就绪 "的记分卡，其中有三条规则。 让我们把我们的衡量标准添加进去: 

* 青铜色"--每项服务都必须有 "Readme"(我们已在快速入门指南中对此进行了定义)。
* 银色"--每项服务都必须定义 "on-call"。

现在，让我们来实施它: 

1. 进入[Builder](https://app.getport.io/dev-portal/data-model) ，选择 "服务 "<PortTooltip id="blueprint">蓝图</PortTooltip>，点击 "记分卡"，然后点击我们现有的 "生产准备就绪 "记分卡: 

<img src='/img/guides/editReadinessScorecard.png' width='30%' />

<br/><br/>

2.用以下内容替换，然后点击 "保存": 

<details>
<summary>Scorecard schema (click to expand)</summary>

```json showLineNumbers
{
  "identifier": "ProductionReadiness",
  "title": "Production Readiness",
  "rules": [
    {
      "identifier": "hasReadme",
      "title": "Has a readme",
      "level": "Bronze",
      "query": {
        "combinator": "and",
        "conditions": [
          {
            "operator": "isNotEmpty",
            "property": "readme"
          }
        ]
      }
    },
    {
      "identifier": "hasTeam",
      "title": "Has Team",
      "level": "Silver",
      "query": {
        "combinator": "and",
        "conditions": [
          {
            "operator": "isNotEmpty",
            "property": "$team"
          }
        ]
      }
    },
    {
      "identifier": "hasSlackChannel",
      "title": "Has a Slack channel",
      "level": "Gold",
      "query": {
        "combinator": "and",
        "conditions": [
          {
            "operator": "isNotEmpty",
            "property": "slack"
          }
        ]
      }
    },
    {
      "identifier": "hasCodeowners",
      "title": "Has Codeowners",
      "description": "Checks if a service has a codeowners file",
      "level": "Silver",
      "query": {
        "combinator": "and",
        "conditions": [
          {
            "operator": "isNotEmpty",
            "property": "code_owners"
          }
        ]
      }
    },
    {
      "identifier": "hasOncall",
      "title": "Has On-call",
      "level": "Gold",
      "query": {
        "combinator": "and",
        "conditions": [
          {
            "operator": "isNotEmpty",
            "property": "on_call"
          }
        ]
      }
    }
  ]
}
```

</details>

点击 "记分卡 "选项卡，您将看到该服务的得分，以及通过/未通过检查的详细信息: 

<img src='/img/guides/entityAfterReadinessScorecard.png' width='100%' />

### 可能的日常整合

* 使用 Port 的 API 从您的 CI 检查记分卡合规性，并相应地通过/未通过。
* 定期通过 Slack 通知未通过金牌/银牌/铜牌验证的服务。
* 每周/每月向管理人员发送报告，显示不符合特定标准的服务数量。

#### 结论

生产就绪状态是需要持续监控和处理的问题。 在微服务密集的环境中，代码所有者和待命管理等问题至关重要。 有了 Port，标准的设置、优先级排序和跟踪都变得非常容易。 使用 Port 的 API，您还可以从任何地方创建/获取/修改记分卡，从而与环境中的其他平台和服务实现无缝集成。

更多相关指南和示例: 

* * [Port's OpsGenie integration](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/incident-management/opsgenie/)
* [Integrate scorecards with Slack](https://docs.getport.io/promote-scorecards/manage-using-3rd-party-apps/slack)
