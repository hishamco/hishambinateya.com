---
Id: 37
Posted Date: 28/5/2017
Tags: Razor Pages,Friendly URLs 
Slug: friendly-urls-in-razor-pages
---
# Friendly URLs in Razor Pages

Websites often need to generate SEO friendly URLs. In Razor Pages the URL is tied to a physical .cshtml file. This default mapping between the URL and physical file makes it difficult for Razor Pages websites to generate SEO friendly URLs.

Last few days the ASP.NET team add a small & great feature to create friendly URLs in Razor Pages. In this short blog post I will show you how to configure the routes in the Razor Pages to create friendly URLs.

Razor Pages out-of-the-box let use access your pages from their locations, So the URLs `http://{your-domain}/Post/Index` and `http://{your-domain}/Post` will be mapped to the file `/Pages/Post/Index.cshtml`. (`Index` file name is optional).

But what if we need URLs like this `http://{your-domain}/Post/2017/5/29`?!! You can decorate your razor page using `@page "{year}/{month}/{day}"` to make this happen :)

As we have seen Razor Pages is already give us the controller to generate a simple & clean URLs, nothing but there are some case may you need much more than that.

Assume our previous file name located in `/Pages/Post/Archive.cshtml` and we wanna apply the same route `http://{your-domain}/Post/2017/5/29`, here where the previous bits was limited, but not anymore :)

Now you easily achieve that with the new extension method named `AddPageRoute` as the following:
```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.RootDirectory = "/Pages";
        options.AddPageRoute("/Post/Archive.cshtml", "Post/{year}/{month}/{day}");
    });
```
Also you can used an optional parameter if you do so
```csharp
options.AddPageRoute("/Categories/Laptops.cshtml", "Laptops/{id?}");
```
The extension is not complicated as we think, basically it provides a simple runtime configuration of routes.
```csharp
public static RazorPagesOptions AddPageRoute(this RazorPagesOptions options, string pageName, string route)
{
    if (options == null)
    {
        throw new ArgumentNullException(nameof(options));
    }

    if (string.IsNullOrEmpty(pageName))
    {
        throw new ArgumentException(nameof(pageName));
    }

    if (string.IsNullOrEmpty(route))
    {
        throw new ArgumentException(nameof(route));
    }

    options.Conventions.Add(new PageConvention(pageName, model =>
    {
        foreach (var selector in model.Selectors)
        {
            selector.AttributeRouteModel.SuppressLinkGeneration = true;
        }

        model.Selectors.Add(new SelectorModel
        {
            AttributeRouteModel = new AttributeRouteModel
            {
                Template = route,
            }
        });
    }));

    return options;
}
```
Now you can create your friendly URLs easily in the Razor Pages :)

Happy Coding !!

**Hint:** The thing I notice in the new API other than routing in ASP.NET Webforms or MVC is inverting the order of parameters, in other words the route appear before the physical page, which is may confusing little bit.