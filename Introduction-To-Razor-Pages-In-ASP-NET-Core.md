# Introduction to Razor Pages in ASP.NET Core

Code: ``Sandbox\RazorPagesContacts``.

Razor Pages can make coding page-focused scenarios easier and more productive than using controllers and views.

If you're looking for a tutorial that uses the Model-View-Controller approach, see [Get started with ASP.NET Core MVC](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc?view=aspnetcore-6.0).

This document provides an introduction to Razor Pages. It's not a step by step tutorial.

## Create a Razor Pages project

Razor Pages is enabled in ``Program.cs``:

```csharp
    var builder = WebApplication.CreateBuilder(args);

    builder.Services.AddRazorPages();

    var app = builder.Build();

    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthorization();

    app.MapRazorPages();

    app.Run();
```

In the preceding code:

* [AddRazorPages](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addrazorpages) adds services for Razor Pages to the app.
* [MapRazorPages](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.builder.razorpagesendpointroutebuilderextensions.maprazorpages) adds endpoints for Razor Pages to the [IEndpointRouteBuilder](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.routing.iendpointroutebuilder).

Consider a basic page:

```csharp
    @page

    <h1>Hello, world!</h1>
    <h2>The time on the server is @DateTime.Now</h2>
```

The preceding code looks a lot like a [Razor view file](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/adding-view?view=aspnetcore-6.0) used in an ASP.NET Core app with controllers and views. What makes it different is the [@page](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/razor?view=aspnetcore-6.0#page) directive. @page makes the file into an MVC action, which means that it handles requests directly, without going through a controller. @page must be the first Razor directive on a page. @page affects the behavior of other [Razor](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/razor?view=aspnetcore-6.0) constructs. Razor Pages file names have a ``.cshtml`` suffix.

A similar page, using a PageModel class, is shown in the following two files. The ``Pages/Index2.cshtml`` file:

```html
    @page
    @using RazorPagesIntro.Pages
    @model Index2Model

    <h2>Separate page model</h2>
    <p>
        @Model.Message
    </p>
```

The ``Pages/Index2.cshtml.cs`` page model:

```csharp
    using Microsoft.AspNetCore.Mvc.RazorPages;
    using Microsoft.Extensions.Logging;
    using System;

    namespace RazorPagesIntro.Pages
    {
        public class Index2Model : PageModel
        {
            public string Message { get; private set; } = "PageModel in C#";

            public void OnGet()
            {
                Message += $" Server time is { DateTime.Now }";
            }
        }
    }
```

``@Model`` in the Razor page allows us to use data from the ``PageModel``.

By convention, the ``PageModel`` class file has the same name as the Razor Page file with ``.cs`` appended. For example, the previous Razor Page is *Pages/Index2.cshtml*. The file containing the ``PageModel`` class is named *Pages/Index2.cshtml.cs*.

The associations of URL paths to pages are determined by the page's location in the file system. The following table shows a Razor Page path and the matching URL:

| File name and path	      | matching URL            |
| ----------------------------| ------------------------|
| /Pages/Index.cshtml	      | / or /Index             | 
| /Pages/Contact.cshtml	      | /Contact                |
| /Pages/Store/Contact.cshtml |	/Store/Contact          |
| /Pages/Store/Index.cshtml	  | /Store or /Store/Index  |

**Notes:**

* The runtime looks for Razor Pages files in the *Pages* folder by default.
* ``Index`` is the default page when a URL doesn't include a page.

## Write a basic form

Razor Pages is designed to make common patterns used with web browsers easy to implement when building an app. Model binding, [Tag Helpers](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-6.0), and HTML helpers work with the properties defined in a Razor Page class. Consider a page that implements a basic "contact us" form for the ``Contact`` model:

For the samples in this document, the ``DbContext`` is initialized in the *Program.cs* file.

The in memory database requires the ``Microsoft.EntityFrameworkCore.InMemory`` NuGet package.

It also requires ``using Microsoft.EntityFrameworkCore;`` in the ``PageModel`` class.

Add the ``DbContext`` to ``Program.cs``.

```csharp
    builder.Services.AddRazorPages();

    builder.Services.AddDbContext<CustomerDbContext>(options =>
        options.UseInMemoryDatabase("name"));

    var app = builder.Build();
```

Create a ``Models/Customer.cs`` model.

```csharp
    using System.ComponentModel.DataAnnotations;

    namespace RazorPagesContacts.Models
    {
        public class Customer
        {
            public int Id { get; set; }

            [Required, StringLength(10)]
            public string? Name { get; set; }
        }
    }
```

The ``Data/CustomerDbContext.cs`` db context:

```csharp
    using Microsoft.EntityFrameworkCore;

    namespace RazorPagesContacts.Data
    {
        public class CustomerDbContext : DbContext
        {
            public CustomerDbContext (DbContextOptions<CustomerDbContext> options)
                : base(options)
            {
            }

            public DbSet<RazorPagesContacts.Models.Customer> Customer => Set<RazorPagesContacts.Models.Customer>();
        }
    }
```

The ``Pages/Customers/Create.cshtml`` view file:

```csharp
    @page
    @model RazorPagesContacts.Pages.Customers.CreateModel
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

    <p>Enter a customer name:</p>

    <form method="post">
        Name:
        <input asp-for="Customer!.Name" />
        <input type="submit" />
    </form>
```

The ``Pages/Customers/Create.cshtml.cs`` page model:

```csharp
    public class CreateModel : PageModel
    {
        private readonly Data.CustomerDbContext _context;

        public CreateModel(Data.CustomerDbContext context)
        {
            _context = context;
        }

        public IActionResult OnGet()
        {
            return Page();
        }

        [BindProperty]
        public Customer? Customer { get; set; }

        public async Task<IActionResult> OnPostAsync()
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            if (Customer != null) _context.Customer.Add(Customer);
            await _context.SaveChangesAsync();

            return RedirectToPage("./Index");
        }
    }
```

By convention, the ``PageModel`` class is called ``<PageName>Model`` and is in the same namespace as the page.

The ``PageModel`` class allows separation of the logic of a page from its presentation. It defines page handlers for requests sent to the page and the data used to render the page. This separation allows:

* Managing of page dependencies through [dependency injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0).
* [Unit testing](https://docs.microsoft.com/en-us/aspnet/core/test/razor-pages-tests?view=aspnetcore-6.0)

The page has an ``OnPostAsync`` handler method, which runs on ``POST`` requests (when a user posts the form). Handler methods for any HTTP verb can be added. The most common handlers are:

* ``OnGet`` to initialize state needed for the page. In the preceding code, the ``OnGet`` method displays the ``CreateModel.cshtml`` Razor Page.
* ``OnPost`` to handle form submissions.

The ``Async`` naming suffix is optional but is often used by convention for asynchronous functions. The preceding code is typical for Razor Pages.

If you're familiar with ASP.NET apps using controllers and views:

* The ``OnPostAsync`` code in the preceding example looks similar to typical controller code.
* Most of the MVC primitives like model binding, [validation](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-6.0), and action results work the same with Controllers and Razor Pages.

The previous ``OnPostAsync`` method:

```csharp
    [BindProperty]
    public Customer? Customer { get; set; }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }

        if (Customer != null) _context.Customer.Add(Customer);
        await _context.SaveChangesAsync();

        return RedirectToPage("./Index");
    }
```

The basic flow of ``OnPostAsync``:

Check for validation errors.

* If there are no errors, save the data and redirect.
* If there are errors, show the page again with validation messages. In many cases, validation errors would be detected on the client, and never submitted to the server.

The ``Pages/Customers/Create.cshtml`` view file:

```csharp
    @page
    @model RazorPagesContacts.Pages.Customers.CreateModel
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

    <p>Enter a customer name:</p>

    <form method="post">
        Name:
        <input asp-for="Customer!.Name" />
        <input type="submit" />
    </form>
```

The rendered HTML from ``Pages/Customers/Create.cshtml``:

```html
    <p>Enter a customer name:</p>

    <form method="post">
        Name:
        <input type="text" data-val="true"
               data-val-length="The field Name must be a string with a maximum length of 10."
               data-val-length-max="10" data-val-required="The Name field is required."
               id="Customer_Name" maxlength="10" name="Customer.Name" value="" />
        <input type="submit" />
        <input name="__RequestVerificationToken" type="hidden"
               value="<Antiforgery token here>" />
    </form>
```

In the previous code, posting the form:

* With valid data:

  * The ``OnPostAsync`` handler method calls the [RedirectToPage](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.redirecttopage) helper method. RedirectToPage returns an instance of [RedirectToPageResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.redirecttopageresult). ``RedirectToPage``:
  * Is an action result.
  * Is similar to ``RedirectToAction`` or ``RedirectToRoute`` (used in controllers and views).
  * Is customized for pages. In the preceding sample, it redirects to the root Index page (``/Index``). ``RedirectToPage`` is detailed in the [URL generation for Pages](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0&tabs=visual-studio#url_gen) section.

* With validation errors that are passed to the server:

  * The ``OnPostAsync`` handler method calls the [Page](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagebase.page) helper method. ``Page`` returns an instance of ``PageResult``. Returning Page is similar to how actions in controllers return ``View``. [PageResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pageresult) is the default return type for a handler method. A handler method that returns ``void`` renders the page.
  * In the preceding example, posting the form with no value results in [ModelState.IsValid](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelstatedictionary.isvalid#microsoft-aspnetcore-mvc-modelbinding-modelstatedictionary-isvalid) returning false. In this sample, no validation  errors are displayed on the client. Validation error handing is covered later in this document.

```csharp
    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }
        ...
    }
```

* With validation errors detected by client side validation:

  * Data is **not** posted to the server.
  * Client-side validation is explained later in this document.

The ``Customer`` property uses [[BindProperty](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute)] attribute to opt in to model binding:

```csharp
    public IActionResult OnGet()
    {
        return Page();
    }

    [BindProperty]
    public Customer? Customer { get; set; }

    public async Task<IActionResult> OnPostAsync()
    ...
```

``[BindProperty]`` should **not** be used on models containing properties that should not be changed by the client. For more information, see [Overposting](https://docs.microsoft.com/en-us/aspnet/core/data/ef-rp/crud?view=aspnetcore-6.0#overposting).

Razor Pages, by default, bind properties only with non-``GET`` verbs. Binding to properties removes the need to writing code to convert HTTP data to the model type. Binding reduces code by using the same property to render form fields (``<input asp-for="Customer.Name">``) and accept the input.

> **Warning**
> 
> For security reasons, you must opt in to binding ``GET`` request data to page model properties. Verify user input before mapping it to > properties. Opting into ``GET`` binding is useful when addressing scenarios that rely on query string or route values.
> 
> To bind a property on ``GET`` requests, set the [``BindProperty``] attribute's ``SupportsGet`` property to ``true``:
> 
> ``[BindProperty(SupportsGet = true)]``

Reviewing the ``Pages/Customers/Create.cshtml`` view file:

```csharp
    @page
    @model RazorPagesContacts.Pages.Customers.CreateModel
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

    <p>Enter a customer name:</p>

    <form method="post">
        Name:
        <input asp-for="Customer!.Name" />
        <input type="submit" />
    </form>
```

* In the preceding code, the [input tag helper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-6.0#the-input-tag-helper) ``<input asp-for="Customer.Name" />`` binds the HTML ``<input>`` element to the ``Customer.Name`` model expression.
* [@addTagHelper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-6.0#addtaghelper-makes-tag-helpers-available) makes Tag Helpers available.

### The home page

#### Pages/Customers/Index.cshtml

```csharp
    @page
    @model RazorPagesContacts.Pages.Customers.IndexModel
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

    <h1>Contacts home page</h1>
    <form method="post">
        <table class="table">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th></th>
                </tr>
            </thead>
            <tbody>
            @if (Model.Customers != null)
            {
                foreach (var contact in Model.Customers)
                {
                    <tr>
                        <td> @contact.Id </td>
                        <td>@contact.Name</td>
                        <td>
                            <!-- <snippet_Edit> -->
                            <a asp-page="./Edit" asp-route-id="@contact.Id">Edit</a> |
                            <!-- </snippet_Edit> -->
                            <!-- <snippet_Delete> -->
                            <button type="submit" asp-page-handler="delete" asp-route-id="@contact.Id">delete</button>
                            <!-- </snippet_Delete> -->
                        </td>
                    </tr>
                }
            }
            </tbody>
        </table>
        <a asp-page="Create">Create New</a>
    </form>
```

#### Pages/Customers/Index.cshtml.cs

```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.AspNetCore.Mvc.RazorPages;
    using Microsoft.EntityFrameworkCore;
    using RazorPagesContacts.Models;

    namespace RazorPagesContacts.Pages.Customers
    {
        public class IndexModel : PageModel
        {
            private readonly Data.CustomerDbContext _context;

            public IndexModel(Data.CustomerDbContext context)
            {
                _context = context;
            }

            public IList<Customer>? Customers { get; set; }

            public async Task OnGetAsync()
            {
                Customers = await _context.Customer.ToListAsync();
            }

            public async Task<IActionResult> OnPostDeleteAsync(int id)
            {
                var contact = await _context.Customer.FindAsync(id);

                if (contact != null)
                {
                    _context.Customer.Remove(contact);
                    await _context.SaveChangesAsync();
                }

                return RedirectToPage();
            }
        }
    }
```

The ``Index.cshtml`` file contains the following markup:

```csharp
    <a asp-page="./Edit" asp-route-id="@contact.Id">Edit</a> |
```

The ``<a /a>`` [Anchor Tag Helper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/anchor-tag-helper?view=aspnetcore-6.0) used the ``asp-route-{value}`` attribute to generate a link to the ``Edit`` page. The link contains route data with the contact ID. For example, ``https://localhost:7241/Edit/1``. [Tag Helpers](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-6.0) enable server-side code to participate in creating and rendering HTML elements in Razor files.

The ``Index.cshtml`` file contains markup to create a delete button for each customer contact:

```csharp
    <button type="submit" asp-page-handler="delete" asp-route-id="@contact.Id">delete</button>
```

The rendered HTML:

```csharp
    <button type="submit" formaction="/Customers?id=1&amp;handler=delete">delete</button>
```

When the delete button is rendered in HTML, its [formaction](https://developer.mozilla.org/docs/Web/HTML/Element/button#attr-formaction) includes parameters for:

* The customer contact ID, specified by the ``asp-route-id`` attribute.
* The handler, specified by the ``asp-page-handler`` attribute.

When the button is selected, a form POST request is sent to the server. By convention, the name of the handler method is selected based on the value of the handler parameter according to the scheme OnPost[handler]Async.

Because the handler is delete in this example, the ``OnPostDeleteAsync`` handler method is used to process the ``POST`` request. If the ``asp-page-handler`` is set to a different value, such as ``remove``, a handler method with the name ``OnPostRemoveAsync`` is selected.

```csharp
    public async Task<IActionResult> OnPostDeleteAsync(int id)
    {
        var contact = await _context.Customer.FindAsync(id);

        if (contact != null)
        {
            _context.Customer.Remove(contact);

            await _context.SaveChangesAsync();
        }

        return RedirectToPage();
    }
```

The ``OnPostDeleteAsync`` method:

* Gets the ``id`` from the query string.
* Queries the database for the customer contact with ``FindAsync``.
* If the customer contact is found, it's removed and the database is updated.
* Calls [RedirectToPage](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.redirecttopage) to redirect to the root Index page (``/Index``).

### The Edit.cshtml file

```csharp
@page "{id:int}"
@model RazorPagesContacts.Pages.Customers.EditModel

@{
    ViewData["Title"] = "Edit";
}

<h1>Edit</h1>

<h4>Customer</h4>
<hr />
<div class="row">
    <div class="col-md-4">
        <form method="post">
            <div asp-validation-summary="ModelOnly" class="text-danger"></div>
            <input type="hidden" asp-for="Customer!.Id" />
            <div class="form-group">
                <label asp-for="Customer!.Name" class="control-label"></label>
                <input asp-for="Customer!.Name" class="form-control" />
                <span asp-validation-for="Customer!.Name" class="text-danger"></span>
            </div>
            <div class="form-group">
                <input type="submit" value="Save" class="btn btn-primary" />
            </div>
        </form>
    </div>
</div>

<div>
    <a asp-page="./Index">Back to List</a>
</div>

@section Scripts {
    @{await Html.RenderPartialAsync("_ValidationScriptsPartial");}
}
```

The first line contains the ``@page "{id:int}"`` directive. The routing constraint ``"{id:int}"`` tells the page to accept requests to the page that contain ``int`` route data. If a request to the page doesn't contain route data that can be converted to an ``int``, the runtime returns an HTTP 404 (not found) error. To make the ID optional, append ``?`` to the route constraint:

```csharp
    @page "{id:int?}"
```

#### Pages/Customers/Edit.cshtml.cs

```csharp
    public class EditModel : PageModel
    {
        private readonly RazorPagesContacts.Data.CustomerDbContext _context;

        public EditModel(RazorPagesContacts.Data.CustomerDbContext context)
        {
            _context = context;
        }

        [BindProperty]
        public Customer? Customer { get; set; }

        public async Task<IActionResult> OnGetAsync(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            Customer = await _context.Customer.FirstOrDefaultAsync(m => m.Id == id);

            if (Customer == null)
            {
                return NotFound();
            }
            return Page();
        }

        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see https://aka.ms/RazorPagesCRUD.
        public async Task<IActionResult> OnPostAsync()
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            if (Customer != null)
            {
                _context.Attach(Customer).State = EntityState.Modified;

                try
                {
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!CustomerExists(Customer.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
            }

            return RedirectToPage("./Index");
        }

        private bool CustomerExists(int id)
        {
            return _context.Customer.Any(e => e.Id == id);
        }
    }
```

### Validation

Validation rules:

* Are declaratively specified in the model class.
* Are enforced everywhere in the app.
* The [System.ComponentModel.DataAnnotations](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations) namespace provides a set of built-in validation attributes that are applied declaratively to a class or property. DataAnnotations also contains formatting attributes like [[DataType]](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.datatypeattribute) that help with formatting and don't provide any validation.

Consider the ``Customer`` model:

```csharp
    using System.ComponentModel.DataAnnotations;

    namespace RazorPagesContacts.Models
    {
        public class Customer
        {
            public int Id { get; set; }

            [Required, StringLength(10)]
            public string? Name { get; set; }
        }
    }
```

Using the following ``Create.cshtml`` view file:

```csharp
    @page
    @model RazorPagesContacts.Pages.Customers.CreateModel
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

    <p>Validation: customer name:</p>

    <form method="post">
        <div asp-validation-summary="ModelOnly"></div>
        <span asp-validation-for="Customer.Name"></span>
        Name:
        <input asp-for="Customer.Name" />
        <input type="submit" />
    </form>

    <script src="~/lib/jquery/dist/jquery.js"></script>
    <script src="~/lib/jquery-validation/dist/jquery.validate.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"></script>
```

The preceding code:

Includes jQuery and jQuery validation scripts.

Uses the ``<div />`` and ``<span />`` [Tag Helpers](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-6.0) to enable:

* Client-side validation.
* Validation error rendering.

Generates the following HTML:

```html
    <p>Enter a customer name:</p>

    <form method="post">
        Name:
        <input type="text" data-val="true"
               data-val-length="The field Name must be a string with a maximum length of 10."
               data-val-length-max="10" data-val-required="The Name field is required."
               id="Customer_Name" maxlength="10" name="Customer.Name" value="" />
        <input type="submit" />
        <input name="__RequestVerificationToken" type="hidden"
               value="<Antiforgery token here>" />
    </form>

    <script src="/lib/jquery/dist/jquery.js"></script>
    <script src="/lib/jquery-validation/dist/jquery.validate.js"></script>
    <script src="/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"></script>
```

Posting the Create form without a name value displays the error message "The Name field is required." on the form. If JavaScript is enabled on the client, the browser displays the error without posting to the server.

The ``[StringLength(10)]`` attribute generates ``data-val-length-max="10"`` on the rendered HTML. ``data-val-length-max`` prevents browsers from entering more than the maximum length specified. If a tool such as [Fiddler](https://www.telerik.com/fiddler) is used to edit and replay the post:

* With the name longer than 10.
* The error message "The field Name must be a string with a maximum length of 10." is returned.

## CSS isolation

Isolate CSS styles to individual pages, views, and components to reduce or avoid:

* Dependencies on global styles that can be challenging to maintain.
* Style conflicts in nested content.

To add a *scoped CSS file* for a page or view, place the CSS styles in a companion ``.cshtml.css`` file matching the name of the ``.cshtml`` file. In the following example, a ``Index.cshtml.css`` file supplies CSS styles that are only applied to the ``Index.cshtml`` page or view.

``Pages/Index.cshtml.css``:

```css
    h1 {
        color: red;
    }
```

CSS isolation occurs at build time. The framework rewrites CSS selectors to match markup rendered by the app's pages or views. The rewritten CSS styles are bundled and produced as a static asset, ``{APP ASSEMBLY}.styles.css``. The placeholder ``{APP ASSEMBLY}`` is the assembly name of the project. A link to the bundled CSS styles is placed in the app's layout.

In the ``<head>`` content of the app's ``Pages/Shared/_Layout.cshtml``, add or confirm the presence of the link to the bundled CSS styles:

```html
    <link rel="stylesheet" href="~/{APP ASSEMBLY}.styles.css" />
```

The styles defined in a scoped CSS file are only applied to the rendered output of the matching file. In the preceding example, any ``h1`` CSS declarations defined elsewhere in the app don't conflict with the ``Index``'s heading style. CSS style cascading and inheritance rules remain in effect for scoped CSS files. For example, styles applied directly to an ``<h1>`` element in the ``Index.cshtml`` file override the scoped CSS file's styles in``Index.cshtml.css``.

Within the bundled CSS file, each page, view, or Razor component is associated with a scope identifier in the format ``b-{STRING}``, where the ``{STRING}`` placeholder is a ten-character string generated by the framework. The following example provides the style for the preceding ``<h1>`` element in the Index page of a Razor Pages app:

```css
    /* /Pages/Index.cshtml.rz.scp.css */
    h1[b-3xxtam6d07] {
        color: red;
    }
```

In the Index page where the CSS style is applied from the bundled file, the scope identifier is appended as an HTML attribute:

```html
    <h1 b-3xxtam6d07>
```

The identifier is unique to an app. At build time, a project bundle is created with the convention ``{STATIC WEB ASSETS BASE PATH}/Project.lib.scp.css``, where the placeholder ``{STATIC WEB ASSETS BASE PATH}`` is the static web assets base path.

If other projects are utilized, such as NuGet packages or [Razor class libraries](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/ui-class?view=aspnetcore-6.0), the bundled file:

* References the styles using CSS imports.
* Isn't published as a static web asset of the app that consumes the styles.

### XSRF/CSRF and Razor Pages

Razor Pages are protected by [Antiforgery validation](https://docs.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-6.0). The [FormTagHelper](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/working-with-forms?view=aspnetcore-6.0#the-form-tag-helper) injects antiforgery tokens into HTML form elements.

## Using Layouts, partials, templates, and Tag Helpers with Razor Pages

Pages work with all the capabilities of the Razor view engine. Layouts, partials, templates, Tag Helpers, ``_ViewStart.cshtml``, and ``_ViewImports.cshtml`` work in the same way they do for conventional Razor views.

Let's declutter this page by taking advantage of some of those capabilities.

Add a [layout page](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/layout?view=aspnetcore-6.0) to ``Pages/Shared/_Layout.cshtml``:

```csharp
    <!DOCTYPE html>
    <html>
    <head>
        <title>RP Sample</title>
        <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
    </head>
    <body>
        <a asp-page="/Index">Home</a>
        <a asp-page="/Customers/Create">Create</a>
        <a asp-page="/Customers/Index">Customers</a> <br />

        @RenderBody()
        
        <script src="~/lib/jquery/dist/jquery.js"></script>
        <script src="~/lib/jquery-validation/dist/jquery.validate.js"></script>
        <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"></script>
    </body>
    </html>
```

The Layout:

Controls the layout of each page (unless the page opts out of layout).
Imports HTML structures such as JavaScript and stylesheets.
The contents of the Razor page are rendered where ``@RenderBody()`` is called.
For more information, see l[ayout page](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/layout?view=aspnetcore-6.0).

```csharp
    @{
        Layout = "_Layout";
    }
```

The layout is in the *Pages/Shared* folder. Pages look for other views (layouts, templates, partials) hierarchically, starting in the same folder as the current page. A layout in the *Pages/Shared* folder can be used from any Razor page under the *Pages* folder.

The layout file should go in the *Pages/Shared* folder.

We recommend you not put the layout file in the Views/Shared folder. Views/Shared is an MVC views pattern. Razor Pages are meant to rely on folder hierarchy, not path conventions.

View search from a Razor Page includes the *Pages* folder. The layouts, templates, and partials used with MVC controllers and conventional Razor views *just* work.

Add a ``Pages/_ViewImports.cshtml`` file:

```csharp
    @namespace RazorPagesContacts.Pages
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

``@namespace`` is explained later in the tutorial. The ``@addTagHelper`` directive brings in the built-in [Tag Helpers](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/?view=aspnetcore-6.0) to all the pages in the Pages folder.

The ``@namespace`` directive set on a page:

```csharp
    @page
    @namespace RazorPagesIntro.Pages.Customers

    @model NameSpaceModel

    <h2>Name space</h2>
    <p>
        @Model.Message
    </p>
```

The ``@namespace`` directive sets the namespace for the page. The ``@model`` directive doesn't need to include the namespace.

When the ``@namespace`` directive is contained in ``_ViewImports.cshtml``, the specified namespace supplies the prefix for the generated namespace in the Page that imports the ``@namespace`` directive. The rest of the generated namespace (the suffix portion) is the dot-separated relative path between the folder containing ``_ViewImports.cshtml`` and the folder containing the page.

For example, the ``PageModel`` class ``Pages/Customers/Edit.cshtml.cs`` explicitly sets the namespace:

```csharp
    namespace RazorPagesContacts.Pages
    {
        public class EditModel : PageModel
        {
            private readonly AppDbContext _db;

            public EditModel(AppDbContext db)
            {
                _db = db;
            }

            // Code removed for brevity.
        }
    }
```

The ``Pages/_ViewImports.cshtml`` file sets the following namespace:

```csharp
    @namespace RazorPagesContacts.Pages
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
```

The generated namespace for the ``Pages/Customers/Edit.cshtml`` Razor Page is the same as the ``PageModel`` class.

``@namespace`` *also works with conventional Razor views*.

Consider the ``Pages/Customers/Create.cshtml`` view file:

```csharp
    @page
    @model RazorPagesContacts.Pages.Customers.CreateModel
    @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

    <p>Validation: customer name:</p>

    <form method="post">
        <div asp-validation-summary="ModelOnly"></div>
        <span asp-validation-for="Customer.Name"></span>
        Name:
        <input asp-for="Customer.Name" />
        <input type="submit" />
    </form>

    <script src="~/lib/jquery/dist/jquery.js"></script>
    <script src="~/lib/jquery-validation/dist/jquery.validate.js"></script>
    <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"></script>
```

The updated ``Pages/Customers/Create.cshtml`` view file with ``_ViewImports.cshtml`` and the preceding layout file:

```csharp
    @page
    @model CreateModel

    <p>Enter a customer name:</p>

    <form method="post">
        Name:
        <input asp-for="Customer.Name" />
        <input type="submit" />
    </form>
```

In the preceding code, the ``_ViewImports.cshtml`` imported the namespace and ``Tag Helpers``. The layout file imported the JavaScript files.

The [Razor Pages starter project](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-6.0&tabs=visual-studio#rpvs17) contains the ``Pages/_ValidationScriptsPartial.cshtml``, which hooks up client-side validation.

For more information on partial views, see [Partial views in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/partial?view=aspnetcore-6.0).

## URL generation for Pages

The ``Create`` page, shown previously, uses ``RedirectToPage``:

```csharp
    public class CreateModel : PageModel
    {
        private readonly CustomerDbContext _context;

        public CreateModel(CustomerDbContext context)
        {
            _context = context;
        }

        public IActionResult OnGet()
        {
            return Page();
        }

        [BindProperty]
        public Customer Customer { get; set; }

        public async Task<IActionResult> OnPostAsync()
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            _context.Customers.Add(Customer);
            await _context.SaveChangesAsync();

            return RedirectToPage("./Index");
        }
    }
```

The app has the following file/folder structure:

* /Pages

  * Index.cshtml

  * Privacy.cshtml

  * /Customers

    * Create.cshtml

    * Edit.cshtml

    * Index.cshtml

The ``Pages/Customers/Create.cshtml`` and ``Pages/Customers/Edit.cshtml`` pages redirect to ``Pages/Customers/Index.cshtml`` after success. The string ``./Index`` is a relative page name used to access the preceding page. It is used to generate URLs to the ``Pages/Customers/Index.cshtml`` page. For example:

* ``Url.Page("./Index", ...)``
* ``<a asp-page="./Index">Customers Index Page</a>``
* ``RedirectToPage("./Index")``

The page name is the path to the page from the root */Pages* folder including a leading ``/`` (for example, ``/Index``). The preceding URL generation samples offer enhanced options and functional capabilities over hard-coding a URL. URL generation uses routing and can generate and encode parameters according to how the route is defined in the destination path.

URL generation for pages supports relative names. The following table shows which Index page is selected using different ``RedirectToPage`` parameters in ``Pages/Customers/Create.cshtml``.

| RedirectToPage(x)	         | Page                     |
|----------------------------|--------------------------|
| RedirectToPage("/Index")   | *Pages/Index*            |     
| RedirectToPage("./Index"); | *Pages/Customers/Index*  |
| RedirectToPage("../Index") | *Pages/Index*            |
| RedirectToPage("Index")	 | *Pages/Customers/*       |

``RedirectToPage("Index")``, ``RedirectToPage("./Index")``, and ``RedirectToPage("../Index")`` are *relative names*. The ``RedirectToPage`` parameter is combined with the path of the current page to compute the name of the destination page.

Relative name linking is useful when building sites with a complex structure. When relative names are used to link between pages in a folder:

* Renaming a folder doesn't break the relative links.
* Links are not broken because they don't include the folder name.

To redirect to a page in a different [Area](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/areas?view=aspnetcore-6.0), specify the area:

```csharp
    RedirectToPage("/Index", new { area = "Services" });
```

For more information, see [Areas in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/areas?view=aspnetcore-6.0) and [Razor Pages route and app conventions in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/razor-pages-conventions?view=aspnetcore-6.0).

## ViewData attribute
