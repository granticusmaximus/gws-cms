---
template: blog-post
title: "Deep Dive: Blazor Data Binding"
slug: /blazor-data-binding
date: 2021-02-16 12:23
description: By now, I have shared with you a few areas of Blazor, and why I
  think that it is an amazing move by Microsoft. I truly hope to see more added
  to this arena in the future. Today, I wanted to talk about data binding in
  Blazor.
---
By now, I have shared with you a few areas of Blazor, and why I think that it is an amazing move by Microsoft. I truly hope to see more added to this arena in the future. Today, I wanted to talk about data binding in Blazor.

Data binding is one of the most powerful features of software development technologies. Data binding is the connection bridge between View and the business logic (View Model) of the application. The following are the ways of doing data banding with Blazor. There are currently 2 areas to consider:

* One Way Binding
* Two Way Binding

Binding data is a fundamental task in single page applications (SPAs). At some point every application needs to either display data (e.g. labels) or receive data (e.g. forms). 

While most SPA frameworks have similar concepts for data binding, either one way binding or two way binding, the way they work and are implemented varies widely. In this post, we're going to have a good look at how one way and two way binding work in Blazor.

## One Way Binding

![one-way-binding-1.jpg](https://images.squarespace-cdn.com/content/v1/5e98fa59ccda362534ca8e48/1613257940053-6B1UDWWE2OL4NEDSMJAL/one-way-binding-1.jpg?format=500w)

One way bindings have a unidirectional flow, meaning that updates to the value only flow one way. A couple of examples of one way binding are rendering a label dynamically or dynamically outputting a CSS class name in markup. 

One way bindings can be constant and not change, but often the value will have a reason to be updated. Otherwise it would probably be better to avoid using a bound value altogether and just type the value out directly. 

In Blazor, when modifying a one way binding the application is going to be responsible for making the change. This could be in response to user action or event such as a button click. The point being, that the user will never be able to modify the value directly, hence one way binding.

Now we have an idea what one way binding is let's take a look at some examples.

```csharp
<h1>@Title</h1> 

@code { private string Title { get; set; } = "Hello, World!"; }
```

In the code above, we have a component which displays a heading. The contents of that heading, `Title`, is a one way bound value. In order to bind one way values we use the `@` symbol followed by the property, the field or even the method we want to bind too.

```csharp
<h1>@Title</h1> 

<button @onclick="UpdateTitle">Update Title</button> 

@code { 

private string Title { get; set; } = "Hello, World!"; 

private void UpdateTitle() 

   { 

        Title = "Hello, Blazor!";

   } 

}
```

In the first example the value is set and never changed, in this example we've added a method which updates the value of `Title` when the button is clicked. 

As we talked about previously, values can only be updated in one direction and here we can see that in action. When the buttons `onclick` event is triggered the `UpdateTitle` method is called and `Title` property is updated to the new value. Executing event handlers in Blazor triggers a re-render which updates the UI.



## One Way Binding Between Components

In the previous examples, we looked at one way binding inside of a component. But what if we want one way binding across components? Using our previous example, say we wanted to display the title of a parent component in a child component, how could we achieve this? By using *component parameters*. 

```csharp
<!-- Parent Component -->

<h1>@Title</h1>

<button @onclick="UpdateTitle">Update Title</button>

<ChildComponent ParentsTitle="Title" />

@code {
    private string Title { get; set; } = "Hello, World!";

    private void UpdateTitle()
    {
        Title = "Hello, Blazor!";
    }
}
<!-- Child Component -->

<h2>Parent Title is: @ParentsTitle</h2>

@code {
    [Parameter] public string ParentsTitle { get; set; }
}
```

In the example, the parent component is passing its title into the child component via the child components `ParentsTitle` *component parameter*. When then components are first rendered the headings will be the following.

```csharp
<!-- Parent Component -->
<h1>Hello, World!</h1>

<!-- Child Component -->
<h2>Parent Title is: Hello, World!</h2>
```

When the *Update Title* button is pressed then the output will become the following.

```csharp
<!-- Parent Component -->
<h1>Hello, Blazor!</h1>

<!-- Child Component -->
<h2>Parent Title is: Hello, Blazor!</h2>
```

Similar to what happened with the earlier example inside a single component. The button click event calls the `UpdateTitle` method and the `Title` property is updated. Then the running of the event handler triggers a re-render of the parent component. 

This also updates the `Title` parameter passed to the child component. Updating the component parameter triggers a re-render of the child component, updating its UI with the new title.

\
\
Two way binding

Now we know all about one way binding, you could probably guess that two way bindings have a bidirectional flow. Allowing values to be updated from two directions. 

The primary use case for two way binding is in forms, although it can be used anywhere that an application requires input from the user. The primary method of achieving two way binding in Blazor is to use the `bind` attribute. 

### The Bind Attribute

The `bind` attribute is a very versatile tool for binding in Blazor and has 3 different forms which allows developers to be very specific about how they want binding to occur.

* `@bind=Property`
* `@bind-Value=Property`
* `@bind-Value=Property` `@bind-Value:event="onevent"`

We're going to look at each of these over the next few examples to see how they work.

### Basic Two Way Binding

```csharp
<h1>@Title</h1>

<input @bind="@Title" />

@code {
    private string Title { get; set; } = "Hello, World!";
}
```

Continuing with our previous examples, we have added an input control which is two way bound to the `Title`value using the `bind` attribute. If you run this code you will notice that the value of `Title` doesn't actually update until you tab out of the input. 

This is because under the covers `bind` is actually setting the `value` attribute of the input to `Title` and setting up a `onchange` handler which will update `Title` when the input loses focus. We can see this if we look at lines 8 and 9 of the compiled components `BuildRenderTree` method.

```csharp
protected override void BuildRenderTree(Microsoft.AspNetCore.Components.RenderTree.RenderTreeBuilder builder)
{
    builder.OpenElement(0, "h1");
    builder.AddContent(1, Title);
    builder.CloseElement();
    builder.AddMarkupContent(2, "\r\n\r\n");
    builder.OpenElement(3, "input");
    builder.AddAttribute(4, "value", Microsoft.AspNetCore.Components.BindMethods.GetValue(Title));
    builder.AddAttribute(5, "onchange", Microsoft.AspNetCore.Components.EventCallback.Factory.CreateBinder(this, __value => Title = __value, Title));
    builder.CloseElement();
}
```

The `bind` attribute understands different control types, for example, a checkbox does not use a `value` attribute, it uses a `checked` attribute. `bind` knows this and will apply the correct attributes accordingly.

This is all good, but what if we want our `Title` to update as we type and not just when the input loses focus? Well, we can do that using a more specific version of `bind`.

### Two Way Binding To A Specific Event

```csharp
<h1>@Title</h1>

<input @bind-value="Title" @bind-value:event="oninput" />

@code {
    private string Title { get; set; } = "Hello, World!";
}
```

We can specify what event the `bind` attribute should use to handle updating the value. As we now know, by default this event is `onchange`, but in the example above we've specified the `oninput` event. This event is fired for each character typed so the `Title`s value is updated continually.

### Two Way Binding Between Components

To create two way binding between components we can once again take advantage of the `bind` attribute. We also need to setup our components with a certain convention, let's look at an example.

```csharp
<h1>@Title</h1>

<ChildComponent @bind-ParentsTitle="Title" />

<button @onclick="UpdateTitle">Update Title</button>

@code {
    private string Title { get; set; } = "Hello, World!";

    private void UpdateTitle()
    {
        Title = "Hello, Blazor!";
    }
}
<h2>Parent Title is: @ParentsTitle</h2>

<button @onclick="UpdateParentsTitle">Update Parents Title</button>
  

@code {

    [Parameter] public string ParentsTitle { get; set; }
    [Parameter] public EventCallback<string> ParentsTitleChanged { get; set; }

    private async Task UpdateParentsTitle()
    {
        ParentsTitle = "Hello, From Child Component!";
        await ParentsTitleChanged.InvokeAsync(ParentsTitle);
    }
}
```

We're adapting our previous example from one way binding and making it two way. There's now a button on the child component which triggers a method to update the `ParentsTitle` and invokes the `ParentsTitleChanged`EventCallback. The parent component has also been updated to use `bind-ParentTitle` when passing its `Title` parameter to the child component. 

If we run the above code we're able to click the button on the parent or the button on the child and both components titles will be updated. So how does this work?

The two key factors here are the EventCallback on the child component and the use of the `bind` attribute on the child component in the parent. By using this version of `bind` in the parent, it's the equivalent of writing this.

```
<ChildComponent @bind-ParentsTitle="Title" @bind-ParentsTitle:event="ParentsTitleChanged" />

```

By default Blazor will look for an event on the child component using the naming convention of {*PropertyName}Changed*. This allows us to use the version of `bind`, only specifying the property name. It is important to note that the event property will need to be marked with the `Parameter` attribute.

However, it's also possible to use a completely different name for the EventCallback property, for example, `ParentsTitleUpdated`. But in this case we would need to use the long form version of bind and specify the event name like so.

```
<ChildComponent @bind-ParentsTitle="Title" @bind-ParentsTitle:event="ParentsTitleUpdated" />
```