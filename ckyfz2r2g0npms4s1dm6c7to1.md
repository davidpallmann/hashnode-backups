## Hello, Cognito + OIDC!

#### This episode: Amazon Cognito and OpenID Connect. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll use Amazon Cognito to secure a "Hello, Cloud" `ASP.NET` project using OpenID Connect and federated identity with Google. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Cognito and OpenID Connect: What are they, and why use them?

[Amazon Cognito](https://aws.amazon.com/cognito/) provides authentication, authorization, and user management for web and mobile apps. We previously introduced Amazon Cognito in [Hello, Cognito!](https://davidpallmann.hashnode.dev/hello-cognito) and explained user pools (for identity verification) and identity pools (for authorization). Our prior Cognito post studied one scenario, authenticating against Cognito from an ASP.NET MVC application using the Amazon Cognito Identity Provider. This time, our use case is authenticating via OpenID Connect.

[OpenID Connect](https://openid.net/connect/) (OIDC) is "a simple identity layer on top of the OAuth 2.0 protocol". Your app can use OIDC to communicate with Cognito whether you're authenticating against a Cognito user pool or a federated identity provider such as Microsoft Azure Active Directory, Okta, Ping Identity, or Salesforce. Users can sign in to a Cognito-managed identity, or to a federated identity provider. 

Cognito provides a hosted UI for user sign-in and sign-up, which you can customize. When you use OIDC to connect to Cognito, the user lands on a Cognito sign-in page. The user can sign in to a Cognito-managed identity (user pool).

![diagram-userpool.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642117243452/yMebNpNmh.png)

If you've configured Cognito with additional identity providers, such as Salesforce, the Cognito sign-in page will offer the option to sign in through those provider(s). The Cognito user pool takes care of token handling and managing authenticated users from all identity providers. This allows your application to use just one set of user tokens.

![diagram-federated.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642117747269/VyBC0s9DL.png)

From ASP.NET, your application can connect with Cognito in several different ways. Our focus today is OpenID Connect. 

| Method | Packages |
| --------- | ----- |
| Cognito Identity Provider | Amazon.AspNetCore.Identity.Cognito |
| | Amazon.Extensions.CognitoAuthentication |
| OpenID Connect | Microsoft.AspNetCore.Authentication.OpenIdConnect |
| | Microsoft.Identity.Web | 

# Our Hello, Cognito OIDC Project

We previously introduced Amazon Cognito in [Hello, Cognito!](https://davidpallmann.hashnode.dev/hello-cognito) and studied one scenario, authenticating against Cognito from an `ASP.NET` MVC application using the Amazon Cognito Identity Provider. This time, our use case is authenticating to Cognito from an `ASP.NET` web project using OpenID Connect. 

We're going to create an `ASP.NET` MVC web application that authenticates with Cognito using OIDC. Users will be able to sign in with a Cognito identity or a Google identity. The sign-in dialogs will be provided by Cognito and Google. 

[source code](https://github.com/davidpallmann/hello-cognito-oidc)

![signin-google-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642260509792/xWlsww0YQ.png)

![diagram_hello-cognito-oidc.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642260473763/kf-EybVai.png)

# One-time Setup

To experiment with Cognito and .NET, you will need:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). [Configure ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) the toolkit to access your AWS account and create an IAM user.

## Step 1: Set Permissions for the AWS Toolkit User

In order to perform this tutorial, your AWS Toolkit User / default AWS profile needs the necessary permissions for Cognito operations. In this step, you'll update permissions for your AWS Toolkit for Visual Studio user. 

Important: You should always follow the principle of [least privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) when it comes to security. This blog series assumes you are working in your own developer account for learning purposes and not working with anything production-related. If you're using an organization-owned AWS account, you should coordinate with your security principal on IAM settings. 

1. Sign in to the [AWS console](https://aws.amazon.com/console/). Select the region at top right you want to be working in. We're using **US West (N. California)**.
2. Navigate to the **Identity and Access Management (IAM)** area of the AWS Console. You can enter **iam** in the search box to find it.
3. Click **Users** on the left panel and select the username you use with the AWS Toolkit for Visual Studio. You created this user when you installed and configured the toolkit.
4. If not already assigned, add the built-in **PowerUserAccess** policy. The  [PowerUserAccess](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html) policy provides developers full access to AWS services and resources, but does not allow management of users and groups.

Your AWS Toolkit for Visual Studio user now has the permissions it needs to perform Cognito operations.

## Step 2: Create a .NET Web Application

In this step you'll create an `ASP.NET` Web API project, then update it to communicate with Amazon Cognito using OIDC.

1. Open a command/terminal window and CD to a development folder.

2. Enter the dotnet new command below.

    ```none
dotnet new webapp -n hello-cognito-oidc
```

    ![02-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642118863257/--CoTYLL5.png)

3. Open the hello-cognito-oidc project in Visual Studio.

4. Press F5 to build and run the application. A simple welcome page opens in a browser window. Note the application's web address, which will be needed in the next step.

5. Stop the program.

## Step 3: Create a Cognito User Pool

In this step you'll create a new Cognito user pool in the AWS Console. 

1. Sign in to the AWS console and select the region you want to work in at top right. We're using **N. California**.

2. Navigate to **Amazon Cognito**. You can enter **cognito** in the search bar.

3. On the left panel, select **User pools** and click **Create user pool**. You will now go through several wizard pages to define your user pool.

4. On the Step 1 Configure sign-in experience page, enter/select the following and click **Next**:

    a. Authentication providers - Provider types: select **Federated identity providers**.

    b. Cognito user pool sign-in options: check **User name**, **Email**, and **Phone number**.

    c. Federated sign-in options: check **Google**.

    ![03-aws-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642178018169/czo9YQLHQ.png)

5. On the Step 2 Configure security requirements page, enter/select the following and click **Next**:

    a. Password policy - Password policy mode: select **Cognito defaults**.

    b. Multi-factor authentication - MFA enforcement: select **Optional MFA**.

    c. MFA methods - check **SMS message**.

    ![03-aws-create-user-pool-step2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642160363383/j12aw6dlv.png)

6. On the Step 3 Configure sign-up experience page, click **Next**.

7. On the Step 4 Configure message delivery page, enter/select the following and click **Next**:

    a. Email - Email provider: select **Send email with Cognito**.

    d. SMS - IAM role - select **Create a new IAM role**.

    e. IAM role name - enter a role name such as **CognitoSMS**.

    ![03-aws-create-user-pool-step4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642160845838/zAAAYn4qt.png)

    ![03-aws-create-user-pool-step4B.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642160872305/IWiSgAPiK2.png)

8. On the Step 5 Connect federated identity providers page, click **Skip for now** and **Next**.

    ![03-aws-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642253105180/qCGC9PEEi.png)

9. On the Step 6 Integrate your app page, enter/select the following and click **Next**:

    a. User pool name: enter **HelloCognitoOIDC**.

    b. Domain - Domain type: select **Use a Cognito domain**.

    c. Cognito domain: enter a domain name. We're using **hellocogoidc**. Make a note of the full domain name. Our is `https://hellocogoidc.auth.us-west-1.amazoncognito.com`.

    ![03-aws-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642253688101/dh-AwUffU.png)

    b. Initial app client - App type: select **Public client**.

    c. App client name: enter **HelloCognitoWeb**.

    d. Client secret: select **Generate a client secret - Recommended**.

    e. Under Allowed callback URLs, enter the .NET application's web address URL captured in the previous step plus a suffix of **/signin-oidc**: `https://localhost:[port]/signin-oidc`. This is the application path that processes a successful OIDC sign-in. 

    ![03-aws-create-user-pool-step6c.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642206704226/evWQHLoWE.png)

    f. Add a sign-out URL. Expand the Advanced app client settings section and click **Add sign-out URL**. Enter the .NET application's web address URL captured in the previous step plus a suffix of **/SignedOut**: `https://localhost:[port]/SignedOut`. This is the application page a sign-out action redirects to. 

    ![03-aws-06-d.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642254240115/EVLMmrdY-.png)

10. On the Step 7 Review and create page, confirm the options match the above instructions, then click **Create user pool** at the bottom of the page. The user pool is created.

    ![userpool-created.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642254464698/mRGNSSrtv.png)

11. Click the User pool name to view its detail. Select the **App integration tab** and record the following:

    a. Record the User pool ID (User pool overview panel).

    b. Click the **HelloCognitoWeb** client Id at bottom of page to view its detail.

    c. Record the Client ID (App client information panel).

    d. Capture the Client secret by clicking the **Show client secret** toggle.

You now have a Cognito user pool named HelloCognitoOIDC. 

## Step 4: Update the .NET Application for Authentication

In this step, you'll configure the .NET application for authentication. You'll add or update packages, Cognito configuration settings, start-up code, a main page, a signout page, and a signed out page.

1. In Visual Studio Solution Explorer, right-click the hello-cognito-oidc project and select **Manage NuGet Packages...**. 

    a. Find and install the **Microsoft.AspNetCore.Authentication.OpenIdConnect** package. 

    b. Find and install the **Microsoft.Identity.Web** package.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642118102033/gm-IqiiTI.png)

2. Open appsettings.Development.json and replace it with the code below at the end of this step. Replace [client-id], [client-secret], [region], [user-pool-id], and [cognito-domain] with the values you recorded in the previous step.

3. Open Program.cs in the editor and replace it with the code below. We replace the generated code, which uses C# minimal code (top level statements), with a traditional body so we can define multiple methods. To provide the OIDC mechanics, authentication is added with cookie and OpenID authentication. Cognito client and URL values are loaded from configuration. A sign-out redirect method is defined, OnRedirectToIdentityProviderForSignOut, which will process a Cognito sign-out, then redirect to a signed out page in our app.

4. Open Shared/_Layout.cshtml in the editor, and replace with the code below. If the user is authenticated, we include a Logout navigation option.  

5. Open index.cshtml in the editor, and replace with the code below. If the user is authenticated, we retrieve claims (username and email address) and show the username on the page.

6. Open index.cshtml.cs in the editor, and replace with the code below. We add an [Authorize] attribute to the class. The home page will initiate a sign-in if the user is not authorized.

7. In Solution Explorer, right-click Pages and select Add > Razor Page. Add a new empty Razor page named SignOut.cshtml. Open SignOut.cshtml in the editor, and replace with the code below. This is the page that the Logout option on the home page will link to.

8. Open Signout.cshtml.cs in the editor, and replace with the code below. 

9. In Solution Explorer, right-click Pages and select Add > Razor Page. Add a new empty Razor page named SignedOut.cshtml. Open SignedOut.cshtml.cs in the editor, and replace with the code below. This is the page that Cognito will redirect to after signing out a user.

10. Open SignedOut.cshtml.cs in the editor, and replace with the code below. 

11. Build the project and ensure it compiles without error.

appSettings.Development.json

```json
{
  "DetailedErrors": true,
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "Authentication": {
    "Cognito": {
      "ClientId": "[client-id]",
      "ClientSecret": "[client-secret]",
      "IncludeErrorDetails": true,
      "MetadataAddress": "https://cognito-idp.[region].amazonaws.com/[user-pool-id]/.well-known/openid-configuration",
      "RequireHttpsMetadata": false,
      "ResponseType": "code",
      "SaveToken": true,
      "TokenValidationParameters": {
        "ValidateIssuer": true
      },
      "LogoutEndpoint": "[cognito-domain]/logout",
      "LogoutRelPath": "/SignedOut"
    }
  }
}
```
Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.IdentityModel.Tokens;

namespace hello_cognito_oidc;

public class Program
{
    static IConfiguration Configuration;
    static async Task Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        Configuration = builder.Configuration;

        // Add services to the container.
        builder.Services.AddRazorPages();

        builder.Services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = CookieAuthenticationDefaults.AuthenticationScheme;
            options.DefaultSignInScheme = CookieAuthenticationDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
            options.DefaultSignOutScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        })
        .AddCookie()
        .AddOpenIdConnect(options =>
        {
            options.ResponseType = Configuration["Authentication:Cognito:ResponseType"];
            options.MetadataAddress = Configuration["Authentication:Cognito:MetadataAddress"];
            options.ClientId = Configuration["Authentication:Cognito:ClientId"];
            options.ClientSecret = Configuration["Authentication:Cognito:ClientSecret"];
            options.SaveTokens = true;
            options.Events = new OpenIdConnectEvents()
            {
                OnRedirectToIdentityProviderForSignOut = OnRedirectToIdentityProviderForSignOut
            };
        });

        var app = builder.Build();

        // Configure the HTTP request pipeline.
        if (!app.Environment.IsDevelopment())
        {
            app.UseExceptionHandler("/Error");
            // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
            app.UseHsts();
        }

        app.UseHttpsRedirection();
        app.UseStaticFiles();

        app.UseRouting();

        app.UseAuthentication();
        app.UseAuthorization();

        app.MapRazorPages();

        app.Run();
    }

    private static Task OnRedirectToIdentityProviderForSignOut(RedirectContext context)
    {
        context.ProtocolMessage.Scope = "openid";
        context.ProtocolMessage.ResponseType = "code";

        var logoutEndpoint = Configuration["Authentication:Cognito:LogoutEndpoint"];
        var clientId = Configuration["Authentication:Cognito:ClientId"];
        var logoutUrl = $"{context.Request.Scheme}://{context.Request.Host}{Configuration["Authentication:Cognito:LogoutRelPath"]}";
        context.ProtocolMessage.IssuerAddress = $"{logoutEndpoint}?client_id={clientId}&logout_uri={logoutUrl}&redirect_uri={logoutUrl}";

        // delete cookies
        context.Properties.Items.Remove(CookieAuthenticationDefaults.AuthenticationScheme);
        // close openid session
        context.Properties.Items.Remove(OpenIdConnectDefaults.AuthenticationScheme);

        return Task.CompletedTask;    }
}
```

_Layout.cshtml

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - hello_cog</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-area="" asp-page="/Index">hello_cog</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
                        aria-expanded="false" aria-label="Toggle navigation">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-page="/Index">Home</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-area="" asp-page="/Privacy">Privacy</a>
                        </li>
                        @if (User.Identity.IsAuthenticated)
                        {
                            <li class="nav-item">
                                <a class="nav-link text-dark" asp-area="" asp-page="/SignOut">Logout</a>
                            </li>
                        }
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>

    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; 2022 - hello_cog - <a asp-area="" asp-page="/Privacy">Privacy</a>
        </div>
    </footer>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>

    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

index.cshtml

```html
@page
@using System.Security.Claims
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

