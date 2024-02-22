---
sidebar_position: 3
---

# 基于事件的更新

通过我们的 AWS 集成，您可以根据实时事件(除标准的计划同步外)触发 AWS 资源与 Port 的同步。 因此，在创建、更新或删除资源后，您的软件目录将很快得到更新。

## 工作原理

许多 AWS 服务都会向[AWS EventBridge](https://aws.amazon.com/eventbridge/) 服务和账户的默认事件总线发送事件。此外，[AWS CloudTrail](https://aws.amazon.com/cloudtrail/) 服务还会自动为来自大多数 AWS 服务的 API 调用发送事件。

用户可以创建一个[AWS EventBridge rule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-rules.html) ，用于消费和转换特定事件，并将其发送至目标，如[AWS SQS Queue](https://aws.amazon.com/sqs/) 。

[The AWS exporter application](./aws.md#exporter-aws-serverless-application) 在安装过程中会创建一个 SQS 队列，并将其配置为输出程序 lambda 的事件源。

这样，您就可以设置一个事件规则，该规则会消耗总线上的任何事件，并将它们发送到 AWS 输出程序的队列中。

在到达队列之前，每个事件都必须作为事件规则的一部分进行转换，以适应出口程序的 lambda 预期事件结构。 转换的目标是将事件映射到出口程序可以处理的单个 AWS 资源。

队列中的事件会被出口程序的 lambda 自动消耗，该 lambda 会将相关 AWS 资源的更新状态与 Port 同步。

###单一事件的输入结构

AWS 输出程序的 lambda 接受具有以下结构的 JSON 事件: 

* `resource_type` - 一个硬编码字符串，代表在 AWS Exporter 的[`config.json`](./aws.md#exporter-configjson-file) 中配置的 AWS 资源类型；
* `region` - 从事件中获取区域的 JQ 查询，通常值为`"\"<awsRegion>\""`；
* action`(可选，默认为 `upsert`)- 用于定义所需操作的 JQ 查询: "upsert "或 "delete "匹配资源的 Port 实体。通常基于事件名称，如下面的示例；
* identifier` - 用于计算资源标识符的 JQ 查询。如果操作是 "upsert"，则应是 AWS Cloud Control API 资源标识符。否则，对于 "删除 "操作，应使用Port实体标识符。建议使用与云控制 API 资源标识符相同的Port实体标识符(如适用)，这样就可以对上载和删除事件使用相同的事件规则(同时也无需使用复杂的 JQ 过滤模式)。

此类 JSON 事件的示例

```json showLineNumbers
{
  "requestFunctionName": "<requestFunctionName>",
  "responseFunctionName": "<responseFunctionName>",
  "eventName": "<eventName>",
  # highlight-start
  "resource_type": "AWS::Lambda::Function",
  "region": "\"<awsRegion>\"",
  "action": "if .eventName | startswith(\"Delete\") then \"delete\" else \"upsert\" end",
  "identifier": "if .responseFunctionName != \"\" then .responseFunctionName else .requestFunctionName end"
  # highlight-end
}
```

:::tip 要获取特定资源类型(如 `AWS::Lambda::Function`)的标识符公式，可以使用以下命令: 

```bash showLineNumbers
aws cloudformation describe-type --type RESOURCE --type-name AWS::Lambda::Function --query "Schema" | jq 'fromjson | .primaryIdentifier | join("|")'

# Result
"/properties/FunctionName"
```

从这里可以看出，lambda 函数的标识符就是函数名。

另一个例子是 `AWS::ECS::Service` 资源类型: 

```bash showLineNumbers
aws cloudformation describe-type --type RESOURCE --type-name AWS::ECS::Service --query "Schema" | jq 'fromjson | .primaryIdentifier | join("|")'

# Result
"/properties/ServiceArn|/properties/Cluster"
```

这里的公式中有两个属性: `/properties/ServiceArn` 和`/properties/Cluster`。 当有多个属性时，标识符中的值用`|`分隔。

更多信息，请阅读[**here**](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-identifier.html) 。

:::

### 活动规则

#### 定义

通过事件规则，您可以指定要消费的确切事件，并定义将修改后的事件发送到的目标(包括对输入的转换)。

在深入 dive[event rule properties](#properties) 之前，这里有一个用于管理 AWS 输出程序事件规则的 "CloudFormation YAML 模板 "的完整示例: 

```yaml showLineNumbers
AWSTemplateFormatVersion: "2010-09-09"
Description: The template used to create event rules for the Port AWS exporter.

Parameters:
  PortAWSExporterStackName:
    Description: Name of the Port AWS exporter stack name
    Type: String
    MinLength: 1
    MaxLength: 255
    AllowedPattern: "^[a-zA-Z][-a-zA-Z0-9]*$"
    Default: serverlessrepo-port-aws-exporter

Resources:
  EventRule0:
    Type: AWS::Events::Rule
    Properties:
      EventBusName: default
      EventPattern:
        source:
          - aws.lambda
        detail-type:
          - AWS API Call via CloudTrail
        detail:
          eventSource:
            - lambda.amazonaws.com
          eventName:
            - prefix: UpdateFunctionConfiguration
            - prefix: CreateFunction
            - prefix: DeleteFunction
      Name: port-aws-exporter-sync-lambda-trails
      State: ENABLED
      Targets:
        - Id: "PortAWSExporterEventsQueue"
          Arn:
            {
              "Fn::ImportValue":
                { "Fn::Sub": "${PortAWSExporterStackName}-EventsQueueARN" },
            }
          InputTransformer:
            InputPathsMap:
              awsRegion: $.detail.awsRegion
              eventName: $.detail.eventName
              requestFunctionName: $.detail.requestParameters.functionName
              responseFunctionName: $.detail.responseElements.functionName
            InputTemplate: |-
              {
                "resource_type": "AWS::Lambda::Function",
                "requestFunctionName": "<requestFunctionName>",
                "responseFunctionName": "<responseFunctionName>",
                "eventName": "<eventName>",
                "region": "\"<awsRegion>\"",
                "identifier": "if .responseFunctionName != \"\" then .responseFunctionName else .requestFunctionName end",
                "action": "if .eventName | startswith(\"Delete\") then \"delete\" else \"upsert\" end"
              }
```

:::info 这个 CloudFormation 堆栈示例可用于对 lambda 函数的更改采取行动。 此外，它还包含一个名为 "PortAWSExporterStackName "的参数，该参数指的是主出口程序的堆栈名称。 这样，我们就可以将事件发送到现有出口程序的事件队列，以同步相关的 AWS 资源。

:::

#### Properties

让我们回顾一下上述事件规则资源的属性。

* EventBusName "属性值为 "default"，因为这是自动从各种 AWS 服务获取事件的总线: 

```yaml showLineNumbers
Properties:
  # highlight-next-line
  EventBusName: default
  ...
```

* EventPattern "属性定义了要消费的事件，包括要处理的事件源、根据事件结构应用的过滤器。在此，我们定义了一种事件模式，以消费来自 lambda 服务的 Cloudtrail API 调用；在此基础上，我们仅过滤了 `UpdateFunctionConfiguration`、`CreateFunction` 和 `DeleteFunction` 事件: 

```yaml showLineNumbers
Properties:
  EventBusName: default
  # highlight-start
  EventPattern:
    source:
      - aws.lambda
    detail-type:
      - AWS API Call via CloudTrail
    detail:
      eventSource:
        - lambda.amazonaws.com
      eventName:
        - prefix: UpdateFunctionConfiguration
        - prefix: CreateFunction
        - prefix: DeleteFunction
  # highlight-end
```

:::tip 要组成事件模式，请了解有关 AWS 服务事件的更多信息，包括不同来源和事件名称[here](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-service-event.html) 。

有关 CloudTrail 支持的服务，请单击[here](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-aws-service-specific-topics.html) 。

:::

* 目标 "属性定义了生成事件的目标。对于 AWS 输出程序，我们希望将其自身的事件队列配置为目标: 

```yaml showLineNumbers
Properties:
  ...
  # highlight-start
  Targets:
    - Id: "PortAWSExporterEventsQueue"
      Arn: { "Fn::ImportValue" : {"Fn::Sub": "${PortAWSExporterStackName}-EventsQueueARN" } }
      ...
  # highlight-end
```

* 目标 "属性中的 "InputTransformer "键定义了在将事件发送到目标队列之前将应用于事件的转换；

```yaml showLineNumbers
Properties:
  ...
  Targets:
    - Id: "PortAWSExporterEventsQueue"
      Arn: { "Fn::ImportValue" : {"Fn::Sub": "${PortAWSExporterStackName}-EventsQueueARN" } }
      # highlight-next-line
      InputTransformer:
        ...
```

* 输入转换器 "中的 "InputPathsMap "键定义了要从事件中提取的字段。它被用于[`JSONPath`](https://goessner.net/articles/JsonPath/index.html) 语法来配置提取: 

```yaml showLineNumbers
Properties:
  ...
  Targets:
    - Id: "PortAWSExporterEventsQueue"
      Arn: { "Fn::ImportValue" : {"Fn::Sub": "${PortAWSExporterStackName}-EventsQueueARN" } }
      InputTransformer:
        # highlight-start
        InputPathsMap:
          awsRegion: $.detail.awsRegion
          eventName: $.detail.eventName
          requestFunctionName: $.detail.requestParameters.functionName
          responseFunctionName: $.detail.responseElements.functionName
        # highlight-end
        ...
```

例如，为 `CreateFunction` lambda API 调用发送到 EventBridge 总线的事件具有以下结构: 

```json showLineNumbers
{
  ...
  "detail": {
    ...
    "eventSource": "lambda.amazonaws.com",
    "eventName": "CreateFunction20150331",
    "awsRegion": "eu-west-1",
    ...
    "requestParameters": {
        "functionName": "test-function",
        ...
    },
    "responseElements": {
        "functionName": "test-function",
        ...
    },
    ...
  }
}
```

下面示例中的 `InputPathsMap` 将只从 API 请求和响应中提取 `awsRegion`, `eventName` 和 `functionName`。

* InputTransformer` 中的 `InputTemplate` 键定义了将发送到目标队列的最终输入的模板，格式为输出器的 lambda[expects](#input-structure-for-a-single-event) 。该模板会被用于 `InputPathsMap` 中的 Values 来渲染: 

```yaml showLineNumbers
Properties:
  ...
  Targets:
    - Id: "PortAWSExporterEventsQueue"
      Arn: { "Fn::ImportValue" : {"Fn::Sub": "${PortAWSExporterStackName}-EventsQueueARN" } }
      InputTransformer:
        InputPathsMap:
          awsRegion: $.detail.awsRegion
          eventName: $.detail.eventName
          requestFunctionName: $.detail.requestParameters.functionName
          responseFunctionName: $.detail.responseElements.functionName
        # highlight-start
        InputTemplate: |-
          {
            "resource_type": "AWS::Lambda::Function",
            "requestFunctionName": "<requestFunctionName>",
            "responseFunctionName": "<responseFunctionName>",
            "eventName": "<eventName>",
            "region": "\"<awsRegion>\"",
            "identifier": "if .responseFunctionName != \"\" then .responseFunctionName else .requestFunctionName end",
            "action": "if .eventName | startswith(\"Delete\") then \"delete\" else \"upsert\" end"
          }
        # highlight-end
```

在这里，"InputTemplate"(输入模板)接收从 "InputPathsMap"(输入路径图)中提取的信息，并构建一条消息，该消息将被发送到 SQS 队列，并被出口程序的 lambda 消耗。 消息包括: 

* 硬编码资源类型(`AWS::Lambda::Function`)；
* 事件中的 `awsRegion` 字段；
* JQ 查询，根据事件名称确定操作("上载 "或 "删除 "Port实体)；
* 根据 API 响应或请求中的函数名计算标识符的 JQ 查询。

:::info 要了解有关 EventBridge 中输入变压器的更多信息，请单击[here](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-transform-target-input.html) 。

:::

## 下一步

### 示例

* 有关实用配置及其相应蓝图定义，请参阅[examples](./examples.md) 页面。
* 参考事件桥规则[Terraform example](https://github.com/port-labs/terraform-aws-port-exporter/tree/main/examples/terraform_deploy_eventbridge_rule)
