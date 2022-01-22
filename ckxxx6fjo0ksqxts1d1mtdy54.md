## Hello, Cognito!

#### This episode: Amazon Cognito. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Cognito and use it to secure a "Hello, Cloud" .NET web application. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Cognito: What is it, and why use It?

[Amazon Cognito](https://aws.amazon.com/cognito/) provides authentication, authorization, and user management for web and mobile apps. You can add user sign-up, sign-in, and access control easily. As AWS says, "Spend your time creating great apps. Let Amazon Cognito handle authentication."

What kind of identities does Cognito support? Cognito can maintain custom identities, or federate with enterprise identities or social identities. You can support SAML 2.0 and OpenID Connect enterprise identities. For social sign-ins, you can support Amazon, Apple, Facebook, and Google identities.

Cognito has user pools and identity pools. **User pools** are for identity verification. Cognito can maintain a user pool, or can federate with another identity provider. **Identity pools** are for authorization or access control. You use identity pools to give users unique identities and give them access to other AWS services. 

A simple scenario is to merely authenticate your user against a Cognito user pool, without authorization.

![diagram-userpool.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640736327839/2nmEZ584u.png)

A more elaborate scenario is federated authentication and authorization. A common sequence for a web application would be 1) Cognito user sign-in with an identity provider, resulting in idP tokens, 2) exchange idP tokens for AWS credentials via a Cognito identity pool, and 3) access other AWS services with those credentials.

![diagram-identitypool.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640736338989/NrqyvBdxG.png)

From ASP.NET, your application can connect with Cognito in several different ways:

| Method | Packages |
| --------- | ----- |
| Cognito Identity Provider | Amazon.AspNetCore.Identity.Cognito |
| | Amazon.Extensions.CognitoAuthentication |
| Open ID Connect | Microsoft.AspNetCore.Authentication.OpenIdConnect |
| | Microsoft.Identity.Web | 

