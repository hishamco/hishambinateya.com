---
Id: 46
Posted Date: 11/9/2017
Tags:  Localization,Razor Pages 
Slug: localization-issue-in-razorpages
---
# Localization Issue in RazorPages

Yesterday when I started to test my `JsonRequestCultureProvider` that I made to solve an issue in [Localization repository](http://github.com/aspnet/localization), I surprisingly shocked!! when the localization doesn't work as expected in the RazorPages, I spent hours and hours digging around with no luck :( after awhile I tried to use `IViewLocalizer` instead of `IStringLocalizer`, the progress improved little bit but again I got an **IndexOutOfRangeException** which made me crazy :) I checked my code, everything seems as expected, but the localization doesn't work!!

Later on today I was searching in the GitHub I realized that bug is already known, especially after the RazorPages was introduced in ASP.NET Core 2.0, the main reason for that is the property `ViewContext.ExecutingFilePath` is not set at all in original source code.

In this very short post I would like to share the workaround by [Pranav K](https://github.com/pranavkm) to fix that annoying bug. Basically the idea is to replace the `DefaultPageFactoryProvider` with a newly one after setting the  `ViewContext.ExecutingFilePath` property.
```csharp
public class LocalizationFixPageFactoryProvider : DefaultPageFactoryProvider
{
    public LocalizationPageFactoryProvider(
        IPageActivatorProvider pageActivator,
        IModelMetadataProvider metadataProvider,
        IUrlHelperFactory urlHelperFactory,
        IJsonHelper jsonHelper,
        DiagnosticSource diagnosticSource,
        HtmlEncoder htmlEncoder,
        IModelExpressionProvider modelExpressionProvider)
        : base(pageActivator, metadataProvider, urlHelperFactory, jsonHelper, diagnosticSource, htmlEncoder, modelExpressionProvider)
    {
    }

    public override Func<PageContext, ViewContext, object> CreatePageFactory(CompiledPageActionDescriptor actionDescriptor)
    {
        var result = base.CreatePageFactory(actionDescriptor);
        return (pageContext, viewContext) =>
        {
            viewContext.ExecutingFilePath = actionDescriptor.RelativePath;
            return result(pageContext, viewContext);
        };
    }
}
```
and inject the `LocalizationFixPageFactoryProvider` in the `ConfigureServices()` method as the following:
```csharp
services.AddSingleton<IPageFactoryProvider, LocalizationFixPageFactoryProvider>();
```
Finally the localization works like a charm!! hope we can a permanent fix in the near future in the ASP.NET Core 2.1.0 release.

Happy Coding!!