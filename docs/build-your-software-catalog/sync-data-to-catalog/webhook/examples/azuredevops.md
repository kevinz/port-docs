---
sidebar_position: 15
description: 将 Azure DevOps 资源收录到您的目录中

---

import ProjectBlueprint from './resources/azuredevops/_example_project_blueprint.mdx'
import RepositoryBlueprint from './resources/azuredevops/_example_repository_blueprint.mdx'
import PipelineBlueprint from './resources/azuredevops/_example_pipeline_blueprint.mdx'
import WorkItemBlueprint from './resources/azuredevops/_example_work_item_blueprint.mdx'
import AzureDevopsWebhookConfig from './resources/azuredevops/_example_webhook_configuration.mdx'

# Azure DevOps

在本示例中，我们将使用脚本从 Azure DevOps API 获取数据并将其导入 Port。我们还将在[Azure DevOps](https://azure.microsoft.com/en-us/products/devops) 和 Port 之间创建 webhook 集成，该集成将导入 "项目"、"资源库"、"工作项 "和 "Pipelines "实体。

## Port 配置

创建以下蓝图定义: 

<details>
<summary>Project blueprint</summary>

<ProjectBlueprint/>

</details>

<details>
<summary>Repository blueprint</summary>

<RepositoryBlueprint/>

</details>

<details>
<summary>Pipeline blueprint</summary>

<PipelineBlueprint/>

</details>

<details>
<summary>Work item blueprint</summary>

<WorkItemBlueprint/>

</details>

## 运行 python 脚本

要从 Azure DevOps 账户向 Port 输入数据，请运行以下命令: 

```bash
export PORT_CLIENT_ID=<ENTER CLIENT ID>
export PORT_CLIENT_SECRET=<ENTER CLIENT SECRET>
export AZURE_DEVOPS_ORG_ID=<ENTER AZURE DEVOPS ORGANIZATION ID>
export AZURE_DEVOPS_APP_PASSWORD=<ENTER AZURE DEVOPS APP PASSWORD>

git clone https://github.com/port-labs/azure-devops-resources.git

cd azure-devops-resources

pip install -r ./requirements.txt

python app.py
```

:::tip 查找有关 python 脚本的更多信息[here](https://github.com/port-labs/azure-devops-resources)

请按照官方文档了解如何[create an azure devops app password](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&amp;tabs=Windows) 。

:::

## Port Webhook 配置

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints)

<details>

<summary>Webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `Azure DevOps Mapper`。
    2.标识符: `azure_devops_mapper`。
    3.描述 : `将 Azure DevOps 资源映射到 Port` 的 webhook 配置。
    4.图标 : `AzureDevops`.
2. **集成配置**选项卡--填写以下 JQ 映射: 
   <AzureDevopsWebhookConfig/>
3.单击页面底部的**保存**。

</details>

## 在 Azure DevOps 中创建 webhook

1. 从你的 Azure DevOps 账户，打开要添加 webhook 的项目。
2. 单击左侧边栏上的**项目设置**。
3. 在常规部分，选择左侧边栏上的**服务钩子**。
4. 单击加 **+** 按钮，为项目创建 webhook。
5. 将弹出一个页面。从列表中选择**网络钩子**，然后点击**下一步**。
6. 在**触发器**下，选择要接收 webhook 通知的事件类型。示例 webhook 配置支持以下事件: 
    1.代码推送
    2.运行状态改变
    3.运行任务状态已更改
    4.运行阶段状态已更改
    5.创建工作项
    6.更新工作项
7.保留**过滤器**部分不变，然后单击**下一步**。
8.在最后一页(**操作设置**)，在 `URL` 文本框中输入您在 Port 中创建 webhook 配置后收到的 webhook `URL` 的值。
9.测试您的 Webhook 订阅，然后单击**完成**。

请关注[this documentation](https://learn.microsoft.com/en-us/azure/devops/service-hooks/events?toc=%2Fazure%2Fdevops%2Fmarketplace-extensibility%2Ftoc.json&amp;view=azure-devops) ，了解有关 Azure DevOps 中 webhook 事件的更多信息。

完成！Azure DevOps 中的版本库、工作项或管道发生的任何更改都会触发指向 Port 提供的 webhook URL 的 webhook 事件。 Port 将根据映射解析事件并相应地更新目录实体。