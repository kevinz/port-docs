---
sidebar_position: 2
title: 特定资产的资源元数据

description: 使用 terraform 为资产引用特定资源元数据

---

# 特定资产的资源元数据

在本例中，您将学习如何导出 GCP 组织的特定资产，以及与之匹配的资产类型的扩展元数据。

下面是完整的 `main.tf` 文件: 

<details>
<summary>Complete Terraform definition file</summary>

```hcl showLineNumbers
terraform {
  required_providers {
    port-labs = {
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
  asset_types = [
    "storage.googleapis.com/Bucket",
    "iam.googleapis.com/ServiceAccount",
    "compute.googleapis.com/Disk",
    "redis.googleapis.com/Instance",
    "compute.googleapis.com/Instance",
    "run.googleapis.com/Service",
    "container.googleapis.com/Cluster"
  ]
}

data "google_storage_bucket" "my_buckets" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => project_assets.results})) : idx => result if result.asset_type == "storage.googleapis.com/Bucket"}
  name     = reverse(split("/", each.value.name))[0]
}

data "google_service_account" "my_accounts" {
  provider   = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "iam.googleapis.com/ServiceAccount"}
  account_id = reverse(split("/", each.value.name))[0]
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_compute_disk" "my_disks" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "compute.googleapis.com/Disk"}
  zone = each.value.location
  name    = reverse(split("/", each.value.name))[0]
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_redis_instance" "my_memorystores" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "redis.googleapis.com/Instance"}
  name     = reverse(split("/", each.value.name))[0]
  region   = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_compute_instance" "my_compute_instances" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "compute.googleapis.com/Instance"}
  name     = reverse(split("/", each.value.name))[0]
  zone     = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_cloud_run_service" "my_run_services" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "run.googleapis.com/Service"}
  name     = reverse(split("/", each.value.name))[0]
  location = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_container_cluster" "my_container_clusters" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "container.googleapis.com/Cluster"}
  name     = reverse(split("/", each.value.name))[0]
  location = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

resource "port-labs_blueprint" "gcp_org_blueprint" {
  title      = "Organization"
  icon       = "GCP"
  identifier = "organization"
  properties = {
    string_props = {
        "link" = {
            title      = "Link"
            required   = false
            format     = "url"
        }
        "createTime" = {
            type       = "string"
            format     = "date-time"
            title      = "Create Time"
        }
    }
  }
}

resource "port-labs_entity" "gcp_org_entity" {
  identifier = data.google_organization.my_org.org_id
  title     = data.google_organization.my_org.domain
  blueprint = port-labs_blueprint.gcp_org_blueprint.identifier
  properties = {
    string_props = {
    "link" =  "https://console.cloud.google.com/welcome?organizationId=${data.google_organization.my_org.org_id}"
    "createTime" = data.google_organization.my_org.create_time
    }
}
}

resource "port_blueprint" "gcp_folder_blueprint" {
  title      = "Folder"
  icon       = "GCP"
  identifier = "folder"
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
    }
  }
  relations = {
    "organization" = {
      target   = port_blueprint.gcp_org_blueprint.identifier
      title    = "Organization"
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_folder_entity" {
  for_each   = { for idx, folder in data.google_folder.my_folders : idx => folder }
  identifier = each.value.folder_id
  title      = each.value.display_name
  blueprint  = port-labs_blueprint.gcp_folder_blueprint.identifier
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

resource "port-labs_blueprint" "gcp_project_blueprint" {
  title      = "Project"
  icon       = "GCP"
  identifier = "project"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "number" = {
        title = "Number"
      }
      "createTime" = {
        format = "date-time"
        title  = "Create Time"
      }
    }
    object_props = {
      "labels" = {
        type  = "object"
        title = "Labels"
      }
    }
  } 
  relations = {
    organization = {
      target   = port-labs_blueprint.gcp_org_blueprint.identifier
      title    = "Organization"
      many     = false
      required = false
    }
    folder = {
      target   = port-labs_blueprint.gcp_folder_blueprint.identifier
      title    = "Folder"
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_project_entity" {
  depends_on = [port-labs_entity.gcp_org_entity, port-labs_entity.gcp_folder_entity]
  for_each   = { for idx, project in data.google_projects.my_projects.projects : idx => project }
  identifier = each.value.project_id
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_project_blueprint.identifier
  properties = {
    string_props = {
      "link"       = "https://console.cloud.google.com/welcome?project=${each.value.project_id}"
      "number"     = each.value.number
      "createTime" = each.value.create_time
    }
    object_props = {
      "labels"     = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "organization" = each.value.parent.type == "organization" ? each.value.parent.id : ""
      "folder"       = each.value.parent.type == "folder" ? each.value.parent.id : ""
    }
  }
}

resource "port-labs_blueprint" "gcp_bucket_blueprint" {
  title      = "Storage Bucket"
  icon       = "Bucket"
  identifier = "storageBucket"
  properties = {
    boolean_props = {
      "uniformBucketLevelAccess" = {
        title    = "Uniform Bucket Level Access"
        required = false
      }
    }
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "location" = {
        title = "Location"
      }
    }
    array_props = {
      "lifecycleRule" = {
        title        = "Lifecycle Rule"
        object_items = {}
      }
      "publicAccessPrevention" = {
        title = "Public Access Prevention"
        enum  = ["inherited", "enforced"]
      }
      "storageClass" = {
        enum  = ["STANDARD", "MULTI_REGIONAL", "REGIONAL", "NEARLINE", "COLDLINE", "ARCHIVE"]
        title = "Storage Class"
      }
      "encryption" = {
        object_items = {}
        title        = "Encryption"
      }
    }
    object_props = {
      "labels" = {
        type  = "object"
        title = "Labels"
      }
    }
  }
  relations = {
    "project" = {
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      title    = "GCP Project"
      required = false
      many     = false
    }
  }
}

resource "port-labs_entity" "gcp_bucket_entity" {
  depends_on = [port-labs_entity.gcp_project_entity]
  for_each   = data.google_storage_bucket.my_buckets
  identifier = each.value.id
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_bucket_blueprint.identifier
  properties = {
    object_props = {
      labels = jsonencode(each.value.labels)
    }
    boolean_props = {
      "uniformBucketLevelAccess" = each.value.uniform_bucket_level_access
    }
    string_props = {
      "link"     = "https://console.cloud.google.com/storage/browser/${each.value.id};tab=objects?project=${each.value.project}"
      "location" = each.value.location
    }
    array_props = {
      string_items = {
        publicAccessPrevention = each.value.public_access_prevention
        storageClass           = each.value.storage_class
      }
      object_items = {
        lifecycleRule = [for item in each.value.lifecycle_rule : jsonencode(item)]
        encryption    = [for item in each.value.encryption : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
}

resource "port-labs_blueprint" "gcp_service_account_blueprint" {
  title      = "Service Account"
  icon       = "Lock"
  identifier = "serviceAccount"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "email" = {
        format = "email"
        title  = "email"
      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = "port-labs_blueprint.gcp_project_blueprint"
      required = false
      many = false
    }
  }
}

resource "port-labs_entity" "gcp_service_account_entity" {
  for_each   = data.google_service_account.my_accounts
  identifier = each.value.account_id
  title      = each.value.display_name
  blueprint  = port-labs_blueprint.gcp_service_account_blueprint.identifier
  properties = {
    string_props = {
      "link"  = "https://console.cloud.google.com/iam-admin/serviceaccounts/details/${each.value.unique_id}?project=${each.value.project}"
      "email" = each.value.email
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}

resource "port-labs_blueprint" "gcp_disk_blueprint" {
  title      = "Disk"
  icon       = "GoogleComputeEngine"
  identifier = "disk"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "description" = {
        title = "Description"
      }
      "zone" = {
        title = "Zone"
      }
      "type" = {
        title = "Type"
      }
      "creationTimestamp" = {
        title = "Creation Timestamp"
      }
    }
    object_props = {
      "labels" = {
        title = "Labels"
      }
    }
    number_props = {
      "size" = {
        title = "Size"
      }
    }
    array_props = {
      string_items = {
        "users" = {
          title = "Users"
        }
      }
    }
  }
  relations = {
    "project" = {
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      title    = "Project"
      required = false
      many     = false
    }
  }
}

resource "port-labs_entity" "gcp_disk_entity" {
  for_each   = data.google_compute_disk.my_disks
  identifier = "${each.value.project}_${each.value.zone}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_disk_blueprint.identifier
  properties = {
    string_props = {
      "link"              = "https://console.cloud.google.com/compute/disksDetail/zones/${each.value.zone}/disks/${each.value.name}?project=${each.value.project}"
      "description"       = each.value.description
      "zone"              = each.value.zone
      "type"              = each.value.type
      "creationTimestamp" = each.value.creation_timestamp
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
    number_props = {
      "size" = each.value.size
    }
    array_props = {
      string_items = {
        "users" = each.value.users
      }
    }
    relations = {
      single_relations = {
        "project" = each.value.project
      }
    }
    depends_on = [port-labs_entity.gcp_project_entity]
  }
}

resource "port-labs_blueprint" "gcp_memorystore_blueprint" {
  title      = "Memorystore"
  icon       = "GoogleCloudPlatform"
  identifier = "memorystore"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "region" = {
        title = "Region"
      }
      "currentLocationId" = {
        title = "Current Location ID"
      }
      "redisVersion" = {
        title = "Redis Version"
      }
      "tier" = {
        enum  = ["BASIC", "STANDARD_HA"]
        title = "Public Access Prevention"
      }
      "readReplicasMode" = {
        enum  = ["READ_REPLICAS_DISABLED", "READ_REPLICAS_ENABLED"]
        title = "Read Replicas Mode"
      }
      "connectMode" = {
        enum  = ["DIRECT_PEERING", "PRIVATE_SERVICE_ACCESS"]
        title = "Connect Mode"
      }
      "createTime" = {
        format = "date-time"
        title  = "Create Time"
      }
      "authorizedNetwork" = {
        title = "Authorized Network"
      }
    }
    number_props = {
      "memorySizeGb" = {
        title = "Memory Size GB"
      }
      "replicaCount" = {
        title = "Replica Count"
      }
    }
    array_props = {
      "nodes" = {
        title        = "Nodes"
        object_items = {}
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
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_memorystore_entity" {
  for_each   = data.google_redis_instance.my_memorystores
  identifier = "${each.value.project}_${each.value.location_id}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_memorystore_blueprint.identifier
  properties = {
    string_props = {
      "link"              = "https://console.cloud.google.com/memorystore/redis/locations/${each.value.region}/instances/${each.value.name}/details/overview?project=${each.value.project}"
      "region"            = each.value.region
      "currentLocationId" = each.value.current_location_id
      "redisVersion"      = each.value.redis_version
      "tier"              = each.value.tier
      "readReplicasMode"  = each.value.read_replicas_mode
      "connectMode"       = each.value.connect_mode
      "createTime"        = each.value.create_time
      "authorizedNetwork" = each.value.authorized_network
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
    array_props = {
      object_items = {
        "nodes" = [for item in each.value.nodes : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}

resource "port-labs_blueprint" "gcp_compute_instance_blueprint" {
  title      = "Compute Instance"
  icon       = "GoogleComputeEngine"
  identifier = "computeInstance"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "zone" = {
        title = "Zone"
      }
      "currentStatus" = {
        enum = ["PROVISIONING", "STAGING", "RUNNING", "STOPPING", "REPAIRING", "TERMINATED", "SUSPENDING", "SUSPENDED"]
        enum_colors = {
          PROVISIONING = "blue"
          STAGING      = "blue"
          RUNNING      = "green"
          STOPPING     = "red"
          REPAIRING    = "yellow"
          TERMINATED   = "red"
          SUSPENDING   = "yellow"
          SUSPENDED    = "lightGray"
        }
        title = "Current Status"
      }
      "machineType" = {
        title = "Machine Type"
      }
      "cpuPlatform" = {
        title = "CPU Platform"
      }
    }
    boolean_props = {
      "deletionProtection" = {
        title = "Deletion Protection"
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
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_compute_instance_entity" {
  for_each   = data.google_compute_instance.my_compute_instances
  identifier = "${each.value.project}_${each.value.zone}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_compute_instance_blueprint.identifier
  properties = {
    string_props = {
      "link"          = "https://console.cloud.google.com/compute/instancesDetail/zones/${each.value.zone}/instances/${each.value.name}?project=${each.value.project}"
      "zone"          = each.value.zone
      "currentStatus" = each.value.current_status
      "machineType"   = each.value.machine_type
      "cpuPlatform"   = each.value.cpu_platform
    }
    boolean_props = {
      "deletionProtection" = each.value.deletion_protection
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}

resource "port-labs_blueprint" "gcp_run_service_blueprint" {
  title      = "Run Service"
  icon       = "Service"
  identifier = "runService"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "location" = {
        title = "Location"
      }
    }
    array_props = {
      "metadata" = {
        title        = "Metadata"
        object_items = {}
      }
      "status" = {
        title        = "Status"
        object_items = {}
      }
      "template" = {
        title        = "Template"
        object_items = {}
      }
      "traffic" = {
        title        = "Traffic"
        object_items = {}

      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      required = false
      many     = false
    }
  }
}

resource "port-labs_entity" "gcp_run_service_entity" {
  for_each   = data.google_cloud_run_service.my_run_services
  identifier = "${each.value.project}_${each.value.location}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_run_service_blueprint.identifier
  properties = {
    string_props = {
      "link"     = "https://console.cloud.google.com/run/detail/${each.value.location}/${each.value.name}?project=${each.value.project}"
      "location" = each.value.location
    }
    array_props = {
      object_items = {
        "metadata" = [for item in each.value.metadata : jsonencode(item)]
        "status"   = [for item in each.value.status : jsonencode(item)]
        "template" = [for item in each.value.template : jsonencode(item)]
        "traffic"  = [for item in each.value.traffic : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}

resource "port-labs_blueprint" "gcp_container_cluster_blueprint" {
  title      = "Container Cluster"
  icon       = "Cluster"
  identifier = "containerCluster"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "location" = {
        title = "Location"
      }
      "description" = {
        title = "Description"
      }
      "masterVersion" = {
        title = "Master Version"
      }
      "network" = {
        title = "Network"
      }
    }
    boolean_props = {
      "enableAutopilot" = {
        title = "Enable Autopilot"
      }
    }
    array_props = {
      "addonsConfig" = {
        title        = "Addons Config"
        object_items = {}
      }
      "clusterAutoscaling" = {
        title        = "Cluster Autoscaling"
        object_items = {}
      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_container_cluster_entity" {
  for_each   = data.google_container_cluster.my_container_clusters
  identifier = "${each.value.project}_${each.value.location}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_container_cluster_blueprint.identifier
  properties = {
    string_props = {
      "link"           = "https://console.cloud.google.com/kubernetes/clusters/details/${each.value.location}/${each.value.name}?project=${each.value.project}"
      "location"       = each.value.location
      "description"    = each.value.description
      "master_version" = each.value.master_version
      "network"        = each.value.network
    }
    booleen_props = {
      "enableAutopilot" = each.value.enable_autopilot
    }
    array_props = {
      object_items = {
        "addonsConfig"       = [for item in each.value.addons_config : jsonencode(item)]
        "clusterAutoscaling" = [for item in each.value.cluster_autoscaling : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
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

:::note  GCP 权限 要读取本示例中的所有类型资产，需要使用至少具有以下权限的组织 GCP IAM 角色: 

```text showLineNumbers
cloudasset.assets.searchAllResources
compute.disks.get
compute.instanceGroupManagers.get
compute.instances.get
compute.projects.get
container.clusters.get
iam.serviceAccounts.get
redis.instances.get
resourcemanager.folders.get
resourcemanager.folders.list
resourcemanager.organizations.get
resourcemanager.projects.get
run.services.get
storage.buckets.get
```

:::

让我们来分解定义文件，了解其中的各个部分: 

## 模块导入

这部分包括导入和设置所需的 Terraform Provider 和模块: 

```hcl showLineNumbers
terraform {
  required_providers {
    port-labs = {
      source  = "port-labs/port-labs"
      version = "~> 0.10.3"
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

provider "port-labs" {
  client_id = "PORT_CLIENT_ID"     # or set the env var PORT_CLIENT_ID
  secret    = "PORT_CLIENT_SECRET" # or set the env var PORT_CLIENT_SECRET
}

provider "google-beta" {
  credentials = "GOOGLE_APPLICATION_CREDENTIALS" # or set the env var GOOGLE_APPLICATION_CREDENTIALS
}
```

## 提取组织、文件夹、项目和特定资产

这部分包括为组织、文件夹、项目和特定资产("buckets"、"service accounts"、"disks"、"memorystores"、"compute instances"、"run services"、"container cluster")定义数据源: 

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
  asset_types = [
    "storage.googleapis.com/Bucket",
    "iam.googleapis.com/ServiceAccount",
    "compute.googleapis.com/Disk",
    "redis.googleapis.com/Instance",
    "compute.googleapis.com/Instance",
    "run.googleapis.com/Service",
    "container.googleapis.com/Cluster"
  ]
}

data "google_storage_bucket" "my_buckets" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => project_assets.results})) : idx => result if result.asset_type == "storage.googleapis.com/Bucket"}
  name     = reverse(split("/", each.value.name))[0]
}

data "google_service_account" "my_accounts" {
  provider   = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "iam.googleapis.com/ServiceAccount"}
  account_id = reverse(split("/", each.value.name))[0]
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_compute_disk" "my_disks" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "compute.googleapis.com/Disk"}
  zone = each.value.location
  name    = reverse(split("/", each.value.name))[0]
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_redis_instance" "my_memorystores" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "redis.googleapis.com/Instance"}
  name     = reverse(split("/", each.value.name))[0]
  region   = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_compute_instance" "my_compute_instances" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "compute.googleapis.com/Instance"}
  name     = reverse(split("/", each.value.name))[0]
  zone     = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_cloud_run_service" "my_run_services" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "run.googleapis.com/Service"}
  name     = reverse(split("/", each.value.name))[0]
  location = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
}

data "google_container_cluster" "my_container_clusters" {
  provider = google-beta
  for_each = {for idx, result in flatten(values({for idx, project_assets in data.google_cloud_asset_resources_search_all.my_assets: idx => [for result in project_assets.results : merge(result, {project_id: project_assets.id})]})) : idx => result if result.asset_type == "container.googleapis.com/Cluster"}
  name     = reverse(split("/", each.value.name))[0]
  location = each.value.location
  project = reverse(split("/", each.value.project_id))[0]
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
        format = "url"
        title  = "Link"
      }
      "createTime" = {
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

## 创建存储桶蓝图和与存储桶匹配的实体

这部分包括配置 `storageBucket` 蓝图和为存储桶创建实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_bucket_blueprint" {
  title      = "Storage Bucket"
  icon       = "Bucket"
  identifier = "storageBucket"
  properties = {
    boolean_props = {
      "uniformBucketLevelAccess" = {
        title    = "Uniform Bucket Level Access"
        required = false
      }
    }
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "location" = {
        title = "Location"
      }
    }
    array_props = {
      "lifecycleRule" = {
        title        = "Lifecycle Rule"
        object_items = {}
      }
      "publicAccessPrevention" = {
        title = "Public Access Prevention"
        enum  = ["inherited", "enforced"]
      }
      "storageClass" = {
        enum  = ["STANDARD", "MULTI_REGIONAL", "REGIONAL", "NEARLINE", "COLDLINE", "ARCHIVE"]
        title = "Storage Class"
      }
      "encryption" = {
        object_items = {}
        title        = "Encryption"
      }
    }
    object_props = {
      "labels" = {
        type  = "object"
        title = "Labels"
      }
    }
  }
  relations = {
    "project" = {
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      title    = "GCP Project"
      required = false
      many     = false
    }
  }
}

resource "port-labs_entity" "gcp_bucket_entity" {
  depends_on = [port-labs_entity.gcp_project_entity]
  for_each   = data.google_storage_bucket.my_buckets
  identifier = each.value.id
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_bucket_blueprint.identifier
  properties = {
    object_props = {
      labels = jsonencode(each.value.labels)
    }
    boolean_props = {
      "uniformBucketLevelAccess" = each.value.uniform_bucket_level_access
    }
    string_props = {
      "link"     = "https://console.cloud.google.com/storage/browser/${each.value.id};tab=objects?project=${each.value.project}"
      "location" = each.value.location
      publicAccessPrevention = each.value.public_access_prevention
      storageClass           = each.value.storage_class
    }
    array_props = {
      object_items = {
        lifecycleRule = [for item in each.value.lifecycle_rule : jsonencode(item)]
        encryption    = [for item in each.value.encryption : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
}
```

## 创建服务账户蓝图和与服务账户匹配的实体

这部分包括配置 `serviceAccount` 蓝图和创建服务账户实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_service_account_blueprint" {
  title      = "Service Account"
  icon       = "Lock"
  identifier = "serviceAccount"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "email" = {
        format = "email"
        title  = "email"
      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      required = false
      many = false
    }
  }
}

resource "port-labs_entity" "gcp_service_account_entity" {
  for_each   = data.google_service_account.my_accounts
  identifier = each.value.account_id
  title      = each.value.display_name
  blueprint  = port-labs_blueprint.gcp_service_account_blueprint.identifier
  properties = {
    string_props = {
      "link"  = "https://console.cloud.google.com/iam-admin/serviceaccounts/details/${each.value.unique_id}?project=${each.value.project}"
      "email" = each.value.email
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}
```

## 创建磁盘蓝图和与磁盘匹配的实体

这部分包括配置 "磁盘 "蓝图和创建磁盘实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_disk_blueprint" {
  title      = "Disk"
  icon       = "GoogleComputeEngine"
  identifier = "disk"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "description" = {
        title = "Description"
      }
      "zone" = {
        title = "Zone"
      }
      "type" = {
        title = "Type"
      }
      "creationTimestamp" = {
        title = "Creation Timestamp"
      }
    }
    object_props = {
      "labels" = {
        title = "Labels"
      }
    }
    number_props = {
      "size" = {
        title = "Size"
      }
    }
    array_props = {
      string_items = {
        "users" = {
          title = "Users"
        }
      }
    }
  }
  relations = {
    "project" = {
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      title    = "Project"
      required = false
      many     = false
    }
  }
}

resource "port-labs_entity" "gcp_disk_entity" {
  for_each   = data.google_compute_disk.my_disks
  identifier = "${each.value.project}_${each.value.zone}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_disk_blueprint.identifier
  properties = {
    string_props = {
      "link"              = "https://console.cloud.google.com/compute/disksDetail/zones/${each.value.zone}/disks/${each.value.name}?project=${each.value.project}"
      "description"       = each.value.description
      "zone"              = each.value.zone
      "type"              = each.value.type
      "creationTimestamp" = each.value.creation_timestamp
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
    number_props = {
      "size" = each.value.size
    }
    array_props = {
      string_items = {
        "users" = each.value.users
      }
    }
    relations = {
      single_relations = {
        "project" = each.value.project
      }
    }
    depends_on = [port-labs_entity.gcp_project_entity]
  }
}
```

## 创建内存库蓝图和与内存库匹配的实体

这部分包括配置 "memorystore "蓝图和创建内存存储实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_memorystore_blueprint" {
  title      = "Memorystore"
  icon       = "GoogleCloudPlatform"
  identifier = "memorystore"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "region" = {
        title = "Region"
      }
      "currentLocationId" = {
        title = "Current Location ID"
      }
      "redisVersion" = {
        title = "Redis Version"
      }
      "tier" = {
        enum  = ["BASIC", "STANDARD_HA"]
        title = "Public Access Prevention"
      }
      "readReplicasMode" = {
        enum  = ["READ_REPLICAS_DISABLED", "READ_REPLICAS_ENABLED"]
        title = "Read Replicas Mode"
      }
      "connectMode" = {
        enum  = ["DIRECT_PEERING", "PRIVATE_SERVICE_ACCESS"]
        title = "Connect Mode"
      }
      "createTime" = {
        format = "date-time"
        title  = "Create Time"
      }
      "authorizedNetwork" = {
        title = "Authorized Network"
      }
    }
    number_props = {
      "memorySizeGb" = {
        title = "Memory Size GB"
      }
      "replicaCount" = {
        title = "Replica Count"
      }
    }
    array_props = {
      "nodes" = {
        title        = "Nodes"
        object_items = {}
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
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_memorystore_entity" {
  for_each   = data.google_redis_instance.my_memorystores
  identifier = "${each.value.project}_${each.value.location_id}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_memorystore_blueprint.identifier
  properties = {
    string_props = {
      "link"              = "https://console.cloud.google.com/memorystore/redis/locations/${each.value.region}/instances/${each.value.name}/details/overview?project=${each.value.project}"
      "region"            = each.value.region
      "currentLocationId" = each.value.current_location_id
      "redisVersion"      = each.value.redis_version
      "tier"              = each.value.tier
      "readReplicasMode"  = each.value.read_replicas_mode
      "connectMode"       = each.value.connect_mode
      "createTime"        = each.value.create_time
      "authorizedNetwork" = each.value.authorized_network
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
    array_props = {
      object_items = {
        "nodes" = [for item in each.value.nodes : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}
```

## 创建计算实例蓝图和与计算实例匹配的实体

这部分包括配置 "计算实例 "蓝图和为计算实例创建实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_compute_instance_blueprint" {
  title      = "Compute Instance"
  icon       = "GoogleComputeEngine"
  identifier = "computeInstance"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "zone" = {
        title = "Zone"
      }
      "currentStatus" = {
        enum = ["PROVISIONING", "STAGING", "RUNNING", "STOPPING", "REPAIRING", "TERMINATED", "SUSPENDING", "SUSPENDED"]
        enum_colors = {
          PROVISIONING = "blue"
          STAGING      = "blue"
          RUNNING      = "green"
          STOPPING     = "red"
          REPAIRING    = "yellow"
          TERMINATED   = "red"
          SUSPENDING   = "yellow"
          SUSPENDED    = "lightGray"
        }
        title = "Current Status"
      }
      "machineType" = {
        title = "Machine Type"
      }
      "cpuPlatform" = {
        title = "CPU Platform"
      }
    }
    boolean_props = {
      "deletionProtection" = {
        title = "Deletion Protection"
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
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_compute_instance_entity" {
  for_each   = data.google_compute_instance.my_compute_instances
  identifier = "${each.value.project}_${each.value.zone}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_compute_instance_blueprint.identifier
  properties = {
    string_props = {
      "link"          = "https://console.cloud.google.com/compute/instancesDetail/zones/${each.value.zone}/instances/${each.value.name}?project=${each.value.project}"
      "zone"          = each.value.zone
      "currentStatus" = each.value.current_status
      "machineType"   = each.value.machine_type
      "cpuPlatform"   = each.value.cpu_platform
    }
    boolean_props = {
      "deletionProtection" = each.value.deletion_protection
    }
    object_props = {
      "labels" = jsonencode(each.value.labels)
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}
```

## 创建运行服务蓝图和与运行服务匹配的实体

这部分包括配置 `runService` 蓝图和创建运行服务的实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_run_service_blueprint" {
  title      = "Run Service"
  icon       = "Service"
  identifier = "runService"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "location" = {
        title = "Location"
      }
    }
    array_props = {
      "metadata" = {
        title        = "Metadata"
        object_items = {}
      }
      "status" = {
        title        = "Status"
        object_items = {}
      }
      "template" = {
        title        = "Template"
        object_items = {}
      }
      "traffic" = {
        title        = "Traffic"
        object_items = {}

      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      required = false
      many     = false
    }
  }
}

resource "port-labs_entity" "gcp_run_service_entity" {
  for_each   = data.google_cloud_run_service.my_run_services
  identifier = "${each.value.project}_${each.value.location}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_run_service_blueprint.identifier
  properties = {
    string_props = {
      "link"     = "https://console.cloud.google.com/run/detail/${each.value.location}/${each.value.name}?project=${each.value.project}"
      "location" = each.value.location
    }
    array_props = {
      object_items = {
        "metadata" = [for item in each.value.metadata : jsonencode(item)]
        "status"   = [for item in each.value.status : jsonencode(item)]
        "template" = [for item in each.value.template : jsonencode(item)]
        "traffic"  = [for item in each.value.traffic : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}
```

## 创建容器集群蓝图和与容器集群相匹配的实体

这部分包括配置 `containerCluster` 蓝图和创建容器集群的实体: 

```hcl showLineNumbers
resource "port-labs_blueprint" "gcp_container_cluster_blueprint" {
  title      = "Container Cluster"
  icon       = "Cluster"
  identifier = "containerCluster"
  properties = {
    string_props = {
      "link" = {
        format = "url"
        title  = "Link"
      }
      "location" = {
        title = "Location"
      }
      "description" = {
        title = "Description"
      }
      "masterVersion" = {
        title = "Master Version"
      }
      "network" = {
        title = "Network"
      }
    }
    boolean_props = {
      "enableAutopilot" = {
        title = "Enable Autopilot"
      }
    }
    array_props = {
      "addonsConfig" = {
        title        = "Addons Config"
        object_items = {}
      }
      "clusterAutoscaling" = {
        title        = "Cluster Autoscaling"
        object_items = {}
      }
    }
  }
  relations = {
    "project" = {
      title    = "Project"
      target   = port-labs_blueprint.gcp_project_blueprint.identifier
      many     = false
      required = false
    }
  }
}

resource "port-labs_entity" "gcp_container_cluster_entity" {
  for_each   = data.google_container_cluster.my_container_clusters
  identifier = "${each.value.project}_${each.value.location}_${each.value.name}"
  title      = each.value.name
  blueprint  = port-labs_blueprint.gcp_container_cluster_blueprint.identifier
  properties = {
    string_props = {
      "link"           = "https://console.cloud.google.com/kubernetes/clusters/details/${each.value.location}/${each.value.name}?project=${each.value.project}"
      "location"       = each.value.location
      "description"    = each.value.description
      "master_version" = each.value.master_version
      "network"        = each.value.network
    }
    booleen_props = {
      "enableAutopilot" = each.value.enable_autopilot
    }
    array_props = {
      object_items = {
        "addonsConfig"       = [for item in each.value.addons_config : jsonencode(item)]
        "clusterAutoscaling" = [for item in each.value.cluster_autoscaling : jsonencode(item)]
      }
    }
  }
  relations = {
    single_relations = {
      "project" = each.value.project
    }
  }
  depends_on = [port-labs_entity.gcp_project_entity]
}
```

## 结果

运行 `terraform apply` 后，您将在 Port 中看到 `organization`, `folder`, `project`, `storageBucket`, `serviceAccount`, `disk`, `memorystore`, `computeInstance`, `runService`, `containerCluster` 实体。