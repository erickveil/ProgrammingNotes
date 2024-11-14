---
title: How to Add MVC Database to your Blazor Application
layout: note
tags:
  - CSharp
  - net
  - asp
  - mvc
  - blazor
  - sql
  - database
---


You've created a Blazor application, but you need to interact with the database. It turns out that Blazor pages interact with databases differently than MVC Views.

You can add the database, model first, using your existing Blazor model by following the code-first steps I set out in a [previous blog post on the subject](https://erickveil.github.io/c%23,/asp.net,/asp,/.net,/crud,/model/first,/model,/mvc/2022/10/13/How-to-Create-a-CRUD-Application-in-ASP.NET-6-Using-the-Code-First-Approach.html), but you will have a little more work to do.

# Annotate The Model Before Migrating

Before you perform a migration, annotate your model to help the process:

```c#
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace BlazorMoodTracker.Data
{
    public class MoodEntry
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Required]
        public DateTime Date { get; set; }

        [Range(1, 5)]
        public int Mood { get; set; }
    }
}
```

Then you're going to migrate the model like I had posted about earlier.

# Blazor Pages Vs MVC Views


## Blazor:

In your Blazor project, you have your pages set up like this:

- Pages
	- _Host.cshtml
	- _Layout.cshtml
	- Index.razor
- appsettings.json
- Program.cs
	
In your `Program.cs`, you most likely have this line:

```c#
app.MapFallbackToPage("/_Host");
```

And in `_Host.cshtml` you likely have this bit:

```
@{
	Layout = "_Layout";
}
```

So what you're going to get if your URI is at the root is Host sets the Layout, and where that renders the body will be your `Index.razor.`

## MVC:

But in MVC, your Views are laid out like this:

- Pages
- Views
	-MoodEntries
		- Create.cshtml
		- Delete.cshtml
		- Details.cshtml
		- Edit.cshtml
		- Index.cshtml
		
Since this is a Blazor project, it's likely that we will want to use the Pages instead of the Views. But `Index.razor` and `Index.cshtml` are not created equal!

# Getting the MVC Pages to Work

This part is optional. It's more for if you want to use the MVC Views instead of the Blazor Pages.

If you run after creating a migration and setting up your MVC views, you will notice right off the bat, that you can't even navigate to your MVC Views. If you try to enter the URI `https://localhost/Index` or whatever your local instance is running from, the page won't be found.

You'll have to add the following to get that running:

**`Prtogram.cs`**

```c#
builder.Services.AddControllersWithViews();

...

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=MoodEntries}/{action=Index}/{id?}");

```

***`appsetting.json`***
Your migration will probably set up the database address automatically. Just make sure it's there.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "BlazorMoodTrackerContext": "Server=(localdb)\\mssqllocaldb;Database=ChangeThisToYourDatabaseID;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

Now you should be able to navigate to the page, but you'll notice that your TagHelpers don't work.

For example, in `Views/MoodEntries/Index.cshtml`, you have this code:

```html
@model IEnumerable<BlazorMoodTracker.Data.MoodEntry>

<p>There are @Model.Count() mood entries.</p>
```

But the `@model` is not injected, and no matter what's in your database `@Model.Count()` will always return `0`. 

None of your `asp-action` or the like will work either. Any anchors that relied on them will appear to be normal text with no hyperlinks.

To fix, you need to manually add the file: `Views/ViewImport.cshtml`:

```html
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using BlazorMoodTracker.Data
@namespace BlazorMoodTracker.Views
@{
    Layout = "_Layout";
}
```

Notice how this looks similar to `Pages/_Hosts.chstml`! In this case, however, we're making the TagHelpers available to the Views files. MVC will automatically look for and use this file. There is no need to reference it elsewhere.

# Accessing the Database Data in Blazor

In MVC, we have our nice tag helpers to interact with the Controller. Stuff like this:

```html
<dl class="row">
    <dt class = "col-sm-2">
        @Html.DisplayNameFor(model => model.Date)
    </dt>
    <dd class = "col-sm-10">
        @Html.DisplayFor(model => model.Date)
    </dd>
    <dt class = "col-sm-2">
        @Html.DisplayNameFor(model => model.Mood)
    </dt>
    <dd class = "col-sm-10">
        @Html.DisplayFor(model => model.Mood)
    </dd>
</dl>
```

But that won't work in a Blazor Page. For that, we will need to inject some C# classes and use the data that way. In Blazor, the `@model` directive is not used in the same way as in Razor Pages or MVC views. Instead, you work with properties and fields within the `@cod`e block to manage your data.

## Create the Interface

You'll need to set up a service to access your data. This is basically going to behave like your Controller in MVC.

In your `Data/` directory, set up your service interface `IMoodEntryService.cs`:

```c#
namespace BlazorMoodTracker.Data
{
    public interface IMoodEntryService
    {
        Task<List<MoodEntry>> GetMoodEntriesAsync();
        Task AddMoodEntryAsync(MoodEntry moodEntry);
        Task<MoodEntry> GetMoodEntryByIdAsync(int id);
        Task UpdateMoodEntryAsync(MoodEntry moodEntry);
        Task DeleteMoodEntryAsync(int id);
    }
}
```

Then implement the concrete class `MoodEntryService.cs`:

```c#
using Microsoft.EntityFrameworkCore;

namespace BlazorMoodTracker.Data
{
    public class MoodEntryService : IMoodEntryService
    {
        private readonly BlazorMoodTrackerContext _context;

        public MoodEntryService(BlazorMoodTrackerContext context)
        {
            _context = context;
        }

        public async Task<List<MoodEntry>> GetMoodEntriesAsync()
        {
            return await _context.MoodEntry.ToListAsync();
        }

        public async Task AddMoodEntryAsync(MoodEntry moodEntry)
        {
            _context.MoodEntry.Add(moodEntry);
            await _context.SaveChangesAsync();
        }

        public async Task<MoodEntry> GetMoodEntryByIdAsync(int id)
        {
            return await _context.MoodEntry.FindAsync(id);
        }

        public async Task UpdateMoodEntryAsync(MoodEntry moodEntry)
        {
            _context.MoodEntry.Update(moodEntry);
            await _context.SaveChangesAsync();
        }

        public async Task DeleteMoodEntryAsync(int id)
        {
            var moodEntry = await _context.MoodEntry.FindAsync(id);
            if (moodEntry != null)
            {
                _context.MoodEntry.Remove(moodEntry);
                await _context.SaveChangesAsync();
            }
        }
    }
}
```

Then register the service in `Program.cs`:

```c#
using BlazorMoodTracker.Services;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddDbContext<BlazorMoodTrackerContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("BlazorMoodTrackerContext")));
builder.Services.AddScoped<IMoodEntryService, MoodEntryService>();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();

```

Finally, Inject and use the service in your Page:

```html
@page "/"
@using BlazorMoodTracker.Data
@inject IMoodEntryService MoodEntryService

<PageTitle>Index</PageTitle>

<h1>Mood Tracker</h1>

<p>Select your current mood:</p>

<div>
    @for (int i = 1; i <= 5; i++)
    {
        <button class="btn btn-primary m-1" @onclick="() => RecordMood(i)">@i</button>
    }
</div>

@if (moodEntries == null || moodEntries.Count == 0) {
    <p>No mood entries recorded yet.</p>
}
else {
    <table class="table">
        <thead>
            <tr>
                <th>Date</th>
                <th>Mood</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var entry in moodEntries)
            {
                <tr>
                    <td>@entry.Date.ToString("g")</td>
                    <td>@entry.Mood</td>
                </tr>
            }
        </tbody>
    </table>
}

@code {
    private List<MoodEntry> moodEntries;

    protected override async Task OnInitializedAsync()
    {
        moodEntries = await MoodEntryService.GetMoodEntriesAsync();
    }

    private async Task RecordMood(int mood)
    {
        var newEntry = new MoodEntry { Date = DateTime.Now, Mood = mood };
        await MoodEntryService.AddMoodEntryAsync(newEntry);
        moodEntries.Add(newEntry);
    }
}

```

Now you have a Service for Blazor (instead of a Controller for MVC) to interact with your Database from your Blazor app!
