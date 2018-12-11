---
Id: 23
Posted Date: 15/1/2017
Tags: TagHelpers,Power BI 
Slug: power-bi-tag-helper-part-1-power-bi-publish-to-web
---
# Power BI Tag Helper: Part 1 - Power BI Publish to Web

Few months ago I wrote a blog post about [Chart Controls using TagHelpers & morris.js](http://hishambinateya.com/chart-controls-using-taghelpers-and-morris.js), which is something interesting, it shows a pretty cool data visualization controls without JavaScript required!! in the view. Today I'm gonna talk about one of the coolest services that Microsoft brings to the market called **Power BI**, which providers a rich data visualizations with business intelligence capabilities. So let us give a little context for those who never heard about Power BI before we dig into the actual post.

Power BI is a business analytics service provided by Microsoft. It provides interactive visualizations with self-service business intelligence capabilities, where end users can create reports and dashboards by themselves, without having to depend on any information technology staff or database administrator. ([Wikipedia](https://en.wikipedia.org/wiki/Power_BI))

Publish to web feature makes your report available to anyone over internet, no authentication required, so you can easily embed interactive Power BI visualizations online, such as in blog posts, websites, through emails or social media, on any device.

### How to use Publish to Web

The following steps describe how to use **Publish to web**:

1.  On a report in your workspace, select **File > Publish to web**.
    
    ![](http://www.hishambinateya.com/images/posts/068f48a5-0f30-41a8-ad12-8aab75838019.png)
    
2.  Review the content on the dialog, and select **Create embed code**.
3.  Review the warning, and confirm that the data is okay to embed in a public website. If so, select **Publish**.
4.  A dialog appears that provides a link that can be sent in email, embedded in code (such as an iFrame), or that you can paste directly into your web page or blog.
5.  If you’ve previously created an embed code for the report, the embed code quickly appears.
    
    ![](http://www.hishambinateya.com/images/posts/f196f7b9-e622-4411-96e2-987488f2f111.png)
    

Now let us dig into the `PowerBITagHelper` which is very straightforward. It basically construct the IFrame that 's generated in the Power BI portal.
```csharp
[HtmlTargetElement("power-bi", TagStructure = TagStructure.NormalOrSelfClosing)]
public class PowerBITagHelper : TagHelper
{
    private static readonly string _powerBIAppUrl = "https://app.powerbi.com/";

    public int Height { get; set; } = 600;

    public int Width { get; set; } = 800;

    [HtmlAttributeName("reportId")]
    public string ReportId { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        var src = $"{_powerBIAppUrl}view?r={ReportId}";
        output.TagName = null;
        output.Content.SetHtmlContent($@"<iframe width=""{Width}"" height=""{Height}"" src=""{src}"" frameborder=""0"" allowFullScreen=""true""></iframe>");
    }
}
```
Now we can simply use it in our view like this:
```html
<power-bi reportId="eyJrIjoiMDRlMWFhZjQtNTc3YS00YjhjLTlmMDgtYzMyMTdhMzA4NjEzIiwidCI6IjI2YWQxODUzLTZiM2UtNGE1NC05MzQ3LWFiM2U3MWNjNmU2MSIsImMiOjl9" /> 
```
The only thing that requires is the report id or code that's generated in the Power BI portal which shown in the figure above, after you run the page you should see your report rendered in the browser.

![](http://www.hishambinateya.com/images/posts/205dffb0-7d6f-44d0-87c3-bc9d6ad83975.png)

You can download the source code for this post from my [PowerBI-TagHelper](https://github.com/hishamco/PowerBI-TagHelper) repository on GitHub.

Happy Coding !!