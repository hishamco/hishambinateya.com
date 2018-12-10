---
Id: 48
Posted Date: 9/10/2017
Tags:  Localization,PO,Orchard,OrchardCore 
Slug: make-po-files-❤-localization-in-asp.net-core-2.0-using-orchard-core
---
# Portable Objects using OrchardCore

Since year and a half I blogged about [Make PO files ?? Localization in ASP.NET Core 1.0](http://hishambinateya.com/make-po-files-%E2%9D%A4-localization-in-asp.net-core-1.0), which is my first attempt to make PO works with ASP.NET Core to become an alternative option with `ResourceManager` (resx).

After 6 months from writing my blog post [Ryan Brandenburg](https://github.com/ryanbrandenburg) who is the one of the ASP.NET Core team, started a design pull request for [POStringLocalizer](https://github.com/aspnet/Localization/pull/309), which made me happy at that time, nothing but the PR is closed for unknown reason :(

Last week I saw a cool tweet from the great [Sébastien Ros](https://twitter.com/sebastienros) which he & one of the community folks added the support of PO files in [OrchardCore](https://github.com/OrchardCMS/Orchard2).
![](http://www.hishambinateya.com/images/posts/5e5210cf-9bc0-4826-ae1d-1516808c2865.png)

And after I retweeted his tweet, he wrote a nice comment asking me to write a blog post about the [OrchardCore](https://github.com/OrchardCMS/OrchardCore) and PO files support :), of course I took the initiative to write a blog post today.
![](http://www.hishambinateya.com/images/posts/b668380e-2656-4852-ac65-794ef6ffa64a.png)

So in this blog post I will not talk about an introduction to PO files because I already did in the old post that I mentioned before, instead I will talk about configuring portable object localization in the ASP.NET Core applications.

First of all you need to add a reference to the `OrchardCore.Localization.Core` from the following package source [https://www.myget.org/F/orchardcore-preview/api/v3/index.json](https://www.myget.org/F/orchardcore-preview/api/v3/index.json).

Then you need to create your PO files that contains the translation, in this case I will create _**fr.po**_ for French:
```po
    msgid "Hello world!"
    msgstr "Bonjour le monde!"
```
you can add other POs files if you need to support more than one culture, but for the sake of the demo I used one PO file,

Now it's the time to configuring the localization service, which is similar to what we are doing normally except adding `AddPortableObjectLocalization()` API.
```csharp
public void ConfigureServices(IServiceCollection services)
{
     services.AddPortableObjectLocalization();
}

public void Configure(IApplicationBuilder app)
{
     var supportedCultures = new List<CultureInfo>
     {
          new CultureInfo("en-US"),
          new CultureInfo("fr-FR")
     };
     var options = new RequestLocalizationOptions
     {
          DefaultRequestCulture = new RequestCulture("en-US"),
          SupportedCultures = supportedCultures,
          SupportedUICultures = supportedCultures
     };

     app.UseRequestLocalization(options);
     ...
}
```
FYI the above PO file added into the root of the application, but you are free to change the location of the PO files by using the interface [ILocalizationFileLocationProvider](https://github.com/OrchardCMS/OrchardCore/blob/master/src/OrchardCore/OrchardCore.Localization.Abstractions/ILocalizationFileLocationProvider.cs), which has [ContentRootPoFileLocationProvider](https://github.com/OrchardCMS/OrchardCore/blob/master/src/OrchardCore/OrchardCore.Localization.Core/ContentRootPoFileLocationProvider.cs) as a default implementation.

Last but not least we knew the PO support pluralization, but I decide to write another blog post in upcoming few days to explain everything related.

Finally at the end of this post I hope OrchardCore makes PO ?? Localization in ASP.NET Core 2.0, and nothing stop you from using PO files in ASP.NET Core anymore.

Happy Coding!!