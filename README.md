# Test-AspNetGoogleOAuth2Authentication
Fixes to .NET sample from ASP.NET MVC template for Google OAuth2.0 Authentication with explanations in comments.

# Explanation
I'm using the default ASP.NET MVC 5 template with Identity Authentication for simplicity, but hopefully this can be modified for different use cases.

**StartupAuth.cs**

Do not customize the redirect path.  It gets replaced by /signin-google anyways and my attempts at getting around that caused "silent" (not in the debugger) Internal Server 500 errors.

    app.UseGoogleAuthentication(new GoogleOAuth2AuthenticationOptions()
    {
        ClientId = "whatevs.apps.googleusercontent.com",
        ClientSecret = "whatevs_secrut",
        Provider = new GoogleOAuth2AuthenticationProvider()
    });

Make sure to add http://whatever.com/signin-google to https://console.developers.google.com/ in your `APIs & auth` > `Credentials` > `Redirect URIs` section.

**RouteConfig.cs**

Add a route to a permanent redirect controller action to your routes.  Permanent redirects are the only thing that will suffice here.  It is not enough to simply direct directly to the Callback URL.

    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
    
        routes.MapRoute(
            name: "Google API Sign-in",
            url: "signin-google",
            defaults: new { controller = "Account", action = "ExternalLoginCallbackRedirect" }
        );
    
        routes.MapRoute(
            name: "Default",
            url: "{controller}/{action}/{id}",
            defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
        );
    }

**AccountController.cs**

Permanent redirect to the built-in callback method and you should be fine.

    [AllowAnonymous]
    public ActionResult ExternalLoginCallbackRedirect(string returnUrl)
    {
        return RedirectPermanent("/Account/ExternalLoginCallback");
    }