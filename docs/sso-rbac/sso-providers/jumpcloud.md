---

title: "JumpCloud
sidebar_position: 4
description: 将 JumpCloud 与 Port 整合

---

# 如何配置 JumpCloud

请按照本分步指南配置 Port 和 JumpCloud 之间的集成。

:::info 为了完成该流程，您需要联系 Port，以交付和接收信息，详情请参见以下指南。

:::

## Port-JumpCloud 集成优势

* 通过 JumpCloud 应用程序连接到 Port 应用程序；
* 用户登录后，您的 JumpCloud 团队将自动与 Port 同步；
* 根据您的 JumpCloud 用户组在 Port 上设置细粒度权限。

### 如何为 Port 配置 JumpCloud 应用程序集成

### 第 1 步: 创建新的 JumpCloud 应用程序

1. 在管理门户中，转到用户身份验证 -> SSO。
2. 单击 "添加新应用程序"。

![JumpCloud new application wizard](../../../static/img/sso/jumpcloud/JumpcloudAddApplication.png)

3.在搜索框中输入 **Auth0**: 

![JumpCloud new application](../../../static/img/sso/jumpcloud/JumpcloudAuth0Search.png)

4.定义 Port 应用程序的初始设置: 
    1. 显示标签为 Port 应用程序插入一个您选择的名称，如 `Port`。
    2. 添加图标(可选): 
    <details>
    <summary>Port Logo</summary>![Port's logo](../../../static/img/sso/general-assets/PortLogoLarge.png)
    </details>
    3. **在 SSO 选项卡中，更改默认 IDP URL 后缀。
    ![JumpCloud initial new application](../../../static/img/sso/jumpcloud/JumpcloudNewSSO.png)

点击 "激活"。

5.点击新创建的应用程序。
    1.下载 IDP 证书: 
    ![Jumpcloud download certificate](../../../static/img/sso/jumpcloud/JumpcloudDownloadCert.png)
    2.从 SSO 选项卡复制 "IDP URL
    ![Jumpcloud IDP URL](../../../static/img/sso/jumpcloud/JumpcloudIDPUrl.png)
6. 通过 intercom/slack，向 Port 提供下载的 "certificate.pem "文件和复制的 "IDP URL"。

:::note 向 Port 提供 "certificate.pem "文件和 "IDP URL "后，系统将向您提供`{CONNECTION_NAME}`。 用提供的值替换下列内容。

:::

:::tip 以下大部分步骤涉及编辑您创建的初始 Port 应用程序。 请记住，您可以随时打开管理控制台并转到用户身份验证 -> SSO，返回到它，Port 应用程序将出现在应用程序列表中。

:::

#### 第二步: 配置你的 JumpCloud 应用程序

在 Port 应用程序中，转到 "SSO "菜单，然后按照以下步骤操作: 

1. 在 `IdP Entity ID:` 下粘贴以下 URL: `https://auth.getport.io`
2. 在 `SP Entity ID:` 下设置: urn:auth0:Port-prod:`{CONNECTION_NAME}`。
3. 在 `ACS URLs` 下设置: https://auth.getport.io/login/callback?connection=`{CONNECTION_NAME}`.

![Jumpcloud SSO configuration](../../../static/img/sso/jumpcloud/JumpcloudConfigureSSO.png)

单击 "保存"。

#### 第三步: 在应用程序配置中添加用户属性

family_name "和 "given_name "属性为必填属性，Port 使用它们来显示登录用户的全名。 请按以下步骤创建这些属性: 

:::note 电子邮件 "用户属性是在创建应用程序时默认创建的，请确保将 "电子邮件 "字段旁边的开关设置为 "开"。

:::

1. 在 Port 应用程序中，转到**用户属性映射**部分下的 "SSO "选项卡: 
2. 单击 "添加属性"。
3. 将 "服务提供商属性名称 "设为 "given_name
4. 在 "Value "字段中输入值: "firstname
5. 再次单击 "添加属性"。
6. 将`服务提供商属性名`设为`family_name`。
7. 在 "Value "字段中输入值: `lastname

![JumpCloud user attributes](../../../static/img/sso/jumpcloud/JumpcloudAttributes.png)

#### 第四步: 在 Port 应用程序中添加 `email_verified` 常量属性

使用 Auth0 需要 JumpCloud 在用户登录时向 Port 传递一个 "email_verified "字段。 JumpCloud 默认情况下不会存储和公开该字段，因此在本步骤中，您将配置该字段，并将其应用于 JumpCloud 账户中的所有用户。

1. 在 Port 应用程序中，转到 "SSO "选项卡的 "**恒定属性 "部分: 
2. 单击 "添加属性"。
3. 将 "服务提供商属性名称 "设为 "email_verified
4. 在 "Value "字段中输入值: "true

![JumpCloud email verified attribute](../../../static/img/sso/jumpcloud/JumpCloudEmailVerified.png)

:::note 也可以手动将组织中每个需要访问 Port 的用户的 "email_verified "字段的值更改为 "true"。 但是，手动授予大量用户访问权限的做法不具可扩展性。

:::

### 步骤 #5: 向组织公开应用程序

1. 在 Port 应用程序中，转到 "用户组 "选项卡。
2. 选择要向其公开 Port 应用程序的用户组: 
    ![JumpCloud add user groups](../../../static/img/sso/jumpcloud/JumpcloudAddUserGroups.png)
3.单击 "保存"。

完成这些步骤后，具有 Port 应用程序所分配角色的用户将在其门户中看到 Port 应用程序，点击后将登录 Port: 

![JumpCloud Portal With Port App](../../../static/img/sso/jumpcloud/JumpcloudPortApplication.png)

---

## 如何允许将 JumpCloud 组拉到 Port

:::note 此阶段为**可选**，只有当您希望将所有 JumpCloud 组固有地接入 Port 时才需要此阶段。

**收益: ** 管理 Port 上的权限和用户访问权限。 **成果: ** 对于每个登录的用户，我们将根据您在下方设置中的定义，自动获取其关联的 JumpCloud 组。

:::

要允许在 Port 中自动支持组群，请按照以下步骤操作: 

1. 在 Port 应用程序中，转到**组属性**部分下的 "SSO "选项卡
2. 选中 "包括组属性 "框
3. 设置组属性名称: "memberOf

![JumpCloud Group configuration](../../../static/img/sso/jumpcloud/JumpcloudGroupConfig.png)

4.单击 "保存"。
