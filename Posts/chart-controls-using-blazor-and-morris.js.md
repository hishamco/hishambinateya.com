---
Id: 61
Posted Date: 30/10/2018
Tags:  Blazor, BlazorComponents, Charts 
Slug: chart-controls-using-blazor-and-morris.js
---
# Chart Controls using Blazor & morris.js

Two years ago I blogged about [Chart Controls using TagHelpers & morris.js](http://www.hishambinateya.com/chart-controls-using-taghelpers-and-morris.js) which shows **morris.js** charts creation using **TagHelpers**. TagHelper was great at the time for creating component-based model, but if you look closely to the `ChartTagHelper` that I created you will really feel the pain.

Yesterday I started to put my hands on **Blazor** which is a cool piece of technology, it has a powerful component-based programming model using `BlazorComponent`, which is the thing that let me start to migrate some of UI related projects and ideas that I blogged about before to **Blazor**, fortunately one of them was chart controls.

### Blazor Component: View as TagHelper is a Dream Comes True!!

Around two years ago [Damian Edwards](https://twitter.com/damianedwards) is filed an issue on GitHub with a crazy idea that aims to use a **cshtml** view as `TagHelper`. The idea super useful for many scenarios - one of them UI components - it will simplify the UI creation and generation, furthermore it will give us a good separation of concern, I'd like to say to Damo _"I think it is the time to say that your dream comes true with_ `BlazorComponent`_" ._

To show the need of the `BlazorComponent` for component-based model, let return back and explore our `ChartTagHelper`
```csharp
public class ChartTagHelper: TagHelper
{
    private IList<PropertyInfo> Properties => Source.First().GetType().GetProperties().ToList();

    private IList<string> PropertyNames => Properties.Select(p => p.Name).ToList();
        
    [HtmlAttributeName("id")]
    public string Id { get; set; }
        
    [HtmlAttributeName("type")]
    public ChartType Type { get; set; }
        
    [HtmlAttributeName("labels")]
    public string Labels { get; set; }
        
    [HtmlAttributeName("source")]
    public IEnumerable<object> Source { get; set; }
        
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.Attributes.SetAttribute("id", Id);
        output.TagName = "div";
            
        if (Source == null)
        {
            return;
        }
            
        if (Type == ChartType.Donut)
        {
            output.PostElement.AppendHtml($@"
<script>
Morris.{Type}({{
  element: '{Id}',
  data: [
    {ConstructDonutChartData()}
  ]
}});
  </script>");
        }
        else
        {
            var xKey = PropertyNames.First();
            var yKeys = PropertyNames.Skip(1).ToList();
                
            output.PostElement.AppendHtml($@"
<script>
Morris.{Type}({{
  element: '{Id}',
  data: [
    {ConstructChartData()}
  ],
  xkey: '{xKey}',
  ykeys: [{String.Join(",",yKeys.Select(p=> "'" + p + "'"))}],
  labels: [{String.Join(",",Labels.Split(',').Select(l => "'" + l + "'"))}]
}});
  </script>");
        }
    }
        
    private string ConstructDonutChartData()
    {
        var labelProperty = Source.First().GetType().GetProperty("label");
        var valueProperty = Source.First().GetType().GetProperty("value");
            
        return String.Join(",", Source.Select(s => 
            $"{{label: '{labelProperty.GetValue(s)}', value: {valueProperty.GetValue(s)}}}"));
    }
        
    private string ConstructChartData()
    {
        var xKey = PropertyNames.First();
        var xProperty = Properties.First();
        var yProperties = Properties.Skip(1);
            
        return String.Join(",", 
            Source.Select(s => $"{{{xKey}:'{xProperty.GetValue(s)}', " + String.Join(",", yProperties.Select(p => p.Name + ":" + p.GetValue(s))) + "}"
        ));
    }
}
```
If we look to the bold code snippets you will recognize the complexity for the UI creation, and when you start building a more complex UI you will really feel the pain :( that is because the `TagHelper` is not built from ground up for UI component-based scenarios, on other hand `BlazorComponent` it did!! that means building & creation UI components is easier than before.

### Blazor Chart Component

Let us start to create our first `BlazorComponent` ..

```csharp
@using Microsoft.AspNetCore.Blazor.Components

<div id="@Id"></div>

@functions{
    [Parameter]
    private string Id { get; set; }

    [Parameter]
    private ChartType Type { get; set; }

    [Parameter]
    private IEnumerable<object> DataSource { get; set; }

    [Parameter]
    private string XKey { get; set; }

    [Parameter]
    private string YKeys { get; set; }

    [Parameter]
    private string Labels { get; set; }

    protected override void OnAfterRender()
    {
        switch (Type)
        {
            case ChartType.Area:
                ChartJSInterop.InitializeAreaChart(Id, DataSource, XKey, YKeys.Split(','), Labels.Split(','));
                break;
            case ChartType.Bar:
                ChartJSInterop.InitializeBarChart(Id, DataSource, XKey, YKeys.Split(','), Labels.Split(','));
                break;
            case ChartType.Donut:
                ChartJSInterop.InitializeDonutChart(Id, DataSource);
                break;
            case ChartType.Line:
                ChartJSInterop.InitializeLineChart(Id, DataSource, XKey, YKeys.Split(','), Labels.Split(','));
                break;
        }
    }
}
```

The code is straightforward, basically it's a **cshtml** view  that exposes few parameters to allow the developer set the chart type, data source, labels .. etc. When`OnAfterRender()` occurs the component will decide which chart type should be rendered using the `ChartJsInterop` class which is nothing but a wrapper that manage the interaction with the **morris.js** charts APIs.

```csharp
public static class ChartJSInterop
{
    public static Task InitializeLineChart(string element, IEnumerable<object> dataSource, string xKey, IEnumerable<string> yKeys, IEnumerable<string> labels)
    {
        return JSRuntime.Current.InvokeAsync<object>("charts.drawLineChart", element, dataSource, xKey, yKeys, labels);
    }

    public static Task InitializeAreaChart(string element, IEnumerable<object> dataSource, string xKey, IEnumerable<string> yKeys, IEnumerable<string> labels)
    {
        return JSRuntime.Current.InvokeAsync<object>("charts.drawAreaChart", element, dataSource, xKey, yKeys, labels);
    }

    public static Task InitializeBarChart(string element, IEnumerable<object> dataSource, string xKey, IEnumerable<string> yKeys, IEnumerable<string> labels)
    {
        return JSRuntime.Current.InvokeAsync<object>("charts.drawBarChart", element, dataSource, xKey, yKeys, labels);
    }

    public static Task InitializeDonutChart(string element, IEnumerable<object> dataSource)
    {
        return JSRuntime.Current.InvokeAsync<object>("charts.drawDonutChart", element, dataSource);
    }
}
```

Now let us consume and make of use of the chart component on the page with single tag as the following:

```html
@page "/charts"
@inherits ChartsComponent

<h1>Charts</h1>

<Chart Id="line-example" Type="ChartType.Line" DataSource="@LineChartDataSource" XKey="month" YKeys="value" Labels="Values" />
<Chart Id="area-example" Type="ChartType.Area" DataSource="@AreaChartDataSource" XKey="year" YKeys="a,b,c" Labels=".NET Core,ASP.NET Core,EntityFramework Core" />
<Chart Id="bar-example" Type="ChartType.Bar" DataSource="@BarChartDataSource" XKey="year" YKeys="i,pr,c" Labels="Issues,Pull Requests,Comments" />
<Chart Id="donut-example" Type="ChartType.Donut" DataSource="@DonutChartDataSource" />
```

Finally when you run the application the chart components will look like the figure below.

![](https://raw.githubusercontent.com/hishamco/BlazorChartControls/master/Screenshot.png)

You can download the source code for this blog post from my [BlazorChartControls](https://github.com/hishamco/BlazorChartControls) repository on GitHub.

Happy Coding!!