@if(User.Identity.IsAuthenticated)
{
    <p>
        Authenticated!
    </p>
    <p>
        Username: @User.Claims.FirstOrDefault(c => c.Type=="cognito:username")?.Value
    </p>
    <p>
        Email address: @User.Claims.FirstOrDefault(c => c.Type=="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress")?.Value
    </p>
}

<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>

    @if (User.Identity.IsAuthenticated)
    {
        var identity = User.Identity as ClaimsIdentity; 

        string username = identity.Claims.FirstOrDefault(c => c.Type == "cognito:username")?.Value;       
        <h1>Welcome @username</h1>

    }
</div>
```

index.cshtml.cs

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace hello_cognito_oidc.Pages;

[Authorize]
public class IndexModel : PageModel
{
    private readonly ILogger<IndexModel> _logger;

    public IndexModel(ILogger<IndexModel> logger)
    {
        _logger = logger;
    }

    public void OnGet()
    {

    }
}
```

SignOut.cshtml

```html
@page
@model SignOutModel
@{
    ViewData["Title"] = "SignOut";
}
<h1>@ViewData["Title"]</h1>

<p>Signing you out.</p>

```

SignOut.cshtml.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authentication.OpenIdConnect;

namespace hello_cognito_oidc.Pages
{
    public class SignOutModel : PageModel
    {
        private readonly ILogger<SignOutModel> _logger;

