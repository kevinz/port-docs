---

title: "Onelogin
sidebar_position: 3
description: 将 Onelogin 与 Port 整合

---

# 如何配置 Onelogin

请按照本分步指南配置 Port 和 Onelogin 之间的集成。

:::info 为了完成该流程，您需要联系 Port，以交付和接收信息，详情请参见以下指南。

:::

## Port-Onelogin 集成优势

* 通过 Onelogin 应用程序连接到 Port 应用程序；
* 用户登录后，您的 Onelogin 角色将自动与 Port 同步；
* 根据 Onelogin 角色在 Port 上设置细粒度权限。

## 如何为 Port 配置 Onelogin 应用程序集成

### 第 1 步: 创建新的 Onelogin 应用程序

1. 在管理控制台中，转到应用程序 -> 应用程序。
2. 单击 "添加应用程序"。

![Onelogin new application wizard](../../../static/img/sso/onelogin/OneloginCreateApp.png)

3.在搜索框中输入 **OpenID Connect**，然后选择 "OpenId Connect (OIDC)": 

![Onelogin new application OIDC](../../../static/img/sso/onelogin/OneloginSelectOidcFromSearch.png)

4.定义 Port 应用程序的初始设置: 
    1. 显示名称": 为 Port 应用程序插入一个您选择的名称，如 `Port`。
    2.添加矩形和正方形图标(可选): 
    ![Port's logo](../../../static/img/sso/general-assets/PortLogo.png) ！[Port's icon](../../../static/img/sso/general-assets/PortIcon.png)

![Onelogin initial new application](../../../static/img/sso/onelogin/OneloginInitialApp.png)

单击 "保存"。

:::tip 以下大部分步骤都涉及编辑您创建的初始 Port 应用程序。 请记住，您可以随时打开管理控制台并转到应用程序 -> 应用程序，返回到该应用程序，Port 应用程序将出现在应用程序列表中。

:::

### 第 2 步: 配置 Onelogin 应用程序

在 Port 应用程序中，转到 "配置 "菜单，然后按照以下步骤操作: 

1. 在 "登录 URL "下粘贴以下登录 URL: 

```text showLineNumbers
https://auth.getport.io/authorize?response_type=token&client_id=96IeqL36Q0UIBxIfV1oqOkDWU6UslfDj&connection={CONNECTION_NAME}&redirect_uri=https%3A%2F%2Fapp.getport.io
```

:::note 我们将提供您的 `{CONNECTION_NAME}` (请在 Slack/Intercom 上联系我们)。

:::

2.在 `重定向 URI's` 下设置: https://auth.getport.io/login/callback"。
    - 重定向 URI 是 Onelogin 为登录请求发送验证响应和 ID 标记的位置。

单击 "保存"。

:::warning 在进入下一步之前，请务必点击保存，因为如果没有填写 "重定向 URI"，尝试保存任何其他应用程序参数都会导致错误。

:::

### 第三步: 配置 OIDC 设置

在 Port 应用程序中，转到 "SSO "菜单，然后按照以下步骤操作: 

1. 复制 "客户 ID "和 "客户secret "并发送给 Port(在松弛频道上)。
2. 点击 "Well-known Configuration "链接，并将页面地址发送给 Port(其格式为 `https://{YOUR_DOMAIN}.onelogin.com/oidc/2/.well-known/openid-configuration`)
3. 将 "令牌端点 - 验证方法 "更改为 "无 (PKCE)": 

![Okta app settings](../../../static/img/sso/onelogin/OneloginSSOSetting.png)

单击 "保存"。

### 步骤 #4: 为所有用户添加 `email_verified` 自定义属性

