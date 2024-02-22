---

sidebar_position: 2

---

import MicroserviceBlueprint from './_bitbucket_exporter_example_repository_blueprint.mdx'
import PRBlueprint from './_bitbucket_exporter_example_pull_request_blueprint.mdx'
import PortAppConfig from './_bitbucket_exporter_example_port_app_config.mdx'
import BitbucketResources from './_bitbucket_exporter_supported_resources.mdx'
import PortMonoRepoAppConfig from './_bitbucket_exporter_example_monorepo_port_app_config.mdx'

# 示例

## 映射软件源、文件内容和拉取请求

在下面的示例中，您将把 Bitbucket 仓库、其 README.md 文件内容和拉取请求引用到 Port，您可以使用以下 Port 蓝图定义和 `port-app-config.yml`: 

<MicroserviceBlueprint/>

<PRBlueprint/>

<PortAppConfig/>

:::tip 

* 请参阅[setup](bitbucket.md#setup) 部分，了解有关 `port-app-config.yml` 设置过程的更多信息；
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 Bitbucket 对象映射和转换为 Port 实体；
* 点击[Here](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repositories/#api-repositories-workspace-repo-slug-get) 查看 Bitbucket 版本库对象结构。
* 点击[Here](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-pullrequests/#api-repositories-workspace-repo-slug-pullrequests-pull-request-id-get) 查看 Bitbucket 拉取请求对象结构。

:::

创建蓝图并提交`port-app-config.yml`文件到`.bitbucket-private`或特定版本库后，您将在版本库中看到与您的版本库相匹配的新实体，以及它们的 README.md 文件内容和拉取请求(请记住，`port-app-config.yml`文件必须位于版本库的**默认分支**中才能生效)。

## 映射软件源和单核处理器

在下面的示例中，您将把 Bitbucket 资源库及其文件夹引用到 Port。 按照这个示例，您可以将不同的服务、包和库从 monorepo 映射到 Port 中的不同实体。您可以使用下面的 Port 蓝图定义和 `port-app-config.yml`: 

<MicroserviceBlueprint/>

<PortMonoRepoAppConfig/>

:::tip 

* 请参阅[setup](bitbucket.md#setup) 部分，了解有关 `port-app-config.yml` 设置过程的更多信息；
* 我们利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 将 GitHub 对象映射和转换为 Port 实体；
* 点击[Here](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repositories/#api-repositories-workspace-repo-slug-get) 查看 Bitbucket 仓库对象结构。
* 点击[Here](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-source/#api-repositories-workspace-repo-slug-src-commit-path-get) 查看 Bitbucket 文件夹对象结构。

:::

## 映射支持的资源

上面的示例展示了一个特定的用例，但 Port 的 Bitbucket 应用程序支持摄取许多其他 Bitbucket 对象，要调整上面的示例，请使用 Bitbucket API 参考资料了解不同支持对象的可用字段: 

<BitbucketResources/>

在添加其他资源的摄取时，请记得在 `resources` 数组中添加一个条目，并相应更改提供给 `kind` 键的值。