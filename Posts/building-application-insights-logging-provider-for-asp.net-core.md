---
Id: 20
Posted Date: 27/11/2016
Tags: Logging,Application Insights 
Slug: building-application-insights-logging-provider-for-asp.net-core
---
# Building Application Insights Logging Provider for ASP.NET Core

Application Insights is an extensible Application Performance Management (APM) service for web developers. It helps you to understand how your application is performing and how it's being used. Application Insights includes a powerful Diagnostic Search tool that enables you to explore and drill in to telemetry sent by the Application Insights SDK from your application. Many events such as user page views are automatically sent by the SDK.

### Set up Application Insights for ASP.NET Core Application

To enable Application Insights to your ASP.NET Core application, simply right click your project and choose Add Application Insights Telemetry as the following:

![](http://www.hishambinateya.com/Images/Posts/6f630696-63ea-42c7-b61c-d9364ba52b83.png)

after that you can install the Application Insights SDK to use it locally. For more information about the set up process I encourge you to checkout this [link](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-asp-net).

### Explore your logs

If you are using NLog, log4Net or `System.Diagnostics.Trace` for diagnostic tracing in your ASP.NET Core application, you can have send your logs to [Azure Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview), where you can explore and search them. Your logs will be merged with the other telemetry coming from your application, so that you can identify the traces associated with servicing each user request, and correlate them with other events and exception reports.

After running your application in debug mode you can see the Application Insights integration with Diagnostic Tools window

![](http://www.hishambinateya.com/Images/Posts/96da03c1-f888-4f22-a19b-544fcd05e561.png)

also you can check the Application Insights dashboard which looking pretty good with the statistics information that let you drill in for further details about what is going on.

![](http://www.hishambinateya.com/Images/Posts/2d44e556-a1c9-43ee-bd9a-f333c7eafec7.png)

### Creating Application Insights Logger

Microsoft already has a [repository](https://github.com/Microsoft/ApplicationInsights-aspnetcore) for Application Insights for ASP.NET Core, it would be nice if we have such tool integrated with ASP.NET [Logging](http://github.com/aspnet/logging) APIs, that is what I wanna explore in this section.

First of all let create a simple class that represents the various settings for the Application Insights
```csharp
public class ApplicationInsightsSettings
{
    public bool? DeveloperMode { get; set; }

    public string InstrumentationKey { get; set; }
}
```
Second we need to write the `ApplicationInsightsLogger` that implements `ILogger` interface.
```csharp
public class ApplicationInsightsLogger : ILogger
{
    private readonly string _name;
    private readonly Func _filter;
    private readonly ApplicationInsightsSettings _settings;
    private readonly TelemetryClient _telemetryClient;

    public ApplicationInsightsLogger(string name)
        : this(name, null, new ApplicationInsightsSettings())
    {
    }

    public ApplicationInsightsLogger(string name, Func filter, ApplicationInsightsSettings settings)
    {
        _name = string.IsNullOrEmpty(name) ? nameof(ApplicationInsightsLogger) : name;
        _filter = filter;
        _settings = settings;
        _telemetryClient = new TelemetryClient();

        if (_settings.DeveloperMode.HasValue)
        {
            TelemetryConfiguration.Active.TelemetryChannel.DeveloperMode = _settings.DeveloperMode;
        }

        if(!_settings.DeveloperMode.Value)
        {
            if (string.IsNullOrWhiteSpace(_settings.InstrumentationKey))
            {
                throw new ArgumentNullException(nameof(_settings.InstrumentationKey));
            }

            TelemetryConfiguration.Active.InstrumentationKey = _settings.InstrumentationKey;
            _telemetryClient.InstrumentationKey = _settings.InstrumentationKey;
        }
    }

    public IDisposable BeginScope(TState state)
    {
        return NoopDisposable.Instance;
    }

    public bool IsEnabled(LogLevel logLevel)
    {
        return (_filter == null || _filter(_name, logLevel));
    }

    public void Log(LogLevel logLevel, EventId eventId, TState state, Exception exception, Func formatter)
    {
        if (!IsEnabled(logLevel))
        {
            return;
        }

        if (exception != null)
        {
            _telemetryClient.TrackException(new ExceptionTelemetry(exception));
            return;
        }

        var message = string.Empty;
        if (formatter != null)
        {
            message = formatter(state, exception);
        }
        else
        {
            if (state != null)
            {
                message += state;
            }
        }
        if (!string.IsNullOrEmpty(message))
        {
            _telemetryClient.TrackTrace(message, GetSeverityLevel(logLevel));
        }
    }

    private static SeverityLevel GetSeverityLevel(LogLevel logLevel)
    {
        switch (logLevel)
        {
            case LogLevel.Critical: return SeverityLevel.Critical;
            case LogLevel.Error: return SeverityLevel.Error;
            case LogLevel.Warning: return SeverityLevel.Warning;
            case LogLevel.Information: return SeverityLevel.Information;
            case LogLevel.Trace:
            default: return SeverityLevel.Verbose;
        }
    }

    private class NoopDisposable : IDisposable
    {
        public static NoopDisposable Instance = new NoopDisposable();

        public void Dispose()
        {
        }
    }
}
```
Last but not least we need to create `ApplicationInsightsLoggerProvider` which implements `ILoggerProvider`.
```csharp
public class ApplicationInsightsLoggerProvider : ILoggerProvider
{
    private readonly Func _filter;
    private readonly ApplicationInsightsSettings _settings;

    public ApplicationInsightsLoggerProvider(Func filter, ApplicationInsightsSettings settings)
    {
        _filter = filter;
        _settings = settings;
    }

    public ILogger CreateLogger(string name)
    {
        return new ApplicationInsightsLogger(name, _filter, _settings);
    }

    public void Dispose()
    {

    }
}
```
Finally we can end up with some of the extension methods that the application may call to use our logging provider.
```csharp
public static class ApplicationInsightsLoggerFactoryExtensions
{
    public static ILoggerFactory AddApplicationInsights(
        this ILoggerFactory factory,
        Func filter,
        ApplicationInsightsSettings settings)
    {
        factory.AddProvider(new ApplicationInsightsLoggerProvider(filter, settings));
        return factory;
    }

    public static ILoggerFactory AddApplicationInsights(
        this ILoggerFactory factory,
        ApplicationInsightsSettings settings)
    {
        factory.AddProvider(new ApplicationInsightsLoggerProvider(null, settings));

        return factory;
    }
}
```
That is it!!  that was the actual code needed to create a logging provider that logs into the Application Insights.  

Then you use simply use it from your application with single line of code.

loggerFactory.AddApplicationInsights(new ApplicationInsightsSettings { DeveloperMode = true });

This can be improved by accessing the settings using the [Configuration](https://github.com/aspnet/configuration) APIs, which is the thing that I did in my repository.

You can download the source code for this post from my [ApplicationInsightsLogging](https://github.com/hishamco/ApplicationInsightsLogging) repository.

Happy Coding !!