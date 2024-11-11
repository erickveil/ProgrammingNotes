---
title: How Root Routing Works In Blazor
layout: note
tags:
  - CSharp
  - asp
  - net
  - blazor
  - razor
---

### `@page` Directive and Routing

The `@page "/"` directive in `Index.razor` specifies that this component should handle requests to the root URL ("/"). ` _Host.cshtml` also has the same root routing. Let's look at how this works:

1.	**`_Host.cshtml`**: This file is typically used in Blazor Server applications as the entry point for the Blazor application. It sets up the initial HTML structure and includes the Blazor script to bootstrap the Blazor application. It usually contains a `<component>` tag that specifies the root component of the Blazor app.
2.	**`Index.razor`**: This is a Blazor component that handles the root URL ("/") within the Blazor application. When the Blazor app is running, it uses client-side routing to navigate between components.

There is no conflict because `_Host.cshtml` is responsible for setting up the Blazor app, while `Index.razor` is responsible for handling the root URL within the Blazor app. The routing in `_Host.cshtml` is server-side, while the routing in `Index.razor` is client-side.
### Why `Index.razor` is Shown

When you run the application, `_Host.cshtml `sets up the Blazor app and specifies the root component. Typically, this is done using the `<component>` tag in `_Host.cshtml`, which might look something like this:

```html
<component type="typeof(App)" render-mode="ServerPrerendered" />
```

The App component is usually defined in `App.razor` and contains a Router component that handles client-side routing. Here is an example of what `App.razor` might look like:

```html
<Router AppAssembly="@typeof(Program).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
        <FocusOnNavigate RouteData="@routeData" Selector="h1" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>
```


### In this setup:

- The Router component looks at the URL and determines which Razor component to display.
- When the URL is "/", the Router component will render `Index.razor` because of the @page "/" directive.

### Summary

- **No Conflict**: `_Host.cshtml` sets up the Blazor app and handles server-side routing, while `Index.razor` handles client-side routing within the Blazor app.
- **`Index.razor` Display**: The Router component in `App.razor` determines which component to display based on the URL. When the URL is "/", it displays `Index.razor`.

This separation of server-side and client-side routing ensures that there is no conflict, and the Blazor app can handle navigation smoothly.