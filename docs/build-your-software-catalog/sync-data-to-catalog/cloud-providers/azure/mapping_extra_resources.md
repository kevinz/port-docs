---

sidebar_position: 3

---

import StorageAccountBlueprint from './examples/storage/_storage_account_blueprint.mdx'
import StorageAppConfig from './examples/storage/_port_app_config.mdx'

# 制图额外资源

您可能已经浏览过[Examples](./examples.md) 页面，注意到 Azure 输出程序支持许多 Azure 资源，但并非所有资源都在示例页面中进行了记录。

本页面将帮助您了解 Azure 集成支持哪些 Azure 资源，以及如何将这些资源映射到 Port。

## Azure 导出器是否支持该资源？

Azure 导出器依赖于基础 Azure 资源通过 `subscription` 拥有的 `List` API 来获取特定类型的所有资源。 如果是示例页面中没有记录的资源，可以通过以下步骤将其映射到 Port: 

1. 在[Azure REST API reference](https://learn.microsoft.com/en-us/rest/api/azure/) 中查找资源。
2. 如果是基础资源(格式为 `Microsoft.<Provider>/<resourceName>`)，可以在类型示例响应和 API URL 中找到！[Storage Account Type](/img/integrations/azure-exporter/StorageAccountTypeInAPIURL.png)
    - 如果它有订阅的列表 API，Azure 输出程序将支持它。
    ![Storage Account List API](/img/integrations/azure-exporter/StorageAccountListAPIIsValid.png)
3.如果是扩展资源(格式为 `Microsoft.<Provider>/<resourceName>/<extensionName>`)，如 `Microsoft.Storage/storageAccounts/blobServices/containers`。
    - 然后，如果 Azure 导出器支持基础资源，则也将支持扩展资源。

## 将资源映射到Port

在 Azure REST API 参考中找到资源并确认 Azure 输出程序支持该资源后，就可以按照以下步骤将其映射到 Port: 

#### 蓝图

为资源创建 Port 蓝图定义。 该蓝图定义基于 Azure REST API 参考中的资源 API 响应。

<details>
<summary>Azure Storage Account Rest API Response Example</summary>

```json
{
  "value": [
    {
      "id": "/subscriptions/{subscription-id}/resourceGroups/res2627/providers/Microsoft.Storage/storageAccounts/sto1125",
      "kind": "Storage",
      "location": "eastus",
      "name": "sto1125",
      "properties": {
        "isHnsEnabled": true,
        "creationTime": "2017-05-24T13:28:53.4540398Z",
        "primaryEndpoints": {
          "web": "https://sto1125.web.core.windows.net/",
          "dfs": "https://sto1125.dfs.core.windows.net/",
          "blob": "https://sto1125.blob.core.windows.net/",
          "file": "https://sto1125.file.core.windows.net/",
          "queue": "https://sto1125.queue.core.windows.net/",
          "table": "https://sto1125.table.core.windows.net/",
          "microsoftEndpoints": {
            "web": "https://sto1125-microsoftrouting.web.core.windows.net/",
            "dfs": "https://sto1125-microsoftrouting.dfs.core.windows.net/",
            "blob": "https://sto1125-microsoftrouting.blob.core.windows.net/",
            "file": "https://sto1125-microsoftrouting.file.core.windows.net/",
            "queue": "https://sto1125-microsoftrouting.queue.core.windows.net/",
            "table": "https://sto1125-microsoftrouting.table.core.windows.net/"
          },
          "internetEndpoints": {
            "web": "https://sto1125-internetrouting.web.core.windows.net/",
            "dfs": "https://sto1125-internetrouting.dfs.core.windows.net/",
            "blob": "https://sto1125-internetrouting.blob.core.windows.net/",
            "file": "https://sto1125-internetrouting.file.core.windows.net/"
          }
        },
        "primaryLocation": "eastus",
        "provisioningState": "Succeeded",
        "routingPreference": {
          "routingChoice": "MicrosoftRouting",
          "publishMicrosoftEndpoints": true,
          "publishInternetEndpoints": true
        },
        "encryption": {
          "services": {
            "file": {
              "keyType": "Account",
              "enabled": true,
              "lastEnabledTime": "2019-12-11T20:49:31.7036140Z"
            },
            "blob": {
              "keyType": "Account",
              "enabled": true,
              "lastEnabledTime": "2019-12-11T20:49:31.7036140Z"
            }
          },
          "keySource": "Microsoft.Storage"
        },
        "secondaryLocation": "centraluseuap",
        "statusOfPrimary": "available",
        "statusOfSecondary": "available",
        "supportsHttpsTrafficOnly": false
      },
      "sku": {
        "name": "Standard_GRS",
        "tier": "Standard"
      },
      "tags": {
        "key1": "value1",
        "key2": "value2"
      },
      "type": "Microsoft.Storage/storageAccounts"
    }
  ]
}
```

</details>

根据 API 响应，可以为资源创建 Port 蓝图定义。

<StorageAccountBlueprint/>

### 整合配置

为资源创建集成配置。 集成配置是一个 YAML 文件，描述了将数据加载到开发者门户的 ETL 流程。

<StorageAppConfig/>

#### 集成配置结构

* 类型 "字段描述了要摄入 Port 的 Azure 资源类型。类型 "字段应设置为[resource guide](https://learn.microsoft.com/en-us/azure/templates/) 中显示的 Azure 资源类型。例如，"存储帐户 "的资源类型可通过[here](https://learn.microsoft.com/en-us/azure/templates/microsoft.storage/storageaccounts?pivots=deployment-language-arm-template) 以及资源对象结构找到。


  ```yaml showLineNumbers
  resources:
    # highlight-start
    - kind: Microsoft.Storage/storageAccounts
    	# highlight-end
    	selector:
    	...
  ```


* 选择器 "字段描述了 Azure 资源选择标准。


  ```yaml showLineNumbers
    resources:
    	- kind: Microsoft.Storage/storageAccounts
    		# highlight-start
    		selector:
    			query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
    			apiVersion: "2023-01-01" # Azure API version to use to fetch the resource
    		# highlight-end
    		port:
  ```


* 查询 "字段是[JQ boolean query](https://stedolan.github.io/jq/manual/#Basicfilters) ，如果评估结果为 "false"，则将跳过该资源。被引用示例 - 跳过同步不在特定区域的资源。


    ```yaml showLineNumbers
    query: .location == "eastus2"
    ```

  - The `apiVersion` field is the Azure API version to use to fetch the resource. This field is required for all resources. For example, the API versions for the `storageAccount` resource was found in the [Storage Account Rest API](https://docs.microsoft.com/en-us/rest/api/storagerp/storageaccounts/list)

    ```yaml showLineNumbers
    apiVersion: "2023-01-01"
    ```

    ![Storage Account API Version](/img/integrations/azure-exporter/StorageAccountAPIVersion.png)

* Port` 字段描述了要从 Azure 资源中创建的Port实体。


  ```yaml showLineNumbers
  resources:
    - kind: Microsoft.Storage/storageAccounts
    	selector:
    		query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
    		apiVersion: "2023-01-01" # Azure API version to use to fetch the resource
    	# highlight-start
    	port:
    		entity:
    			mappings: # Mappings between one Azure object to a Port entity. Each value is a JQ query.
    				identifier: .id
    				title: .name
    				blueprint: '"azureStorageAccount"'
    				properties:
    					location: .location
    					provisioningState: .properties.provisioningState
    	# highlight-end
  ```


* 实体 "字段描述从 Azure 资源创建的 Port 实体。
    - 映射 "字段描述了 Azure 资源与Port实体之间的映射。
        + `identifier` 字段描述 Azure 资源标识符。所有资源都需要此字段。


        ```yaml showLineNumbers
        mappings:
        	# highlight-start
        	identifier: .id
        	# highlight-end
        ```

      - The `title` field describes the Azure resource title. This field is required for all resources.

        ```yaml showLineNumbers
        mappings:
        	# highlight-start
        	title: .name
        	# highlight-end
        ```

      - The `blueprint` field describes the Port blueprint to be used to create the Port entity. This field is required for all resources.


        ```yaml showLineNumbers
        mappings:
        	# highlight-start
        	blueprint: '"azureStorageAccount"'
        	# highlight-end
        ```


```
- The `properties` field describes the Azure resource properties to be mapped to the Port
```


        ```yaml showLineNumbers
        	mappings:
        		identifier: .id
        		title: .name
        		blueprint: '"azureStorageAccount"'
        		# highlight-start
        		properties:
        			location: .location
        			provisioningState: .properties.provisioningState
        		# highlight-end
        ```
