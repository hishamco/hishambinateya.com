---
Id: 62
Posted Date: 20/11/2018
Tags:  Validation, Blazor
Slug: part1-validation-controls-using-blazor-basic-validation-controls
---
# Part I: Validation Controls using Blazor - Basic Validation Controls

Last time I blogged about [Chart Controls using Blazor & morris.js](http://www.hishambinateya.com/chart-controls-using-blazor-and-morris.js) which is shows how cools & easy to use chart controls using Blazor!! today I will continue blog about another kind of controls, which is validation controls.

As we know validations are very important in almost if not all kind of web applications to avoid unexpected input from the users. If we go back 13 years ago, ASP.NET Web Forms provided a rich set of validation controls to protect the user input. Today I 'd like to start doing similar things using Blazor.

### RequiredFieldValidator Component

The `RequiredFieldValidator` component ensures that the required field is not empty.
 
 ```csharp
@using Microsoft.AspNetCore.Blazor.Components

@functions{
    [Parameter]
    private string ControlToValidate { get; set; }

    [Parameter]
    private string ErrorMessage { get; set; }

    protected override void OnAfterRender()
    {
        ValidationJSInterop.IsRequired(ControlToValidate, ErrorMessage);
    }
}
```
### RangeValidator Component

The `RangeValidator` component verifies that the input value falls within a predetermined range.

```csharp
@using Microsoft.AspNetCore.Blazor.Components

@functions{
    [Parameter]
    private string ControlToValidate { get; set; }

    [Parameter]
    private string ErrorMessage { get; set; }

    [Parameter]
    private int MinimumValue { get; set; }

    [Parameter]
    private int MaximumValue { get; set; }

    protected override void OnAfterRender()
    {
        ValidationJSInterop.IsInRange(ControlToValidate, MinimumValue, MaximumValue, ErrorMessage);
    }
}
```
### CompareValidator Component

The `CompareValidator` component compares a value in one component with a fixed value or a value in another component.

```csharp
@using Microsoft.AspNetCore.Blazor.Components

@functions{
    [Parameter]
    private string ControlToValidate { get; set; }

    [Parameter]
    private string ControlToCompare { get; set; }

    [Parameter]
    private string ErrorMessage { get; set; }

    [Parameter]
    private object ValueToCompare { get; set; }

    protected override void OnAfterRender()
    {
        ValidationJSInterop.CompareTo(ControlToValidate, ControlToCompare, ValueToCompare, ErrorMessage);
    }
}
```
### RegularExpressionValidator Component

The `RegularExpressionValidator` allows validating the input text by matching against a pattern of a regular expression.

```chsharp
@using Microsoft.AspNetCore.Blazor.Components

@functions{
    [Parameter]
    private string ControlToValidate { get; set; }

    [Parameter]
    private string ErrorMessage { get; set; }

    [Parameter]
    private string ValidationExpression { get; set; }

    protected override void OnAfterRender()
    {
        ValidationJSInterop.ValidateRegEx(ControlToValidate, ValidationExpression, ErrorMessage);
    }
}
```
If you looked closely to the code snippets above, you will notice that the validation components aren't render anything as a markup!! the idea that I come up with is to inject the necessary HTML attributes that help to accomplish the validation scenarios that i'm looking for, such as: `required`, `min`, `max` attributes .. etc.

All the magic happen in the **validation.js **which inject the required HTML attributes and elements to trigger the **boostrap validations**.
```javascript
window.validation = {
    isRequired: function (element, msg) {
        var e = document.getElementById(element);
        e.required = true;
        var div = document.createElement("div");
        div.classList.add('invalid-feedback');
        div.innerHTML = msg;
        e.parentNode.insertBefore(div, e.nextSibling);
        validation.validateForms();
    },
    isInRange: function (element, min, max, msg) {
        var e = document.getElementById(element);
        e.min = min;
        e.max = max;
        var div = document.createElement("div");
        div.classList.add('invalid-feedback');
        div.innerHTML = msg;
        e.parentNode.insertBefore(div, e.nextSibling);
        validation.validateForms();
    },
    compareTo: function (element, compareElement, valueToCompare, msg) {
        var e = document.getElementById(element);
        if (compareElement == null) {
            var valueHiddenField = new HTMLInputElement();
            valueHiddenField.type = "hidden";
            var div = document.createElement("div");
            div.classList.add('invalid-feedback');
            div.innerHTML = msg;
            $(e).on('keyup keypress blur change', function () {
                e.classList.remove('is-valid', 'is-invalid');
                if (e.value === valueHiddenField.value) {
                    e.classList.add('is-valid');
                }
                else {
                    e.classList.add('is-invalid');
                }
            });
        }
        else {
            var ec = document.getElementById(compareElement);
            var div = document.createElement("div");
            div.classList.add('invalid-feedback');
            div.innerHTML = msg;
            e.parentNode.insertBefore(div, e.nextSibling);
            $(e).on('keyup keypress blur change', function () {
                e.classList.remove('is-valid','is-invalid');
                if (e.value == ec.value) {
                    e.classList.add('is-valid');
                }
                else {
                    lastChild.innerHTML = msg;
                    e.classList.add('is-invalid');
                }
            });
        }
        validation.validateForms();
    },
    validateRegEx: function (element, expression, msg) {
        var e = document.getElementById(element);
        e.pattern = expression;
        var div = document.createElement("div");
        div.classList.add('invalid-feedback');
        div.innerHTML = msg;
        e.parentNode.insertBefore(div, e.nextSibling);
        validation.validateForms();
    },
    validateForms: function () {
        var forms = document.getElementsByClassName('needs-validation');
        var validation = Array.prototype.filter.call(forms, function (form) {
            form.addEventListener('submit', function (event) {
                if (form.checkValidity() === false) {
                    event.preventDefault();
                    event.stopPropagation();
                }
                form.classList.add('was-validated');
            }, false);
        });
    }
};
```
After that we can use JavaScript Interoperability to call the above function from within the components themselves using the `ValidationJSInterop`.

Now, let us use our validation controls on action by adding a single line of markup per control to make the work done!!
```html
@page "/validation"

<h1>Form Validation</h1>
<form class="needs-validation" novalidate>
    <div class="form-group">
        <label for="txtEmail">Email address</label>
        <input type="text" class="form-control" id="txtEmail" placeholder="Email">
        <RequiredFieldValidator ControlToValidate="txtEmail" ErrorMessage="The email is required." />
    </div>
    <div class="form-group">
        <label for="txtPassword">Password</label>
        <input type="password" class="form-control" id="txtPassword" placeholder="Password">
        <RequiredFieldValidator ControlToValidate="txtPassword" ErrorMessage="The password is required." />
    </div>
    <div class="form-group">
        <label for="txtConfirmPassword">Confirm Password</label>
        <input type="password" class="form-control" id="txtConfirmPassword" placeholder="Confirm Password">
        <CompareValidator ControlToValidate="txtConfirmPassword" ControlToCompare="txtPassword" ErrorMessage="The confirm password doesn't match." />
    </div>
    <div class="form-group">
        <label for="txtAge">Age</label>
        <input type="number" class="form-control" id="txtAge" placeholder="Age">
        <RangeValidator ControlToValidate="txtAge" MinimumValue="15" MaximumValue="40" ErrorMessage="Age (15 - 40)." />
    </div>
    <div class="form-group">
        <label for="txtDomain">Domain Name</label>
        <input type="text" class="form-control" id="txtDomain" placeholder="Domain Name">
        <RegularExpressionValidator ControlToValidate="txtDomain" ValidationExpression="(.*?)[^w{3}.]([a-zA-Z0-9]([a-zA-Z0-9-]{0,65}[a-zA-Z0-9])?.)+[a-zA-Z]{2,6}" ErrorMessage="Invalid domain name." />
    </div>
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```
Finally when you run the application the validation components will look like the figure below.

![](https://raw.githubusercontent.com/hishamco/BlazorValidationControls/master/Screenshot.png)

You can download the source code for this blog post from my [BlazorValidationControls](https://github.com/hishamco/BlazorValidationControls) repository on GitHub.

Happy Coding!!