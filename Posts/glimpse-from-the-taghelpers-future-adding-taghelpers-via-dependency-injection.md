---
Id: 32
Posted Date: 4/4/2014
Tags: TagHelpers,TagHelperComponents,Razor,Dependency Injection
Slug: glimpse-from-the-taghelpers-future-adding-taghelpers-via-dependency-injection
---
# Glimpse from the TagHelpers Future - Adding TagHelpers via Dependency Injection

Last week when I asked [Damian Edwards](https://twitter.com/DamianEdwards) how to add TagHelper via dependency injection (DI), he referred me to a new feature called **TagHelperComponent** that will be shipped in ASP.NET Core 2.0.

Perhaps it is too early, but I would like to give you some glimpse of the new TagHelperComponents

### What are TagHelperComponents?

TagHelperComponents are effectively to add TagHelpers via DI system, however they are not fully fledged TagHelpers, TagHelperComponents enable you to have some certain types of TagHelpers like things light up inside the application automatically.

### Where are TagHelperComponent fits?

As we know TagHelpers should participate in the view compilation, so that''s why you can''t put them in DI, and the should be declared in the razor file. Now we can TagHelperComponents to add TagHelpers into DI with some limitations.  

As [Damian Edwards](https://twitter.com/DamianEdwards) said:

> In ASP.NET Core 2.0 the TagHelperComponents specifically will be enabled in two places: head & body tags

Why these places only?!! If you are using an ASP.NET Core for time you will probably notice that default template has several Application Insights scripts, which is similar to this:

@inject Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet AppInsightsJs

@Html.Raw(AppInsightsJs.FullScript)

also there are few other places. The issue that there are many profiling & diagnostics libraries such as [Glimpse](https://github.com/Glimpse/Glimpse), [Application Insights](https://github.com/Microsoft/ApplicationInsights-aspnetcore) and much more, need to inject some neccesary scripts in the `head` or `body` tags. So `TagHelperComponent` come for rescue to automate the process of injecting needed scripts & styles, that''s I think why ASP.NET Core team have decided to enable the `head` or `body` tags in 2.0 release.

### TagHelperComponent in Action

As I mentioned before that `TagHelperComponent` very useful in profiling & diagnostics libraries, but not only that there are many other libraries they need sort type of script injection such as Google Analytics .

Surprisingly I worked two months ago on a small project called `jsLogger` which I blogged about it here [Integrate JavaScript Logging with ASP.NET Core Logging APIs](http://www.hishambinateya.com/integrate-javascript-logging-with-asp.net-core-logging-apis) basically I need to inject some JavaScript code to send the logs to the server.

This is a part of how the layout file was:
```html
@inject JavaScriptLoggingSnippet JavaScriptLoggingSnippet
 
 <head>
     ...
    @Html.Raw(JavaScriptLoggingSnippet.Script)
 </head>
```
I should use the DI and inject the script where ever I want myself, but no longer anymore!! today I modified the source code to use the newly added `TagHelperComponent` as the following:
```csharp
public class JavaScriptLoggingTagHelperComponent : TagHelperComponent
{
    private readonly string _script;

    public JavaScriptLoggingTagHelperComponent(JavaScriptLoggingSnippet jsLoggingSnippet)
    {
        _script = jsLoggingSnippet.Script;
    }

    public override int Order => 1;

    public override Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        if (string.Equals(context.TagName, "head", StringComparison.Ordinal))
        {
            output.PostContent.AppendHtml(_script);
        }

        return Task.CompletedTask;
    }
}
```
After that we can add the `JavaScriptLoggingTagHelperComponent` to the DI container in `ConfigureServices()` method

services.AddSingleton<ITagHelperComponent, JavaScriptLoggingTagHelperComponent>();

Now I no longer need to maintain the injected JavaScript stuff in my view, because the script will be injected at runtime with the help of the `TagHelperComponent`.

Perhaps what I showed you before is not enough to understand the need of such feature, but what about this!!
```html
<script type="text/javascript">
    (function (i, s, o, g, r, a, m) {
        i\[''GoogleAnalyticsObject''\] = r; i\[r\] = i\[r\] || function () {
            (i\[r\].q = i\[r\].q || \[\]).push(arguments)
        }, i\[r\].l = 1 \* new Date(); a = s.createElement(o),
            m = s.getElementsByTagName(o)\[0\]; a.async = 1; a.src = g; m.parentNode.insertBefore(a, m)
    })(window, document, ''script'', ''https://www.google-analytics.com/analytics.js'', ''ga'');

    ga(''create'', ''UA-61337531-1'', ''auto'', { ''name'': ''mscTracker'' });
    ga(''create'', ''UA-61337531-4'', ''auto'', { ''name'': ''liveaspnetTracker'' });
    ga(''mscTracker.send'', ''pageview'');
    ga(''liveaspnetTracker.send'', ''pageview'');
</script>
```
this is a snippet from [live.asp.net code](https://github.com/aspnet/live.asp.net/blob/dev/src/live.asp.net/Views/Shared/_AnalyticsHead.cshtml#L4-L16) where Google Analytics required a script to be injected in the `head` tag, FYI this just the first  part of the entire script needed, the other part in the [\_AnalyticsBody.cshtml](https://github.com/aspnet/live.asp.net/blob/dev/src/live.asp.net/Views/Shared/_AnalyticsBody.cshtml). Of course you wanna need some mechanism to inject these scripts in your behalf :) this is where `TagHelperComponent` shine.

You can look to the updated source code for my [jsLogger](https://github.com/hishamco/jsLogger/commit/277a9ba46ddcef813b91036e616925c019171148) repository on GitHub.

Happy Coding !!