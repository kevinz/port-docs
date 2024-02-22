---

sidebar_position: 3

---

import PortTooltip from "/src/components/tooltip/tooltip.jsx";

# 使用 Terraform 创建 Azure 资源

在以下指南中，您将在 Port 中构建一个自助操作，执行[Azure pipeline](/create-self-service-experiences/setup-backend/azure-pipeline/azure-pipeline.md) ，使用 Terraform 模板在 Azure 中部署[storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview) 。

## 先决条件

* 您需要[Port credentials](/build-your-software-catalog/sync-data-to-catalog/api/api.md#find-your-port-credentials) 来创建操作。
* 您需要您的 Azure DevOps 组织。
* 您需要在 Azure Pipelines yaml 中配置的[webhook](/create-self-service-experiences/setup-backend/azure-pipeline/#define-incoming-webhook-in-azure) 的名称。

## 示例 - 创建存储账户

请按照以下步骤开始操作: 

1. 在 Azure Devops 项目中创建以下变量: 
    1.使用组 id `port-credentials` 创建Port凭证。
        1. `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
        2. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    2.创建 Azure 云凭证，id 为 `azure-service-principal`。
     提示
     请按照此[guide](https://learn.microsoft.com/en-us/azure/developer/terraform/get-started-cloud-shell-bash?tabs=bash#create-a-service-principal) 创建服务 principal，以便获取 Azure 凭据。
     :::
        1. `ARM_CLIENT_ID` - 应用程序的 Azure 客户 ID(APP ID)。
        2. `ARM_CLIENT_SECRET` - 应用程序的 Azure 客户端secret(密码)。
        3. `ARM_SUBSCRIPTION_ID` - 应用程序的 Azure 订阅 ID。
        4. `ARM_TENANT_ID` - Azure[Tenant ID](https://learn.microsoft.com/en-us/azure/azure-portal/get-subscription-tenant-id) 。
2.使用以下 JSON 定义创建 Port<PortTooltip id="blueprint">蓝图</PortTooltip>: 

<details>
   <summary>Port Azure Storage Account Blueprint</summary>
   :::note
   Keep in mind that this can be any blueprint you require; the provided example is just for reference.
   :::

```json showLineNumbers
{
    "identifier": "azureStorage",
    "title": "Azure Storage Account",
    "icon": "Azure",
    "schema": {
        "properties": {
            "storage_name": {
                "title": "Account Name",
                "type": "string",
                "minLength": 3,
                "maxLength": 63,
                "icon": "DefaultProperty"
            },
            "storage_location": {
                "icon": "DefaultProperty",
                "title": "Location",
                "type": "string"
            },
            "url": {
                "title": "URL",
                "format": "url",
                "type": "string",
                "icon": "DefaultProperty"
            }
        },
        "required": [
            "storage_name",
            "storage_location"
        ]
    },
    "mirrorProperties": {},
    "calculationProperties": {},
    "relations": {}
}
```

  </details>

3.在[self-service page](https://app.getport.io/self-serve) 或使用以下 JSON 定义创建 Port 操作: 

<details>

  <summary>Port Action</summary>
   :::tip
- `<AZURE-DEVOPS-ORG>` - your Azure DevOps organization name, can be found in your Azure DevOps URL: `https://dev.azure.com/{AZURE-DEVOPS-ORG}`;
- `<AZURE-DEVOPS-WEBHOOK-NAME>` - the name you gave to the webhook resource in the Azure yaml pipeline file.

:::

```json showLineNumbers
{
    "identifier": "azure_pipelines_create_azure",
    "title": "Azure Pipelines Create Azure",
    "icon": "Azure",
    "userInputs": {
    "properties": {
        "storage_name": {
            "icon": "Azure",
            "title": "Storage Name",
            "description": "The Azure Storage Account",
            "type": "string"
        },
        "storage_location": {
            "title": "Storage Location",
            "icon": "Azure",
            "type": "string",
            "default": "westus2"
        }
    },
    "required": [
        "storage_name"
    ],
    "order": [
        "storage_name",
        "storage_location"
    ]
    },
    "invocationMethod": {
        "type": "AZURE-DEVOPS",
        "webhook": "<AZURE-DEVOPS-WEBHOOK-NAME>",
        "org": "<AZURE-DEVOPS-ORG>"
    },
    "trigger": "CREATE",
    "description": "Use azure pipelines to terraform an azure resource ",
    "requiredApproval": false,
}
```

</details>

4.在 GitHub 仓库根目录下的 `terraform` 文件夹中创建以下 Terraform 模板: 
 :::提示
 分叉我们的[example repository](https://github.com/port-labs/pipelines-terraform-azure) 以开始使用。
 :::
    1. `main.tf` - 该文件将包含定义要在 Azure 云中创建的存储帐户和要在 Port 中创建的实体的资源块。
    2. `variables.tf` - 此文件将包含在资源块中被引用的变量声明式，例如 Port 凭据和 Port 运行 ID。
    3. `output.tf` - 该文件将包含 "应用 "操作成功完成后生成的存储帐户的 URL。该 URL 将在创建 Port 实体时被引用到 `endpoint` 属性中。

<details>
  <summary><b>main.tf</b></summary>

```hcl showLineNumbers title="main.tf"
terraform {
    required_providers {
        azurerm = {
            source  = "hashicorp/azurerm"
            version = "~> 3.0.2"
        }
        port = {
            source  = "port-labs/port-labs"
            version = "~> 1.0.0"
        }
    }

    required_version = ">= 1.1.0"
}

provider "azurerm" {

    features {}
}

provider "port" {
    client_id = var.port_client_id
    secret    = var.port_client_secret
}

resource "azurerm_storage_account" "storage_account" {
    name                = var.storage_account_name
    resource_group_name = var.resource_group_name

    location                 = var.location
    account_tier             = "Standard"
    account_replication_type = "LRS"
    account_kind             = "StorageV2"
}

resource "port_entity" "azure_storage_account" {
    count      = length(azurerm_storage_account.storage_account) > 0 ? 1 : 0
    identifier = var.storage_account_name
    title      = var.storage_account_name
    blueprint  = "azureStorage"
    run_id     = var.port_run_id
    properties = {
        string_props = {
        "storage_name"     = var.storage_account_name,
        "storage_location" = var.location,
        "endpoint"         = azurerm_storage_account.storage_account.primary_web_endpoint
        }
    }

    depends_on = [azurerm_storage_account.storage_account]
}
```

</details>

<details>

  <summary><b>variables.tf</b></summary>
  :::note
  Replace the default `resource_group_name` with a resource group from your Azure account. Check this [guide](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) to find your resource groups. You may also wish to set the default values of other variables.
  :::

```hcl showLineNumbers title="variables.tf"
variable "resource_group_name" {
    type        = string
    default     = "myTFResourceGroup"
    description = "RG name in Azure"
}

variable "location" {
    type        = string
    default     = "westus2"
    description = "RG location in Azure"
}

variable "storage_account_name" {
    type        = string
    description = "Storage Account name in Azure"
    default     = "demo"
}

variable "port_run_id" {
    type        = string
    description = "The runID of the action run that created the entity"
}

variable "port_client_id" {
    type        = string
    description = "The Port client ID"
}

variable "port_client_secret" {
    type        = string
    description = "The Port client secret"
}
```

</details>

<details>
<summary><b>output.tf</b></summary>

```hcl showLineNumbers title="output.tf"
output "endpoint_url" {
    value = azurerm_storage_account.storage_account.primary_web_endpoint
}
```

</details>

5.创建 Azure Pipelines: 

<details>

<summary>Azure Devops pipelines script</summary>

```yaml showLineNumbers title="azure-pipelines.yml"
trigger: none

pool:
  vmImage: "ubuntu-latest"

resources:
  webhooks:
    - webhook: PortWebhook
      connection: PortWebhook

variables:
  - group: port-credentials
  - group: azure-service-principal
  - name: STORAGE_NAME
    value: ${{ parameters.PortWebhook.payload.properties.storage_name }}
  - name: STORAGE_LOCATION
    value: ${{ parameters.PortWebhook.payload.properties.storage_location }}
  - name: PORT_RUN_ID
    value: ${{ parameters.PortWebhook.context.runId }}

jobs:
- job: DeployJob
  displayName: 'Deploy to Azure and Port'
  steps:
  - checkout: self
    displayName: 'Checkout repository'

  - bash: |
      startedAt=$(date -u +%Y-%m-%dT%H:%M:%S.000Z)
      echo "##vso[task.setvariable variable=startedAt]$startedAt"
      echo "Started at $startedAt"
    displayName: 'Set Start Time'

  - script: |
      sudo apt-get update
      sudo apt-get install -y jq
    displayName: Install jq

  - script: |
      accessToken=$(curl -X POST \
            -H 'Content-Type: application/json' \
            -d '{"clientId": "$(PORT_CLIENT_ID)", "clientSecret": "$(PORT_CLIENT_SECRET)"}' \
            -s 'https://api.getport.io/v1/auth/access_token' | jq -r '.accessToken')
      echo "##vso[task.setvariable variable=accessToken;isOutput=true]$accessToken"
    displayName: 'Fetch Access Token and Run ID'
    name: getToken

  - bash: |      
      terraform init -input=false
    displayName: 'Initialize configuration'
    failOnStderr: true
    workingDirectory: 'terraform'

  - script: |
      terraform validate
    displayName: 'Terraform Validate'
    workingDirectory: 'terraform'

  - script: |
      tf_plan_and_apply() {
          local plan_type=$1
          local target_option=""

          if [ "$plan_type" == "azure" ]; then
            target_option="-target=azurerm_storage_account.storage_account"
          fi

          terraform plan \
            -input=false \
            -out=tf${plan_type}-${BUILD_BUILDNUMBER}.tfplan \
            -var="storage_account_name=${STORAGE_NAME}" \
            -var="location=${STORAGE_LOCATION}" \
            $target_option

          terraform apply -auto-approve -input=false tf${plan_type}-${BUILD_BUILDNUMBER}.tfplan
      }

      tf_plan_and_apply azure
      tf_plan_and_apply port
    displayName: 'Terraform changes to Azure and Port'
    workingDirectory: 'terraform'
    env:
      TF_VAR_resource_group_name: arete-resources
      TF_VAR_port_client_id: $(PORT_CLIENT_ID)
      TF_VAR_port_client_secret: $(PORT_CLIENT_SECRET)
      TF_VAR_port_run_id: $(PORT_RUN_ID)

  - script: |
      completedAt=$(date -u +%Y-%m-%dT%H:%M:%S.000Z)
      terraform_output=$(terraform output endpoint_url | sed 's/"//g')
      echo ${terraform_output}

      curl -X PATCH \
        -H 'Content-Type: application/json' \
        -H 'Authorization: Bearer $(getToken.accessToken)' \
        -d '{
            "status": "SUCCESS",
            "message": {"run_status":"Completed resource creation at $(completedAt)", "url":"$(terraform_output)" }
          }' \
        "https://api.getport.io/v1/actions/runs/$(PORT_RUN_ID)"
    displayName: 'Update Run Status'
    workingDirectory: 'terraform'
```

</details>

6.从 Port 应用程序的[self-service](https://app.getport.io/self-serve) 页面触发操作。
