---

sidebar_position: 2

---

import RepositoryBlueprint from './_gitlab_exporter_example_repository_blueprint.mdx'
import PRBlueprint from './_gitlab_exporter_example_merge_request_blueprint.mdx'
import PortAppConfig from './_gitlab_exporter_example_port_app_config.mdx'
import GitlabResources from './_gitlab_exporter_supported_resources.mdx'

import PipelineBlueprint from './example-pipeline-job/_git_exporter_example_pipeline_blueprint.mdx'
import JobBlueprint from './example-pipeline-job/_git_exporter_example_job_blueprint.mdx'
import PortPipelineJobAppConfig from './example-pipeline-job/_gitlab_exporter_example_pipeline_job_port_app_config.mdx'

import IssueBlueprint from './example-issue/_git_exporter_example_issue_blueprint.mdx'
import PortIssueAppConfig from './example-issue/_gitlab_exporter_example_issue_port_app_config.mdx'

import MonoRepoAppConfig from './example-repository-folders/_gitlab_export_example_monorepo_port_app_config.mdx'

import FolderBlueprint from './example-repository-folders/_gitlab_exporter_example_folder_blueprint.mdx'
import PortFoldersAppConfig from './example-repository-folders/_gitlab_exporter_example_repo_folders_port_app_config.mdx'

import RepositoryGroupBlueprint from './example-groups-subgroups/_gitlab_exporter_example_repository_blueprint.mdx'
import GroupBlueprint from './example-groups-subgroups/_gitlab_exporter_example_group_blueprint.mdx'
import PortGroupsAppConfig from './example-groups-subgroups/_gitlab_exporter_example_group_repository_port_app_config.mdx'

# 示例

## 映射项目、文件内容和合并请求

在下面的示例中，您将把 GitLab 项目、其 README.md 文件内容和合并请求引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<RepositoryBlueprint/>

<PRBlueprint/>

<PortAppConfig/>

:::tip  了解更多信息

* 请参阅[setup](gitlab.md#setup) 部分，了解集成配置设置过程的更多信息。
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitLab 对象映射和转换为 Port 实体。
* 点击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects) 查看 GitLab 项目对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/merge_requests.html#list-project-merge-requests) 查看 GitLab 合并请求对象结构。

:::

创建蓝图并保存集成配置后，您将在 Port 中看到与您的项目相匹配的新实体，以及它们的 `README.md` 文件内容和合并请求。

## 映射组、分组和项目

在以下示例中，您将把 GitLab 组、子组和项目被引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<GroupBlueprint/>

<RepositoryGroupBlueprint/>

<PortGroupsAppConfig/>

:::tip  了解更多信息

* 请参阅[setup](gitlab.md#setup) 部分，了解集成配置设置过程的更多信息。
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitLab 对象映射和转换为 Port 实体。
* 单击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects) 查看 GitLab 项目对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-subgroups) 查看 GitLab 子组对象结构。

:::

## 绘制项目、Pipelines 和工作图

在下面的示例中，您将把 GitLab 项目及其 Pipelines 和作业运行引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<RepositoryBlueprint/>

<PipelineBlueprint/>

<JobBlueprint/>

<PortPipelineJobAppConfig/>

:::tip  了解更多信息

* 请参阅[setup](gitlab.md#setup) 部分，了解集成配置设置过程的更多信息。
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitLab 对象映射和转换为 Port 实体。
* 点击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects) 查看 GitLab 项目对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/pipelines.html#list-project-pipelines) 查看 GitLab Pipelines 对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/jobs.html#list-project-jobs) 查看 GitLab 作业对象结构。

:::

创建蓝图并保存集成配置后，您将在 Port 中看到与您的项目相匹配的新实体，以及它们的 Pipelines 和作业。

## 映射项目和 monorepos

在以下示例中，您将把 GitLab 项目及其 monorepo 文件夹引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<RepositoryBlueprint/>

<MonoRepoAppConfig/>

:::tip  要了解更多信息 要检索 monorepo 的根文件夹，可以在 `port-app-config.yml` 中使用以下语法: 

```yaml
- kind: folder
  selector:
    query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
    folders: # Specify the repositories and folders to include under this relative path.
      - path: "/" # Relative path to the folders within the repositories
        repos: # List of repositories to include folders from.
          - backend-service
          - frontend-service
```

:::

:::tip 

例如，您也可以为每个 monorepo 版本库指定不同的路径: 

```yaml
- kind: folder
  selector:
    query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
    folders: # Specify the repositories and folders to include under this relative path.
      - path: "apps/"
        repos:
          - gaming-apps
      - path: "microservices/"
        repos:
          - backend-services
```

:::

:::tip 

* 请参阅[setup](gitlab.md#setup) 部分，了解集成配置设置过程的更多信息。
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitLab 对象映射和转换为 Port 实体。
* 点击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects) 查看 GitLab 项目对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/repositories.html#list-repository-tree) 查看 GitLab 仓库树对象结构。

:::

## 映射项目和文件夹

在以下示例中，您将把 GitLab 项目及其文件夹引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<RepositoryBlueprint/>

<FolderBlueprint/>

<PortFoldersAppConfig/>

:::tip  了解更多信息

* 请参阅[setup](gitlab.md#setup) 部分，了解集成配置设置过程的更多信息。
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitLab 对象映射和转换为 Port 实体。
* 点击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects) 查看 GitLab 项目对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/repositories.html#list-repository-tree) 查看 GitLab 仓库树对象结构。

:::

## 绘制项目和问题图

在下面的示例中，您将把 GitLab 项目及其问题引用到 Port，您可以使用以下 Port 蓝图定义和集成配置: 

<RepositoryBlueprint/>

<IssueBlueprint/>

<PortIssueAppConfig/>

:::tip  了解更多信息

* 请参阅[setup](gitlab.md#setup) 部分，了解集成配置设置过程的更多信息。
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitLab 对象映射和转换为 Port 实体。
* 点击[Here](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects) 查看 GitLab 项目对象结构。
* 点击[Here](https://docs.gitlab.com/ee/api/issues.html#list-project-issues) 查看 GitLab 问题对象结构。

:::

创建蓝图并保存集成配置后，您将在 Port 中看到与项目相匹配的新实体及其问题。

## 映射支持的资源

上述示例展示了一个特定的使用案例，但 Port 的 GitLab 集成支持摄取许多其他 GitLab 对象，要调整上述示例，请使用 GitLab API 参考资料了解不同支持对象的可用字段: 

<GitlabResources/>

在添加其他资源的摄取时，请记得在 `resources` 数组中添加一个条目，并相应更改提供给 `kind` 键的值。