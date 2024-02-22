---
title: Okta

sidebar_position: 1
description: 将 Okta 与 Port 集成

---

# 如何配置 Okta

请按照本分步指南配置 Port 和 Okta 之间的集成。

:::info 为了完成该流程，您需要联系我们，我们需要提供的确切信息以及 Port 要求您提供的信息已在本文档中列出。

:::

## Port-Okta 整合效益

* 通过 Okta 应用程序连接到 Port 应用程序。
* 用户登录后，Okta 团队将自动与 Port 同步。
* 根据您的 Okta 小组在 Port 上设置细粒度权限。

## 如何为 Port 配置 Okta 应用程序集成

### 步骤 #1: 创建一个新的 Okta 应用程序

1. 在管理控制台中，转到应用程序 -> 应用程序。
2. 单击 "创建应用程序集成"。

![Okta new application wizard](../../../static/img/sso/okta/OktaCreateApp.png)

3.创建 OIDC 应用程序集成。在向导中选择 **OIDC - OpenID Connect**。

![Okta new application OIDC integration](../../../static/img/sso/okta/OktaCreateAppIntegration.png)

4.选择 ** 单页应用程序** 作为应用程序类型。

![Okta new application type](../../../static/img/sso/okta/OktaSetAppType.png)

单击 "下一步"。

### 第 2 步: 配置您的 Okta 应用程序

在 "常规设置 "下: 

1. 选择 "应用程序集成名称"(将出现在您的 Okta 应用程序上的特定名称)。
2. 添加应用程序徽标(可选)。
    ![Port's logo](../../../static/img/sso/general-assets/PortLogo.png)
3.在 "授权类型 "下标记所有选项。
4.在 `登录重定向 URI` 下设置: https://auth.getport.io/login/callback"。
    - 登录重定向 URI 是 Okta 为登录请求发送身份验证响应和 ID 标记的位置。
5.删除签出重定向 URI。
6.在 "分配 "下: 设置 "允许您组织中的每个人访问"。
    ![Okta app settings](../../../static/img/sso/okta/AppIntegrationSettings.png)
    ![Okta app settings assignments](../../../static/img/sso/okta/AppSettingsAssignments.png)

### 第三步: 配置 OIDC 设置

点击Okta管理界面右上角的用户邮件，悬停在okta域名上(格式为`{YOUR_COMPANY_NAME}.okta.com`)，然后点击 `复制到剪贴板`，即可获取您的 "Okta域名": 

![Get Okta domain](../../../static/img/sso/okta/OktaGetDomain.png)

在 "常规 "选项卡下: 

1. 复制 "客户 ID"，连同上一步的 "Okta 域名 "一起发送到 Port(在松弛频道/内部通信中)。
    ![Okta app settings](../../../static/img/sso/okta/OktaAppSettingsPage.png)
2.单击 "常规设置 "选项卡上的 "编辑 "按钮。
    2.1 将`登录由`选项设为`Okta 或应用程序`:![Okta app settings](../../../static/img/sso/okta/OktaAppLoginInitiation.png)2.2 选中`应用程序可见性`中的所有选项:![Okta app settings](../../../static/img/sso/okta/OktaAppVisibilitySettings.png)2.3 选中`登录流`为`重定向到应用程序以启动登录(OIDC 兼容)`2.4 在`启动登录 URI` 下粘贴以下 URI: 


   ```text showLineNumbers
   https://auth.getport.io/authorize?response_type=token&client_id=96IeqL36Q0UIBxIfV1oqOkDWU6UslfDj&connection={CONNECTION_NAME}&redirect_uri=https%3A%2F%2Fapp.getport.io
   ```


:::note 我们将提供您的 `{CONNECTION_NAME}` (请在 Slack/Intercom 上联系我们)。 :::: 

![Okta app settings login flow](../../../static/img/sso/okta/OktaAppLoginflowSettings.png)

2.5 单击 "保存"，大功告成！现在您将在 Okta 面板上看到 Port 应用程序。

![Okta dashboard with Port app](../../../static/img/sso/okta/OktaDashboard.png)

---

## 如何允许将 Okta 组拉入 Port

:::note 该阶段是**可选的**，只有当您希望将所有 Okta 组固有地接入 Port 时才需要该阶段。

**收益: ** 管理 Port 上的权限和用户访问。 **成果: ** 对于每个登录的用户，我们将根据您在下面设置中的定义，自动获取其关联的 Okta 组。

:::

要允许在 Port 中自动支持 Okta 组，请按照以下步骤操作: 

1. 在 "应用程序 "页面下，选择 Port App 并转到 "登录 "选项卡: 
    ![Okta application sign-on settings](../../../static/img/sso/okta/OktaAppSignOnSettings.png)
2.在 "OpenID 连接令牌 "下单击 "编辑": 
    ![Okta application connect id token](../../../static/img/sso/okta/OktaAppConnectToken.png)
3.添加`Groups claim type`并选择选项`filter`，然后: 
    3.1 Values = `groups`3.2 根据需要选择所需的 regex 短语:::注
    要导入所有组，请插入带`.*`值的`匹配 regex`。
    :::![Okta application set group claims](../../../static/img/sso/okta/OktaAppSetGroupClaims.png)3.3 单击 `保存`。