        public SignOutModel(ILogger<SignOutModel> logger)
        {
            _logger = logger;
        }

        public IActionResult OnGet()
        {
            var callbackUrl = Url.Page("/SignedOut", pageHandler: null, values: null, protocol: Request.Scheme);
            return SignOut(
                new AuthenticationProperties { RedirectUri = callbackUrl },
                CookieAuthenticationDefaults.AuthenticationScheme, OpenIdConnectDefaults.AuthenticationScheme
            );
        }
    }
}

```

SignedOut.cshtml

```html
@page
@model SignedOutModel
@{
    ViewData["Title"] = "SignedOut";
}
<h1>@ViewData["Title"]</h1>

<p>You have been signed out.</p>
```

SignedOut.cshtml.cs

```csharp
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;

namespace hello_cognito_oidc.Pages
{
    public class SignedOutModel : PageModel
    {
        private readonly ILogger<SignedOutModel> _logger;

        public SignedOutModel(ILogger<SignedOutModel> logger)
        {
            _logger = logger;
        }

    }
}
```

## Step 5: Run the Program and Test Cognito Sign-in

In this step we'll run the .NET application and test user authentication and registration with Cognito. 

1. In Visual Studio, press **F5** to debug run the program.

2. You are redirected to a sign-in screen provided by Cognito. Click **Sign up** to add a new user.

    ![signin-cognito.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642180995211/oNcVjFNOQ.png)

5. On the Sign up page, enter a username email, and password. Click **Sign up**.

    ![signup-cognito.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642181265709/bG11ZY1cy.png)

6. You are next prompted to enter a code. Go to the email account you registered with, where there should be a confirmation message with a code. 

    ![06_verification_mail.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885526403/yoZpnKy6WD.png)

7. Enter the verification code from the email on the web page and click **Confirm Account**.  You are returned to the welcome page, this time with your username and email shown on the page

    ![signup-confirm-email-code.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642181509857/RiX-fTBYZ.png)

    ![welcome_signed-in.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642181705215/MCbA9b4HW.png)

8. To test logout, click **Logout** at top right. You are signed out of Cognito and redirected to the app's SignedOut page. To log in again, click Home and you are taken back to Cognito's sign-in screen. 

    ![welcome_signed-out.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642181771664/lwxQEMby-.png)

9. Let's view the user in the AWS console. Navigate to Amazon Cognito > User pools > HelloCognitoOIDC and click the **Users** tab. The user you just registered is listed, along with their email address and confirmation status. You can click on the username for more details.

    ![06_view_user_aws.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642256899870/Tzn3Q9U--.png)

10. Finally, let's test forgotten password functionality. Click **Logout**, then **Home**. On the sign-in page, click **Forgot your password?**. On the Forgot your password? page, enter the username you registered with and click **Reset my password**. 

    ![forgot-password.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642182078187/ily4cCWP_.png)

10. You'll receive a password verification email. Open the email and copy the verification code. On the reset password page, enter your verification code, a new password, and password confirmation. Then click **Change password**. Your password is reset, which you can test by signing in again.

    ![forgot-password-code.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642182206413/hffwbBJCm.png)

11. Stop the program.

Congratulations! You have added authentication with a Cognito user pool to an `ASP.NET` 6 web application using OIDC! 

However, we're not done yet. Next, we'd like to add a second scenario, identity federation to Google.

## Step 6: Configure Google Project

In this step, you'll create a Google developer project and configure it for federation credentials for Cognito and your application.

1. Log into the Google Cloud console at [https://console.cloud.google.com/](https://console.cloud.google.com/).

2. At the top of the page, click **Select a Project**, then **New Project**. 

3. Enter a project name (we're using **HelloCognitoOIDC**) and create a project.

4. Click **Select Project** to select the project.

5. Navigate to **APIs & Services**, then select **Credentials**

6. Click the **+ Create Credentials** link at the top of the page and 
select **OAuth client ID**.

7. On the Create OAuth client ID page, click **Configure Consent Screen** and enter the following.

    a. Select **External** and click **Create**.

    ![configure-google-oauth2-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642381085409/dm7Eff_Yd.png)

8. On the Edit app registration page, enter the following, then click **Save and Continue**:

    a. App information - App name: **HelloCognitoOIDC**

    b. User support email: your email address

    c. App logo: optionally, you may select an image for your sign-in screen

    d. App domain - application home page: leave blank

    e. Application privacy policy link: leave blank

    f. Application terms of service link: leave blank

    g. Authorized domains: click **+ Add Domain** and enter **amazoncognito.com**.

    h. Developer contact information - email addresses: enter your email address.

    ![configure-google-oauth2-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642257170162/MlKMctdM5.png)

9. click **Save and Continue** two more times to complete the app registration.

10. Select **Credentials** from the left panel.

11. Click **+ Create Credentials** > **OAuth client ID**.

    ![configure-google-oauth2-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642381243233/dAWGO2WM5.png)

12. On the Create OAuth client ID page, enter/select the following:

    a. Application type: **Web application**.

    b. Name: **HelloCognitoOIDC**.

    c. Authorized JavaScript origins: click **+ Add URI** and enter your user pool domain, which you should have captured during Step 3 #9. Ours is **https://hellocogoidc.auth.us-west-1.amazoncognito.com**

    d. Authorized redirect URIs: click **+ Add URI** and enter your user pool domain, suffixed with **/oauth2/idpresponse**. Ours is **https://hellgogoidc.auth.us-west-1.amazoncognito.com/oauth2/idpresponse**.

    e. click **Create**.

    ![configure-google-oauth2-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642257379433/OPvGUWhiF.png)

13. An OAuth client created dialog will appear. 

    a. Copy and record Your Client ID.

    b. Copy and record Your Client Secret.

    c. Click OK.

    ![configure-google-oauth2-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642202136883/jhJP3FGzM.png)

## Step 7: Configure Cognito for Google Identity Provider

In this step, you'll configure the Google identity provider in your Cognito user pool.

1. Back in the AWS console, return back to the Amazon Cognito > User Pools page and click **HelloCognitoOIDC** to view its detail.

2. Select the **Sign-in experience tab**, then click **Add identity provider** on the Federated identity provider sign-in panel.

3. Click **Google** and click **Add identity provider**.

4. In the Google panel, enter:

    a. Client ID: the Google Client ID you recorded in the previous step.

    b. Client Secret: the Google client secret you recorded in the previous step.

    c. Authorized scopes, enter **profile email openid**.

    d. Map attributes between Google and your user pool: associate user pool attribute **email** with Google attribute **email**, and user pool attribute **username** with Google attribute **sub**.

    e. Click **Add identity provider**. On the user pools > HelloCognitoOIDC page, on the Federated identity provider sign-in panel, Google should now be listed.

    ![add_identity_provider_google.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642258059784/pWbL3N69U.png)

5. Select the **App integration** tab.

6. Select the **HelloCognitoWeb** app client to view its detail

7. On the Hosted UI panel, click **Edit**.

    a. Under Identity providers, add **Google**.

    b. Click **Save changes** at bottom.

That was a lot to configure, and I hope you were careful doing it. Now for the payoff, seeing it work.

## Step 8: Run the Program and Test Google Sign-in

In this step we'll run our program again—unchanged from our earlier Cognito sign-in test—and see that we can now also sign-in via Google. 

1. In Visual Studio, press F5 to run the program. 

2. If you are not taken to a sign-in screen (already signed in), click Logout and then Home.

3. You should now see a slightly different Cognito sign-in screen that offers a Continue with Google option. Users can now authenticate with the Cognito user pool, as in the past, or with a Google identity. 

    ![signin-google-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642197110892/QgoEd3JLk.png)

    Note that sign-ins are limited to your own Google account. That's because the Google project you created is in a Testing status. As Google explains [here](https://support.google.com/cloud/answer/10311615?hl=en#publishing-status), "The non-production Google project will only allow the owning developer to sign in, plus any addition user accounts you configure." If you want other accounts besides your own to be able to sign in, go back to the Google console, select your project, navigate to APIs & Services, click OAuth consent screen, and add test users.

    ![google-pub-status.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642201559277/dh-cbOdnL.png)

4. Click **Continue with Google** and select/confirm a Google account.

    ![signin-google-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642197357076/yBQCebbDP.png)

5. You are directed back to the application welcome page. Your Google user ID and email address are displayed on the page.

    ![signin-google-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642197427539/ua0gNtyW2.png)

6. Test sign out by clicking Logout. To log in again, click Home.

Congratulations are really in order now. You've not only learned how to connect with Cognito by OIDC, you've also federated identity with Google. Give yourself a big pat on the back.

## Step 9: Shut It Down

When you're done with Hello, Cognito, delete your Cognito user pool. You don't want to accrue charges for something you aren't using.

1. In the AWS Console, navigate to Amazon Cognito > User pools.
2. Click the **HelloCognitoOIDC** user pool to view its detail.
3. Select the App Integration tab.
4. Under Domain, select **Delete Cognito domain** from the Actions drop-down and confirm.
5. Under App client list, select the **HelloCognitoWeb** app client, click **Delete**, and confirm.
6. Navigate back to Amazon Cognito > User pools.
7. Select the **HelloCognitoOIDC** user pool, click **Delete**, and confirm.

# Where to Go From Here

Security is a heavy responsibility, and addressing it means analyzing scenarios and making many choices. Amazon Cognito's support for common identity management standards and ability to federate to identity providers covers a wealth of scenarios.

In this tutorial, we used OpenID Connect and looked at two Cognito authentication scenarios: user authentication to Cognito user pool identities, and identity federation with Google. You set up an Amazon Cognito user pool, created an `ASP.NET` 6 web application for OIDC authentication to Cognito, and tested it. You saw Cognito sign-in dialogs handle new user registration, login, logout, and forgot password. You then configured Google and Cognito to federate identity, and saw that work as well without having to change the .NET application. Although the Cognito sign-in prompts were very generic, you do have the ability to customize their look with an image and CSS.

So far, we've limited ourselves to authentication with Cognito. In a future blog post we'll also explore authorization.

# Further Reading

AWS DOCUMENTATION

[Amazon Cognito](https://aws.amazon.com/cognito/)

[Getting Started with Amazon Cognito](https://aws.amazon.com/cognito/getting-started/)

[Adding user pool sign-in through a third party](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-identity-federation.html)

[Adding OIDC identity providers to a user pool](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-oidc-idp.html)

[Amazon Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)

[Customizing the built-in sign-in and sign-up webpages](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-app-ui-customization.html)

[.NET on AWS Developer Center](https://aws.amazon.com/developer/language/net/)

Code Samples

[ASP.NET MVC .NET Core OpenID Cognito](https://github.com/aws-samples/aws-netcore-aspnetmvc-amazon-cognito-authentication-authorization-samples)

[ASP.NET Core Cognito Integration](https://github.com/gianibob82/aspnet-core-cognito-integration)

Videos

[Authentication for Your Applications: Getting Started with Amazon Cognito](https://www.youtube.com/watch?v=OAR4ZHP8DEg) by Quint Van Deman

[Protecting your .NET Core Serverless App with Amazon Cognito](https://www.youtube.com/watch?v=sUAXCv86BuE) by Mike Bentzen

Blogs

[Protecting your .NET Core Serverless App with Amazon Cognito](https://mike.bentzen.com.au/post/2018-12-11-dotnetcore-aspnet-with-aws-cognito/) by Mike Bentzen

[Hello, Cloud series home](https://davidpallmann.hashnode.dev/series/hello-cloud)
 