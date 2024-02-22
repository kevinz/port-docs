---
sidebar_position: 6
title: 从后台导入数据

---

import PortTooltip from "/src/components/tooltip/tooltip.jsx"

# 从后台导入

Port 提供了一个简单的脚本，可用于使用 Backstage API 将数据从 Backstage 实例导入 Port。 该脚本会根据 Backstage 实例中的数据初始化 Port 中的<PortTooltip id="blueprint">蓝图</PortTooltip>和<PortTooltip id="entity">实体</PortTooltip>。

导入脚本的源代码是公开的，可从[**GitHub**](https://github.com/port-labs/backstage-import.git) 上获取。

:::tip  先决条件

* 虽然本指南并不强制要求填写，但我们建议您在继续阅读之前填写[onboarding process](/quickstart) 。
* [Docker](https://docs.docker.com/engine/install/).
* 后台实例。

:::

## 运行脚本

1. 克隆项目存储库: 

```bash showLineNumbers
git clone https://github.com/port-labs/backstage-import.git
```

2.在克隆的版本库中，创建包含以下值的 `.env` 文件: 

```bash showLineNumbers
BACKSTAGE_URL=<YOUR BACKSTAGE URL i.e https://demo.backstage.io>
PORT_CLIENT_ID=<YOUR PORT CLIENT ID>
PORT_CLIENT_SECRET=<YOUR PORT CLIENT SECRET>
```

3.运行导入脚本: 

```bash showLineNumbers
./import.sh
```

完成！脚本完成后，您将在 Port 中看到新的<PortTooltip id="blueprint">蓝图</PortTooltip>，以及与后台实例中的数据相匹配的<PortTooltip id="entity">实体</PortTooltip>。

## 下一步

#### 使用 Gitops 管理资源

将所有数据导入 Port 后，您可能会希望开始通过 Git 中的规范文件来管理这些数据。

1. 请访问[Port account](https://app.getport.io/) 。
2. 单击右上角的"... "图标，然后选择 "导出数据"。
3. 选择要导出的蓝图，选择 "GitOps "格式，然后点击 "导出"。

这将把所有规范文件下载到本地计算机，然后将它们推送到 GitOps 仓库并开始管理。

要了解更多关于被用于 GitOps 管理 Port 实体的信息，请参阅[GitHub](/build-your-software-catalog/sync-data-to-catalog/git/github/gitops/gitops.md) 和[Bitbucket](/build-your-software-catalog/sync-data-to-catalog/git/bitbucket/gitops/gitops.md) GitOps 页面。