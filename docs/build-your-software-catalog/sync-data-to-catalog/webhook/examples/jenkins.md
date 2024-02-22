---
sidebar_position: 6
description: 将 Jenkins 构建和作业事件纳入目录

---

import JenkinsBuildBlueprint from './resources/jenkins/_example_jenkins_build_blueprint.mdx'
import JenkinsBuildWebhookConfig from './resources/jenkins/_example_jenkins_build_webhook_configuration.mdx'
import JenkinsJobBlueprint from './resources/jenkins/_example_jenkins_job_blueprint.mdx'
import JenkinsJobWebhookConfig from './resources/jenkins/_example_jenkins_job_webhook_configuration.mdx'

# Jenkins

在本示例中，您将在[Jenkins](https://www.jenkins.io/) 和 Port 之间创建一个 webhook 集成，用于接收作业和构建实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Jenkins job blueprint</summary>

<JenkinsJobBlueprint/>

</details>

<details>

<summary>Jenkins build blueprint (including the Jenkins job relation)</summary>
<JenkinsBuildBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Jenkins job and build webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Jenkins Mapper`；
    2.标识符 : `jenkins_mapper`；
    3.Description : `将 Jenkins 构建和作业映射到 Port` 的 webhook 配置；
    4.图标 : `Jenkins`；
2. **集成配置**选项卡 - 填写以下 JQ 映射: 
   <JenkinsBuildWebhookConfig/>
3.点击页面底部的**保存**。

</details>

## 在 Jenkins 中创建 webhook

1. 进入 Jenkins 面板；
2. 在页面左侧的侧边栏中选择 **管理 Jenkins**，然后点击 **管理插件**；
3. 导航至**可用插件**选项卡，在搜索栏中搜索**通用事件**。安装[Generic Event](https://plugins.jenkins.io/generic-event/) 或一个合适的插件，该插件可将 Jenkins 中发生的所有事件通知一些端点；
4. 返回 Jenkins 面板，点击左侧菜单中的**管理 Jenkins**；
5. 点击 "**配置系统**"选项卡，向下滚动到 "**事件调度器**"部分；
6. 在文本框中输入创建 webhook 配置后收到的 `url` 键的值；
7. 单击页面底部的**保存**；

:::tip 为了查看 Jenkins webhooks 中可用的不同有效载荷和事件、[click here](https://plugins.jenkins.io/generic-event/)

:::

完成！作业或构建流程的任何更改(排队、开始、完成、Finalizer 等)都会触发 webhook 事件，并将其发送到 Port 提供的 webhook URL。 Port 将根据映射解析事件，并相应地更新目录实体。