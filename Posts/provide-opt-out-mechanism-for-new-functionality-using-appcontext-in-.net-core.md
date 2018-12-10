---
Id: 34
Posted Date: 25/4/2017
Tags: Tips & Tricks,.NET Core,AppContext
Slug: provide-opt-out-mechanism-for-new-functionality-using-appcontext-in-.net-core
---
# Provide opt-out mechanism for new functionality using AppContext in .NET Core

The `AppContext` class provides members for setting and getting information about an application's context. Today I want to talk about something that you might not know about the `AppContext`.

### Compatibility Switches

The `AppContext` enables the library authors to provider an opt-out mechanism for their new functionality. This is typically important when change is made to existing functionality.

With `AppContext` libraries define and expose compatibility switches, while code that depends on them can set those switches, to affect the library behavior. By default libraries provide the new functionality and only modify it if the switch is set.

Let us clarify what I said using a real example ..

Assume that I'm developing a _LocalizationLibrary_, in the version 1.0 the library contains the following class:
```csharp
public static class App
{
    public static IList<string> SupportedCultures => new List<string> { "ar", "en", "fr" };

    public static void SetLocale(string cultureName)
    {
         if(SupportedCultures.Contains(cultureName))
         {
              ...
         }
         else
         {
              throw new NotSupportedException("The culture is not supported.");
         }
    }
    ...
}
```
`SetLocale()` is responsible for setting the current culture for the application if the provided culture is supported.
```csharp
App.SetLocale("fr");
```
the above code snippet should works as expected and set the locale to French, nothing but if we executing the code underneath the `NotSupportedException()` will be thrown, because the `Contains()` is case sensitive.
```csharp
App.SetLocale("Fr");
```
In the version 2.0 of my _LocalizationLlibrary_ I want to fix the culture case sensitive comparison issue without breaking changes!! this is seems crazy little bit. In this case we can use `AppContext` compatibility feature by defining **LocalizationLibrary.IsCultureCaseInsensitive** switch to check whether the culture case sensitive or not. This ensures that all the APIs depend on the original behavior will still work.
```csharp
public static void SetLocale(string cultureName)
{
    bool isFound;

    if (AppContext.TryGetSwitch("LocalizationLibrary.IsCultureCaseInsensitive", out var isEnabled) && isEnabled == true)
    {
        isFound = SupportedCultures.Contains(cultureName, StringComparer.OrdinalIgnoreCase);
    }
    else
    {
        isFound = SupportedCultures.Contains(cultureName);
    }

    if (isFound)
    {
        Console.WriteLine($"Culture: {cultureName}");
    }
    else
    {
        throw new NotSupportedException("The culture is not supported.");
    }
}
```
in the above code snippet I used `TryGetSwitch()` to check the existence of the switch in the `runtimeconfig.template.json` , after that I need to check `isEnabled` parameter to get the actual value of the defined switch, based on that we can run both the old and new behavior by toggling the switch value in the `runtimeconfig.template.json` file.
```json
{
  "configProperties": {
    "LocalizationLibrary.IsCultureCaseInsensitive": "false"
  }
}
```
As we seen before the compatibility switches is a cool feature of `AppContext` to enable quirks mode, this same infrastructure is used by the .NET Framework internally, to enable developers to opt-out of updates to existing functionality. For more information you please checkout the [Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/api/system.appcontext?view=netframework-4.7).

Happy Coding !!