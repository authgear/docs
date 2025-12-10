---
description: Add authentication for ASP.NET app with Authgear
---

# ASP.NET Core MVC

[![LLM | View as markdown](https://img.shields.io/badge/LLM-View%20as%20markdown-blue)](https://raw.githubusercontent.com/authgear/docs/refs/heads/main/get-started/regular-web-app/asp.net-core-mvc.md)

In this guide, you will learn how to add authentication features with [Authgear](https://www.authgear.com/) by implementing an [OpenID Connect](https://docs.authgear.com/concepts/identity-fundamentals#open-id-connect) flow, then retrieving OAuth tokens, to call APIs. View [implementation](https://github.com/authgear/authgear-example-dotnet) on GitHub.

### Learning objectives

You will learn the following throughout the article:

* How to add user login, sign-up, and logout to [ASP.NET](http://asp.net) Core Applications.
* How to use the [ASP.NET](http://asp.net) Core Authorization Middleware to protect [ASP.NET](http://asp.net) Core application routes.

## Add authentication to [ASP.NET](http://asp.net) Core App

### **Prerequisites**

Before you get started, you will need the following:

* A **free Authgear account**. [Sign up](https://accounts.portal.authgear.com/signup) if you don't have one already.
* [.NET 7](https://dotnet.microsoft.com/en-us/download) downloaded and installed on your machine. You can also use [Visual Studio](https://visualstudio.microsoft.com/) and [VS code](https://code.visualstudio.com/) to automatically detect the .NET version.

### Part 1: Configure Authgear

To use Authgear services, you’ll need to have an application set up in the Authgear [Dashboard](https://portal.authgear.com/). The Authgear application is where you will configure how you want to authenticate and manage your users.

#### Step 1: Configure an application

Use the interactive selector to create a new **Authgear OIDC Client application** or select an existing application that represents the project you want to integrate with.

<figure><img src="../../.gitbook/assets/image (34) (1).png" alt=""><figcaption></figcaption></figure>

Every application in Authgear is assigned an alphanumeric, unique client ID that your application code will use to call Authgear APIs through the OpenID Connect Client in the .NET app. Note down the Authgear `ISSUER` (for example, [https://example-auth.authgear.cloud](https://example-auth.authgear.cloud)), `CLIENT ID`, `CLIENT SECRET`, and `OpenID Token Endpoint` ([https://example-auth.authgear.cloud/oauth2/token](https://example-auth.authgear.cloud/oauth2/token)) from the output. You will use these values in the next step for the client app config.

<figure><img src="../../.gitbook/assets/image (35) (1).png" alt=""><figcaption></figcaption></figure>

#### Step 2: Configure **Redirect URI**

A **Redirect URI** of your application is the URL that Authgear will redirect to after the user has authenticated in order for the **OpenID Connect middleware** to complete the authentication process. In our case, it will be a home page for our [ASP.NET](http://asp.net) and it will run at[http://localhost:5002](http://localhost:5002).

Set the following redirect URI: [http://localhost:5002/signin-oidc](http://localhost:5002/signin-oidc) If not set, users will not be returned to your application after they log in.

#### Step 3: Enable Access Token

Also, enable **Issue JWT as an access token** option under the **Access Token** section of the app configuration:

<figure><img src="../../.gitbook/assets/Untitled (19).png" alt=""><figcaption></figcaption></figure>

#### Step 4: Choose a Login method

After you create the **Authgear app**, you choose how users need to **authenticate on the login page**. From the **Authentication** tab, navigate to **Login Methods**, you can choose a **login method** from various options including, by email, mobile, or social, just using a username or the custom method you specify. For this demo, we choose the **Email+Passwordless** approach where our users are asked to register an account and log in by using their emails. They will receive a One-time password (OTP) to their emails and verify the code to use the app.

<figure><img src="../../.gitbook/assets/image (36) (1).png" alt=""><figcaption></figcaption></figure>

### Part 2: Configure [ASP.NET](http://asp.net) Core application to use Authgear

This guide will be used to provide a way for your users to log in to your [ASP.NET](http://asp.net/) Core application. The [project source code](https://github.com/authgear/authgear-example-dotnet) can be found on GitHub. If you are familiar with the steps, you can skip this part and clone the code repository and run the code sample by following the [README.md](https://github.com/authgear/authgear-example-dotnet/blob/main/README.md) file there.

#### Step 1: Install dependencies

To integrate Authgear with [ASP.NET](http://asp.net/) Core you will use both the Cookie and OpenID Connect (OIDC) authentication handlers. If you are not using a sample project and are integrating Authgear into your own existing project, then please make sure that you add `Microsoft.AspNetCore.Authentication.OpenIdConnect` packages to your application. Run the following command in your terminal or use your editor to include the NuGet package there:

```bash
Install-Package Microsoft.AspNetCore.Authentication.OpenIdConnect
```

#### Step 2: **Install and configure OpenID Connect Middleware**

To enable authentication in your [ASP.NET](http://asp.net) Core application, use the OpenID Connect (OIDC) middleware. Open `Startup` the class and in the `ConfigureServices` method, add the authentication services and call the `AddAuthentication` method. To enable cookie authentication, call the `AddCookie` method. Next, configure the OIDC authentication handler by adding method `AddOpenIdConnect` implementation. Configure other parameters, such as `Issuer`, `ClientId`, `ClientSecret` , and `Scope`. Here, is what looks like `Startup.cs` after you apply these changes:

```csharp
public class Startup
 {

     public IWebHostEnvironment Environment { get; }
     public IConfiguration Configuration { get; }

     public Startup(IWebHostEnvironment environment, IConfiguration config)
     {
         Environment = environment;
         Configuration = config;
     }

     // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
     public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
     {
         app.UseRouting();
         app.UseAuthentication();
         app.UseAuthorization();
         app.UseEndpoints(endpoints =>
         {
             endpoints.MapRazorPages();
         });
     }

     public void ConfigureServices(IServiceCollection services)
     {
         // Prevent WS-Federation claim names being written to tokens
         JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();

         services.AddAuthentication(options =>
         {
             options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
             options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
         })
         .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme, options =>
         {
             // Use the strongest setting in production, which also enables HTTP on developer workstations
             options.Cookie.SameSite = SameSiteMode.Strict;
         })
         .AddOpenIdConnect(options =>
         {

             // Use the same settings for temporary cookies
             options.NonceCookie.SameSite = SameSiteMode.Strict;
             options.CorrelationCookie.SameSite = SameSiteMode.Strict;

             // Set the main OpenID Connect settings
             options.Authority = Configuration.GetValue<string>("OpenIdConnect:Issuer");
             options.ClientId = Configuration.GetValue<string>("OpenIdConnect:ClientId");
             options.ClientSecret = Configuration.GetValue<string>("OpenIdConnect:ClientSecret");
             options.ResponseType = OpenIdConnectResponseType.Code;
             options.ResponseMode = OpenIdConnectResponseMode.Query;
             string scopeString = Configuration.GetValue<string>("OpenIDConnect:Scope");
             options.Scope.Clear();
             scopeString.Split(" ", StringSplitOptions.TrimEntries).ToList().ForEach(scope =>
             {
                 options.Scope.Add(scope);
             });

             // If required, override the issuer and audience used to validate ID tokens
             options.TokenValidationParameters = new TokenValidationParameters
             {
                 ValidIssuer = options.Authority,
                 ValidAudience = options.ClientId
             };

             // This example gets user information for display from the user info endpoint
             options.GetClaimsFromUserInfoEndpoint = true;

             // Handle the post logout redirect URI
             options.Events.OnRedirectToIdentityProviderForSignOut = (context) =>
             {
                 context.ProtocolMessage.PostLogoutRedirectUri = Configuration.GetValue<string>("OpenIdConnect:PostLogoutRedirectUri");
                 return Task.CompletedTask;
             };

             // Save tokens issued to encrypted cookies
             options.SaveTokens = true;

             // Set this in developer setups if the OpenID Provider uses plain HTTP
             options.RequireHttpsMetadata = false;
         });

         services.AddAuthorization();
         services.AddRazorPages();

         // Add this app's types to dependency injection
         services.AddSingleton<TokenClient>();
     }
 }
```

#### Step 3: Add Protected resource

Assume that there is a protected resource like a Razor page `Protected.cshtml` that is used to represent views:

```html
@page "/protected"
@model ProtectedModel

@addTagHelper*, Microsoft.AspNetCore.Mvc.TagHelpers

<style type="text/css">
button
{
  width: 200px;
}
</style>

<h1>Protected View</h1>

<h3>
    <p>Welcome: @Model.Username</a>
    <p>Current Access Token: @Model.AccessToken</a>
    <p>Current Refresh Token: @Model.RefreshToken</a>
    
    <form method="post">
        <p><button value="RefreshToken" asp-page-handler="RefreshToken">Refresh Token</button></p>
        <p><button value="Logout" asp-page-handler="Logout">Logout</button></p>
    </form>
</h3>
```

And `ProtectedModel.cs` class to which `Authorize` the attribute is applied requires authorization.

```csharp
[Authorize]
public class ProtectedModel : PageModel
{
    public string Username { get; set; }
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }

    private readonly TokenClient tokenClient;

    public ProtectedModel(TokenClient tokenClient)
    {
        this.tokenClient = tokenClient;
    }

    public async Task OnGet()
    {
        ClaimsPrincipal user = this.User;
        var givenName = user.FindFirstValue("given_name");
        var familyName = user.FindFirstValue("family_name");
        this.Username = $"{givenName} {familyName}";

        this.AccessToken = await this.tokenClient.GetAccessToken(this.HttpContext);
        this.RefreshToken = await this.tokenClient.GetRefreshToken(this.HttpContext);
    }

    public async Task<IActionResult> OnPostRefreshToken()
    {
        await this.tokenClient.RefreshAccessToken(this.HttpContext);
        this.AccessToken = await this.tokenClient.GetAccessToken(this.HttpContext);
        this.RefreshToken = await this.tokenClient.GetRefreshToken(this.HttpContext);
        return Page();
    }

    public async Task OnPostLogout()
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        await HttpContext.SignOutAsync(OpenIdConnectDefaults.AuthenticationScheme);
    }
}
```

To see protected data, users need to go through the authentication process via Authgear.

<figure><img src="../../.gitbook/assets/Untitled (21).png" alt=""><figcaption></figcaption></figure>

If a user has not authenticated yet, `Unauthenticated.chtml` the page is rendered, an OpenID Connect redirect flow is triggered and the user needs to authenticate through the Authgear login page. See **Run the Application** section

After successful authentication, you should see the protected page with the following details:

<figure><img src="../../.gitbook/assets/Untitled (20).png" alt="" width="531"><figcaption></figcaption></figure>

#### Step 4: Get and Use Refresh Token

As part of the OAuth 2.0 standard, we can use the refresh token returned by the token endpoint to get a new access token. Doing so enables our application to replace an expired access token without requiring the user to repeat the entire login process.

The following code in `ProtectedModel.cs` is responsible for doing that:

```csharp
public async Task<IActionResult> OnPostRefreshToken()
{
    await this.tokenClient.RefreshAccessToken(this.HttpContext);
    this.AccessToken = await this.tokenClient.GetAccessToken(this.HttpContext);
    this.RefreshToken = await this.tokenClient.GetRefreshToken(this.HttpContext);
    return Page();
}
```

**Note:** You must include `offline_access` in your OAuth 2.0 scope for the Authgear authorization server to return a refresh token.

#### Step 5: Logout

The Logout button on the `Protected.cshtml` page calls the `OnPostLogout()` method in ProtectedModel.cs. The method will delete the current user session and redirect to Authgear's end session endpoint for the user to complete the logout process.

The code sample below shows the implementation of the `OnPostLogout()` method:

```csharp
public async Task OnPostLogout()
    {
        await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
        await HttpContext.SignOutAsync(OpenIdConnectDefaults.AuthenticationScheme);
    }
```

#### Step 6: Set up and run the application

Start by cloning the project into your local machine:

```bash
git clone 
```

Make the project directory your current working directory:

```bash
cd authgear-example-dotnet
```

Update the following configuration variables in the `appsettings.json` file with your Authgear app settings values from **Part1** such as `Issuer`, `ClientId`, `ClientSecret`, and Authgear endpoint:

```bash
{
    "OpenIDConnect": {
        "ClientId": "{your-client-id}",
        "ClientSecret": "{your-client-secret}",
        "Issuer": "{your-authgear-app-endpoint}",
        "Scope": "openid offline_access",
        "PostLogoutRedirectUri": "<http://localhost:5002>",
        "TokenEndpoint": "{your-authgear-app-endpoint}/oauth2/token"
    },
    "Urls": "<http://localhost:5002>",
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft": "Warning",
            "Microsoft.Hosting.Lifetime": "Information"
        }
    }
}
```

Execute the following command to run the [ASP.NET](http://asp.net/) Core web application:

```bash
dotnet build
dotnet run
```

You can now visit [http://localhost:5002](http://localhost:5002) to access the application. When you click on the **"View Protected Data"** button, [ASP.NET](http://asp.net) Core takes you to the **Authgear’s Login page**.

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt="" width="464"><figcaption></figcaption></figure>

Your users can log in to your application through a page hosted by Authgear, which provides them with a secure, standards-based login experience that you can customize with your own branding and various authentication methods, such as [social logins](https://www.authgear.com/features/social-login), [passwordless](https://www.authgear.com/features/passwordless-authentication), [biometrics logins](https://www.authgear.com/features/biometric-authentication), [one-time-password (OTP)](https://www.authgear.com/features/whatsapp-otp) with SMS/WhatsApp, and multi-factor authentication (MFA).

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt="" width="464"><figcaption></figcaption></figure>

After you have authenticated, a protected view is rendered. The application receives an Access token that it uses to present user data on the screen, and tokens that could be used in upstream requests to some backend API, to access data on behalf of the user.

<figure><img src="../../.gitbook/assets/Untitled (20).png" alt=""><figcaption></figcaption></figure>

#### Next steps

This guide showed how to quickly implement an end-to-end OpenID Connect flow in .NET with Authgear. Only simple code is needed, after which protected views are secured with built-in UI login pages.
