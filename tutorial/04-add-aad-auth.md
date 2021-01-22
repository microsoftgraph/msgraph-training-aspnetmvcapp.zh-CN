<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1a39a-101">在此练习中，你将扩展上一练习中的应用程序，以支持使用 Azure AD 进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="1a39a-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="1a39a-102">这是获取调用 Microsoft Graph API 所需的 OAuth 访问令牌所必需的。</span><span class="sxs-lookup"><span data-stu-id="1a39a-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="1a39a-103">在此步骤中，你将 OWIN 中间件和 [Microsoft 身份验证](https://www.nuget.org/packages/Microsoft.Identity.Client/) 库集成到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="1a39a-103">In this step you will integrate the OWIN middleware and the [Microsoft Authentication Library](https://www.nuget.org/packages/Microsoft.Identity.Client/) library into the application.</span></span>

1. <span data-ttu-id="1a39a-104">在解决方案资源管理器 **中右键单击图形教程** 项目，然后选择>**新项目...** 选择 **Web 配置文件，** 命名该文件 `PrivateSettings.config` ，然后选择"**添加"。**</span><span class="sxs-lookup"><span data-stu-id="1a39a-104">Right-click the **graph-tutorial** project in Solution Explorer and select **Add > New Item...**. Choose **Web Configuration File**, name the file `PrivateSettings.config` and select **Add**.</span></span> <span data-ttu-id="1a39a-105">用以下代码替换它的全部内容。</span><span class="sxs-lookup"><span data-stu-id="1a39a-105">Replace its entire contents with the following code.</span></span>

    ```xml
    <appSettings>
        <add key="ida:AppID" value="YOUR APP ID" />
        <add key="ida:AppSecret" value="YOUR APP PASSWORD" />
        <add key="ida:RedirectUri" value="https://localhost:PORT/" />
        <add key="ida:AppScopes" value="User.Read Calendars.Read" />
    </appSettings>
    ```

    <span data-ttu-id="1a39a-106">替换为 `YOUR_APP_ID_HERE` 应用程序注册门户中的应用程序 ID，并 `YOUR_APP_PASSWORD_HERE` 替换为生成的客户端密码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the client secret you generated.</span></span> <span data-ttu-id="1a39a-107">如果你的客户端密码包含与 `&` () ，请务必将它们替换为 `&amp;` `PrivateSettings.config` 。</span><span class="sxs-lookup"><span data-stu-id="1a39a-107">If your client secret contains any ampersands (`&`), be sure to replace them with `&amp;` in `PrivateSettings.config`.</span></span> <span data-ttu-id="1a39a-108">此外，请务必修改 `PORT` 值以 `ida:RedirectUri` 匹配应用程序的 URL。</span><span class="sxs-lookup"><span data-stu-id="1a39a-108">Also be sure to modify the `PORT` value for the `ida:RedirectUri` to match your application's URL.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="1a39a-109">如果你使用的是源代码管理（如 git），那么现在应该将文件从源代码管理中排除，以避免意外泄露应用 ID 和 `PrivateSettings.config` 密码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-109">If you're using source control such as git, now would be a good time to exclude the `PrivateSettings.config` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="1a39a-110">更新 `Web.config` 以加载此新文件。</span><span class="sxs-lookup"><span data-stu-id="1a39a-110">Update `Web.config` to load this new file.</span></span> <span data-ttu-id="1a39a-111">将 (`<appSettings>` 第 7 行) 替换为以下内容</span><span class="sxs-lookup"><span data-stu-id="1a39a-111">Replace the `<appSettings>` (line 7) with the following</span></span>

    ```xml
    <appSettings file="PrivateSettings.config">
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="1a39a-112">实现登录</span><span class="sxs-lookup"><span data-stu-id="1a39a-112">Implement sign-in</span></span>

<span data-ttu-id="1a39a-113">首先初始化 OWIN 中间件以对应用使用 Azure AD 身份验证。</span><span class="sxs-lookup"><span data-stu-id="1a39a-113">Start by initializing the OWIN middleware to use Azure AD authentication for the app.</span></span>

1. <span data-ttu-id="1a39a-114">右键单击解决方案 **资源管理器App_Start** 文件夹，然后选择"添加>**类..."。** 命名文件， `Startup.Auth.cs` 然后选择"**添加"。**</span><span class="sxs-lookup"><span data-stu-id="1a39a-114">Right-click the **App_Start** folder in Solution Explorer and select **Add > Class...**. Name the file `Startup.Auth.cs` and select **Add**.</span></span> <span data-ttu-id="1a39a-115">将全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-115">Replace the entire contents with the following code.</span></span>

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
    > <span data-ttu-id="1a39a-116">此代码使用值配置 OWIN 中间件，并 `PrivateSettings.config` 定义两个回调方法和 `OnAuthenticationFailedAsync` `OnAuthorizationCodeReceivedAsync` 。</span><span class="sxs-lookup"><span data-stu-id="1a39a-116">This code configures the OWIN middleware with the values from `PrivateSettings.config` and defines two callback methods, `OnAuthenticationFailedAsync` and `OnAuthorizationCodeReceivedAsync`.</span></span> <span data-ttu-id="1a39a-117">当登录过程从 Azure 返回时，将调用这些回调方法。</span><span class="sxs-lookup"><span data-stu-id="1a39a-117">These callback methods will be invoked when the sign-in process returns from Azure.</span></span>

1. <span data-ttu-id="1a39a-118">现在更新 `Startup.cs` 文件以调用 `ConfigureAuth` 该方法。</span><span class="sxs-lookup"><span data-stu-id="1a39a-118">Now update the `Startup.cs` file to call the `ConfigureAuth` method.</span></span> <span data-ttu-id="1a39a-119">将其中的全部内容 `Startup.cs` 替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-119">Replace the entire contents of `Startup.cs` with the following code.</span></span>

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

1. <span data-ttu-id="1a39a-120">向 `Error` 类添加 `HomeController` 操作，以将 `message` 和 `debug` 查询参数转换为 `Alert` 对象。</span><span class="sxs-lookup"><span data-stu-id="1a39a-120">Add an `Error` action to the `HomeController` class to transform the `message` and `debug` query parameters into an `Alert` object.</span></span> <span data-ttu-id="1a39a-121">打开 `Controllers/HomeController.cs` 并添加以下函数。</span><span class="sxs-lookup"><span data-stu-id="1a39a-121">Open `Controllers/HomeController.cs` and add the following function.</span></span>

    ```cs
    public ActionResult Error(string message, string debug)
    {
        Flash(message, debug);
        return RedirectToAction("Index");
    }
    ```

1. <span data-ttu-id="1a39a-122">添加控制器以处理登录。</span><span class="sxs-lookup"><span data-stu-id="1a39a-122">Add a controller to handle sign-in.</span></span> <span data-ttu-id="1a39a-123">右键单击解决方案资源管理器 **中的"控制器**"文件夹，然后选择">**控制器..."。** 选择 **MVC 5 控制器 - 空**，然后选择"**添加"。**</span><span class="sxs-lookup"><span data-stu-id="1a39a-123">Right-click the **Controllers** folder in Solution Explorer and select **Add > Controller...**. Choose **MVC 5 Controller - Empty** and select **Add**.</span></span> <span data-ttu-id="1a39a-124">命名控制器， `AccountController` 然后选择"**添加"。**</span><span class="sxs-lookup"><span data-stu-id="1a39a-124">Name the controller `AccountController` and select **Add**.</span></span> <span data-ttu-id="1a39a-125">将文件的全部内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-125">Replace the entire contents of the file with the following code.</span></span>

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

    <span data-ttu-id="1a39a-126">这将定义和 `SignIn` `SignOut` 操作。</span><span class="sxs-lookup"><span data-stu-id="1a39a-126">This defines a `SignIn` and `SignOut` action.</span></span> <span data-ttu-id="1a39a-127">`SignIn`该操作将检查请求是否已经过身份验证。</span><span class="sxs-lookup"><span data-stu-id="1a39a-127">The `SignIn` action checks if the request is already authenticated.</span></span> <span data-ttu-id="1a39a-128">如果没有，它将调用 OWIN 中间件来对用户进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="1a39a-128">If not, it invokes the OWIN middleware to authenticate the user.</span></span> <span data-ttu-id="1a39a-129">该操作 `SignOut` 调用 OWIN 中间件以注销。</span><span class="sxs-lookup"><span data-stu-id="1a39a-129">The `SignOut` action invokes the OWIN middleware to sign out.</span></span>

1. <span data-ttu-id="1a39a-130">保存更改并启动项目。</span><span class="sxs-lookup"><span data-stu-id="1a39a-130">Save your changes and start the project.</span></span> <span data-ttu-id="1a39a-131">单击登录按钮，应重定向到 `https://login.microsoftonline.com` 。</span><span class="sxs-lookup"><span data-stu-id="1a39a-131">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="1a39a-132">使用 Microsoft 帐户登录，并同意请求的权限。</span><span class="sxs-lookup"><span data-stu-id="1a39a-132">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="1a39a-133">浏览器重定向到应用，显示令牌。</span><span class="sxs-lookup"><span data-stu-id="1a39a-133">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="1a39a-134">获取用户详细信息</span><span class="sxs-lookup"><span data-stu-id="1a39a-134">Get user details</span></span>