The [Cognito Identity Provider](https://aws.amazon.com/blogs/developer/now-generally-available-the-asp-net-core-identity-provider-for-amazon-cognito/), our focus today, is a [custom storage provider](https://docs.microsoft.com/en-us/aspnet/identity/overview/extensibility/overview-of-custom-storage-providers-for-aspnet-identity) for ASP.NET identity. That's a very different approach from Open ID Connect, which adds no Amazon-specific packages to an application.  

# Our Hello, Cognito Project

We're going to create an `ASP.NET` MVC web application that uses Cognito for user sign-in and new user registration. We'll limit ourselves to a Cognito user pool today, and tackle federation and social identities another time. The app we'll be creating uses the Cognito Identity Provider and is inspired by the AWS [ASP.NET Core Identity Provider for Amazon Cognito](https://github.com/aws/aws-aspnet-cognito-identity-provider) sample, updated here for .NET 6. We'll use our own application dialogs for authentication prompts. 

[source code](https://github.com/davidpallmann/hello-cognito)

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

## Step 2: Create a Cognito User Pool

In this step you'll create a new Cognito user pool in the AWS Console. 

1. Sign in to the AWS console and select the region you want to work in at top right. We're using **N. California**.

2. Navigate to **Amazon Cognito**. You can enter **cognito** in the search bar.

3. On the left panel, select **User pools** and click **Create user pool**. You will now go through several wizard pages to define your user pool.

4. On the Step 1 Configure sign-in experience page, enter/select the following and click **Next**.

    a. Authentication providers - Provider types: select **Cognito user pool**.

    b. Cognito user pool sign-in options: check **User name**, **Email**, and **Phone number**.

    ![03-aws-create-user-pool-step1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642160208811/87IZ_S_sG.png)

5. On the Step 2 Configure security requirements page, enter/select the following:

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

8. On the Step 6 Integrate your app page, enter/select the following and click **Next**:

    a. User pool name: enter **HelloCognito**.

    b. Domain - Domain type: select **User a Cognito domain**.

    c. Cognito domain: enter a domain name. We're using **hellocog**.

    b. Initial app client - App type: select **Public client**.

    c. App client name: enter **HelloCognitoWeb**.

    d. Client secret: select **Generate a client secret - Recommended**.

    ![03-aws-create-user-pool-step6.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642161163372/TVgSyyX6h.png)

    ![03-aws-create-user-pool-step6a.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642161310581/Z0VQXtf61.png)

    ![03-aws-create-user-pool-step6bb.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642161534530/TFF2LEqL5.png)

9. On the Ste 7 Review and create page, confirm the options match the above instructions, then click **Create user pool** at the bottom of the page. You should now see your HelloCognito user pool name listed on the Cogito User pools page.

10. Click the User pool name to view its detail. Select the **App integration tab** and record the following:

    a. Record the User pool ID (User pool overview panel).

    b. Click the **HelloCognitoWeb** client Id at bottom of page to view its detail.

    c. Record the Client ID (App client information panel).

    d. Capture the Client secret by clicking the **Show client secret** toggle.

## Step 3: Create .NET Web Application

In this step you'll create an `ASP.NET` web application, add Cognito packages and identity scaffolding pages, and modify application code for Cognito.

1. Open a command/terminal window and CD to a development folder.

2. Enter the dotnet new command below.

    ```none
dotnet new mvc -au Individual -n hello-cognito
```

    ![02-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640715651315/iHTqWkq2f.png)

3. In Visual Studio Solution Explorer, right-click the hello-cognito project and select **Manage NuGet Packages...**. 

    a. Find and install the **Amazon.AspNetCore.Identity.Cognito** package. 

    b. Find and install the **Amazon.Extensions.CognitoAuthentication** package.

     ![02-vs-packages.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640816877672/mlpRVQSk4.png)

4. In Solution Explorer, right-click the hello-cognito project and select **Add > New Scaffolded Item**. Complete the dialogs as follows.

     a. Select **Identity** from the category list on the left, then click **Install**.  

     ![03-vs-add-scaffolded-item.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640814497494/bNEw_9qlJ.png)

    b. In the Add Identity dialog, under Select an existing layout page, click the ellipsis (...) button and select your /Views/Shared/_Layout.cshtml file.

    c. Click the **Override all files** checkbox.

    d. Under Data context class, select the one available option, **ApplicationDbContext(hello_cognito.Data)**.

    ![02-vs-add-scaffolded-identity.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640815346605/MdbX8NhqH.png)

    e. Click **Add** and wait for updates to complete. New Razor pages and code files have been added under folder Areas/Identity/Pags/Account. 

    ![02-vs-add-scaffolded-identity-running.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640815752057/P-zDgyP2m.png)

5. Open appsettings.Development.json in the code editor and add the "Web" section listed at the end of this step, with the following replacements you captured in Step 2:

    a. Replace [client-id] with your cilent ID.

    b. Replace [client-secret] with your client secret.

    c. Replace [userpool-id] with the user pool ID.

6. Next you will replace the Razor pages and code for several of the scaffolding identity pages. We're updating the login, logout, register new user, confirm email, and reset password pages with the appropriate fields for our Cognito user pool. In place of the IdentityUser object in the generated scaffolding code, we're using the CognitoUser object. Some of the Razor pages do not change, but we're including both the .cshtml and .cshtml.cs file listings for completeness.

    a. Open Login.cshtml in the code editor and replace with the contents at the end of this step. Then open Login.cshtml.cs in the code editor and replace its contents.

    b. Replace the code for _LoginPartial.cshtml in the Views/Shared folder.

    c. Replace the code for LoginWith2fa.cshtml and LoginWith2fa.cshtml.cs.

    d. Replace the code for Logout.cshtml and Logout.cshtml.cs.

    e. Replace the code for ConfirmEmail.cshtml and ConfirmEmail.cshtml.cs.

    f. Replace the code for ForgotPassword.cshtml and ForgotPassword.cshtml.cs.

    g.  Replace the code for Register.cshtml and Register.cs.

    h.  Replace the code for ResetPassword.cshtml and ResetPassword.cs. 

    i. Save your changes.

7. Build the project and ensure it compiles without error.

appsettings.Development.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AWS": {
    "Region": "us-west-1",
    "UserPoolClientId": "[client-id]",
    "UserPoolClientSecret": "[client-secret]",
    "UserPoolId": "[userpool-id]"
  }
}
```

_LoginPartial.cshtml (in Views\Shared)

```csharp
@using Microsoft.AspNetCore.Identity
@using Amazon.Extensions.CognitoAuthentication
@inject SignInManager<CognitoUser> SignInManager
@inject UserManager<CognitoUser> UserManager

