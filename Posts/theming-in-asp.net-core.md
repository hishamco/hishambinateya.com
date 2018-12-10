---
Id: 38
Posted Date: 5/6/2017
Tags: Themes 
Slug: theming-in-asp.net-core
---
# Theming in ASP.NET Core

Themes allow you to define the look and feel of your pages and then apply the look consistently across pages in a web application. In previous versions of ASP.NET specifically WebForms we already have the concept of Themes & Skins which give us the ability to change the look and feel of controls & pages within the web application. Unfortunately we can't use this because the WebForms is not supported yet in ASP.NET Core.

In this blog post I wanna explore how can we apply the theming in ASP.NET Core.

### ViewLocationExpanders

The view location expanders is one of those concepts that perhaps not known to many of us, ASP.NET Core provide us with a simple contract named [`IViewLocationExpander`](https://github.com/aspnet/Mvc/blob/760c8f38678118734399c58c2dac981ea6e47046/src/Microsoft.AspNetCore.Mvc.Razor/IViewLocationExpander.cs) which is basically used to determine searched paths for a view.

If I'm not wrong ASP.NET out-of-the-box have two implementations for this contract, [PageViewLocationExpander](https://github.com/aspnet/Mvc/blob/a8eb5bee7023a2a5d49fcdaea0c25b3416a69211/src/Microsoft.AspNetCore.Mvc.RazorPages/Infrastructure/PageViewLocationExpander.cs) which is used for determine the RazorPages path, and [`LanguageViewLocationExpander`](https://github.com/aspnet/Mvc/blob/18785dbed6ef63990f1732c509a19cf8ef533cb5/src/Microsoft.AspNetCore.Mvc.Razor/LanguageViewLocationExpander.cs) which is responsible for determine the path of the views specially when we 're using localization.

In this post seems the ViewLocationExpander is the right option to apply the theming in any ASP.NET Core web application, the main idea here is to tweak the view location paths to let the Razor engine looks to the themes folder instead of the default view locations.

Let us dig into that ...

First of all we need to create _AppSettings_ section in the **appSettings.json** and add the _Theme_ property to hold the default theme name that will be used for our web application.
```json
{
  "AppSettings": {
    "Theme": "LightTheme"
  },
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Debug",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
}
```
Then we need to create `IViewLocationExpander` implementation to set the view locations that the razor engine will looks for all the views.
```csharp
public class ThemeViewLocationExpander : IViewLocationExpander
{
    private const string ValueKey = "theme";

    public IEnumerable<string> ExpandViewLocations(ViewLocationExpanderContext context, IEnumerable<string> viewLocations)
    {
        if (context == null)
        {
            throw new ArgumentNullException(nameof(context));
        }

        if (viewLocations == null)
        {
            throw new ArgumentNullException(nameof(viewLocations));
        }

        context.Values.TryGetValue(ValueKey, out string theme);

        if (!string.IsNullOrEmpty(theme))
        {
            return ExpandViewLocationsCore(viewLocations, theme);
        }

        return viewLocations;
    }

    public void PopulateValues(ViewLocationExpanderContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException(nameof(context));
        }

        var appSettings = context.ActionContext.HttpContext.RequestServices
            .GetService(typeof(IOptions<AppSettings>)) as IOptions<AppSettings>;

        context.Values[ValueKey] = appSettings.Value.Theme;
    }

    private IEnumerable<string> ExpandViewLocationsCore(IEnumerable<string> viewLocations, string theme)
    {
        foreach (var location in viewLocations)
        {
            yield return location.Insert(7, $"Themes/{theme}/");
        }
    }
}
```
So if you notice in the code snippet before in the `ExpandViewLocations()` method that I'm inserting the **Themes/{ThemeName}** in the position 7 of current view location, which is mainly after the **/Views/** token, so that means the razor engine will no longer looking for the views into the default locations, instead it will look inside the theme location.

After that in the `ConfigureServices()` method we need to bind the _AppSettings_ section from the configuration file to the `AppSettings` class also we need to add the `ThemeViewLocationExpander` that I created before to the `ViewLocationExpanders` property of the `RazorViewEngineOptions.`
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<AppSettings>(Configuration.GetSection("AppSettings"));

    services.AddMvc();

    services.Configure<RazorViewEngineOptions>(options =>
    {
        options.ViewLocationExpanders.Add(new ThemeViewLocationExpander());
    });
}
```
The last trick I made in the `Configure()` method to let the static files be serving from the root by using the `StaticFileOptions` with setting `RequestPath` property to empty string
```csharp
var appSettings = app.ApplicationServices.GetRequiredService<IOptions<AppSettings>>();

app.UseStaticFiles(new StaticFileOptions()
{
    FileProvider = new PhysicalFileProvider(Path.Combine(Directory.GetCurrentDirectory(),
        $@"Views/Themes/{appSettings.Value.Theme}")),
    RequestPath = string.Empty
});
```
Finally when you run the application the themes should look like the following:

**Light Theme**

![](http://www.hishambinateya.com/images/posts/06b0299d-d916-40bf-9b1e-f31039f7620b.png)

**Dark Theme**

**![](http://www.hishambinateya.com/images/posts/5ae73252-a678-4403-9f08-6a3fac7f6c89.png)**

You can download the source code for this post from my [Theming](http://github.com/hishamco/theming) repository on GitHub.

Happy Coding !!