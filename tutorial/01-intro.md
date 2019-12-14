<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="84c7e-101">本教程向您介绍如何生成使用 Microsoft Graph API 检索用户的日历信息的 ASP.NET MVC web 应用程序。</span><span class="sxs-lookup"><span data-stu-id="84c7e-101">This tutorial teaches you how to build an ASP.NET MVC web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="84c7e-102">如果您只想下载已完成的教程，可以通过两种方式下载它。</span><span class="sxs-lookup"><span data-stu-id="84c7e-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="84c7e-103">下载[ASP.NET 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet)，以分钟为单位获取工作代码。</span><span class="sxs-lookup"><span data-stu-id="84c7e-103">Download the [ASP.NET quick start](https://developer.microsoft.com/graph/quick-start?platform=option-dotnet) to get working code in minutes.</span></span>
> - <span data-ttu-id="84c7e-104">下载或克隆[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)。</span><span class="sxs-lookup"><span data-stu-id="84c7e-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="84c7e-105">先决条件</span><span class="sxs-lookup"><span data-stu-id="84c7e-105">Prerequisites</span></span>

<span data-ttu-id="84c7e-106">在开始本教程之前，您应在开发计算机上安装[Visual Studio](https://visualstudio.microsoft.com/vs/) 。</span><span class="sxs-lookup"><span data-stu-id="84c7e-106">Before you start this tutorial, you should have [Visual Studio](https://visualstudio.microsoft.com/vs/) installed on your development machine.</span></span> <span data-ttu-id="84c7e-107">如果没有 Visual Studio，请访问 "下载选项" 的上一个链接。</span><span class="sxs-lookup"><span data-stu-id="84c7e-107">If you do not have Visual Studio, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="84c7e-108">本教程是使用 Visual Studio 2019 版本16.4.0 编写的。</span><span class="sxs-lookup"><span data-stu-id="84c7e-108">This tutorial was written with Visual Studio 2019 version 16.4.0.</span></span> <span data-ttu-id="84c7e-109">本指南中的步骤可能适用于其他版本，但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="84c7e-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="watch-the-tutorial"></a><span data-ttu-id="84c7e-110">观看教程</span><span class="sxs-lookup"><span data-stu-id="84c7e-110">Watch the tutorial</span></span>

<span data-ttu-id="84c7e-111">此模块已记录，在 Office 开发 YouTube 频道中可用。</span><span class="sxs-lookup"><span data-stu-id="84c7e-111">This module has been recorded and is available in the Office Development YouTube channel.</span></span>

<!-- markdownlint-disable MD033 MD034 -->
<br/>

> [!VIDEO https://www.youtube-nocookie.com/embed/a2teHZ5WuNc]
<!-- markdownlint-enable MD033 MD034 -->

## <a name="feedback"></a><span data-ttu-id="84c7e-112">反馈</span><span class="sxs-lookup"><span data-stu-id="84c7e-112">Feedback</span></span>

<span data-ttu-id="84c7e-113">请在[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)中提供有关本教程的任何反馈。</span><span class="sxs-lookup"><span data-stu-id="84c7e-113">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp).</span></span>
