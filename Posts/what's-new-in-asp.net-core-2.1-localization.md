---
Id: 55
Posted Date: 16/6/2018
Tags:  Localization,ASP.NET Core 2.1 
Slug: what's-new-in-asp.net-core-2.1-localization
---
# What's new in ASP.NET Core 2.1 Localization

It has been a while since I wrote my last post, I was too busy last few months. but I'm back again ASP.NET Core 2.1 is out, new release .. new stuff .. Today I want to mention to you guys what is new in localization that not everyone knows about in this release.

### 1- Allow setting RootNamespace in ResourceLocationAttribute

The first one was originally raised by an old bug, the localization is not working after changing the _rootnamespace_. Then fix was finally came up in this release by introducing a new `ResourceLocaltionAttribute` which allows you to specify the _rootnamespace_ at runtime to avoid renaming _rootnamespace_ issue after creating the project.

### 2- Add a builder APIs for configuring UseRequestLocalization

I like the second one not because I made a PR for :) but because it's really simplify the creation for localization options in `UseRequestLocalization()`, in the previous version you need to write something like:
```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en-AU"),
    new CultureInfo("fr-FR")
};
var options = new RequestLocalizationOptions    
{    
    DefaultRequestCulture = new RequestCulture("en-US"),    
    SupportedCultures = supportedCultures,    
    SupportedUICultures = supportedCultures    
};
app.UseRequestLocalization(options);
```
to setup the localization options, but now you can simplify it builder API:
```csharp
var supportedCultures = new [] { "en-US", "en-AU", "fr-FR" };
app.UseRequestLocalization(options =>
    options
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures)
        .SetDefaultCulture(supportedCultures\[0\])
);

Or

app.UseRequestLocalization("en-US", "en-AU", "fr-FR");
```
### 3- Log diagnostic information for searched localization resources

The last one is tiny, but very very useful. In the past if a resource is missed for any reason the user will not get any kind of information of what's going on, which is take time for debugging, but now the localization APIs write a debug log that shows where we searched for a given resource.