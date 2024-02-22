---

sidebar_position: 1
title: GCP 资产清单
description: 使用 Terraform 被引用的通用资产信息

---

# 全球联络点资产清单

在本示例中，您要将有关 GCP 组织及其所有资产的一些通用信息导出到 Port。

下面是完整的 `main.tf` 文件: 

<details>
<summary>Complete Terraform definition file</summary>

```hcl showLineNumbers
terraform {
  required_providers {
    port = {
      source  = "port-labs/port-labs"
      version = "~> 1"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = ">= 3.53, < 5.0"
    }
  }
}

locals {
  domain = "GCP_ORGANIZATION_DOMAIN_NAME" # set the organization's domain
}

provider "port" {
  client_id = "PORT_CLIENT_ID"     # or set the env var PORT_CLIENT_ID
  secret    = "PORT_CLIENT_SECRET" # or set the env var PORT_CLIENT_SECRET
}

provider "google-beta" {
  credentials = "GOOGLE_APPLICATION_CREDENTIALS" # or set the env var GOOGLE_APPLICATION_CREDENTIALS
}

data "google_organization" "my_org" {
  provider   = google-beta
  domain = local.domain
}

data "google_cloud_asset_resources_search_all" "my_folder_assets" {
  provider = google-beta
  scope    = "organizations/${data.google_organization.my_org.org_id}"
  asset_types = [
    "cloudresourcemanager.googleapis.com/Folder"
  ]
}

data "google_folder" "my_folders" {
  for_each = {for idx, result in data.google_cloud_asset_resources_search_all.my_folder_assets.results : idx => result}
  folder = "folders/${reverse(split("/", each.value.name))[0]}"
}

data "google_projects" "my_projects" {
  provider   = google-beta
  filter = "parent.id:*"
}

data "google_cloud_asset_resources_search_all" "my_assets" {
  for_each = {for idx, result in data.google_projects.my_projects.projects: idx => result}
  provider = google-beta
  scope    = "projects/${each.value.project_id}"
}

resource "port_blueprint" "gcp_org_blueprint" {
  title      = "Organization"
  icon       = "GCP"
  identifier = "gcpOrganization"
  properties = {
    string_props = {
      "link" = {
        type   = "string"
        format = "url"
        title  = "Link"
      }
      "createTime" = {
        type   = "string"
        format = "date-time"
        title  = "Create Time"
      }
    }
  }
}

resource "port_entity" "gcp_org_entity" {
  identifier = data.google_organization.my_org.org_id
  title      = data.google_organization.my_org.domain
  blueprint  = port_blueprint.gcp_org_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?organizationId=${data.google_organization.my_org.org_id}"
      "createTime" = data.google_organization.my_org.create_time
    }
  }
}

resource "port_blueprint" "gcp_folder_blueprint" {
  title      = "Folder"
  icon       = "GCP"
  identifier = "gcpFolder"
  properties = {
    string_props = {
      "link" = {
        type   = "string"
        format = "url"
        title  = "Link"
      }
      "createTime" = {
        type   = "string"
        format = "date-time"
        title  = "Create Time"
      }
    }
  }
  relations = {
    "organization" = {
      title    = "Organization"
      target   = port_blueprint.gcp_org_blueprint.identifier
      required = false
    }
  }
}

resource "port_entity" "gcp_folder_entity" {
  for_each   = { for idx, folder in data.google_folder.my_folders : idx => folder }
  identifier = each.value.folder_id
  title      = each.value.display_name
  blueprint  = port_blueprint.gcp_folder_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?folder=${each.value.folder_id}"
      "createTime" = each.value.create_time
    }
  }
  relations = {
    single_relations = {
      "organization" = data.google_organization.my_org.org_id
    }
  }
}

resource "port_blueprint" "gcp_project_blueprint" {
  title      = "Project"
  icon       = "GCP"
  identifier = "gcpProject"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "createTime" = {
        format = "date-time"
        title  = "Create Time"
      }
      "number" = {
        title = "Number"
      }
    }
    object_props = {
      "labels" = {
        title = "Labels"
      }
    }
  }
  relations = {
    "organization" = {
      target   = port_blueprint.gcp_org_blueprint.identifier
      title    = "Organization"
      required = false
      many     = false
    }
    "folder" = {
      target   = port_blueprint.gcp_folder_blueprint.identifier
      title    = "Folder"
      required = false
      many     = false
    }
  }
}

resource "port_entity" "gcp_project_entity" {
  for_each   = { for idx, project in data.google_projects.my_projects.projects : idx => project }
  identifier = each.value.project_id
  title      = each.value.name
  blueprint  = port_blueprint.gcp_project_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?project=${each.value.project_id}"
      "createTime" = each.value.create_time
      "number" = each.value.number
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "organization" = each.value.parent.type == "organization" ? each.value.parent.id : ""
      "folder"       = each.value.parent.type == "folder" ? each.value.parent.id : ""
    }
  depends_on = [port_entity.gcp_org_entity, port_entity.gcp_folder_entity]
}

resource "port_blueprint" "gcp_asset_blueprint" {
  title      = "GCP Asset"
  icon       = "GCP"
  identifier = "gcpAsset"
  properties = {
    string_props = {
      "name" = {
        title = "name"
      }
      "assetType" = {
        title = "Asset Type"
      }
      "dscription" = {
        title = "Description"
      }
      "location" = {
        title = "Location"
      }
    }
    array_props = {
      string_props = {
        "networkTags" = {
          title = "Network Tags"
        }
        "additionalAttributes" = {
          title = "Additional Attributes"
        }
      }
    }
    object_props = {
      "labels" = {
        title = "Labels"
      }
    }
  }
  relations = {
      "project" = {
        target   = port_blueprint.gcp_project_blueprint.identifier
        title    = "Project"
        required = false
        many     = false
      }
    }
  }
}

resource "port_entity" "gcp_asset_entity" {
  for_each  = { for idx, result in flatten(values({ for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets : idx => [for result in project_assets.results : merge(result, { project_id : project_assets.id })] })) : idx => result }
  title     = each.value.display_name == "" ? reverse(split("/", each.value.name))[0] : each.value.display_name
  blueprint = port_blueprint.gcp_asset_blueprint.identifier
  properties = {
    string_props = {
      "name"        = each.value.name
      "assetType"   = each.value.asset_type
      "description" = each.value.description
      "location"    = each.value.location
    }
    array_props = {
      string_items = {
        "networkTags"          = [for item in each.value.network_tags : item]
        "additionalAttributes" = [for item in each.value.additional_attributes : item]
      }
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "project" = reverse(split("/", each.value.project_id))[0]
    }
  }

  depends_on = [port_entity.gcp_project_entity]
}
```

