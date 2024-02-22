---
sidebar_position: 4
---

import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 使用 Terraform 在 Azure 云中部署资源

本示例演示了如何使用 Terraform 模板通过 Port Actions 在 Azure 中部署[storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview) 。

工作流程通过 Jenkins 管道执行。

## 先决条件

1. 在 Jenkins 中安装以下插件: 
    1. [Azure Credentials](https://plugins.jenkins.io/azure-credentials/) - 该插件提供 Jenkins Credentials 中的 "Azure Service Principal "类型。
    2. [Terraform](https://plugins.jenkins.io/terraform/) - 此插件提供了一个构建包装器，可简化应用、计划和销毁等 Terraform 命令的执行。
    3. [Generic Webhook Trigger](https://plugins.jenkins.io/generic-webhook-trigger/) - 该插件使 Jenkins 能够根据传入的 HTTP 请求接收和触发作业，从 JSON 或 XML 有效载荷中提取数据并将其作为变量提供。

## 示例 - 创建存储账户

请按照以下步骤开始操作: 

1. 创建以下 Jenkins 证书: 
    1.使用 `Username with password` 类型和 id `port-credentials` 创建Port凭据。
        1. `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
        2. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
    2.使用 `Azure 服务 Principal` 类型和 id `azure` 创建 Azure 凭据。
     提示
     请按照此[guide](https://learn.microsoft.com/en-us/azure/developer/terraform/get-started-cloud-shell-bash?tabs=bash#create-a-service-principal) 创建服务委托以获取 Azure 凭据。
     :::
        1. `ARM_CLIENT_ID` - 应用程序的 Azure 客户 ID(APP ID)。
        2. `ARM_CLIENT_SECRET` - 应用程序的 Azure 客户端secret(密码)。
        3. `ARM_SUBSCRIPTION_ID` - 应用程序的 Azure 订阅 ID。
        4. `ARM_TENANT_ID` - Azure[Tenant ID](https://learn.microsoft.com/en-us/azure/azure-portal/get-subscription-tenant-id) 。
    3. `WEBHOOK_TOKEN` - Webhook 令牌，只有提供该令牌才能触发作业。
2.创建具有以下属性的 Port<PortTooltip id="blueprint">蓝图</PortTooltip>: 

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

3.使用以下 JSON 定义在[self-service hub](https://app.getport.io/self-serve) 中创建 Port 操作: 

<details>

  <summary>Port Action</summary>
   :::note
   Make sure to replace the placeholders for `JENKINS_URL` and `JOB_TOKEN`.
   :::

```json showLineNumbers
{
    "identifier": "create_azure_storage",
    "title": "Create Azure Storage",
    "icon": "Azure",
    "userInputs": {
        "properties": {
            "storage_name": {
                "title": "Storage Name",
                "type": "string",
                "minLength": 3,
                "maxLength": 63
            },
            "storage_location": {
                "icon": "DefaultProperty",
                "title": "Storage Location",
                "description": "storage account geo region",
                "type": "string"
            }
        },
        "required": [
            "storage_name"
        ],
        "order": [
            "storage_name"
        ]
    },
    "invocationMethod": {
        "type": "WEBHOOK",
        "agent": false,
        "url": "https://<JENKINS_HOST>/generic-webhook-trigger/invoke?token=<JOB_TOKEN>",
        "synchronized": false,
        "method": "POST"
    },
    "trigger": "CREATE",
    "requiredApproval": false
}
```

</details>

4.在 GitHub 仓库根目录下的 `terraform` 文件夹中创建以下 Terraform 模板: 
    1. `main.tf` - 该文件将包含资源块，用于定义要在 Azure 云中创建的存储帐户和要在 Port 中创建的实体。
    2. `variables.tf` - 此文件将包含在资源块中被用于的变量声明式，例如 Port 凭据和 Port 运行 ID。
    3. `output.tf` - 该文件将包含 "应用 "操作成功完成后生成的存储帐户的 URL。该 URL 将在创建 Port 实体时被用于到 `endpoint` 属性中。

<details>
  <summary>Terraform `main.tf` template</summary>


  ```yaml showLineNumbers
    # Configure the Azure provider
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

<summary>Terraform `variables.tf` template</summary>
:::note
Replace the default `resource_group_name` with a resource group from your Azure account. Check this [guide](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) to find your resource groups. You may also wish to set the default values of other variables.
:::


```yaml showLineNumbers
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
<summary>Terraform `output.tf` template</summary>


  ```yaml showLineNumbers
    output "endpoint_url" {
        value = azurerm_storage_account.storage_account.primary_web_endpoint
    }
  ```


</details>

5.创建 Jenkins Pipelines: 
    1.[Enable webhook trigger for a pipeline](../jenkins-pipeline.md#enabling-webhook-trigger-for-a-pipeline)
    2.[Define variables for a pipeline](../jenkins-pipeline.md#defining-variables) 定义 STORAGE_NAME、STORAGE_LOCATION、PORT_RUN_ID 和 BLUEPRINT_ID 变量。
    3.[Token Setup](../jenkins-pipeline.md#token-setup) 定义令牌，使其与 Port Action 中配置的 `JOB_TOKEN` 匹配。

<details>

<summary>Jenkins Pipeline Script</summary>
:::note 
Please make sure to modify the `YOUR_USERNAME` and `YOUR_REPO` placeholders in the URL of the git repository in the `Checkout` stage. Alternatively you can use our [example repository](https://github.com/port-labs/pipelines-terraform-azure).

:::

```groovy showLineNumbers
import groovy.json.JsonSlurper

pipeline {
    agent any
    tools {
        "org.jenkinsci.plugins.terraform.TerraformInstallation" "terraform"
    }
    environment {
        TF_HOME = tool('terraform')
        TF_IN_AUTOMATION = "true"
        PATH = "$TF_HOME:$PATH"

        PORT_ACCESS_TOKEN = ""
        endpoint_url = ""

    }

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'STORAGE_NAME', value: '$.payload.properties.storage_name'],
                [key: 'STORAGE_LOCATION', value: '$.payload.properties.storage_location'],
                [key: 'PORT_RUN_ID', value: '$.context.runId'],
                [key: 'BLUEPRINT_ID', value: '$.context.blueprint']
            ],
            causeString: 'Triggered by Port',
            allowSeveralTriggersPerBuild: true,
            tokenCredentialId: "WEBHOOK_TOKEN",

            regexpFilterExpression: '',
            regexpFilterText: '',
            printContributedVariables: true,
            printPostContent: true
        )
    }

    stages {
        stage('Checkout') {
            steps {
                // example repo: git@github.com:port-labs/pipelines-terraform-azure.git
                git branch: 'main', credentialsId: 'github', url: 'git@github.com:<YOUR_USERNAME>/<YOUR_REPO>.git'
            }
        }
        stage('Get access token') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'port-credentials', 
                    usernameVariable: 'PORT_CLIENT_ID', 
                    passwordVariable: 'PORT_CLIENT_SECRET')]) {
                    script {
                        // Execute the curl command and capture the output
                        def result = sh(returnStdout: true, script: """
                            accessTokenPayload=\$(curl -X POST \
                                -H "Content-Type: application/json" \
                                -d '{"clientId": "${PORT_CLIENT_ID}", "clientSecret": "${PORT_CLIENT_SECRET}"}' \
                                -s "https://api.getport.io/v1/auth/access_token")
                            echo \$accessTokenPayload
                        """)

                        // Parse the JSON response using JsonSlurper
                        def jsonSlurper = new JsonSlurper()
                        def payloadJson = jsonSlurper.parseText(result.trim())

                        // Access the desired data from the payload
                        PORT_ACCESS_TOKEN = payloadJson.accessToken
                    }
                }
            }
        }

        stage('Terraform Azure') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: 'azure',
                    subscriptionIdVariable: 'ARM_SUBSCRIPTION_ID',
                    clientIdVariable: 'ARM_CLIENT_ID',
                    clientSecretVariable: 'ARM_CLIENT_SECRET',
                    tenantIdVariable: 'ARM_TENANT_ID'
                ), usernamePassword(credentialsId: 'port-credentials', usernameVariable: 'TF_VAR_port_client_id', passwordVariable: 'TF_VAR_port_client_secret')]) {
                    dir('terraform') {
                        script {
                            echo 'Initializing Terraform'
                            sh 'terraform init'

                            echo 'Validating Terraform configuration'
                            sh 'terraform validate'

                            echo 'Creating Terraform Plan for Azure changes'
                            sh """
                            terraform plan -out=tfazure -var storage_account_name=$STORAGE_NAME -var location=$STORAGE_LOCATION -var port_run_id=$PORT_RUN_ID -target=azurerm_storage_account.storage_account
                            """

                            echo 'Applying Terraform changes to Azure'
                            sh 'terraform apply -auto-approve -input=false tfazure'

                            echo 'Creating Terraform Plan for Port changes'
                            sh """
                            terraform plan -out=tfport -var storage_account_name=$STORAGE_NAME -var location=$STORAGE_LOCATION -var port_run_id=$PORT_RUN_ID
                            """

                            echo 'Applying Terraform changes to Port'
                            sh 'terraform apply -auto-approve -input=false tfport'
                        }
                    }
                }
            }
        }
        stage('Notify Port') {
            steps {
                script {
                    def logs_report_response = sh(script: """
                        curl -X POST \
                            -H "Content-Type: application/json" \
                            -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                            -d '{"message": "Created port entity"}' \
                            "https://api.getport.io/v1/actions/runs/$PORT_RUN_ID/logs"
                    """, returnStdout: true)

                    println(logs_report_response)
                }
            }
        }

        stage('Update Run Status') {
            steps {
                script {
                    def status_report_response = sh(script: """
                        curl -X PATCH \
                          -H "Content-Type: application/json" \
                          -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                          -d '{"status":"SUCCESS", "message": {"run_status": "Jenkins CI/CD Run completed successfully!"}}' \
                             "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}"
                    """, returnStdout: true)

                    println(status_report_response)
                }
            }
        }
    }

    post {

        failure {
            // Update Port Run failed.
            script {
                def status_report_response = sh(script: """
                    curl -X PATCH \
                        -H "Content-Type: application/json" \
                        -H "Authorization: Bearer ${PORT_ACCESS_TOKEN}" \
                        -d '{"status":"FAILURE", "message": {"run_status": "Failed to create azure resource ${STORAGE_NAME}"}}' \
                            "https://api.getport.io/v1/actions/runs/${PORT_RUN_ID}"
                """, returnStdout: true)

                println(status_report_response)
            }
        }

        // Clean after build
        always {
            cleanWs(cleanWhenNotBuilt: false,
                    deleteDirs: true,
                    disableDeferredWipeout: false,
                    notFailBuild: true,
                    patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                               [pattern: '.propsfile', type: 'EXCLUDE']])
        }
    }
}
```

</details>

6.从 Port 应用程序的[self-service](https://app.getport.io/self-serve) 标签触发操作。
