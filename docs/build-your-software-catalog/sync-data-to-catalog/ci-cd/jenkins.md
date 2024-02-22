---
sidebar_position: 1
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import Prerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import AzurePremise from "../templates/_ocean_azure_premise.mdx"
import DockerParameters from "./_jenkins-docker-parameters.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'

# Jenkins

我们的 Jenkins 集成允许您根据映射和定义将 Jenkins 环境中的 "任务"、"构建 "和 "用户 "导入 Port。

## 常见被用于情况

* 映射 Jenkins 环境中的 "工作"、"构建 "和 "用户"。
* 实时观察对象更改(创建/更新/删除)，并自动将更改应用到 Port 中的实体。

## 先决条件

<Prerequisites />

生成用于验证 Jenkins API 调用的令牌: 

1. 在 Jenkins 横幅框中，点击用户名打开用户菜单。
2. 导航至您的**用户名** > **配置** > **API令牌**。
3. 单击添加新令牌。
4. 单击生成。
5. 复制生成的 API 令牌作为 `JENKINS_TOKEN`。

<img src='/img/build-your-software-catalog/sync-data-to-catalog/jenkins/configure-api-token.png' width='80%' />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                           | Description                                                                                                        | Required |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------- |
| `port.clientId`                     | Your port client id ([Get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))                                                                                               | ✅       |
| `port.clientSecret`                 | Your port client secret ([Get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))                                                                                           | ✅       |
| `integration.identifier`            | Change the identifier to describe your integration                                                                 | ✅       |
| `integration.type`                  | The integration type                                                                                               | ✅       |
| `integration.eventListener.type`    | The event listener type                                                                                            | ✅       |
| `integration.secrets.jenkinsUser`   | The Jenkins username                                                                                               | ✅       |
| `integration.secrets.jenkinsToken`  | The Jenkins password or token                                                                                     | ✅       |
| `integration.config.jenkinsHost`    | The Jenkins host                                                                                                  | ✅       |
| `integration.config.appHost`        | The host of the Port Ocean app. Used to set up the integration endpoint as the target for webhooks created in Jenkins  | ❌       |
| `scheduledResyncInterval`           | The number of minutes between each resync                                                                          | ❌       |
| `initializePortResources`           | Default true, When set to true the integration will create default blueprints and the port App config Mapping      | ❌       |


<br/>

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-jenkins-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=120 \
    --set integration.identifier="my-jenkins-integration"  \
    --set integration.type="jenkins"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.secrets.jenkinsUser="JENKINS_USER"  \
    --set integration.secrets.jenkinsToken="JENKINS_TOKEN" \
    --set integration.config.jenkinsHost="JENKINS_HOST"
```

</TabItem>

<TabItem value="one-time" label="Scheduled">
  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the Jenkins integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />
<br/>

下面是 `jenkins-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: Jenkins Exporter Workflow

# This workflow responsible for running Jenkins exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Jenkins Integration
        run: |
          # Set Docker image and run the container
          integration_type="jenkins"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
            -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
            -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
            -e OCEAN__INTEGRATION__CONFIG__JENKINS_USER=${{ secrets.OCEAN__INTEGRATION__CONFIG__JENKINS_USER }} \
            -e OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN }} \
            -e OCEAN__INTEGRATION__CONFIG__JENKINS_HOST=${{ secrets.OCEAN__INTEGRATION__CONFIG__JENKINS_HOST }} \
            -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
            -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
            $image_name

          exit $?
