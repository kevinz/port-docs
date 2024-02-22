---
sidebar_position: 14
title: 故障排除

sidebar_label: ❓ 疑难解答

---

# 疑难解答

本页包含用户在使用 Port 时遇到的常见问题的答案，按主题分类。

## General

#### 使用 Port 建立内部开发人员门户网站需要专家吗？

<details>
<summary><b>Answer (click to expand)</b></summary>

Port 的设计目的是让您在几分钟内建立一个开发人员门户，快速定义数据模型，然后将有关软件和资源的数据输入其中。

我们相信 "自带数据模型"，因为每个组织希望如何建立 Port 和软件模型都不尽相同。我们的文档和[other resources](/resources) 可以帮助您开始使用。

如果您想了解 Port 是否适合您，可以通过[in-person demo](https://www.getport.io/demo-request) 与我们联系，我们将很乐意为您建立一个适合您的门户网站。

</details>

---

#### 为什么不是后台？

<details>
<summary><b>Answer (click to expand)</b></summary>

Spotify 的 Backstage 非常准确地认识到了端到端开发环境的简化需求。 它还很灵活，可以让您根据自己的数据模型构建软件目录。 不过，它需要编码、实施人员和领域专业知识。 您还需要在部署、配置和更新方面进行投资。您可以阅读 Port 和 Backstage 的详细比较[here](https://www.getport.io/compare/backstage-vs-port) 。

</details>

---

#### Port 真的免费吗？

<details>
<summary><b>Answer (click to expand)</b></summary>

Port 最多可免费使用 15 个用户，您可以查看我们的[pricing page](https://www.getport.io/pricing) 了解更多信息。使用免费版 Port，您可以建立一个先进的、功能齐全的内部开发人员门户。

免费版包含 Port 中的所有功能，但出于合理使用的考虑，除了 SSO 和对软件目录实体数量的一定限制(最多 10,000 个)。

如果您正在评估 Port，它可以为您提供所需的一切，如果您在一定时期内需要 SSO，请联系我们。

</details>

---

## 组织

#### 如何为我的组织设置 SSO？

<details>
<summary><b>Answer (click to expand)</b></summary>

1. 在 SSO 面板中设置应用程序。您可以在[here](https://docs.getport.io/sso-rbac/sso-providers/) 找到每个受支持 Provider 的文档。
2. 请与我们联系并提供所需凭证，以便完成设置。
3. 完成设置后，Port 将为您提供 `CONNECTION_NAME`。请返回文档并在需要时进行替换。

</details>

---

#### 如何排除 SSO 连接的故障？

<details>
<summary><b>Answer (click to expand)</b></summary>

1. 确保用户拥有使用应用程序的权限。
2. 查看错误的 URL，有时它们会嵌入错误中。例如，请查看以下 URL: 

```
https://app.getport.io/?error=access_denied&error_description=access_denied%20(User%20is%20not%20assigned%20to%20the%20client%20application.)&state=*********
```

在 "error_description "后面，您可以看到 "User%20is%20not%20assigned%20to%20the%20client%20application"。 在这种情况下，用户没有分配给 SSO 应用程序，因此无法通过它访问 Port。

3.确保您使用的是 Provider 提供的正确的 `CONNECTION_NAME`，并根据我们的设置文档正确设置应用程序。

</details>

---

#### 为什么我不能邀请其他会员访问我的门户网站？

<details>
<summary><b>Answer (click to expand)</b></summary>

使用免费层级时，Port 只允许您连接到一个组织。 如果您的同事在其他组织，您将无法邀请他/她。 请通过[Slack](https://www.getport.io/community) 或 Intercom 联系我们，我们将帮助您解决问题。

</details>

---

## action

#### 在 Port 中触发一个操作后，为什么会停留在 "进行中"，而 Git Providers 中什么也没发生？

<details>
<summary><b>Answer (click to expand)</b></summary>

请确保

1. 操作后端设置正确。这包括组织/组名称、存储库和工作流程文件名称。
2. 对于 Gitlab，请确保[Port execution agent](https://docs.getport.io/create-self-service-experiences/setup-backend/gitlab-pipeline/Installation#installing-the-agent) 已正确安装。触发动作时，可以查看代理的日志，了解触发了哪些 URL。

</details>

---

## 目录

#### 我的一个目录页面没有显示所有实体，或者缺少某些数据，这是为什么？

<details>
<summary><b>Answer (click to expand)</b></summary>

1. 检查右上角的表格筛选器。确保没有应用筛选器，或没有隐藏属性。
2. 有时用户会应用[initial filters](https://docs.getport.io/customize-pages-dashboards-and-plugins/page/catalog-page/#initial-filters) 来提高目录页面的加载速度。确保您丢失的实体没有被过滤掉。

</details>

---

## 安全

#### Port 有哪些安全措施？

<details>
<summary><b>Answer (click to expand)</b></summary>

我们在 Port 的设计上花了很多心思，以确保其安全性。 因此，它不存储secret或凭证，也不需要 IP 白名单。

您可以查看我们安全的只推送架构[here](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog) 。

安全和隐私是 Port 的重中之重。 我们采用行业标准的加密协议，数据在静态和传输过程中都经过加密，客户端之间完全隔离，并对数据访问进行日志记录和审计。 Port 符合 SOC2 和 ISO/IEC 27001:2022 标准，并定期进行五重测试、产品安全性和合规性审查。

您可以在我们的安全[page](https://www.getport.io/security) 中找到有关 Port 的**安全政策**的完整报道。

</details>

---