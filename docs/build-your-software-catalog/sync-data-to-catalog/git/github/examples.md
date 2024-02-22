---

sidebar_position: 2

---

import RepositoryBlueprint from './_github_exporter_example_repository_blueprint.mdx'
import PRBlueprint from './_github_exporter_example_pull_request_blueprint.mdx'
import PortAppConfig from './_github_exporter_example_port_app_config.mdx'
import GitHubResources from './_github_exporter_supported_resources.mdx'

import WorkflowBlueprint from './example-workflow-workflowrun/_git_exporter_example_workflow_blueprint.mdx'
import WorkflowRunBlueprint from './example-workflow-workflowrun/_git_exporter_example_workflow_run_blueprint.mdx'
import PortWfWfrAppConfig from './example-workflow-workflowrun/_github_exporter_example_wf_wfr_port_app_config.mdx'

import BranchBlueprint from './example-branch-protection/_git_exporter_example_branch_blueprint.mdx'
import PortBrAppConfig from './example-branch-protection/_github_exporter_example_branch_port_app_config.mdx'

import PortMonoRepoAppConfig from './example-monorepo/_github_exporter_example_monorepo_port_app_config.mdx'

import IssueBlueprint from './example-issue/_git_exporter_example_issue_blueprint.mdx'
import PortIssueAppConfig from './example-issue/_github_exporter_example_issue_port_app_config.mdx'

import PRFolderBlueprint from './example-repository-folders/_github_exporter_example_pull_request_blueprint.mdx'
import FolderBlueprint from './example-repository-folders/_github_exporter_example_folder_blueprint.mdx'
import PortFolderMappingAppConfig from './example-repository-folders/_github_exporter_example_repo_folders_port_app_config.mdx'

import TeamBlueprint from './example-repository-teams/_github_export_example_team_blueprint.mdx'
import RepositoryTeamBlueprint from './example-repository-teams/_github_export_example_repository_with_teams_relation_blueprint.mdx'
import PortRepositoryTeamMappingAppConfig from './example-repository-teams/_github_exporter_example_repository_with_teams_port_app_config.mdx'

import DependabotAlertBlueprint from './example-repository-alerts/_github_exporter_example_dependabot_alert_blueprint.mdx'
import CodeScanAlertBlueprint from './example-repository-alerts/_github_exporter_example_codeScan_alert_blueprint.mdx'

import PortRepositoryDependabotAlertMappingAppConfig from './example-repository-alerts/_github_exporter_example_repo_dependabot_port_app_config.mdx'

import RepoEnvironmentBlueprint from './example-deployments-environments/_github_exporter_example_environment_blueprint.mdx'
import DeploymentBlueprint from './example-deployments-environments/_github_exporter_example_deployment_blueprint.mdx'
import PortRepoDeploymentAndEnvironmentAppConfig from './example-deployments-environments/_github_exporter_example_deployments_and_environments_port_app_config.mdx'

import UsersBlueprint from './example-repository-admins/_github_exporter_example_users_blueprint.mdx'
import GithubUsersBlueprint from './example-repository-admins/_github_exporter_example_github_users_blueprint.mdx'
import RepositoryAdminBlueprint from './example-repository-admins/_github_export_example_repository_with_admins_relation_blueprint.mdx'
import RepositoryAdminAppConfig from './example-repository-admins/_github_exporter_example_admins_users_port_app_config.mdx'

# 示例

## 映射软件源、文件内容和拉取请求

在下面的示例中，您将把 GitHub 仓库、其 README.md 文件内容和拉取请求引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<PRBlueprint/>

<PortAppConfig/>

:::tip 

