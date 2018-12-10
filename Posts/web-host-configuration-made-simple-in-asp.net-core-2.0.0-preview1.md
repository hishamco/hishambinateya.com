---
Id: 36
Posted Date: 15/5/2017
Tags: Hosting,Configuration 
Slug: web-host-configuration-made-simple-in-asp.net-core-2.0.0-preview1
---
# Web Host Configuration Made Simple in ASP.NET Core 2.0.0-preview1

In the previous versions of the ASP.NET Core the below code snippet is simplest and shortest code that you may need to write in `Program.cs` to configure the `WebHost`:
```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var host = new WebHostBuilder()
            .UseKestrel()
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseIISIntegration()
            .UseEnvironment(EnvironmentName.Development)
            .UseStartup<Startup>()
            .Build();

        host.Run();
    }
}
```
where you need to configure the web server, content root, IIS integration and much more. Also you may need to read the configuration from the `appsettings.json` and/or use logging in your application, by adding the following:
```csharp
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        var builder = new ConfigurationBuilder()
            .SetBasePath(env.ContentRootPath)
            .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
            .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
            .AddEnvironmentVariables();
        Configuration = builder.Build();
    }

    public IConfigurationRoot Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {

    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole(Configuration.GetSection("Logging"));
        loggerFactory.AddDebug();
            ...
    }
}
```
Of course this is a tedious and repetitive code that we need to write in almost or every web application before, but not anymore ..

In **ASP.NET Core 2.0.0-preview1** this code has been simplified by introducing a new API in the `WebHost` class, named `CreateDefaultBuilder`. so now the web host configuration code will look like this:
```csharp
var host = WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .Build()

host.Run();
```
Really!! Yes, so using the new API will:

1.  Uses Kestrel as the web server
2.  Sets the content root
3.  Loads the configuration from `appsettings.json` and `appsettings.[EnvironmentName].json`
4.  Loads the configuration from User Secrets (when `EnvironmentName` is `Development`)
5.  Loads the configuration from environment variables
6.  Configures the `ILoggerFactory` to log to the console and debug output
7.  Enables IIS integration
8.  Adds the developer exception page (when `EnvironmentName` is `Development`)

Happy coding !!