---
Id: 30
Posted Date: 20/3/2017
Tags: RazorPages,Authentication,Authorization 
Slug: authentication-and-authorization-in-razorpages
---
# Authentication & Authorization in RazorPages

RazorPages is the new programming model the introduced on July 2016 in the [RazorPages](https://github.com/aspnet/razorpages) repo as a prototype for proving the concept of MVC View Pages, where the complicated MVC folder structure unneeded in some cases.

For more information you can refer to my blog post [here](http://www.hishambinateya.com/welcome-razor-pages).

There are some changes occurred since then, but the fundamentals are still the same, also the source code move to [MVC](https://github.com/aspnet/Mvc) repo last few months. A lot of enhancements is going on, and as [Damian Edwards](https://twitter.com/DamianEdwards) mentioned few times RazorPages will be the big feature that will be shipped at ASP.NET Core 2.0.

RazorPages is the successor of the old ASP.NET Web Pages that many of us - even I - like and love, RazorPages enhance and bring some new features that may not there before or difficult to accomplish, one of those are authentication & authorization.

Today we will dig little bit on this, we will see how RazorPages allow us to use AUTH.

### Authentication

Authentication is the process to verifying the identity of a user of the application.

No need to talk too much here, because ASP.NET Core Identity is a membership system which allow you to create users, login with username and password, or using external login providers such as Microsoft Accounts, Facebook, Twitter .. etc.

So the code is still the same, I encourage you to read about the [identity](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/identity), below is the basic setup for using cookie-based authentication
```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseCookieAuthentication(new CookieAuthenticationOptions
    {
        LoginPath = "/Login",
        AutomaticAuthenticate = true,
        AutomaticChallenge = true
    });
}
```
### Authorization

Authorization is the process to granting or denying access to application resources.

Here is where the RazorPages rocks :) RazorPages using a convention-based approach that make developer life easier than before, I still remember that I need to create a punch of `Web.Config` files especially when I need to protect nested folders, really It was tedious and headache, but that's what we had at the time, but no long now with RazorPages.

Simply you need to specify all the AUTH that your application needs in the `RazorPagesOptions`, let us have a look to the following code:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddRazorPages(options =>
    {
        options.AuthorizeFolder("/Users", "Admins");
        options.AllowAnonymousToPage("/User/Contact");
    });
}
```
Wow!! is it that easy?!! of course, what you we need to care about is _"Who Do What_" and let the RazorePages take care about the rest. In above snippet we used `AuthorizeFolder()` to protect the give path which is "/Users" in our case, also you should to specify a policy such as "Admins", but it's no longer required anymore after my [pull request](http://github.com/aspnet/Mvc/pull/5942), also we make the "Contact.cshtml" public by using `AllowAnonymousToPage()`.

**Note:** `AllowAnonymousToPage()` and `AllowAnonymousToFolder()` is not included yet while I'm writing the blog post, but it's in own way, after merge this [pull request](https://github.com/aspnet/Mvc/pull/5987) :)

You can checkout the source code for this project from [RazorPages](https://github.com/aspnet/Mvc/tree/dev/src/Microsoft.AspNetCore.Mvc.RazorPages) in [MVC](https://github.com/aspnet/mvc) repository on GitHub.

Happy Coding !!