```

</TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the Jenkins integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Jenkins 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

请确保配置以下[Jenkins Credentials](https://www.jenkins.io/doc/book/using/using-credentials/)的 "Secret Text "类型: 

<DockerParameters />
<br/>

下面是 `Jenkinsfile` groovy Pipelines 文件的示例: 

```text showLineNumbers
pipeline {
    agent any

    stages {
        stage('Run Jenkins Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__JENKINS_USER', variable: 'OCEAN__INTEGRATION__CONFIG__JENKINS_USER'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__JENKINS_HOST', variable: 'OCEAN__INTEGRATION__CONFIG__JENKINS_HOST'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="jenkins"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__JENKINS_USER=$OCEAN__INTEGRATION__CONFIG__JENKINS_USER \
                                -e OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN=$OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN \
                                -e OCEAN__INTEGRATION__CONFIG__JENKINS_HOST=$OCEAN__INTEGRATION__CONFIG__JENKINS_HOST \
                                -e OCEAN__PORT__CLIENT_ID=$OCEAN__PORT__CLIENT_ID \
                                -e OCEAN__PORT__CLIENT_SECRET=$OCEAN__PORT__CLIENT_SECRET \
                                $image_name

                            exit $?
                        ''')
                    }
                }
            }
        }
    }
}
```

  </TabItem>
<TabItem value="azure" label="Azure Devops">
<AzurePremise name="Jenkins" />

<DockerParameters />
<br/>

下面是 `jenkins-integration.yml` Pipelines 文件的示例: 

```yaml showLineNumbers
trigger:
- main

pool:
  vmImage: "ubuntu-latest"

variables:
  - group: port-ocean-credentials

steps:
- script: |
    # Set Docker image and run the container
    integration_type="jenkins"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
        -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
        -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
        -e OCEAN__INTEGRATION__CONFIG__JENKINS_USER=${OCEAN__INTEGRATION__CONFIG__JENKINS_USER} \
        -e OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN=${OCEAN__INTEGRATION__CONFIG__JENKINS_TOKEN} \
        -e OCEAN__INTEGRATION__CONFIG__JENKINS_HOST=${OCEAN__INTEGRATION__CONFIG__JENKINS_HOST} \
        -e OCEAN__PORT__CLIENT_ID=${OCEAN__PORT__CLIENT_ID} \
        -e OCEAN__PORT__CLIENT_SECRET=${OCEAN__PORT__CLIENT_SECRET} \
        $image_name

    exit $?
  displayName: 'Ingest Data into Port'
```

  </TabItem>

  </Tabs>
</TabItem>

</Tabs>

<AdvancedConfig/>

## 接收 Jenkins 对象

Jenkins 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 Jenkins 获取 "job "数据的过程: 

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: job
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .url | split("://")[1] | sub("^.*?/"; "") | gsub("%20"; "-") | gsub("/"; "-") | .[:-1]
          title: .fullName
          blueprint: '"jenkinsJob"'
          properties:
            jobName: .name
            url: .url
            jobStatus: '{"notbuilt": "created", "blue": "passing", "red": "failing"}[.color]'
            timestamp: .time
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 Jenkins API 事件中的现有字段和值进行选择、修改、连接、转换和其他操作。

### 配置结构

集成配置决定了将从 Jenkins 查询哪些资源，以及将在 Port 中创建哪些实体和属性。

* 集成配置的根密钥是 "资源 "密钥: 


  ```yaml showLineNumbers
  # highlight-next-line
  resources:
    - kind: job
      selector:
      ...
  ```


* 类型 "键是 Jenkins 对象的指定符: 


  ```yaml showLineNumbers
    resources:
      # highlight-next-line
      - kind: job
        selector:
        ...
  ```


* 通过 "选择器 "和 "查询 "键，您可以过滤哪些指定 "类型 "的对象将被录入软件目录: 


  ```yaml showLineNumbers
  resources:
    - kind: job
      # highlight-start
      selector:
        query: "true" # JQ boolean expression. If evaluated to false - this object will be skipped.
      # highlight-end
      port:
  ```


* Port"、"实体 "和 "映射 "键被用来将 Jenkins 对象字段映射到Port实体。要创建多个同类映射，可在 `resources` 数组中添加另一项；


  ```yaml showLineNumbers
  resources:
    - kind: job
      selector:
        query: "true"
      port:
        # highlight-start
        entity:
          mappings: # Mappings between one Jenkins object to a Port entity. Each value is a JQ query.
            identifier: .url | split("://")[1] | sub("^.*?/"; "") | gsub("%20"; "-") | gsub("/"; "-") | .[:-1]
            title: .fullName
            blueprint: '"jenkinsJob"'
            properties:
              jobName: .name
              url: .url
              jobStatus: '{"notbuilt": "created", "blue": "passing", "red": "failing"}[.color]'
              timestamp: .time
        # highlight-end
    - kind: job # In this instance job is mapped again with a different filter
      selector:
        query: '.name == "MyJobName"'
      port:
        entity:
          mappings: ...
  ```


:::tip 蓝图键 注意 `blueprint` 键的值 - 如果要使用硬编码字符串，需要用 2 组引号封装，例如使用一对单引号 (`'`)，然后再用一对双引号 (`"`): ::: 

#### 将数据输入Port

要使用[integration configuration](#configuration-structure) 引用 Jenkins 对象，可以按照以下步骤操作: 

1. 转到 DevPortal Builder 页面。
2. 选择要被 Jenkins 引用的蓝图。
3. 从菜单中选择**摄取数据**选项。
4. 在 CI/CD 类别下选择 Jenkins。
5. 根据需要修改[configuration](#configuration-structure) 。
6. 单击 `Resync`。

## 示例

蓝图和相关集成配置示例: 

### 工作

<details>
<summary>Job blueprint</summary>

```json showLineNumbers
{
  "identifier": "jenkinsJob",
  "description": "This blueprint represents a job in Jenkins",
  "title": "Jenkins Job",
  "icon": "Jenkins",
  "schema": {
      "properties": {
          "jobName": {
              "type": "string",
              "title": "Job Name"
          },
          "jobStatus": {
              "type": "string",
              "title": "Job Status",
              "enum": [
                  "created",
                  "unknown",
                  "passing",
                  "failing"
              ],
              "enumColors": {
                  "passing": "green",
                  "created": "darkGray",
                  "failing": "red",
                  "unknown": "orange"
              }
          },
          "timestamp": {
              "type": "string",
              "format": "date-time",
              "title": "Timestamp",
              "description": "Last updated timestamp of the job"
          },
          "url": {
              "type": "string",
              "title": "Project URL"
          },
          "parentJob": {
              "type": "object",
              "title": "Parent Job"
          }
      },
      "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: job
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .url | split("://")[1] | sub("^.*?/"; "") | gsub("%20"; "-") | gsub("/"; "-") | .[:-1]
          title: .fullName
          blueprint: '"jenkinsJob"'
          properties:
            jobName: .name
            url: .url
            jobStatus: '{"notbuilt": "created", "blue": "passing", "red": "failing"}[.color]'
            timestamp: .time
            parentJob: .__parentJob
```

</details>

#### 建设

<details>
<summary>Build blueprint</summary>

```yaml showLineNumbers
{
  "identifier": "jenkinsBuild",
  "description": "This blueprint represents a build event from Jenkins",
  "title": "Jenkins Build",
  "icon": "Jenkins",
  "schema": {
      "properties": {
          "buildStatus": {
              "type": "string",
              "title": "Build Status",
              "enum": [
                  "SUCCESS",
                  "FAILURE",
                  "UNSTABLE"
              ],
              "enumColors": {
                  "SUCCESS": "green",
                  "FAILURE": "red",
                  "UNSTABLE": "yellow"
              }
          },
          "buildUrl": {
              "type": "string",
              "title": "Build URL",
              "description": "URL to the build"
          },
          "timestamp": {
              "type": "string",
              "format": "date-time",
              "title": "Timestamp",
              "description": "Last updated timestamp of the build"
          },
          "buildDuration": {
              "type": "number",
              "title": "Build Duration",
              "description": "Duration of the build"
          }
      },
      "required": []
  },
  "mirrorProperties": {
      "previousBuildStatus": {
          "title": "Previous Build Status",
          "path": "previousBuild.buildStatus"
      }
  },
  "calculationProperties": {},
  "relations": {
      "parentJob": {
          "title": "Jenkins Job",
          "target": "jenkinsJob",
          "required": false,
          "many": false
      },
      "previousBuild": {
          "title": "Previous Build",
          "target": "jenkinsBuild",
          "required": false,
          "many": false
      }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: build
    selector:
      query: "true"
    port:
      entity:
        mappings:
          identifier: .url | split("://")[1] | sub("^.*?/"; "") | gsub("%20"; "-") | gsub("/"; "-") | .[:-1]
          title: .displayName
          blueprint: '"jenkinsBuild"'
          properties:
            buildStatus: .result
            buildUrl: .url
            buildDuration: .duration
            timestamp: '.timestamp / 1000 | todate'
          relations:
            parentJob: .url | split("://")[1] | sub("^.*?/"; "") | gsub("%20"; "-") | gsub("/"; "-") | .[:-1] | gsub("-[0-9]+$"; "")
            previousBuild: .previousBuild.url | split("://")[1] | sub("^.*?/"; "") | gsub("%20"; "-") | gsub("/"; "-") | .[:-1]
```

</details>

#### 用户

<details>
<summary>User blueprint</summary>

```json showLineNumbers
{
  "identifier": "jenkinsUser",
  "description": "This blueprint represents a jenkins user",
  "title": "Jenkins User",
  "icon": "Jenkins",
  "schema": {
      "properties": {
          "url": {
              "type": "string",
              "title": "URL",
              "format": "url"
          },
          "lastUpdateTime": {
              "type": "string",
              "format": "date-time",
              "title": "Last Update",
              "description": "Last updated timestamp of the user"
          }
      },
      "required": []
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {}
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
resources:
  - kind: user
  selector:
    query: "true"
  port:
    entity:
      mappings:
        identifier: .user.id
        title: .user.fullName
        blueprint: '"jenkinsUser"'
        properties:
          url: .user.absoluteUrl
          lastUpdateTime: if .lastChange then (.lastChange/1000) else now end | strftime("%Y-%m-%dT%H:%M:%SZ")
```

</details>