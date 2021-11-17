---
template: blog-post
title: Browser DOM and Component Events
slug: Blazor-dom-and-component-events
date: 2021-08-23 21:28
description: Blazor is a new single-page application (SPA) framework form
  Microsoft. NET framework in favor of JavaScript. The Document Object Model
  (DOM) is a platform and language agnostic interface that treats an XML or HTML
  document as a tree structure
featuredImage: /assets/blazor-series.png
---
# Browser DOM events

When rendering any mark-up, it is possible to assign standard JavaScript events on the rendered HTML elements so that our own Blazor C# methods are executed. For example, we have used the `@onclick` Directive in many samples elsewhere:

```
<button @onclick=ButtonClicked>Click me</button>
```

These event directives have full IntelliSense support within the Visual Studio editor, so starting to type the `@` symbol should present us with a comprehensive list of available directives, along with a description identifying which argument class type the event passes us in our event handler. DOM events start with `@on`.

> Sets the ‘@onabort’ attribute to the provided string or delegate value. A delegate value should be of type ‘Microsoft.AspNetCore.Components.Web.ProgressEventArgs’

**Warning:** When writing a Blazor app that runs entirely on the server, Blazor will hook events in the browser and send them to server so our C# methods can be invoked. This can lead to a noticeable slow-down for frequently fired events such as `onmousemove`.

**Note:** Because JavaScript invocation of C# methods is asynchronous, this means that in C# methods we cannot cancel events as we can in JavaScript. This is because cancelling browser DOM events is a synchronous operation, by the time our C# has been asynchronously invoked it is already too late to cancel the event .

## Component Events

The `EventCallback<T>` class is a special Blazor class that can be exposed as a Parameter so that components can easily notify consumers when something of interest has occurred.

Once a public property of type `EventCallback<T>` has been declared and decorated with the `[Parameter]` attribute, consuming components can specify in Razor mark-up which method to call when the event is triggered.

## Adding an event to the Counter component

In a new Blazor app, edit the **/Pages/Counter.razor** file and add a new callback parameter.

```
[Parameter]

public EventCallback<int> OnMultipleOfThree { get; set; }
```

This declares a new `EventCallback` named OnMultipleOfThree that any consuming component can register an interest in. The `<int>` specifies that the value emitted by the event callback will be a `System.Int32`.

Now if we edit the **IncrementCount** method we can emit this event whenever the counter is incremented to a multiple of 3.

```c#
private async Task IncrementCount()

{

currentCount++;

**if** (currentCount % 3 == 0)

await OnMultipleOfThree.InvokeAsync(currentCount);

}
```

## Subscribing to EventCallback<T>

Edit the **/Pages/Index.razor** page so that we embed the Counter component and subscribe to its **OnMultipleOfThree** event. Change its mark-up to the following.

@page "/"

Last multiple **of** three = @LastMultipleOfThree

```c#
<Counter OnMultipleOfThree=@UpdateLastMultipleOfThreeValue/>

@code

{

  int LastMultipleOfThree = 0;

  private void

  {

     LastMultipleOfThree = value;

  }

}
```

* **Line 9**\
  Declares a class member of type `int` that stores the last multiple of 3 value.
* **Line 3**\
  Displays the value of LastMultipleOfThree
* **Line 5**\
  Embeds the **Counter** component and sets its OnMultipleOfThree event to execute the **UpdateLastMultipleOfThreeValue** method when the event is emitted.
* **Line 11**\
  The value received from the event is used to update the value of **LastMultipleOfThree**.

## Adding Actions and EventCallback

To communicate changes, we can use `EventCallback`. `EventCallback<T>` differs a bit from what we might be used to in .NET. `EventCallback<T>` is a class that is specially made for Blazor to be able to have the event callback exposed as a parameter for the component.

In .NET in general, you can add multiple listeners to an event (multi-cast), but with `EventCallback<T>`, you will only be able to add one listener (single-cast).

It is worth mentioning that you can, of course, use events the way you are used to from .NET in Blazor as well. However, you probably want to use `EventCallback<T>` because there are many upsides to using `EventCallback`over traditional .NET events.

.NET events use classes and `EventCallback` uses structs. This means that in Blazor, we don't have to perform a null check before calling `EventCallback` because a struct cannot be null.

`EventCallback` is asynchronous and can be awaited. When `EventCallback` has been called, Blazor will automatically execute `StateHasChanged` on the consuming component to make sure the component updates (if it needs to be updated).

So, if you require multiple listeners, you can use `Action<T>`, otherwise, you should use `EventCallback<T>`.

Some event types have event arguments that we can access. They are optional, so in most cases, you don't need to add them. You can add them by specifying them in a method or you can use a lambda expression, like this:

```
<button @onclick="@((e)=>message=$"x:{e.ClientX} y:{e.ClientY}")">Click me</button>
```

`button` will set a variable called `message` to a string containing the mouse coordinates. The lambda has one parameter, `e`, which is of the `MouseArgs` type. You don't have to specify the type, however. The compiler understands what type the parameter is.