</details>

要自己使用这个示例，只需替换 `domain`、`client_id`、`secret` 和 `credentials` 的占位符，然后运行以下命令设置新的后端、创建新的基础架构并更新软件目录: 

```shell showLineNumbers
# install modules and create an initial state
terraform init
# To view Terraform's planned changes based on your .tf definition file:
terraform plan
# To apply the changes and update the catalog
terraform apply
```

:::note  GCP 权限 要能读取本例中的组织、文件夹、项目和资产，需要使用至少具有以下权限的组织 GCP IAM 角色: 

```text showLineNumbers
cloudasset.assets.searchAllResources
resourcemanager.folders.get
resourcemanager.folders.list
resourcemanager.organizations.get
resourcemanager.projects.get
```

:::

让我们来分解定义文件，了解其中的各个部分: 

## 模块导入

这部分包括导入和设置所需的 Terraform Provider 和模块: 

```hcl showLineNumbers
terraform {
  required_providers {
    port = {
      source  = "port-labs/port-labs"
      version = "~> 1.0.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = ">= 3.53, < 5.0"
    }
  }
}

locals {
  domain = "GCP_ORGANIZATION_DOMAIN_NAME" # set the organization's domain
}

provider "port" {
  client_id = "PORT_CLIENT_ID"     # or set the env var PORT_CLIENT_ID
  secret    = "PORT_CLIENT_SECRET" # or set the env var PORT_CLIENT_SECRET
}

provider "google-beta" {
  credentials = "GOOGLE_APPLICATION_CREDENTIALS" # or set the env var GOOGLE_APPLICATION_CREDENTIALS
}
```

## 提取组织、文件夹、项目和资产

这部分包括定义组织、文件夹、项目和资产的数据源: 

```hcl showLineNumbers
data "google_organization" "my_org" {
  provider   = google-beta
  domain = local.domain
}

data "google_cloud_asset_resources_search_all" "my_folder_assets" {
  provider = google-beta
  scope    = "organizations/${data.google_organization.my_org.org_id}"
  asset_types = [
    "cloudresourcemanager.googleapis.com/Folder"
  ]
}

data "google_folder" "my_folders" {
  for_each = {for idx, result in data.google_cloud_asset_resources_search_all.my_folder_assets.results : idx => result}
  folder = "folders/${reverse(split("/", each.value.name))[0]}"
}

data "google_projects" "my_projects" {
  provider   = google-beta
  filter = "parent.id:*"
}

data "google_cloud_asset_resources_search_all" "my_assets" {
  for_each = {for idx, result in data.google_projects.my_projects.projects: idx => result}
  provider = google-beta
  scope    = "projects/${each.value.project_id}"
}
```

## 创建组织蓝图和与组织匹配的实体

这部分包括配置 "组织 "蓝图和为组织创建实体: 

