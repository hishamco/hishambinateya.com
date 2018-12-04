---
Id: 59
Posted Date: 25/9/2018
Tags:  Localization,TagHelperComponent
Slug: taghelpercomponent-show-case-adding-language-direction
---
# TagHelperComponent Show Case - Adding Language Direction

Last few days while I was contributing to [SimplCommerce](https://github.com/simplcommerce/SimplCommerce) I filed an issue for adding language direction to the platform because there's a need for such a feature. In this blog post I will explain my experiment during implement this feature.

### Localization & RTL

RTL means Right-To-Left and is for users who use a different alphabet, which is read and written from another direction. There are many languages that support RTL for example: Arabic, Hebrew, Persian .. etc

My as Arabian origin I notice there is a lack for supporting RTL in localization in many programming languages and web technologies including ASP.NET Core. Also the html document by default using LTR direction, so that's why I need a way to set the language direction automatically without human been interfering.

### Introducing LanguageDirectionTagHelperComponent

At the first glance many of us can implement such a feature using `TagHelper`, but my thinking take me away to a `TagHelperComponent`, because essentially what I need is to inject an a **dir** attribute in the **body** tag, without the need to add anything in all the views.

Now let us write the `LanguageDirectionTagHelperComponent`

```csharp
public class LanguageDirectionTagHelperComponent : TagHelperComponent
{
    private const string LanguageDirectionAttribute = "dir";
    private const string BodyTagName = "body";
    private readonly HttpContext _httpContext;
    
    public LanguageDirectionTagHelperComponent(IHttpContextAccessor httpContextAccessor)
    {
        _httpContext = httpContextAccessor.HttpContext;
    }
    
    public override int Order => 1;
    
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        if (string.Equals(context.TagName, BodyTagName, StringComparison.Ordinal))
        {
            var languageDirection = _httpContext.Features.Get<IRequestCultureFeature>()
                .RequestCulture.UICulture.GetLanguageDirection()
                .ToString().ToLower();
            if (!output.Attributes.ContainsName(LanguageDirectionAttribute))
            {
                output.Attributes.Add(LanguageDirectionAttribute, languageDirection);
            }
            else
            {
                output.Attributes.SetAttribute(LanguageDirectionAttribute, languageDirection);
            }
        }
    }
}
```
If we look to the above code snippet is straightforward, the idea is to add a direction attribute for the **body** tag based on the current `RequestCulture.UICulture`.

Then adding the following line in the `ConfigureServices()` method is enough to make the things happen

services.AddScoped<ITagHelperComponent, LanguageDirectionTagHelperComponent>(); 

That's it now the language direction is setup for you in all the views. For more details you can check the source code for this post from my [pull request](https://github.com/simplcommerce/SimplCommerce/pull/568) on GitHub.

Happy Coding!!