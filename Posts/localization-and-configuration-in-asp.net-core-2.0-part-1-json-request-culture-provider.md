---
Id: 50
Posted Date: 5/11/2017
Tags:  Localization,Configuration,JSON 
Slug: localization-and-configuration-in-asp.net-core-2.0-part-1-json-request-culture-provider
---
# Localization JSON Request Culture Provider

It had been awhile since I blogged about localization stuff, today I wanna blog about how to use JSON file to specify the current cultures for the ASP.NET Core application.

As we knew **appsettings.json** is one of the famous files that every ASP.NET Core applications may have, which is store the settings for the application. One of the things that may we need to store are the culture & UI culture for the application, which is the things that I will talk about in this post.
```json
{
  "Localization": {
    "Culture": "ar-YE",
    "UICulture": "ar-YE"
  }
}
```
In the JSON file above, I declared a _Localization_ section that contains both _Culture_ and _UICulture_ keys, Now it's the time to create a `JsonRequestCultureProvider` to read the localization information from **appsettings.json** or any JSON file that you like.
```csharp
public class JsonRequestCultureProvider : RequestCultureProvider
{
    public static readonly string DefaultJsonFileName = "AppSettings.json";

    public static readonly string LocalizationSection = "Localization";

    public string JsonFileName { get; set; } = DefaultJsonFileName;

    public string CultureKey { get; set; } = "culture";

    public string UICultureKey { get; set; } = "ui-culture";

    public IConfigurationRoot Configuration { get; set; }

    public override Task<ProviderCultureResult> DetermineProviderCultureResult(HttpContext httpContext)
    {
        if (httpContext == null)
        {
            throw new ArgumentNullException();
        }

        var env = httpContext.RequestServices.GetService<IHostingEnvironment>();
        var builder = new ConfigurationBuilder()
            .SetBasePath(env.ContentRootPath)
            .AddJsonFile(JsonFileName);

        Configuration = builder.Build();

        string culture = null;
        string uiCulture = null;
        var localizationSection = Configuration.GetSection(LocalizationSection);

        if (!string.IsNullOrEmpty(CultureKey))
        {
            culture = localizationSection[CultureKey];
        }

        if (!string.IsNullOrEmpty(UICultureKey))
        {
            uiCulture = localizationSection[UICultureKey];
        }

        if (culture == null && uiCulture == null)
        {
            return Task.FromResult((ProviderCultureResult)null);
        }

        if (culture != null && uiCulture == null)
        {
            uiCulture = culture;
        }

        if (culture == null && uiCulture != null)
        {
            culture = uiCulture;
        }

        var providerResultCulture = new ProviderCultureResult(culture, uiCulture);

        return Task.FromResult(providerResultCulture);
    }
}
```
The code above is similar to other built-in request culture providers, the good part is we can use the [Configuration](http://github.com/aspnet/configuration) APIs to simplify the process of reading the localization information from the JSON file.

Finally we can use the `JsonRequestCultureProvider` easily in the `Configure()` method as the following:
```csharp
var supportedCultures = new[]
{
    new CultureInfo("en-US"),
    new CultureInfo("ar-YE")
};

var options = new RequestLocalizationOptions
{
    DefaultRequestCulture = new RequestCulture("en-US"),
    SupportedCultures = supportedCultures,
    SupportedUICultures = supportedCultures
};

options.RequestCultureProviders.Insert(0, new JsonRequestCultureProvider());
app.UseRequestLocalization(options);
```
If you look closely I inserted the newly created request culture provider as the first position to be a winning provider, so the its code can be executed before any other request culture provider.

You can download the source code for this post from my [My.AspNetCore.Localization.Json](https://github.com/hishamco/My.AspNetCore.Localization.Json) repository on GitHub.

Happy Coding ...