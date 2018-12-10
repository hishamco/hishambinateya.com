---
Id: 40
Posted Date: 18/6/2017
Tags: IStartup,IHostingStartup,Hosting 
Slug: istartup-vs-ihostingstartup
---
# IStartup vs IHostingStartup

### IStartup Interface

Everyone who work with ASP.NET Core familiar with `Startup` class that responsible for bootstrapping the services needed by your application, as well as setting up the application pipeline. This class basically used a convention based pattern, but you can use the interface `IStartup` if you do so.

This allows you to use the conventions too if you are running in multiple environments, for more information you can have a look to one of my old blog posts [Convention-Based Application Startup in Multiple Environments](http://www.hishambinateya.com/convention-based-application-startup-in-multiple-environments).

The `IStartup` interface is very useful in many cases, especially if you need to resolve the `Startup` class from the DI container for whatever the reason. There's a very good blog post from [Flip W.](http://twitter.com/filip_woj) who wrote a post about [Resolving ASP.NET Core Startup class from the DI container](https://www.strathweb.com/2017/06/resolving-asp-net-core-startup-class-from-the-di-container/).

### IHostingStartup Interface

The thing that some of the ASP.NET Core developers doesn't know about is `IHostingStartup` that represents platform specific configuration that will be applied to a `IWebHostBuilder` when building an `IWebHost`.

To give you a real world example of `IHostingStartup`, let us look to the following:
```csharp
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseKestrel()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseIISIntegration()
        .UseStartup()
        **.UseApplicationInsights()**
        .Build();

    host.Run();
}
```
which the **Program.cs** code that we usually wrote, if you look closely to `UseApplicationInsights()` extension method, you will notice that the previous version of ASP.NET Core applications have a dependency to **Microsoft Application Insights**, but this is changed in ASP.NET Core 2.0 Preview basically you don't have to do anything, the application does not have a dependency on Application Insights anymore, but still you have the usual application insights telemetry in Visual Studio.

How this work? if we look closely to the [ApplicationInsightsHostingStartup](https://github.com/aspnet/AzureIntegration/blob/bcc563a62b94a396009a3bce6c3ac03a9ba91c3e/src/Microsoft.AspNetCore.ApplicationInsights.HostingStartup/ApplicationInsightsStartupLoader.cs) code
```csharp
public class ApplicationInsightsHostingStartup : IHostingStartup
{
    private const string ApplicationInsightsSettingsFile = "ApplicationInsights.settings.json";

    public void Configure(IWebHostBuilder builder)
    {
        builder.UseApplicationInsights();
        builder.ConfigureServices(InitializeServices);
    }
    ...
}
```
it implements the `IHostingStartup` interface, which is lightup the application insights experience dynamically with the help of the `HostingStartup` attribute.
```csharp
\[assembly: HostingStartup(typeof(Microsoft.AspNetCore.ApplicationInsights.HostingStartup.ApplicationInsightsHostingStartup))\]
```
With this you can inject a DLL into an ASP.NET Core application without the application knows anything about the injected DLL, nothing but we need to setup some environment variables, to let the runtime load the dependency.
```
ASPNETCORE\_HOSTINGSTARTUPASSEMBLIES={Assembly.HostingStartupClass}
```
the first environment variable will point to the assembly that has the class which implements `IHostingStartup` interface
```
    DOTNET_ADDITIONAL_DEPS={AssemblyPath}\additionaldeps
```
the second environment variable will tell the runtime to take a look in there to see if there are any dependency files that might match and merge them into our applications dependency tree.

Happy Coding !!