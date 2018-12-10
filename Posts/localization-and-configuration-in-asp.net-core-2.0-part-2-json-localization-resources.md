---
Id: 51
Posted Date: 7/11/2017
Tags:  Localization,Configuration,JSON 
Slug: localization-and-configuration-in-asp.net-core-2.0-part-2-json-localization-resources
---
# Localization & Configuration in ASP.NET Core 2.0: Part 2 - JSON Localization Resources

In the previous blog post [Localization & Configuration in ASP.NET Core 2.0: Part 1 - JSON Request Culture Provider](http://www.hishambinateya.com/localization-and-configuration-in-asp.net-core-2.0-part-1-json-request-culture-provider) I showed how to use JSON files to determine the application culture. Today I will move forward to see how we can manage the localization resources in JSON files instead of resx files which is the only option that we have nowdays.

### Why JSON not Resx?

As I mentioned before ASP.NET Core 1.0 uses a `ResourceManager` to manage the resources that have been stored in **resx** files, this has some advantages and disadvantages.

IMHO the main advantage is to support backward compatibility, so no need to change the resource files at all, specially if you migrate your old application to ASP.NET Core.

The main disadvantage which I personally suffer - perhaps many of you - that I need to re-compile the application every time when I change or add a resource, which is very very bad, but that's how **resx** files works.

In other hand JSON file is flexible and don't have the restriction of the **resx** files, so it doesn't need to be compiled, also we can use the [Configuration](http://github.com/aspnet/configuration) APIs to take the advantage for reloading the file when it's changed by setting the `reloadOnChange` parameter.

### Introducing JsonStringLocalizer

Now it's the time to create the `JsonStringLocalizer`, in the code snippet underneath I created a `StringLocalizer` that read the resource from JSON files with the help of the [Configuration](http://github.com/aspnet/configuration) APIs.
```csharp
public class JsonStringLocalizer : IStringLocalizer
{
    private readonly ConcurrentDictionary<string, IEnumerable<KeyValuePair<string, string>>> _resourcesCache = new ConcurrentDictionary<string, IEnumerable<KeyValuePair<string, string>>>();
    private readonly string _resourcesPath;
    private readonly string _resourceName;
    private readonly ILogger _logger;

    private string _searchedLocation;

    public JsonStringLocalizer(
        string resourcesPath,
        string resourceName,
        ILogger logger)
    {
        _resourcesPath = resourcesPath ?? throw new ArgumentNullException(nameof(resourcesPath));
        _resourceName = resourceName ?? throw new ArgumentNullException(nameof(resourceName));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public LocalizedString this[string name]
    {
        get
        {
            if (name == null)
            {
                throw new ArgumentNullException(nameof(name));
            }

            var value = GetStringSafely(name);

            return new LocalizedString(name, value ?? name, resourceNotFound: value == null, searchedLocation: _searchedLocation);
        }
    }

    public LocalizedString this[string name, params object[] arguments]
    {
        get
        {
            if (name == null)
            {
                throw new ArgumentNullException(nameof(name));
            }

            var format = GetStringSafely(name);
            var value = string.Format(format ?? name, arguments);

            return new LocalizedString(name, value, resourceNotFound: format == null, searchedLocation: _searchedLocation);
        }
    }

    public IEnumerable<LocalizedString> GetAllStrings(bool includeParentCultures) =>
        GetAllStrings(includeParentCultures, CultureInfo.CurrentUICulture);

    public IStringLocalizer WithCulture(CultureInfo culture)
    {
        throw new NotImplementedException();
    }

    protected IEnumerable<LocalizedString> GetAllStrings(bool includeParentCultures, CultureInfo culture)
    {
        throw new NotImplementedException();
    }

    protected string GetStringSafely(string name)
    {
        if (name == null)
        {
            throw new ArgumentNullException(nameof(name));
        }

        var culture = CultureInfo.CurrentUICulture;
        var resources = _resourcesCache.GetOrAdd(culture.Name, _ =>
        {
            var resourceFile = $"{_resourceName}.{culture.Name}.json";
            _searchedLocation = Path.Combine(_resourcesPath, resourceFile);
            IEnumerable<KeyValuePair<string, string>> value = null;

            if (File.Exists(_searchedLocation))
            {
                var builder = new ConfigurationBuilder()
                .SetBasePath(_resourcesPath)
                .AddJsonFile(resourceFile, optional: false, reloadOnChange: true);

                var config = builder.Build();
                value = config.AsEnumerable();
            }

            return value;
        });
        var resource = resources?.SingleOrDefault(s => s.Key == name);
        _logger.SearchedLocation(name, _searchedLocation, culture);

        return resource?.Value ?? null;
    }
}
```
For better performance I used the `ConcurrentDictionary` to cache the resources instead of hitting the disk in each request.

After that you can prepare your resource files as the following:
```json
{
  "Hello": "Bonjour"
}
```
Finally you can use the `JsonStringLocalizer` by adding the following in `ConfigureServices()` method:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddJsonLocalization(options => options.ResourcesPath = "Resources");
}
```
You can download the source code for this post from my [My.Extensions.Localization.Json](https://github.com/hishamco/My.Extensions.Localization.Json) repository on GitHub.

Happy Coding!!