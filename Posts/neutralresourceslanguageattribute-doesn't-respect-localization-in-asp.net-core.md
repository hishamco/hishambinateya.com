---
Id: 18
Posted Date: 6/11/2016
Tags: Localization,Tips & Tricks 
Slug: neutralresourceslanguageattribute-doesn't-respect-localization-in-asp.net-core
---
# NeutralResourcesLanguageAttribute Doesn't Respect Localization in ASP.NET Core

I have written many localization blog posts in ASP.NET because I like it :). While I was working in [localization repo](http://github.com/aspnet/localization) and trying to help people to fix their issue, I notice that there 're many folks facing a strange issue while they are localize their web applications. The issue is the localization APIs are using the default culture even they are specifying another culture while application is running?!!

I would like to say for those folks if you come up with such situation, please check your code and the order of the middleware first. Then check you if you already set the language attribute in the **project.json** as the following:
```json
{
  "language": "fr-FR",
  "version": "1.1.0-*",
  "dependencies": {
    "Microsoft.AspNetCore.Localization": "1.1.0-*",
    "Microsoft.AspNetCore.Server.IISIntegration": "1.1.0-*",
    "Microsoft.AspNetCore.Server.Kestrel": "1.1.0-*",
    "Microsoft.Extensions.Configuration.CommandLine": "1.1.0-*",
    "Microsoft.Extensions.Localization": "1.1.0-*"
  },
  "buildOptions": {
    "emitEntryPoint": true
  },
  "publishOptions": {
    "include": [
      "web.config"
    ]
  },
  "frameworks": {
    "net451": {},
    "netcoreapp1.1": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "version": "1.1.0-*",
          "type": "platform"
        }
      }
    }
  },
  "tools": {
    "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-*"
  },
  "scripts": {
    "postpublish": "dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%"
  }
}
```
**"language": "fr-FR"** setting this attribute may causes a critical issue if you didn't aware, because this attribute is set the `NeutralResourcesLanguageAttribute` behind the scene. Basically it will do exactly what the next line did.
```csharp
[assembly:NeutralResourcesLanguage("fr-FR")]
```
For those who don't know the `NeutralResourcesLanguageAttribute` this attribute informs the `ResourceManager` of an application's default culture. Of course this is a critical issue, because if this property is set for a certain culture for example French (fr-FR), so when the `ResourceManager` trying to lookup for the resource file it will look for **Resource.resx** instead of **Resource.fr-FR.resx**. What that means?!! IF you already set the default culture using the localization APIs with non French one for example English (en-US). The `ResourceManager` will look for the same resource file for both culture, this is the main issue :(

My advice is try to avoid setting **language** attribute in **project.json** or setting `NeutralResourcesLanguageAttribute` unless it is match the default culture that you 're setting in the localization APIs, otherwise you will be in trouble.

Finally hope this post give you a tip of how can simple attribute affect the entire application, if we don't know what exactly doing behind the scene.

Happy Coding !!