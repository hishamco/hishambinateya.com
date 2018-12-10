---
Id: 49
Posted Date: 23/10/2017
Tags:  Razor Pages,Uploading Files 
Slug: uploading-files-in-razor-pages
---
# Uploading Files in Razor Pages

In this short blog post I will talking about how to upload files in Razor Page. The file upload in the ASP.NET Core is changed quite a bit from the previous version of ASP.NET where `HttpPostedFileBase` was used, but now we can use the newly added interface `IFromFile` which has a few similar properties and methods.

In this post I will use [Damian Edwards](https://twitter.com/DamianEdwards) [RazorPagesSample](https://github.com/DamianEdwards/RazorPagesSample) for the sake of the demo, instead of creating a new one.

First of all I need to add the Photo & PhotoPath properties to the `Customer` model, to keep track of the customer photo and its path.
```csharp
public class Customer
{
    public int Id { get; set; }

    [Required, StringLength(100)]
    public string Name { get; set; }

    public string PhotePath { get; set; }

    [Required]
    [NotMapped]
    public IFormFile Phote { get; set; }**
}
```
Then we need to add the input file field in **New.cshtml** to allow the user to upload the customer's photo
```html
<h1>New Customer</h1>
<form method="post" class="form-horizontal" enctype="multipart/form-data">
    <div asp-validation-summary="All" class="text-danger"></div>
    <div class="form-group">
        <label asp-for="Customer.Name" class="col-md-2 control-label"></label>
        <div class="col-md-10">
            <input asp-for="Customer.Name" class="form-control" />
            <span asp-validation-for="Customer.Name" class="text-danger"></span>
        </div>
    </div>
    <div class="form-group">
        <label asp-for="Customer.Phote" class="col-md-2 control-label"></label>
        <div class="col-md-10">
            <input asp-for="Customer.Phote" type="file" class="form-control" style="height:auto" />
            <span asp-validation-for="Customer.Phote" class="text-danger"></span>
        </div>
    </div>
    <div class="form-group">
        <div class="col-md-offset-2 col-md-10">
            <button id="btnSave" type="submit" class="btn btn-primary">Save</button>
        </div>
    </div>
</form>
```
So the view result show be similar to the following:

![](http://www.hishambinateya.com/images/posts/63d7a1b5-0c77-4dd3-83c1-293fe4c0e729.png)

Now it's the time to add the uploaded feature by using the `FileStream` to write the `Customer.Photo` property - which hold the uploaded file - into the **Uploads** directory as the following:
```csharp
private async Task UploadPhoto()
{
    var uploadsDirectoryPath = Path.Combine(_environment.WebRootPath, "Uploads");
    var uploadedfilePath = Path.Combine(uploadsDirectoryPath, Customer.Phote.FileName);

    using (var fileStream = new FileStream(uploadedfilePath, FileMode.Create))
    {
        await Customer.Phote.CopyToAsync(fileStream);
    }
}
```
note that I'm used the option `File.Create` to override the existent file if there is. After that we need to call the `UploadPhoto()` method in `OnPost()` as the following:
```csharp
public async Task<IActionResult> OnPost()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    Customer.PhotePath = Customer.Phote.FileName;
    await UploadPhoto();
    _db.Customers.Add(Customer);
    await _db.SaveChangesAsync();

    Message = "New customer created successfully!";

    return RedirectToPage("./Index");
}
```
That's it, now it's time to tweak the **Index.cshtml** little bit to show the customer photo
```html
<h1>Customers</h1>
<p>
    <div asp-if="Model.ShowMessage" class="alert alert-success alert-dismissible" role="alert">
        <button type="button" class="close" data-dismiss="alert" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        @Model.Message
    </div>
</p>
<form method="post">
    <table class="table">
        <thead>
            <tr>
                <th>@Html.DisplayNameFor(m => m.Customers[0].Id)</th>
                <th>@Html.DisplayNameFor(m => m.Customers[0].Name)</th>
                **<th>@Html.DisplayNameFor(m => m.Customers[0].Phote)</th>**
                <th>&nbsp;</th>
            </tr>
        </thead>
        <tbody>
            <tr asp-if="Model.Customers.Count == 0">
                <td colspan="3">No customers yet.</td>
            </tr>
            @foreach (var customer in Model.Customers)
            {
            <tr>
                <td>@Html.DisplayFor(_ => customer.Id)</td>
                <td>@Html.DisplayFor(_ => customer.Name)</td>
                <td><img src="~/Uploads/@customer.PhotePath" class="img-thumbnail" width="50" /></td>
                <td>
                    <a asp-page="./Edit" asp-route-id="@customer.Id" class="btn btn-xs btn-primary">edit</a>
                    <button type="submit" asp-page-handler="delete" asp-route-id="@customer.Id" class="btn btn-xs btn-danger">delete</button>
                </td>
            </tr>
            }
        </tbody>
    </table>

    <a asp-page="./New" class="btn btn-sm btn-success">Create</a>
</form>
```
Finally you can test the uploaded feature by adding some customer and navigate to the **Index.cshml** to see something similar as the following:

![](http://www.hishambinateya.com/images/posts/3db9f2a3-8511-4c78-82bb-962088e3af09.png)

You can download the source code for this post from my [RazorPagesSample](https://github.com/hishamco/RazorPagesSample) repository on GitHub.

Happy Coding ...