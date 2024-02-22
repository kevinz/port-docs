---
sidebar_position: 2
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"

# Codefresh 工作流程模板

我们的[Codefresh workflow template](https://github.com/port-labs/port-codefresh-workflow-template) 允许您直接从 Codefresh 工作流程模板中创建/更新和查询 Port 中的实体。

<br></br>
<br></br>

![CircleCI Illustration](/img/build-your-software-catalog/sync-data-to-catalog/codefresh/codefresh-iilustration.jpg)

## 💡 Codefresh 集成 Usage

例如，Port 的 Codefresh 工作流模板提供了将 Port 与 Codefresh CI 工作流集成的原生方法: 

* 报告正在运行的**CI任务**的状态；
* 更新软件目录中有关**微服务**新**构建版本的信息；
* 获取现有**实体**。

## 安装

要被引用 Port 的 Codefresh 工作流程模板，您需要执行以下步骤: 

1. 访问 GitHub 中的[workflow template repository](https://github.com/port-labs/port-codefresh-workflow-template) ；
2. 将文件[`portWorkflowTemplate.yml`](https://github.com/port-labs/port-codefresh-workflow-template/blob/main/portWorkflowTemplate.yml) 复制到您的一个 codefresh `git 源码`，然后提交到您的 git 源码；
3. 使用命令 apply[`rbac.yml`](https://github.com/port-labs/port-codefresh-workflow-template/blob/main/rbac.yml) 文件的内容，将所需的服务账户、集群角色和角色绑定添加到您的 codefresh 运行时名称空间: `kubectl apply -f rbac.yml -n YOUR_NAMESPACE`；
4. 使用 base64 编码后，添加包含`PORT_CLIENT_ID`和`PORT_CLIENT_SECRET`的所需 secret。可被引用[`portCredentials.yml`](https://github.com/port-labs/port-codefresh-workflow-template/blob/main/portCredentials.yml) 作为示例。

:::tip 如果使用 `portCredentials.yml` 中显示的准确格式保存 CLIENT_ID 和 SECRET，那么从工作流模板调用模板时就不需要提供参数 `PORT_CREDENTIALS_SECRET`、`PORT_CLIENT_ID_KEY` 和 `PORT_CLIENT_SECRET_KEY`。

:::

#### 验证

要验证工作流程模板的安装，请按照以下步骤操作: 

1. 进入 Codefresh 界面；
2. 在 CI OPS 类别下，点击工作流程模板；
3. 在搜索栏中输入 "Port"，工作流程模板就会出现。

## Usage

Port 的 Codefresh 工作流程模板支持以下方法: 

* 创建/更新目录实体--使用 "upsert-entity "模板调用，接收新实体或需要更新的实体的标识符和其他属性；
* 获取目录实体--使用 "get-entity "模板调用，接收现有实体的标识符并将其检索出来供 CI 使用。

<Tabs groupId="usage" defaultValue="upsert" values={[
{label: "Create/Update", value: "upsert"},
{label: "Get", value: "get"}
]}>

<TabItem value="upsert">

```yaml showLineNumbers
- name: entity-upsert
  templateRef:
    name: port
    # highlight-next-line
    template: entity-upsert
  arguments:
    parameters:
    # Note: if you save the CLIENT_ID and CLIENT_SECRET in the same format shown
    # in the portCredentials.yml file, there is no need to provide
    # PORT_CREDENTIALS_SECRET, PORT_CLIENT_ID_KEY, PORT_CLIENT_SECRET_KEY
    - name: PORT_CREDENTIALS_SECRET
       value: "port-credentials"
    - name: PORT_CLIENT_ID_KEY
      value: "PORT_CLIENT_ID"
    - name: PORT_CLIENT_SECRET_KEY
      value: "PORT_CLIENT_SECRET"
    - name: BLUEPRINT_IDENTIFIER
      value: "myBlueprint"
    - name: ENTITY_IDENTIFIER
      value: "myEntity"
    - name: ENTITY_TITLE
      value: "myTitle"
    - name: ENTITY_PROPERTIES
      value: |
      {
        "myStringProp": "My value",
        "myNumberProp": 1,
        "myBooleanProp": true,
        "myArrayProp": ["myVal1", "myVal2"],
        "myObjectProp": {"myKey": "myVal", "myExtraKey": "myExtraVal"}
      }
```

**输入**


| Input                     | Description                                                               | Notes                                           |
| ------------------------- | ------------------------------------------------------------------------- | ----------------------------------------------- |
| `PORT_CREDENTIALS_SECRET` | Name of the secret to get the `CLIENT_ID` and `CLIENT_SECRET` from        | Default value: `port-credentials`               |
| `PORT_CLIENT_ID_KEY`      | Key in the secret where the base64 encoded `PORT_CLIENT_ID` is stored     | Default value: `PORT_CLIENT_ID`                 |
| `PORT_CLIENT_SECRET_KEY`  | key in the secret where the base64 encoded `PORT_CLIENT_SECRET` is stored | Default value: `PORT_CLIENT_SECRET`             |
| `BLUEPRINT_IDENTIFIER`    | Identifier of the blueprint to create an entity of                        | **Required**                                    |
| `ENTITY_IDENTIFIER`       | Identifier of the new (or existing) entity                                | Leave empty to get an auto-generated identifier |
| `ENTITY_TITLE`            | Title of the new (or existing) entity                                     |                                                 |
| `ENTITY_TEAM`             | Teams array of the new (or existing) entity                               |                                                 |
| `ENTITY_ICON`             | Icon of the new (or existing) entity                                      |                                                 |
| `ENTITY_PROPERTIES`       | Properties of the new (or existing) entity                                |                                                 |
| `ENTITY_RELATIONS`        | Relations of the new (or existing) entity.                                |                                                 |


**输出**


| Output              | Description                                | Notes |
| ------------------- | ------------------------------------------ | ----- |
| `ENTITY_IDENTIFIER` | identifier of the new (or existing) entity |       |


</TabItem>
<TabItem value="get">

```yaml showLineNumbers
- name: entity-get
  templateRef:
    name: port
    # highlight-next-line
    template: entity-get
  arguments:
    parameters:
    # Note: if you save the CLIENT_ID and CLIENT_SECRET in the same format shown
    # in the portCredentials.yml file, there is no need to provide
    # PORT_CREDENTIALS_SECRET, PORT_CLIENT_ID_KEY, PORT_CLIENT_SECRET_KEY
    - name: PORT_CREDENTIALS_SECRET
       value: "port-credentials"
    - name: PORT_CLIENT_ID_KEY
      value: "PORT_CLIENT_ID"
    - name: PORT_CLIENT_SECRET_KEY
      value: "PORT_CLIENT_SECRET"
    - name: BLUEPRINT_IDENTIFIER
      value: "microservice"
    - name: ENTITY_IDENTIFIER
      value: "morp"
```

**输入**


| Input                     | Description                                                               | Notes                               |
| ------------------------- | ------------------------------------------------------------------------- | ----------------------------------- |
| `PORT_CREDENTIALS_SECRET` | Name of the secret to get the `CLIENT_ID` and `CLIENT_SECRET` from        | Default value: `port-credentials`   |
| `PORT_CLIENT_ID_KEY`      | Key in the secret where the base64 encoded `PORT_CLIENT_ID` is stored     | Default value: `PORT_CLIENT_ID`     |
| `PORT_CLIENT_SECRET_KEY`  | Key in the secret where the base64 encoded `PORT_CLIENT_SECRET` is stored | Default value: `PORT_CLIENT_SECRET` |
| `BLUEPRINT_IDENTIFIER`    | Identifier of the blueprint the target entity is from                     | **Required**                        |
| `ENTITY_IDENTIFIER`       | Identifier of the target entity                                           |                                     |


**输出**


| Output                      | Description                                           | Notes |
| --------------------------- | ----------------------------------------------------- | ----- |
| `PORT_COMPLETE_ENTITY`      | Complete entity JSON                                  |       |
| `PORT_BLUEPRINT_IDENTIFIER` | Identifier of the blueprint the target entity is from |       |
| `PORT_ENTITY_IDENTIFIER`    | Identifier of the target entity                       |       |
| `PORT_ENTITY_TITLE`         | Title of the entity                                   |       |
| `PORT_ENTITY_PROPERTIES`    | All properties of the entity in JSON format           |       |
| `PORT_ENTITY_RELATIONS`     | All relations of the entity in JSON format            |       |


</TabItem>
</Tabs>

## 示例

有关 Port 的 Codefresh 工作流程模板的实用示例，请参阅[examples](./examples.mdx) 页面。