---
Id: 44
Posted Date: 22/8/2018
Tags: IIS, IIS Administration 
Slug: iis-administration-api
---
# IIS Administration API

Last few days I saw three cool episodes for [Code Conversions](https://channel9.msdn.com/Shows/Code-Conversations) show by the great [Jon Galloway](https://twitter.com/jongalloway) & Jimmy Campbell which they are talking about IIS Administration API:

*   .[NET Core Plugin Architecture in the IIS Administration API](https://channel9.msdn.com/Shows/Code-Conversations/NET-Core-Plugin-Architecture-in-the-IIS-Administration-API-with-Jimmy-Campbell)
*   [Website Resource Implementation in IIS Administration API](https://channel9.msdn.com/Shows/Code-Conversations/Website-Resource-Implementation-in-IIS-Administration-API-with-Jimmy-Campbell)
*   [The File API in IIS Administration API](https://channel9.msdn.com/Shows/Code-Conversations/The-File-API-in-IIS-Administration-API-with-Jimmy-Campbell)

### What's IIS Administration API

The IIS Administration API are simply a REST API for managing Internet Information Service (IIS). The main reason that leads me to write this blog post is the entire IIS Administrations is built on top ASP.NET Core with pluggability architecture, the thing that makes me more interesting :)

### Using IIS Administration API

Let me show you how easy it's to deal with the newly IIS Administration API. Before writing utilize the IIS Administration API we need some steps:

1.  Navigate to [https://manage.iis.net](https://manage.iis.net)
2.  Click 'Get Access Token'
3.  Generate an access token and copy it to the clipboard
4.  Exit the access tokens window and return to the connection screen
5.  Paste the access token into the Access Token field of the connection screen
6.  Click 'Connect'

Now let us illustrate how to create a new website in IIS:
```csharp
    // Initialize API Client
    var httpClientHandler = new HttpClientHandler() {
        Credentials = new NetworkCredential(userName, password, domain)
    };
    var apiClient = new HttpClient(httpClientHandler);
    apiClient.DefaultRequestHeaders.Add("Access-Token", "Bearer {token}");
    
    // Create a Website
    var myWebsite = new {
        name = "MyWebsite",
        physical_path = @"C:\MyWebsite",
        bindings = new[] {
            new { port = 8080, is_https = false, ip_address = "*" }
        }
    };
    var result = apiClient.PostAsJsonAsync<object>("https://localhost:55539/api/webserver/websites", myWebsite).Result;
    if (result.StatusCode != HttpStatusCode.Created) {
        HandleError(result);
        return;
    }
    
    var site = JObject.Parse(result.Content.ReadAsStringAsync().Result);
```
In the above code I first initialize the API Client with the Access Token that we got, then I provide the website information like what we are doing in the IIS (name, path and binding).

After that I use the API `PostAsJsonAsync()` with the object that we created to create a new website in IIS, last thing we can verifying using the `StatusCode` whether the website is created or not.

We saw how easy it is to consume and use the IIS Administration API, but there many of them to let you do many many things programmatically using a RESTful APIs.

For more information you can download the source code for IIS Administration API from the [IIS.Administration](https://github.com/Microsoft/IIS.Administration) repository on GitHub.

Happy Coding !!