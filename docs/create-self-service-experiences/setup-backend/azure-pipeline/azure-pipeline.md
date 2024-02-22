# Azure Pipelines 自助操作

Port 可通过[incoming webhooks triggers](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops&amp;tabs=schema#define-a-webhooks-resource) 触发[Azure pipelines](https://azure.microsoft.com/en-us/products/devops/pipelines) 。

![AzurePipelinesArchitecture](../../../../static/img/self-service-actions/portAzurePipelineArchitecture.png)

上图所示步骤如下: 

1. 在 Port 中调用一个操作；
2. Port 使用 SHA-1 对操作有效载荷进行签名，签名值为[`clientSecret`](../../../build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) ，并将其放入 `X-Port-Signature` 请求头中。
    信息
    使用请求标头验证 webhook 请求有以下好处: - 确保请求有效载荷未被篡改
    - 确保信息发送方是 Port
    - 确保收到的信息不是旧信息的重放
    :::
3.Port 通过 `POST` 请求将调用的 `WEBHOOK` 发布到 `https://dev.azure.com/{org_name}/_apis/public/distributedtask/webhooks/{webhook_name}?api-version=6.0-preview`.

流程示例如下

1. 开发人员要求运行 Azure Pipelines；
2. Port 向 Azure webhook `URL` 发送带有操作有效载荷的 `POST` 请求；
3. Azure webhook 接收新的操作请求；
4. Azure 网络钩子触发 Pipelines；

## 在 Azure 中定义传入 webhook

要在 Azure 中定义传入 webhook，请按照以下步骤操作: 

1. 创建一个**服务连接**，类型为**传入 Webhook**；
2. 在 **Secret** 关键字段中输入您的 Port `clientSecret` 值；
3. 将 "X-Port-Signature "头放入 "Headers "字段；
4. 在 "服务连接名称 "字段中输入服务连接名称；
5. 在 Azure Pipelines yaml 中添加服务连接资源: 


   ```yaml
   resources:
     webhooks:
       - webhook: { webhookName }
         connection: { Service connection name }
   ```

   The complete documentation showing how to configure Azure incoming webhooks can be found [here](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/resources?view=azure-devops&tabs=schema#define-a-webhooks-resource).

## 在 Port 中定义 Azure 管道操作

要在 Port 中定义 Azure 管道调用方法，请按照以下步骤操作: 

1. 转到要配置动作的蓝图；
2. 按照[actions](../../../create-self-service-experiences/setup-ui-for-action/#action-structure) 页面中的说明添加新操作；
3. 在 `invocationMethod` 关键字中添加以下信息: 

```json showLineNumbers
{
  ...
  "userInputs": {
    ...
  },
  // highlight-start
  "invocationMethod": {
    "type": "AZURE-DEVOPS",
    "org": "<AZURE-DEVOPS-ORG>",
    "webhook": "<AZURE-DEVOPS-WEBHOOK-NAME>"
  },
  // highlight-end
  "trigger": "CREATE"
  ...
}
```

:::tip 

* `<AZURE-DEVOPS-ORG>` - 您的 Azure DevOps 组织名称，可在您的 Azure DevOps URL 中找到: https://dev.azure.com/{azure-devops-org}`；
* `<AZURE-DEVOPS-WEBHOOK-NAME>` - 你在 Azure yaml Pipelines 文件中给 webhook 资源起的名称。

:::

## 示例

有关使用 Azure Pipelines 的实用自助操作，请引用[deployment example](./examples/run-service-deployment.md) 页面。