使用 OpenID 需要 Onelogin 在用户登录时向 Port 传递 "email_verified "字段。 Onelogin 默认不存储和公开该字段，因此在本步骤中，您将配置该字段，并将其应用于 Onelogin 账户中的所有用户。 这里概述的步骤也可在[Onelogin documentation](https://developers.onelogin.com/openid-connect/guides/email-verified) 中找到。

1. 在管理控制台中，转到用户 -> 自定义用户字段。
2. 点击 "新建用户字段"。
3. 输入以下详细信息: 
    1. 姓名已验证电子邮件
    2. 短名": email_verified

![Onelogin email verified user field](../../../static/img/sso/onelogin/OneloginEmailVerifiedUserField.png)

自定义字段默认为 "空"，要将其值改为 "true"，需要创建一条自定义映射规则: 

:::note 也可以为组织中需要访问 Port 的每个用户手动将 "电子邮件已验证 "字段的值更改为 "true"。 不过，手动授予大量用户访问权限的做法不具扩展性。

:::

:::tip 此处指定的映射将把 Onelogin 组织中每个 "Status "为 "Active "的用户的 "Email Verified "自定义字段的值设置为 "true"。 如果您需要特定映射，请随意使用不同的映射。

:::

1. 转到用户 -> 映射
2. 点击 "新建映射
3. 输入映射详细信息: 
    1. 名称为映射插入一个友好的名称，如`设置已验证电子邮件`；
    2. 条件设置条件:  - 状态 - 是 - 激活；
    3. 动作设置操作: 设置电子邮件已验证 - true。
4.单击 "保存"。

![Onelogin Email Verified Mapping Rule](../../../static/img/sso/onelogin/OneloginEmailVerifiedMappingRule.png)

创建映射规则后，返回 "用户"->"映射"，然后单击 "重新应用所有映射"。 新映射在应用前可能会处理几分钟。 您可以进入 "活动"->"任务"，或查看特定用户，确认其 "电子邮件已验证 "字段设置为 "true"(而不是默认的空字段)，从而检查映射任务的状态。

### 步骤 #5: 配置 OpenID 索赔

在 Port 应用程序中，转到 "参数 "菜单并按照以下步骤操作: 

1. 点击 `+` 按钮；
2. 在出现的表单中，在`字段名称`下写入openid"，然后单击 "保存"；
3. 在出现的下拉值中，选择 `OpenID name`。

再重复两次，并添加以下参数: 

1. 字段名": 电子邮件，"值": 电子邮件
2. `Field Name`: email_verified, `Values`: 已验证电子邮件(自定义)

流程结束后，"参数 "部分将如下所示: 

![Onelogin App Parameters Setting](../../../static/img/sso/onelogin/OneloginParametersSetting.png)

单击 "保存"。

### 步骤 #6: 向组织公开应用程序

1. 在 "应用程序 "页面，选择 Port 应用程序并进入 "访问 "菜单。
2. 在 "角色 "部分，选择要向其公开 Port 应用程序的角色: 
    ![Onelogin Assign App Roles](../../../static/img/sso/onelogin/OneloginAssignAppRoles.png)
3.单击 "保存"。

完成这些步骤后，具有 Port 应用程序所分配角色的用户将在其门户中看到 Port 应用程序，点击后将登录 Port: 

[Onelogin Portal With Port App](../../../static/img/sso/onelogin/OneloginPortalWithApp.png)

---

## 如何允许将 Onelogin 角色拉到 Port

:::note 本阶段是**可选的**，只有当您希望将所有 Onelogin 角色固有地接入 Port 时才需要本阶段。

**收益: ** 管理 Port 上的权限和用户访问权限。 **成果: ** 对于每个登录的用户，我们将根据您在下面设置中的定义，自动获取其关联的 Onelogin 角色。

:::

要允许在 Port 中自动支持 Onelogin 角色，请按照以下步骤操作: 

1. 在 "应用程序 "页面，选择 Port 应用程序，进入 "参数 "菜单；
2. 点击 `Groups` claims: 
    ![Onelogin App Parameters Setting](../../../static/img/sso/onelogin/OneloginParametersSetting.png)
3.更新组索赔: 
    1.将 "未选中时的默认值 "更改为 "用户角色"；
    2.从下拉菜单中选择 `分号分隔输入`: 
    [Onelogin App Groups Claim Setting](../../../static/img/sso/onelogin/OneloginGroupsClaim.png) 3. 单击 `保存`。
4.单击`保存`。
