<!-- markdownlint-disable MD002 MD041 -->

在此演示中, 您将把 Microsoft Graph 合并到应用程序中。 对于此应用程序, 您将使用[microsoft graph 客户端库进行 .net](https://github.com/microsoftgraph/msgraph-sdk-dotnet)以调用 microsoft graph。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

首先扩展您在`GraphHelper`上一模块中创建的类。 首先, 将以下`using`语句添加到`Helpers/GraphHelper.cs`文件顶部。

```cs
using graph_tutorial.TokenStorage;
using Microsoft.Identity.Client;
using System.Collections.Generic;
using System.Configuration;
using System.Linq;
using System.Security.Claims;
using System.Web;
```

然后, 将以下代码添加到`GraphHelper`类中。

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
                // Get the signed in user's id and create a token cache
                string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
                SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId,
                    new HttpContextWrapper(HttpContext.Current));

                var idClient = new ConfidentialClientApplication(
                    appId, redirectUri, new ClientCredential(appSecret),
                    tokenStore.GetMsalCacheInstance(), null);

                var accounts = await idClient.GetAccountsAsync();

                // By calling this here, the token can be refreshed
                // if it's expired right before the Graph call is made
                var result = await idClient.AcquireTokenSilentAsync(
                    graphScopes.Split(' '), accounts.FirstOrDefault());

                requestMessage.Headers.Authorization =
                    new AuthenticationHeaderValue("Bearer", result.AccessToken);
            }));
}
```

考虑此代码执行的操作。

- 函数使用调用`AcquireTokenSilentAsync`的身份验证提供程序初始化 a `GraphServiceClient` `GetAuthenticatedClient`
- 在`GetEventsAsync`函数中:
  - 将调用的 URL 为`/v1.0/me/events`。
  - `Select`函数将为每个事件返回的字段限制为仅供视图实际使用的字段。
  - `OrderBy`函数按其创建日期和时间对结果进行排序, 最新项目最先开始。

现在, 为日历视图创建一个控制器。 右键单击 "解决方案资源管理器" 中的 "**控制器**" 文件夹, 然后选择 "**添加 > 控制器 ...**"。选择 " **MVC 5 控制器-空**", 然后选择 "**添加**"。 命名控制器`CalendarController` , 然后选择 "**添加**"。 将新文件的全部内容替换为以下代码。

```cs
using graph_tutorial.Helpers;
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
            return Json(events, JsonRequestBehavior.AllowGet);
        }
    }
}
```

现在, 您可以对此进行测试。 启动应用程序, 登录, 然后单击导航栏中的 "**日历**" 链接。 如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。

## <a name="display-the-results"></a>显示结果

现在, 您可以添加一个视图, 以对用户更友好的方式显示结果。 在 "解决方案资源管理器" 中, 右键单击 "**视图/日历**" 文件夹, 然后选择 "**添加 > 视图 ...**"。为视图`Index`命名, 然后选择 "**添加**"。 将新文件的全部内容替换为以下代码。

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

这将遍历一组事件并为每个事件添加一个表行。 从中`return Json(events, JsonRequestBehavior.AllowGet);`的`Index`函数中`Controllers/CalendarController.cs`移除该行, 并将其替换为以下代码。

```cs
return View(events);
```

启动应用程序, 登录, 然后单击 "**日历**" 链接。 应用现在应呈现一个事件表。

![事件表的屏幕截图](./images/add-msgraph-01.png)