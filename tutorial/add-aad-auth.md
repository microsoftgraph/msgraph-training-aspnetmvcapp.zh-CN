<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="33b9f-101">在本练习中, 你将扩展上一练习中的应用程序, 以支持 Azure AD 的身份验证。</span><span class="sxs-lookup"><span data-stu-id="33b9f-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="33b9f-102">若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph, 这是必需的。</span><span class="sxs-lookup"><span data-stu-id="33b9f-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="33b9f-103">在此步骤中, 将 OWIN 中间件和[Microsoft 身份验证库](https://www.nuget.org/packages/Microsoft.Identity.Client/)库集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="33b9f-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

<span data-ttu-id="33b9f-104">右键单击 "解决方案资源管理器" 中的 "**教程**" 项目, 然后选择 "**添加 > 新项 ...**"。选择 " **Web 配置文件**", 命名`PrivateSettings.config`该文件, 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-104">Right-click the **graph-tutorial** project in Solution Explorer and choose **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and choose **Add**.</span></span> <span data-ttu-id="33b9f-105">用以下代码替换它的全部内容。</span><span class="sxs-lookup"><span data-stu-id="33b9f-105">Replace its entire contents with the following code.</span></span>

```xml
<appSettings>
    <add key="ida:AppID" value="YOUR APP ID" />
    <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
    <add key="ida:RedirectUri" value="http://localhost:PORT/" />
    <add key="ida:AppScopes" value="User.Read Calendars.Read" />
</appSettings>
```

<span data-ttu-id="33b9f-106">将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID, `YOUR_APP_PASSWORD_HERE`并将替换为您生成的客户端密码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="33b9f-107">如果你的客户端密码包含任何`&`& (), 请务必将其`&amp;`替换`PrivateSettings.config`为。</span><span class="sxs-lookup"><span data-stu-id="33b9f-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="33b9f-108">此外, 请务必修改的`PORT`值, 以`ida:RedirectUri`匹配您的应用程序的 URL。</span><span class="sxs-lookup"><span data-stu-id="33b9f-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="33b9f-109">如果您使用的是源代码管理 (如 git), 现在可以从源代码管理中排除`PrivateSettings.config`该文件, 以避免无意中泄漏您的应用程序 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="33b9f-110">更新`Web.config`以加载此新文件。</span><span class="sxs-lookup"><span data-stu-id="33b9f-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="33b9f-111">将`<appSettings>` (第7行) 替换为以下项</span><span class="sxs-lookup"><span data-stu-id="33b9f-111">Replace the `<appSettings>` (line 7) with the following</span></span>

```xml
<appSettings file="PrivateSettings.config">
```

## <a name="implement-sign-in"></a><span data-ttu-id="33b9f-112">实施登录</span><span class="sxs-lookup"><span data-stu-id="33b9f-112">Implement sign-in</span></span>

<span data-ttu-id="33b9f-113">首先, 初始化 OWIN 中间件以使用应用程序的 Azure AD 身份验证。</span><span class="sxs-lookup"><span data-stu-id="33b9f-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span> <span data-ttu-id="33b9f-114">右键单击 "解决方案资源管理器" 中的 " **App_Start** " 文件夹, 然后选择 "**添加 > 类 ...**"。命名该文件`Startup.Auth.cs` , 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-114">Right-click the **App_Start** folder in Solution Explorer and choose **Add > Class...**. Name the file `Startup.Auth.cs` and choose **Add**.</span></span> <span data-ttu-id="33b9f-115">将整个内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-115">Replace the entire contents with the following code.</span></span>

```cs
using Microsoft.Identity.Client;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;
using Microsoft.IdentityModel.Tokens;
using Microsoft.Owin.Security;
using Microsoft.Owin.Security.Cookies;
using Microsoft.Owin.Security.Notifications;
using Microsoft.Owin.Security.OpenIdConnect;
using Owin;
using System.Configuration;
using System.Threading.Tasks;
using System.Web;

namespace graph_tutorial
{
    public partial class Startup
    {
        // Load configuration settings from PrivateSettings.config
        private static string appId = ConfigurationManager.AppSettings["ida:AppId"];
        private static string appSecret = ConfigurationManager.AppSettings["ida:AppSecret"];
        private static string redirectUri = ConfigurationManager.AppSettings["ida:RedirectUri"];
        private static string graphScopes = ConfigurationManager.AppSettings["ida:AppScopes"];

        public void ConfigureAuth(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());

            app.UseOpenIdConnectAuthentication(
              new OpenIdConnectAuthenticationOptions
              {
                  ClientId = appId,
                  Authority = "https://login.microsoftonline.com/common/v2.0",
                  Scope = $"openid email profile offline_access {graphScopes}",
                  RedirectUri = redirectUri,
                  PostLogoutRedirectUri = redirectUri,
                  TokenValidationParameters = new TokenValidationParameters
                  {
                      // For demo purposes only, see below
                      ValidateIssuer = false

                      // In a real multi-tenant app, you would add logic to determine whether the
                      // issuer was from an authorized tenant
                      //ValidateIssuer = true,
                      //IssuerValidator = (issuer, token, tvp) =>
                      //{
                      //  if (MyCustomTenantValidation(issuer))
                      //  {
                      //    return issuer;
                      //  }
                      //  else
                      //  {
                      //    throw new SecurityTokenInvalidIssuerException("Invalid issuer");
                      //  }
                      //}
                  },
                  Notifications = new OpenIdConnectAuthenticationNotifications
                  {
                      AuthenticationFailed = OnAuthenticationFailedAsync,
                      AuthorizationCodeReceived = OnAuthorizationCodeReceivedAsync
                  }
              }
            );
        }

        private static Task OnAuthenticationFailedAsync(AuthenticationFailedNotification<OpenIdConnectMessage,
          OpenIdConnectAuthenticationOptions> notification)
        {
            notification.HandleResponse();
            string redirect = $"/Home/Error?message={notification.Exception.Message}";
            if (notification.ProtocolMessage != null && !string.IsNullOrEmpty(notification.ProtocolMessage.ErrorDescription))
            {
                redirect += $"&debug={notification.ProtocolMessage.ErrorDescription}";
            }
            notification.Response.Redirect(redirect);
            return Task.FromResult(0);
        }

        private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
        {
            var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                .WithRedirectUri(redirectUri)
                .WithClientSecret(appSecret)
                .Build();

            string message;
            string debug;

            try
            {
                string[] scopes = graphScopes.Split(' ');

                var result = await idClient.AcquireTokenByAuthorizationCode(
                    scopes, notification.Code).ExecuteAsync();

                message = "Access token retrieved.";
                debug = result.AccessToken;
            }
            catch (MsalException ex)
            {
                message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
                debug = ex.Message;
            }

            notification.HandleResponse();
            notification.Response.Redirect($"/Home/Error?message={message}&debug={debug}");
        }
    }
}
```

<span data-ttu-id="33b9f-116">此代码使用中`PrivateSettings.config`的值配置 OWIN 中间件, `OnAuthenticationFailedAsync`并定义两个回调方法`OnAuthorizationCodeReceivedAsync`和。</span><span class="sxs-lookup"><span data-stu-id="33b9f-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="33b9f-117">当登录进程从 Azure 返回时, 将调用这些回调方法。</span><span class="sxs-lookup"><span data-stu-id="33b9f-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

<span data-ttu-id="33b9f-118">现在, 更新`Startup.cs`文件以调用`ConfigureAuth`方法。</span><span class="sxs-lookup"><span data-stu-id="33b9f-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="33b9f-119">将整个内容`Startup.cs`替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

```cs
using Microsoft.Owin;
using Owin;

[assembly: OwinStartup(typeof(graph_tutorial.Startup))]

namespace graph_tutorial
{
    public partial class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            ConfigureAuth(app);
        }
    }
}
```

<span data-ttu-id="33b9f-120">`Error`向`HomeController`类中添加一个操作, 以将`message`和`debug`查询参数转换为`Alert`一个对象。</span><span class="sxs-lookup"><span data-stu-id="33b9f-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="33b9f-121">打开`Controllers/HomeController.cs`并添加以下函数。</span><span class="sxs-lookup"><span data-stu-id="33b9f-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

```cs
public ActionResult Error(string message, string debug)
{
    Flash(message, debug);
    return RedirectToAction("Index");
}
```

<span data-ttu-id="33b9f-122">添加控制器以处理登录。</span><span class="sxs-lookup"><span data-stu-id="33b9f-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="33b9f-123">右键单击 "解决方案资源管理器" 中的 "**控制器**" 文件夹, 然后选择 "**添加 > 控制器 ...**"。选择 " **MVC 5 控制器-空**", 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-123">Right-click the **Controllers** folder in Solution Explorer and choose **Add > Controller...**. Choose **MVC 5 Controller - Empty** and choose **Add**.</span></span> <span data-ttu-id="33b9f-124">命名控制器`AccountController` , 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-124">Name the controller `AccountController` and choose **Add**.</span></span> <span data-ttu-id="33b9f-125">将文件的全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-125">Replace the entire contents of the file with the following code.</span></span>

```cs
using Microsoft.Owin.Security;
using Microsoft.Owin.Security.Cookies;
using Microsoft.Owin.Security.OpenIdConnect;
using System.Security.Claims;
using System.Web;
using System.Web.Mvc;

namespace graph_tutorial.Controllers
{
    public class AccountController : Controller
    {
        public void SignIn()
        {
            if (!Request.IsAuthenticated)
            {
                // Signal OWIN to send an authorization request to Azure
                Request.GetOwinContext().Authentication.Challenge(
                    new AuthenticationProperties { RedirectUri = "/" },
                    OpenIdConnectAuthenticationDefaults.AuthenticationType);
            }
        }

        public ActionResult SignOut()
        {
            if (Request.IsAuthenticated)
            {
                Request.GetOwinContext().Authentication.SignOut(
                    CookieAuthenticationDefaults.AuthenticationType);
            }

            return RedirectToAction("Index", "Home");
        }
    }
}
```

<span data-ttu-id="33b9f-126">这将`SignIn`定义 and `SignOut`操作。</span><span class="sxs-lookup"><span data-stu-id="33b9f-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="33b9f-127">该`SignIn`操作将检查是否已对请求进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="33b9f-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="33b9f-128">如果不是, 它将调用 OWIN 中间件以对用户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="33b9f-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="33b9f-129">该`SignOut`操作将调用 OWIN 中间件以注销。</span><span class="sxs-lookup"><span data-stu-id="33b9f-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

<span data-ttu-id="33b9f-130">保存所做的更改并启动项目。</span><span class="sxs-lookup"><span data-stu-id="33b9f-130">Save your changes and start the project.</span></span> <span data-ttu-id="33b9f-131">单击 "登录" 按钮, 您应会被重定向`https://login.microsoftonline.com`到。</span><span class="sxs-lookup"><span data-stu-id="33b9f-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="33b9f-132">使用你的 Microsoft 帐户登录, 并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="33b9f-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="33b9f-133">浏览器重定向到应用程序, 并显示令牌。</span><span class="sxs-lookup"><span data-stu-id="33b9f-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="33b9f-134">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="33b9f-134">Get user details</span></span>

<span data-ttu-id="33b9f-135">首先, 创建一个新文件来保存所有 Microsoft Graph 调用。</span><span class="sxs-lookup"><span data-stu-id="33b9f-135">Start by creating a new file to hold all of your Microsoft Graph calls.</span></span> <span data-ttu-id="33b9f-136">右键单击 "解决方案资源管理器" 中的 " **graph-教程**" 文件夹, 然后选择 "**添加 > 新文件夹**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-136">Right-click the **graph-tutorial** folder in Solution Explorer, and choose **Add > New Folder**.</span></span> <span data-ttu-id="33b9f-137">将文件夹`Helpers`命名为。</span><span class="sxs-lookup"><span data-stu-id="33b9f-137">Name the folder `Helpers`.</span></span> <span data-ttu-id="33b9f-138">右键单击此新文件夹, 然后选择 "**添加 > 类 ...**"。命名该文件`GraphHelper.cs` , 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-138">Right click this new folder and choose **Add > Class...**. Name the file `GraphHelper.cs` and choose **Add**.</span></span> <span data-ttu-id="33b9f-139">将此文件的内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-139">Replace the contents of this file with the following code.</span></span>

```cs
using Microsoft.Graph;
using System.Net.Http.Headers;
using System.Threading.Tasks;

namespace graph_tutorial.Helpers
{
    public static class GraphHelper
    {
        public static async Task<User> GetUserDetailsAsync(string accessToken)
        {
            var graphClient = new GraphServiceClient(
                new DelegateAuthenticationProvider(
                    async (requestMessage) =>
                    {
                        requestMessage.Headers.Authorization =
                            new AuthenticationHeaderValue("Bearer", accessToken);
                    }));

            return await graphClient.Me.Request().GetAsync();
        }
    }
}
```

<span data-ttu-id="33b9f-140">这将`GetUserDetails`实现使用 MICROSOFT Graph SDK 调用`/me`终结点并返回结果的函数。</span><span class="sxs-lookup"><span data-stu-id="33b9f-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

<span data-ttu-id="33b9f-141">更新中`OnAuthorizationCodeReceivedAsync` `App_Start/Startup.Auth.cs`的方法以调用此函数。</span><span class="sxs-lookup"><span data-stu-id="33b9f-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="33b9f-142">首先, 将以下`using`语句添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="33b9f-142">First, add the following `using` statement to the top of the file.</span></span>

```cs
using graph_tutorial.Helpers;
```

<span data-ttu-id="33b9f-143">将现有`try`块替换`OnAuthorizationCodeReceivedAsync`为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

```cs
try
{
    string[] scopes = graphScopes.Split(' ');

    var result = await idClient.AcquireTokenByAuthorizationCode(
        scopes, notification.Code).ExecuteAsync();

    var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

    string email = string.IsNullOrEmpty(userDetails.Mail) ?
        userDetails.UserPrincipalName : userDetails.Mail;

    message = "User info retrieved.";
    debug = $"User: {userDetails.DisplayName}, Email: {email}";
}
```

<span data-ttu-id="33b9f-144">现在, 如果您保存更改并启动应用程序, 登录后您应该可以看到用户的姓名和电子邮件地址, 而不是访问令牌。</span><span class="sxs-lookup"><span data-stu-id="33b9f-144">Now if you save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="33b9f-145">存储令牌</span><span class="sxs-lookup"><span data-stu-id="33b9f-145">Storing the tokens</span></span>

<span data-ttu-id="33b9f-146">现在, 您可以获取令牌, 现在是时候实现将它们存储在应用程序中的方法了。</span><span class="sxs-lookup"><span data-stu-id="33b9f-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="33b9f-147">由于这是一个示例应用程序, 我们将使用此会话来存储令牌。</span><span class="sxs-lookup"><span data-stu-id="33b9f-147">Since this is a sample app, we'll use the session to store the tokens.</span></span> <span data-ttu-id="33b9f-148">实际应用程序将使用更可靠的安全存储解决方案, 就像数据库一样。</span><span class="sxs-lookup"><span data-stu-id="33b9f-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="33b9f-149">右键单击 "解决方案资源管理器" 中的 " **graph-教程**" 文件夹, 然后选择 "**添加 > 新文件夹**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-149">Right-click the **graph-tutorial** folder in Solution Explorer, and choose **Add > New Folder**.</span></span> <span data-ttu-id="33b9f-150">将文件夹`TokenStorage`命名为。</span><span class="sxs-lookup"><span data-stu-id="33b9f-150">Name the folder `TokenStorage`.</span></span> <span data-ttu-id="33b9f-151">右键单击此新文件夹, 然后选择 "**添加 > 类 ...**"。命名该文件`SessionTokenStore.cs` , 然后选择 "**添加**"。</span><span class="sxs-lookup"><span data-stu-id="33b9f-151">Right click this new folder and choose **Add > Class...**. Name the file `SessionTokenStore.cs` and choose **Add**.</span></span> <span data-ttu-id="33b9f-152">将此文件的内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="33b9f-152">Replace the contents of this file with the following code.</span></span>

```cs
using Microsoft.Identity.Client;
using Newtonsoft.Json;
using System.Security.Claims;
using System.Threading;
using System.Web;

namespace graph_tutorial.TokenStorage
{
    // Simple class to serialize into the session
    public class CachedUser
    {
        public string DisplayName { get; set; }
        public string Email { get; set; }
        public string Avatar { get; set; }
    }

    // Adapted from https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-v2
    public class SessionTokenStore
    {
        private static readonly ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);

        private readonly string userId = string.Empty;
        private readonly string cacheId = string.Empty;
        private readonly string cachedUserId = string.Empty;
        private HttpContext httpContext = null;
        private ITokenCache tokenCache;

        public SessionTokenStore(string userId, HttpContext httpcontext)
        {
            this.userId = userId;
            this.cacheId = $"{userId}_TokenCache";
            this.cachedUserId = $"{userId}_UserCache";
            this.httpContext = httpcontext;
        }

        public void Initialize(ITokenCache tokenCache)
        {
            this.tokenCache = tokenCache;
            this.tokenCache.SetBeforeAccess(BeforeAccessNotification);
            this.tokenCache.SetAfterAccess(AfterAccessNotification);
            Load();
        }

        public bool HasData()
        {
            return (httpContext.Session[cacheId] != null && ((byte[])httpContext.Session[cacheId]).Length > 0);
        }

        public void Clear()
        {
            httpContext.Session.Remove(cacheId);
        }

        private void Load()
        {
            sessionLock.EnterReadLock();
            tokenCache.DeserializeMsalV3((byte[])httpContext.Session[cacheId]);
            sessionLock.ExitReadLock();
        }

        private void Persist()
        {
            sessionLock.EnterReadLock();
            httpContext.Session[cacheId] = tokenCache.SerializeMsalV3();
            sessionLock.ExitReadLock();
        }

        private void BeforeAccessNotification(TokenCacheNotificationArgs args)
        {
            Load();
        }

        private void AfterAccessNotification(TokenCacheNotificationArgs args)
        {
            if (args.HasStateChanged)
            {
                Persist();
            }
        }

        public void SaveUserDetails(CachedUser user)
        {
            sessionLock.EnterReadLock();
            httpContext.Session[cachedUserId] = JsonConvert.SerializeObject(user);
            sessionLock.ExitReadLock();
        }

        public CachedUser GetUserDetails()
        {
            sessionLock.EnterReadLock();
            var cachedUser = JsonConvert.DeserializeObject<CachedUser>((string)httpContext.Session[cachedUserId]);
            sessionLock.ExitReadLock();
            return cachedUser;
        }
    }
}
```

<span data-ttu-id="33b9f-153">此代码将创建`SessionTokenStore`一个适用于 MSAL 库的`TokenCache`类的类。</span><span class="sxs-lookup"><span data-stu-id="33b9f-153">This code creates a `SessionTokenStore` class that works with the MSAL library's `TokenCache` class.</span></span> <span data-ttu-id="33b9f-154">此处的大部分代码都涉及将和反序列`TokenCache`化到会话。</span><span class="sxs-lookup"><span data-stu-id="33b9f-154">Most of the code here involves serializing and deserializing the `TokenCache` to the session.</span></span> <span data-ttu-id="33b9f-155">它还提供了用于将用户的详细信息序列化和反序列化到会话中的类和方法。</span><span class="sxs-lookup"><span data-stu-id="33b9f-155">It also provides a class and methods to serialize and deserialize the user's details to the session.</span></span>

<span data-ttu-id="33b9f-156">现在, 将以下`using`语句添加到`App_Start/Startup.Auth.cs`文件顶部。</span><span class="sxs-lookup"><span data-stu-id="33b9f-156">Now, add the following `using` statement to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

```cs
using graph_tutorial.TokenStorage;
using System.IdentityModel.Claims;
```

<span data-ttu-id="33b9f-157">现在, 更新`OnAuthorizationCodeReceivedAsync`函数以创建`SessionTokenStore`类的实例, 并将其提供给`ConfidentialClientApplication`对象的构造函数。</span><span class="sxs-lookup"><span data-stu-id="33b9f-157">Now update the `OnAuthorizationCodeReceivedAsync` function to create an instance of the `SessionTokenStore` class and provide that to the constructor for the `ConfidentialClientApplication` object.</span></span> <span data-ttu-id="33b9f-158">这将导致 MSAL 使用您的缓存实现来存储令牌。</span><span class="sxs-lookup"><span data-stu-id="33b9f-158">That will cause MSAL to use your cache implementation for storing tokens.</span></span> <span data-ttu-id="33b9f-159">将现有的 `OnAuthorizationCodeReceivedAsync` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="33b9f-159">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

```cs
private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
{
    var idClient = ConfidentialClientApplicationBuilder.Create(appId)
        .WithRedirectUri(redirectUri)
        .WithClientSecret(appSecret)
        .Build();

    var signedInUserId = notification.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
    var tokenStore = new SessionTokenStore(signedInUserId, HttpContext.Current);
    tokenStore.Initialize(idClient.UserTokenCache);

    try
    {
        string[] scopes = graphScopes.Split(' ');

        var result = await idClient.AcquireTokenByAuthorizationCode(
            scopes, notification.Code).ExecuteAsync();

        var userDetails = await GraphHelper.GetUserDetailsAsync(result.AccessToken);

        var cachedUser = new CachedUser()
        {
            DisplayName = userDetails.DisplayName,
            Email = string.IsNullOrEmpty(userDetails.Mail) ?
            userDetails.UserPrincipalName : userDetails.Mail,
            Avatar = string.Empty
        };

        tokenStore.SaveUserDetails(cachedUser);
    }
    catch (MsalException ex)
    {
        string message = "AcquireTokenByAuthorizationCodeAsync threw an exception";
        notification.HandleResponse();
        notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
    }
    catch (Microsoft.Graph.ServiceException ex)
    {
        string message = "GetUserDetailsAsync threw an exception";
        notification.HandleResponse();
        notification.Response.Redirect($"/Home/Error?message={message}&debug={ex.Message}");
    }
}
```

<span data-ttu-id="33b9f-160">若要汇总更改, 请执行以下操作:</span><span class="sxs-lookup"><span data-stu-id="33b9f-160">To summarize the changes:</span></span>

- <span data-ttu-id="33b9f-161">现在, 代码会将`ConfidentialClientApplication`默认用户令牌缓存替换为的实例`SessionTokenStore`。</span><span class="sxs-lookup"><span data-stu-id="33b9f-161">The code now replaces the `ConfidentialClientApplication`'s default user token cache with an instance of the  `SessionTokenStore`.</span></span> <span data-ttu-id="33b9f-162">MSAL 库将处理存储令牌的逻辑, 并在需要时对其进行刷新。</span><span class="sxs-lookup"><span data-stu-id="33b9f-162">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
- <span data-ttu-id="33b9f-163">现在, 该代码将从 Microsoft Graph 获取的用户详细信息`SessionTokenStore`传递给要存储在会话中的对象。</span><span class="sxs-lookup"><span data-stu-id="33b9f-163">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
- <span data-ttu-id="33b9f-164">成功后, 代码不再重定向, 它只是返回。</span><span class="sxs-lookup"><span data-stu-id="33b9f-164">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="33b9f-165">这样, OWIN 中间件便可以完成身份验证过程。</span><span class="sxs-lookup"><span data-stu-id="33b9f-165">This allows the OWIN middleware to complete the authentication process.</span></span>

<span data-ttu-id="33b9f-166">由于令牌缓存存储在会话中, 因此请在注销`SignOut`前更新`Controllers/AccountController.cs`中的操作以清除令牌存储。首先, 将以下`using`语句添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="33b9f-166">Since the token cache is stored in the session, update the `SignOut` action in `Controllers/AccountController.cs` to clear the token store before signing out. First, add the following `using` statement to the top of the file.</span></span>

```cs
using graph_tutorial.TokenStorage;
```

<span data-ttu-id="33b9f-167">然后, 将现有`SignOut`函数替换为以下函数。</span><span class="sxs-lookup"><span data-stu-id="33b9f-167">Then, replace the existing `SignOut` function with the following.</span></span>

```cs
public ActionResult SignOut()
{
    if (Request.IsAuthenticated)
    {
        string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, System.Web.HttpContext.Current);

        tokenStore.Clear();

        Request.GetOwinContext().Authentication.SignOut(
            CookieAuthenticationDefaults.AuthenticationType);
    }

    return RedirectToAction("Index", "Home");
}
```

<span data-ttu-id="33b9f-168">缓存的用户详细信息是应用程序中的每个视图都需要的内容, `BaseController`因此更新类以从会话加载此信息。</span><span class="sxs-lookup"><span data-stu-id="33b9f-168">The cached user details are something that every view in the application will need, so update the `BaseController` class to load this information from the session.</span></span> <span data-ttu-id="33b9f-169">打开`Controllers/BaseController.cs`并将以下`using`语句添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="33b9f-169">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

```cs
using graph_tutorial.TokenStorage;
using System.Security.Claims;
using System.Web;
using Microsoft.Owin.Security.Cookies;
```

<span data-ttu-id="33b9f-170">然后添加以下函数。</span><span class="sxs-lookup"><span data-stu-id="33b9f-170">Then add the following function.</span></span>

```cs
protected override void OnActionExecuting(ActionExecutingContext filterContext)
{
    if (Request.IsAuthenticated)
    {
        // Get the signed in user's id and create a token cache
        string signedInUserId = ClaimsPrincipal.Current.FindFirst(ClaimTypes.NameIdentifier).Value;
        SessionTokenStore tokenStore = new SessionTokenStore(signedInUserId, System.Web.HttpContext.Current);

        if (tokenStore.HasData())
        {
            // Add the user to the view bag
            ViewBag.User = tokenStore.GetUserDetails();
        }
        else
        {
            // The session has lost data. This happens often
            // when debugging. Log out so the user can log back in
            Request.GetOwinContext().Authentication.SignOut(CookieAuthenticationDefaults.AuthenticationType);
            filterContext.Result = RedirectToAction("Index", "Home");
        }
    }

    base.OnActionExecuting(filterContext);
}
```

<span data-ttu-id="33b9f-171">启动服务器并执行登录过程。</span><span class="sxs-lookup"><span data-stu-id="33b9f-171">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="33b9f-172">您应该最后返回到主页, 但 UI 应更改以指示您已登录。</span><span class="sxs-lookup"><span data-stu-id="33b9f-172">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

<span data-ttu-id="33b9f-174">单击右上角的用户头像以访问 "**注销**" 链接。</span><span class="sxs-lookup"><span data-stu-id="33b9f-174">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="33b9f-175">单击 "**注销**" 重置会话并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="33b9f-175">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="33b9f-177">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="33b9f-177">Refreshing tokens</span></span>

<span data-ttu-id="33b9f-178">此时, 您的应用程序具有访问令牌, 该令牌是在 API `Authorization`调用的标头中发送的。</span><span class="sxs-lookup"><span data-stu-id="33b9f-178">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="33b9f-179">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="33b9f-179">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="33b9f-180">但是, 此令牌的生存期较短。</span><span class="sxs-lookup"><span data-stu-id="33b9f-180">However, this token is short-lived.</span></span> <span data-ttu-id="33b9f-181">令牌在发出后会过期一小时。</span><span class="sxs-lookup"><span data-stu-id="33b9f-181">The token expires an hour after it is issued.</span></span> <span data-ttu-id="33b9f-182">这就是刷新令牌变得有用的地方。</span><span class="sxs-lookup"><span data-stu-id="33b9f-182">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="33b9f-183">刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。</span><span class="sxs-lookup"><span data-stu-id="33b9f-183">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="33b9f-184">由于应用程序使用的是 MSAL 库和`TokenCache`对象, 因此您无需实现任何令牌刷新逻辑。</span><span class="sxs-lookup"><span data-stu-id="33b9f-184">Because the app is using the MSAL library and a `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="33b9f-185">此`ConfidentialClientApplication.AcquireTokenSilentAsync`方法将为您执行所有逻辑。</span><span class="sxs-lookup"><span data-stu-id="33b9f-185">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="33b9f-186">它首先检查缓存的令牌, 如果它未过期, 它将返回。</span><span class="sxs-lookup"><span data-stu-id="33b9f-186">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="33b9f-187">如果它已过期, 则使用缓存的刷新令牌获取新的。</span><span class="sxs-lookup"><span data-stu-id="33b9f-187">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="33b9f-188">您将在下面的模块中使用此方法。</span><span class="sxs-lookup"><span data-stu-id="33b9f-188">You'll use this method in the following module.</span></span>