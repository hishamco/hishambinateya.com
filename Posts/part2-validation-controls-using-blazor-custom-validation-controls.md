---
Id: 63
Posted Date: 10/12/2018
Tags:  Validation, Blazor
Slug: part2-validation-controls-using-blazor-custom-validation-controls
---
# Part II: Validation Controls using Blazor - Custom Validation Controls

Last two I was busy with moving all my blog posts to [GitHub](https://www.github.com) as markdown format. Today I will continue on the second part of **Chart Controls using Blazor**.

I'd like to mention that I was very interested in [Damian Edwards](https://twitter.com/DamianEdwards) comments after I watched a the latest recorded episode of [ASP.NET Community Standup](https://www.live.asp.net), after a long discussion with [Jon Galloway](https://twitter.com/jongalloway) about validations and SPA frameworks. The interesting part the ASP.NET team may go with **Model Validation** approach during building validation in Blazor in the upcoming versions, the thing that let me thinking if I can write another part in this series to talk about the model validation.

Since two weeks I talked about the basic validation controls which are used quite often in almost web application, but they are limited in some cases, with that `CustomValidator` come for the rescue to handle any sort of validation logic that you want to write.

### CustomValidator Component

The `CustomValidator` component allows writing specific custom validation routines for both the client-side and the server-side validation.
 
 ```csharp
@using Microsoft.AspNetCore.Blazor.Components

@functions{
    [Parameter]
    private string ControlToValidate { get; set; }

    [Parameter]
    private string ClientValidationFunction { get; set; }

    [Parameter]
    private string ServerValidationFunction { get; set; }

    [Parameter]
    private string ErrorMessage { get; set; }

    protected override void OnAfterRender()
    {
        ValidationJSInterop.Validate(ControlToValidate, ClientValidationFunction, ServerValidationFunction, ErrorMessage);
    }
}
```
The client-side validation is accomplished through the `ClientValidationFunction` property, the validation logic should be written in JavaScript. On other hand the server-side validation is accomplished through the `ServerValidationFunction` property, the validation logic should be written C#.

According to your needs you can write a C# function
```csharp
@functions{
    [JSInvokable]
    public static bool CheckEmailAvailability(string email)
    {
        // Checking email availability logic
        ...
        return isAvailable;
    }
}
```
or a JavaScript function
```javascript
<script>
    function checkEmailAvailability(email) {
        // Checking email availability logic
        ...
        return isAvailable;
    }
</script>
```

both are same, but the validating place are difference.

All the magic happen in the **validation.js** to trigger the **boostrap validations** based on the user input.
```javascript
window.validation = {
    ...
    validate: function (element, clientFunction, serverFunction, msg) {
        var e = document.getElementById(element);
        var div = document.createElement("div");
        div.classList.add('invalid-feedback');
        div.innerHTML = msg;
        e.parentNode.insertBefore(div, e.nextSibling);
        if (serverFunction == null) {
            e.setAttribute("onchange", clientFunction)
            $(e).on('change', function () {
                var isValid = window[clientFunction](e.value);
                if (isValid) {
                    if (e.classList.contains('is-invalid')) {
                        e.classList.remove('is-invalid');
                    }
                }
                else {
                    e.classList.add('is-invalid');
                }
            });
        }
        else {
            $(e).on('change', function () {
                var isValid = DotNet.invokeMethod('Blazory', serverFunction, e.value);
                if (isValid) {
                    if (e.classList.contains('is-invalid')) {
                        e.classList.remove('is-invalid');
                    }
                }
                else {
                    e.classList.add('is-invalid');
                }
            });
        }
        validateForms();
    },
    ...
};
```
If you notice, I used a `DotNet.invokeMethod()` which is a hidden gem in blazor that lets you call a C# function - with JsInvokable attribute - from within JavaScript.

After that we can use JavaScript Interoperability to call the above function from within the component itself using the `ValidationJSInterop`.

Now, let us use our validation control on action by adding a single line of markup per control to make the work done!!
```html
@page "/validation"

<h1>Form Validation</h1>
<form class="needs-validation" novalidate>
    <div class="form-group">
        <label for="txtEmail">Email address</label>
        <input type="text" class="form-control" id="txtEmail" placeholder="Email">
        <CustomValidator ControlToValidate="txtEmail" ServerValidationFunction="CheckEmailAvailability" ErrorMessage="The email is already taken." />
    </div>
    ...
</form>
```
Finally when you run the application the validation components will look like the figure below.

![](https://raw.githubusercontent.com/hishamco/BlazorValidationControls/master/Screenshot.png)

You can download the source code for this blog post from my [BlazorValidationControls](https://github.com/hishamco/BlazorValidationControls) repository on GitHub.

Happy Coding!!