# Get started with Razor Pages in ASP.NET Core

## Create an ASP.Net Core WebApp

In this process there are two checkbox options.

Configure for https: - Check this.

Do not use top-level statements: - Check this.

This option will allow you to use a Namespace and Class structure in your C# files.

This file structure is created.

![solution-structure](assets/images/solution-structure.jpg "solution-structure")

You can run the application with ``CTRL-F5``.

Visual Studio:

Runs the app, which launches the [Kestrel server](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-6.0).

Launches the default browser at <https://localhost:7184/>, which displays the apps UI.

### Pages folder

Contains Razor pages and supporting files. Each Razor page is a pair of files:

A .cshtml file that has HTML markup with C# code using Razor syntax.

A .cshtml.cs file that has C# code that handles page events.

Supporting files have names that begin with an underscore. For example, the *_Layout.cshtml* file configures UI elements common to all  pages. This file sets up the navigation menu at the top of the page and the copyright notice at the bottom of the page. For more information, see [Layout in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0).

### wwwroot folder

Contains static assets, like HTML files, JavaScript files, and CSS files. For more information, see [Static files in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/static-files?view=aspnetcore-6.0).

### appsettings.json

Contains configuration data, like connection strings. For more information, see [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0).

### Program.cs

Contains the following code:

```csharp
    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddRazorPages();

    var app = builder.Build();

    // Configure the HTTP request pipeline.
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error");
        // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }

    app.UseHttpsRedirection();

    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthorization();

    app.MapRazorPages();

    app.Run();
```

The following lines of code in this file create a ``WebApplicationBuilder`` with preconfigured defaults, add Razor Pages support to the [Dependency Injection (DI) container](<https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0>), and build the app:

```csharp
    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddRazorPages();

    var app = builder.Build();
```

The developer exception page is enabled by default and provides helpful information on exceptions. Production apps should not be run in development mode because the developer exception page can leak sensitive information.

