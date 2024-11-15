---
title: How to Pass Data From Controller to View in am MVC Project
layout: note
tags:
  - CSharp
  - asp
  - net
---


`ViewData` is a dictionary object in ASP.NET Core MVC and Razor Pages that allows you to pass data from a controller or page model to a view. It is a part of the `ViewDataDictionary` class and is used to store and retrieve data using a key-value pair.

### Why Use ViewData?

1. **Flexibility**: ViewData can store any type of data, making it flexible for passing various types of information to the view.
2. **Dynamic Nature**: It is dynamic, meaning you don't need to define a strongly-typed model to pass data.
3. **Temporary Data**: It is useful for passing temporary data that does not require a strongly-typed model.

### Setting the Title with ViewData

In this MVC example, the title of the page is set using ViewData:

```
@{
    ViewData["Title"] = "Create";
}
```

This approach is often used because it allows the title to be dynamically set from the controller or page model, making it easier to manage and change titles across different views.

### Why Not Use `<PageTitle>`?

The `<PageTitle>` element is specific to Blazor components and is used to set the title of the page in Blazor applications. However, in traditional ASP.NET Core MVC or Razor Pages, `ViewData` is a more common and established way to set the title and other metadata.

### Example of Using `<PageTitle>` in Blazor

If you were using a Blazor component, you could set the title like this:

```html
<PageTitle>My Page Title</PageTitle>
```

### Key Members of ViewData

1. **Keys**: Gets a collection containing the keys in the `ViewDataDictionary`.
2. **Values**: Gets a collection containing the values in the `ViewDataDictionary`.
3. **Model**: Gets or sets the model object for the view.
4. **Count**: Gets the number of elements contained in the `ViewDataDictionary`.
5. **Item**: Gets or sets the value associated with the specified key.

### Common Methods

1. **Add**: Adds an element with the provided key and value to the `ViewDataDictionary`.
2. **ContainsKey**: Determines whether the ViewDataDictionary contains a specific key.
3. **Remove**: Removes the value with the specified key from the `ViewDataDictionary`.
4. **Clear**: Removes all keys and values from the `ViewDataDictionary`.
5. **TryGetValue**: Gets the value associated with the specified key.

You can actually set any arbitrary Key Value in the `ViewData` to make it available.

### Another Example

#### Controller:

Here we set values in the Controller. These values will be available in a View.

```c#
public class MoodEntriesController : Controller
{
    public IActionResult Create()
    {
        ViewData["Title"] = "Create Mood Entry";
        ViewData["Message"] = "Please fill out the form below to create a new mood entry.";
        return View();
    }
}
```

#### View:

Here we set and use values in the View itself. Although, you will usually use this to set values in a Controller, you can also set values in a View to be made available in sub-views.

```html
@{
    ViewData["Title"] = "Create";
    var message = ViewData["Message"] as string;
}

<h1>@ViewData["Title"]</h1>
<p>@message</p>

<!-- Form and other content here -->
```






