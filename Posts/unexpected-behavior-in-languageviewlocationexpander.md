---
Id: 19
Posted Date: 18/11/2016
Tags: Localization 
Slug: unexpected-behavior-in-languageviewlocationexpander
---
# Unexpected Behavior in LanguageViewLocationExpander

In the last blog post I talked about [NeutralResourcesLanguageAttribute Doesn't Respect Localization in ASP.NET Core](http://www.hishambinateya.com/neutralresourceslanguageattribute-doesn%27t-respect-localization-in-asp.net-core) which is a strange issue that the developers may not expect while the localizing their web applications. Today I will point into another unexpected issue may you not think about too :) .

Personally I faced this issue while I'm working on this [pull request](https://github.com/aspnet/Entropy/pull/134), the issue occurs when you are using `LanguageViewLocaltionExpander` to provide a view location.

Assume the view have **fr-FR** locale (**_Layout.fr-FR.cshtml**), and injected `IViewLocalizer` into the view. If the current culture is `new CultureInfo("fr-FR")` you will be 100% sure that the resource search look for **_Layout.fr-FR.resx** even I :) but let me tell you that you and I are wrong!! beucase the resource search look for **_Layout.fr.fr-FR.resx**?!! which is something unbelievable.

The unxpected behavior in `LanguageViewLocaltionExpander` is the resource names that the resource search looking for in **_Layout.{parent culture}.{culture}.resx** format, while all of us expecting **_Layout.{culture}.resx** format which is the right choice IMHO.

That is it!! I know this blog post is very very short, but hope it gives you a knowledge that let you carefully name the resource file names when you are using `LanguageViewLocaltionExpander` along side with `IViewLocalizer`.

Finally there is a detailed explaination about this issue in this [link](https://github.com/aspnet/Mvc/issues/5441)., hopefully we will get a fix for it in the upcoming release of the ASP.NET Core.

Happy Coding !!