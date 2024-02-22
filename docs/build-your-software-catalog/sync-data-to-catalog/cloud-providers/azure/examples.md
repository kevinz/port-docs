---

sidebar_position: 2

---

import ResourceGroupBlueprint from './examples/resource_group/_blueprint.mdx'
import ResourceGroupAppConfig from './examples/resource_group/_port_app_config.mdx'

import StorageAccountBlueprint from './examples/storage/_storage_account_blueprint.mdx'
import StorageContainerBlueprint from './examples/storage/_storage_container_blueprint.mdx'
import StorageAppConfig from './examples/storage/_port_app_config.mdx'

import ResourcesAppConfig from './examples/compute_resources/_port_app_config.mdx'
import AKSBlueprint from './examples/compute_resources/_aks_blueprint.mdx'
import ContainerAppBlueprint from './examples/compute_resources/_container_app_blueprint.mdx'
import LoadBalancerBlueprint from './examples/compute_resources/_load_balancer_blueprint.mdx'
import VirtualMachineBlueprint from './examples/compute_resources/_virtual_machine_blueprint.mdx'
import WebAppBlueprint from './examples/compute_resources/_web_app_blueprint.mdx'

import DatabaseAppConfig from './examples/database_resources/_port_app_config.mdx'
import PostgresFlexibleServerBlueprint from './examples/database_resources/_postgres_flexible_server_blueprint.mdx'

# 示例

:::info 本页面中的资源只是 Azure 输出程序支持的少数资源。 如果没有找到要映射到 Port 的 Azure 资源，请访问[Mapping Extra Resources](mapping_extra_resources.md) 页面，了解 Azure 集成支持哪些 Azure 资源以及如何将它们映射到 Port。

:::

## 映射资源组

在以下示例中，您将把 Azure 资源组引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<ResourceGroupBlueprint/>

<ResourceGroupAppConfig/>

以下是我们用来创建这些蓝图和应用程序配置的 API 引用: 

* * [Resource Group](https://docs.microsoft.com/en-us/rest/api/resources/resourcegroups/list)

## 映射存储资源

在下面的示例中，您将把 Azure 存储账户和容器引用到 Port，您可以使用下面的 Port 蓝图定义和集成配置: 

:::note 存储账户与资源组有关系，因此需要创建[Resource Group blueprint](#mapping-resource-groups) 。

:::

<StorageAccountBlueprint/>

<StorageContainerBlueprint/>

<StorageAppConfig/>

以下是我们用来创建这些蓝图和应用程序配置的 API 引用: 

* * [Storage Account](https://docs.microsoft.com/en-us/rest/api/storagerp/storageaccounts/list)
* [Storage Container](https://learn.microsoft.com/en-us/rest/api/storagerp/blob-containers/list?tabs=HTTP)

## 映射计算资源

在以下示例中，您将把 Azure 资源引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

:::note 下面的资源与资源组有关系，因此需要创建[Resource Group blueprint](#mapping-resource-groups) 。

:::

<AKSBlueprint/>

<ContainerAppBlueprint/>

<LoadBalancerBlueprint/>

<VirtualMachineBlueprint/>

<WebAppBlueprint/>

<ResourcesAppConfig/>

以下是我们用来创建这些蓝图和应用程序配置的 API 引用: 

* * [AKS](https://learn.microsoft.com/en-us/rest/api/aks/managed-clusters/list?tabs=HTTP)
* [Container App](https://learn.microsoft.com/en-us/rest/api/containerapps/stable/container-apps/list-by-subscription?tabs=HTTP)
* [Load Balancer](https://learn.microsoft.com/en-us/rest/api/load-balancer/load-balancers/list-all?tabs=HTTP)
* [Virtual Machine](https://learn.microsoft.com/en-us/rest/api/compute/virtual-machines/list-all?tabs=HTTP)
* [Web App](https://learn.microsoft.com/en-us/rest/api/appservice/web-apps/list)

## 映射数据库资源

在下面的示例中，您将把 Azure 数据库资源引用到 Port，您可以使用下面的 Port 蓝图定义和集成配置: 

:::note 下面的数据库资源与资源组有关系，因此需要创建[Resource Group blueprint](#mapping-resource-groups) 。

:::

<PostgresFlexibleServerBlueprint/>

<DatabaseAppConfig/>

以下是我们用来创建这些蓝图和应用程序配置的 API 引用: 

* * [Postgres Flexible Server](https://docs.microsoft.com/en-us/rest/api/azure-postgresql/flexibleservers)

:::info 本页面中的资源只是 Azure 输出程序支持的少数资源。 如果没有找到要映射到 Port 的 Azure 资源，请访问[Mapping Extra Resources](mapping_extra_resources.md) 页面，了解 Azure 集成支持哪些 Azure 资源以及如何将它们映射到 Port。

:::