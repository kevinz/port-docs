---
sidebar_position: 4
sidebar_label: 🛠️ Usage 方法

---

import CredentialsGuide from "../build-your-software-catalog/sync-data-to-catalog/api/_template_docs/_find_credentials.mdx";
import ApiRef from "../api-reference/_learn_more_reference.mdx"
import InstallTerraform from "../build-your-software-catalog/sync-data-to-catalog/iac/terraform/_terraform_provider_base.mdx"

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";

# 🛠️ Usage 方法

Port 支持各种集成、应用程序和工具，可让您对软件目录进行建模、摄取数据和调用操作，其中一些核心设置方法如下: 

* Port 的 API；
* Port 的 Terraform Provider；
* Port 的网络用户界面。

请参阅下面的章节，了解如何使用不同的方法来定义您的蓝图和关系: 

:::tip  先决条件

要使用 Port 的 API 和 Terraform Provider，您需要 API 凭据。

<CredentialsGuide />

:::

<Tabs groupId="config" queryString="current-method" defaultValue="api" values={[
{label: "API", value: "api"},
{label: "Terraform", value: "tf"},
{label: "UI", value: "ui"},
]}>

<TabItem value="api">

Port 的[API](../api-reference/api-reference.mdx) 提供了一个方便的 REST 接口，用于在软件目录中执行 CRUD 操作。

Port 的 API 基本 URL 为: `https://api.getport.io/v1`。

应用程序接口支持所有常见的 HTTP 方法: "GET"、"POST"、"PUT"、"PATCH"、"DELETE"。

例如，所有应用程序接口端点都遵循基于资源的访问模式: 

* 基于蓝图的路由以https://api.getport.io/v1/blueprints`；
* 基于实体的路由以https://api.getport.io/v1/blueprints/{blueprint_identifier}/entities`(因为它们是与蓝图绑定的资源)；
* 基于自助操作的路由从以下地址开始: https://api.getport.io/v1/blueprints/{blueprint_identifier}/actions`；
* 基于记分卡的路由从: `https://api.getport.io/v1/blueprints/{blueprint_identifier}/scorecards`开始；
* 等等。

要进一步了解不同 API 对象的 JSON 结构，请参阅各自的类别和结构参考资料，例如: [blueprint structure](../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md#blueprint-structure) 。

<ApiRef />

</TabItem>

<TabItem value="tf">

Port's[Terraform provider](https://registry.terraform.io/providers/port-labs/port-labs/) 允许您使用 Infrastructure-as-Code (IaC) 配置软件目录。

<InstallTerraform />

要使用 Port 的 Terraform Provider 创建蓝图，您需要一个定义[`port_blueprint`](https://registry.terraform.io/providers/port-labs/port-labs/latest/docs/resources/port_blueprint) 资源的`.tf`文件: 

```hcl showLineNumbers
resource "port_blueprint" "myBlueprint" {
  title      = "My blueprint"
  icon       = "My icon"
  identifier = "myIdentifier"
  description = "My description"
}
```

然后运行 "terraform plan "查看将要创建的新蓝图，再运行 "terraform apply "在 Port 的软件目录内创建您定义的蓝图。

要了解有关不同 API 对象的 Terraform 资源定义的更多信息，请参阅各自的类别和结构参考资料，例如[configure blueprints in Port](../build-your-software-catalog/define-your-data-model/setup-blueprint/setup-blueprint.md?definition=tf#configure-blueprints-in-port) 和[Terraform provider](../build-your-software-catalog/sync-data-to-catalog/iac/terraform/terraform.md) 。

</TabItem>

<TabItem value="ui">

要使用用户界面配置软件目录，请访问[DevPortal Builder](https://app.getport.io/dev-portal) 页面，然后按照 "添加蓝图 "用户界面、"测试数据 "模态和其他可视化工具帮助您配置 Port。

</TabItem>

</Tabs>