The following code sets the exception endpoint to /Error and enables [HTTP Strict Transport Security Protocol (HSTS)](https://docs.microsoft.com/en-us/aspnet/core/security/enforcing-ssl?view=aspnetcore-6.0#http-strict-transport-security-protocol-hsts) when the app is not running in development mode:

```csharp
    // Configure the HTTP request pipeline.
    if (!app.Environment.IsDevelopment())
    {
        app.UseExceptionHandler("/Error");
        // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }
```

For example, the preceding code runs when the app is in production or test mode. For more information, see [Use multiple environments in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments?view=aspnetcore-6.0).

The following code enables various Middleware:

``app.UseHttpsRedirection();`` : Redirects HTTP requests to HTTPS.

``app.UseStaticFiles();`` : Enables static files, such as HTML, CSS, images, and JavaScript to be served. For more information, see Static files in ASP.NET Core.

``app.UseRouting();`` : Adds route matching to the middleware pipeline. For more information, see Routing in ASP.NET Core

``app.MapRazorPages();Z`` : Configures endpoint routing for Razor Pages.

``app.UseAuthorization();`` : Authorizes a user to access secure resources. This app doesn't use authorization, therefore this line could be removed.

``app.Run();`` : Runs the app.

## Add a model to a Razor Pages app in ASP.NET Core

In this tutorial, classes are added for managing movies in a database. The app's model classes use [Entity Framework Core (EF Core)](https://docs.microsoft.com/en-us/ef/core) to work with the database. EF Core is an object-relational mapper (O/RM) that simplifies data access. You write the model classes first, and EF Core creates the database.

The model classes are known as **POCO** classes (from "Plain-Old CLR Objects") because they don't have a dependency on EF Core. They define the properties of the data that are stored in the database.

### Add a data model

Create a new folder on the root of your project named **Models** and add a new class named **Movie**.

```csharp
    using System.ComponentModel.DataAnnotations;

    namespace RazorPagesMovie.Models
    {
        public class Movie
        {
            public int ID { get; set; }
            public string Title { get; set; } = string.Empty;

            [DataType(DataType.Date)]
            public DateTime ReleaseDate { get; set; }
            public string Genre { get; set; } = string.Empty;
            public decimal Price { get; set; }
        }
    }
```

We are using data annotations for the **ReleaseDate** property to set the data type. By adding this the user doesn't have to enter Time information in the date field and only the date is shown, not the time information.

Build the project to make sure there are no errors.

### Scaffold the movie model

In this section, the movie model is scaffolded. That is, the scaffolding tool produces pages for Create, Read, Update, and Delete (CRUD) operations for the movie model.

1. Open NuGet package manager and browse to **Microsoft.EntityFrameworkCore.Design** and add it to your project.
2. Create the **Pages/Movies** folder.
3. Right-click on the **Pages/Movies** folder > **Add > New Scaffolded Item**.
4. In the Add New Scaffold dialog, select Razor Pages using Entity Framework (CRUD) > **Add**.

![db-context](assets/images/db-context.jpg "db-context")

**Model class**: Movie (RazorPagesMovie.Models)
**Data context class**: RazorPagesMovie.Data.RazorPagesMovieContext

The ``appsettings.json`` file is updated with the connection string used to connect to a local database.

### Files created and updated

The scaffold process creates the following files:

Pages/Movies: Create, Delete, Details, Edit, and Index.

Data/RazorPagesMovieContext.cs

The scaffold process updates the **Program.cs** file.

#### using statements

```csharp
    using Microsoft.EntityFrameworkCore;
    using Microsoft.Extensions.DependencyInjection;
    using RazorPagesMovie.Data;
```

#### db context

```csharp
    var builder = WebApplication.CreateBuilder(args);
        
        // Adds this db context
        builder.Services.AddDbContext<RazorPagesMovieContext>(options =>
            options.UseSqlServer(builder.Configuration.GetConnectionString("RazorPagesMovieContext") ?? throw newInvalidOperationException("Connection string 'RazorPagesMovieContext' not found.")));
```

## Create the initial database schema using EF's migration feature

The migrations feature in Entity Framework Core provides a way to:

* Create the initial database schema.
* Incrementally update the database schema to keep it in sync with the app's data model. Existing data in the database is preserved.

Open the NuGet Package Manager console 

In the PMC, enter the following commands:

```powershell
    Add-Migration InitialCreate
    Update-Database
```

**Note:** the ``RazorPagesMovie.Data.mdf`` database is stored in my *Users\alanr* folder.

The preceding commands install the [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/get-started/overview/install#get-the-entity-framework-core-tools) tools and run the migrations command to generate code that creates the initial database schema.

The following warning is displayed, which is addressed in a later step:

> No type was specified for the decimal column 'Price' on entity type 'Movie'. This will cause values to be silently truncated if they do not fit in the default precision and scale. Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'.

The ``migrations`` command generates code to create the initial database schema. The schema is based on the model specified in ``DbContext``. The ``InitialCreate`` argument is used to name the migrations. Any name can be used, but by convention a name is selected that describes the migration.

The ``update`` command runs the ``Up`` method in migrations that have not been applied. In this case, ``update`` runs the ``Up`` method in the ``Migrations/<time-stamp>_InitialCreate.cs`` file, which creates the database.

### Examine the context registered with dependency injection

ASP.NET Core is built with dependency injection. Services, such as the EF Core database context, are registered with [dependency injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0) during application startup. Components that require these services (such as Razor Pages) are provided via constructor parameters. The constructor code that gets a database context instance is shown later in the tutorial.

The scaffolding tool automatically created a database context and registered it with the dependency injection container. The following highlighted code is added to the ``Program.cs`` file by the scaffolder:

```csharp
    builder.Services.AddDbContext<RazorPagesMovieContext>(options =>
            options.UseSqlServer(builder.Configuration.GetConnectionString("RazorPagesMovieContext") ?? throw newInvalidOperationException("Connection string 'RazorPagesMovieContext' not found.")));
```

The data context ``RazorPagesMovieContext``:

* Derives from [Microsoft.EntityFrameworkCore.DbContext](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext).
* Specifies which entities are included in the data model.
* Coordinates EF Core functionality, such as Create, Read, Update and Delete, for the ``Movie`` model.

#### Data/RazorPagesMovieContext.cs

```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading.Tasks;
    using Microsoft.EntityFrameworkCore;
    using RazorPagesMovie.Models;

    namespace RazorPagesMovie.Data
    {
        public class RazorPagesMovieContext : DbContext
        {
            public RazorPagesMovieContext (DbContextOptions<RazorPagesMovieContext> options)
                : base(options)
            {
            }

            public DbSet<RazorPagesMovie.Models.Movie> Movie { get; set; }
        }
    }
```
