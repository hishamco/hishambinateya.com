---
Id: 45
Posted Date: 4/9/2017
Tags:  Razor Pages,Conventions 
Slug: razor-pages-conventions
---
# Razor Pages Conventions

This is a short blog post that I'm going to illustrate all the page conventions in the Razor Pages. For the those who don't have a knowledge about Razor Pages please checkout my old blog post [Welcome to Razor Pages](http://www.hishambinateya.com/welcome-razor-pages), also I encourage you to read about the Razor Page in more details from [ASP.NET documentation](https://docs.microsoft.com/en-us/aspnet/core/mvc/razor-pages/?tabs=visual-studio).

The Razor Pages a great piece of technology that shine for the first time in ASP.NET Core 2.0. It brings a new programming model, that is suited for page focus web applications.

One of the cool things that I like about it is the page conventions, which I'm trying to explore today. There are five conventions that Razor Pages have, basically they are an extension methods for `PageConventionCollection` class, these page conventions are widely used in almost Razor Pages web applications:

**1- AllowAnonymousToPage**

This convention will allow the anonymous users to access a specific page in your web application.
```csharp
services.AddMvc()
   .AddRazorPagesOptions(options =>
   {
      options.Conventions.AllowAnonymousToPage("/Pages/Admin/Login");
   });
```
**2- AllowAnonymousToFolder**

This convention will allow the anonymous users to access a specific folder in your web application.
```csharp
services.AddMvc()
   .AddRazorPagesOptions(options =>
   {
      options.Conventions.AllowAnonymousToFolder("/Pages/SharedFolder");
   });
```
**3- AuthorizePage**

This convention will authorize the access to a specific page for an authorized users or a certain security policy.
```csharp
services.AddMvc()
   .AddRazorPagesOptions(options =>
   {
      options.Conventions.AuthorizePage("/Pages/SecurePage");
      options.Conventions.AuthorizePage("/Pages/OffersPage", "SubscriberPolicy");
   });
```
**4- AuthorizeFolder**

This convention will authorize the access to a specific folder for an authorized users or a certain security policy.
```csharp
services.AddMvc()
   .AddRazorPagesOptions(options =>
   {
      options.Conventions.AuthorizeFolder("/Pages/Admin");
      options.Conventions.AuthorizeFolder("/Pages/Offers", "SubscriberPolicy");
   });
```
**5- AddPageRoute**

This convention will allow you to add a specified route to a page.
```csharp
services.AddMvc()
   .AddRazorPagesOptions(options =>
   {
      options.Conventions.AddPageRoute("/Post/Archive.cshtml", "Post/{year}/{month}/{day}")
   });
```
If you notice the convention from 1- 4 are related to the AUTH in Razor Pages, if you need to more information about that you can checkout [Authentication & Authorization in Razor Pages](http://www.hishambinateya.com/authentication-and-authorization-in-razorpages). The last convention is related to the routing in Razor Pages, which allows you to build a friendly Urls, you can checkout [Friendly URLs in Razor Pages](http://www.hishambinateya.com/friendly-urls-in-razor-pages).

Happy Coding!!