* 请参阅[setup](github.md#setup) 部分，了解有关 `port-app-config.yml` 设置过程的更多信息；
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitHub 对象映射和转换为 Port 实体；
* 点击[Here](https://docs.github.com/en/rest/repos/repos#get-a-repository) 查看 GitHub 仓库对象结构。
* 点击[Here](https://docs.github.com/en/rest/pulls/pulls#get-a-pull-request) 查看 GitHub 拉取请求对象结构。

:::

创建蓝图并将 `port-app-config.yml` 文件提交到您的 `.github-private` 版本库(用于全局配置)或任何特定版本库(用于每个版本库配置)后，您将在 Port 中看到与您的版本库匹配的新实体，以及它们的 README.md 文件内容和拉取请求(请记住，`port-app-config.yml` 文件必须位于版本库的**默认分支**中才能生效)。

## 映射存储库、工作流和工作流运行

在下面的示例中，您将把 GitHub 资源库、其工作流和工作流运行被引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<WorkflowBlueprint/>

<WorkflowRunBlueprint/>

<PortWfWfrAppConfig/>

:::tip 

* 请参阅[setup](github.md#setup) 部分，了解有关 `port-app-config.yml` 设置过程的更多信息；
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitHub 对象映射和转换为 Port 实体；
* 点击[Here](https://docs.github.com/en/rest/repos/repos#get-a-repository) 查看 GitHub 仓库对象结构。
* 点击[Here](https://docs.github.com/en/rest/actions/workflows#get-a-workflow) 查看 GitHub 工作流对象结构。
* 点击[Here](https://docs.github.com/en/rest/actions/workflow-runs#get-a-workflow-run) 查看 GitHub 工作流运行对象结构。

:::

创建蓝图并提交`port-app-config.yml`文件到您的`.github-private`版本库(用于全局配置)或任何特定版本库(用于每个版本库配置)后，您将在 Port 中看到与您的版本库匹配的新实体，以及它们的工作流和工作流运行(请记住，`port-app-config.yml`文件必须在版本库的**默认分支**中才能生效)。

## 映射存储库和问题

在下面的示例中，您将把 GitHub 仓库及其问题引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<IssueBlueprint/>

<PortIssueAppConfig/>

:::tip 

* 请参阅[setup](github.md#setup) 部分，了解有关 `port-app-config.yml` 设置过程的更多信息；
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitHub 对象映射和转换为 Port 实体；
* 点击[Here](https://docs.github.com/en/rest/repos/repos#get-a-repository) 查看 GitHub 仓库对象结构。
* 点击[Here](https://docs.github.com/en/rest/issues/issues#get-an-issue) 查看 GitHub 问题对象结构。

:::

创建蓝图并提交`port-app-config.yml`文件到您的`.github-private`版本库(用于全局配置)或任何特定版本库(用于每个版本库配置)后，您将在 Port 中看到与您的版本库相匹配的新实体及其问题(请记住，`port-app-config.yml`文件必须位于版本库的**默认分支**中才能生效)。

## 映射软件源和单核处理器

在下面的示例中，您将把 GitHub 软件源及其文件夹引用到 Port。 按照这个示例，您可以将不同的服务、包和库从 monorepo 映射到 Port 中的不同实体。您可以使用下面的 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<PortMonoRepoAppConfig/>

:::tip 要引用 monorepo 的根文件夹，可以在 `port-app-config.yml` 中使用以下语法: 

```yaml
- kind: folder
    selector:
      query: "true" # JQ boolean query. If evaluated to false - skip syncing the object.
      folders: # Specify the repositories and folders to include under this relative path.
        - path: "*" # Relative path to the folders within the repositories.
          repos: # List of repositories to include folders from.
            - backend-service
            - frontend-service
```

:::
:::tip 

* 请参阅[setup](github.md#setup) 部分，了解有关 `port-app-config.yml` 设置过程的更多信息；
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitHub 对象映射和转换为 Port 实体；
* 点击[Here](https://docs.github.com/en/rest/repos/repos#get-a-repository) 查看 GitHub 仓库对象结构。
* 点击[Here](https://docs.github.com/en/rest/git/trees#get-a-tree) 查看 GitHub 文件夹对象结构。

:::

## 映射版本库、版本库文件夹和拉取请求

在下面的示例中，您将把 GitHub 仓库、仓库的根文件夹和仓库的拉取请求引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<PRFolderBlueprint/>

<FolderBlueprint/>

<PortFolderMappingAppConfig/>

## 映射资源库和团队

在下面的示例中，您将把 GitHub 仓库及其团队引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

:::note 团队是 GitHub 组织级别的资源，因此您需要在[global integration configuration](github.md#setup) 中指定团队的映射(通过 Port 的用户界面或 `.github-private` 仓库中的 `port-app-config.yml` 文件) 。

:::

<TeamBlueprint/>

<RepositoryTeamBlueprint/>

<PortRepositoryTeamMappingAppConfig/>

:::tip 要检索版本库的团队，您需要在 `port-app-config.yml` 中的版本库资源类型的 `selector` 中添加 `teams` 属性: 

```yaml
- kind: repository
    selector:
    	query: 'true'  # JQ boolean query. If evaluated to false - skip syncing the object.
      // highlight-next-line
    	teams: true  # Boolean flag to indicate whether to include the repository teams.
```

:::

## 映射存储库、部署和环境

在下面的示例中，您将把 GitHub 软件源及其部署和环境引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<RepoEnvironmentBlueprint/>

<DeploymentBlueprint/>

<PortRepoDeploymentAndEnvironmentAppConfig/>

## 映射软件源、Dependabot 警报和代码扫描警报

在下面的示例中，您将把 GitHub 仓库及其警报(Dependabot 和代码扫描警报)被引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<DependabotAlertBlueprint/>

<CodeScanAlertBlueprint/>

<PortRepositoryDependabotAlertMappingAppConfig/>

:::info  支持的警报 对于代码扫描警报，只支持默认分支上的打开警报

:::

## 映射版本库和分支保护规则

在下面的示例中，您将把 GitHub 仓库及其主要分支保护规则被引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryBlueprint/>

<BranchBlueprint/>

<PortBrAppConfig/>

:::info  支持的分支保护规则 目前只支持默认的分支保护规则

:::

## 映射版本库、版本库管理员和用户

在下面的示例中，您将把 GitHub 仓库、其管理员和相关用户引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<RepositoryAdminBlueprint/>

<GithubUsersBlueprint/>

<UsersBlueprint/>

<RepositoryAdminAppConfig/>

:::info  支持的 GitHub 用户类型 由于 Github 有严格的隐私政策，GitHub API 只在以下情况下返回电子邮件: 

1. 用户有公开的电子邮件地址
2. 您的组织正在使用 GitHub 企业云计划，用户在 GitHub 组织内配置了 SAML SSO 身份。

在其他情况下，GitHub API 会返回用户电子邮件的 "空 "值。

:::

## 映射支持的资源

上述示例展示了一个特定的使用案例，但 Port 的 GitHub 应用程序支持摄取许多其他 GitHub 对象，要调整上述示例，请使用 GitHub API 参考资料了解不同支持对象的可用字段: 

<GitHubResources/>

在添加其他资源的摄取时，请记得在 `resources` 数组中添加一个条目，并相应更改提供给 `kind` 键的值。