# <a name="how-to-run-the-completed-project"></a>如何运行已完成的项目

## <a name="prerequisites"></a>先决条件

若要在此文件夹中运行已完成的项目, 您需要以下各项:

- 在开发计算机上安装的[Visual Studio](https://visualstudio.microsoft.com/vs/) 。 如果没有 Visual Studio, 请访问 "下载选项" 的上一个链接。 (**注意:** 本教程是使用 Visual Studio 2017 版本15.81 编写的。 本指南中的步骤可能适用于其他版本, 但尚未经过测试。
- 使用 Outlook.com 上的邮箱的个人 Microsoft 帐户, 或者是 microsoft 工作或学校帐户。

如果你没有 Microsoft 帐户, 可以使用以下几种方法获取免费帐户:

- 你可以[注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 你可以[注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program)以获取免费的 office 365 订阅。

## <a name="register-a-web-application-with-the-application-registration-portal"></a>在应用程序注册门户中注册 web 应用程序

1. 打开浏览器并导航到[应用程序注册门户](https://apps.dev.microsoft.com)。 使用**个人帐户**(亦称: Microsoft 帐户) 或**工作或学校帐户**登录。

1. 选择页面顶部的 "**添加应用**"。

    > **注意:** 如果在页面上看到多个 "**添加应用程序**" 按钮, 请选择与 "**聚合应用程序**" 列表对应的项。

1. 在 "**注册应用程序**" 页上, 将**应用程序名称**设置为**ASP.NET Graph 教程**, 然后选择 "**创建**"。

    ![在应用注册门户网站中创建新应用程序的屏幕截图](/tutorial/images/arp-create-app-01.png)

1. 在 " **ASP.NET Graph 教程注册**" 页上的 "**属性**" 部分, 复制**应用程序 Id** , 因为稍后将需要它。

    ![新创建的应用程序 ID 的屏幕截图](/tutorial/images/arp-create-app-02.png)

1. 向下滚动到 "**应用程序机密**" 部分。

    1. 选择 "**生成新密码**"。
    1. 在 "**生成新密码**" 对话框中, 复制框中的内容, 因为稍后将需要它。

        > **重要说明:** 此密码永远不会再次显示, 因此请务必立即复制它。

    ![新创建的应用程序密码的屏幕截图](/tutorial/images/arp-create-app-03.png)

1. 向下滚动到 "**平台**" 部分。

    1. 选择 "**添加平台**"。
    1. 在 "**添加平台**" 对话框中, 选择 " **Web**"。

        ![为应用程序创建平台的屏幕截图](/tutorial/images/arp-create-app-04.png)

    1. 在 " **Web**平台" 框中, 输入`http://localhost:64107/` **重定向 url**的 url。

        ![应用程序新添加的 Web 平台的屏幕截图](/tutorial/images/arp-create-app-05.png)

1. 滚动到页面底部, 然后选择 "**保存**"。

## <a name="configure-the-sample"></a>配置示例

1. 将`PrivateSettings.config.example`文件重命名`PrivateSettings.config`为。
1. 编辑`PrivateSettings.config`文件并进行以下更改。
    1. 将`YOUR_APP_ID_HERE`替换为你从应用注册门户获取的**应用程序 Id** 。
    1. 将`YOUR_APP_PASSWORD_HERE`替换为你在应用注册门户中获取的密码。
1. 在`graph-tutorial.sln` Visual Studio 中打开。 在 "解决方案资源管理器" 中, 右键单击**graph 教程**解决方案并选择 "**还原 NuGet 包**"。

## <a name="run-the-sample"></a>运行示例

在 Visual Studio 中, 按**F5**或选择 "**调试" > "开始调试**"。