---
sidebar_position: 1
title: 定义您的数据模型

sidebar_label: 定义您的数据模型

---

# 定义数据模型

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/E6pw_YZsjHM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

定义软件目录的数据模型与定义数据库结构类似。 您也可以使用 Port 公司预先定义的通用数据模型。

建立数据模型有两个主要构件: 

* 蓝图 - 代表**实体类型**。蓝图包含您希望在软件目录中表示的实体的模式。例如: 微服务和环境蓝图。
* 关系 - 允许您定义蓝图之间的依赖关系模型。关系 "将 Port 的目录转化为面向图形的目录。

<br></br>
<br></br>

![Basic blueprints relation](../../../static/img/blueprints-relation-basic-example.png)

## 通用数据模型

* 软件开发生命周期(SDLC)
    - 通用蓝图: 服务、部署、环境、package、Pipelines、Pull Request 等。
* 云
    - 常见蓝图: Lambda、EKS、Kafka、S3、Postgres 等。
* Kubernetes 和 Argo 目录
    - 常见蓝图: 集群、CronJob、Namespace、Pods、副本集、Istio、ArgoApp、ArgoProject 等。
* C4(后台风格)
    - 常用蓝图: 系统、域、资源、组件、组。
* 多云架构
    - 通用蓝图: 云供应商、区域、账户等。
* 单租户
    - 常见蓝图: 应用程序、客户、运行中的应用程序等。

在[live demo](https://demo.getport.io/dev-portal) 这个示例中，我们可以看到一个使用 Blueprints &amp; Relations 的综合数据模型示例。 🎬

## 🧱 步骤 1 -[Setup blueprints](./setup-blueprint/setup-blueprint.md)

## 🔀 步骤 2 -[Relate blueprints](./relate-blueprints/relate-blueprints.md)