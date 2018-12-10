---
Id: 35
Posted Date: 26/4/2017
Tags: Platform Abstractions
Slug: goodbye-platform-abstractions
---
# Goodbye Platform Abstractions!!

The [Platform Abstraction](https://github.com/aspnet/PlatformAbstractions) was good piece of technology in ASP.NET Core 1, this component was contain a bunch of the APIs related to the application environment.

But after a long discussion [here](https://github.com/aspnet/PlatformAbstractions/issues/50) to remove this component completely from ASP.NET Core, today I would like to say goodbye Platform Abstractions!! because this component will no longer available in ASP.NET Core 2.0.

### Equivalent APIs

The first question may asked after reading the blog post title is: what is the equavalent APIs for Platform Abstractions? let me show you the replacing APIs usage below:

**PlatformServices.Default.Application.ApplicationBasePath**

`AppContext.BaseDirectory`

`AppDomain.CurrentDomain.BaseDirectory`

**PlatformServices.Default.Application.ApplicationName**

`Assembly.GetEntryAssembly().GetName().Name`

`AppDomain.CurrentDomain.SetupInformation.ApplicationName`

**PlatformServices.Default.Application.ApplicationVersion**

`Assembly.GetEntryAssembly().GetName().Version`

**PlatformServices.Default.Application.RuntimeFramework**

`Assembly.GetEntryAssembly().GetCustomAttribute<TargetFrameworkAttribute>().FrameworkName`

`AppDomain.CurrentDomain.SetupInformation.TargetFrameworkName`

IMHO this is the right choice in the right time to remove this component, because **Reflection APIs** nowadays is rich enough in .NET Core to accomplish the same goal.

Happy Coding !!