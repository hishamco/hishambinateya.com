---
Id: 27
Posted Date: 19/2/2017
Tags: jsLogger,Logging
Slug: integrate-javascript-logging-with-asp.net-core-logging-apis
---
# Integrate JavaScript Logging with ASP.NET Core Logging APIs

### JavaScript is Hard!

ASP.NET Core provides us a rich [Logging](https://github.com/aspnet/logging) APIs which have a set of logger providers including: `ConsoleLoggerPtovider`, `AzureAppServicesDiagnosticsLoggerProvider`, `EventLogLoggerProvider` and much more.

This let C# developers happy than before, because you need to implement them loggers yourself in the past, however a lot of us writing JavaScript code in almost web applications which is little hard, but find the client-side exceptions & errors are even harder.

There are many client-side loggers that you can name it, which are fit our needs, but today I wanna talk about the client-side logging from different angle. Me and you as developers suffer a lot from unexpected javascript error that happen occasionally, sometimes the reason is silly such missing a curly braces or broke a link .. etc.

With that I was thinking last few days that it would be nice to integrate the client-side & server-side logs, so all the logs will be logged from the ASP.NET Core logger providers that we like and love instead of managing two different logger providers for both client-side & server-side.

### Logging JavaScript Events

Basically the idea is very simple, I need to inject the client-side logging APIs script into our view, after that I need the `JavaScriptLoggingMiddleware` to listening to the upcoming script logs and forward them to ASP.NET Core logger providers.

With that we're ready to show some code, but before that I need to mention that I didn't introduce a new logging APIs, but I could, so the javascript `console` object is enough for logging. In both cases we need to intercept the `console` logs as the following:
```javascript
(function () {
    var trace = console.trace;
    var debug = console.debug;
    var info = console.info;
    var warn = console.warn;
    var error = console.error;

    console.trace = function (message) {
        log(logLevel.Trace, message);
        trace.call(this, arguments);
    };

    console.debug = function (message) {
        log(logLevel.Debug, message);
        debug.call(this, arguments);
    };

    console.info = function (message) {
        log(logLevel.Information, message);
        info.call(this, arguments);
    };

    console.warn = function (message) {
        log(logLevel.Warning, message);
        warn.call(this, arguments);
    };

    console.error = function (message) {
        log(logLevel.Error, message);
        error.call(this, arguments);
    };
})();
```
The `log` is a method that I created to post the actual logs to the server, which will handled by the `JavaScriptLoggingMiddleware` that shown below.

At this point I was wondering whether to store jsLogger script into the disk and render it using a tag helper or not!! after awhile I inspired by Application Insights and decided to store it into a resx file and retrieve them later using `JavaScriptLoggingSnippet`.
```csharp
public class JavaScriptLoggingMiddleware
{
    private readonly ILogger _logger;
    private readonly RequestDelegate _next;

    public JavaScriptLoggingMiddleware(ILoggerFactory loggerFactory, RequestDelegate next)
    {
        _logger = loggerFactory.CreateLogger<JavaScriptLoggingMiddleware>();
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Path == "/log" && context.Request.Method == "POST" && context.Request.HasFormContentType)
        {
            var form = await context.Request.ReadFormAsync();
            var level = Convert.ToInt32(form["level"].First());
            var message = form["message"].First();

            switch ((LogLevel)level)
            {
                case LogLevel.Trace:
                    _logger.LogTrace(message);
                    break;
                case LogLevel.Debug:
                    _logger.LogDebug(message);
                    break;
                case LogLevel.Information:
                    _logger.LogInformation(message);
                    break;
                case LogLevel.Warning:
                    _logger.LogWarning(message);
                    break;
                case LogLevel.Error:
                    _logger.LogError(message);
                    break;
                default:
                    return;
            }
        }
        else
        {
            await _next(context);
        }
    }
}
```
If you look closely to the code above, you will notice that the middleware listening for specific url, when it hit I need to map the client-side log levels with the server-side once, after that I log them normally.

### I can finally log JavaScript Exceptions!

Last but not least, sometimes we come up to a situation that we need to catch the global javascript exception that may occur for whatever reason could be. `window.onerror` is a good place to catch such exception.

I added a `JavaScriptLoggingOptions` which is shown below to make things configurable in the way that you want.
```csharp
public class JavaScriptLoggingOptions
{
    public bool HandleGlobalExceptions { get; set; }
}
```
By adding this simple property, the `JavaScriptLoggingSnippet` is now able to choose the proper script to render in the view.
```csharp
public class JavaScriptLoggingSnippet
{
    private readonly JavaScriptLoggingOptions _loggingOptions;

    private static readonly HtmlString JavaScriptLoggingScript = new HtmlString(Resources.Script);

    private static readonly HtmlString JavaScriptLoggingGlobalExceptionHandlingScript = new HtmlString(Resources.GlobalExceptionHandlingScript);

    public JavaScriptLoggingSnippet(IOptions<JavaScriptLoggingOptions> loggingOptions)
    {
        _loggingOptions = loggingOptions.Value;
    }

    public HtmlString Script =>
        _loggingOptions.HandleGlobalExceptions ? FullScript : JavaScriptLoggingScript;

    private static HtmlString FullScript => JavaScriptLoggingScript.Concat(HtmlString.NewLine,
        JavaScriptLoggingGlobalExceptionHandlingScript);
}
```
### What it takes to make everything happen?

We need few steps to make this happen:

First we need to configure the `JavaScriptLogging` service in the `ConfigureServices` method
```csharp
services.AddJavaScriptLogging(options =>
{
    options.HandleGlobalExceptions = true;
});
```
After that adding the `JavaScriptLoggingMiddleware` to the `Configure` method
```csharp
app.UseJavaScriptLogging();
```
Finally add the following line into your view or layout page to render the jsLogger script.
```html
@Html.Raw(JavaScriptLoggingSnippet.Script)
```
You can download the source code for this post from my [jsLogger](https://github.com/hishamco/jsLogger) repository on GitHub.

Happy Coding !!