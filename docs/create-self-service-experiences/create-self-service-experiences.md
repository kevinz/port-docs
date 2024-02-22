---
title: 创建自助操作

sidebar_label: ⚡️ 创建自助服务操作

---

# ⚡️ 创建自助行动

<center>

<iframe width="60%" height="400" src="https://www.youtube.com/embed/KHuGBQlErWo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen allow="fullscreen;"></iframe>

</center>

通过允许开发人员自由运行和使用自助服务操作(如搭建服务脚手架或调配云资源)来提高开发人员的工作效率。 开发人员自助服务可带来一致性和可重复性，并确保开发人员做正确的事情，因为它直观清晰，所有这些都有人工审批或消费策略等护栏，以符合组织标准。

1. **Not Opinionated** - 使用低代码用户界面组件设置任何自助服务操作用户界面；
2. **同步**；
3. **Leverages existing infrastructure** and automations as the backend of the defined action；
4. **与您的基础架构和架构**松耦合；
5. **有状态** - 每个调用的操作都会通过添加/修改/删除一个或多个实体来影响软件目录；
6. **设计安全**--通过使用基于事件的模型，不需要敏感基础架构的密钥，所有操作都经过审计，内部嵌入了人工审批和 TTL 等防护措施。

## 💡 常见的自助服务操作

* **脚手架**一项新服务；
* **打开**个 terraform PR，创建云账户；
* **启动**个 jupyter 笔记本；
* **创建**云资源；
* **脚手架**新的内部 packages；
* **提供**临时 DevEnv；
* **重新部署**镜像标签；
* **回滚**一个正在运行的服务；
* **更改***副本数量。

在[live demo](https://demo.getport.io/self-serve) 这个示例中，我们可以看到自助服务体验的例子。

## 它是如何工作的？

1. 用户在 Port 的用户界面上**执行操作；
2.  **将包含用户输入和相关操作元数据的有效载荷**发送到您的基础架构；
3.  **任务被触发**，**用户不断收到有关任务进展的提示**；
4. 动作运行后，您可以使用 Port 的 API 更新 Port 的状态，并提供日志等信息和由此产生的处理程序的链接。

<center>

![](../../static/img/self-service-actions/selfserviceHLarch.png)

</center>

## 从 Port 启用操作的步骤

在 Port 中创建自助服务体验与传统的前端-后端模式非常相似。 Port 为您提供无代码组件，为用户创建您想要的体验，并与您提供的现有工作流和自动化集成。

### 步骤 1 - 设置行动

Port 支持各种输入类型，包括构建带有条件和步骤的向导，以最适合您希望用户获得的体验。

<center>

![](../../static/img/self-service-actions/setup_ui.png)

</center>

### 第 2 步 - 设置后端

设置表单提交时负责处理操作的逻辑。

后端逻辑是你的，所以你需要它做什么，它就能做什么: 

* 为注入 Values 的 IaC 文件创建拉取请求；
* 触发 Github 工作流或自定义 Python/Bash 脚本；
* 调用内部 API；
* 等等。

Port 支持多种不同的操作后端，提供安全、合规的架构。

作为后端及其逻辑实现的一部分，您能够通过发送 API 请求或摄取与已执行操作相关联的新数据(例如，脚手架流程结束后添加新的服务实体)来保持软件目录的更新

<center>

![](../../static/img/self-service-actions/backend-integrations.png)

</center>

### 第 3 步 - 反映行动进展

Port 可让您以丰富的方式向用户更新操作进度的当前状态，包括成功/进行中/失败状态、实时日志、ChatOps 通知、友好的指示性错误信息等。

### 可选步骤 - ✋🏼 设置护栏

Port 支持多种方式为消耗的操作添加手动审批、策略和 TTL，以确保在考虑组织标准的前提下允许特定操作。