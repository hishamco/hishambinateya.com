---
Id: 54
Posted Date: 29/1/2018
Tags: Role-Based Authorization,Security,RazorPages 
Slug: role-based-authorization-in-razor-pages
---
# Using Role-Based Authorization in Razor Pages

Long time ago I blogged about [Authentication & Authorization in RazorPages](http://www.hishambinateya.com/authentication-and-authorization-in-razorpages) which I introduced the authentication & authorization processes in Razor Pages, and after a while I wrote another blog post about [Razor Pages Conventions](http://www.hishambinateya.com/razor-pages-conventions) which I showed you in some details how Razor Pages provide a convention-based to access control of the page(s) and folder(s).

One of the missing features that I notice - and requested by the folks - is **Role-Based Authorization**. AFAIK there's no direct way to support this feature yet using the Razor Pages Conventions.

But there 's a HACK if you need to support such feature by using `AuthorizePage` or `AuthorizeFolder` with the overload that accept a policy as string. Again you need to define your policy yourself and pass its name to above methods. Perhaps the easiest way to achieve this by using `AddAuthorization` options as the following:
```csharp
services.AddAuthorization(options =>
{ 
     options.AddPolicy("RequireAdministratorRole", policy => policy.RequireRole("Administrator"));
}); 
```
Seems this not the elegant way to add such feature, so that's why I decide to write a new blog post that explain how can we extend the `PageConventionCollection` to support **Role-Based Authorization** with few lines of code.

First of all let us create `PageConventionCollectionExtensions` class, with adding another overload for `AuthorizeFolder` and `AuthorizePage` that accepts roles as array of string.
```csharp
public static PageConventionCollection AuthorizeFolder(this PageConventionCollection conventions, string folderPath, string[] roles)
{
    if (conventions == null)
    {
        throw new ArgumentNullException(nameof(conventions));
    }

    if (string.IsNullOrEmpty(folderPath))
    {
        throw new ArgumentException("Argument cannot be null or empty.", nameof(folderPath));
    }

    var policy = new AuthorizationPolicyBuilder().
        RequireRole(roles).Build();
    var authorizeFilter = new AuthorizeFilter(policy);
    conventions.AddFolderApplicationModelConvention(folderPath, model => model.Filters.Add(authorizeFilter));
    return conventions;
}
```
If you notice above that I used `AuthorizationPolicyBuilder` to create a new policy that accepts a set of role names using `RequireRole` function, after that I create an authorization filter using the created policy, which will be added to the `Filters` collection in page application model.

Then we can do the same thing with `AuthorizePage` extension method.
```csharp
public static PageConventionCollection AuthorizePage(this PageConventionCollection conventions, string pageName, string[] roles)
{
    if (conventions == null)
    {
        throw new ArgumentNullException(nameof(conventions));
    }

    if (string.IsNullOrEmpty(pageName))
    {
        throw new ArgumentException("Argument cannot be null or empty.", nameof(pageName));
    }

    var policy = new AuthorizationPolicyBuilder().
        RequireRole(roles).Build();
    var authorizeFilter = new AuthorizeFilter(policy);
    conventions.AddPageApplicationModelConvention(pageName, model => model.Filters.Add(authorizeFilter));
    return conventions;
}
```
Finally we can use the newly added methods like the other Razor Pages Conventions as the following:
```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.Conventions.AuthorizePage("/Private", new [] { "Administrators" });
        options.Conventions.AuthorizeFolder("/Account", new[] { "Administrators" });
    });
```
That is it!! now you 're free to use **Role-Based Authorization** in Razor Pages with the same conventions that you know and love.

Happy Coding!!