<span data-ttu-id="1a39a-135">用户登录后，可以从 Microsoft Graph 获取其信息。</span><span class="sxs-lookup"><span data-stu-id="1a39a-135">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="1a39a-136">右键单击 **解决方案资源管理器中的图形教程** 文件夹，然后选择> **文件夹**。</span><span class="sxs-lookup"><span data-stu-id="1a39a-136">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="1a39a-137">命名文件夹 `Helpers` 。</span><span class="sxs-lookup"><span data-stu-id="1a39a-137">Name the folder `Helpers`.</span></span>

1. <span data-ttu-id="1a39a-138">右键单击此新文件夹，然后选择">**类..."。** 命名文件， `GraphHelper.cs` 然后选择"**添加"。**</span><span class="sxs-lookup"><span data-stu-id="1a39a-138">Right click this new folder and select **Add > Class...**. Name the file `GraphHelper.cs` and select **Add**.</span></span> <span data-ttu-id="1a39a-139">将此文件的内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-139">Replace the contents of this file with the following code.</span></span>

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
                        (requestMessage) =>
                        {
                            requestMessage.Headers.Authorization =
                                new AuthenticationHeaderValue("Bearer", accessToken);
                            return Task.FromResult(0);
                        }));

                return await graphClient.Me.Request().GetAsync();
            }
        }
    }
    ```

    <span data-ttu-id="1a39a-140">这将实现该 `GetUserDetails` 函数，该函数使用 Microsoft Graph SDK 调用 `/me` 终结点并返回结果。</span><span class="sxs-lookup"><span data-stu-id="1a39a-140">This implements the `GetUserDetails` function, which uses the Microsoft Graph SDK to call the `/me` endpoint and return the result.</span></span>

1. <span data-ttu-id="1a39a-141">更新 `OnAuthorizationCodeReceivedAsync` 方法以 `App_Start/Startup.Auth.cs` 调用此函数。</span><span class="sxs-lookup"><span data-stu-id="1a39a-141">Update the `OnAuthorizationCodeReceivedAsync` method in `App_Start/Startup.Auth.cs` to call this function.</span></span> <span data-ttu-id="1a39a-142">将以下 `using` 语句添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="1a39a-142">Add the following `using` statement to the top of the file.</span></span>

    ```cs
    using graph_tutorial.Helpers;
    ```

1. <span data-ttu-id="1a39a-143">将现有 `try` 块 `OnAuthorizationCodeReceivedAsync` 替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-143">Replace the existing `try` block in `OnAuthorizationCodeReceivedAsync` with the following code.</span></span>

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

1. <span data-ttu-id="1a39a-144">保存更改并启动应用，登录后应看到用户名和电子邮件地址，而不是访问令牌。</span><span class="sxs-lookup"><span data-stu-id="1a39a-144">Save your changes and start the app, after sign-in you should see the user's name and email address instead of the access token.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="1a39a-145">存储令牌</span><span class="sxs-lookup"><span data-stu-id="1a39a-145">Storing the tokens</span></span>

<span data-ttu-id="1a39a-146">现在，你可以获取令牌，是时候实现在应用中存储令牌的方法了。</span><span class="sxs-lookup"><span data-stu-id="1a39a-146">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="1a39a-147">由于这是一个示例应用，你将使用会话存储令牌。</span><span class="sxs-lookup"><span data-stu-id="1a39a-147">Since this is a sample app, you will use the session to store the tokens.</span></span> <span data-ttu-id="1a39a-148">实际应用将使用更可靠的安全存储解决方案，如数据库。</span><span class="sxs-lookup"><span data-stu-id="1a39a-148">A real-world app would use a more reliable secure storage solution, like a database.</span></span> <span data-ttu-id="1a39a-149">在本节中，你将：</span><span class="sxs-lookup"><span data-stu-id="1a39a-149">In this section, you will:</span></span>

- <span data-ttu-id="1a39a-150">实现令牌存储类，以在用户会话中序列化和存储 MSAL 令牌缓存和用户详细信息。</span><span class="sxs-lookup"><span data-stu-id="1a39a-150">Implement a token store class to serialize and store the MSAL token cache and the user's details in the user session.</span></span>
- <span data-ttu-id="1a39a-151">更新身份验证代码以使用令牌存储类。</span><span class="sxs-lookup"><span data-stu-id="1a39a-151">Update the authentication code to use the token store class.</span></span>
- <span data-ttu-id="1a39a-152">更新基本控制器类，以向应用程序的所有视图公开存储的用户详细信息。</span><span class="sxs-lookup"><span data-stu-id="1a39a-152">Update the base controller class to expose the stored user details to all views in the application.</span></span>

1. <span data-ttu-id="1a39a-153">右键单击 **解决方案资源管理器中的图形教程** 文件夹，然后选择> **文件夹**。</span><span class="sxs-lookup"><span data-stu-id="1a39a-153">Right-click the **graph-tutorial** folder in Solution Explorer, and select **Add > New Folder**.</span></span> <span data-ttu-id="1a39a-154">命名文件夹 `TokenStorage` 。</span><span class="sxs-lookup"><span data-stu-id="1a39a-154">Name the folder `TokenStorage`.</span></span>

1. <span data-ttu-id="1a39a-155">右键单击此新文件夹，然后选择">**类..."。** 命名文件， `SessionTokenStore.cs` 然后选择"**添加"。**</span><span class="sxs-lookup"><span data-stu-id="1a39a-155">Right click this new folder and select **Add > Class...**. Name the file `SessionTokenStore.cs` and select **Add**.</span></span> <span data-ttu-id="1a39a-156">将此文件的内容替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="1a39a-156">Replace the contents of this file with the following code.</span></span>

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

                    var userTenantId = user.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid").Value ??
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

1. <span data-ttu-id="1a39a-157">将以下 `using` 语句添加到文件 `App_Start/Startup.Auth.cs` 顶部。</span><span class="sxs-lookup"><span data-stu-id="1a39a-157">Add the following `using` statements to the top of the `App_Start/Startup.Auth.cs` file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    ```

