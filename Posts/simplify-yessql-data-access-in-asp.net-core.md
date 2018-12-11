---
Id: 31
Posted Date: 27/3/2017
Tags: YesSQL,APIs 
Slug: simplify-yessql-data-access-in-asp.net-core
---
# Prettify YesSQL Data Access APIs in ASP.NET Core

Two weeks ago I blogged about [Introduction to YesSQL](http://www.hishambinateya.com/introduction-to-yessql), which shows a brief introduction to YesSQL, difference between YesSQL & NoSQL and how we can use it.  

Today I will write about YesSQL in action in ASP.NET Core, and how we can go further to introduce a new APIs to simplify the data access process.

My aim in this post is to eliminate the headache by reducing the code required for creating the `DbConnectionFactory`, `StorageFactory` and initializing the configuration which required for the `Store` by introducing a simple and readable APIs that make developers life easier.

The naming convention for the new APIs are originally inspired by the EntityFramework Core, the data access layer that we know and love. EF provide as with as set of APIs such as `UseSqlServer()`, `UseSqlite()` .. etc, basically these APIs let us focus on the real code that we need to write, not in the database configuration setup that should be written on our behalf, the same flavor that I wanna bring while we're working with YesSQL :) so let us dig into the code.

First of all let us add the abstraction for the database provider options, which is simply have the provider name and its configuration as the following:
```csharp
public interface IDbProviderOptions
{
    string ProviderName { get; set; }

    Configuration Configuration { get; set; }
}

public class DbProviderOptions : IDbProviderOptions
{
    public string ProviderName { get; set; }

    public Configuration Configuration { get; set; }
}
```
Then we need to add `AddDbProvider` extension method, which is responsible to register a single instance of `IStore` service, using the options that will be provided in `Action<IDbProviderOptions>`
```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddDbProvider(
        this IServiceCollection services,
        Action<IDbProviderOptions> setupAction)
    {
        if (services == null)
        {
            throw new ArgumentNullException(nameof(services));
        }

        if (setupAction == null)
        {
            throw new ArgumentNullException(nameof(setupAction));
        }

        var options = new DbProviderOptions();
        setupAction.Invoke(options);
        services.AddSingleton<IStore>(new Store(options.Configuration));

        return services;
    }
}
```
After that we need to add the required helper functions which will be used in our applications
```csharp
public static class InMemoryDbProviderOptionsExtensions
{
    public static void UseInMemory(
        this IDbProviderOptions options)
    {
        if (options == null)
        {
            throw new ArgumentNullException(nameof(options));
        }

        var connectionString = "Data Source=:memory:";

        var configuration = new Configuration
        {
            ConnectionFactory = new DbConnectionFactory<SqliteConnection>(connectionString, true),
            DocumentStorageFactory = new InMemoryDocumentStorageFactory(),
            IsolationLevel = IsolationLevel.Serializable
        };

        options.ProviderName = "InMemory";
        options.Configuration = configuration;
    }
}
```
The above code snippet is simplify the InMemory database provider via `UseInMemory()` extension method,`UseSqLite()` for SqLite data provider and `UseSqlServer()` for SQL Server data provider.
```csharp
public static class SqLiteDbOptionsExtensions
{
    public static void UseSqLite(
        this IDbProviderOptions options, string connectionString)
    {
        if (options == null)
        {
            throw new ArgumentNullException(nameof(options));
        }

        var configuration = new Configuration
        {
            ConnectionFactory = new DbConnectionFactory<SqliteConnection>(connectionString, true),
            DocumentStorageFactory = new SqlDocumentStorageFactory(),
            IsolationLevel = IsolationLevel.Serializable
        };

        options.ProviderName = "SQLite";
        options.Configuration = configuration;
    }
}

public static class SqlServerDbProviderOptionsExtensions
{
    public static void UseSqlServer(
        this DbProviderOptions options, string connectionString)
    {
        if (options == null)
        {
            throw new ArgumentNullException(nameof(options));
        }

        var configuration = new Configuration
        {
            ConnectionFactory = new DbConnectionFactory<SqlConnection>(connectionString, true),
            DocumentStorageFactory = new SqlDocumentStorageFactory(),
            IsolationLevel = IsolationLevel.ReadUncommitted
        };

        options.ProviderName = "SQL Server";
        options.Configuration = configuration;
    }
}
```
Similarly for the other database providers such as MySQL, Postgres .. etc.

Now it's the time to use the newly created APIs in the `ConfigureServices()` method as usual:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    //services.AddDbProvider(options => options.UseInMemory());

    //services.AddDbProvider(options =>
    //    options.UseSqLite("Filename=YesSql.db"));

    services.AddDbProvider(options =>
        options.UseSqlServer("Server=.;Database=YesSql;Integrated Security=True"));
    ...
}

You can download the source code for this post from my [YesSQL](https://github.com/hishamco/yessql/tree/master/samples/YesSql.Samples.Web) repository on GitHub.

Happy Coding !!