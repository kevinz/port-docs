import ExampleImageBlueprint from "../_ci_example_image_blueprint.mdx";
import ExampleCiJobBlueprint from "../_ci_example_ci_job_blueprint.mdx";
import ExampleCiAction from "../_ci_example_action.mdx";

# 示例

## 基本创建/更新示例

在本示例中，您将为 `ciJob` 和 `镜像` 实体创建蓝图，并创建它们之间的关系。 使用 Port 的 GitHub 操作，您将在每次运行 GitHub 工作流时创建新实体: 

<ExampleImageBlueprint />

<ExampleCiJobBlueprint />

创建蓝图后，您可以在 GitHub 工作流 `yml` 文件中添加以下代码段，以便在 GitHub 工作流中创建 `ciJob` 实体: 

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    operation: UPSERT
    identifier: new-cijob-run
    icon: GithubActions
    blueprint: ciJob
    properties: |
      {
        "commitHash": "${{ env.GITHUB_SHA }}",
        "jobBranch": "${{ env.GITHUB_SERVER_URL }}/${{ env.GITHUB_REPOSITORY }}/tree/${{ env.GITHUB_REF_NAME }}",
        "runLink": "${{ env.GITHUB_SERVER_URL }}/${{ env.GITHUB_REPOSITORY }}/actions/runs/${{ env.GITHUB_RUN_ID }}",
        "triggeredBy": "${{ env.GITHUB_ACTOR }}"
      }
```

:::tip 出于安全考虑，建议将 `CLIENT_ID` 和 `CLIENT_SECRET` 保存为[GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) ，并按上例所示进行访问。

:::

## 基本获取示例

下面的示例从上一个示例中获取了 `ciJob` 实体。 如果您的 CI 流程在部署服务的最新版本时需要引用 ciJob 中的数据(例如，触发上一个作业的用户)，这将非常有用。

在 GitHub 工作流程 `yml` 文件中添加以下任务: 

```yaml showLineNumbers
get-entity:
  runs-on: ubuntu-latest
  outputs:
    entity: ${{ steps.port-github-action.outputs.entity }}
  steps:
    - id: port-github-action
      uses: port-labs/port-github-action@v1
      with:
        clientId: ${{ secrets.CLIENT_ID }}
        clientSecret: ${{ secrets.CLIENT_SECRET }}
        operation: GET
        identifier: new-cijob-run
        blueprint: ciJob

use-entity:
  runs-on: ubuntu-latest
  needs: get-entity
  steps:
    - run: echo '${{needs.get-entity.outputs.entity}}' | jq .properties.triggeredBy
```

第一个任务 "get-entity "使用 GitHub 操作获取 "new-cijob-run "实体。 第二个任务 "use-entity "被引用，并打印实体的 "triggeredBy "属性。

## 关系示例

下面的示例除了上例中的 `ciJob` 实体外，还添加了一个 `image` 实体。 `image` 实体表示 ciJob 运行时生成的工件。

将以下代码段添加到 GitHub 工作流程 `yml` 文件中: 

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    operation: UPSERT
    identifier: example-image
    title: Example Image
    icon: Docker
    blueprint: image
    properties: |
      {
        "imageTag": "v1",
        "synkHighVulnerabilities": "0",
        "synkMediumVulnerabilities": "0",
        "gitRepoUrl": "https://github.com/my-org/my-cool-repo",
        "imageRegistry": "docker.io/cool-image",
        "size": "0.71",
        "unitTestCoverage": "20",
        "unitTestCoverage": "50",
      }
```

剩下的工作就是将新的 `image` 实体映射到 `ciJob` 上，这样就可以知道 ciJob 创建了哪幅镜像。

```yaml
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    operation: UPSERT
    identifier: new-cijob-run
    icon: GithubActions
    blueprint: ciJob
    relations: |
      {
        "image": ["example-image"]
      }
```

就这样！实体已更新并在用户界面中可见。

## 基本更新运行示例

在本示例中，您将创建一个自助服务操作，用于部署服务的最新版本，并根据部署状态更新操作运行。

在`镜像`蓝图操作中添加以下操作: 

<ExampleCiAction/>

:::info 这里的示例旨在展示使用 Port 自助操作，然后使用 Port 的 GitHub 操作更新其日志、状态和其他信息时的常见流程。

要使用 Port 的 GitHub 操作进行这些更新，您的后端必须是 GitHub 工作流，或者您选择的其他后端触发 GitHub 工作流作为其逻辑的一部分。

:::

在 Port 中触发操作后，将在 Port 中创建一个新的操作运行(并生成一个匹配的 `runId`)。 runId 可用于更新操作状态和报告日志到 Port。

要更新新的自助服务操作运行，请将以下代码段添加到 GitHub 工作流 `yml` 文件中(注意，您需要将正确的 `runId` 传递给操作): 

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    operation: PATCH_RUN
    runId: ${{ env.PORT_RUN_ID }}
    icon: GithubActions
    status: "SUCCESS"
    logMessage: "Deployment completed successfully"
```

:::tip 上面的示例展示了如何更新状态并为操作运行添加新的 logging 消息，但也可以只更新操作运行的特定字段。

例如，可以触发 GitHub 操作，只更新日志，而不改变其状态。

请注意，一旦 Port 操作运行有了状态，就不能再更新，对目录所做的更改也不能再与该操作绑定，因此最好的做法是，只有当操作完成所有目录更改和逻辑后，才更新该操作的状态

:::

就是这样！操作状态和日志在 Port 中更新。