<ul class="navbar-nav">
@if (SignInManager.IsSignedIn(User))
{
    <li class="nav-item">
        <a  class="nav-link text-dark" asp-area="Identity" asp-page="/Account/Manage/Index" title="Manage">Hello @User.Identity?.Name!</a>
    </li>
    <li class="nav-item">
        <form  class="form-inline" asp-area="Identity" asp-page="/Account/Logout" asp-route-returnUrl="@Url.Action("Index", "Home", new { area = "" })">
            <button  type="submit" class="nav-link btn btn-link text-dark">Logout</button>
        </form>
    </li>
}
else
{
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="Identity" asp-page="/Account/Register">Register</a>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="Identity" asp-page="/Account/Login">Login</a>
    </li>
}
</ul>

```

Login.cshtml

```html
@page
@model LoginModel

@{
    ViewData["Title"] = "Log in";
}

<h2>@ViewData["Title"]</h2>
<div class="row">
    <div class="col-md-4">
        <section>
            <form method="post">
                <h4>Use a local account to log in.</h4>
                <hr />
                <div asp-validation-summary="All" class="text-danger"></div>
                <div class="form-group">
                    <label asp-for="Input.UserName"></label>
                    <input asp-for="Input.UserName" class="form-control" />
                    <span asp-validation-for="Input.UserName" class="text-danger"></span>
                </div>
                <div class="form-group">
                    <label asp-for="Input.Password"></label>
                    <input asp-for="Input.Password" class="form-control" />
                    <span asp-validation-for="Input.Password" class="text-danger"></span>
                </div>
                <div class="form-group">
                    <div class="checkbox">
                        <label asp-for="Input.RememberMe">
                            <input asp-for="Input.RememberMe" />
                            @Html.DisplayNameFor(m => m.Input.RememberMe)
                        </label>
                    </div>
                </div>
                <div class="form-group">
                    <button type="submit" class="btn btn-default">Log in</button>
                </div>
                <div class="form-group">
                    <p>
                        <a asp-page="./ForgotPassword">Forgot your password?</a>
                    </p>
                    <p>
                        <a asp-page="./Register" asp-route-returnUrl="@Model.ReturnUrl">Register as a new user</a>
                    </p>
                </div>
            </form>
        </section>
    </div>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}

```

Login.cshtml.cs

```csharp
#nullable disable

using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Threading.Tasks;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class LoginModel : PageModel
    {
        private readonly SignInManager<CognitoUser> _signInManager;
        private readonly ILogger<LoginModel> _logger;

        public LoginModel(SignInManager<CognitoUser> signInManager, ILogger<LoginModel> logger)
        {
            _signInManager = signInManager;
            _logger = logger;
        }

        [BindProperty]
        public InputModel Input { get; set; }

        public IList<AuthenticationScheme> ExternalLogins { get; set; }

        public string ReturnUrl { get; set; }

        [TempData]
        public string ErrorMessage { get; set; }

        public class InputModel
        {
            [Required]
            public string UserName { get; set; }

            [Required]
            [DataType(DataType.Password)]
            public string Password { get; set; }

            [Display(Name = "Remember me?")]
            public bool RememberMe { get; set; }
        }

        public async Task OnGetAsync(string? returnUrl = null)
        {
            if (!string.IsNullOrEmpty(ErrorMessage))
            {
                ModelState.AddModelError(string.Empty, ErrorMessage);
            }

            returnUrl = returnUrl ?? Url.Content("~/");

            // Clear the existing external cookie to ensure a clean login process
            await HttpContext.SignOutAsync(IdentityConstants.ExternalScheme).ConfigureAwait(false);

            ExternalLogins = (await _signInManager.GetExternalAuthenticationSchemesAsync().ConfigureAwait(false)).ToList();

            ReturnUrl = returnUrl;
        }

        public async Task<IActionResult> OnPostAsync(string? returnUrl = null)
        {
            returnUrl = returnUrl ?? Url.Content("~/");

            if (ModelState.IsValid)
            {
                var result = await _signInManager.PasswordSignInAsync(Input.UserName, Input.Password, Input.RememberMe, lockoutOnFailure: false);
                if (result.Succeeded)
                {
                    _logger.LogInformation("User logged in.");
                    return LocalRedirect(returnUrl);
                }
                else if (result.RequiresTwoFactor)
                {
                    return RedirectToPage("./LoginWith2fa", new { ReturnUrl = returnUrl, RememberMe = Input.RememberMe });
                }
                else if (result.IsCognitoSignInResult())
                {
                    if (result is CognitoSignInResult cognitoResult)
                    {
                        if (cognitoResult.RequiresPasswordChange)
                        {
                            _logger.LogWarning("User password needs to be changed");
                            return RedirectToPage("./ChangePassword");
                        }
                        else if (cognitoResult.RequiresPasswordReset)
                        {
                            _logger.LogWarning("User password needs to be reset");
                            return RedirectToPage("./ResetPassword");
                        }
                    }

                }

                ModelState.AddModelError(string.Empty, "Invalid login attempt.");
                return Page();
            }

            // If we got this far, something failed, redisplay form
            return Page();
        }
    }
}
```

LoginWith2fa.cshtml
```html
@page
@model LoginWith2faModel
@{
    ViewData["Title"] = "Two-factor authentication";
}

