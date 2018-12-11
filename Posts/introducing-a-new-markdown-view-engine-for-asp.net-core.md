---
Id: 21
Posted Date: 1/1/2017
Tags: Markdown,ViewEngine 
Slug: introducing-a-new-markdown-view-engine-for-asp.net-core
---
# Introducing a new Markdown View Engine for ASP.NET Core

One of the things that I have been working on has been a new view engine option for ASP.NET Core. ASP.NET Core has always supported the concept of _view engines_ which allow you to use a different template syntax options for your views.

ASP.NET Core shipped with a default view engine that called **Razor** which is of course is the view engine that knew and love from the previous version of ASP.NET.

The new view engine option I have been working on is using markdown syntax which is become popular last few years, especially for those who are working on GitHub.

### What is Markdown?

Markdown is a lightweight markup language with plain text formatting syntax designed so that it can be converted to HTML and many other formats using a tool by the same name. Markdown is often used to format readme files, for writing messages in online discussion forums, and to create rich text using a plain text editor. [_(Wikipedia)_](https://en.wikipedia.org/wiki/Markdown)

### Compared with Razor Syntax

To illustrate the similarities with Razor syntax let me show you some examples that [Scott Gu](https://twitter.com/scottgu) mentioned them long time back in his blog post [Introducing "Razor" - new view engine for ASP.NET](https://weblogs.asp.net/scottgu/introducing-razor).

Hello World Sample with Razor

![](https://aspblogs.blob.core.windows.net/media/scottgu/Media/image_thumb_0E9E3527.png)

Building it with Razor Syntax
```html
<h1>Razor Example</h1>

<h3>
    Hello @name, the year is @DateTime.Now.Year
</h3>

<p>
   Checkout <a href="/Product/Details/@productId">this product</a>
</p>
```
Building it with Markdown Syntax
```markdown
    # Razor Example
    
    ### Hello @name, the year is @DateTime.Now.Year
    
    Checkout [this product](/Product/Details/@productId)
```
Loops and Nested HTML Sample

![](https://aspblogs.blob.core.windows.net/media/scottgu/Media/image_thumb_155D5078.png)

Building it with Razor Syntax
```html
<ul id="products">

    @foreach(var p in products) {
        <li>@p.Name ($@p.Price)</li>
    }

</ul>
```
Building it with Markdown Syntax
```markdown
    @foreach (var p in products) {
      - @p.Name: ($@p.Price)
    }
```
If-Blocks and Multi-line Statements

If Statements

Building it with Razor Syntax
```html
@if(products.Count == 0) {
    <p>Sorry - no products in this category<p>
} else {
    <p>We have a products for you!</p>
}
```
Building it with Markdown Syntax
```markdown
    @if (products.Count == 0) {
    Sorry - no products in this category
    } else {
    We have products for you!
    }
```
Multi-line Statements

Building it with Razor Syntax
```html
@{
    int number = 1;
    string message = "Number is " " + number;
}

<p>Your Message: @message</p>

Building it with Markdown Syntax

    @var number = 1
    @var message = ""Number is "" + number
    
    Your Message: @message
```
Integrating Content and Code

Does it break with email addresses and other usages of @ in HTML?

![](https://aspblogs.blob.core.windows.net/media/scottgu/Media/image_thumb_20963EE8.png)

Building it with Razor Syntax
```html
<p>
    Send mail to scottgu@microsoft.com telling him the time: @DateTime.Now.
</p>
```
Building it with Markdown Syntax
```markdown
Send mail to scottgu@microsoft.com telling him the time: @DateTime.Now.
```
Identifying Nested Content

![](https://aspblogs.blob.core.windows.net/media/scottgu/Media/image_thumb_285E318A.png)

Building it with Razor Syntax
```html
@if (DateTime.Now.Year == 2010) {
    <span>
        if year is 2010 then print this <br/>
        multi-line text block and
        the date: @DateTime.Now
    </span>
}
```
Building it with Markdown Syntax
```markdown
@if (DateTime.Now.Year == 2011) {
If the year is 2011 then print this
multi-line text block and
the date: @DateTime.Now
}
```
Layout/MasterPage Scenarios – The Basics

Simple Layout Example

![](https://aspblogs.blob.core.windows.net/media/scottgu/Media/image_thumb_3BCB4591.png)

Building it with Razor Syntax
```html
<!DOCTYPE html>
<html>
    <head>
        <title>Simple Site</title>
    </head>
    <body>

        <div id="header">
            <a href="/">Home</a>
            <a href="/About">About</a>
        </div>

        <div id="body">
            @RenderBody()
        </div>
    <body>
</html>
```
```html
@{
    LayoutPage = "SiteLayout.cshtml";
}

<h1>About this Site</h1>

<p>
    This is some content that will make up the "about"
    page of our web-site. We'll use this in conjunction
    with a layout template. The content you are seeing here
    comes from Home.cshtml file.
</p>
<p>
    And obviously I can have code in here too. Here is the
    current date/time: @DateTime.Now
</p>
```
Building it with Markdown Syntax
```markdown
<!DOCTYPE html>
<html>
    <head>
        <title>Simple Site</title>
    </head>
    <body>

        <div id="header">
            <a href="/">Home</a>
            <a href="/About">About</a>
        </div>

        <div id="body">
            @Body
        </div>

    </body>
</html>
```
```markdown
@Layout SiteLayout

# About this SiteThis is some content that will make up the ""about""

page of our web-site. We'll use this in conjunction
with a layout template. The content you are seeing here
comes from Home.md file.

And obviously I can have code in here too. Here is the
current date/time: @DateTime.Now
```
Layout/MasterPage Scenarios – Adding Sections Overrides

Coming soon ...

As we have seen before markdown view engine make it easier to write clean views without writing HTML tags, but with the limitation of markdown syntax, we saw that we mix both html and markdown especially in layout pages to wrap up the content pages within <body> and use some styles and scripts, thanks [CommonMark.NET](https://www.nuget.org/packages/CommonMark.NET/) that makes my life happy and make such thing possible.

FYI this is just a prototype which is still in development so that is why I quote some of the Razor syntax to add dynamic functionality to the content, but still I'm thinking if Razor-like syntax is better choice - like what we saw in **@Body** - or we may use handlebars - like **{{Body}}** - or any other sytax to provider rich functionalities.

You can download the source code for this post from my [MarkdownViewEngine](https://github.com/hishamco/MarkdownViewEngine) repository on GitHub.

Happy Coding !!