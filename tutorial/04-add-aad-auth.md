<!-- markdownlint-disable MD002 MD041 -->

在本练习中, 你将扩展上一练习中的应用程序, 以支持 Azure AD 的身份验证。 若要获取所需的 OAuth 访问令牌以调用 Microsoft Graph API, 这是必需的。 在此步骤中, 将 OWIN 中间件和[Microsoft 身份验证库](https://www.nuget.org/packages/Microsoft.Identity.Client/)库集成到应用程序中。

1. 右键单击 "解决方案资源管理器" 中的 "**教程**" 项目, 然后选择 "**添加 > 新项 ...**"。选择 " **Web 配置文件**", 命名`PrivateSettings.config`该文件, 然后选择 "**添加**"。 用以下代码替换它的全部内容。

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    将`YOUR_APP_ID_HERE`替换为应用程序注册门户中的应用程序 ID, `YOUR_APP_PASSWORD_HERE`并将替换为您生成的客户端密码。 如果你的客户端密码包含任何`&`& (), 请务必将其`&amp;`替换`PrivateSettings.config`为。 此外, 请务必修改的`PORT`值, 以`ida:RedirectUri`匹配您的应用程序的 URL。

    > [!IMPORTANT]
    > 如果您使用的是源代码管理 (如 git), 现在可以从源代码管理中排除`PrivateSettings.config`该文件, 以避免无意中泄漏您的应用程序 ID 和密码。

1. 更新`Web.config`以加载此新文件。 将`<appSettings>` (第7行) 替换为以下项

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a>实施登录

首先, 初始化 OWIN 中间件以使用应用程序的 Azure AD 身份验证。

1. 右键单击 "解决方案资源管理器" 中的 " **App_Start** " 文件夹, 然后选择 "**添加 > 类 ...**"。命名该文件`Startup.Auth.cs` , 然后选择 "**添加**"。 将整个内容替换为以下代码。

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

    > [!NOTE]
    > 此代码使用中`PrivateSettings.config`的值配置 OWIN 中间件, `OnAuthenticationFailedAsync`并定义两个回调方法`OnAuthorizationCodeReceivedAsync`和。 当登录进程从 Azure 返回时, 将调用这些回调方法。

1. 现在, 更新`Startup.cs`文件以调用`ConfigureAuth`方法。 将整个内容`Startup.cs`替换为以下代码。

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

1. `Error`向`HomeController`类中添加一个操作, 以将`message`和`debug`查询参数转换为`Alert`一个对象。 打开`Controllers/HomeController.cs`并添加以下函数。

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. 添加控制器以处理登录。 右键单击 "解决方案资源管理器" 中的 "**控制器**" 文件夹, 然后选择 "**添加 > 控制器 ...**"。选择 " **MVC 5 控制器-空**", 然后选择 "**添加**"。 命名控制器`AccountController`并选择 "**添加**"。 将文件的全部内容替换为以下代码。

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

    这将`SignIn`定义 and `SignOut`操作。 该`SignIn`操作将检查是否已对请求进行身份验证。 如果不是, 它将调用 OWIN 中间件以对用户进行身份验证。 该`SignOut`操作将调用 OWIN 中间件以注销。

1. 保存所做的更改并启动项目。 单击 "登录" 按钮, 您应会被重定向`https://login.microsoftonline.com`到。 使用你的 Microsoft 帐户登录, 并同意请求的权限。 浏览器重定向到应用程序, 并显示令牌。

### <a name="get-user-details"></a>获取用户详细信息

用户登录后, 可以从 Microsoft Graph 获取其信息。

1. 右键单击 "解决方案资源管理器" 中的 "**绘图-教程**" 文件夹, 然后选择 "**添加 > 新文件夹**"。 将文件夹`Helpers`命名为。

1. 右键单击此新文件夹, 然后选择 "**添加 > 类 ...**"。命名该文件`GraphHelper.cs` , 然后选择 "**添加**"。 将此文件的内容替换为以下代码。

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

    这将`GetUserDetails`实现使用 MICROSOFT Graph SDK 调用`/me`终结点并返回结果的函数。

1. 更新中`OnAuthorizationCodeReceivedAsync` `App_Start/Startup.Auth.cs`的方法以调用此函数。 将以下`using`语句添加到文件顶部。

    ```cs
    using graph_tutorial.Helpers;
    ```

1. 将现有`try`块替换`OnAuthorizationCodeReceivedAsync`为以下代码。

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

1. 保存所做的更改并启动应用程序, 登录后您应该可以看到用户的姓名和电子邮件地址, 而不是访问令牌。

## <a name="storing-the-tokens"></a>存储令牌

现在, 您可以获取令牌, 现在是时候实现将它们存储在应用程序中的方法了。 由于这是一个示例应用程序, 因此您将使用会话来存储令牌。 实际应用程序将使用更可靠的安全存储解决方案, 就像数据库一样。 在本节中, 您将执行以下操作:

- 实现令牌存储类以在用户会话中序列化并存储 MSAL 令牌缓存和用户的详细信息。
- 更新身份验证代码以使用令牌存储类。
- 更新基本控制器类, 以向应用程序中的所有视图公开存储的用户详细信息。

1. 右键单击 "解决方案资源管理器" 中的 "**绘图-教程**" 文件夹, 然后选择 "**添加 > 新文件夹**"。 将文件夹`TokenStorage`命名为。

1. 右键单击此新文件夹, 然后选择 "**添加 > 类 ...**"。命名该文件`SessionTokenStore.cs` , 然后选择 "**添加**"。 将此文件的内容替换为以下代码。

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT license.

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

        public class SessionTokenStore
        {
            private static readonly ReaderWriterLockSlim sessionLock = new ReaderWriterLockSlim(LockRecursionPolicy.NoRecursion);

            private HttpContext httpContext = null;
            private string tokenCacheKey = string.Empty;
            private string userCacheKey = string.Empty;

            public SessionTokenStore(ITokenCache tokenCache, HttpContext context, ClaimsPrincipal user)
            {
                httpContext = context;

                if (tokenCache != null)
                {
                    tokenCache.SetBeforeAccess(BeforeAccessNotification);
                    tokenCache.SetAfterAccess(AfterAccessNotification);
                }

                var userId = GetUsersUniqueId(user);
                tokenCacheKey = $"{userId}_TokenCache";
                userCacheKey = $"{userId}_UserCache";
            }

            public bool HasData()
            {
                return (httpContext.Session[tokenCacheKey] != null &&
                    ((byte[])httpContext.Session[tokenCacheKey]).Length > 0);
            }

            public void Clear()
            {
                sessionLock.EnterWriteLock();

                try
                {
                    httpContext.Session.Remove(tokenCacheKey);
                }
                finally
                {
                    sessionLock.ExitWriteLock();
                }
            }

            private void BeforeAccessNotification(TokenCacheNotificationArgs args)
            {
                sessionLock.EnterReadLock();

                try
                {
                    // Load the cache from the session
                    args.TokenCache.DeserializeMsalV3((byte[])httpContext.Session[tokenCacheKey]);
                }
                finally
                {
                    sessionLock.ExitReadLock();
                }
            }

            private void AfterAccessNotification(TokenCacheNotificationArgs args)
            {
                if (args.HasStateChanged)
                {
                    sessionLock.EnterWriteLock();

                    try
                    {
                        // Store the serialized cache in the session
                        httpContext.Session[tokenCacheKey] = args.TokenCache.SerializeMsalV3();
                    }
                    finally
                    {
                        sessionLock.ExitWriteLock();
                    }
                }
            }

            public void SaveUserDetails(CachedUser user)
            {

                sessionLock.EnterWriteLock();
                httpContext.Session[userCacheKey] = JsonConvert.SerializeObject(user);
                sessionLock.ExitWriteLock();
            }

            public CachedUser GetUserDetails()
            {
                sessionLock.EnterReadLock();
                var cachedUser = JsonConvert.DeserializeObject<CachedUser>((string)httpContext.Session[userCacheKey]);
                sessionLock.ExitReadLock();
                return cachedUser;
            }

            private string GetUsersUniqueId(ClaimsPrincipal user)
            {
                // Combine the user's object ID with their tenant ID

                if (user != null)
                {
                    var userObjectId = user.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value ??
                        user.FindFirst("oid").Value;

                    var userTenantId = user.FindFirst("http://schemas.microsoft.com/identity/claims/objectidentifier").Value ??
                        user.FindFirst("tid").Value;

                    if (!string.IsNullOrEmpty(userObjectId) && !string.IsNullOrEmpty(userTenantId))
                    {
                        return $"{userObjectId}.{userTenantId}";
                    }
                }

                return null;
            }
        }
    }
    ```

1. 将以下`using`语句添加到`App_Start/Startup.Auth.cs`文件顶部。

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. 将现有的 `OnAuthorizationCodeReceivedAsync` 函数替换为以下内容。

    ```cs
    private async Task OnAuthorizationCodeReceivedAsync(AuthorizationCodeReceivedNotification notification)
    {
        var idClient = ConfidentialClientApplicationBuilder.Create(appId)
            .WithRedirectUri(redirectUri)
            .WithClientSecret(appSecret)
            .Build();

        var signedInUser = new ClaimsPrincipal(notification.AuthenticationTicket.Identity);
        var tokenStore = new SessionTokenStore(idClient.UserTokenCache, HttpContext.Current, signedInUser);

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

    > [!NOTE]
    > 此新版本中的更改`OnAuthorizationCodeReceivedAsync`执行以下操作:
    >
    > - 代码现在使用`SessionTokenStore`类对`ConfidentialClientApplication`的默认用户令牌缓存进行包装。 MSAL 库将处理存储令牌的逻辑, 并在需要时对其进行刷新。
    > - 现在, 该代码将从 Microsoft Graph 获取的用户详细信息`SessionTokenStore`传递给要存储在会话中的对象。
    > - 成功后, 代码不再重定向, 它只是返回。 这样, OWIN 中间件便可以完成身份验证过程。

1. 更新`SignOut`操作以在注销前清除令牌存储。将以下`using`语句添加到的顶部`Controllers/AccountController.cs`。

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. 将现有的 `SignOut` 函数替换为以下内容。

    ```cs
    public ActionResult SignOut()
    {
        if (Request.IsAuthenticated)
        {
            var tokenStore = new SessionTokenStore(null,
                System.Web.HttpContext.Current, ClaimsPrincipal.Current);

            tokenStore.Clear();

            Request.GetOwinContext().Authentication.SignOut(
                CookieAuthenticationDefaults.AuthenticationType);
        }

        return RedirectToAction("Index", "Home");
    }
    ```

1. 打开`Controllers/BaseController.cs`并将以下`using`语句添加到文件顶部。

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. 添加以下函数。

    ```cs
    protected override void OnActionExecuting(ActionExecutingContext filterContext)
    {
        if (Request.IsAuthenticated)
        {
            // Get the user's token cache
            var tokenStore = new SessionTokenStore(null,
                System.Web.HttpContext.Current, ClaimsPrincipal.Current);

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

1. 启动服务器并执行登录过程。 您应该最后返回到主页, 但 UI 应更改以指示您已登录。

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. 单击右上角的用户头像以访问 "**注销**" 链接。 单击 "**注销**" 重置会话并返回到主页。

    ![带有 "注销" 链接的下拉菜单的屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>刷新令牌

此时, 您的应用程序具有访问令牌, 该令牌是在 API `Authorization`调用的标头中发送的。 这是允许应用代表用户访问 Microsoft Graph 的令牌。

但是, 此令牌的生存期较短。 令牌在发出后会过期一小时。 这就是刷新令牌变得有用的地方。 刷新令牌允许应用在不要求用户重新登录的情况下请求新的访问令牌。

由于应用程序使用的是 MSAL 库并将`TokenCache`对象序列化, 因此您无需实现任何令牌刷新逻辑。 此`ConfidentialClientApplication.AcquireTokenSilentAsync`方法将为您执行所有逻辑。 它首先检查缓存的令牌, 如果它未过期, 它将返回。 如果它已过期, 则使用缓存的刷新令牌获取新的。 您将在下面的模块中使用此方法。
