---
Id: 57
Posted Date: 17/7/2018
Tags:  Localization
Slug: localization-and-generics
---
# Localization & Generics

It has been awhile since I wrote about localization stuff, today I wanna blog about creating an `InMemoryStringLocalizer` in ASP.NET Core.

### Why In-Memory StringLocalizer?

AFAIK creating unit tests for localization is not an easy task especially creating resource files **Resx**, perhaps it is much simpler if you work with Visual Studio, but what if you not?!! ASP.NET Core doesn't enforce you to work from within Visual Studio, so you can use your favorite IDE or editor that you like.

With that we need a mechanism to create a localization resources with minimal efforts during unit testing, that 's where `InMemoryStringLocalizer` shine.

Another usage for the `InMemoryStringLocalizer` is creating a demos or experimental localization APIs, so no need to bother yourself by creating **Resx** file, you can do as fast as possible now.

### What you need to use In-Memory StringLocalizer?

Simple what you need to work with the `InMemoryStringLocalizer` is nothing but using `AddInMemoryLocalization()` in `ConfigureService()` method as the following:
```csharp
services.AddInMemoryLocalization(new List<Resource>
{
    new Resource("en-US", "Hello", "Hello"),
    new Resource("fr-FR", "Hello", "Bonjour"),
});
```
After that you can use the localization APIs in the same way of using them with `ResourceManagerStringLocalizer`.

### Deep dive into In-Memory StringLocalizer

First I create `InMemoryStringLocalizerFactory` which responsible for creating `InMemoryStringLocalizer`.
```csharp
public class InMemoryStringLocalizerFactory : IStringLocalizerFactory
{
    private readonly IList<Resource> _resources;

    public InMemoryStringLocalizerFactory(
        IList<Resource> resources)
    {
        _resources = resources ?? throw new ArgumentNullException(nameof(resources));
    }

    public IStringLocalizer Create(Type resourceSource)
        => new InMemoryStringLocalizer(\_resources);

    public IStringLocalizer Create(string baseName, string location)
        => new InMemoryStringLocalizer(\_resources);
}
```
Then the `InMemoryStringLocalizer` uses a `List<Resource>` in memory, it could be `Dictionary`, to store all the user define resources.
```csharp
public class InMemoryStringLocalizer : IStringLocalizer
{
    private readonly CultureInfo _culture;
    private readonly IList<Resource> _resources;

    public InMemoryStringLocalizer(
        IList<Resource> resources)
    {
        _resources = resources ?? throw new ArgumentNullException(nameof(resources));
    }

    public InMemoryStringLocalizer(
        IList<Resource> resources,
        CultureInfo culture)
    {
        _resources = resources ?? throw new ArgumentNullException(nameof(resources));
        _culture = culture ?? throw new ArgumentNullException(nameof(culture));
    }

    public virtual LocalizedString this[string name]
    {
        get
        {
            if (name == null)
            {
                throw new ArgumentNullException(nameof(name));
            }

            var culture = _culture ?? CultureInfo.CurrentUICulture;
            var value = _resources.SingleOrDefault(r => r.Culture == culture.Name && r.Key == name)?.Value;

            return new LocalizedString(name, value ?? name, resourceNotFound: value == null);
        }
    }

    public virtual LocalizedString this[string name, params object[] arguments]
    {
        get
        {
            if (name == null)
            {
                throw new ArgumentNullException(nameof(name));
            }

            var culture = _culture ?? CultureInfo.CurrentUICulture;
            var format = _resources.SingleOrDefault(r => r.Culture == culture.Name && r.Key == name)?.Value;
            var value = string.Format(format ?? name, arguments);

            return new LocalizedString(name, value, resourceNotFound: format == null);
        }
    }

    public IStringLocalizer WithCulture(CultureInfo culture)
        => new InMemoryStringLocalizer(_resources, culture);

    public IEnumerable<LocalizedString> GetAllStrings(bool includeParentCultures)
        => _resources.Select(r => new LocalizedString(r.Key, r.Value, true)).ToList();
}
```
Finally, it is the time to add an extension method to register the new localizer.
```csharp
public static IServiceCollection AddInMemoryLocalization(this IServiceCollection services, IList<Resource> resources)
{
    if (services == null)
    {
        throw new ArgumentNullException(nameof(services));
    }

    if (resources == null)
    {
        throw new ArgumentNullException(nameof(resources));
    }

    services.TryAddSingleton<IStringLocalizerFactory>(new InMemoryStringLocalizerFactory(resources));
    services.TryAddTransient(typeof(IStringLocalizer<>), typeof(StringLocalizer<>));

    return services;
}
```
For more details you can check the source code in my [localization repository](https://github.com/hishamco/Localization/tree/inMemoryStringLocalizer).

Happy Coding!!