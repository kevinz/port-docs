---
sidebar_position: 1
---

import Tabs from "@theme/Tabs";
import TabItem from "@theme/TabItem";
import Image from "@theme/IdealImage";

# AWS

我们与 AWS 的集成可根据您的配置将 AWS 资源导出到 Port。 您可以将集成定义为按计划运行和按事件运行。

我们与 AWS 的集成支持实时事件处理，因此可以在 Port 内部准确、实时地呈现 AWS 基础设施。

:::tip Port 的 AWS 输出程序是开源的，请查看源代码[**here**](https://github.com/port-labs/port-aws-exporter) 。

:::

## 💡 AWS 输出程序常见用例

例如，我们的 AWS 导出器可让您利用 AWS 账户中的数据轻松丰富软件目录: 

* 映射账户中的资源，包括 **S3数据桶**、**lambda函数**、**SQS队列**、**RDS DB实例**、**ECS服务**和许多其他资源类型。
* 被用于关系可在 Port 内创建完整、易懂的 AWS 账户地图。

## 工作原理

Port 的 AWS 输出程序可以检索[AWS Cloud Control API](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/supported-resources.html) 支持的所有资源。

开源 AWS 导出器允许您将 AWS 中的数据提取、转换、加载(ETL)到所需的软件目录数据模型中。

出口程序是使用安装在账户上的[AWS serverless application](https://aws.amazon.com/serverless/sam/) 部署的。

[The serverless application](#exporter-aws-serverless-application) 需要一个[JSON configuration file](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/#exporter-configjson-file) 来描述将数据加载到开发者门户的 ETL 流程，还需要一个具有必要权限的[IAM policy](/build-your-software-catalog/sync-data-to-catalog/cloud-providers/aws/#iam-policy) 来列出和读取配置的资源。

导出器利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 AWS 对象中的现有字段和值进行选择、修改、连接、转换和其他操作。

### 导出器 `config.json` 文件

通过 `config.json` 文件，您可以指定要从 AWS 账户导出的确切资源，以及要在 Port 中填写数据的实体和属性。

下面是`config.json`文件的示例片段，演示了将`lambda 函数'数据从账户导入软件目录的 ETL 过程: 

```json showLineNumbers
{
  "resources": [
    {
      "kind": "AWS::Lambda::Function",
      "selector": {
        "aws": {
          "regions": ["us-east-1", "us-west-1"]
        }
      },
      "port": {
        "entity": {
          "mappings": [
            {
              "identifier": ".FunctionName",
              "title": ".FunctionName",
              "blueprint": "function",
              "properties": {
                "memory": ".MemorySize",
                "timeout": ".Timeout",
                "runtime": ".Runtime"
              }
            }
          ]
        }
      }
    }
  ]
}
```

#### 结构

* config.json "文件的根键是 "resources "键；
* kind "键是 AWS 云控制 API 中资源类型的指定符，遵循 "service-providers::service-name::data-type-name "格式: 


  ```json showLineNumbers
  # highlight-next-line
  "resources": [
    {
      # highlight-next-line
      "kind": "AWS::Lambda::Function",
      ...
  ```


::tip 要使用 AWS CLI 生成受支持的资源类型列表，请结合以下命令的结果: 


  ```bash showLineNumbers
  aws cloudformation list-types --type RESOURCE --visibility PUBLIC --provisioning-type FULLY_MUTABLE
  aws cloudformation list-types --type RESOURCE --visibility PUBLIC --provisioning-type IMMUTABLE
  aws cloudformation list-types --type RESOURCE --visibility PRIVATE --provisioning-type FULLY_MUTABLE
  aws cloudformation list-types --type RESOURCE --visibility PRIVATE --provisioning-type IMMUTABLE
  ```


要确定是否支持特定的 `<RESOURCE_TYPE>`，请使用此命令(需要 JQ，输出为 `true` 或 `false`): 


  ```bash showLineNumbers
  aws cloudformation describe-type --type RESOURCE --type-name <RESOURCE_TYPE> --query "ProvisioningType" | jq -c '. == "FULLY_MUTABLE" or . == "IMMUTABLE"'
  ```


更多信息，请阅读[**here**](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-types.html#resource-types-determine-support) : 

* selector "键可以让你准确地筛选出指定 "类型 "中的哪些对象将被收录到软件目录中；
* 选择器 "中的 "查询 "键是一个 JQ 布尔值查询。如果对某个对象的评估值为 false，则不会同步该对象；
* 选择器 "中的 "aws "键允许你指定要同步的 "区域 "列表。


  ```json showLineNumbers
  "resources": [
    {
      "kind": "AWS::Lambda::Function",
      # highlight-start
      "selector": {
        "query": "true",
        "aws": {
          "regions": [
            "us-east-1",
            "us-west-1"
          ]
        }
      },
      # highlight-end
      "port":
      ...
  ```


:::tip 您可以选择在每个地区部署导出器，也可以选择在一个地区部署导出器。 如果您想在一个地区安装导出器，但要导出多个地区的数据，"地区 "配置键可能会有用: 

一些被用于的示例: 

* 要同步默认区域中指定 "类型 "的所有对象，请勿指定 "选择器"；
* 同步不属于 AWS Amplify 服务的所有 lambdas: 


    ```json showLineNumbers
    query: .FunctionName | startswith("amplify") | not
    ```


* 等等

信息

* 对于某些资源，您必须提供额外的信息。例如，为了同步特定负载平衡器的所有 `AWS ELB 监听器`，请使用 `regions_config` 和 `resources_models` 键: 


    ```json showLineNumbers
    "selector": {
        "aws": {
          "regions": [
            "eu-west-1"
          ],
          # highlight-start
          "regions_config": {
            "eu-west-1": {
              "resources_models": [
                "{\"LoadBalancerArn\": \"<AWS_ELB_ARN>\"}"
              ]
            }
          }
          # highlight-end
        }
    }
    ```


带有所需属性的特殊资源表可在[here](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations-list.html#resource-operations-list-containers) 上找到: 

* Port"、"实体 "和 "映射 "键打开了用于将 AWS 资源字段映射到Port实体的部分，"映射 "键是一个数组，其中每个对象都与[entity](/build-your-software-catalog/sync-data-to-catalog/sync-data-to-catalog.md#entity-json-structure) 的结构相匹配。
* 除了 `blueprint` 必须是静态字符串外，每个映射值都是 JQ 查询。
* 映射中的 "itemsToParse "键使从单个 AWS 资源创建多个实体成为可能。
    - 任何 JQ 表达式都可以在这里被用于，只要它的 evaluated 值是一个项目数组。
    - 项目 "将作为一个键添加到 JQ 上下文中，该键包含对 "itemsToParse "中指定的数组中的项目的引用。对于对象数组，可以使用 `.item.KEY_NAME`语法访问对象中的键。
   <br></br>


  ```json showLineNumbers
  "resources": [
    {
      "kind": "AWS::Lambda::Function",
      "selector": {
        "aws": {
          "regions": [
            "us-east-1",
            "us-west-1"
          ]
        }
      },
      # highlight-start
      "port": {
        "entity": {
          "mappings": [
            {
              "identifier": ".FunctionName",
              "title": ".FunctionName",
              "blueprint": "function",
              "properties": {
                "memory": ".MemorySize",
                "timeout": ".Timeout",
                "runtime": ".Runtime"
              }
            },
            {
              "itemsToParse": ".Layers",
              "identifier": ".item",
              "title": ".item",
              "blueprint": "functionLayer",
              "properties": {}
            }
          ]
        }
      }
      # highlight-end
    }
    ...
  ```


:::info  **重要**

* **当资源之间存在关系时，资源的顺序很重要**。
AWS 输出程序将按照`config.json`中的顺序同步资源，因此请确保按照逻辑顺序对资源进行排序。
    例如，如果您有从 SNS Topic 到 lambda 函数的关系，请将 Lambda 函数配置放在前面。
* 从本质上讲，AWS 输出程序将保持未映射属性的值不变。有关不同实体创建策略的进一步解释，请访问[here](/build-your-software-catalog/sync-data-to-catalog/api/api.md?operation=create-update#usage) 。

:::

:::tip  查看资源类型模式 要查看资源类型模式并用它来组成映射，请使用[this](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) 引用。

请注意，并非参考资料中列出的所有资源类型都可与云控制 API 配合使用。确定资源类型是否可用的方法讨论了[here](#structure) 。

有关其他选项和信息，请阅读[here](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-types.html#resource-types-schemas) 。

:::

#### 更改配置

默认情况下，出口程序会在安装时将预先配置好的 `config.json` 文件保存到[`AWS S3 Bucket`](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingBucket.html) 。S3 存储桶名为 `port-aws-exporter-<AWS_REGION>-<AWS_ACCOUNT_ID>`，由出口程序创建。

更改配置(例如更改某个属性的映射): 

1. 从数据桶中创建一个 `config.json` 文件副本，进行所需的更改，并将其保存到本地。
2. 使用 AWS CLI 将文件上传到数据桶，替换现有文件。  
用您的 Values 替换 `<BUCKET_NAME>` 和 `<PATH_TO_CONFIG_FILE>`，然后运行以下命令: 

```bash
aws s3api put-object --bucket "<BUCKET_NAME>" --key "config.json" --body "<PATH_TO_CONFIG_FILE>"
```

### IAM 政策

AWS 导出器使用[`AWS IAM Policy`](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html) ，其中指定了要导出的 AWS 资源(在[`config.json`](#exporter-configjson-file) 中配置的资源)的 `list` 和 `read` 权限 。

例如，要导出 "lambda 函数"，需要创建一个定义如下的策略: 

```json showLineNumbers
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "lambda:GetFunction",
        "lambda:GetFunctionCodeSigningConfig",
        "lambda:ListFunctions"
      ],
      "Resource": "*"
    }
  ]
}
```

:::tip  查找策略的资源操作 为了查找应添加到策略中的资源操作，应查看资源类型模式，并查找**read** 和 **list**`处理程序`的权限。

使用以下命令获取特定 `<RESOURCE_TYPE>` 的资源操作(需要 AWS CLI 和 JQ): 

```bash showLineNumbers
aws cloudformation describe-type --type RESOURCE --type-name <RESOURCE_TYPE> --query "Schema" | jq -c 'fromjson | .handlers | with_entries(select([.key] | inside(["list", "read"]))) | map(.permissions) | flatten'
```

更多详细信息，请访问[here](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-types.html#resource-types-schemas) 。

:::

### Port证书secret

为了管理 Port 中的实体，出口程序需要访问您的 Port 凭据，这些凭据通过[`AWS Secrets Manager`](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html) secret 提供给出口程序。

secret值应采用以下格式: 

```json showLineNumbers
{ "id": "<CLIENT_ID>", "clientSecret": "<CLIENT_SECRET>" }
```

与邮政信箱类似，您可以创建和管理自己的secret，也可以委托出口商管理。 无论哪种方式，您都需要在secret准备就绪后，自行将 Port 的凭据添加到secret中。

#### 导出器 AWS 无服务器应用程序

出口商 AWS 无服务器应用程序 "是您安装出口商[CloudFormation stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html) 的方式。

堆栈由几个部分组成: 

* [S3 bucket](#changing-the-configuration) - 应保存[`config.json`](#exporter-configjson-file) 的位置；
* [ASM secret](#port-credentials-secret) - 应保存 Port 凭据(客户 ID 和 secret)的位置 ，以便输出程序与 Port 的 API 交互；
* [Lambda function](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-concepts.html#gettingstarted-concepts-function) - 您可以调用以运行输出程序代码的资源。[IAM policy](#iam-policy) 连接到 Lambda 函数的执行角色；
* [SQS queue](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html) - 事件队列，供输出程序使用。请阅读[here](./event-based-updates.md) ，了解如何使用导出器消费来自不同 AWS 服务的实时事件并对其采取行动；
* [EventsBridge scheduled rule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html) - 按计划运行输出程序的规则。

## 开始

继续访问[Installation](./Installation.md) 部分，在您的 Port 环境中设置 AWS 输出程序。