<h2>@ViewData["Title"]</h2>
<hr />
<p>Your login is protected by an SMS 2FA code. Enter your code below.</p>
<div class="row">
    <div class="col-md-4">
        <form method="post" asp-route-returnUrl="@Model.ReturnUrl">
            <input asp-for="RememberMe" type="hidden" />
            <div asp-validation-summary="All" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Input.TwoFactorCode"></label>
                <input asp-for="Input.TwoFactorCode" class="form-control" autocomplete="off" />
                <span asp-validation-for="Input.TwoFactorCode" class="text-danger"></span>
            </div>
            <div class="form-group">
                <div class="checkbox">
                    <label asp-for="Input.RememberMachine">
                        <input asp-for="Input.RememberMachine" />
                        @Html.DisplayNameFor(m => m.Input.RememberMachine)
                    </label>
                </div>
            </div>
            <div class="form-group">
                <button type="submit" class="btn btn-default">Log in</button>
            </div>
        </form>
    </div>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

LoginWith2fa.cshtml.cs

```csharp
using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
using System;
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class LoginWith2faModel : PageModel
    {
        private readonly CognitoSignInManager<CognitoUser> _signInManager;
        private readonly ILogger<LoginWith2faModel> _logger;

        public LoginWith2faModel(SignInManager<CognitoUser> signInManager, ILogger<LoginWith2faModel> logger)
        {
            _signInManager = signInManager as CognitoSignInManager<CognitoUser>;
            _logger = logger;
        }

        [BindProperty]
        public InputModel Input { get; set; }

        public bool RememberMe { get; set; }

        public string ReturnUrl { get; set; }

        public class InputModel
        {
            [Required]
            [StringLength(7, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)]
            [DataType(DataType.Text)]
            [Display(Name = "2FA code")]
            public string TwoFactorCode { get; set; }

            [Display(Name = "Remember this machine")]
            public bool RememberMachine { get; set; }
        }

        public async Task<IActionResult> OnGetAsync(bool rememberMe, string returnUrl = null)
        {
            // Ensure the user has gone through the username & password screen first
            var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();

            if (user == null)
            {
                throw new InvalidOperationException($"Unable to load two-factor authentication user.");
            }

            ReturnUrl = returnUrl;
            RememberMe = rememberMe;

            return Page();
        }

        public async Task<IActionResult> OnPostAsync(bool rememberMe, string returnUrl = null)
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            returnUrl = returnUrl ?? Url.Content("~/");

            var user = await _signInManager.GetTwoFactorAuthenticationUserAsync();
            if (user == null)
            {
                throw new InvalidOperationException($"Unable to load two-factor authentication user.");
            }

            var authenticatorCode = Input.TwoFactorCode.Replace(" ", string.Empty).Replace("-", string.Empty);

            var result = await _signInManager.RespondToTwoFactorChallengeAsync(authenticatorCode, rememberMe, Input.RememberMachine);

            if (result.Succeeded)
            {
                _logger.LogInformation("User with ID '{UserId}' logged in with 2fa.", user.UserID);
                return LocalRedirect(returnUrl);
            }
            else
            {
                _logger.LogWarning("Invalid 2FA code entered for user with ID '{UserId}'.", user.UserID);
                ModelState.AddModelError(string.Empty, "Invalid 2FA code.");
                return Page();
            }
        }
    }
}
```

Logout.cshtml

```html
@page
@model LogoutModel
@{
    ViewData["Title"] = "Log out";
}

<header>
    <h1>@ViewData["Title"]</h1>
    <p>You have successfully logged out of the application.</p>
</header>
```

Logout.cshtml.cs

