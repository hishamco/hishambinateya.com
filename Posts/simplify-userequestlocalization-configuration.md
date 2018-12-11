---
Id: 26
Posted Date: 12/2/2017
Tags: Localization,Fluent APIs
Slug: why-you-arenot-using-imiddlewaresimplify-userequestlocalization-configuration
---
# Simplify UseRequestLocalization Configuration

The [Localization](https://github.com/aspnet/Localization) repository is one of my favorite ASP.NET Core repositories, but the thing that I dislike is the way to configure it using `UseRequestLocalization` which is used the options setter delegate pattern, I don't have a doubt about the pattern itself, but I hate the much of the code that I need to write.

For that I was thinking to write a Fluent APIs to simplify the `UseRequestLocalization` configuration, with that we can introduce three extensions methods: `AddSupportedCultures`, `AddSupportedUICultures` and `SetDefaultCulture` as the following:
```csharp
public static class RequestLocalizationOptionsExtensions
{
    public static RequestLocalizationOptions AddSupportedCultures(
        this RequestLocalizationOptions options,
        params string[] cultures)
    {
        var supportedCultures = new List<CultureInfo>();
        foreach (var culture in cultures)
        {
            supportedCultures.Add(new CultureInfo(culture));
        }
        options.SupportedCultures = supportedCultures;
        return options;
    }

    public static RequestLocalizationOptions AddSupportedUICultures(
        this RequestLocalizationOptions options,
        params string[] uiCultures)
    {
        var supportedUICultures = new List<CultureInfo>();
        foreach (var culture in uiCultures)
        {
            supportedUICultures.Add(new CultureInfo(culture));
        }
        options.SupportedUICultures = supportedUICultures;
        return options;
    }

    public static RequestLocalizationOptions SetDefaultCulture(
        this RequestLocalizationOptions options,
        string defaultCulture)
    {
        options.DefaultRequestCulture = new RequestCulture(defaultCulture);
        return options;
    }
}
```
Then we need to add `ApplicationBuilderExtensions` to make use of them as the following:  
```csharp
public static class ApplicationBuilderExtensions
{
    public static IApplicationBuilder UseRequestLocalization(
        this IApplicationBuilder app,
        Action<RequestLocalizationOptions> optionsAction)
    {
        if (app == null)
        {
            throw new ArgumentNullException(nameof(app));
        }
        
        if (optionsAction == null)
        {
            throw new ArgumentNullException(nameof(optionsAction));
        }
        
        var options = new RequestLocalizationOptions();
        optionsAction.Invoke(options);
        return app.UseMiddleware<RequestLocalizationMiddleware>(Options.Create(options));
    }

    public static IApplicationBuilder UseRequestLocalization(
        this IApplicationBuilder app,
        params string[] cultures)
    {
        if (app == null)
        {
            throw new ArgumentNullException(nameof(app));
        }

        if (cultures == null)
        {
            throw new ArgumentNullException(nameof(cultures));
        }

        if (cultures.Length == 0)
        {
            throw new ArgumentException(nameof(cultures));
        }
        var options = new RequestLocalizationOptions()
            .AddSupportedCultures(cultures)
            .AddSupportedUICultures(cultures)
            .SetDefaultCulture(cultures[0]);
            
        return app.UseMiddleware<RequestLocalizationMiddleware>(Options.Create(options));
    }
}
```
Now we can use the simplified version in the `Configure` method as the following:
```csharp
app.UseRequestLocalization(options =>
    options
        .AddSupportedCultures("en-US", "fr-FR")
        .AddSupportedUICultures("en-US", "fr-FR")
        .SetDefaultCulture("en-US")   
);
```
Also we can go further by passing a `params` which represents supported formats and UI cultures, the first param represents the default culture.
```csharp
app.UseRequestLocalization("en-US", "fr-FR");
```
**In addition, this would be nice in order to avoid the namespace imports for**

using System.Collections.Generic;
using System.Globalization;

You can download the source code for this post from my [Localization](https://github.com/hishamco/localization/tree/useRequestLocalization) repository on GitHub.

Happy Coding ...