<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="aaca1-101">在此演示中, 将向应用程序中引入 Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="aaca1-101">In this demo you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="aaca1-102">对于此应用程序, 您将使用[Microsoft Graph 客户端库进行 .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet)以调用 microsoft graph。</span><span class="sxs-lookup"><span data-stu-id="aaca1-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="aaca1-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="aaca1-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="aaca1-104">首先扩展您在`GraphHelper`上一模块中创建的类。</span><span class="sxs-lookup"><span data-stu-id="aaca1-104">Start by extending the `GraphHelper` class you created in the last module.</span></span>

1. <span data-ttu-id="aaca1-105">将以下`using`语句添加到`Helpers/GraphHelper.cs`文件顶部。</span><span class="sxs-lookup"><span data-stu-id="aaca1-105">Add the following `using` statements to the top of the `Helpers/GraphHelper.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using Microsoft.Identity.Client;
    using System.Collections.Generic;
    using System.Configuration;
    using System.Linq;
    using System.Security.Claims;
    using System.Web;
    ```

1. <span data-ttu-id="aaca1-106">将以下代码添加到`GraphHelper`类中。</span><span class="sxs-lookup"><span data-stu-id="aaca1-106">Add the following code to the `GraphHelper` class.</span></span>

    ```cs
    // Load configuration settings from PrivateSettings.config
    private static string appId = ConfigurationManager.AppSettings["ida:AppId"];
    private static string appSecret = ConfigurationManager.AppSettings["ida:AppSecret"];
    private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
    private static string graphScopes = ConfigurationManager.AppSettings["ida:AppScopes"];

    public static async Task<IEnumerable<Event>> GetEventsAsync()
    {
        var graphClient = GetAuthenticatedClient();

        var events = await graphClient.Me.Events.Request()
            .Select("subject,organizer,start,end")
            .OrderBy("createdDateTime DESC")
            .GetAsync();

        return events.CurrentPage;
    }

    private static GraphServiceClient GetAuthenticatedClient()
    {
        return new GraphServiceClient(
            new DelegateAuthenticationProvider(
                async (requestMessage) =>
                {
                    var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                        .WithRedirectUri(redirectUri)
                        .WithClientSecret(appSecret)
                        .Build();

                    var tokenStore = new SessionTokenStore(idClient.UserTokenCache,
                            HttpContext.Current, ClaimsPrincipal.Current);

                    var accounts = await idClient.GetAccountsAsync();

                    // By calling this here, the token can be refreshed
                    // if it's expired right before the Graph call is made
                    var scopes = graphScopes.Split(' ');
                    var result = await idClient.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                        .ExecuteAsync();

                    requestMessage.Headers.Authorization =
                        new AuthenticationHeaderValue("Bearer", result.AccessToken);
                }));
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="aaca1-107">考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="aaca1-107">Consider what this code is doing.</span></span>
    >
    > - <span data-ttu-id="aaca1-108">函数使用调用`AcquireTokenSilent`的身份验证提供程序初始化 a `GraphServiceClient` `GetAuthenticatedClient`</span><span class="sxs-lookup"><span data-stu-id="aaca1-108">The `GetAuthenticatedClient` function initializes a `GraphServiceClient` with an authentication provider that calls `AcquireTokenSilent`.</span></span>
    > - <span data-ttu-id="aaca1-109">在`GetEventsAsync`函数中:</span><span class="sxs-lookup"><span data-stu-id="aaca1-109">In the `GetEventsAsync` function:</span></span>
    >   - <span data-ttu-id="aaca1-110">将调用的 URL 为`/v1.0/me/events`。</span><span class="sxs-lookup"><span data-stu-id="aaca1-110">The URL that will be called is `/v1.0/me/events`.</span></span>
    >   - <span data-ttu-id="aaca1-111">`Select`函数将为每个事件返回的字段限制为仅供视图实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="aaca1-111">The `Select` function limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="aaca1-112">`OrderBy`函数按其创建日期和时间对结果进行排序, 最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="aaca1-112">The `OrderBy` function sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="aaca1-113">为日历视图创建一个控制器。</span><span class="sxs-lookup"><span data-stu-id="aaca1-113">Create a controller for the calendar views.</span></span> <span data-ttu-id="aaca1-114">右键单击 "解决方案资源管理器" 中的 "**控制器**" 文件夹, 然后选择 "**添加 > 控制器 ...**"。选择 " **MVC 5 控制器-空**", 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="aaca1-114">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="aaca1-115">命名控制器`CalendarController`并选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="aaca1-115">Name the controller `CalendarController` and select **Add**.</span></span> <span data-ttu-id="aaca1-116">将新文件的全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="aaca1-116">Replace the entire contents of the new file with the following code.</span></span>

    ```cs
    using graph_tutorial.Helpers;
    using System;
    using System.Threading.Tasks;
    using System.Web.Mvc;

    namespace graph_tutorial.Controllers
    {
        public class CalendarController : BaseController
        {
            // GET: Calendar
            [Authorize]
            public async Task<ActionResult> Index()
            {
                var events = await GraphHelper.GetEventsAsync();

                // Change start and end dates from UTC to local time
                foreach (var ev in events)
                {
                    ev.Start.DateTime = DateTime.Parse(ev.Start.DateTime).ToLocalTime().ToString();
                    ev.Start.TimeZone = TimeZoneInfo.Local.Id;
                    ev.End.DateTime = DateTime.Parse(ev.End.DateTime).ToLocalTime().ToString();
                    ev.End.TimeZone = TimeZoneInfo.Local.Id;
                }

                return Json(events, JsonRequestBehavior.AllowGet);
            }
        }
    }
    ```

1. <span data-ttu-id="aaca1-117">启动应用程序, 登录, 然后单击导航栏中的 "**日历**" 链接。</span><span class="sxs-lookup"><span data-stu-id="aaca1-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="aaca1-118">如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。</span><span class="sxs-lookup"><span data-stu-id="aaca1-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="aaca1-119">显示结果</span><span class="sxs-lookup"><span data-stu-id="aaca1-119">Display the results</span></span>

<span data-ttu-id="aaca1-120">现在, 您可以添加一个视图, 以对用户更友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="aaca1-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="aaca1-121">在 "解决方案资源管理器" 中, 右键单击 "**视图/日历**" 文件夹, 然后选择 "**添加 > 视图 ...**"。为视图`Index`命名, 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="aaca1-121">In Solution Explorer, right-click the **Views/Calendar** folder and select **Add > View...**. Name the view `Index` and select **Add**.</span></span> <span data-ttu-id="aaca1-122">将新文件的全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="aaca1-122">Replace the entire contents of the new file with the following code.</span></span>

    ```html
    @model IEnumerable<Microsoft.Graph.Event>

    @{
        ViewBag.Current = "Calendar";
    }

    <h1>Calendar</h1>
    <table class="table">
        <thead>
            <tr>
                <th scope="col">Organizer</th>
                <th scope="col">Subject</th>
                <th scope="col">Start</th>
                <th scope="col">End</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model)
            {
                <tr>
                    <td>@item.Organizer.EmailAddress.Name</td>
                    <td>@item.Subject</td>
                    <td>@Convert.ToDateTime(item.Start.DateTime).ToString("M/d/yy h:mm tt")</td>
                    <td>@Convert.ToDateTime(item.End.DateTime).ToString("M/d/yy h:mm tt")</td>
                </tr>
            }
        </tbody>
    </table>
    ```

    <span data-ttu-id="aaca1-123">这将遍历一组事件并为每个事件添加一个表行。</span><span class="sxs-lookup"><span data-stu-id="aaca1-123">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="aaca1-124">从中`return Json(events, JsonRequestBehavior.AllowGet);`的`Index`函数中`Controllers/CalendarController.cs`移除该行, 并将其替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="aaca1-124">Remove the `return Json(events, JsonRequestBehavior.AllowGet);` line from the `Index` function in `Controllers/CalendarController.cs`, and replace it with the following code.</span></span>

    ```cs
    return View(events);
    ```

1. <span data-ttu-id="aaca1-125">启动应用程序, 登录, 然后单击 "**日历**" 链接。</span><span class="sxs-lookup"><span data-stu-id="aaca1-125">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="aaca1-126">应用现在应呈现一个事件表。</span><span class="sxs-lookup"><span data-stu-id="aaca1-126">The app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