```csharp
#nullable disable

using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
using System.Threading.Tasks;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class LogoutModel : PageModel
    {
        private readonly SignInManager<CognitoUser> _signInManager;
        private readonly ILogger<LogoutModel> _logger;

        public LogoutModel(SignInManager<CognitoUser> signInManager, ILogger<LogoutModel> logger)
        {
            _signInManager = signInManager;
            _logger = logger;
        }

        public void OnGet()
        {
        }

        public async Task<IActionResult> OnPost(string returnUrl = null)
        {
            await _signInManager.SignOutAsync();
            _logger.LogInformation("User logged out.");
            if (returnUrl != null)
            {
                return LocalRedirect(returnUrl);
            }
            else
            {
                return Page();
            }
        }
    }
}
```

ConfirmEmail.cshtml

```html
@page
@model ConfirmEmailModel
@{
    ViewData["Title"] = "Confirm Email";
}

<h2>@ViewData["Title"]</h2>

<div class="row">
    <div class="col-md-4">
        <form asp-route-returnUrl="@Model.ReturnUrl" method="post">
            <h4>Confirm your new account.</h4>
            <hr />
            <div asp-validation-summary="All" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Input.Code"></label>
                <input asp-for="Input.Code" class="form-control" />
                <span asp-validation-for="Input.Code" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-default">Confirm Account</button>
        </form>
    </div>
</div>

@section Scripts {
<partial name="_ValidationScriptsPartial" />
}
```

ConfirmEmail.cshtml.cs

```csharp

#nullable disable

using System;
using System.ComponentModel.DataAnnotations;
using System.Linq;
using System.Security.Claims;
using System.Threading.Tasks;
using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class ConfirmEmailModel : PageModel
    {
        private readonly CognitoUserManager<CognitoUser> _userManager;

        public ConfirmEmailModel(UserManager<CognitoUser> userManager)
        {
            _userManager = userManager as CognitoUserManager<CognitoUser>;
        }

        [BindProperty]
        public InputModel Input { get; set; }

        public string ReturnUrl { get; set; }

        public class InputModel
        {
            [Required]
            [Display(Name = "Code")]
            public string Code { get; set; }
        }

        public void OnGet(string returnUrl = null)
        {
            ReturnUrl = returnUrl;
        }

        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            returnUrl = returnUrl ?? Url.Content("~/");
            if (ModelState.IsValid)
            {
                var userId = User.Claims.FirstOrDefault(c => c.Type == ClaimTypes.Name).Value;

                var user = await _userManager.FindByIdAsync(userId);
                if (user == null)
                {
                    return NotFound($"Unable to load user with ID '{userId}'.");
                }

                var result = await _userManager.ConfirmSignUpAsync(user, Input.Code, true);
                if (!result.Succeeded)
                {
                    throw new InvalidOperationException($"Error confirming account for user with ID '{userId}':");
                }
                else
                {
                    return returnUrl != null ? LocalRedirect(returnUrl) : Page() as IActionResult;
                }
            }

            // If we got this far, something failed, redisplay form
            return Page();
        }
    }
}
```

ForgotPassword.cshtml

```html
@page
@model ForgotPasswordModel
@{
    ViewData["Title"] = "Forgot your password?";
}

<h1>@ViewData["Title"]</h1>
<h4>Enter your email.</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form method="post">
            <div asp-validation-summary="All" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Input.Email"></label>
                <input asp-for="Input.Email" class="form-control" />
                <span asp-validation-for="Input.Email" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-primary">Submit</button>
        </form>
    </div>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

ForgotPassword.cshtml.cs
```csharp
#nullable disable

using System;
using System.Collections.Generic;
using System.ComponentModel.DataAnnotations;
using System.Text.Encodings.Web;
using System.Text;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.AspNetCore.WebUtilities;

using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class ForgotPasswordModel : PageModel
    {
        private readonly CognitoUserManager<CognitoUser> _userManager;

        public ForgotPasswordModel(UserManager<CognitoUser> userManager)
        {
            _userManager = userManager as CognitoUserManager<CognitoUser>;
        }

        [BindProperty]
        public InputModel Input { get; set; }

        public class InputModel
        {
            [Required]
            [EmailAddress]
            public string Email { get; set; }
        }

        public async Task<IActionResult> OnPostAsync()
        {
            if (ModelState.IsValid)
            {
                var user = await _userManager.FindByEmailAsync(Input.Email);
                if (user == null || !(await _userManager.IsEmailConfirmedAsync(user)))
                {
                    // Don't reveal that the user does not exist or is not confirmed
                    return RedirectToPage("./ResetPassword");
                }

                // Cognito will send notification to user with reset token the user can use to reset their password.
                await user.ForgotPasswordAsync();

                return RedirectToPage("./ResetPassword");
            }

            return Page();
        }
    }
}
```


Register.cshtml

```html
@page
@model RegisterModel
@{
    ViewData["Title"] = "Register";
}

