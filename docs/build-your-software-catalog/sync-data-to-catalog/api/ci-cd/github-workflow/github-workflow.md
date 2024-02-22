---

sidebar_position: 1

---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# GitHub 工作流程

我们的[GitHub action](https://github.com/marketplace/actions/port-github-action) 允许您直接从 GitHub 工作流中创建/更新和查询 Port 中的实体。

<br></br>
<br></br>

![Github Illustration](/img/build-your-software-catalog/sync-data-to-catalog/github/github-action-illustration.jpg)

:::tip  公共仓库 我们的 GitHub 行动是开源的，请参见[**here**](https://github.com/port-labs/port-github-action)

:::

## 💡 常见的 Github 工作流程 Usage

例如，Port 的 GitHub 操作提供了将 Port 与 GitHub 工作流集成的原生方式: 

* 报告正在运行的**CI任务**的状态；
* 更新软件目录中有关**微服务**新**构建版本的信息；
* 获取现有**实体**。

## 安装

要安装 Port 的 GitHub 操作，请按照以下步骤操作: 

1. 在 GitHub 工作流程中添加以下一行作为一个步骤: 

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
```

2.将您的 Port `CLIENT_ID` 和 `CLIENT_SECRET` 添加为[GitHub secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) ；
    1.此步骤不是强制性的，但为了避免在工作流程中以明文传递 `CLIENT_ID` 和 `CLIENT_SECRET`，建议使用此步骤；
3. 确保您的 Port 安装中已有蓝图，以便使用 GitHub 操作创建/更新实体。

## Usage

Port 的 GitHub 操作支持以下方法: 

* 创建/更新目录实体--使用 "UPSERT "操作调用，接收新实体或需要更新的实体的标识符和其他属性；
* 批量创建/更新目录实体--使用 "BULK_UPSERT "操作调用，接收一些新实体或需要更新的实体的实体定义；
* 获取目录实体--使用 "GET "操作调用，接收现有实体的标识符并将其检索出来供 CI 使用；
* 搜索目录实体 - 使用`SEARCH`操作调用，接收查询并检索实体供 CI 使用；
* 删除目录实体--使用`DELETE`操作调用，接收现有实体的标识符并将其删除；
* 更新正在运行的操作 - 使用`PATCH_RUN`操作调用，接收现有操作运行的标识符以及需要更新的运行的其他属性。
* 创建一个正在运行的操作 - 使用 `CREATE_RUN` 操作调用，接收现有蓝图、操作和实体(可选)的标识符，以及要运行操作的输入属性。

<Tabs groupId="usage" defaultValue="upsert" values={[
{label: "Create/Update Entity", value: "upsert"},
{label: "Bulk Create/Update Entities", value: "bulk_upsert"},
{label: "Get Entity", value: "get"},
{label: "Search Entities", value: "search"},
{label:"Delete Entity", value: "delete"},
{label: "Update Running Action", value: "patch_run"},
{label: "Create Action Run", value: "create_run"}
]}>

<TabItem value="upsert">

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    # highlight-next-line
    operation: UPSERT
    identifier: myEntity
    icon: myIcon
    blueprint: myBlueprint
    team: "['myTeam']"
    properties: |-
      {
        "myStringProp": "My value",
        "myNumberProp": 1,
        "myBooleanProp": true,
        "myArrayProp": ["myVal1", "myVal2"],
        "myObjectProp": {"myKey": "myVal", "myExtraKey": "myExtraVal"}
      }
```

</TabItem>

<TabItem value="bulk_upsert">

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    # highlight-next-line
    operation: BULK_UPSERT
    runId: myRunId
    entities: |-
      [
       {
          "identifier": "myEntity",
          "icon": "myIcon",
          "blueprint": "myBlueprint",
          "team": [
             "myTeam"
          ],
          "properties": {
             "myStringProp": "My value",
             "myNumberProp": 1,
             "myBooleanProp": true,
             "myArrayProp": [
                "myVal1",
                "myVal2"
             ],
             "myObjectProp": {
                "myKey": "myVal",
                "myExtraKey": "myExtraVal"
             }
          }
       }
      ]
```

</TabItem>

<TabItem value="get">

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
        # highlight-next-line
        operation: GET
        identifier: myEntity
        blueprint: myBlueprint
use-entity:
  runs-on: ubuntu-latest
  needs: get-entity
  steps:
    - run: echo '${{needs.get-entity.outputs.entity}}' | jq .properties.myProp
```

</TabItem>

<TabItem value="search">

```yaml showLineNumbers
search-entities:
  runs-on: ubuntu-latest
  outputs:
    entities: ${{ steps.port-github-action.outputs.entities }}
  steps:
    - id: port-github-action
      uses: port-labs/port-github-action@v1
      with:
        clientId: ${{ secrets.CLIENT_ID }}
        clientSecret: ${{ secrets.CLIENT_SECRET }}
        # highlight-next-line
        operation: SEARCH
        query: |-
          {
             "rules": [
                {
                   "operator": "=",
                   "value": "myEntity",
                   "property": "$identifier"
                }
             ],
             "combinator": "and"
          }
use-entities:
  runs-on: ubuntu-latest
  needs: search-entities
  steps:
    - run: echo '${{needs.search-entities.outputs.entities}}' | jq .[0].myProp
```

</TabItem>

<TabItem value="delete">

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    # highlight-next-line
    operation: DELETE
    identifier: myEntity
    blueprint: myBlueprint
```

</TabItem>

<TabItem value="patch_run">

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    # highlight-next-line
    operation: PATCH_RUN
    runId: myRunId
    status: "SUCCESS"
    logMessage: "My log message"
    summary: "My summary"
    link: `["https://mylink.com"]`
```

</TabItem>

<TabItem value="create_run">

```yaml showLineNumbers
- uses: port-labs/port-github-action@v1
  with:
    clientId: ${{ secrets.CLIENT_ID }}
    clientSecret: ${{ secrets.CLIENT_SECRET }}
    # highlight-next-line
    operation: CREATE_RUN
    icon: GithubActions
    blueprint: myBlueprint
    action: myAction
    identifier: myEntity
    properties: |-
      {
        "text": "test",
        "number": 1,
        "boolean": true
      }
```

</TabItem>
</Tabs>

## 示例

有关 Port 在 GitHub 上的实际操作示例，请参阅[examples](./examples.md) 页面。