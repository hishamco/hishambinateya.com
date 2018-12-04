---
Id: 56
Posted Date: 9/7/2018
Tags:  RazorPages,Partials,AJAX
Slug: introducing-partial-action-result-helper-in-razor-pages
---
# Adding Partial Helper to Razor Pages

As almost of the ASP.NET Core know that Razor Pages are cool and sexy!! even though there 're some missing pieces that needs to be added, one of them is partial action result helper which this blog post is talking about.

I have seen some of blog posts that talking about Razor Pages & AJAX, there 's some difficulty because ASP.NET Core doesn't have a built-in support for AJAX such previous versions of MVC, but you can still use jQuery to make an AJAX requests and integrate it with the Razor Pages or you can use **Unobtrusive.Ajax** to make things even cleaner & simpler.

Although we need a mechanism to use partials in both **GET** or **POST** verb from within **RazorPage / PageModel**. With that you can use **PartialViewResult**, but there are a lot boiler plate that we need to write, let us have a look ..  
```csharp
public IActionResult OnPostSearchBooks()
{
    var searchResults = new List<string>
    {
        $"Book 1 (ISBN: {Guid.NewGuid()})",
        $"Book 2 (ISBN: {Guid.NewGuid()})",
    };

    return new PartialViewResult
    {
        ViewName = "_SearchResults",
        ViewData = new ViewDataDictionary<List<string>>(this.ViewData, searchResults),
    };
}
```
Don't be hold you can simplify this with the new **Partial** action result helper, which supposed to be a part of **ASP.NET Core 2.2** if my [PR](https://github.com/aspnet/Mvc/pull/7994) is merged.
```csharp
public IActionResult OnPostSearchBooks()
{
    var searchResults = new List<string>
    {
        $"Book 1 (ISBN: {Guid.NewGuid()})",
        $"Book 2 (ISBN: {Guid.NewGuid()})",
    };

    **return Partial("_SearchResults", searchResults);**
}
```
Now it's the time to see how we implement this helper .. to make this happen we need to modify the **RazorPage & PageModel**.  

First we need to add the following two helper methods to **PageBase**:  
```csharp
public virtual PartialViewResult Partial(string viewName)
{
    return Partial(viewName, model: null);
}

public virtual PartialViewResult Partial(string viewName, object model)
{
    ViewContext.ViewData.Model = model;

    return new PartialViewResult
    {
        ViewName = viewName,
        ViewData = ViewContext.ViewData
    };
}
```
Then similarly we can add the above snippet into the **PageModel** except using `ViewData` instead of `ViewContext.ViewData`  

That's it!! now you 're able to use the newly added helper directly from within the **RazorPage & PageModel**.  

Finally If you need to see a complete sample, I'd like to refer to the [Parnav K](https://github.com/pranavkm) sample which is a part of [Entropy repository](https://github.com/aspnet/Entropy/tree/master/samples/Mvc.RazorPagePartial)  and  I will update the sample once my PR is merge.  

Happy Coding!!