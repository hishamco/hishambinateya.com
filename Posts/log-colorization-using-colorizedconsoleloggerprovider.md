---
Id: 25
Posted Date: 30/1/2017
Tags: Logging 
Slug: log-colorization-using-colorizedconsoleloggerprovider
---
# Log Colorization using ColorizedConsoleLoggerProvider

ASP.NET Core provides us a rich [Logging](https://github.com/aspnet/logging) APIs which have a set of logger providers including: `ConsoleLoggerPtovider`, `AzureAppServicesDiagnosticsLoggerProvider`, `EventLogLoggerProvider` and much more.

AFAIK `ConsoleLoggerPtovider` is one of the commonly used logger provider, one of its limitation that doesn't give you an elegant way to colorize your log messages. The log colorization is one of the useful features in the console, because there are a lot of scenarios that required you to colorize some parts of your log message such as Timestamps, SQL queries .. etc.

Occasionally there is someone already opened an [issue](https://github.com/aspnet/Logging/issues/530) in [Logging](https://github.com/aspnet/logging) repo describes the needs for such feature, below is the image that he attached in his issue which is nothing but some of the EntityFramework logs. Believe it or not what you see is a white gibberish, of course you need a lot of the time and effort to find the whole SQL queries, in case if you need to analyze them or at least what to know what's EntityFramework produces or any other scenario.

[![](https://cloud.githubusercontent.com/assets/4528464/20841864/4bcad886-b8ad-11e6-936e-397c276c64ae.png)](https://cloud.githubusercontent.com/assets/4528464/20841864/4bcad886-b8ad-11e6-936e-397c276c64ae.png)

so It would be nice if log colorization is a built-in feature in the `ConsoleLoggerProvider`, believe it or not if it doesn't help you today absolutely will helps you one day.

Now let us dig into the code, first of all we need to create two interfaces `IColorizer` & `IClassification` to provider the necessary information for `RegexClassification` class.
```csharp
public interface IColorizer
{
    ConsoleColor Color { get; set; }
}

public interface IClassification
{
    string ClassificationName { get; set; }
}
```
Then we need to create a `RegexClassification` class which is hold the essential information that are used by the `ColorizedConsoleLogger` such as pattern and color.
```csharp
public class RegexClassification : IClassification, IColorizer
{
    public string ClassificationName { get; set; }

    public string RegexPattern { get; set; }

    public bool IgnoreCase { get; set; }

    public ConsoleColor Color { get; set; } = ConsoleColor.Gray;
}
```
Last but not least I don't wanna bother you with `ColorizedConsoleLoggerProvider` & `ColorizedConsoleLogger` source code because they are cloned versions of `ConsoleLoggerProvider` & `ConsoleLogger` with little modification. The important thing here is the actual colorization code which is belong to the `WriteMessage` method that shown [here](https://github.com/hishamco/ColorizedConsoleLoggerProvider/blob/master/ColorizedConsole/ColorizedConsoleLogger.cs#L177).
```csharp
if (Classifications != null)
{
    foreach (var classification in Classifications)
    {
        --System.Console.CursorTop;
        foreach (Match match in Regex.Matches(message, classification.RegexPattern))
        {
            System.Console.CursorLeft = match.Index + _messagePadding.Length;
            Console.Write(match.Value, null, classification.Color);
        }
        ++System.Console.CursorTop;
        System.Console.CursorLeft = 0;
    }
}
```
Finally we need to create an extension method `AddColorizedConsole`, to use it in our applications.
```csharp
public static class ColorizedConsoleLoggerExtensions
{
    public static ILoggerFactory AddColorizedConsole(this ILoggerFactory factory, RegexClassification[] classifications)
    {
        factory.AddProvider(new ColorizedConsoleLoggerProvider((n, l) => l >= LogLevel.Information, false, classifications));
        return factory;
    }
}
```
### How to enable colorization and how to customize with different colors

To enable the log colorization we can simple use `AddColorizedConsole` extension method which is created before.
```csharp
public class Program
{
    private static readonly ILogger _logger;

    static Program()
    {
        var factory = new LoggerFactory();
        _logger = factory.CreateLogger<Program>();
        var patterns = new[]
        {
                new RegexClassification
                {
                    ClassificationName = "URL",
                    IgnoreCase = false,
                    RegexPattern = @"(http|https|ftp|)\://|[a-zA-Z0-9\-\.]+\.[a-zA-Z](:[a-zA-Z0-9]*)?/?([a-zA-Z0-9\-\._\?\,\'/\\\+&amp;amp;%\$#\=~])*[^\.\,\)\(\s]",
                    Color = ConsoleColor.Magenta
                },
                new RegexClassification
                {
                    ClassificationName = "SQL",
                    IgnoreCase = true,
                    RegexPattern = @"\""Select (Distinct )?(Top \d+ )?(\*|[a-zA-Z]+(,[a-zA-Z]+)?) From [a-zA-Z]+\""",
                    Color = ConsoleColor.Yellow
                },
                new RegexClassification
                {
                    ClassificationName = "DateTime",
                    RegexPattern = @"\d{1,2}\/\d{2}\/\d{4} \d{1,2}:\d{2}:\d{2} [paPA][Mm]",
                    Color = ConsoleColor.Green
                }
            };
        factory.AddColorizedConsole(patterns);
    }

    static void Main(string[] args)
    {
        _logger.LogInformation($"The application starting @ {DateTime.Now}.");
        _logger.LogInformation($"-= Colorized Console Logger Provider =-");
        _logger.LogInformation("GitHub: https://github.com/hishamco/ColorizedConsoleLoggerProvider");
        _logger.LogWarning("The query \"Select * From Employees\" takes long execution time.");
        _logger.LogError("The query \"Select Top 3 Name From Employees\" causes an error.");
        _logger.LogInformation("Executing the query \"Select Distinct City,Address From Employees\".");
        _logger.LogInformation($"The application stopping @ {DateTime.Now}.");
    }
}
```
In the above example we defined and initialized an array of `RegexClassification` that contains all the patterns that we need the `ColorizedConsoleLogger` to handle them during the logging process, also we define the customize colors for each pattern, after that we called the `AddColorizedConsole` to handle everything in your behalf. The other stuff are the simple code required for [Logging](https://github.com/aspnet/logging) APIs, so nothing fancy just one function and it will work like a charm!!

After running the application you should notice the log colorization as the following:

![](http://www.hishambinateya.com/images/posts/b6ec5b8f-0ac2-4965-8a50-e0180fd22a2c.png)

You can download the source code for this post from my [ColorizedConsoleLoggerProvider](https://github.com/hishamco/ColorizedConsoleLoggerProvider) repository on GitHub.

Happy Coding !!