1. <span data-ttu-id="1a39a-158">将现有的 `OnAuthorizationCodeReceivedAsync` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="1a39a-158">Replace the existing `OnAuthorizationCodeReceivedAsync` function with the following.</span></span>

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
    > <span data-ttu-id="1a39a-159">此新版本中的更改 `OnAuthorizationCodeReceivedAsync` 将执行以下操作：</span><span class="sxs-lookup"><span data-stu-id="1a39a-159">The changes in this new version of `OnAuthorizationCodeReceivedAsync` do the following:</span></span>
    >
    > - <span data-ttu-id="1a39a-160">代码现在使用类包装默认 `ConfidentialClientApplication` 用户令牌 `SessionTokenStore` 缓存。</span><span class="sxs-lookup"><span data-stu-id="1a39a-160">The code now wraps the `ConfidentialClientApplication`'s default user token cache with the `SessionTokenStore` class.</span></span> <span data-ttu-id="1a39a-161">MSAL 库将处理存储令牌的逻辑，并根据需要刷新令牌。</span><span class="sxs-lookup"><span data-stu-id="1a39a-161">The MSAL library will handle the logic of storing the tokens and refreshing it when needed.</span></span>
    > - <span data-ttu-id="1a39a-162">代码现在会将从 Microsoft Graph 获取的用户详细信息传递给 `SessionTokenStore` 要存储在会话中的对象。</span><span class="sxs-lookup"><span data-stu-id="1a39a-162">The code now passes the user details obtained from Microsoft Graph to the `SessionTokenStore` object to store in the session.</span></span>
    > - <span data-ttu-id="1a39a-163">成功后，代码将不再重定向，它仅返回。</span><span class="sxs-lookup"><span data-stu-id="1a39a-163">On success, the code no longer redirects, it just returns.</span></span> <span data-ttu-id="1a39a-164">这允许 OWIN 中间件完成身份验证过程。</span><span class="sxs-lookup"><span data-stu-id="1a39a-164">This allows the OWIN middleware to complete the authentication process.</span></span>

