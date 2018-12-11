---
Id: 22
Posted Date: 9/1/2017
Tags: Markdown 
Slug: mdocs-building-a-project-documentation-using-markdown-view-engine
---
# mDocs: Building a project documentation using Markdown View Engine

In the last blog post I talked about [Introducing a new Markdown View Engine for ASP.NET Core](http://www.hishambinateya.com/introducing-a-new-markdown-view-engine-for-asp.net-core) the new ASP.NET Core view engine that using the markdown syntax for the views. While I'm working on it, I come up with an idea to build a useful tool for building a project documentation. AFAIK the markdown syntax is one of the simplest and easiest formats for creating the documentation.Of course with Markdown View Engine it will be super easy to build such tool.

There 's nothing fancy in mDocs, it is a simple ASP.NET Core web application that using Markdown View Engine. The project contains one controller named `DocumentationController` which contains a single action to serve all the markdown views as the following:
```csharp
public class DocumentationController : Controller
{
    [Route("/{*url}")]
    public IActionResult Index(string url)
    {
        if (url == null || url.EndsWith("/"))
        {
            url += "Index";
        }

        return View(url);
    }
}
```
Furthermore I added a piece of code to change the default controller convention, to read the views from the Docs folder as the following:
```csharp
public class DocsConvention : IControllerModelConvention
{
    public void Apply(ControllerModel controller)
    {
        if (controller.ControllerName == "Documentation")
        {
            controller.ControllerName = "Docs";
        }
    }
}
```
### Using mDocs

As I mentioned before that mDocs is basically an ASP.NET Core web application, so anyone familiar can run it easily, but I will write down the detailed steps for other folks:

1.  Run `dotnet-pack` command on the [Markdown View Engine](https://github.com/hishamco/MarkdownViewEngine/) project.
2.  Use "Manage NuGet Packages" to restore the generated .nupkg file.
3.  Run `dotnet-restore`.
4.  Skip the steps 4 and 5 if you wanna use the default template.
5.  Add all of your markdown pages inside the "/Views/Docs/" folder.
6.  Add page title by adding **@page title="Page Title"** on the top of each .md page.
7.  Run `dotnet-run`.
8.  Open the browser with **localhost:5000/**.

That is it!!

Last thing I wanna point out that mDocs came up with a sample documentation for **.NET Core Command-Line Interface (CLI)** using the [Read the docs](https://readthedocs.org/) theme, the figure below illustrate a sample page of it.

![](http://www.hishambinateya.com/images/posts/97b0cdeb-b3b4-4dde-8b0f-63de85771a8e.png)

You can download the source code for this post from my [mDocs](https://github.com/hishamco/mDocs) repository on GitHub.

Happy Coding !!