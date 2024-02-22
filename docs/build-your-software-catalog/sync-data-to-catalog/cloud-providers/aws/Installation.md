---
sidebar_position: 2
---

import FindCredentials from "/docs/build-your-software-catalog/sync-data-to-catalog/api/_template_docs/_find_credentials_collapsed.mdx";
import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# 安装

## 先决条件

* 您将需要[Port credentials](/build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) 来安装 AWS 输出程序: 
   <FindCredentials />
* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) 是身份验证所必需的。确保已设置 AWS `Access key id` 和 `Secret access key`。如果没有，请在终端运行 `aws configure` 进行配置。

对于[step-by-step installation](#step-by-step-installation) (未被用于 Terraform) ，也要安装: 

* * [AWS SAM CLI](https://docs.aws.amazon.com/serverless-app:lication-model/latest/developerguide/install-sam-cli.html)
* [JQ](https://stedolan.github.io/jq/download/)

### Terraform 安装(推荐)

在终端运行以下脚本

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
terraform apply -var 'resources=["ecs_service", "lambda", "sns", "sqs", "s3_bucket", "rds_db_instance", "dynamodb_table", "ec2_instance"]'
```

:::info 上述脚本执行以下操作: 

1. 在 Port 环境中创建资源蓝图。
2. 在 AWS 环境中使用以下资源部署 AWS 输出程序 -[S3 bucket](https://aws.amazon.com/s3/),[mapping configuration file](./aws.md#exporter-configjson-file),[AWS secret](./aws.md#port-credentials-secret),[AWS IAM policy](./aws.md#iam-policy) ；
3. 设置[Event Bridge Rules](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) ，触发出口程序更新资源；
4. 首次调用 AWS 输出程序 Lambda 函数以获取当前资源状态。

:::

:::tip 您可以从上述脚本的资源数组中删除不想导出的资源。

:::

#### AWS 出口商 Terraform 模块

设置基本配置后，上述模板将部署 AWS 出口商 Terraform 模块。

欲了解更多信息，请访问[AWS exporter module docs](/build-your-software-catalog/sync-data-to-catalog/iac/terraform/modules/aws-exporter-module.md)

## 分步安装

此处概述的步骤可用于使用 CloudFormation 手动安装 AWS 输出程序。

要部署应用程序，您需要填写以下参数: 

* **云形成相关参数: **
    - 应用程序名称"- 通过 "AWS CloudFormation "创建的应用程序的堆栈名称。
* **桶相关参数: **
    - `CreateBucket` - 如果希望应用程序创建和管理您的存储桶，则为 `true`；如果希望自行创建存储桶，则为 `false`。
    - `BucketName` - 你的邮筒的名称，或一个新邮筒的全局唯一名称。
    - `ConfigJsonFileKey` - 文件桶中[`config.json`](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/#exporter-configjson-file) 的文件密钥(路径)。
* ** IAM 策略相关参数: **
    - `CustomIAMPolicyARN` -[IAM policy](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/#iam-policy) 的 ARN。
* **与 secret 相关的参数: **
    - `CustomPortCredentialsSecretARN` - Port凭据secret的 ARN；
        **或
    - `SecretName` - 要创建的新Port凭据secret的名称。
* **与 Lambda 相关的参数: **
    - `FunctionName` - 输出程序 lambda 的函数名。
    - `ScheduleExpression` -[schedule expression](https://docs.aws.amazon.com/lambda/latest/dg/services-cloudwatchevents-expressions.html) ，用于定义出口程序的事件日程。
    - `ScheduleState` - 计划表的初始状态 - `ENABLED` 或 `DISABLED`。建议仅在成功运行一次后启用。

1. 准备一个[`config.json`](#exporter-configjson-file) 文件，该文件将定义将哪些 AWS 资源摄取到 Port；
2. 创建[`IAM policy`](#iam-policy) ，为`config.json`中的 AWS 资源提供`list`和`read`权限；

:::tip  创建策略 IAM 策略参考[here](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html) 。

:::

3. Deploy our [`serverless application`](#exporter-aws-serverless-application).

   <Tabs groupId="deploy-options" defaultValue="AWSConsole" values={[{label: "AWS Console", value: "AWSConsole"}, {label: "AWS CLI", value: "AWSCLI"}]}>

   <TabItem value="AWSConsole">

   You can deploy the application from the AWS console through this [link](https://eu-west-1.console.aws.amazon.com/lambda/home?region=eu-west-1#/create/app?applicationId=arn:aws:serverlessrepo:eu-west-1:185657066287:applications/port-aws-exporter).

   </TabItem>
      
   <TabItem value="AWSCLI">

   Follow these steps:

   1. Create a `parameters.json` file to override certain parameters values. For example (replace the placeholders):

      ```json showLineNumbers
      [
        {
          "Name": "CustomIAMPolicyARN",
          "Value": "<YOUR_IAM_POLICY_ARN>"
        },
        {
          "Name": "CustomPortCredentialsSecretARN",
          "Value": "<YOUR_PORT_CREDENTIALS_SECRET_ARN>"
        },
        {
          "Name": "CreateBucket",
          "Value": "false"
        },
        {
          "Name": "BucketName",
          "Value": "<YOUR_BUCKET_NAME>"
        },
        {
          "Name": "ScheduleExpression",
          "Value": "rate(1 hour)"
        },
        {
          "Name": "ScheduleState",
          "Value": "DISABLED"
        }
      ]
      ```

   2. Use the following command to create a change set:

      ```bash showLineNumbers
      aws serverlessrepo create-cloud-formation-change-set --application-id arn:aws:serverlessrepo:eu-west-1:185657066287:applications/port-aws-exporter --stack-name port-aws-exporter --capabilities CAPABILITY_IAM CAPABILITY_RESOURCE_POLICY --parameter-overrides file://parameters.json

      # Result
      {
        "ApplicationId": "arn:aws:serverlessrepo:eu-west-1:185657066287:applications/port-aws-exporter",
        "ChangeSetId": "<ChangeSetId>",
        ...
      }
      ```

   3. With the `<ChangeSetId>` from the previous command output, deploy the change set:

      ```bash showLineNumbers
      aws cloudformation execute-change-set --change-set-name "<ChangeSetId>"
      ```

   </TabItem>

   </Tabs>

:::info 

部署完成后，请使用以下 AWS SAM CLI 命令来获取出口程序资源的有用列表: 

```bash showLineNumbers
sam list stack-outputs --stack-name serverlessrepo-port-aws-exporter
```

名单包括

* Lambda 函数 ARN` - 输出端 Lambda 的 ARN；
* Port凭据secret ARN` - Port凭据secret的 ARN；
* `ConfigBucketName` - 输出者的存储桶名称。

:::

:::tip  部署无服务器应用程序

有关如何部署无服务器应用程序的详细信息，请单击[here](https://docs.aws.amazon.com/serverlessrepo/latest/devguide/serverlessrepo-how-to-consume.html) 。

:::

4. 使用您的证书更新[`Port credentials secret`](#port-credentials-secret) ；

:::tip  修改secret 要了解如何修改secret的值，请查找[here](https://docs.aws.amazon.com/secretsmanager/latest/userguide/manage_update-secret.html) 。

:::

5. 将 `config.json` 上传到出口程序的 S3 存储桶。

:::tip  将文件上传到 S3 存储桶 要了解如何将文件上传到 S3，请查看[here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html) 。

:::

### 测试应用程序

为了测试已部署的应用程序，应使用空测试事件 (`{}`) 运行出口程序的 lambda，并查看执行状态和日志。

:::tip  使用测试事件调用函数 如何使用测试事件调用 lambda 函数的参考文献可在[here](https://docs.aws.amazon.com/lambda/latest/dg/testing-functions.html#invoke-with-event) 上找到。

:::

:::info 导出器的 lambda 可以运行超过 15 分钟(Lambda 函数的最长运行时间)。 如果函数运行超过 10 分钟，并且还有任何资源需要同步，则会启动一个新的 lambda 实例来继续同步过程。

:::

### 故障排除

#### 查看日志

要在一个地方查看所有 lambda 实例的日志，可以使用[Cloudwatch Logs](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html#monitoring-cloudwatchlogs-console) 或[AWS SAM Logs](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html) : 

```bash showLineNumbers
sam logs --stack-name serverlessrepo-port-aws-exporter --tail
```

### 更新日程表设置

首次成功运行导出程序后，您可能想要更改 CloudFormation 堆栈的调度相关属性: 

* `ScheduleExpression` - 确保设置的时间间隔长于输出程序的执行时间；
* `ScheduleState` - 将计划状态设置为 `ENABLED`。

如果被用于 Terrafom 模块，请更新 `schedule_state` 和 `schedule_expression` 变量。

:::info 为了确定 lambda 的执行时间，可以通过[view the logs](#view-the-logs) ，搜索第一行和最后一行日志。当输出程序完成同步工作时，它会写入以下日志: "Done handling your resources"("完成处理您的资源")。

:::

:::tip  更新应用程序 更新应用程序设置或版本的步骤与部署新应用程序的步骤相同，类似于[installation](#installation) 的第 3 步。默认情况下，运行更新/部署程序时将被用于最新可用的导出程序版本。

更多详情，请点击[here](https://docs.aws.amazon.com/serverlessrepo/latest/devguide/serverlessrepo-how-to-consume-new-version.html) 。

:::

#### 配置 AWS 输出程序以在事件中运行

除了按计划运行外，AWS 导出器还可用于对实时事件采取action，如创建、更新和删除 AWS 账户中的资源。 这样，您就可以配置资源，以便在其发生变化时立即实时同步。

要配置 AWS 输出程序将事件用作触发器，请按照以下步骤操作: 

1. 根据与您希望 AWS 输出程序实时更新的资源相匹配的特定事件，准备一个[event rule](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/event-based-updates#definition) ，并将其保存到 Cloudformation YAML 模板 (`template.yml`) 。
2. 使用此命令部署事件规则: 


   ```bash showLineNumbers
   aws cloudformation deploy --template-file template.yml --stack-name port-aws-exporter-event-rules
   ```


## 更多信息

* 有关实用配置及其相应的蓝图定义，请参阅[examples](./examples.md) 页面；
* 了解[working with Cloudformation stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) 的更多方式；
* 深入 dive[Event-based Updates](./event-based-updates.md) 机理。
