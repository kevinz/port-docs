---
sidebar_position: 1
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";
import Image from "@theme/IdealImage";

# Azure

我们与 Azure 的集成可根据您的配置将 Azure 资源导出到 Port。 初次导入数据后，集成还会监听来自 Azure 的实时事件，实时更新 Port 内部的数据。

我们与 Azure 的集成支持实时事件处理，因此可以在 Port 内部准确、实时地呈现 Azure 基础架构。

:::tip Port 的 Azure 输出程序是开源的，请查看源代码[**here**](https://github.com/port-labs/ocean/tree/main/integrations/azure) 。

:::

## 💡 Azure 集成常见用例

例如，我们与 Azure 的集成可让您直接从 Azure 订阅中获取数据，从而轻松填充软件目录: 

* 映射 Azure 订阅中的所有资源，包括**AKS**、**存储帐户**、**容器应用程序**、**负载平衡器**和其他 Azure 对象；
* 实时关注 Azure 对象的更改(创建/更新/删除)，并自动将更改应用到 Port 中的实体；
* 使用关系在 Port 中创建完整、易于理解的 Azure 基础架构视图。

## 安装

要安装 Port 的 Azure 输出程序，请遵循[installation](./installation.md) 指南。

## 工作原理

Port 的 Azure 导出器可以检索 Azure API 支持的所有资源，并将其作为现有蓝图的实体导出到 Port。

通过 Azure 导出器，可以将 Azure API 中的数据提取、转换、加载(ETL)到所需的软件目录数据模型中。

导出器是通过 Azure[Container App](https://learn.microsoft.com/en-us/azure/container-apps/overview) 部署的，而 Azure 已部署到您的 Azure 订阅中。

Azure 输出程序使用 YAML 配置来描述将数据加载到开发人员门户网站的 ETL 流程。 这种方法反映了一种中间立场，即过于主观的 Azure 可视化可能不适合每个人，而过于宽泛的方法可能会给开发人员门户网站带来不必要的复杂性。

下面是配置中的一个示例片段，演示了从 Azure 获取 "容器应用程序 "数据并将其导入软件目录的 ETL 流程: 

```yaml showLineNumbers
resources:
  # Extract
  # highlight-start
  - kind: Microsoft.App/containerApps
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      apiVersion: "2023-05-01" # Azure API version to use to fetch the resource
    # highlight-end
    port:
      entity:
        mappings:
          # Transform & Load
          # highlight-start
          identifier: '.id | split("/") | .[3] |= ascii_downcase |.[4] |= ascii_downcase | join("/")' # lowercase only the resourceGroups namespace and name to align how azure API returns the resource group reference
          title: .name
          blueprint: '"azureContainerApp"'
          properties:
            location: .location
            provisioningState: .properties.provisioningState
            outboundIpAddresses: .properties.outboundIpAddresses
            externalIngress: .properties.configuration.ingress.external
            hostName: .properties.configuration.ingress.fqdn
            minReplicas: .properties.template.scale.minReplicas
            maxReplicas: .properties.template.scale.maxReplicas
          relations:
            resourceGroup: '.id | split("/") | .[3] |= ascii_downcase |.[4] |= ascii_downcase | .[:5] |join("/")'
          # highlight-end
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 Azure API 事件中的现有字段和值进行选择、修改、连接、转换和其他操作。

#### 整合配置结构

集成配置是一个 YAML 文件，描述了将数据加载到开发者门户的 ETL 流程。

* 资源 "部分描述了要导入 Port 的 Azure 资源。


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: Microsoft.App/containerApps
      selector:
      ...
  ```

- The `kind` field describes the Azure resource type to be ingested into Port.
  The `kind` field should be set to the Azure resource type as it appears in the [resource guide](https://learn.microsoft.com/en-us/azure/templates/). e.g. The resource type for the `Container App` could be found [here](https://learn.microsoft.com/en-us/azure/templates/microsoft.app/containerapps?pivots=deployment-language-arm-template) as well with the resource object structure.

  ```yaml showLineNumbers
  resources:
    # highlight-start
    - kind: Microsoft.App/containerApps
      # highlight-end
      selector:
      ...
  ```

- The `selector` field describes the Azure resource selection criteria.


  ```yaml showLineNumbers
  resources:
    - kind: Microsoft.App/containerApps
      # highlight-start
      selector:
        query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
        apiVersion: "2023-05-01" # Azure API version to use to fetch the resource
      # highlight-end
  ```


* 查询 "字段是[JQ boolean query](https://stedolan.github.io/jq/manual/#Basicfilters) ，如果评估结果为 "false"，则将跳过该资源。被引用示例 - 跳过同步不在特定区域的资源。


    ```yaml showLineNumbers
    query: .location == "eastus2"
    ```

  - The `apiVersion` field is the Azure API version to use to fetch the resource. This field is required for all resources. You can find the API version for each resource in the [Azure Resources reference](https://learn.microsoft.com/en-us/azure/templates/). For example, the supported API versions for the `containerApps` resource was found in the [Container App Changelog](https://learn.microsoft.com/en-us/azure/templates/microsoft.app/change-log/containerapps).

    ```yaml showLineNumbers
    apiVersion: "2023-05-01"
    ```


* Port` 字段描述了要从 Azure 资源中创建的Port实体。


  ```yaml showLineNumbers
  resources:
    - kind: Microsoft.App/containerApps
      selector:
        query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
        apiVersion: "2023-05-01" # Azure API version to use to fetch the resource
      # highlight-start
      port:
        entity:
          mappings:
            identifier: .id
            title: .name
            blueprint: '"azureContainerApp"'
            properties:
              location: .location
              provisioningState: .properties.provisioningState
              outboundIpAddresses: .properties.outboundIpAddresses
              externalIngress: .properties.configuration.ingress.external
              hostName: .properties.configuration.ingress.fqdn
              minReplicas: .properties.template.scale.minReplicas
              maxReplicas: .properties.template.scale.maxReplicas
            relations:
              resourceGroup: '.id | split("/") | .[3] |= ascii_downcase |.[4] |= ascii_downcase | .[:5] |join("/")'
      # highlight-end
  ```

  - The `entity` field describes the Port entity to be created from the Azure resource.

    ```yaml showLineNumbers
    resources:
      - kind: Microsoft.App/containerApps
        selector:
          query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
          apiVersion: "2023-05-01" # Azure API version to use to fetch the resource
        port:
          # highlight-start
          entity:
            mappings:
              identifier: .id
              title: .name
              blueprint: '"azureContainerApp"'
              properties:
                location: .location
                provisioningState: .properties.provisioningState
                outboundIpAddresses: .properties.outboundIpAddresses
                externalIngress: .properties.configuration.ingress.external
                hostName: .properties.configuration.ingress.fqdn
                minReplicas: .properties.template.scale.minReplicas
                maxReplicas: .properties.template.scale.maxReplicas
              relations:
                resourceGroup: '.id | split("/") | .[3] |= ascii_downcase |.[4] |= ascii_downcase | .[:5] |join("/")'
          # highlight-end
    ```


#### 授权

要将资源导出到 Port，导出器需要有访问 Azure 订阅的权限。具体方法是为导出器分配一个[user-assigned identity](https://learn.microsoft.com/en-us/azure/container-apps/managed-identity?tabs=portal,dotnet#add-a-user-assigned-identity) ，并授予该身份访问 Azure 订阅所需的权限。

作为安装过程的一部分，您需要定义导出程序对 Azure 订阅的权限。这将通过指定允许导出程序在 Azure 订阅上执行的[actions](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-definitions#actions) 来定义。

下面是一个可以分配给输出程序的操作权限示例: 

```yaml showLineNumbers
action_permissions_list = [
"Microsoft.Resources/subscriptions/resourceGroups/read",
"Microsoft.Resources/subscriptions/resources/read",
"Microsoft.app/containerapps/read",
"Microsoft.Storage/storageAccounts/*/read",
"Microsoft.ContainerService/managedClusters/read",
"Microsoft.Network/loadBalancers/read",
```

:::tip 要查找更多可分配给导出器的操作，可以使用[Azure Resource provider operation reference](https://learn.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations) 并查找要导出到 Port 的资源。

:::

## 开始

继续阅读[installation](./installation.md) 指南，了解如何安装 Azure 输出程序。