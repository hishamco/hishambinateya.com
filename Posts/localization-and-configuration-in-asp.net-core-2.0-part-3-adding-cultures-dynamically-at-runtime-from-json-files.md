---
Id: 52
Posted Date: 27/11/2017
Tags:  Localization,Configuration,JSON 
Slug: localization-and-configuration-in-asp.net-core-2.0-part-3-adding-cultures-dynamically-at-runtime-from-json-files
---
# Localization & Configuration in ASP.NET Core 2.0: Part 3 - Adding Cultures Dynamically at Runtime From JSON Files

It's around 3 weeks from last blog I wrote, because I was facing some technical & Internet issues. In the last two blog posts I planned to write two parts about Localization & Configuration specifically with JSON, but yesterday I planned to write another part that uses the goodness of these two blog posts.

If we did a quick recap, in the first part [Localization & Configuration in ASP.NET Core 2.0: Part 1 - JSON Request Culture Provider](http://www.hishambinateya.com/localization-and-configuration-in-asp.net-core-2.0-part-1-json-request-culture-provider) I talked about how to get the localization cultures information from JSON file, while in the second part [Localization & Configuration in ASP.NET Core 2.0: Part 2 - JSON Localization Resources](http://www.hishambinateya.com/localization-and-configuration-in-asp.net-core-2.0-part-2-json-localization-resources) I showed you how to use the localization resources from withing a JSON file.

Today I wanna close this series by writing a short blog post of how to add cultures dynamically at runtime from within the JSON file, using the [Configuration](http://github.com/aspnet/configuration) APIs.

As we know in all the localization application written using ASP.NET Core we need to add the `UseRequestLocalization()` middleware to the ASP.NET pipeline as the following:
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

app.UseRequestLocalization(options);
```
In case that we need to add new `RequestCultureProvider` as we had seen in the part 1, we should add the following line before adding the `UseRequestLocalization()` middleware to the ASP.NET pipeline.
```csharp
options.RequestCultureProviders.Insert(0, new JsonRequestCultureProvider());
```
Now we can move both supported culture & supported UI cultures into a **appSettings.json** as the following:
```json
{
  "Localization": {
    "Culture": "ar-YE",
    "UICulture": "ar-YE",
    "DefaultRequestCulture": "en-US",
    "SupportedCultures": [ "en-US", "ar-YE" ],
    "SupportedUICultures": [ "en-US", "ar-YE" ]
  }
}
```
Now it's the time to fetch these values and inject it to the `RequestLocalizationMiddleware` by adding a new extension method as the following:
```csharp
public static IApplicationBuilder WithDynamicCultures(this IApplicationBuilder app)
{
    var env = app.ApplicationServices.GetService<IHostingEnvironment>();
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("AppSettings.json");
    var config = builder.Build();
    var defaultCulture = config["Localization:DefaultCulture"];
    var cultures = config.GetSection("Localization:SupportedCultures")
        .AsEnumerable().Skip(1).ToList();
    var uiCultures = config.GetSection("Localization:SupportedUICultures")
        .AsEnumerable().Skip(1).ToList();
    var defaultRequestCulture = new RequestCulture(defaultCulture);
    var supportedCultures = new List<CultureInfo>();
    var supportedUICultures = new List<CultureInfo>();
    if (cultures.Count > 0)
    {
        supportedCultures = cultures.Select(i => new CultureInfo(i.Value)).ToList();
    }

    if (uiCultures.Count > 0)
    {
        supportedUICultures = uiCultures.Select(i => new CultureInfo(i.Value)).ToList();
    }

    var localizationOptions = app.ApplicationServices.GetService<IOptions<RequestLocalizationOptions>>().Value;
    localizationOptions.DefaultRequestCulture = defaultRequestCulture;
    localizationOptions.SupportedCultures = supportedCultures;
    localizationOptions.SupportedUICultures = supportedUICultures;

    return app.UseMiddleware<RequestLocalizationMiddleware>(Options.Create(localizationOptions));
}
```
The above code snippet is straightforward, basically it fetches the localization options values from the **appSettings.json** and filled the **RequestLocalizationOptions** instance to pass it to the `RequestLocalizationMiddleware`.

Last thing we can rewrite the first code snippet that I mentioned before to add the localization middleware as the following:
```csharp
var options = new RequestLocalizationOptions();
options.RequestCultureProviders.Insert(0, new JsonRequestCultureProvider());
app.UseRequestLocalization(options)
    .WithDynamicCultures();
```
Now you will be able to add both JSON resources and cultures dynamically at runtime.

Happy Coding!!