<h2>@ViewData["Title"]</h2>

<div class="row">
    <div class="col-md-4">
        <form asp-route-returnUrl="@Model.ReturnUrl" method="post">
            <h4>Create a new account.</h4>
            <hr />
            <div asp-validation-summary="All" class="text-danger"></div>
            <div class="form-group">
                <label asp-for="Input.UserName"></label>
                <input asp-for="Input.UserName" class="form-control" />
                <span asp-validation-for="Input.UserName" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Input.Email"></label>
                <input asp-for="Input.Email" class="form-control" />
                <span asp-validation-for="Input.Email" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Input.Password"></label>
                <input asp-for="Input.Password" class="form-control" />
                <span asp-validation-for="Input.Password" class="text-danger"></span>
            </div>
            <div class="form-group">
                <label asp-for="Input.ConfirmPassword"></label>
                <input asp-for="Input.ConfirmPassword" class="form-control" />
                <span asp-validation-for="Input.ConfirmPassword" class="text-danger"></span>
            </div>
            <button type="submit" class="btn btn-default">Register</button>
        </form>
    </div>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

Register.cshtml.cs

```csharp
#nullable disable

using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class RegisterModel : PageModel
    {
        private readonly SignInManager<CognitoUser> _signInManager;
        private readonly CognitoUserManager<CognitoUser> _userManager;
        private readonly ILogger<RegisterModel> _logger;
        private readonly CognitoUserPool _pool;

        public RegisterModel(
            UserManager<CognitoUser> userManager,
            SignInManager<CognitoUser> signInManager,
            ILogger<RegisterModel> logger,
            CognitoUserPool pool)
        {
            _userManager = userManager as CognitoUserManager<CognitoUser>;
            _signInManager = signInManager;
            _logger = logger;
            _pool = pool;
        }

        [BindProperty]
        public InputModel Input { get; set; }

        public string ReturnUrl { get; set; }

        public class InputModel
        {
            [Required]
            [EmailAddress]
            [Display(Name = "Email")]
            public string Email { get; set; }

            [Required]
            [Display(Name = "UserName")]
            public string UserName { get; set; }

            [Required]
            [StringLength(100, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)]
            [DataType(DataType.Password)]
            [Display(Name = "Password")]
            public string Password { get; set; }

            [DataType(DataType.Password)]
            [Display(Name = "Confirm password")]
            [Compare("Password", ErrorMessage = "The password and confirmation password do not match.")]
            public string ConfirmPassword { get; set; }
        }

        public void OnGet(string? returnUrl = null)
        {
            ReturnUrl = returnUrl;
        }

        public async Task<IActionResult> OnPostAsync(string? returnUrl = null)
        {
            returnUrl = returnUrl ?? Url.Content("~/");
            if (ModelState.IsValid)
            {
                var user = _pool.GetUser(Input.UserName);
                user.Attributes.Add(CognitoAttribute.Email.AttributeName, Input.Email);

                var result = await _userManager.CreateAsync(user, Input.Password);
                if (result.Succeeded)
                {
                    _logger.LogInformation("User created a new account with password.");

                    await _signInManager.SignInAsync(user, isPersistent: false);

                    return RedirectToPage("./ConfirmEmail");
                }
                foreach (var error in result.Errors)
                {
                    ModelState.AddModelError(string.Empty, error.Description);
                }
            }

            // If we got this far, something failed, redisplay form
            return Page();
        }
    }
}
```

ResetPassword.cshtml

```html
@page
@model ResetPasswordModel
@{
    ViewData["Title"] = "Reset password";
}

<h1>@ViewData["Title"]</h1>
<h2>Reset your password.</h2>
<hr />
<div class="row">
    <div class="col-md-4">
        <form method="post">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input asp-for="Input.Code" type="hidden" />
            <div class="form-floating">
                <input asp-for="Input.Email" class="form-control" autocomplete="username" aria-required="true" />
                <label asp-for="Input.Email" class="form-label"></label>
                <span asp-validation-for="Input.Email" class="text-danger"></span>
            </div>
            <div class="form-floating">
                <input asp-for="Input.Password" class="form-control" autocomplete="new-password" aria-required="true" />
                <label asp-for="Input.Password" class="form-label"></label>
                <span asp-validation-for="Input.Password" class="text-danger"></span>
            </div>
            <div class="form-floating">
                <input asp-for="Input.ConfirmPassword" class="form-control" autocomplete="new-password" aria-required="true" />
                <label asp-for="Input.ConfirmPassword" class="form-label"></label>
                <span asp-validation-for="Input.ConfirmPassword" class="text-danger"></span>
            </div>
            <button type="submit" class="w-100 btn btn-lg btn-primary">Reset</button>
        </form>
    </div>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}

```

