# <a name="how-to-run-the-completed-project"></a>如何运行已完成的项目

## <a name="prerequisites"></a>先决条件

若要在此文件夹中运行已完成的项目，您需要以下各项：

- 在开发计算机上安装的[Visual Studio](https://visualstudio.microsoft.com/vs/) 。 如果没有 Visual Studio，请访问 "下载选项" 的上一个链接。 （**注意：** 本教程是使用 Visual Studio 2019 版本16.2.3 编写的。 本指南中的步骤可能适用于其他版本，但尚未经过测试。
- 使用 Outlook.com 上的邮箱的个人 Microsoft 帐户，或者是 Microsoft 工作或学校帐户。

如果你没有 Microsoft 帐户，可以使用以下几种方法获取免费帐户：

- 你可以[注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 你可以[注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program)以获取免费的 office 365 订阅。

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>向 Azure Active Directory 管理中心注册 web 应用程序

1. 确定您的 ASP.NET 应用程序的 SSL URL。 在 Visual Studio 的 "解决方案资源管理器" 中，选择 "**绘图教程**" 项目。 在 "**属性**" 窗口中，找到 " **SSL URL**" 的值。 复制此值。

    ![Visual Studio 的 "属性" 窗口的屏幕截图](/tutorial/images/vs-project-url.png)

1. 打开浏览器，并转到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。 使用**个人帐户**（亦称为“Microsoft 帐户”）或**工作或学校帐户**登录。

1. 在左侧导航栏中选择 " **Azure Active Directory** "，然后选择 "**管理**" 下的 "**应用程序注册**"。

    ![应用注册的屏幕截图 ](/tutorial/images/aad-portal-app-registrations.png)

1. 选择“新注册”****。 在“注册应用”**** 页上，按如下方式设置值。

    - 将“名称”**** 设置为“`ASP.NET Graph Tutorial`”。
    - 将“受支持的帐户类型”**** 设置为“任何组织目录中的帐户和个人 Microsoft 帐户”****。
    - 在“重定向 URI”**** 下，将第一个下拉列表设置为“`Web`”，并将值设置为在第 1 步中复制的 ASP.NET 应用 URL。

    !["注册应用程序" 页的屏幕截图](/tutorial/images/aad-register-an-app.png)

1. 选择“注册”****。 在 " **ASP.NET Graph 教程**" 页上，复制**应用程序（客户端） ID**的值并保存它，下一步将需要它。

    ![新应用注册的应用程序 ID 的屏幕截图](/tutorial/images/aad-application-id.png)

1. 选择“管理”**** 下的“身份验证”****。 找到“隐式授予”**** 部分，并启用“ID 令牌”****。 选择“保存”****。

    ![隐式 grant 部分的屏幕截图](/tutorial/images/aad-implicit-grant.png)

1. 选择“管理”**** 下的“证书和密码”****。 选择“新客户端密码”**** 按钮。 在“说明”**** 中输入值，并选择一个“过期”**** 选项，再选择“添加”****。

    !["添加客户端密码" 对话框的屏幕截图](/tutorial/images/aad-new-client-secret.png)

1. 离开此页前，先复制客户端密码值。 将在下一步中用到它。

    > [!IMPORTANT]
    > 此客户端密码不会再次显示，所以请务必现在就复制它。

    ![新添加的客户端密码的屏幕截图](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a>配置示例

1. 将`PrivateSettings.config.example`文件重命名`PrivateSettings.config`为。
1. 编辑`PrivateSettings.config`文件并进行以下更改。
    1. 将`YOUR_APP_ID_HERE`替换为你从应用注册门户获取的**应用程序 Id** 。
    1. 将`YOUR_APP_PASSWORD_HERE`替换为你从应用注册门户获取的**应用程序密码**。
1. 在`graph-tutorial.sln` Visual Studio 中打开。 在 "解决方案资源管理器" 中，右键单击**graph 教程**解决方案并选择 "**还原 NuGet 包**"。

## <a name="run-the-sample"></a>运行示例

在 Visual Studio 中，按**F5**或选择 "**调试" > "开始调试**"。
