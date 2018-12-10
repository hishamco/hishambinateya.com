---
Id: 42
Posted Date: 25/7/2018
Tags: Razor Pages,Events 
Slug: razor-pages-commands-and-events
---
# Razor Pages Commands & Events

Before two weeks ago I blogged about [Customize Razor Pages Handlers](http://hishambinateya.com/customize-razor-pages-handlers), I achieve that by introducing sort of Command Pattern to the Razor Pages, which allows you to write your custom commands that we will be executed on the _HTTP POST_.

Today I wanna go further with the pattern that I introduced in the last blog post by adding events to the Razor Pages, which is something similar to what we have seen in ASP.NET WebForms in the previous versions of the ASP.NET. In reality I inspire this from the prototype that I'm working on [GitHub](https://github.com/hishamco/webforms), to bring the WebForms back in ASP.NET Core, I know It's a crazy idea :) believe it or not this not like the one that we knew, my aim was to bring an event programming model to the ASP.NET Core along side with MVC & Razor Pages.

Basically the idea is not that tough, I just want to bind some of the **JavaScript** events such as `onclick`, `onchange` .. etc with Razor Pages Handler Methods. From my WebForms prototype I found that the simplest way is by associated the `form.submit()`with the specified event to let the current form doing the postback instead of clicking the submit button.

Let us see how can we tweak the previous blog post code to allow this ..

The major changes will happen in the `CommandTagHelper`, so we need to support more _HTML_ controls and attach the events that we intrest in.
```csharp
 [HtmlTargetElement("input", Attributes = CommandNameAttributeName)]
 [HtmlTargetElement("select", Attributes = CommandNameAttributeName)]
 [HtmlTargetElement("button", Attributes = CommandNameAttributeName)]
 [HtmlTargetElement("button", Attributes = CommandArgumentsAttributeName + "*")]
 public class CommandTagHelper : TagHelper
 {
     private const string CommandNameAttributeName = "asp-command";
     private const string CommandArgumentsAttributeName = "asp-command-";

     private string _eventName;

     [HtmlAttributeName(CommandNameAttributeName)]
     public string Name { get; set; }

     [HtmlAttributeName(DictionaryAttributePrefix = CommandArgumentsAttributeName)]
     public IDictionary<string, string> Arguments { get; set; } =
         new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);

     [HtmlAttributeNotBound]
     [ViewContext]
     public ViewContext ViewContext { get; set; }

     public override void Process(TagHelperContext context, TagHelperOutput output)
     {
         switch (context.TagName)
         {
             case "button":
                 _eventName = "onclick";
                 break;
             case "input":
                 switch (output.Attributes["type"].Value)
                 {
                     case "checkbox":
                     case "radio":
                     case "submit":
                         _eventName = "onclick";
                         break;
                     case "text":
                         _eventName = "onchange";
                         break;
                     default:
                         // Ignore other types
                         break;
                 }
                 break;
             case "select":
                 _eventName = "onchange";
                 break;
             default:
                 break;
         }

         var pagePath = $@"{ViewContext.RouteData.Values["page"].ToString()}/{Name}";
         if (Arguments != null && Arguments.Count != 0)
         {
             var queryString = string.Join("&", Arguments.Select(r => $"{r.Key}={r.Value}"));
             pagePath = $"{pagePath}?{queryString}";
         }

         var eventAttribute = output.Attributes[_eventName];
         if (eventAttribute != null)
         {
             output.Attributes.Remove(eventAttribute);
         }

         output.Attributes.Add(_eventName, $@"this.form.action=""{pagePath}"";this.form.submit()");
     }
 }
```
If you notice in the last blog post I passed the handler name via the _HTTP POST_ which fit in my case, but after I heard [Damian Edwards](https://twitter.com/DamianEdwards) feedback in [ASP.NET Community Standup](http://live.asp.net/) ,seems we can do this in better and easy way by adding the **Command** key to the page route like the following:

@page "{command?}"

but again, this is a tedious work to do in every page!!

thanks to [Pranav K](https://github.com/pranavkm) for his suggestion [here](https://github.com/aspnet/Mvc/issues/6535#issuecomment-316769293) to register this globally using the following code snippet:
```csharp
using Microsoft.AspNetCore.Mvc.ApplicationModels;
...

.AddMvc()
.AddRazorPagesOptions(options =>
{
    options.Conventions.AddFolderRouteModelConvention("/", model =>
    {
        foreach (var selector in model.Selectors)
        {
            var attributeRouteModel = selector.AttributeRouteModel;
            attributeRouteModel.Template = AttributeRouteModel.CombineTemplates(attributeRouteModel.Template, "{handler?}");
        }
    });
});
```
Now you are able to execute the Razor Pages Handler Methods on certain event if your page model inherits from `CommandPageModel`.

Last thing I wanna show you is one of the common scenarios that I have seen several times in MVC which is a **Cascade DropDownList**, which is not a simple thing you can do previously like WebForms, but using this technique it will be super easy:
```html
<select asp-items="Model.Countries" asp-command="OnCountryChange" class="form-control"></select>

<select asp-items="Model.Cities" class="form-control"></select>
```
You can download the source code for this post from my [RazorPagesCommands](https://github.com/hishamco/RazorPagesCommands) repository on GitHub.

Happy Coding !!