ResetPassword.cshtml.cs

```csharp
using Amazon.AspNetCore.Identity.Cognito;
using Amazon.Extensions.CognitoAuthentication;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;
using System;
using System.ComponentModel.DataAnnotations;
using System.Threading.Tasks;

namespace hello_cognito.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class ResetPasswordModel : PageModel
    {
        private readonly CognitoUserManager<CognitoUser> _userManager;
        private readonly ILogger<LoginWith2faModel> _logger;

        public ResetPasswordModel(UserManager<CognitoUser> userManger, ILogger<LoginWith2faModel> logger)
        {
            _userManager = userManger as CognitoUserManager<CognitoUser>;
            _logger = logger;
        }

        [BindProperty]
        public InputModel Input { get; set; }

        public string ReturnUrl { get; set; }

        public class InputModel
        {
            [Required]
            [EmailAddress]
            [Display(Name = "Email")]
            public string Email { get; set; }

            [Required]
            [DataType(DataType.Text)]
            [Display(Name = "Reset Token")]
            public string ResetToken { get; set; }

            [Required]
            [StringLength(100, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)]
            [DataType(DataType.Password)]
            [Display(Name = "New password")]
            public string NewPassword { get; set; }

            [DataType(DataType.Password)]
            [Display(Name = "Confirm password")]
            [Compare("NewPassword", ErrorMessage = "The password and confirmation password do not match.")]
            public string ConfirmPassword { get; set; }
        }

        public void OnGet(string returnUrl = null)
        {
            ReturnUrl = returnUrl;
        }

        public async Task<IActionResult> OnPostAsync(string returnUrl = null)
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            returnUrl = returnUrl ?? Url.Content("~/");

            var user = await _userManager.FindByEmailAsync(Input.Email);
            if (user == null)
            {
                throw new InvalidOperationException($"Unable to retrieve user.");
            }

            var result = await _userManager.ResetPasswordAsync(user, Input.ResetToken, Input.NewPassword);

            if (result.Succeeded)
            {
                _logger.LogInformation("Password reset for user with ID '{UserId}'.", user.UserID);
                return LocalRedirect(returnUrl);
            }
            else
            {
                _logger.LogInformation("Unable to rest password for user with ID '{UserId}'.", user.UserID);
                foreach (var item in result.Errors)
                {
                    ModelState.AddModelError(item.Code, item.Description);
                }
                return Page();
            }
        }
    }
}
```

## Step 4: Run the Program

In this step we'll run the .NET application and test user authentication and registration with Cognito. 

1. In Visual Studio, press **F5** to debug run the program.

2. The welcome page appears in a browser window.

    ![06_welcome.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885001019/I-jYx900Z.png)

3. Click **Login** at top right. On the Login page, click **Register as a new user**.

    ![06_login.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885054181/YCebGRMk_.png)

4. On the Register as a new user page, enter a username, email, password, and password confirmation. 

    ![06_register.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885399024/u5kfWB5pCF.png)

5. Click **Register**. You are prompted to enter a code. Go to the email account you registered with, where there should be a confirmation message with a code. 

    ![06_verification_mail.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885526403/yoZpnKy6WD.png)

    ![06_confirm_email.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885592416/dZaFtlQKt.png)

6. Enter the verification code from the email on the web page and click **Confirm**.  You are returned to the welcome page, this time with your identity shown at the top right.

    ![06_registered.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885693777/vhWvN0sET.png)

7. To test login, first click **Logout** at top right. Then click **Login**. Enter your username and password and check Remember me. Then click **Login**. You are signed in, and your identity again appears at the top right of the welcome page.

    ![06_login_davidpallmann.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640885845743/a5mOPb0av.png)

