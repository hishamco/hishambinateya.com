---
Id: 47
Posted Date: 2/10/2017
Tags: RazorPages,ViewBag 
Slug: bring-viewbag-back-to-razor-pages
---
# ViewBag in Razor Pages

As we knew `ViewBag` is a dynamic view data dictionary, which is widely used by developers since the early days of ASP.NET MVC, and it's still supported in ASP.NET Core.

But for those who are new to the Razor Pages, they may will notice that `ViewBag` is no longer supported!! especially in the `PageModel`, because you can still used in view as the following:
```html
@page
@model IndexModel
@{
    ViewBag.Title = "Home page";
}
```
<h1>@ViewBag.Title</h1>

So if you like to bring the `ViewBag` back to Razor Page you could follow the rest of this short blog post.

If we look to the [`Controller`](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNetCore.Mvc.ViewFeatures/Controller.cs) class in MVC source code, you will notice that the `ViewBag` property is already declared there, similarly we will create a new class called `PageModelBase` that extends `PageModel` class as the following:
```csharp
public class PageModelBase : PageModel
{
    private DynamicViewData _viewBag;

    public dynamic ViewBag
    {
        get
        {
            if (_viewBag == null)
            {
                _viewBag = new DynamicViewData(() => ViewData);
            }
            return _viewBag;
        }
    }
}
```
now you are free to use the `ViewBag` property again as we normally use it in the past.

IMHO the ASP.NET team pushing the developers to use a `PageModel` properties instead, but still `ViewBag` is a great piece of technology that allow us to pass a state to the layout pages.

Happy Coding !!