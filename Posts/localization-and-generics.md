---
Id: 58
Posted Date: 27/8/2018
Tags:  Localization,Generics
Slug: localization-and-generics
---
# Localization & Generics

Around two weeks ago I saw an issue in the [home repository](https://github.com/aspnet/Home/issues/3422), that localization not working with generic class. With that I think it would be nice to write a blog post that explains how to localize generic classes.

### Why we need to localize a generic class?

At first glance I asked myself why we need localization with generics, because I never faced a use case that require to localize a generic class since I started localization in ASP.NET Core. So after digging with the earlier issue, it seems we need to localize generic classes in some cases, one of them is when we use Identity classes such as: `ApplicationUser<T>`.

### How to localize a generic class?

When you start to localize a generic class the first issue that you will face is resource naming!! assume we have a class `AppIdentityUser<T>`, the first two options may raise in your mind as a developer for French culture are:

*   **AppIdentityUser.fr.resx**
*   **AppIdentityUser<T>.fr.resx**

no one of them will work, even though you are using **AppIdentityUser`1.fr.resx** which is the name got by reflection?!!

It's strange little bit for me when no one of the above resource names works :( for that I start to dive into the localization source and start debugging to discover what is the name is generated to look for the **.resx**  file. After awhile I saw that the _basename_ that used for such resource is used a fully qualified name for the type, so if we have `AppIdentityUser<String>` for instance the resource name will be **AppidentityUser`1[[System.String, System.Private.CoreLib, Version=4.0.0.0, Culture=neutral, PublicKeyToken=7cec85d7bea7798e]].fr.resx** What?!! I knew the name is weird but this how the generic class will be named, and you need n number of resources based on your how many generic types you have.

For that I start think out-of-the-box for providing an option that allow us to rid off the fully qualified name if it needed, to be clear I'm not against provide such name, perhaps it useful in some cases, but in many cases I see the type name is a good option without worrying about resource naming.

What I come up with is simple and straightforward, I striped everything from the generated _basename_ except the type name, so the resource name will be same as usual.
```csharp
public class LocalizationOptions
{
    public string ResourcesPath { get; set; } = string.Empty;

    public bool UseGenericResources { get; set; } = true;
}
```
I just added a new property in the `LocalizationOptions` named `UseGenericResources` that allow the user to choose the resource naming for the generic classes. After that I modified the generated  _basename_ in the `ResourceManagerStringLocalizerFactory` as the following:
```csharp
private readonly bool _useGenericResources;

public ResourceManagerStringLocalizerFactory(
    IOptions<LocalizationOptions> localizationOptions,
    ILoggerFactory loggerFactory)
{
    if (localizationOptions == null)
    {
        throw new ArgumentNullException(nameof(localizationOptions));
    }

    if (loggerFactory == null)
    {
        throw new ArgumentNullException(nameof(loggerFactory));
    }

    _resourcesRelativePath = localizationOptions.Value.ResourcesPath ?? string.Empty;
    _useGenericResources = localizationOptions.Value.UseGenericResources;
    _loggerFactory = loggerFactory;

    if (!string.IsNullOrEmpty(_resourcesRelativePath))
    {
        _resourcesRelativePath = _resourcesRelativePath.Replace(Path.AltDirectorySeparatorChar, '.')
            .Replace(Path.DirectorySeparatorChar, '.') + ".";
    }
}

public IStringLocalizer Create(Type resourceSource)
{
    if (resourceSource == null)
    {
        throw new ArgumentNullException(nameof(resourceSource));
    }

    var typeInfo = resourceSource.GetTypeInfo();

    var baseName = GetResourcePrefix(typeInfo);
    if (!_useGenericResources)
    {
        if (baseName.Contains("`1"))
        {
            var genericStartIndex = baseName.IndexOf("`1");
            baseName = baseName.Substring(0, genericStartIndex);
        }
    } 
    var assembly = typeInfo.Assembly;

    return _localizerCache.GetOrAdd(baseName, _ => CreateResourceManagerStringLocalizer(assembly, baseName));
}

public IStringLocalizer Create(string baseName, string location)
{
    if (baseName == null)
    {
        throw new ArgumentNullException(nameof(baseName));
    }
}
```
That's it .. now you're free to use generic class resources either with what ASP.NET Core localization resource name that provide or using the type name by toggling one property in `ConfigureServices()` method as the following:

```csharp
services.AddLocalization(options =>
{
    options.ResourcesPath = "Resources";
    options.UseGenericResources = false;
});
```
Happy Coding!!