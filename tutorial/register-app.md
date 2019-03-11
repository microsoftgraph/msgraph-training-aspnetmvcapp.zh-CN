<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c8d12-101">在本练习中, 将使用应用程序注册表门户 (ARP) 创建新的 Azure AD web 应用程序注册。</span><span class="sxs-lookup"><span data-stu-id="c8d12-101">In this exercise, you will create a new Azure AD web application registration using the Application Registry Portal (ARP).</span></span>

1. <span data-ttu-id="c8d12-102">打开浏览器并导航到[应用程序注册门户](https://apps.dev.microsoft.com)。</span><span class="sxs-lookup"><span data-stu-id="c8d12-102">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="c8d12-103">使用**个人帐户**(亦称: Microsoft 帐户) 或**工作或学校帐户**登录。</span><span class="sxs-lookup"><span data-stu-id="c8d12-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="c8d12-104">选择页面顶部的 "**添加应用**"。</span><span class="sxs-lookup"><span data-stu-id="c8d12-104">Select **Add an app** at the top of the page.</span></span>

    > [!NOTE]
    > <span data-ttu-id="c8d12-105">如果在页面上看到多个 "**添加应用程序**" 按钮, 请选择与 "**聚合应用程序**" 列表对应的项。</span><span class="sxs-lookup"><span data-stu-id="c8d12-105">If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="c8d12-106">在 "**注册应用程序**" 页上, 将**应用程序名称**设置为**ASP.NET Graph 教程**, 然后选择 "**创建**"。</span><span class="sxs-lookup"><span data-stu-id="c8d12-106">On the **Register your application** page, set the **Application Name** to **ASP.NET Graph Tutorial** and select **Create**.</span></span>

    ![在应用注册门户网站中创建新应用程序的屏幕截图](./images/arp-create-app-01.png)

1. <span data-ttu-id="c8d12-108">在 " **ASP.NET Graph 教程注册**" 页上的 "**属性**" 部分, 复制**应用程序 Id** , 因为稍后将需要它。</span><span class="sxs-lookup"><span data-stu-id="c8d12-108">On the **ASP.NET Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![新创建的应用程序 ID 的屏幕截图](./images/arp-create-app-02.png)

1. <span data-ttu-id="c8d12-110">向下滚动到 "**应用程序机密**" 部分。</span><span class="sxs-lookup"><span data-stu-id="c8d12-110">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="c8d12-111">选择 "**生成新密码**"。</span><span class="sxs-lookup"><span data-stu-id="c8d12-111">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="c8d12-112">在 "**生成新密码**" 对话框中, 复制框中的内容, 因为稍后将需要它。</span><span class="sxs-lookup"><span data-stu-id="c8d12-112">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="c8d12-113">**重要说明:** 此密码永远不会再次显示, 因此请务必立即复制它。</span><span class="sxs-lookup"><span data-stu-id="c8d12-113">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![新创建的应用程序密码的屏幕截图](./images/arp-create-app-03.png)

1. <span data-ttu-id="c8d12-115">确定您的 ASP.NET 应用程序的 URL。</span><span class="sxs-lookup"><span data-stu-id="c8d12-115">Determine your ASP.NET app's URL.</span></span> <span data-ttu-id="c8d12-116">在 Visual Studio 的 "解决方案资源管理器" 中, 选择 "**绘图教程**" 项目。</span><span class="sxs-lookup"><span data-stu-id="c8d12-116">In Visual Studio's Solution Explorer, select the **graph-tutorial** project.</span></span> <span data-ttu-id="c8d12-117">在 "**属性**" 窗口中, 查找**URL**的值。</span><span class="sxs-lookup"><span data-stu-id="c8d12-117">In the **Properties** window, find the value of **URL**.</span></span> <span data-ttu-id="c8d12-118">复制此值。</span><span class="sxs-lookup"><span data-stu-id="c8d12-118">Copy this value.</span></span>

    ![Visual Studio 的 "属性" 窗口的屏幕截图](./images/vs-project-url.png)

1. <span data-ttu-id="c8d12-120">向下滚动到 "**平台**" 部分。</span><span class="sxs-lookup"><span data-stu-id="c8d12-120">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="c8d12-121">选择 "**添加平台**"。</span><span class="sxs-lookup"><span data-stu-id="c8d12-121">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="c8d12-122">在 "**添加平台**" 对话框中, 选择 " **Web**"。</span><span class="sxs-lookup"><span data-stu-id="c8d12-122">In the **Add Platform** dialog, select **Web**.</span></span>

        ![为应用程序创建平台的屏幕截图](./images/arp-create-app-04.png)

    1. <span data-ttu-id="c8d12-124">在 " **Web**平台" 框中, 输入从 Visual Studio 项目的属性中复制的用于**重定向 url**的 URL。</span><span class="sxs-lookup"><span data-stu-id="c8d12-124">In the **Web** platform box, enter the URL you copied from the Visual Studio project's properties for the **Redirect URLs**.</span></span>

        ![应用程序新添加的 Web 平台的屏幕截图](./images/arp-create-app-05.png)

1. <span data-ttu-id="c8d12-126">滚动到页面底部, 然后选择 "**保存**"。</span><span class="sxs-lookup"><span data-stu-id="c8d12-126">Scroll to the bottom of the page and select **Save**.</span></span>