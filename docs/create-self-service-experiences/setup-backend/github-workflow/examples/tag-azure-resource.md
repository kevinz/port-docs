---

sidebar_position: 3

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortTooltip from "/src/components/tooltip/tooltip.jsx";

# 为 Azure 资源添加标签

在以下指南中，您将在 Port 中创建一个自助操作，执行[GitHub workflow](/create-self-service-experiences/setup-backend/github-workflow/github-workflow.md) ，为[storage account](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview) 添加标记。

## 先决条件

1. 本指南假定你已经拥有一个 Azure 存储账户的蓝图和一些资源。如果尚未创建蓝图，请首先参考本[guide](/create-self-service-experiences/setup-backend/github-workflow/examples/create-azure-resource.md) 。
2. 事先了解 Port Actions 对于学习本指南至关重要。了解有关它们的更多信息[here](/create-self-service-experiences/setup-ui-for-action/) 。
3. 一个 GitHub 仓库，其中包含您的操作资源，即 github 工作流文件。

## 示例--为存储账户添加标签

请按照以下步骤开始操作: 

1. 创建以下 GitHub Action secrets: 
    1.创建以下 Port 凭据: 
        1. `PORT_CLIENT_ID` - Port客户端 ID[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 。
        2. `PORT_CLIENT_SECRET` - Port客户端secret[learn more](/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token).
    2.创建以下 Azure 云凭证: 
     提示
     按照此[guide](https://learn.microsoft.com/en-us/azure/developer/terraform/get-started-cloud-shell-bash?tabs=bash#create-a-service-principal) 创建服务 principal，以便获取 Azure 凭据。
     :::
        1. `ARM_CLIENT_ID` - 应用程序的 Azure 客户 ID(APP ID)。
        2. `ARM_CLIENT_SECRET` - 应用程序的 Azure 客户端secret(密码)。
        3. `ARM_SUBSCRIPTION_ID` - 应用程序的 Azure 订阅 ID。
        4. `ARM_TENANT_ID` - Azure[Tenant ID](https://learn.microsoft.com/en-us/azure/azure-portal/get-subscription-tenant-id) 。

<br />
2. Install Port's GitHub app by clicking [here](https://github.com/apps/getport-io/installations/new).
<br />

3.在[self-service page](https://app.getport.io/self-serve) 或使用以下 JSON 定义创建 Port 操作: 

<details>

  <summary>Port Action: Add Tags to Azure Storage</summary>
   :::tip
- `<GITHUB-ORG>` - your GitHub organization or user name.
- `<GITHUB-REPO-NAME>` - your GitHub repository name.

:::

```json showLineNumbers
{
  "identifier": "add_tags_to_azure_storage",
  "title": "Add Tags to Azure Storage",
  "icon": "Azure",
  "userInputs": {
    "properties": {
      "tags": {
        "title": "Tags",
        "type": "object"
      }
    },
    "required": [
      "tags"
    ],
    "order": []
  },
  "invocationMethod": {
   "type": "GITHUB",
    "org": "<GITHUB-ORG>",
    "repo": "<GITHUB-REPO-NAME>",
    "workflow": "tag-azure-resource.yml",
    "omitUserInputs": false,
    "omitPayload": false,
    "reportWorkflowStatus": true
  },
  "trigger": "DAY-2",
  "description": "Add tags to azure storage acount",
  "requiredApproval": false
}
```

</details>
<br />

<Tabs groupId="cicd-method" queryString="cicd-method">

<TabItem value="terraform" label="Terraform">
4. Update the following Terraform templates in the `terraform` folder at the root of your GitHub repository:
    :::tip
    Fork our [example repository](https://github.com/port-labs/pipelines-terraform-azure) to get started.
    :::

```
1. `main.tf` - Include a tags field within the configuration of the storage account resource.
2. `variables.tf` – Introduce a new variable named `resource_tags`.
```

<details>
  <summary><b>main.tf</b></summary>

```hcl showLineNumbers title="main.tf"
...

resource "azurerm_storage_account" "storage_account" {
    name                = var.storage_account_name
    resource_group_name = var.resource_group_name

    location                 = var.location
    account_tier             = "Standard"
    account_replication_type = "LRS"
    account_kind             = "StorageV2"
    // highlight-start
    tags                     = var.resource_tags
    // highlight-end
}

...
```

</details>

<details>

  <summary><b>variables.tf</b></summary>

```hcl showLineNumbers title="variables.tf"
// ...
variable "resource_tags" {
  type = map(string)
  default = {
    Environment = "Production"
  }
}
// ...
```

</details>

<br />
5. Create a workflow file under `.github/workflows/tag-azure-resource.yml` with the following content:

<details>

<summary>GitHub workflow script</summary>
  :::note
  Replace the following variables for the `terraform init` step: 
  1. `RESOURCE_GROUP_NAME` with a resource group from your Azure account. Check this [guide](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) to find your resource groups. 
  2. `STORAGE_ACCOUNT_NAME`: The storage account containing.
  3. `TF_STATE_CONTAINER`: The name of the container used for storing the Terraform state files.
  4. `TF_STATE_KEY`: Indicate the key that uniquely identifies the configuration file.
  :::

```yaml showLineNumbers title="tag-azure-resource.yml"
name: "Tag Azure Resource"

on: 
  workflow_dispatch:
    inputs:
      tags:
        required: true
        type: string
      port_payload:
        required: true
        description:
            Port's payload, including details for who triggered the action and
            general context (blueprint, run id, etc...)
        type: string

env: 
  TF_LOG: INFO
  TF_INPUT: false

jobs:
  terraform:
    name: "Add Tags to Azure Resource"
    runs-on: ubuntu-latest
    defaults:
      run:
        shell: bash
        # We keep Terraform files in the terraform directory.
        working-directory: ./terraform
        # working-directory: ./

    steps:
      - name: Checkout the repository to the runner
        uses: actions/checkout@v2

      - name: Setup Terraform with specified version on the runner
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.6.0

      - name: Terraform init
        id: init
        # run: terraform init 
        env:
          ARM_CLIENT_ID: ${{ secrets.ARM_CLIENT_ID }}
          ARM_CLIENT_SECRET: ${{ secrets.ARM_CLIENT_SECRET }}
          ARM_TENANT_ID: ${{ secrets.ARM_TENANT_ID }}
          ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
          // highlight-start
          RESOURCE_GROUP_NAME: YourResourceGroup
          STORAGE_ACCOUNT_NAME: YourStorageAccount
          TF_STATE_CONTAINER: tfstate
          TF_STATE_KEY: terraform.tfstate
          // highlight-end
        run: |
          terraform init \
            -backend-config="resource_group_name=$RESOURCE_GROUP_NAME" \
            -backend-config="storage_account_name=$STORAGE_ACCOUNT_NAME" \
            -backend-config="container_name=$TF_STATE_CONTAINER" \
            -backend-config="key=$TF_STATE_KEY" \
            -input=false

      - name: Terraform format
        id: fmt
        run: terraform fmt -check

      - name: Terraform validate
        id: validate
        run: terraform validate

      - name: Run Terraform Plan and Apply (Azure)
        id: plan-azure
        env: 
            ARM_CLIENT_ID: ${{ secrets.ARM_CLIENT_ID }}
            ARM_CLIENT_SECRET: ${{ secrets.ARM_CLIENT_SECRET }}
            ARM_TENANT_ID: ${{ secrets.ARM_TENANT_ID }}
            ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
            TF_VAR_port_client_id: ${{ secrets.PORT_CLIENT_ID }}
            TF_VAR_port_client_secret: ${{ secrets.PORT_CLIENT_SECRET }}
            TF_VAR_port_run_id: ${{fromJson(inputs.port_payload).context.runId}}
            TF_VAR_storage_account_name: ${{fromJson(inputs.port_payload).context.entity}}
            TF_VAR_resource_tags: ${{ github.event.inputs.tags }}
        run: |
          terraform plan \
            -input=false \
            -out=tfazure-${GITHUB_RUN_NUMBER}.tfplan

          terraform apply -auto-approve -input=false tfazure-${GITHUB_RUN_NUMBER}.tfplan

      - name: Terraform Azure Status
        if: steps.plan-azure.outcome == 'failure'
        run: exit 1

      - name: Create a log message
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(inputs.port_payload).context.runId}}
          logMessage: Added tags to ${{fromJson(inputs.port_payload).context.entity}}
```

</details>
</TabItem>

<TabItem value="azurecli" label="Azure CLI">

4.创建一个 GitHub Action secret `AZURE_CREDENTIALS`，其值如下: (请参阅[Using secrets in GitHub Actions](https://github.com/Azure/login?tab=readme-ov-file#login-with-a-service-principal-secret:~:text=below%3A%20(Refer%20to-,Using%20secrets%20in%20GitHub%20Actions,-.)%3C/keepr%3E)%3C/keepr%3E.)

```json
AZURE_CREDENTIALS = {
  "clientSecret":  "******",
  "subscriptionId":  "******",
  "tenantId":  "******",
  "clientId":  "******"
}
```

5.在`.github/workflows/tag-azure-resource.yml`下创建一个工作流程文件，内容如下: 

<details>

<summary>GitHub workflow script</summary>
  :::note
  Replace the `RESOURCE_GROUP_NAME` with a resource group from your Azure account. Check this [guide](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) to find your resource groups. 
  :::

```yaml showLineNumbers title="tag-azure-resource.yml"
name: "Tag Azure Resource CLI"

on: 
  workflow_dispatch:
    inputs:
      tags:
        required: true
        type: string
      port_payload:
        required: true
        description:
            Port's payload, including details for who triggered the action and
            general context (blueprint, run id, etc...)
        type: string

jobs:
    build-and-deploy:
      runs-on: ubuntu-latest
      steps:

      - name: Install jq
        run: sudo apt-get install jq -y

      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Azure CLI script
        uses: azure/CLI@v1
        env: 
        // highlight-start
          RESOURCE_GROUP: YourResourceGroup
        // highlight-end
          STORAGE_NAME: ${{ fromJson(inputs.port_payload).context.entity }}
          TAGS: ${{ github.event.inputs.tags }}
        with:
          azcliversion: latest
          inlineScript: |
            az account show
            resource=$(az resource show -g ${RESOURCE_GROUP} -n ${STORAGE_NAME} --resource-type Microsoft.Storage/storageAccounts --query "id" --output tsv)
            tags=$(echo ${TAGS} | jq -r 'to_entries|map("\(.key)=\(.value|tojson)")|join(" ")')
            az tag create --resource-id $resource --tags $tags
```

</details>

</TabItem>

</Tabs>
<br />
6. Trigger the action from the [self-service](https://app.getport.io/self-serve) page of your Port application.