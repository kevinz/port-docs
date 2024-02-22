import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import HelmPrerequisites from "../templates/_ocean_helm_prerequisites_block.mdx"
import HelmParameters from "../templates/_ocean-advanced-parameters-helm.mdx"
import ResourceMapping from "../templates/_resource-mapping.mdx"
import DockerParameters from "./_docker-parameters.mdx"
import SupportedResources from "./_supported-resources.mdx"
import AdvancedConfig from '../../../generalTemplates/_ocean_advanced_configuration_note.md'
import SonarcloudAnalysisBlueprint from "/docs/build-your-software-catalog/sync-data-to-catalog/webhook/examples/resources/sonarqube/_example_sonarcloud_analysis_blueprint.mdx";
import SonarcloudAnalysisConfiguration from "/docs/build-your-software-catalog/sync-data-to-catalog/webhook/examples/resources/sonarqube/_example_sonarcloud_analysis_configuration.mdx";

# SonarQube

通过我们的 SonarQube 集成(由[Ocean](https://ocean.getport.io) 提供) ，您可以根据您的映射和定义，从 SonarQube 账户将 "项目"、"问题 "和 "分析 "导入 Port。

## 常见被引用情况

* 映射 SonarQube 组织环境中的 "项目"、"问题 "和 "分析"。
* 实时观察对象更改(创建/更新/删除)，并自动将更改应用到 Port 中的实体。
* 使用自助操作创建/删除 SonarQube 对象。

## 先决条件

<HelmPrerequisites />

## 安装

从以下安装方法中选择一种: 

<Tabs groupId="installation-methods" queryString="installation-methods">

<TabItem value="real-time-always-on" label="Real Time & Always On" default>

使用该安装选项意味着集成将能使用 webhook 实时更新 Port。

本表总结了安装时可用的参数，请在下面的脚本中按自己的需要进行设置，然后复制并在终端运行: 


| Parameter                                | Description                                                                                                                                                                                  | Example                             | Required |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- | -------- |
| `port.clientId`                          | Your port client id ([How to get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))                                 |                                     | ✅       |
| `port.clientSecret`                      | Your port client secret ([How to get the credentials](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#find-your-port-credentials))                             |                                     | ✅       |
| `integration.secrets.sonarApiToken`      | The [SonarQube API token](https://docs.sonarsource.com/sonarqube/9.8/user-guide/user-account/generating-and-using-tokens/#generating-a-token)                                                |                                     | ✅       |
| `integration.config.sonarOrganizationId` | The SonarQube [organization Key](https://docs.sonarsource.com/sonarcloud/appendices/project-information/#project-and-organization-keys) (Not required when using on-prem sonarqube instance) | myOrganization                      | ✅       |
| `integration.config.appHost`             | A URL bounded to the integration container that can be accessed by sonarqube. When used the integration will create webhooks on top of sonarqube to listen to any live changes in the data   | https://my-ocean-integration.com    | ❌       |
| `integration.config.sonarUrl`            | Required if using **On-Prem**, Your SonarQube instance URL                                                                                                                                   | https://my-sonar-cloud-instance.com | ❌       |


<HelmParameters />

<br/>
<Tabs groupId="deploy" queryString="deploy">

<TabItem value="helm" label="Helm" default>
To install the integration using Helm, run the following command:

```bash showLineNumbers
helm repo add --force-update port-labs https://port-labs.github.io/helm-charts
helm upgrade --install my-sonarqube-integration port-labs/port-ocean \
    --set port.clientId="PORT_CLIENT_ID"  \
    --set port.clientSecret="PORT_CLIENT_SECRET"  \
    --set initializePortResources=true  \
    --set scheduledResyncInterval=120  \
    --set integration.identifier="my-sonarqube-integration"  \
    --set integration.type="sonarqube"  \
    --set integration.eventListener.type="POLLING"  \
    --set integration.secrets.sonarApiToken="MY_API_TOKEN"  \
    --set integration.config.sonarOrganizationId="MY_ORG_KEY"
```

</TabItem>
<TabItem value="argocd" label="ArgoCD" default>
To install the integration using ArgoCD, follow these steps:

1. 在你的 git 仓库的 `argocd/my-ocean-sonarqube-integration` 中创建一个 `values.yaml` 文件，内容如下: 

:::note 请记住替换 `MY_ORG_KEY` 和 `MY_API_TOKEN` 的占位符。

:::

```yaml showLineNumbers
initializePortResources: true
scheduledResyncInterval: 120
integration:
  identifier: my-ocean-sonarqube-integration
  type: sonarqube
  eventListener:
    type: POLLING
  config:
  // highlight-next-line
    sonarOrganizationId: MY_ORG_KEY
  secrets:
  // highlight-next-line
    sonarApiToken: MY_API_TOKEN
```

<br/>

2.创建以下 "my-ocean-sonarqube-integration.yaml "配置清单，安装 "my-ocean-sonarqube-integration "ArgoCD应用程序: 

:::note 记住要替换 `YOUR_PORT_CLIENT_ID``YOUR_PORT_CLIENT_SECRET` 和 `YOUR_GIT_REPO_URL` 的占位符。

多种来源的 ArgoCD 文档可在[here](https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository) 上找到。

:::

<details>
  <summary>ArgoCD Application</summary>

```yaml showLineNumbers
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-ocean-sonarqube-integration
  namespace: argocd
spec:
  destination:
    namespace: my-ocean-sonarqube-integration
    server: https://kubernetes.default.svc
  project: default
  sources:
  - repoURL: 'https://port-labs.github.io/helm-charts/'
    chart: port-ocean
    targetRevision: 0.1.14
    helm:
      valueFiles:
      - $values/argocd/my-ocean-sonarqube-integration/values.yaml
      // highlight-start
      parameters:
        - name: port.clientId
          value: YOUR_PORT_CLIENT_ID
        - name: port.clientSecret
          value: YOUR_PORT_CLIENT_SECRET
  - repoURL: YOUR_GIT_REPO_URL
  // highlight-end
    targetRevision: main
    ref: values
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

</details>
<br/>

3.使用 `kubectl` 配置应用程序清单: 

```bash
kubectl apply -f my-ocean-sonarqube-integration.yaml
```

</TabItem>
</Tabs>

</TabItem>

<TabItem value="one-time" label="Scheduled">

  <Tabs groupId="cicd-method" queryString="cicd-method">
  <TabItem value="github" label="GitHub">
This workflow will run the SonarQube integration once and then exit, this is useful for **scheduled** ingestion of data.

:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用[Real Time & Always On](?installation-methods=real-time-always-on#installation) 安装选项

:::

确保配置以下[Github Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions) : 

<DockerParameters />

<br/>

下面是 `sonarqube-integration.yml` 工作流程文件的示例: 

```yaml showLineNumbers
name: SonarQube Exporter Workflow

# This workflow responsible for running SonarQube exporter.

on:
  workflow_dispatch:

jobs:
  run-integration:
    runs-on: ubuntu-latest

    steps:
      - name: Run Sonarqube Integration
        run: |
          # Set Docker image and run the container
          integration_type="sonarqube"
          version="latest"

          image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

          docker run -i --rm --platform=linux/amd64 \
          -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
          -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
          -e OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN=${{ secrets.OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN }} \
          -e OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID=${{ secrets.OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID }} \
          -e OCEAN__INTEGRATION__CONFIG__SONAR_URL=${{ secrets.OCEAN__INTEGRATION__CONFIG__SONAR_URL }} \
          -e OCEAN__PORT__CLIENT_ID=${{ secrets.OCEAN__PORT__CLIENT_ID }} \
          -e OCEAN__PORT__CLIENT_SECRET=${{ secrets.OCEAN__PORT__CLIENT_SECRET }} \
          $image_name

          exit $?
```

  </TabItem>
  <TabItem value="jenkins" label="Jenkins">
This pipeline will run the SonarQube integration once and then exit, this is useful for **scheduled** ingestion of data.

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
        stage('Run SonarQube Integration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN', variable: 'OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN'),
                        string(credentialsId: 'OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID', variable: 'OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_ID', variable: 'OCEAN__PORT__CLIENT_ID'),
                        string(credentialsId: 'OCEAN__PORT__CLIENT_SECRET', variable: 'OCEAN__PORT__CLIENT_SECRET'),
                    ]) {
                        sh('''
                            #Set Docker image and run the container
                            integration_type="sonarqube"
                            version="latest"
                            image_name="ghcr.io/port-labs/port-ocean-${integration_type}:${version}"
                            docker run -i --rm --platform=linux/amd64 \
                                -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
                                -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
                                -e OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN=$OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN \
                                -e OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID=$OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID \
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
This pipeline will run the SonarQube integration once and then exit, this is useful for **scheduled** ingestion of data.

:::tip 你的 Azure Devops 代理应该能够运行 docker 命令。

:::
:::warning 如果希望集成使用 webhooks 实时更新 Port，则应使用 安装选项。[Real Time & Always On](?installation-methods=real-time-always-on#installation) 

:::

确保使用[Azure Devops variable groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups?view=azure-devops&amp;tabs=yaml) 配置以下变量。将它们添加到名为 `port-ocean-credentials` 的变量组中: 

<DockerParameters />

<br/>

下面是 `sonar-integration.yml` Pipelines 文件的示例: 

```yaml showLineNumbers
trigger:
- main

pool:
  vmImage: "ubuntu-latest"

variables:
  - group: port-ocean-credentials

steps:
- script: |
    echo Add other tasks to build, test, and deploy your project.
    # Set Docker image and run the container
    integration_type="sonarqube"
    version="latest"

    image_name="ghcr.io/port-labs/port-ocean-$integration_type:$version"

    docker run -i --rm \
    -e OCEAN__EVENT_LISTENER='{"type":"ONCE"}' \
    -e OCEAN__INITIALIZE_PORT_RESOURCES=true \
    -e OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN=${OCEAN__INTEGRATION__CONFIG__SONAR_API_TOKEN} \
    -e OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID=${OCEAN__INTEGRATION__CONFIG__SONAR_ORGANIZATION_ID} \
    -e OCEAN__INTEGRATION__CONFIG__SONAR_URL=${OCEAN__INTEGRATION__CONFIG__SONAR_URL} \
    -e OCEAN__PORT__CLIENT_ID=${OCEAN__PORT__CLIENT_ID} \
    -e OCEAN__PORT__CLIENT_SECRET=${OCEAN__PORT__CLIENT_SECRET} \
    $image_name

    exit $?
  displayName: 'Ingest SonarQube Data into Port'
```

  </TabItem>
  </Tabs>

</TabItem>

</Tabs>

<AdvancedConfig/>

## 接收 SonarQube 对象

SonarQube 集成使用 YAML 配置来描述将数据加载到开发者门户的过程。

下面是配置中的一个示例片段，演示了从 SonarQube 获取 "项目 "数据的过程: 

```yaml showLineNumbers
resources:
  - kind: projects
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"sonarQubeProject"'
          identifier: .key
          title: .name
          properties:
            organization: .organization
            link: .__link
            lastAnalysisStatus: .__branch.status.qualityGateStatus
            lastAnalysisDate: .__branch.analysisDate
            numberOfBugs: .__measures[]? | select(.metric == "bugs") | .value
            numberOfCodeSmells: .__measures[]? | select(.metric == "code_smells") | .value
            numberOfVulnerabilities: .__measures[]? | select(.metric == "vulnerabilities") | .value
            numberOfHotSpots: .__measures[]? | select(.metric == "security_hotspots") | .value
            numberOfDuplications: .__measures[]? | select(.metric == "duplicated_files") | .value
            coverage: .__measures[]? | select(.metric == "coverage") | .value
            mainBranch: .__branch.name
            tags: .tags
```

该集成利用[JQ JSON processor](https://stedolan.github.io/jq/manual/) 对 SonarQube 的 API 事件中的现有字段和 Values 进行选择、修改、连接、转换和其他操作。

<ResourceMapping name="SonarQube" category="Code quality &amp; security providers" components={{
SupportedResources: SupportedResources
}}>

```yaml showLineNumbers
resources:
  - kind: projects
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"sonarQubeProject"'
          identifier: .key
          title: .name
          properties:
            organization: .organization
            link: .__link
            lastAnalysisStatus: .__branch.status.qualityGateStatus
            lastAnalysisDate: .__branch.analysisDate
            numberOfBugs: .__measures[]? | select(.metric == "bugs") | .value
            numberOfCodeSmells: .__measures[]? | select(.metric == "code_smells") | .value
            numberOfVulnerabilities: .__measures[]? | select(.metric == "vulnerabilities") | .value
            numberOfHotSpots: .__measures[]? | select(.metric == "security_hotspots") | .value
            numberOfDuplications: .__measures[]? | select(.metric == "duplicated_files") | .value
            coverage: .__measures[]? | select(.metric == "coverage") | .value
            mainBranch: .__branch.name
            tags: .tags
      # highlight-end
  - kind: projects # In this instance project is mapped again with a different filter
    selector:
      query: '.name == "MyProjectName"'
    port:
      entity:
        mappings: ...
```

</ ResourceMapping>

## 示例

蓝图和相关集成配置示例: 

### 项目

<details>
<summary>Projects blueprint</summary>

```json showLineNumbers
{
  "identifier": "sonarQubeProject",
  "title": "SonarQube Project",
  "icon": "sonarqube",
  "schema": {
    "properties": {
      "organization": {
        "type": "string",
        "title": "Organization",
        "icon": "TwoUsers"
      },
      "link": {
        "type": "string",
        "format": "url",
        "title": "Link",
        "icon": "Link"
      },
      "lastAnalysisDate": {
        "type": "string",
        "format": "date-time",
        "icon": "Clock",
        "title": "Last Analysis Date"
      },
      "numberOfBugs": {
        "type": "number",
        "title": "Number Of Bugs"
      },
      "numberOfCodeSmells": {
        "type": "number",
        "title": "Number Of CodeSmells"
      },
      "numberOfVulnerabilities": {
        "type": "number",
        "title": "Number Of Vulnerabilities"
      },
      "numberOfHotSpots": {
        "type": "number",
        "title": "Number Of HotSpots"
      },
      "numberOfDuplications": {
        "type": "number",
        "title": "Number Of Duplications"
      },
      "coverage": {
        "type": "number",
        "title": "Coverage"
      },
      "mainBranch": {
        "type": "string",
        "icon": "Git",
        "title": "Main Branch"
      },
      "tags": {
        "type": "array",
        "title": "Tags"
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
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: projects
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"sonarQubeProject"'
          identifier: .key
          title: .name
          properties:
            organization: .organization
            link: .__link
            lastAnalysisStatus: .__branch.status.qualityGateStatus
            lastAnalysisDate: .__branch.analysisDate
            numberOfBugs: .__measures[]? | select(.metric == "bugs") | .value
            numberOfCodeSmells: .__measures[]? | select(.metric == "code_smells") | .value
            numberOfVulnerabilities: .__measures[]? | select(.metric == "vulnerabilities") | .value
            numberOfHotSpots: .__measures[]? | select(.metric == "security_hotspots") | .value
            numberOfDuplications: .__measures[]? | select(.metric == "duplicated_files") | .value
            coverage: .__measures[]? | select(.metric == "coverage") | .value
            mainBranch: .__branch.name
            tags: .tags
```

</details>

### 问题

<details>
<summary>Issue blueprint</summary>

```json showLineNumbers
{
  "identifier": "sonarQubeIssue",
  "title": "SonarQube Issue",
  "icon": "sonarqube",
  "schema": {
    "properties": {
      "type": {
        "type": "string",
        "title": "Type",
        "enum": ["CODE_SMELL", "BUG", "VULNERABILITY"]
      },
      "severity": {
        "type": "string",
        "title": "Severity",
        "enum": ["MAJOR", "INFO", "MINOR", "CRITICAL", "BLOCKER"],
        "enumColors": {
          "MAJOR": "orange",
          "INFO": "green",
          "CRITICAL": "red",
          "BLOCKER": "red",
          "MINOR": "yellow"
        }
      },
      "link": {
        "type": "string",
        "format": "url",
        "icon": "Link",
        "title": "Link"
      },
      "status": {
        "type": "string",
        "title": "Status",
        "enum": ["OPEN", "CLOSED", "RESOLVED", "REOPENED", "CONFIRMED"]
      },
      "assignees": {
        "title": "Assignees",
        "type": "string",
        "icon": "TwoUsers"
      },
      "tags": {
        "type": "array",
        "title": "Tags"
      },
      "createdAt": {
        "type": "string",
        "format": "date-time",
        "title": "Created At"
      }
    }
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "sonarQubeProject": {
      "target": "sonarQubeProject",
      "required": false,
      "title": "SonarQube Project",
      "many": false
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: issues
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"sonarQubeIssue"'
          identifier: .key
          title: .message
          properties:
            type: .type
            severity: .severity
            link: .__link
            status: .status
            assignees: .assignee
            tags: .tags
            createdAt: .creationDate
          relations:
            sonarQubeProject: .project
```

</details>

#### 分析

<details>
<summary>Analysis blueprint</summary>

```json showLineNumbers
{
  "identifier": "sonarQubeAnalysis",
  "title": "SonarQube Analysis",
  "icon": "sonarqube",
  "schema": {
    "properties": {
      "branch": {
        "type": "string",
        "title": "Branch",
        "icon": "GitVersion"
      },
      "fixedIssues": {
        "type": "number",
        "title": "Fixed Issues"
      },
      "newIssues": {
        "type": "number",
        "title": "New Issues"
      },
      "coverage": {
        "title": "Coverage",
        "type": "number"
      },
      "duplications": {
        "type": "number",
        "title": "Duplications"
      },
      "createdAt": {
        "type": "string",
        "format": "date-time",
        "title": "Created At"
      }
    }
  },
  "mirrorProperties": {},
  "calculationProperties": {},
  "relations": {
    "sonarQubeProject": {
      "target": "sonarQubeProject",
      "required": false,
      "title": "SonarQube Project",
      "many": false
    }
  }
}
```

</details>

<details>
<summary>Integration configuration</summary>

```yaml showLineNumbers
createMissingRelatedEntities: true
deleteDependentEntities: true
resources:
  - kind: analysis
    selector:
      query: "true"
    port:
      entity:
        mappings:
          blueprint: '"sonarQubeAnalysis"'
          identifier: .analysisId
          title: .__commit.message // .analysisId
          properties:
            branch: .__branchName
            fixedIssues: .measures.violations_fixed
            newIssues: .measures.violations_added
            coverage: .measures.coverage_change
            duplications: .measures.duplicated_lines_density_change
            createdAt: .__analysisDate
          relations:
            sonarQubeProject: .__project
```

</details>

## 让我们来测试一下

本节包括为保证质量而扫描代码库时 SonarQube 的响应数据示例。 此外，还包括根据上一节提供的 Ocean 配置从重新同步事件中创建的实体。

### 有效载荷

下面是 SonarQube 提供的有效载荷结构示例: 

<details>
<summary> Project response data</summary>

```json showLineNumbers
{
  "organization": "peygis",
  "key": "PeyGis_Chatbot_For_Social_Media_Transaction",
  "name": "Chatbot_For_Social_Media_Transaction",
  "isFavorite": true,
  "tags": [],
  "visibility": "public",
  "eligibilityStatus": "COMPLETED",
  "eligible": true,
  "isNew": false,
  "analysisDateAllBranches": "2023-09-09T03:03:20+0200",
  "__measures": [
    {
      "metric": "bugs",
      "value": "6",
      "bestValue": false
    },
    {
      "metric": "code_smells",
      "value": "216",
      "bestValue": false
    },
    {
      "metric": "duplicated_files",
      "value": "2",
      "bestValue": false
    },
    {
      "metric": "vulnerabilities",
      "value": "1",
      "bestValue": false
    },
    {
      "metric": "security_hotspots",
      "value": "8",
      "bestValue": false
    }
  ],
  "__branch": {
    "name": "master",
    "isMain": true,
    "type": "LONG",
    "status": {
      "qualityGateStatus": "ERROR",
      "bugs": 6,
      "vulnerabilities": 1,
      "codeSmells": 216
    },
    "analysisDate": "2023-09-07T14:38:41+0200",
    "commit": {
      "sha": "5b01b6dcb200df0bfd1c66df65be30f9ea5423d8",
      "author": {
        "name": "Username",
        "login": "Username@github",
        "avatar": "9df2ac1caa70b0a67ff0561f7d0363e5"
      },
      "date": "2023-09-07T14:38:36+0200",
      "message": "Merge pull request #21 from PeyGis/test-sonar"
    }
  },
  "__link": "https://sonarcloud.io/project/overview?id=PeyGis_Chatbot_For_Social_Media_Transaction"
}
```

</details>

<details>
<summary> Issue response data</summary>

```json showLineNumbers
{
  "key": "AYhnRlhI0rLhE5EBPGHW",
  "rule": "xml:S1135",
  "severity": "INFO",
  "component": "PeyGis_Chatbot_For_Social_Media_Transaction:node_modules/json-schema/draft-zyp-json-schema-04.xml",
  "project": "PeyGis_Chatbot_For_Social_Media_Transaction",
  "line": 313,
  "hash": "8346d5371c3d1b0d1d57937c7b967090",
  "textRange": {
    "startLine": 313,
    "endLine": 313,
    "startOffset": 3,
    "endOffset": 56
  },
  "flows": [],
  "status": "OPEN",
  "message": "Complete the task associated to this \"TODO\" comment.",
  "effort": "0min",
  "debt": "0min",
  "assignee": "Username@github",
  "author": "email@gmail.com",
  "tags": [],
  "creationDate": "2018-04-06T02:44:46+0200",
  "updateDate": "2023-05-29T13:30:14+0200",
  "type": "CODE_SMELL",
  "organization": "peygis",
  "cleanCodeAttribute": "COMPLETE",
  "cleanCodeAttributeCategory": "INTENTIONAL",
  "impacts": [
    {
      "softwareQuality": "MAINTAINABILITY",
      "severity": "LOW"
    }
  ],
  "__link": "https://sonarcloud.io/project/issues?open=AYhnRlhI0rLhE5EBPGHW&id=PeyGis_Chatbot_For_Social_Media_Transaction"
}
```

</details>

<details>
<summary> Analysis response data</summary>

```json showLineNumbers
{
  "analysisId": "AYpvptJNv89mE9ClYP-q",
  "firstAnalysis": false,
  "measures": {
    "violations_added": "0",
    "violations_fixed": "0",
    "coverage_change": "0.0",
    "duplicated_lines_density_change": "0.0",
    "ncloc_change": "0"
  },
  "branch": {
    "analysisDate": "2023-09-07T12:38:41.279Z",
    "isMain": true,
    "name": "master",
    "commit": {
      "sha": "5b01b6dcb200df0bfd1c66df65be30f9ea5423d8",
      "author": {
        "avatar": "9df2ac1caa70b0a67ff0561f7d0363e5",
        "login": "Username@github",
        "name": "Username"
      },
      "date": "2023-09-07T12:38:36Z",
      "message": "Merge pull request #21 from PeyGis/test-sonar"
    },
    "type": "LONG",
    "status": {
      "qualityGateStatus": "ERROR"
    }
  },
  "__branchName": "master",
  "__analysisDate": "2023-09-07T12:38:41.279Z",
  "__commit": {
    "sha": "5b01b6dcb200df0bfd1c66df65be30f9ea5423d8",
    "author": {
      "avatar": "9df2ac1caa70b0a67ff0561f7d0363e5",
      "login": "Username@github",
      "name": "Username"
    },
    "date": "2023-09-07T12:38:36Z",
    "message": "Merge pull request #21 from PeyGis/test-sonar"
  },
  "__project": "PeyGis_Chatbot_For_Social_Media_Transaction"
}
```

</details>

#### 映射结果

结合样本有效载荷和 Ocean 配置，可生成以下 Port 实体: 

<details>
<summary> Project entity in Port</summary>

```json showLineNumbers
{
  "identifier": "PeyGis_Chatbot_For_Social_Media_Transaction",
  "title": "Chatbot_For_Social_Media_Transaction",
  "blueprint": "sonarQubeProject",
  "team": [],
  "properties": {
    "organization": "peygis",
    "link": "https://sonarcloud.io/project/overview?id=PeyGis_Chatbot_For_Social_Media_Transaction",
    "lastAnalysisDate": "2023-09-07T12:38:41.000Z",
    "numberOfBugs": 6,
    "numberOfCodeSmells": 216,
    "numberOfVulnerabilities": 1,
    "numberOfHotSpots": 8,
    "numberOfDuplications": 2,
    "mainBranch": "master",
    "tags": []
  },
  "relations": {},
  "icon": "sonarqube"
}
```

</details>

<details>
<summary> Issue entity in Port</summary>

```json showLineNumbers
{
  "identifier": "AYhnRlhI0rLhE5EBPGHW",
  "title": "Complete the task associated to this \"TODO\" comment.",
  "blueprint": "sonarQubeIssue",
  "team": [],
  "properties": {
    "type": "CODE_SMELL",
    "severity": "INFO",
    "link": "https://sonarcloud.io/project/issues?open=AYhnRlhI0rLhE5EBPGHW&id=PeyGis_Chatbot_For_Social_Media_Transaction",
    "status": "OPEN",
    "assignees": "Username@github",
    "tags": [],
    "createdAt": "2018-04-06T00:44:46.000Z"
  },
  "relations": {
    "sonarQubeProject": "PeyGis_Chatbot_For_Social_Media_Transaction"
  },
  "icon": "sonarqube"
}
```

</details>

<details>
<summary> Analysis entity in Port</summary>

```json showLineNumbers
{
  "identifier": "AYpvptJNv89mE9ClYP-q",
  "title": "Merge pull request #21 from PeyGis/test-sonar",
  "blueprint": "sonarQubeAnalysis",
  "team": [],
  "properties": {
    "branch": "master",
    "fixedIssues": 0,
    "newIssues": 0,
    "coverage": 0,
    "duplications": 0,
    "createdAt": "2023-09-07T12:38:41.279Z"
  },
  "relations": {
    "sonarQubeProject": "PeyGis_Chatbot_For_Social_Media_Transaction"
  },
  "icon": "sonarqube"
}
```

</details>

## 通过 webhook 进行替代安装

虽然上述海洋集成是推荐的安装方法，但您可能更喜欢使用 webhook 从 SonarQube 引用数据。 如果是这样，请使用以下说明: 

<details>

<summary><b>Webhook installation (click to expand)</b></summary>

在本示例中，您将在[SonarQube's SonarCloud](https://www.sonarsource.com/products/sonarcloud/) 和 Port 之间创建一个 webhook 集成，用于接收 SonarQube 代码质量 "分析 "实体。

<h2> Port configuration </h2>

创建以下蓝图定义: 

<details>
<summary>SonarQube analysis blueprint</summary>

<SonarcloudAnalysisBlueprint/>

</details>

创建以下 webhook 配置[using Port's UI](/build-your-software-catalog/sync-data-to-catalog/webhook/?operation=ui#configuring-webhook-endpoints) : 

<details>
<summary>SonarQube analysis webhook configuration</summary>

1. **基本信息** 选项卡 - 填写以下详细信息: 
    1.title: `SonarQube mapper`；
    2.标识符 : `sonarqube_mapper`；
    3.Description : `将 SonarQube 警报映射到 Port` 的 webhook 配置；
    4.图标 : `sonarqube`；
2. **集成配置**选项卡 - 填写以下JQ映射: 
   <SonarcloudAnalysisConfiguration/>
3.向下滚动到 **高级设置**，输入以下详细信息: 
    1. secret: `WEBHOOK_SECRET`；
    2.签名头名称:  `x-sonar-webhook-hmac-sha256`；
    3.签名算法: 从下拉选项中选择 `sha256`；
    4.点击页面底部的**保存**。
    记住将 `WEBHOOK_SECRET` 替换为您在 SonarCloud 中创建 webhook 时指定的真实secret。

</details>

<h2> Create a webhook in SonarCloud </h2>

1. 进入[SonarCloud](https://sonarcloud.io/projects) ，选择要配置 webhook 的项目；
2. 点击页面左下角的**管理**，然后选择**Webhooks**；
3. 点击**创建**
4. 输入以下详细信息: 
    1. `Name` - 被引用一个有意义的名称，如 Port Webhook；
    2. `URL` - 输入您在创建 webhook 配置后收到的 `url` 密钥的值；
    3. `Secret` - 输入您在创建 webhook 时指定的 secret 值；
5.单击页面底部的**创建**。

:::tip 为了查看 SonarQube webhook 中可用的不同有效载荷和事件、[look here](https://docs.sonarqube.org/latest/project-administration/webhooks/)

:::

完成！您运行的任何新分析(例如，对新 PR 或对 PR 的更改)都将触发 webhook 事件，SonarCloud 将把该事件发送到 Port 提供的 webhook URL。 Port 将根据映射解析事件，并相应地更新目录实体。

<h2> Let's Test It </h2>

本节包括当扫描代码库以保证质量时从 SonarQube 发送的 webhook 事件示例。 此外，还包括根据上一节提供的 webhook 配置从事件中创建的实体。

<h3> Payload </h3>

下面是扫描 SonarQube 资源库时发送到 webhook URL 的有效载荷结构示例: 

<details>
<summary> Webhook event payload</summary>

```json showLineNumbers
{
  "serverUrl": "https://sonarcloud.io",
  "taskId": "AYi_1w1fcGD_RU1S5-r_",
  "status": "SUCCESS",
  "analysedAt": "2023-06-15T16:15:05+0000",
  "revision": "575718d8287cd09630ff0ff9aa4bb8570ea4ef29",
  "changedAt": "2023-06-15T16:15:05+0000",
  "project": {
    "key": "Username_Test_Python_App",
    "name": "Test_Python_App",
    "url": "https://sonarcloud.io/dashboard?id=Username_Test_Python_App"
  },
  "branch": {
    "name": "master",
    "type": "LONG",
    "isMain": true,
    "url": "https://sonarcloud.io/dashboard?id=Username_Test_Python_App"
  },
  "qualityGate": {
    "name": "My Quality Gate",
    "status": "ERROR",
    "conditions": [
      {
        "metric": "code_smells",
        "operator": "GREATER_THAN",
        "value": "217",
        "status": "ERROR",
        "errorThreshold": "5"
      },
      {
        "metric": "ncloc",
        "operator": "GREATER_THAN",
        "value": "8435",
        "status": "ERROR",
        "errorThreshold": "20"
      },
      {
        "metric": "new_branch_coverage",
        "operator": "LESS_THAN",
        "status": "NO_VALUE",
        "errorThreshold": "1"
      },
      {
        "metric": "new_sqale_debt_ratio",
        "operator": "GREATER_THAN",
        "value": "1.0303030303030303",
        "status": "OK",
        "errorThreshold": "5"
      },
      {
        "metric": "new_violations",
        "operator": "GREATER_THAN",
        "value": "3",
        "status": "ERROR",
        "errorThreshold": "1"
      }
    ]
  },
  "properties": {}
}
```

</details>

<h3> Mapping Result </h3>

结合示例有效载荷和 webhook 配置可生成以下 Port 实体: 

```json showLineNumbers
{
  "identifier": "AYi_1w1fcGD_RU1S5-r_",
  "title": "Test_Python_App-AYi_1w1fcGD_RU1S5-r_",
  "blueprint": "sonarCloudAnalysis",
  "properties": {
    "serverUrl": "https://sonarcloud.io",
    "status": "SUCCESS",
    "projectName": "Test_Python_App",
    "projectUrl": "https://sonarcloud.io/dashboard?id=Username_Test_Python_App",
    "branchName": "master",
    "branchType": "LONG",
    "branchUrl": "https://sonarcloud.io/dashboard?id=Username_Test_Python_App",
    "qualityGateName": "My Quality Gate",
    "qualityGateStatus": "ERROR",
    "qualityGateConditions": [
      {
        "metric": "code_smells",
        "operator": "GREATER_THAN",
        "value": "217",
        "status": "ERROR",
        "errorThreshold": "5"
      },
      {
        "metric": "ncloc",
        "operator": "GREATER_THAN",
        "value": "8435",
        "status": "ERROR",
        "errorThreshold": "20"
      },
      {
        "metric": "new_branch_coverage",
        "operator": "LESS_THAN",
        "status": "NO_VALUE",
        "errorThreshold": "1"
      },
      {
        "metric": "new_sqale_debt_ratio",
        "operator": "GREATER_THAN",
        "value": "1.0303030303030303",
        "status": "OK",
        "errorThreshold": "5"
      },
      {
        "metric": "new_violations",
        "operator": "GREATER_THAN",
        "value": "3",
        "status": "ERROR",
        "errorThreshold": "1"
      }
    ]
  },
  "relations": {}
}
```

</details>