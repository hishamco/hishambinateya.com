---
Id: 28
Posted Date: 5/310/2017
Tags: Application Startup,Startup,Hosting Environment 
Slug: convention-based-application-startup-in-multiple-environments
---
# Convention-Based Application Startup in Multiple Environments

ASP.NET Core by convention has a special class named `Startup` which configures the request pipeline to handles all requests made to the application.

You specify the startup class name in the `Main` function using the extension method `UseStartup<TStartup>()`which is a part of  [WebHostBuilderExtensions](https://github.com/aspnet/Hosting/blob/dev/src/Microsoft.AspNetCore.Hosting/WebHostBuilderExtensions.cs).

The secret part which is few of us may heard about is you can define separate `Startup` classes for different environments, and the appropriate one will be selected at runtime.

Before that let us take a simple example to illustrate how we can deal with multiple environment within `Startup` class.
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.Run(async (context) =>
        {
            await context.Response.WriteAsync("[Development] Hello World!");
        });
    }
}
```
If you look closely to the above code which is the Startup class for default Empty ASP.NET that shipped with Visual Studio.NET 2017 you will notice that I highlight few lines of code where we need to do different things based on the hosting environment, in our case there are one place only, perhaps your application has many of those checks.

With that let's show what ASP.NET Core offers out of the box, to deal with such situations:

First thing we need to specify the startup assembly name in the `UseStartup("startupAssembly")` method, after that the hosting will load that startup assembly and search for a `Startup` or `Startup[Environment]` type with the help of [`StartupLoader`](https://github.com/aspnet/Hosting/blob/rel/1.1.0/src/Microsoft.AspNetCore.Hosting/Internal/StartupLoader.cs).
```csharp
var host = new WebHostBuilder()
                .UseKestrel()
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseApplicationInsights()
                .UseAzureAppServices()
                **.UseStartup(Assembly.GetEntryAssembly().FullName)**
                .Build();

host.Run();
```
Now we can used the convention in the `Startup` class as the following:
```csharp
public class **StartupDevelopment**
{
    public void ConfigureDevelopmentServices(IServiceCollection services)
    {
    }

    public void ConfigureDevelopment(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        app.UseDeveloperExceptionPage();

        app.Run(async (context) =>
        {
            await context.Response.WriteAsync("Hello World!");
        });
    }
}

public class StartupProduction
{
    public void ConfigureProductionServices(IServiceCollection services)
    {
    }

    public void ConfigureProduction(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        app.Run(async (context) =>
        {
            await context.Response.WriteAsync("Hello World!");
        });
    }
}
```
Cool .. we are done!! the only things that we need is to postfix the `EnvironmentName` with the `Startup` class and its methods to make this work.

Finally I'm sure that this stuff is very useful to separate the code while working in multiple hosting environments. If you prefer that old style no problem, but at least you know something new that ASP.NET Core offers for you with zero cost :)

Happy Coding !!