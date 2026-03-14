# Identity.API / Program.cs
```csharp
//Program.cs

var builder = WebApplication.CreateBuilder(args);
//This is the starting point of every .NET app. It sets up three things: configuration (reads from appsettings.json and environment variables),
//a DI container where you register all your services, and a logger. Think of it as "prepare everything before the app starts."

builder.AddServiceDefaults();
//A custom eShop method that every microservice calls. It adds health checks, distributed tracing, and service discovery in one line
//so every service behaves consistently without repeating setup code.

builder.Services.AddControllersWithViews();
//Registers the MVC framework. The "WithViews" part means this service serves actual HTML pages (Razor views), not just JSON.
//This is needed because the login page is a real HTML form that the user interacts with in the browser.

builder.AddNpgsqlDbContext<ApplicationDbContext>("identitydb");
builder.Services.AddMigration<ApplicationDbContext, UsersSeed>();
//The first line connects EF Core to a Postgres database. The connection string named "identitydb" is injected automatically
//by Aspire at runtime. The second line runs database migrations on startup (creates/updates tables) and then seeds default
//users like alice and bob for development. Not recommended for production because two instances starting at the same time can conflict.
// Apply database migration automatically. Note that this approach is not
// recommended for production scenarios. Consider generating SQL scripts from
// migrations instead.

builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();
//ASP.NET Identity is the user management system. It handles storing users, hashing passwords, managing roles, and generating tokens for
//password resets and 2FA. AddEntityFrameworkStores tells it to save all of this to the Postgres database. This has nothing to do with
//OAuth or JWTs — it just knows about users. IdentityServer (next section) is what issues tokens.

builder.Services.AddIdentityServer(options =>
{
    //options.IssuerUri = "null";
    options.Authentication.CookieLifetime = TimeSpan.FromHours(2);

    options.Events.RaiseErrorEvents = true;
    options.Events.RaiseInformationEvents = true;
    options.Events.RaiseFailureEvents = true;
    options.Events.RaiseSuccessEvents = true;

    // TODO: Remove this line in production.
    options.KeyManagement.Enabled = false;
})
.AddInMemoryIdentityResources(Config.GetResources())
.AddInMemoryApiScopes(Config.GetApiScopes())
.AddInMemoryApiResources(Config.GetApis())
.AddInMemoryClients(Config.GetClients(builder.Configuration))
.AddAspNetIdentity<ApplicationUser>()
// TODO: Not recommended for production - you need to store your key material somewhere secure
.AddDeveloperSigningCredential();
//IdentityServer is what makes this service an OAuth2/OpenID Connect server — it issues JWTs to client apps.
//The options block sets the SSO cookie lifetime (2 hours) and enables event logging. The AddInMemoryXxx calls load all
//OAuth config (which apps can connect, what scopes exist, which APIs are protected) from a static Config class into memory at startup.
//AddAspNetIdentity is the bridge that connects IdentityServer to ASP.NET Identity so it can validate user credentials.
//AddDeveloperSigningCredential creates a temporary key for signing tokens — only for dev, never production.

builder.Services.AddTransient<IProfileService, ProfileService>();
builder.Services.AddTransient<ILoginService<ApplicationUser>, EFLoginService>();
builder.Services.AddTransient<IRedirectService, RedirectService>();
//Three custom services. EFLoginService validates username and password against the database. ProfileService decides which
//claims (user ID, roles, email) get embedded into the JWT when a token is issued. RedirectService handles safe redirects
//after login — it checks that the return URL is not malicious before sending the user back. All three are Transient meaning
//a fresh instance is created per request.

var app = builder.Build();

app.MapDefaultEndpoints();
//This comes from the eShop ServiceDefaults shared project. It maps two HTTP endpoints that every microservice exposes: /health and /alive.
//These are called by Aspire's dashboard, Kubernetes, or a load balancer to check if the service is running correctly. /alive is a
//simple "am I running" check. /health is a deeper check that also verifies the database connection is healthy. If either returns a failure,
//the infrastructure knows to restart the pod or stop sending traffic to it.

app.UseStaticFiles();
app.UseCookiePolicy(new CookiePolicyOptions { MinimumSameSitePolicy = SameSiteMode.Lax });
app.UseRouting();
app.UseIdentityServer();
app.UseAuthorization();
app.MapDefaultControllerRoute();
//Order matters here. UseStaticFiles serves CSS/JS early so auth logic never runs for them. UseCookiePolicy fixes a Chrome 80+ bug
//that breaks SSO over HTTP in dev. UseRouting matches the URL to an endpoint. UseIdentityServer runs auth
//(reads the SSO cookie, sets the current user, and exposes OAuth endpoints like /connect/token). UseAuthorization enforces
//[Authorize] attributes — it must come after identity is established. MapDefaultControllerRoute wires up the MVC controllers including the login page.

app.Run();

```
## Complete Flow
 
When the app starts, **`WebApplication.CreateBuilder`** sets up **configuration, logging, and the DI container**, then **`AddServiceDefaults`** layers in **health checks and tracing** that every eShop service shares.
 
**MVC** is registered so the app can serve both the **HTML login page** and **API responses**.
 
**EF Core** connects to **Postgres** and immediately runs any pending **migrations** then seeds **default users**.
 
**ASP.NET Identity** is registered on top of that database so the app can **store users, hash passwords, and manage roles**.
 
**IdentityServer** is then added as the **OAuth2 engine** — it loads all **client apps, API scopes, and resources** into memory from `Config.cs`, and **bridges to ASP.NET Identity** so it can validate credentials when a user logs in.
 
Three custom services are registered:
- **`EFLoginService`** — checks passwords
- **`ProfileService`** — decides what claims go into the token
- **`RedirectService`** — safely sends users back after login
 
Once **`builder.Build()`** compiles all of this into a running app, the **middleware pipeline** takes over in this order:
 
| Step | Middleware | What it does |
|------|-----------|-------------|
| 1 | `UseStaticFiles` | Serves CSS/JS first, no auth needed |
| 2 | `UseCookiePolicy` | Fixes Chrome 80+ SSO bug over HTTP |
| 3 | `UseRouting` | Matches the URL to an endpoint |
| 4 | `UseIdentityServer` | Reads SSO cookie, identifies the user, exposes OAuth endpoints |
| 5 | `UseAuthorization` | Enforces `[Authorize]` rules |
| 6 | `MapDefaultControllerRoute` | Routes to login controller and MVC actions |
 
---
 
## The Three Jobs of This Service
 
> This single service does **three things** at once:
 
| Job | Technology | Responsibility |
|-----|-----------|---------------|
| **User Store** | ASP.NET Identity + EF Core | Stores users, hashes passwords, manages roles |
| **Authorization Server** | IdentityServer | Issues JWTs trusted by every other microservice |
| **Login UI** | MVC + Razor Views | Renders the HTML login page users interact with |


# Identity.API / Configuration / Config.cs

# Identity.API / appsettings.json

# Identity.API / Models / ApplicationUser.cs

# Identity.API / Data / ApplicationDbContext.cs

# Identity.API / Data / Migrations /

# Identity.API / UsersSeed.cs

# Identity.API / Services / EFLoginService.cs

# Identity.API / Services / ProfileService.cs

# Identity.API / Services / RedirectService.cs

# Identity.API / Services / ILoginService.cs

# Identity.API / Services / IRedirectService.cs

# Identity.API / Quickstart / Account / AccountController.cs

# Identity.API / Views / Account /

 