8. Let's view the user in the AWS console. Navigate to Amazon Cognito > User pools > HelloCognito and click the **Users** tab. The user you just registered is listed, along with their email address and confirmation status. You can click on the username for more details.

    ![06_view_user_aws.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640887543716/PFJCKAdEI.png)

9. Finally, let's test forgotten password functionality. Click **Logout**, then **Login**. On the Login page, click **Forgot your password?**. On the Forgot your password? page, enter the email address you registered with and click **Submit**. 

    ![06_forgot_password.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640886396752/muY0e9wrm.png)

10. You'll receive a password verification email. Open the email and copy the verification code. On the Reset password web page, enter your email address, the verification code, a new password, and password confirmation. Then click **Reset your password**. Your password is reset, which you can test by signing in again.

    ![06_password_verification_code.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640886586260/BZxZm1MUk.png)

    ![06_forgotpassword_enter_new_password.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1640886842049/t1z1BGfFN.png)

Congratulations! You have added authentication with a Cognito user pool to an `ASP.NET` 6 web application.

## Step 5: Shut It Down

When you're done with Hello, Cognito, delete your Cognito user pool. You don't want to accrue charges for something you aren't using.

1. In the AWS Console, navigate to Amazon Cognito > User pools.
2. Click the **HelloCognito** user pool to view its detail.
3. Select the App Integration tab.
4. Under Domain, select **Delete Cognito domain** from the Actions drop-down and confirm.
5. Under App client list, select the **HellCognito** app client, click **Delete**, and confirm.
6. Navigate back to Amazon Cognito > User pools.
7. Select the **HelloCognito** user pool, click **Delete**, and confirm.

# Where to Go From Here

Security can be tough. It's an ever-changing landscape, with many possible scenarios. Cognito is a service you can trust for your authentication and authorization needs.

In this tutorial, we looked at one Cognito scenario, user authentication. You set up an Amazon Cognito user pool, updated an `ASP.NET` 6 web application for Cognito, and tested it. You saw Cognito handle new user registration, login, logout, and forgot password. We used the Cognito Identity Provider, and we chose to handle the UI for those functions in our app. For contrast, you should also explore connecting to Cognito with Open ID Connect and having sign-in dialogs provided by Cognito.

Cognito can do a lot more, including federation with enteprise and social network identity providers. To go further with Cognito, learn more about the service and plan how you can apply it for your use cases. 

# Further Reading

AWS DOCUMENTATION

[Amazon Cognito](https://aws.amazon.com/cognito/)

[Getting Started with Amazon Cognito](https://aws.amazon.com/cognito/getting-started/)

[Amazon Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)

[AWS SDK for .NET - Authenticating Users with Amazon Cognito](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/cognito-apis-intro.html)

[Amazon Cognito Credentials Provider](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/cognito-creds-provider.html)

[Amazon Cognito Extension Library](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/cognito-authentication-extension.html)

[Access AWS services from an ASP.NET Core app using Amazon Cognito identity pools](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/access-aws-services-from-an-asp-net-core-app-using-amazon-cognito-identity-pools.html)

[.NET on AWS Developer Center](https://aws.amazon.com/developer/language/net/)

AWS Samples

[AWS ASP.NET Cognito Identity Provider and sample](https://github.com/aws/aws-aspnet-cognito-identity-provider)

[Cognito .NET Desktop App sample](https://github.com/aws-samples/aws-cognito-dot-net-desktop-app)

Videos

[Using AWS Cognito in your .NET Core app](https://www.youtube.com/watch?v=LcD_-p1gCww) by Greg Eppel

[Authentication for Your Applications: Getting Started with Amazon Cognito](https://www.youtube.com/watch?v=OAR4ZHP8DEg)

[Authenticating Serverless ASP.NET Core Web App Using Amazon Cognito](https://www.youtube.com/watch?v=M6qTrI7kmZk) by Sreelaxmi Pai

[Protecting your .NET Core Serverless App with Amazon Cognito](https://www.youtube.com/watch?v=sUAXCv86BuE) by Mike Bentzen

Blogs

[Now generally available: the ASP.NET Core Identity Provider for Amazon Cognito](https://aws.amazon.com/blogs/developer/now-generally-available-the-asp-net-core-identity-provider-for-amazon-cognito/)

[Proecting your .NET Core Serverless App with Amazon Cognito](https://mike.bentzen.com.au/post/2018-12-11-dotnetcore-aspnet-with-aws-cognito/) by Mike Bentzen

[Hello, Cloud series home](https://davidpallmann.hashnode.dev/series/hello-cloud)
 