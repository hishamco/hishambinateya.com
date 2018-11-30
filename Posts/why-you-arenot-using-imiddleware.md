---
Posted Date: 9/10/2018
Tags:  Middlewares 
Slug: why-you-arenot-using-imiddleware
---
# Why you aren't using IMiddleware?

 Few hours ago I saw a nice blog post [Customizing ASP.​NET Core Part 06: MiddleWares](https://asp.net-hacker.rocks/2018/10/08/customizing-aspnetcore-06-middlewares.html) by [Jürgen Gutsch](https://twitter.com/sharpcms) which leads me to raise a question that I quote from our twitter dialog:

> I'm not sure why I never seen anyone using IMiddleware instead of writing InvokeAsync manually?!!

With that I start writing this short blog post as an extension for Jürgen blog post.

### IMiddlewrare the Hidden Gem

The `IMiddleware` interface is an extensibility point for middleware activation. It defines middleware for application's request pipeline like the _Conventional Middleware_ where `InvokeAsync()` handles the requests and returns a `Task` that represents the execution of the middleware.

### IMiddleware Benifits

There 're some benifits for using `IMiddleware`:

*   The middleware is strongly typing
*   Activation per request, which allows you to inject a scoped services into the middleware constructor

### IMiddleware Usage

For those who are creating or using middlewares in the ASP.NET Core, the middlware is declared as a class with some conventions, such as the class name end with _Middleware_ like `SimpleMiddleware`, with a special method called `InvokeAsync()` which handles the request when the middleware is executed in the ASP.NET Core pipeline.
```csharp
public class SimpleMiddleware
{
    private readonly RequestDelegate _next;

    public SimpleMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Do something ...

        await _next(context);
    }
}
```
That's how all almost if not all of us knew how we can wrote a middleware, but with `IMiddleware` interface we can write the above code snippet with the one below, furthermore you will have all the advantages that I mentioned above such as injecting a scoped services which is something very cool.
```csharp
public class SimpleMiddleware : IMiddleware
{
    private readonly ScopedService _scopedService;

    public SimpleMiddleware(ScopedService scopedService)
    {
        _scopedService = scopedService;
    }

    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
       // Do something ...

        await next(context);
    }
}
```
After that you can add the `SimpleMiddleware` to the built-in container as the following:
```csharp
services.AddTransient<SimpleMiddleware>(); 
```
Last but not least I wanna mention a very crucial point, _**such middleware not allows you to pass an option of type `IOption<T>` as argument**._

Finally `IMiddleware` interface was hidden from many of us since 4 months ago, so it's the time to try it now.

For more information you can checkout the [ASP.NET Core Docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/extensibility?view=aspnetcore-2.1).

Happy Coding ...