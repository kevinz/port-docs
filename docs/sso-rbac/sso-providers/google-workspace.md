---
title: Google Workspace"(谷歌工作空间

sidebar_position: 5
description: 将 Google Workspace 与 Port 整合在一起

---

import Image from "@theme/IdealImage";
import webAndMobile from "../../../static/img/sso/google-workspace/webAndMobile.png"
import addSamlApp from "../../../static/img/sso/google-workspace/addSamlApp.png"
import appNameAndIcon from "../../../static/img/sso/google-workspace/appNameAndIcon.png"
import urlAndCert from "../../../static/img/sso/google-workspace/urlAndCert.png"
import ACSandEntity from "../../../static/img/sso/google-workspace/ACSandEntity.png"
import SSOandCert from "../../../static/img/sso/google-workspace/SSOandCert.png"
import attributeMapping from "../../../static/img/sso/google-workspace/attributeMapping.png"
import userAccessInApp from "../../../static/img/sso/google-workspace/userAccessInApp.png"
import turnAccessOn from "../../../static/img/sso/google-workspace/turnAccessOn.png"
import loginUsingApp from "../../../static/img/sso/google-workspace/loginUsingApp.png"
import acsURLandEntityID from "../../../static/img/sso/google-workspace/acsURLandEntityID.png"

# 如何配置 Google Workspace

请按照本分步指南配置 Port 和 Google Workspace 之间的集成。

:::info 为了完成该流程，您需要联系我们，以获得您需要的信息，以及 Port 需要您提供的信息。 所有内容详述如下。

:::

## Port-Google Workspace 集成优势

* 通过 Google Workspace 应用程序连接到 Port 应用程序。
* 用户登录后，Google Workspace 团队将自动与 Port 同步。
* 根据 Google Workspace 小组在 Port 上设置细粒度权限。

## 创建 Google Workspace 应用程序

1. 在[Google Admin Console](https://admin.google.com/) 侧边栏菜单中，导航至**应用程序** -> **网络和移动应用程序**: 

<center>

<Image img={webAndMobile} style={{ width: 300 }} />

</center>

<br/>

2.点击 "添加应用程序"，然后点击 "添加自定义 SAML 应用程序": 

<center>

<Image img={addSamlApp} style={{ width: 300 }} />

</center>

<br/>

3.定义 Port 应用程序的初始设置: 
    1. 应用程序名称为 Port 应用程序输入您选择的名称，如 `Port`。
    2. 添加 "应用程序图标"(可选): 
    <details>
    <summary>Port Logo</summary>![Port's logo](../../../static/img/sso/general-assets/PortIcon.png)
    </details>
    3. 按`继续`

<center>

<Image img={appNameAndIcon} style={{ width: 600 }} />

</center>

<br/>

4.注意以下几点: 
    1.您的 `SSO URL`；
    2.您的 "证书

<center>

<Image img={SSOandCert} style={{ width: 550 }} />

</center>

<br/>

把这些传给 Port。<br/>

按 "继续"。

5.配置新应用程序，如下图所示: 

* `ACS url` - `https://auth.getport.io/login/callback?connection={connection_name}`
* 实体 ID - `urn:auth0:port-prod:{CONNECTION_NAME}`

:::note 我们将提供您的 `{CONNECTION_NAME}` (请在 Slack/Intercom 上联系我们)。

:::

按 "继续

<center>

<Image img={acsURLandEntityID} style={{ width: 600 }} />

</center>

<br/>

6.创建以下映射: 

谷歌目录属性_: 

* **`主要电子邮件`** -> `电子邮件
* **`First name`** -> `name`

_Google成员资格_(可选):  此映射仅在希望将群组传递给 Port 时才相关。

* **`Google群组`**(列表) -> `groups`.

按 "完成 "键

<center>

<Image img={attributeMapping} style={{ width: 550 }} />

</center>

<br/>

7.指定应用程序的权限: 

创建应用程序后，您需要设置谁可以访问此应用程序的权限。

导航至新应用程序页面，点击 **用户访问**: 

<center>

<Image img={userAccessInApp} style={{ width: 550 }} />

</center>

<br/>

然后从左侧菜单中选择为 "所有人"、"组 "或 "组织单位 "启用应用程序。

确保您要启用应用程序的任何选项都选中了 "ON "复选框: 

<center>

<Image img={turnAccessOn} style={{ width: 550 }} />

</center>

<br/>

7.使用新的 Google 应用程序登录: 

<center>

<Image img={loginUsingApp} style={{ width: 250 }} />

</center>
