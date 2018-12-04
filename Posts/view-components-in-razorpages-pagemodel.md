---
Id: 53
Posted Date: 16/1/2018
Tags: RazorPages,ViewComponents
Slug: view-components-in-razorpages-pagemodel
---
# Using View Components in RazorPages PageModel

It's a month or so from the last blog that I wrote for unexpected reasons. I'm back to start with RazorPages again, this time with using **View Components** in RazorPages. This is not an introduction to **View Components**, but if you need so I prefer to read the [ASP.NET documentations](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-components) for more details.

Seems RazorPages until the date that I'm writing this blog post is missing the `ViewComponent` helper factory methods in the RazorPages PageModel, but don't be hold you can bring those factory methods if you come across this scenario by copying them from the `Controller` and use them in your custom PageModel class.

Instead of create a new custom PageModel class, i will write them as an extension methods as the following:
```csharp
public static class PageModelExtensions
{
    [NonAction]
    public static ViewComponentResult ViewComponent(this PageModel pageModel, string componentName)
    {
        return ViewComponent(pageModel, componentName, arguments: null);
    }

    [NonAction]
    public static ViewComponentResult ViewComponent(this PageModel pageModel, Type componentType)
    {
        return ViewComponent(pageModel, componentType, arguments: null);
    }

    [NonAction]
    public static ViewComponentResult ViewComponent(this PageModel pageModel, string componentName, object arguments)
    {
        return new ViewComponentResult
        {
            ViewComponentName = componentName,
            Arguments = arguments,
            ViewData = pageModel.ViewData,
            TempData = pageModel.TempData
        };
    }

    [NonAction]
    public static ViewComponentResult ViewComponent(this PageModel pageModel, Type componentType, object arguments)
    {
        return new ViewComponentResult
        {
            ViewComponentType = componentType,
            Arguments = arguments,
            ViewData = pageModel.ViewData,
            TempData = pageModel.TempData
        };
    }
}
```
Now you 're free to use the **View Components** in the RazorPages by using:`this.ViewComponent()` and choosing the overload the suited for you.

Happy Coding!!