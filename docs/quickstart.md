---
sidebar_position: 2
title: 入门

sidebar_label: ⏱️ 入门

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# ⏱️ 入门

[signing up](https://app.getport.io) 登入 Port 后，系统会提示您按照入职流程进行操作，包括将 Git 仓库导入开发者门户。

完成入驻流程后，Port 将为您创建一些组件(使用您的真实数据😎)，以便向您展示门户网站的潜力。

我们强烈建议您完成入职流程，以便对 Port 有一个基本的了解，并知道一个好的开发人员门户网站能如何帮助您和您的开发人员。

:::info  跳过入职 如果您选择**跳过**入职流程，您仍然可以通过将 Port 连接到所需的 Git Provider 来为您创建这些组件: 

* 进入门户网站[data-sources page](https://app.getport.io/dev-portal/data-sources) 。
* 点击右上角的 "+ 数据源"，选择所需的 Git Provider。

:::

## 门户网站初始体验

<br/>
<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/ggXL2ZsPVQM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>
<br/>
In this walkthrough, we will go over the components in your portal and learn about the different pillars of Port.

#### Homepage

让我们从[homepage](https://app.getport.io/organization/home) 开始。主页是开发人员日常工作的中心。它是一个完全可定制的仪表板，您可以在其中创建小部件，以可视化方式跟踪与您和您的开发人员相关的数据。

最初，您的主页包含两个小部件: 

1. 介绍门户及其内容的标记符文件。这样的小工具可被用来向开发人员传递信息，或描述门户中的操作、蓝图和实体。
2. 带有演示视频的 iframe 小工具。

**了解更多信息: **

* * [Dashboard widgets](https://docs.getport.io/customize-pages-dashboards-and-plugins/dashboards/#widget-types)

---

### 蓝图

蓝图是 Port 的基本构件，用于为您想添加到软件目录中的任何数据源建模。 请访问[builder](https://app.getport.io/dev-portal/data-model) - 这里是您创建、编辑和关联蓝图的地方。

如您所见，在将 Git Provider 连接到 Port 之后，一个新的 "服务 "蓝图就会自动创建。 该蓝图代表组织中的一项服务，在 Git 仓库中实现。它带有一些预定义的[properties](https://docs.getport.io/build-your-software-catalog/define-your-data-model/setup-blueprint/properties/) 。

**了解更多信息: **

* * [Setup blueprints](https://docs.getport.io/build-your-software-catalog/define-your-data-model/setup-blueprint/)
* [Relate Blueprints](https://docs.getport.io/build-your-software-catalog/define-your-data-model/relate-blueprints/)

---

### 实体

实体是一个蓝图的实例，代表该蓝图属性所定义的数据。 实体显示在门户网站的[software catalog](https://app.getport.io/services) 页面。

将 Git Provider 连接到 Port 后，您将在目录的 "服务 "页面看到所有服务(Git 仓库)。

**了解更多信息: **

* * [Creating entities](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/#creating-entities)

---

### 自助action

Port 允许您创建灵活的、受权限控制的操作，供开发人员使用。 操作是通过门户网站的[self-service](https://app.getport.io/self-serve) 页面创建和执行的。

您应该可以在 Port 中看到多个操作。 Port 中的操作包括两个部分: 

* 前台--您在此定义操作类型及其输入。
* 后端 - 触发操作逻辑的地方。Port 支持多种后端。

:::info  完成操作 门户中的现有操作只定义了前端。 要完成操作的设置，请单击该操作所附的链接，按照专门的指南进行操作。

:::

**了解更多信息: **

* * [Self-service experiences](https://docs.getport.io/create-self-service-experiences/)

---

#### 记分卡

Port 的另一个主要支柱是记分卡。 记分卡用于定义和跟踪资源的衡量标准，并可用于在组织中执行标准。 记分卡是根据蓝图定义的，可在[builder](https://app.getport.io/dev-portal/data-model) 中从蓝图本身创建/修改。

看看你的 "服务 "蓝图，它有一个 "生产就绪 "记分卡，定义并跟踪三条规则。

**了解更多信息: **

* * [Promote scorecards](https://docs.getport.io/promote-scorecards/)

---

### 仪表板

除了[homepage](#homepage) 之外，您还可以在[software catalog](https://app.getport.io/services) 中创建仪表盘。这些仪表盘用于跟踪和可视化有关[entities](#entities) 的数据。

您的软件目录应该已经有两个仪表板页面: 

#### 服务概览

该仪表盘包含有关于您的服务的真实数据的小部件。 它是完全可定制的，可作为您如何使用仪表盘来跟踪和可视化您的实体数据的示例。

#### 记分卡仪表板

这个仪表盘也是服务数据的可视化，但这次的重点是服务的记分卡。 它可以作为一个例子，说明如何使用仪表盘来跟踪指标，并轻松查看标准在整个资源中的执行情况。

**了解更多信息: **

* * [Dashboard page](https://docs.getport.io/customize-pages-dashboards-and-plugins/page/dashboard-page)
* [Dashboard widgets](https://docs.getport.io/customize-pages-dashboards-and-plugins/dashboards/)

---

#What's next?

现在，您已经对 Port 的主要支柱有了基本了解，可以开始定制您的门户，以满足您的需求。 以下是一些开始的好方法: 

❇️ 完成您的[self-service actions](#self-service-actions) 设置并试用它们。 ❇️ 完成我们的[guides](https://docs.getport.io/guides-and-tutorials) ，学习如何使用 Port 解决实际用例。 ❇️ 加入我们的[community Slack](https://www.getport.io/community) ，提问并分享您使用 Port 的经验。 ❇️ 正在使用 **monorepos**? 请参阅我们的[quick guide](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/git/working-with-monorepos) ，了解如何调整您的配置。