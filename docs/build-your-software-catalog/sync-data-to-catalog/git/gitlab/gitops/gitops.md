---
sidebar_position: 3
---

import Tabs from "@theme/Tabs"
import TabItem from "@theme/TabItem"
import PortYmlStructure from '../../_port_yml_gitops_structure_template.md'
import BasicFileProperties from '../../_basic_file_properties_template.md'
import RelativeFileProperties from '../../_relative_file_properties_template.md'
import GitOpsPushEvent from '../../_git_gitops_push_events_explanation.mdx'

# GitOps

Port 的 GitLab 集成功能可让您采用 GitOps 方法管理 Port 实体，使您的代码项目成为您要管理的各种基础架构资产的真实来源。

## 💡 GitLab GitOps 常见用例

* 将 GitLab 作为**microservices**、**projects**、**packages**、**libraries**和其他软件目录资产的真实来源；
* 允许开发人员通过更新其 Git 项目中的文件来保持目录的最新状态；
* 创建一种标准化的方式来记录企业中的软件目录资产；

## 使用 GitOps 管理实体

要使用 GitOps 管理实体，需要在项目的**默认分支**(通常为`main`)中添加一个`port.yml`文件。

`port.yml` 文件可以指定一个或多个 Port 实体，这些实体将被摄取到 Port 中，对 `port.yml` 文件所做的任何更改也将反映在 Port 内部。

该配置可将 GitLab 项目转化为软件目录的真实源。

### GitOps `port.yml` 文件

port.yml` 文件用于指定使用 GitOps 管理的 Port 实体，并从 Git 项目中引用其数据。

下面是有效 `port.yml` 文件的示例: 

<Tabs groupId="format">

<TabItem value="single" label="Single entity">

```yaml showLineNumbers
identifier: myEntity
title: My Entity
blueprint: myBlueprint
properties:
  myStringProp: myValue
  myNumberProp: 5
  myUrlProp: https://example.com
relations:
  mySingleRelation: myTargetEntity
  myManyRelation:
    - myTargetEntity1
    - myTargetEntity2
```

</TabItem>

<TabItem value="multiple" label="Multiple entities">

```yaml showLineNumbers
- identifier: myEntity1
  title: My Entity1
  blueprint: myFirstBlueprint
  properties:
    myStringProp: myValue
    myNumberProp: 5
    myUrlProp: https://example.com
  relations:
    mySingleRelation: myTargetEntity
    myManyRelation:
      - myTargetEntity1
      - myTargetEntity2
- identifier: myEntity
  title: My Entity2
  blueprint: mySecondBlueprint
  properties:
    myStringProp: myValue
    myNumberProp: 5
    myUrlProp: https://example.com
```

</TabItem>

</Tabs>

由于两种有效的 `port.yml` 格式都遵循相同的结构，下文将根据单一实体示例来解释格式。

###`port.yml` 结构

<PortYmlStructure/>

### 接收项目文件内容

<BasicFileProperties/>

#### 使用相对路径

<RelativeFileProperties/>

## 高级

有关高级用例和配置，请参阅[advanced](../advanced.md) 页面。