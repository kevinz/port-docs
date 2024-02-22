---
sidebar_position: 1
title: 建立您的软件目录

sidebar_label: 🏗️ 建立您的软件目录

---

# 🏗️ 建立您的软件目录

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/-ir4YG-XMPU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

Port 的软件目录是软件、环境、资源等的中央元数据存储。 它的构建模块是蓝图和关系，您可以使用它们来构建一个反映您的确切数据模型的目录。 您也可以使用 Port 的一种通用数据模型来构建目录。

## 步骤 1 - 定义数据模型

这一步首先要确定主要实体(大多数人从服务开始)要包含的信息，并定义相关的蓝图(在本例中是服务蓝图)。

接下来的步骤将是定义更多蓝图，如云资源或集群、它应包括的数据以及它与其他实体的关系。 例如，如果你想管理软件目录中的软件包，你将定义一个软件包蓝图。

通过以这种方式定义数据模型的结构，您可以确保软件目录准确地反映您的工程组织，从而为您提供所需的集中式、有主见的软件目录。

![Basic blueprints relation](../../static/img/software-catalog/blueprint/exampleBlueprintsAndRelationsLayout.png)

查看如何-->[Define your data model](./define-your-data-model/define-your-data-model.md)

## 🔄 第 2 步 - 将数据导入目录

设置 Port 软件目录的下一步是将数据导入目录，这包括使用 Port 的集成和 API 将数据导入目录，以便用相关数据填充蓝图。

Provider 的软件目录提供了集成功能，使存在于各种工具和资源库中的数据浮出水面，从而在您的开发运营架构中创建了一个中央元数据存储，便于将所有相关信息集中到一个地方。

通过将数据摄入软件目录，您可以确保对软件、基础架构、云资源、CI/CD 等有一个全面、最新的了解。

查看如何-->[Ingest data to the software catalog](./sync-data-to-catalog/sync-data-to-catalog.md)

![Port integrations](../../static/img/software-catalog/integrations.png)