---
Id: 66
Posted Date: 20/9/2019
Tags: Localization
Slug: what-is-new-in-localization-in-asp.net-core-3.0
---
# What is new in Localization in ASP.NET Core 3.0

It has been a while since I wrote my latest blog post for ASP.NET Core community, because my left ankle was broken and did a surgey since then, the thing that let me stay on the bed for around half a year.

Last week I just start writing the first blog post about [My Journey with Orchard Core](http://hishambinateya.com/my-journey-with-orchard-core).

Today I will continue again in ASP.NET Core blog posts, and I will start with my favorate Localization :), this will be a short and quick one that show off the new stuff in localization that shipped with ASP.NET Core 3.0.

This release has shipped few features, enhancements and one bug fix, which all of them are done by me, so if you like them I will be greate, otherwise don't blame anyone except me :)

**1- Add Content-Language header in localization middleware**

When you're working with localization you will find yourself usually doing this

```csharp
app.UseRequestLocalization("en", "ar", "fr"); 
```

This set the culture properly on the `HttpContext`, in a way that requesting in a language not supported fallbacks to the default culture and culture variants fallback to languages, etc.

The issue that you will find it that you need to manage the response header `Content-Language` yourself.

Now this has been added optionally to the ` RequestLocalizationMiddleware` by toggling the property `ApplyCurrentCultureToResponseHeaders` which has been introduced to `RequestLocalizationOptions`.

```csharp
app.UseRequestLocalization(new RequestLocalizationOptions
{
    ApplyCurrentCultureToResponseHeaders = true
});
```

**2- Adding AddInitialRequestCultureProvider extension method**

When you are working with localization with a custom `RequestCultureProvider` you will find yourself doing this all the time:

```csharp
options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
{

}));
```

but not anymore, now you can simply use the `AddInitialRequestCultureProvider` extension to make your life easier

```csharp
options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
{

}));
```

It's a tiny feature but it's very useful.

**3- Change unsupported culture log level**

The issue was the localization middlware will log tons if not hunderds of logs if the requested culture is unsupported, image that you will recieve 1 warning log per request which is noisy and trouble over the time.

For that we simply decide to change the `LogLevel.Warning` to `LogLevel.Debug` to reduce the amount of logs at least.

**4- Fix localization issue after changing rootnamespace**

After digging into the issue [AspNetCore#10639](https://github.com/aspnet/AspNetCore/issues/10639), seems there was a bug in `GetResourcePrefix()` that causes the localization not working properly after changing rootnamespace, especially if you're working with class libraries.

Now I'm glad to say that the bug has been fixed and nothing block you to change the rootnamespace if you are using class libraries.

If you're interested you can checkout the following pull requests:

- https://github.com/aspnet/AspNetCore/pull/13479
- https://github.com/aspnet/AspNetCore/pull/12595
- https://github.com/aspnet/Localization/pull/458
- https://github.com/aspnet/AspNetCore/pull/12153
- https://github.com/aspnet/AspNetCore/pull/12556
- https://github.com/aspnet/Extensions/pull/2081

Last but not least I hope to write some docs for update 3.0 in Globalization and localization article, which is discussed here https://github.com/aspnet/AspNetCore.Docs/issues/14225.

Finally I enchourge everyone who didn't heard or work with these new bits to have a try.

Happy Coding !!