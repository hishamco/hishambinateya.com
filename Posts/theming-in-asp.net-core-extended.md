---
Id: 39
Posted Date: 13/6/2017
Tags: Themes 
Slug: theming-in-asp.net-core-extended
---
# Theming in ASP.NET Core Extended

In the last post I blogged about [Theming in ASP.NET Core](http://hishambinateya.com/theming-in-asp.net-core), after awhile I saw one comment from Mark Rendle

> Please, don't do this. Themes are what CSS is for. You don't need to create multiple copies of identical Razor views just to change the appearance of things in the browser. Just provide different CSS files (with related image sets or whatever) and use an app setting or user preference to generate the link tag.

This blog post is responding to the above comment and remove some of the confusion from the latest post.

The main issue that may raised by some people after reading the previous post is when I used the light & dark themes, because I changed the colors only. I knew that we can switching between them easily using a simple condition in the layout page, and there's no need to duplicate the views per theme. Perhaps it's my fault!! because I didn't provide a proper themes last time. Nothing but my intend was to show you how to use the theming in ASP.NET Core and how ViewLocalizationExpanders can simplify the entire process of theming.

Today I tweaked the code little bit to move the common views to the _Views_ folder, and keep the different one in _Themes_ folder. In this way there's no need to duplicate all the views per theme anymore!!
```csharp
private IEnumerable<string> ExpandViewLocationsCore(IEnumerable<string> viewLocations, string theme)
{
    foreach (var location in viewLocations)
    {
        yield return location;
        yield return location.Insert(7, $"Themes/{theme}/");
    }
}
```
the change was trivial by adding this line of code:
```csharp
yield return location;
```
the Razor view engine will lookup for the views in both the _Views_ and _Themes_ folders.

That is it!! now whatever the layout was complex or different the theming will work as expected.

Now if your run the sample again with these settings:
```json
  "AppSettings": {
    "Theme": "LimonTheme"
  }
```
the layout will look like this

![](http://www.hishambinateya.com/Images/Posts/34257b71-7a72-4d0f-ac8d-778cc2e7a4e8.png)

FYI I used [Goovin Bootstrap Theme](https://bootstrapmade.com/demo/Groovin/) but you can pick the one that you like, again no need to duplicate the views :)

Last thing I wanna point to is Michael Esteves's comment

> I think this is also a interesting way to have an old and new version of your site running side by side allowing the user to pick which one they want while the new version is in development. So not just css changes but layout changes.

the idea is very interesting, because sometimes you may need to show the visitors a preview of your new website design while it's in beta version.

You can download the source code for this post from my [Theming](http://github.com/hishamco/theming) repository on GitHub.

Happy Coding !!