1. <span data-ttu-id="1a39a-165">更新 `SignOut` 操作以在退出之前清除令牌存储。将以下 `using` 语句添加到 。 `Controllers/AccountController.cs`</span><span class="sxs-lookup"><span data-stu-id="1a39a-165">Update the `SignOut` action to clear the token store before signing out. Add the following `using` statement to the top of `Controllers/AccountController.cs`.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    ```

1. <span data-ttu-id="1a39a-166">将现有的 `SignOut` 函数替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="1a39a-166">Replace the existing `SignOut` function with the following.</span></span>

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

1. <span data-ttu-id="1a39a-167">打开 `Controllers/BaseController.cs` 以下语句 `using` 并将其添加到文件顶部。</span><span class="sxs-lookup"><span data-stu-id="1a39a-167">Open `Controllers/BaseController.cs` and add the following `using` statements to the top of the file.</span></span>

    ```cs
    using graph_tutorial.TokenStorage;
    using System.Security.Claims;
    using System.Web;
    using Microsoft.Owin.Security.Cookies;
    ```

1. <span data-ttu-id="1a39a-168">添加以下函数。</span><span class="sxs-lookup"><span data-stu-id="1a39a-168">Add the following function.</span></span>

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

1. <span data-ttu-id="1a39a-169">启动服务器并完成登录过程。</span><span class="sxs-lookup"><span data-stu-id="1a39a-169">Start the server and go through the sign-in process.</span></span> <span data-ttu-id="1a39a-170">你最终应返回主页，但 UI 应更改以指示你已登录。</span><span class="sxs-lookup"><span data-stu-id="1a39a-170">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![登录后主页的屏幕截图](./images/add-aad-auth-01.png)

1. <span data-ttu-id="1a39a-172">单击右上角的用户头像以访问 **"注销"** 链接。</span><span class="sxs-lookup"><span data-stu-id="1a39a-172">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="1a39a-173">单击 **"注销** "可重置会话，并返回到主页。</span><span class="sxs-lookup"><span data-stu-id="1a39a-173">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![包含"注销"链接的下拉菜单屏幕截图](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="1a39a-175">刷新令牌</span><span class="sxs-lookup"><span data-stu-id="1a39a-175">Refreshing tokens</span></span>

<span data-ttu-id="1a39a-176">此时，应用程序具有访问令牌，该令牌在 API 调用 `Authorization` 标头中发送。</span><span class="sxs-lookup"><span data-stu-id="1a39a-176">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="1a39a-177">这是允许应用代表用户访问 Microsoft Graph 的令牌。</span><span class="sxs-lookup"><span data-stu-id="1a39a-177">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="1a39a-178">但是，此令牌是短期的。</span><span class="sxs-lookup"><span data-stu-id="1a39a-178">However, this token is short-lived.</span></span> <span data-ttu-id="1a39a-179">令牌在颁发后一小时过期。</span><span class="sxs-lookup"><span data-stu-id="1a39a-179">The token expires an hour after it is issued.</span></span> <span data-ttu-id="1a39a-180">刷新令牌在这里变得有用。</span><span class="sxs-lookup"><span data-stu-id="1a39a-180">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="1a39a-181">刷新令牌允许应用请求新的访问令牌，而无需用户重新登录。</span><span class="sxs-lookup"><span data-stu-id="1a39a-181">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="1a39a-182">由于应用使用 MSAL 库并序列化对象，因此不需要 `TokenCache` 实现任何令牌刷新逻辑。</span><span class="sxs-lookup"><span data-stu-id="1a39a-182">Because the app is using the MSAL library and serializing the `TokenCache` object, you do not have to implement any token refresh logic.</span></span> <span data-ttu-id="1a39a-183">`ConfidentialClientApplication.AcquireTokenSilentAsync`该方法将执行所有逻辑。</span><span class="sxs-lookup"><span data-stu-id="1a39a-183">The `ConfidentialClientApplication.AcquireTokenSilentAsync` method does all of the logic for you.</span></span> <span data-ttu-id="1a39a-184">它首先检查缓存的令牌，如果它未过期，它将返回它。</span><span class="sxs-lookup"><span data-stu-id="1a39a-184">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="1a39a-185">如果已过期，它将使用缓存的刷新令牌获取新的刷新令牌。</span><span class="sxs-lookup"><span data-stu-id="1a39a-185">If it is expired, it uses the cached refresh token to obtain a new one.</span></span> <span data-ttu-id="1a39a-186">您将在下面的模块中使用此方法。</span><span class="sxs-lookup"><span data-stu-id="1a39a-186">You'll use this method in the following module.</span></span>
