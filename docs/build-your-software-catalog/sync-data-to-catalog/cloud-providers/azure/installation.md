---
sidebar_position: 1
---

# 安装

使用 Terraform 在 Azure Container App 上部署了 azure 输出程序。它引用了我们的 Terraform[Ocean](https://ocean.getport.io) Integration Factory[module](https://registry.terraform.io/modules/port-labs/integration-factory/ocean/latest) 来部署输出程序。

:::tip 在 Azure 集成示例中可以找到部署 Azure 输出程序的多种方法[README](https://registry.terraform.io/modules/port-labs/integration-factory/ocean/latest/examples/azure_container_app_azure_integration)

:::

##被 Azure 输出程序引用的 Azure 基础设施

Azure 输出程序被引用以下 Azure 基础设施: 

* Azure 容器应用程序；
* Azure 事件网格(被引用到 Port 的实时数据同步): 
    - Azure Event Grid System Topic of type `Microsoft.Resources.Subscriptions`；
    - Azure 事件网格订阅；

:::warning 由于 Azure 的限制，每个订阅只能创建一个**类型为 "Microsoft.Resources.Subscriptions "的事件网格系统主题，因此如果您已经有一个主题，则需要使用 `event_grid_system_topic_name=<your-event-grid-system-topic-name>`将其传递给集成。

如果系统主题已经存在，但未提供给集成部署，集成将无法创建新主题。

:::

## 先决条件

* [Terraform](https://www.terraform.io/downloads.html) >= 0.15.0
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) >= 2.26.0
* [Permissions](#permissions)

## Permissions

In order to successfully deploy the Azure exporter, it's crucial to ensure that the user who deploys the integration in the Azure subscription has the appropriate access permissions. One of the following permission assignments are required:

- Option 1: the user can have the `Owner` Azure role assigned to him for the subscription that the integration will be deployed on. This role provides comprehensive control and access rights;
- Option 2: for a more limited approach, the user should possess the minimum necessary permissions required to carry out the integration deployment. These permissions will grant the user access to specific resources and actions essential for the task without granting full `Owner` privileges. The following steps will guide you through the process of creating a custom role and assigning it to the user along with other required roles:

  - Create a [custom role](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles#steps-to-create-a-custom-role) with the following permissions:

    <details>

    <summary> Custom Resource Definition </summary>

    ```json showLineNumbers
    {
      "id": "<ROLE_DEFINITION_ID>",
      "properties": {
        "roleName": "Azure Exporter Deployment",
        "description": "",
        "assignableScopes": ["/subscriptions/<SUBSCRIPTION_ID>"],
        "permissions": [
          {
            "actions": [
              "Microsoft.ManagedIdentity/userAssignedIdentities/read",
              "Microsoft.ManagedIdentity/userAssignedIdentities/write",
              "Microsoft.ManagedIdentity/userAssignedIdentities/assign/action",
              "Microsoft.ManagedIdentity/userAssignedIdentities/listAssociatedResources/action",
              "Microsoft.Authorization/roleDefinitions/read",
              "Microsoft.Authorization/roleDefinitions/write",
              "Microsoft.Authorization/roleAssignments/write",
              "Microsoft.Authorization/roleAssignments/read",
              "Microsoft.Resources/subscriptions/resourceGroups/write",
              "Microsoft.OperationalInsights/workspaces/tables/write",
              "Microsoft.Resources/deployments/read",
              "Microsoft.Resources/deployments/write",
              "Microsoft.OperationalInsights/workspaces/read",
              "Microsoft.OperationalInsights/workspaces/write",
              "microsoft.app/containerapps/write",
              "microsoft.app/managedenvironments/read",
              "microsoft.app/managedenvironments/write",
              "Microsoft.Resources/subscriptions/resourceGroups/read",
              "Microsoft.OperationalInsights/workspaces/sharedkeys/action",
              "microsoft.app/managedenvironments/join/action",
              "microsoft.app/containerapps/listsecrets/action",
              "microsoft.app/containerapps/delete",
              "microsoft.app/containerapps/stop/action",
              "microsoft.app/containerapps/start/action",
              "microsoft.app/containerapps/authconfigs/write",
              "microsoft.app/containerapps/authconfigs/delete",
              "microsoft.app/containerapps/revisions/restart/action",
              "microsoft.app/containerapps/revisions/activate/action",
              "microsoft.app/containerapps/revisions/deactivate/action",
              "microsoft.app/containerapps/sourcecontrols/write",
              "microsoft.app/containerapps/sourcecontrols/delete",
              "microsoft.app/managedenvironments/delete",
              "Microsoft.Authorization/roleAssignments/delete",
              "Microsoft.Authorization/roleDefinitions/delete",
              "Microsoft.OperationalInsights/workspaces/delete",
              "Microsoft.ManagedIdentity/userAssignedIdentities/delete",
              "Microsoft.Resources/subscriptions/resourceGroups/delete"
            ],
            "notActions": [],
            "dataActions": [],
            "notDataActions": []
          }
        ]
      }
    }
    ```

    </details>

  - [Assign the following roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal) to the user on the subscription that will be used to deploy the integration:
    - The custom `Azure Exporter Deployment` role we defined above.
    - The `API Management Workspace Contributor` role.
    - The `EventGrid Contributor` role.
    - The `ContainerApp Reader` role.
    - The `EventGrid EventSubscription Contributor` role.

## 安装

1. 登录[Port](https://app.getport.io) 并浏览到[builder page](https://app.getport.io/dev-portal)
2. 展开其中一个蓝图并单击蓝图上的摄取按钮，打开摄取模态。
    ![Dev Portal Builder Ingest Button](/img/integrations/azure-exporter/DevPortalBuilderIngestButton.png)
3.单击云 Provider 部分下的 Azure Exporter 选项: 
    ![Dev Portal Builder Azure Exporter Option](/img/integrations/azure-exporter/DevPortalIngestCloudProvider.png)
4.编辑并复制安装命令。
提示
安装命令包含占位符，允许您自定义集成的配置。例如，您可以更新命令并指定 `event_grid_system_topic_name` 参数(如果已经有)。
    - 如果订阅中已有类型为 "Microsoft.Resources.Subscriptions "的 Event Grid 系统主题，请指定 "event_grid_system_topic_name "参数；
    - 如果要监听更多事件，请指定 `event_grid_event_filter_list` 参数；
    - 如果您希望集成拥有更多权限，请指定 `action_permissions_list` 参数。
    :::![Dev Portal Builder Azure Exporter Installation](/img/integrations/azure-exporter/DevPortalIngestAzureInstallation.png)
5.在终端中运行命令部署 Azure 输出程序。

## 映射配置

您可以在集成页面中更新导出器的配置，您可以使用配置来添加或删除将从订阅中被引用的 Azure 资源。

![Dev Portal Ingest Azure Mapping Configuration](/img/integrations/azure-exporter/DevPortalIngestAzureMappingConfiguration.png)

## 更多信息

* 有关实用配置及其相应的蓝图定义，请参阅[examples](./examples.md) 页面。
