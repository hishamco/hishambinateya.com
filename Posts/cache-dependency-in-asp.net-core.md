---
Id: 32
Posted Date: 18/4/2017
Tags: Caching 
Slug: cache-dependency-in-asp.net-core
---
# Cache Dependency in ASP.NET Core

Caching is the process of storing frequently used data, this data is costly to generate for reuse. Usually this data are stored in memory because retrieving data from memory is very fast and more efficient than retrieving the data from other locations such as database.

ASP.NET Core provide us both in-memory and distributed caching. [Caching](https://github.com/aspnet/caching) repository has three different caching storage, In-Memory, Redis and SQL Server.

### Cache Dependencies

Dependencies allow us to invalidate a particular item within the `Cache` based on file, key changes or at a fixed time.

In the previous version of ASP.NET there are three types of dependencies supported:

*   File based dependency
*   Key based dependency
*   Time based dependency

In ASP.NET Core we can invalidate a cache item based on time using the `MemoryCacheEntryOptions` class, nothing but the other cache dependencies are not there!! but that doesn't mean they are impossible to implement. So in this blog post I will show you how we can implement a file based dependency.

First of all we need to create the `FileCacheDependency` class:
```csharp
public class FileCacheDependency
{
    public FileCacheDependency(string filename)
    {
        FileName = filename;
    }

    public string FileName { get; }
}
```
This is simple enough to store the file name that when need to monitor to invalidate the cache items, based on its change, second we need to add an extension method to add file based dependency.
```csharp
public static class MemoryCacheExtensions
{
    public static void Set<TItem>(this IMemoryCache cache, string key, TItem value, FileCacheDependency dependency)
    {
        var fileInfo = new FileInfo(dependency.FileName);
        var fileProvider = new PhysicalFileProvider(fileInfo.DirectoryName);
        cache.Set(key, value, new MemoryCacheEntryOptions()
                            .AddExpirationToken(fileProvider.Watch(fileInfo.Name)));

    }
}
```
Simply I used `PhysicalFileProvider` to monitor the give file, thanks `Watch()` method :)

Then we can use it as the following:
```csharp
var cache = new MemoryCache(new MemoryCacheOptions());
var dependency = new FileCacheDependency("FilePath");

cache.Set("cacheKey", "cachValue", dependency);
```
Similarly we can create `TimeCacheDependency` class:
```csharp
public enum CacheItemPolicy
{
    AbsoluteExpiration,
    SlidingExpiration
}

public class TimeCacheDependency
{
    public TimeCacheDependency(TimeSpan time, CacheItemPolicy policy = CacheItemPolicy.AbsoluteExpiration)
    {
        Time = time;
    }

    public TimeSpan Time { get; }

    public CacheItemPolicy Policy { get; }
}
```
Finally add an extension method to `MemoryCacheExtensions` class
```csharp
public static void Set<TItem>(this IMemoryCache cache, string key, TItem value, TimeCacheDependency dependency)
{
    var options = new MemoryCacheEntryOptions();

    if (dependency.Policy == CacheItemPolicy.AbsoluteExpiration)
    {
        options.SetAbsoluteExpiration(dependency.Time);
    }
    else
    {
        options.SetSlidingExpiration(dependency.Time);
    }
    cache.Set(key, value, options);
}
```
That's it!!  also you can implement the key based dependency if it's required.

Happy Coding !!