```hcl showLineNumbers
resource "port_blueprint" "gcp_org_blueprint" {
  title      = "Organization"
  icon       = "GCP"
  identifier = "gcpOrganization"
  properties = {
    string_props = {
      "link" = {
        type   = "string"
        format = "url"
        title  = "Link"
      }
      "createTime" = {
        type   = "string"
        format = "date-time"
        title  = "Create Time"
      }
    }
  }
}

resource "port_entity" "gcp_org_entity" {
  identifier = data.google_organization.my_org.org_id
  title      = data.google_organization.my_org.domain
  blueprint  = port_blueprint.gcp_org_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?organizationId=${data.google_organization.my_org.org_id}"
      "createTime" = data.google_organization.my_org.create_time
    }
  }
}
```

## 创建文件夹蓝图和与文件夹匹配的实体

这部分包括配置 "文件夹 "蓝图和为文件夹创建实体: 

```hcl showLineNumbers
resource "port_blueprint" "gcp_folder_blueprint" {
  title      = "Folder"
  icon       = "GCP"
  identifier = "gcpFolder"
  properties = {
    string_props = {
      "name" = {
        format = "url"
        title  = "Link"
      }
      "createTime" = {
        format = "date-time"
        title  = "Create Time"
      }
    }
  }
  relations = {
    "organization" = {
      title    = "Organization"
      target   = port_blueprint.gcp_org_blueprint.identifier
      required = false
      many     = false
    }
  }
}

resource "port_entity" "gcp_folder_entity" {
  for_each   = { for idx, folder in data.google_folder.my_folders : idx => folder }
  identifier = each.value.folder_id
  title      = each.value.display_name
  blueprint  = port_blueprint.gcp_folder_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?folder=${each.value.folder_id}"
      "createTime" = each.value.create_time
    }
  }
  relations = {
    single_relations = {
      organization = data.google_organization.my_org.org_id
    }
  }
}
```

## 创建项目蓝图和与项目匹配的实体

这部分包括配置 "项目 "蓝图和为项目创建实体: 

```hcl showLineNumbers
resource "port_blueprint" "gcp_project_blueprint" {
  title      = "Project"
  icon       = "GCP"
  identifier = "gcpProject"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "createTime" = {
        format = "date-time"
        title  = "Create Time"
      }
      "number" = {
        title = "Number"
      }
    }
    object_props = {
      "labels" = {
        title = "Labels"
      }
    }
  }
  relations = {
    "organization" = {
      title    = "Organization"
      target   = port_blueprint.gcp_org_blueprint.identifier
      many     = false
      required = false
    }
    "folder" = {
      title    = "Folder"
      target   = port_blueprint.gcp_folder_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port_entity" "gcp_project_entity" {
  for_each   = { for idx, project in data.google_projects.my_projects.projects : idx => project }
  identifier = each.value.project_id
  title      = each.value.name
  blueprint  = port_blueprint.gcp_project_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?project=${each.value.project_id}"
      "number"     = each.value.number
      "createTime" = each.value.create_time
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "organization" = each.value.parent.type == "organization" ? each.value.parent.id : ""
      "folder"       = each.value.parent.type == "folder" ? each.value.parent.id : ""
    }
  }
  depends_on = [port_entity.gcp_org_entity, port_entity.gcp_folder_entity]
}
```

## 创建 GCP 资产蓝图和与资产匹配的实体

这部分包括配置 `gcpAssets` 蓝图和创建资产实体: 

```hcl showLineNumbers
resource "port_blueprint" "gcp_asset_blueprint" {
  title      = "GCP Asset"
  icon       = "GCP"
  identifier = "gcpAsset"
  properties = {
    string_props = {
      "name" = {
        title = "Name"
      }
      "assetType" = {
        title = "Asset Type"
      }
      "description" = {
        title = "Description"
      }
      "location" = {
        title = "Location"
      }
    }
    object_props = {
      "labels" = {
        title = "Labels"
      }
    }
    array_props = {
      string_items = {
        "additionalAttributes" = {
          title = "Additional Attributes"
        }
      }
      "networkTags" = {
        title = "Network Tags"
      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = port_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port_entity" "gcp_asset_entity" {
  for_each  = { for idx, result in flatten(values({ for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets : idx => [for result in project_assets.results : merge(result, { project_id : project_assets.id })] })) : idx => result }
  title     = each.value.display_name == "" ? reverse(split("/", each.value.name))[0] : each.value.display_name
  blueprint = port_blueprint.gcp_asset_blueprint.identifier
  properties = {
    string_props = {
      "name"        = each.value.name
      "description" = each.value.description
      "assetType"   = each.value.asset_type
      "location"    = each.value.location
    }
    array_props = {
      string_items = {
        "additional_attributes" = each.value.additional_attributes
        "networkTags"           = each.value.network_tags
      }
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "project" = reverse(split("/", each.value.project_id))[0]
    }
  }
  depends_on = [port_entity.gcp_project_entity]
}
```

## 结果

运行 `terraform apply` 后，您将在 Port 中看到组织、文件夹、项目和 `gcpAssets` 实体。