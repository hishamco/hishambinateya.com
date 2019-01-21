---
Id: 64
Posted Date: 21/1/2019
Tags:  Logging, Blazor, JavaScript
Slug: integrate-javascript-logging-with-blazor
---
# Integrate JavaScript Logging with Blazor

Around year ago I wrote about [Integrate JavaScript Logging with ASP.NET Core Logging APIs](http://www.hishambinateya.com/integrate-javascript-logging-with-asp.net-core-logging-apis) which is shows how integrate `jsLogger` with the current ASP.NET Core logging APIs, it was very useful to catch **JavaScript** errors at the time, since then when **Blazor** is showed up as expermintal project I realize that `jsLogger` will be better to integrate with **Blazor**.

### Introducing jsLogger APIs

Some of the logging APIs have been changed since I wrote my ealier blog post, so I will implement the new `JavaScriptLogger` with the latest and greates bits.
 
 ```csharp
public class JavaScriptLogger : ILogger
{
    private readonly string _name;

    public JavaScriptLogger(string name)
    {
        _name = name;
    }

    public IDisposable BeginScope<TState>(TState state) => NullScope.Instance;

    public bool IsEnabled(LogLevel logLevel)
    {
        if (logLevel == LogLevel.None)
        {
            return false;
        }
        else
        {
            return true;
        }
    }

    public void Log<TState>(LogLevel logLevel, EventId eventId, TState state, Exception exception, Func<TState, Exception, string> formatter)
    {
        if (!IsEnabled(logLevel))
        {
            return;
        }

        if (formatter == null)
        {
            throw new ArgumentNullException(nameof(formatter));
        }

        var message = formatter(state, exception);
        if (!string.IsNullOrEmpty(message) || exception != null)
        {
            message = $"[{_name}] {message}";
            JSRuntime.Current.InvokeAsync<object>("jsLogger.log", logLevel.ToString(), message);
        }
    }
}
```
The `JavaScriptLogger` is straignforward, except I use `JSRuntime` class for JavaScript interoperability, and invoke the `log` function that I will show it later.

`JavaScriptLoggerProvider` is the one how responsible to construct our `JavaScriptLogger` via the factory method `CreateLogger`.

```csharp
public class JavaScriptLoggerProvider : ILoggerProvider
{
    private readonly ConcurrentDictionary<string, JavaScriptLogger> _loggers;

    public JavaScriptLoggerProvider()
    {
            _loggers = new ConcurrentDictionary<string, JavaScriptLogger>();
    }

    public ILogger CreateLogger(string categoryName)
    {
        if (string.IsNullOrWhiteSpace(categoryName))
        {
            throw new ArgumentNullException(nameof(categoryName));
        }

        return _loggers.GetOrAdd(categoryName, new JavaScriptLogger(categoryName));
    }

    public void Dispose()
    {

    }
}
```

Last we need to able to register our `JavaScriptLoggerProvider` by introducing an extention method named `AddWebConsole` to extend `ILoggingBuilder`.

```csharp
public static class JavaScriptLoggerFactoryExtensions
{
    public static ILoggingBuilder AddWebConsole(this ILoggingBuilder builder)
    {
        builder.Services.TryAddEnumerable(ServiceDescriptor.Singleton<ILoggerProvider, JavaScriptLoggerProvider>());
        return builder;
    }
}
```

### Utilize jsLogger from within Blazor

As I mentioned above the `log` method is responsible for logging into the web browser, this method is declared inside `jsLogger.js` as the following

```javascript
window.jsLogger = {
    log: function (level, message) {
        switch (level) {
            case 'Trace':
                console.trace(message);
                break;
            case 'Debug':
                console.debug(message);
                break;
            case 'Information':
                console.info(message);
                break;
            case 'Warning':
                console.warn(message);
                break;
            case 'Error':
            case 'Critical':
                console.error(message);
                break;
        }
    }
}
```
Basically it utilizes the `console` object to log the entries.

Now, the remaming is to utilize the logging APIs that we defined above like the other loggers, so we need to register our logger via calling `AddWebConsole()` in `ConfigurationServices`.
```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddLogging(builder => builder.AddWebConsole());
    }

    public void Configure(IBlazorApplicationBuilder app)
    {
        app.AddComponent<App>("app");
    }
}
```
Then we will use our logger like any built-in logger, in my case I will use it in the `Counter.cshtml` page as the following:
```html
@using Microsoft.Extensions.Logging

@page "/counter"
@inject ILoggerFactory LoggerFactory

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" onclick="@IncrementCount">Click me</button>

@functions {
    int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
        var logger = LoggerFactory.CreateLogger("BLAZOR");
        logger.LogInformation("The counter incremented by 1.");
    }
}
```
Finally when you run the application and clicking the **Click me** button you will see the logs appear on the browser console.

![JavaScript Logger](https://raw.githubusercontent.com/hishamco/hishambinateya.com/master/Posts/images/97ce3ef2-1e4a-4180-aaa8-e08ad38a8a30.png)

You can download the source code for this blog post from my [BlazorJavaScriptLogger](https://github.com/hishamco/BlazorJavaScriptLogger) repository on GitHub.

Happy Coding!!