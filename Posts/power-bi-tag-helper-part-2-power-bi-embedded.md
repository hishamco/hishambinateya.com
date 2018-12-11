---
Id: 24
Posted Date: 22/1/2017
Tags: TagHelpers,Power BI 
Slug: power-bi-tag-helper-part-2-power-bi-embedded
---
# Power BI Tag Helper: Part 2 - Power BI Embedded

In the last blog post I talked about the [Power BI Tag Helper: Part 1 - Power BI Publish to Web](http://hishambinateya.com/power-bi-tag-helper:-part-1-power-bi-publish-to-web) which I explored the publish to web feature in the Power BI portal. Today I will continute with the second part which I will explain what is the Power BI Embedded? How we can use it? An what the informations needed in case of authoring a TagHelper to embed a Power BI report into our ASP.NET Core application.

### What is Power BI Embedded?

Power BI Embedded is an Azure service that let you host Power BI reports and dashboards into your web or mobile applications. Your users don't need Power BI account and they access Power BI reports and dashboards via your application.

I would like to mention two of the greatest resources about the Power BI Embedded that I found in [Channel 9](https://channel9.msdn.com) (Azure Friday Show), [Scott Hanselman](https://twitter.com/shanselman) talks to Josh Caplan about:

**Azure Friday: Introducing Microsoft Power BI Embedded**

[![](https://sec.ch9.ms/ch9/2794/7c4503d0-b1ec-4827-81b2-4e9fee4a2794/PowerBIImbedded_960.jpg)](https://sec.ch9.ms/ch9/2794/7c4503d0-b1ec-4827-81b2-4e9fee4a2794/PowerBIImbedded_high.mp4)

**Azure Friday: PowerBI Embedded GA**

[![](https://sec.ch9.ms/ch9/c40f/e8cc2985-5a03-4260-b0d7-3fb0a178c40f/AzureFridayPowerBIEmbedded_960.jpg)](https://sec.ch9.ms/ch9/c40f/e8cc2985-5a03-4260-b0d7-3fb0a178c40f/AzureFridayPowerBIEmbedded_high.mp4)

### How to use Power BI Embedded

Resources for Microsoft Power BI Embedded are provisioned through the [Azure ARM APIs](https://msdn.microsoft.com/library/mt712306.aspx). In this case, the resource you provision is a **Power BI Workspace Collection**.

A Workspace Collection is the top-level Azure resource and a container for the content that will be embedded in your application. A Workspace Collection can be created in two ways:

*   Manually using the Azure Portal
*   Programmatically using the Azure Resource Manager(ARM) APIs

The following steps describe how to build a **Workspace Collection** using the Azure Portal :

1.  Open and sign into **Azure Portal**: [http://portal.azure.com](http://portal.azure.com).
2.  Click **\+ New** on the top panel.
3.  Under **Data + Analytics** click **Power BI Embedded**.  
      
    ![](http://www.hishambinateya.com/images/posts/7f3d81bb-a290-40fb-b9d2-f1cf25652064.png)
    
4.  On the **Workspace Collection Blade**, enter the required information.  
      
    ![](http://www.hishambinateya.com/images/posts/188b774b-ee00-4f39-b7cc-95e656238bd8.png)
    
5.  Click **Create**.

The Workspace Collection will take a few moments to provision. When completed, you'll be taken to the Workspace Collection Blade.  
  
![](http://www.hishambinateya.com/images/posts/def61fa8-a814-48a1-899e-c8327ae7a0fb.png)

At this point you need to provision a new workspace in the workspace collection that we created above, also you need to import your **.PBIX** file, for that please refer to this [link](https://github.com/Microsoft/azure-docs/blob/master/articles/power-bi-embedded/power-bi-embedded-get-started-sample.md) it will guide you for all the steps that you need to achieve this.

Now let us dig into the `PowerBITagHelper` which is very straightforward, it basically construct the script required by Power BI JavaScript.
```csharp
[HtmlTargetElement("power-bi", TagStructure = TagStructure.NormalOrSelfClosing)]
public class PowerBITagHelper : TagHelper
{
    [HtmlAttributeName("accessKey")]
     public string AccessKey { get; set; }

    public int Height { get; set; } = 600;

    [HtmlAttributeName("reportId")]
    public string ReportId { get; set; }

    public int Width { get; set; } = 800;

    [HtmlAttributeName("workspaceCollectionName")]
    public string WorkspaceCollectionName { get; set; }

    [HtmlAttributeName("workspaceId")]
    public string WorkspaceId { get; set; }

    private string AccessToken
    {
        get
        {
            // For more information refer to this link https://docs.microsoft.com/en-us/azure/power-bi-embedded/power-bi-embedded-iframe
            var token1 = "{" +
                "\"typ\":\"JWT\"," +
                "\"alg\":\"HS256\"" +
           "}";
            var token2 = "{" +
                $"\"wid\":\"{WorkspaceId}\"," +
                $"\"rid\":\"{ReportId}\"," +
                $"\"wcn\":\"{WorkspaceCollectionName}\"," +
               "\"iss\":\"PowerBISDK\"," +
               "\"ver\":\"0.2.0\"," +
               "\"aud\":\"https://analysis.windows.net/powerbi/api\"," +
               "\"nbf\":" + DateTime.UtcNow.ToUnixTimestamp() + "," +
               "\"exp\":" + DateTime.UtcNow.AddHours(1).ToUnixTimestamp() +
            "}";

            var inputValue = $"{UrlEncode(token1)}.{UrlEncode(token2)}";
            var hash = new HMACSHA256(Encoding.UTF8.GetBytes(AccessKey))
                .ComputeHash(Encoding.UTF8.GetBytes(inputValue));
            var signature = UrlEncode(hash);

            return $"{inputValue}.{signature}";
        }
    }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = null;
        if (WorkspaceCollectionName != null && WorkspaceId != null && AccessKey != null & ReportId != null)
        {
            string style = $"width:{Width}px;height:{Height}px";
            if(context.AllAttributes.TryGetAttribute("style", out var styleAttribute))
            {
                style = $"{styleAttribute.Value};{style}";
            }

            var content = $@"<div id=""reportContainer"" style=""{style}""></div>

            <script>
                $(function() {{
                    var embedConfiguration = {{
                        type: 'report',
                        accessToken: '{AccessToken}',
                        id: '{ReportId}',
                        embedUrl: 'https://embedded.powerbi.com/appTokenReportEmbed?reportId={ReportId}'
                    }};
                var $reportContainer = $('#reportContainer');
                var report = powerbi.embed($reportContainer.get(0), embedConfiguration);
                }})
            </script>";
            output.Content.SetHtmlContent(content);
        }
    }
}
```
Now we can simply use it in our view like this:
```html
<power-bi workspaceCollectionName="DemoWorkspaceCollection"
          workspaceId="361fe362-4c7a-47d3-b27b-b4cd3441ead0"
          accesskey="npQ6PPgsbCB8zR5OVEmpPDf3AqtlvQRTdz5tzkFtXIP41ZPwgXpKBQQhthV7CkYSzeVRJg1PSkbU1p2AqY0ExA=="
          reportId="1f7004ba-d989-4dfa-b101-eba56c68f70b" /> 
```
Last thing you need to provide: workspace collection name, workspace id, access key and report id, you can get all the information except that last one from the Azure portal, so please refer to the figure above, after you run the page you should see your report rendered in the browser.

![](http://www.hishambinateya.com/images/posts/389f159d-7232-47cc-98c9-1472bb0050fe.png)

On more thing I want to clarify don't worry about the information that you provide to the TagHelper especially the "Access Key**"** I know it's sensitive information but all are required to generate the **Access Token** for the application at the server side and they will not sent or rendered in the html source.

You can download the source code for this post from my [PowerBI-TagHelper](https://github.com/hishamco/PowerBI-TagHelper) repository on GitHub.

Happy Coding !!