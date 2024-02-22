---

sidebar_position: 6
title: 开始使用 AWS

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import FindCredentials from "/docs/build-your-software-catalog/sync-data-to-catalog/api/_template_docs/_find_credentials.mdx";

# 开始使用 AWS

本指南只需 4 分钟即可完成，旨在将 AWS 环境快速集成到 Port 中。

:::tip  先决条件

* 虽然本指南并非强制要求，但我们建议您在继续之前完成[onboarding process](/quickstart) 。
* 您需要一个激活的[Port account](https://app.getport.io/) 。
* 安装 AWS 输出程序需要[Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli) 。
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) 被引用用于身份验证。确保您的 AWS `Access key id` 和 `Secret access key` 已设置。如果没有，请在终端运行 `aws configure` 进行配置。

:::

<br/>

### 本指南的目标

在本指南中，我们将安装 Port 的 AWS 导出器，将 AWS 资源导入 Port，并根据其数据创建可视化。

完成这项工作后，你就会了解它如何使你的组织中的不同角色受益: 

* 开发人员和开发人员可以跟踪 AWS 资源及其健康状况。
* 平台工程师将能够创建和监控仪表板和可视化，从 AWS 环境中显示有价值的数据。

<br/>

#### 安装 Port 的 AWS 输出程序

1. 进入[Port application](https://app.getport.io/) ，点击右上角的"..."，然后点击 "凭据"。复制您的 "客户 ID "和 "客户 secret"。
2. 在以下命令中替换`YOUR-PORT-CLIENT-ID`和`YOUR-PORT-CLIENT-SECRET`，然后复制并在终端运行: 

_Note that this command will clone a repository in the current working directory of your terminal_

<details>
<summary><b>AWS exporter installation (click to expand)</b></summary>

```bash showLineNumbers
# Export your Port credentials
export PORT_CLIENT_ID=YOUR-PORT-CLIENT-ID
export PORT_CLIENT_SECRET=YOUR-PORT-CLIENT-SECRET

# Clone the terraform template
git clone https://github.com/port-labs/template-assets.git

cd template-assets/aws

# Initialize the Terraform requirements
terraform init

# Deploy the aws exporter and provide the resources you want to export
terraform apply -var 'resources=["lambda", "s3_bucket"]'
```

</details>

:::info  支持的 AWS 资源 在上面的代码段中，我们只摄取了 `lambda` 和 `s3_bucket` 资源。 您可以通过将其他资源添加到 `resources` 列表来摄取它们。 其他支持的资源包括 `ecs_service`、`sns`、`sqs`、`rds_db_instance`、`dynamodb_table` 和 `ec2_instance`。

:::

#### 刚刚发生了什么？

出口程序的安装做了三件事: 

1. 它为你在安装命令中指定的资源创建了蓝图，你可以在[builder](https://app.getport.io/dev-portal/data-model).
2. 它为从 AWS 环境获取的资源创建了实体，你可以在[software catalog](https://app.getport.io/catalog) 中看到它们。
3. 它会在 AWS 环境的 s3 存储桶中创建一个 JSON 配置文件，其中包含一个映射定义，用于用环境中的数据填充实体的属性。例如，您的[lambdas catalog page](https://app.getport.io/lambdas) 包含以下已填充属性的实体: 

<img src='/img/guides/awsLambdaEntityPage.png' width='100%' />

<br/><br/>

:::info  提示 - 更新配置

要在安装后更改配置，请使用[AWS exporter page](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/#changing-the-configuration) 中描述的命令。

💁🏽 _本指南无需更改任何配置，仅供参考_

:::

<br/>

###可视化 AWS 资源中的数据

Port 的主页是一个仪表盘，允许您对实体中的数据创建各种可视化效果。 让我们在主页中创建一个简单的饼图，显示我们的 `Lambdas` 按 `Runtime` 的分布情况: 

1. 请访问[homepage](https://app.getport.io/homepage) 。
2. 点击右上角的 "+ 添加 "按钮，选择 "饼图"。
3. 像这样填写表格，然后点击`保存`: 

<img src='/img/guides/awsLambdaPieChartForm.png' width='50%' />

<br/><br/>

现在你应该能在主页上看到饼图了: 

<img src='/img/guides/awsLambdaPieChart.png' width='50%' />

#### 结论

AWS 环境复杂且数据丰富。 Port 的 AWS 导出器可让您轻松地将 AWS 环境中的资源导入 Port，但这仅仅是个起点。 数据导入后，您可以使用 Port 可视化相关数据、使用自助操作自动化日常任务、跟踪资源的指标等。

如需深入了解 Port 与 AWS 的集成(包括示例) ，请参阅[AWS integration](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/) 部分。

更多指南和教程即将推出，在此期间如有任何问题，请随时通过[community slack](https://www.getport.io/community) 或[Github project](https://github.com/port-labs?view_as=public) 联系我们。