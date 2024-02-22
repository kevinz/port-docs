---
sidebar_position: 1
---

# AWS 输出模块

AWS 输出程序模块被用于用于在 AWS 账户中部署 Port AWS 输出程序。

:::note  注意 有关完整安装，包括资源蓝图和根据 AWS 事件更新 AWS 蓝图、

请使用完整的 ** [AWS exporter Terraform installation](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/Installation.md#terraform-installation-recommended)**

:::

## 先决条件

在使用本模块之前，请确保您已完成以下前提条件: 

1. 在本地计算机上安装和配置 AWS 命令行界面 (CLI)。
    有关说明，请参阅[AWS CLI Documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) 。
2.导出您的[Port credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials) : 


   ```bash
   export PORT_CLIENT_ID="YOUR_PORT_CLIENT_ID"
   export PORT_CLIENT_SECRET="YOUR_PORT_CLIENT_SECRET"
   ```


## 部署输出程序

1. 创建 AWS 输出程序[config.json](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/aws.md#exporter-configjson-file) 文件。
2. 将模块添加到 HCL 配置中。您可以使用这个简单的 HCL。
      <details>
      <summary>HCL example</summary>

   ```hcl
   data "aws_region" "current" {}
   data "aws_caller_identity" "current" {}

   module "port_aws_exporter" {
      source  = "port-labs/port-exporter/aws"
      version = "0.1.1"
      config_json   = "${path.module}/examples/run_module_example/config.json"
      lambda_policy = "${path.module}/defaults/policy.json"
      bucket_name = "port-aws-exporter-${data.aws_region.current.name}-${data.aws_caller_identity.current.account_id}"
   }
   ```

   </details>

### Running the module

After configuring the module, run the following Terraform commands:

```bash
terraform init       # Initialize the Terraform configuration.
terraform plan       # Preview the changes to be applied
terraform apply      # Apply the changes and provision the resources in your AWS account, providing the path to your variables file using the --var-file option.
```

### 变量

您可以使用内置模块变量配置和自定义 AWS 输出程序: 

<details>
<summary>View all variables</summary>

`stack_name` - CloudFormation 堆栈的名称。

`secret_name` - Port凭据的secret名称。

您也可以使用 `custom_port_credentials_secret_arn` 变量来提供现有的secret。

`create_bucket` - 是为出口程序配置创建一个新的数据桶，还是使用现有的数据桶。

`bucket_name` - 用于导出配置的 bucket 名称。 Lambda 也会用它来写入中间临时文件。

`config_json` -(必填)导出器配置的文件路径/JSON 格式字符串。

`config_s3_key` - (必填)导出器配置的 s3 密钥名称。

`function_name` - AWS Lambda 函数的名称。

iam_policy_name` - Port 输出者角色的策略名称。

custom_port_credentials_secret_arn` - (可选)Port凭据(客户 ID 和客户secret)的secret ARN。

secret值的格式应为: `{"id":"<PORT_CLIENT_ID>"，"clientSecret":"<PORT_CLIENT_SECRET>"}`

`lambda_policy` -(可选)要授予 Lambda 函数的 AWS 策略 json 的路径或 JSON 格式字符串。 如果未被用于，则使用默认的导出器策略。

`events_queue_name` - 发送到 Port 输出程序的事件队列名称。

`schedule_state` - `ENABLED` 或 `DISABLED`。 建议仅在成功运行一次后启用。 同时确保更新计划表达式的时间间隔，使其长于出口程序的执行时间。

`schedule_expression` -(必填) 时间表表达式，用于定义输出程序的事件时间表，如下所示[spec](https://docs.aws.amazon.com/lambda/latest/dg/services-cloudwatchevents-expressions.html) 。

</details>

:::tip  LAMBDA FUNCTION IAM POLICY 默认情况下，导出程序将引用[default exporter policy](https://github.com/port-labs/terraform-aws-port-exporter/blob/main/defaults/policy.json).

为了使用自定义[AWS policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) ，请创建一个新的 `policy.json` 文件，并将其路径传递给 `lambda_policy_file` 变量。

:::

### 导出器 CloudFormation 堆栈

完成安装后，您将在 CloudFormation Stacks 中看到 Port 输出程序部署，其名称为

`serverlessrepo-<exporter_stack_name>` - 基于 `stack_name` 变量。

:::note 堆栈将部署到用户 AWS CLI 中配置的 AWS 区域

:::

### 删除出口商

要删除 AWS 输出程序模块，只需运行

```bash
terraform destroy
```

## 更多信息

* 有关出口程序的更多信息，请参阅[AWS exporter docs](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/) 。
* 访问 Terraform 模块[Github repository](https://github.com/port-labs/terraform-aws-port-exporter) : 
    - [module example](https://github.com/port-labs/terraform-aws-port-exporter/tree/main/examples/run_module_example) 文件夹，查看 Terraform 模块的更完整示例。
    - Event Bridge 规则[Terraform example](https://github.com/port-labs/terraform-aws-port-exporter/tree/main/examples/terraform_deploy_eventbridge_rule) ，以了解为 AWS 输出程序部署[Eventbridge rule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) 的示例。
