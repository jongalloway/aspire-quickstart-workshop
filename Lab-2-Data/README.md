# Tutorial: Connect an ASP.NET Core app to SQL Server using .NET Aspire and Entity Framework Core

In this tutorial, you create an ASP.NET Core app that uses a .NET Aspire Entity Framework Core SQL Server component to connect to SQL Server to read and write support ticket data. [Entity Framework Core](https://learn.microsoft.com/ef/core/) is a lightweight, extensible, open source object-relational mapper that enables .NET developers to work with databases using .NET objects. You'll learn how to:

> - Create a basic .NET app that is set up to use .NET Aspire components
> - Add a .NET Aspire component to connect to SQL Server
> - Configure and use .NET Aspire Component features to read and write from the database

> [!NOTE]  
> This tutorial is based on the [Connect an ASP.NET Core app to SQL Server using .NET Aspire and Entity Framework Core](https://learn.microsoft.com/dotnet/aspire/database/sql-server-components) tutorial in Microsoft Learn.

## Create the sample solution

1. At the top of Visual Studio, navigate to **File** > **New** > **Project**.
1. In the dialog window, search for *Blazor* and select **Blazor Web App**. Choose **Next**.
1. On the **Configure your new project** screen:
    - Enter a **Project Name** of **AspireSQLEFCore**.
    - Leave the rest of the values at their defaults and select **Next**.
1. On the **Additional information** screen:
    - Make sure **.NET 8.0** is selected.
    - Ensure the **Interactive render mode** is set to **None**.
    - Check the **Enlist in .NET Aspire orchestration** option and select **Create**.

Visual Studio creates a new ASP.NET Core solution that is structured to use .NET Aspire. The solution consists of the following projects:

- **AspireSQLEFCore**: A Blazor project that depends on service defaults.
- **AspireSQLEFCore.AppHost**: An orchestrator project designed to connect and configure the different projects and services of your app. The orchestrator should be set as the startup project.
- **AspireSQLEFCore.ServiceDefaults**: A shared class library to hold configurations that can be reused across the projects in your solution.

## Create the database model and context classes

To represent a user submitted support request, add the following `SupportTicket` model class at the root of the *AspireSQLEFCore* project.

```csharp
using System.ComponentModel.DataAnnotations;

namespace AspireSQLEFCore;

public sealed class SupportTicket
{
    public int Id { get; set; }
    [Required]
    public string Title { get; set; } = string.Empty;
    [Required]
    public string Description { get; set; } = string.Empty;
}
```

Add the following `TicketDbContext` data context class at the root of the **AspireSQLEFCore** project. The class inherits `DbContext` to work with Entity Framework and represent your database.

```csharp
using Microsoft.EntityFrameworkCore;
using System.Reflection.Metadata;

namespace AspireSQLEFCore;

public class TicketContext(DbContextOptions options) : DbContext(options)
{
    public DbSet<SupportTicket> Tickets => Set<SupportTicket>();
}
```

## Add the .NET Aspire component to the Blazor app

Add the [.NET Aspire Entity Framework Core Sql Server library](https://learn.microsoft.com/dotnet/aspire/database/sql-server-entity-framework-component?tabs=dotnet-cli) package to your *AspireSQLEFCore* project:

```dotnetcli
dotnet add package Aspire.Microsoft.EntityFrameworkCore.SqlServer --prerelease
```

Your *AspireSQLEFCore* project is now set up to use .NET Aspire components. Here's the updated *AspireSQLEFCore.csproj* file:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
   <PackageReference Include="Aspire.Microsoft.EntityFrameworkCore.SqlServer" Version="8.0.0-preview.6.24214.1" />
   <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="8.0.4">
     <PrivateAssets>all</PrivateAssets>
     <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
   </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\AspireSQLEFCore.ServiceDefaults\AspireSQLEFCore.ServiceDefaults.csproj" />
  </ItemGroup>

</Project>
```

## Configure the .NET Aspire component

In the *Program.cs* file of the *AspireSQLEFCore* project, add a call to the `AddSqlServerDbContext` extension method after the creation of the `builder` but before the call to `AddServiceDefaults`. For more information, see [.NET Aspire service defaults](https://learn.microsoft.com/dotnet/aspire/fundamentals/service-defaults). Provide the name of your connection string as a parameter.

```csharp
using AspireSQLEFCore;
using AspireSQLEFCore.Components;

var builder = WebApplication.CreateBuilder(args);
builder.AddSqlServerDbContext<TicketContext>("sqldata");

builder.AddServiceDefaults();

// Add services to the container.
builder.Services.AddRazorComponents().AddInteractiveServerComponents();

var app = builder.Build();

app.MapDefaultEndpoints();
```

This method accomplishes the following tasks:

- Registers a `TicketDbContext` with the DI container for connecting to the containerized Azure SQL Database.
- Automatically enable corresponding health checks, logging, and telemetry.

## Migrate and seed the database

While developing locally, you need to create a database inside the SQL Server container. Update the *Program.cs* file with the following code to automatically run Entity Framework migrations during startup.

```csharp
using AspireSQLEFCore;
using AspireSQLEFCore.Components;

var builder = WebApplication.CreateBuilder(args);
builder.AddSqlServerDbContext<TicketContext>("sqldata");

builder.AddServiceDefaults();

// Add services to the container.
builder.Services.AddRazorComponents().AddInteractiveServerComponents();

var app = builder.Build();

app.MapDefaultEndpoints();

if (app.Environment.IsDevelopment())
{
    using (var scope = app.Services.CreateScope())
    {
        var context = scope.ServiceProvider.GetRequiredService<TicketContext>();
        context.Database.EnsureCreated();
    }
}
else
{
    app.UseExceptionHandler("/Error", createScopeForErrors: true);
    // The default HSTS value is 30 days.
    // You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}
```

## Create the form

The app requires a form for the user to be able to submit support ticket information and save the entry to the database.

Use the following Razor markup to create a basic form, replacing the contents of the *Home.razor* file in the *AspireSQLEFCore/Components/Pages* directory:

```razor
@page "/"
@inject TicketContext context

<div class="row">
    <div class="col-md-6">
        <div>
            <h1 class="display-4">Request Support</h1>
        </div>
        <EditForm Model="@Ticket" FormName="Tickets" method="post"
                                OnValidSubmit="@HandleValidSubmit" class="mb-4">
            <DataAnnotationsValidator />
            <div class="mb-4">
                <label>Issue Title</label>
                <InputText class="form-control" @bind-Value="@Ticket.Title" />
                <ValidationMessage For="() => Ticket.Title" />
            </div>
            <div class="mb-4">
                <label>Issue Description</label>
                <InputText class="form-control" @bind-Value="@Ticket.Description" />
                <ValidationMessage For="() => Ticket.Description" />
            </div>
            <button class="btn btn-primary" type="submit">Submit</button>
            <button class="btn btn-danger mx-2" type="reset" @onclick=@ClearForm>Clear</button>
        </EditForm>

        <table class="table table-striped">
            @foreach (var ticket in Tickets)
            {
                <tr>
                    <td>@ticket.Id</td>
                    <td>@ticket.Title</td>
                    <td>@ticket.Description</td>
                </tr>
            }
        </table>
    </div>
</div>

@code {
    [SupplyParameterFromForm]
    private SupportTicket Ticket { get; set; } = new();
    private List<SupportTicket> Tickets = [];

    private void ClearForm()
    {
        Ticket = new();
        StateHasChanged();

    }

    protected override async Task OnInitializedAsync()
    {
        Tickets = await context.Tickets.ToListAsync();
    }

    private async Task HandleValidSubmit()
    {
        context.Tickets.Add(Ticket);
        await context.SaveChangesAsync();
        Tickets = await context.Tickets.ToListAsync();
    }
}
```

For more information about creating forms in Blazor, see [ASP.NET Core Blazor forms overview](https://learn.microsoft.com/aspnet/core/blazor/forms).

## Configure the AppHost

The *AspireSQLEFCore.AppHost* project is the orchestrator for your app. It's responsible for connecting and configuring the different projects and services of your app. The orchestrator should be set as the startup project.

Add the [.NET Aspire Entity Framework Core Sql Server library](https://learn.microsoft.com/dotnet/aspire/database/sql-server-entity-framework-component?tabs=dotnet-cli) package to your *AspireStorage.AppHost* project:

```dotnetcli
dotnet add package Aspire.Microsoft.EntityFrameworkCore.SqlServer --prerelease
```

Replace the contents of the *Program.cs* file in the *AspireSQLEFCore.AppHost* project with the following code:

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var sql = builder.AddSqlServer("sql")
                 .AddDatabase("sqldata");

builder.AddProject<Projects.AspireSQLEFCore>("aspiresql")
       .WithReference(sql);

builder.Build().Run();
```

The preceding code adds a SQL Server Container resource to your app and configures a connection to a database called `sqldata`. The Entity Framework classes you configured earlier will automatically use this connection when migrating and connecting to the database.

## Run and test the app locally

The sample app is now ready for testing. Verify that the submitted form data is persisted to the database by completing the following steps:

1. Select the run button at the top of Visual Studio (or <kbd>F5</kbd>) to launch your .NET Aspire app dashboard in the browser.
1. On the projects page, in the **AspireSQLEFCore** row, click the link in the **Endpoints** column to open the UI of your app.

    ![A screenshot showing the home page of the .NET Aspire support application.](https://learn.microsoft.com/dotnet/aspire/docs/database/media/app-home-screen.png)

1. Enter sample data into the `Title` and `Description` form fields.
1. Select the **Submit** button, and the form submits the support ticket for processing — and clears the form.
1. The data you submitted displays in the table at the bottom of the page when the page reloads.
